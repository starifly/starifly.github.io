---
title: kamailio 代理 sip 消息
author: starifly
date: 2020-03-31T22:15:07+08:00
lastmod: 2020-10-02
categories: [kamailio,freeswitch,sip]
tags: [kamailio,freeswitch,sip,opensips,linux]
draft: true
slug: kamailio-proxy-sip-message
---

最近帮客户对接 SIP 线路，因为线路方与 freeswitch 信令协商过程中给过来的 rtp 地址不能路由出去，于是想能不能通过什么方式修改 SDP 中的地址，方法是通过 opensips 或者 kamailio 这样的第三方开源软件代理 SIP 消息，在这个过程中修改 SDP 中的 rtp 地址。经过几番辛苦调试，查阅相关资料和官方文档，几经修改路由脚本，终于实现了代理并成功修改了 rtp 地址，正当准备庆祝之时，却发现虽然修改地址的目的达到了，却产生了另一个很矛盾的问题，因为 修改了 rtp 地址后，freeswitch 发现 rtp 通信的源地址端口和目的地址端口不一致，它又自动调整了 rtp 地址，到这里聪明的人一定会想到，既然这样，咱们就禁用你这个自动调整的功能，是的，确实可以禁用这个功能，但是禁用了之后又产生了另一个问题，因为 freeswitch 不支持 rtp 源地址端口和目的地址端口不一致的情况，这就导致虽然能收到 rtp 包，freeswitch 却不会转发给内线。至此好像无解，其实可以经由中转服务器去做 SNAT 和 DNAT ，这样就能保证进来包的源地址和发出包的目的地址是一样的。

```shell
iptables -t nat -A PREROUTING -s 192.168.5.140 -d 192.168.5.127 -p udp --dport 6000:10000 -j DNAT --to-destination 58.251.18.141
iptables -t nat -A POSTROUTING -d 58.251.18.141 -p udp --dport 6000:10000 -j SNAT --to 192.168.5.127

iptables -t nat -A PREROUTING -s 192.168.5.140 -d 192.168.5.127 -p udp --dport 5187 -j DNAT --to-destination 120.79.174.58
iptables -t nat -A POSTROUTING -d 120.79.174.58 -p udp --dport 5187 -j SNAT --to 192.168.5.127

iptables -t nat -A PREROUTING -s 58.251.18.141 -d 192.168.5.127 -p udp --dport 10000:10040 -j DNAT --to-destination 192.168.5.140
iptables -t nat -A POSTROUTING -d 192.168.5.140 -p udp --dport 10000:10040 -j SNAT --to 192.168.5.127

iptables -t nat -A PREROUTING -s 120.79.174.58 -d 192.168.5.127 -p udp --dport 5078 -j DNAT --to-destination 192.168.5.140
iptables -t nat -A POSTROUTING -d 192.168.5.140 -p udp --dport 5078 -j SNAT --to 192.168.5.127
```

下面记录一下其中的一些点。

1. 要使 freeswitch 能通过 kamailio 转发消息，要在 gw.xml 文件中添加如下配置（点对点模式）：

```conf
<!--param name="proxy" value="192.168.5.127:5078"/-->
<param name="outbound-proxy" value="192.168.5.127:5078"/>
```

其中“192.168.5.104:5078”为 kamailio 监听的地址和端口。

2. sip 报文中 contact ：包含的SIP/SIPS URI是UA希望用来接收请求的地址，后续请求可以用它来联系到当前UA。 如果代理服务器没有插入Record-Route字段来希望自己留在后续请求消息的传输路径上，那么可以忽略这些代理服务器，后续请求直接用Contact字段的URI来通讯。 当Contact中包含一个显示名称时，带有所有的URI参数的URI应该放入尖括号<>中。

3. 外线挂机的时候，BYE 请求没有正常发送到 freeswitch，所以要用 fix_contact 函数用于修复 contact 地址

4. 用 $si 变量来区分 kamailio 代理两端的通信

5. fix_nat_sdp函数参数用法

fix_nated_sdp(flags [, ip_address [, sdp_fields]])函数解析：  
flags必选参数，该值可以是以下值或以下值的按位或得到的值：  
         0x01 --在SDP中增加“a=direction:active”行;  
         0x02 --使用消息的源地址或者ip_address参数指定的Ip地址重写SDP中媒体IP地址（“c=”）。  
         0x04--在SDP中增加”a=nortpproxy:yes”行。  
         0x08 --使用消息的源地址或者ip_address参数指定的Ip地址重写SDP中源IP地址（“o=”）。  
         0x10 --强制重写空的媒体IP和空的源IP地址。如果没有此标志，空IP将保持不变。  
ip_address可选参数，指定用来重写SDP的IP，如果不指定该参数，默认使用信令消息的源地址。  
sdp_fields可选参数，指定要附加到SDP消息后的sdp字段，每个sdp字段前面必须有“\r\n”

6. kamailio 安装

方式一：源代码编译安装

```shell
yum install bison flex mysql-server mysql-client libmysqlclient-dev libncurses5 libncurses5-dev mysql-devel ncurses-devel bison pcre-devel libpcap-devel
tar xzvf  kamailio-5.3_src.tar.gz
cd /opt/kamailio/kamailio-5.3
make FLAVOUR=kamailio cfg
make all && make install
cd /usr/local/etc/kamailio/ && mv kamailio.cfg{,.orig}
```

修改 kamailio 配置文件，有两个版本：

版本一：

```conf
#!KAMAILIO
#
# Kamailio (OpenSER) SIP Server v5.3 - default configuration script
#     - web: https://www.kamailio.org
#     - git: https://github.com/kamailio/kamailio
#
# Direct your questions about this file to: <sr-users@lists.kamailio.org>
#
# Refer to the Core CookBook at https://www.kamailio.org/wiki/
# for an explanation of possible statements, functions and parameters.
#
# Note: the comments can be:
#     - lines starting with #, but not the pre-processor directives,
#       which start with #!, like #!define, #!ifdef, #!endif, #!else, #!trydef,
#       #!subst, #!substdef, ...
#     - lines starting with //
#     - blocks enclosed in between /* */
#
# Several features can be enabled using '#!define WITH_FEATURE' directives:
#
# *** To run in debug mode:
#     - define WITH_DEBUG
#
# *** To enable mysql:
#     - define WITH_MYSQL
#
# *** To enable authentication execute:
#     - enable mysql
#     - define WITH_AUTH
#     - add users using 'kamctl'
#
# *** To enable IP authentication execute:
#     - enable mysql
#     - enable authentication
#     - define WITH_IPAUTH
#     - add IP addresses with group id '1' to 'address' table
#
# *** To enable persistent user location execute:
#     - enable mysql
#     - define WITH_USRLOCDB
#
# *** To enable presence server execute:
#     - enable mysql
#     - define WITH_PRESENCE
#
# *** To enable nat traversal execute:
#     - define WITH_NAT
#     - install RTPProxy: http://www.rtpproxy.org
#     - start RTPProxy:
#        rtpproxy -l _your_public_ip_ -s udp:localhost:7722
#     - option for NAT SIP OPTIONS keepalives: WITH_NATSIPPING
#
# *** To enable PSTN gateway routing execute:
#     - define WITH_PSTN
#     - set the value of pstn.gw_ip
#     - check route[PSTN] for regexp routing condition
#
# *** To enable database aliases lookup execute:
#     - enable mysql
#     - define WITH_ALIASDB
#
# *** To enable speed dial lookup execute:
#     - enable mysql
#     - define WITH_SPEEDDIAL
#
# *** To enable multi-domain support execute:
#     - enable mysql
#     - define WITH_MULTIDOMAIN
#
# *** To enable TLS support execute:
#     - adjust CFGDIR/tls.cfg as needed
#     - define WITH_TLS
#
# *** To enable XMLRPC support execute:
#     - define WITH_XMLRPC
#     - adjust route[XMLRPC] for access policy
#
# *** To enable anti-flood detection execute:
#     - adjust pike and htable=>ipban settings as needed (default is
#       block if more than 16 requests in 2 seconds and ban for 300 seconds)
#     - define WITH_ANTIFLOOD
#
# *** To block 3XX redirect replies execute:
#     - define WITH_BLOCK3XX
#
# *** To block 401 and 407 authentication replies execute:
#     - define WITH_BLOCK401407
#
# *** To enable VoiceMail routing execute:
#     - define WITH_VOICEMAIL
#     - set the value of voicemail.srv_ip
#     - adjust the value of voicemail.srv_port
#
# *** To enhance accounting execute:
#     - enable mysql
#     - define WITH_ACCDB
#     - add following columns to database
#!ifdef ACCDB_COMMENT
  ALTER TABLE acc ADD COLUMN src_user VARCHAR(64) NOT NULL DEFAULT '';
  ALTER TABLE acc ADD COLUMN src_domain VARCHAR(128) NOT NULL DEFAULT '';
  ALTER TABLE acc ADD COLUMN src_ip varchar(64) NOT NULL default '';
  ALTER TABLE acc ADD COLUMN dst_ouser VARCHAR(64) NOT NULL DEFAULT '';
  ALTER TABLE acc ADD COLUMN dst_user VARCHAR(64) NOT NULL DEFAULT '';
  ALTER TABLE acc ADD COLUMN dst_domain VARCHAR(128) NOT NULL DEFAULT '';
  ALTER TABLE missed_calls ADD COLUMN src_user VARCHAR(64) NOT NULL DEFAULT '';
  ALTER TABLE missed_calls ADD COLUMN src_domain VARCHAR(128) NOT NULL DEFAULT '';
  ALTER TABLE missed_calls ADD COLUMN src_ip varchar(64) NOT NULL default '';
  ALTER TABLE missed_calls ADD COLUMN dst_ouser VARCHAR(64) NOT NULL DEFAULT '';
  ALTER TABLE missed_calls ADD COLUMN dst_user VARCHAR(64) NOT NULL DEFAULT '';
  ALTER TABLE missed_calls ADD COLUMN dst_domain VARCHAR(128) NOT NULL DEFAULT '';
#!endif

####### Include Local Config If Exists #########
import_file "kamailio-local.cfg"

####### Defined Values #########

# *** Value defines - IDs used later in config
#!ifdef WITH_MYSQL
# - database URL - used to connect to database server by modules such
#       as: auth_db, acc, usrloc, a.s.o.
#!ifndef DBURL
#!define DBURL "mysql://kamailio:kamailiorw@localhost/kamailio"
#!endif
#!endif
#!ifdef WITH_MULTIDOMAIN
# - the value for 'use_domain' parameters
#!define MULTIDOMAIN 1
#!else
#!define MULTIDOMAIN 0
#!endif

# - flags
#   FLT_ - per transaction (message) flags
#	FLB_ - per branch flags
#!define FLT_ACC 1
#!define FLT_ACCMISSED 2
#!define FLT_ACCFAILED 3
#!define FLT_NATS 5

#!define FLB_NATB 6
#!define FLB_NATSIPPING 7

####### Global Parameters #########

### LOG Levels: 3=DBG, 2=INFO, 1=NOTICE, 0=WARN, -1=ERR
#!ifdef WITH_DEBUG
debug=4
log_stderror=yes
#!else
debug=2
log_stderror=no
#!endif

memdbg=5
memlog=5

log_facility=LOG_LOCAL0
log_prefix="{$mt $hdr(CSeq) $ci} "

/* number of SIP routing processes for each UDP socket
 * - value inherited by tcp_children and sctp_children when not set explicitely */
children=4

/* uncomment the next line to disable TCP (default on) */
disable_tcp=yes

/* number of SIP routing processes for all TCP/TLS sockets */
# tcp_children=8

/* uncomment the next line to disable the auto discovery of local aliases
 * based on reverse DNS on IPs (default on) */
# auto_aliases=no

/* add local domain aliases */
# alias="sip.mydomain.com"

/* uncomment and configure the following line if you want Kamailio to
 * bind on a specific interface/port/proto (default bind on all available) */
# listen=udp:10.0.0.10:5060
#listen=udp:eno16780032:5078
listen=udp:10.0.1.129:5060

/* life time of TCP connection when there is no traffic
 * - a bit higher than registration expires to cope with UA behind NAT */
tcp_connection_lifetime=3605

/* upper limit for TCP connections (it includes the TLS connections) */
tcp_max_connections=2048

#!ifdef WITH_TLS
enable_tls=yes

/* upper limit for TLS connections */
tls_max_connections=2048
#!endif

####### Custom Parameters #########

/* These parameters can be modified runtime via RPC interface
 * - see the documentation of 'cfg_rpc' module.
 *
 * Format: group.id = value 'desc' description
 * Access: $sel(cfg_get.group.id) or @cfg_get.group.id */

#!ifdef WITH_PSTN
/* PSTN GW Routing
 *
 * - pstn.gw_ip: valid IP or hostname as string value, example:
 * pstn.gw_ip = "10.0.0.101" desc "My PSTN GW Address"
 *
 * - by default is empty to avoid misrouting */
pstn.gw_ip = "" desc "PSTN GW Address"
pstn.gw_port = "" desc "PSTN GW Port"
#!endif

#!ifdef WITH_VOICEMAIL
/* VoiceMail Routing on offline, busy or no answer
 *
 * - by default Voicemail server IP is empty to avoid misrouting */
voicemail.srv_ip = "" desc "VoiceMail IP Address"
voicemail.srv_port = "5060" desc "VoiceMail Port"
#!endif

####### Modules Section ########

/* set paths to location of modules */
# mpath="/usr/lib/kamailio/modules/"

#!ifdef WITH_MYSQL
loadmodule "db_mysql.so"
#!endif

loadmodule "jsonrpcs.so"
loadmodule "kex.so"
loadmodule "corex.so"
loadmodule "tm.so"
loadmodule "tmx.so"
loadmodule "sl.so"
loadmodule "rr.so"
loadmodule "pv.so"
loadmodule "maxfwd.so"
loadmodule "usrloc.so"
loadmodule "registrar.so"
loadmodule "textops.so"
loadmodule "textopsx.so"
loadmodule "siputils.so"
loadmodule "xlog.so"
loadmodule "sanity.so"
loadmodule "ctl.so"
loadmodule "cfg_rpc.so"
loadmodule "acc.so"
loadmodule "counters.so"
loadmodule "nathelper.so"
loadmodule "nat_traversal.so"
loadmodule "dialog.so"

#!ifdef WITH_AUTH
loadmodule "auth.so"
loadmodule "auth_db.so"
#!ifdef WITH_IPAUTH
loadmodule "permissions.so"
#!endif
#!endif

#!ifdef WITH_ALIASDB
loadmodule "alias_db.so"
#!endif

#!ifdef WITH_SPEEDDIAL
loadmodule "speeddial.so"
#!endif

#!ifdef WITH_MULTIDOMAIN
loadmodule "domain.so"
#!endif

#!ifdef WITH_PRESENCE
loadmodule "presence.so"
loadmodule "presence_xml.so"
#!endif

#!ifdef WITH_NAT
loadmodule "nathelper.so"
loadmodule "rtpproxy.so"
#!endif

#!ifdef WITH_TLS
loadmodule "tls.so"
#!endif

#!ifdef WITH_ANTIFLOOD
loadmodule "htable.so"
loadmodule "pike.so"
#!endif

#!ifdef WITH_XMLRPC
loadmodule "xmlrpc.so"
#!endif

#!ifdef WITH_DEBUG
loadmodule "debugger.so"
#!endif

# ----------------- setting module-specific parameters ---------------


# ----- jsonrpcs params -----
modparam("jsonrpcs", "pretty_format", 1)
/* set the path to RPC fifo control file */
# modparam("jsonrpcs", "fifo_name", "/var/run/kamailio/kamailio_rpc.fifo")
/* set the path to RPC unix socket control file */
# modparam("jsonrpcs", "dgram_socket", "/var/run/kamailio/kamailio_rpc.sock")

# ----- ctl params -----
/* set the path to RPC unix socket control file */
# modparam("ctl", "binrpc", "unix:/var/run/kamailio/kamailio_ctl")

# ----- tm params -----
# auto-discard branches from previous serial forking leg
modparam("tm", "failure_reply_mode", 3)
# default retransmission timeout: 30sec
modparam("tm", "fr_timer", 30000)
# default invite retransmission timeout after 1xx: 120sec
modparam("tm", "fr_inv_timer", 120000)

# ----- rr params -----
# set next param to 1 to add value to ;lr param (helps with some UAs)
modparam("rr", "enable_full_lr", 0)
# do not append from tag to the RR (no need for this script)
modparam("rr", "append_fromtag", 0)

# ----- registrar params -----
modparam("registrar", "method_filtering", 1)
/* uncomment the next line to disable parallel forking via location */
# modparam("registrar", "append_branches", 0)
/* uncomment the next line not to allow more than 10 contacts per AOR */
# modparam("registrar", "max_contacts", 10)
/* max value for expires of registrations */
modparam("registrar", "max_expires", 3600)
/* set it to 1 to enable GRUU */
modparam("registrar", "gruu_enabled", 0)

# ----- acc params -----
/* what special events should be accounted ? */
modparam("acc", "early_media", 0)
modparam("acc", "report_ack", 0)
modparam("acc", "report_cancels", 0)
/* by default ww do not adjust the direct of the sequential requests.
 * if you enable this parameter, be sure the enable "append_fromtag"
 * in "rr" module */
modparam("acc", "detect_direction", 0)
/* account triggers (flags) */
modparam("acc", "log_flag", FLT_ACC)
modparam("acc", "log_missed_flag", FLT_ACCMISSED)
modparam("acc", "log_extra",
	"src_user=$fU;src_domain=$fd;src_ip=$si;"
	"dst_ouser=$tU;dst_user=$rU;dst_domain=$rd")
modparam("acc", "failed_transaction_flag", FLT_ACCFAILED)
/* enhanced DB accounting */
#!ifdef WITH_ACCDB
modparam("acc", "db_flag", FLT_ACC)
modparam("acc", "db_missed_flag", FLT_ACCMISSED)
modparam("acc", "db_url", DBURL)
modparam("acc", "db_extra",
	"src_user=$fU;src_domain=$fd;src_ip=$si;"
	"dst_ouser=$tU;dst_user=$rU;dst_domain=$rd")
#!endif

# ----- usrloc params -----
/* enable DB persistency for location entries */
#!ifdef WITH_USRLOCDB
modparam("usrloc", "db_url", DBURL)
modparam("usrloc", "db_mode", 2)
modparam("usrloc", "use_domain", MULTIDOMAIN)
#!endif

# ----- auth_db params -----
#!ifdef WITH_AUTH
modparam("auth_db", "db_url", DBURL)
modparam("auth_db", "calculate_ha1", yes)
modparam("auth_db", "password_column", "password")
modparam("auth_db", "load_credentials", "")
modparam("auth_db", "use_domain", MULTIDOMAIN)

# ----- permissions params -----
#!ifdef WITH_IPAUTH
modparam("permissions", "db_url", DBURL)
modparam("permissions", "db_mode", 1)
#!endif

#!endif

# ----- alias_db params -----
#!ifdef WITH_ALIASDB
modparam("alias_db", "db_url", DBURL)
modparam("alias_db", "use_domain", MULTIDOMAIN)
#!endif

# ----- speeddial params -----
#!ifdef WITH_SPEEDDIAL
modparam("speeddial", "db_url", DBURL)
modparam("speeddial", "use_domain", MULTIDOMAIN)
#!endif

# ----- domain params -----
#!ifdef WITH_MULTIDOMAIN
modparam("domain", "db_url", DBURL)
/* register callback to match myself condition with domains list */
modparam("domain", "register_myself", 1)
#!endif

#!ifdef WITH_PRESENCE
# ----- presence params -----
modparam("presence", "db_url", DBURL)

# ----- presence_xml params -----
modparam("presence_xml", "db_url", DBURL)
modparam("presence_xml", "force_active", 1)
#!endif

#!ifdef WITH_NAT
# ----- rtpproxy params -----
modparam("rtpproxy", "rtpproxy_sock", "udp:127.0.0.1:7722")

# ----- nathelper params -----
modparam("nathelper", "natping_interval", 30)
modparam("nathelper", "ping_nated_only", 1)
modparam("nathelper", "sipping_bflag", FLB_NATSIPPING)
modparam("nathelper", "sipping_from", "sip:pinger@kamailio.org")

# params needed for NAT traversal in other modules
modparam("nathelper|registrar", "received_avp", "$avp(RECEIVED)")
modparam("usrloc", "nat_bflag", FLB_NATB)
#!endif

#!ifdef WITH_TLS
# ----- tls params -----
modparam("tls", "config", "/etc/kamailio/tls.cfg")
#!endif

#!ifdef WITH_ANTIFLOOD
# ----- pike params -----
modparam("pike", "sampling_time_unit", 2)
modparam("pike", "reqs_density_per_unit", 16)
modparam("pike", "remove_latency", 4)

# ----- htable params -----
/* ip ban htable with autoexpire after 5 minutes */
modparam("htable", "htable", "ipban=>size=8;autoexpire=300;")
#!endif

#!ifdef WITH_XMLRPC
# ----- xmlrpc params -----
modparam("xmlrpc", "route", "XMLRPC");
modparam("xmlrpc", "url_match", "^/RPC")
#!endif

#!ifdef WITH_DEBUG
# ----- debugger params -----
modparam("debugger", "cfgtrace", 1)
modparam("debugger", "log_level_name", "exec")
#!endif

####### Routing Logic ########

# main routing logic

route{

    /* ********* ROUTINE CHECKS  ********************************** */
    # filter too old messages
    if (!mf_process_maxfwd_header("10"))
    {
        log("LOG: Too many hops\n");
        sl_send_reply("483","Too Many Hops");
        break;
    };

    force_rport();
    # if (client_nat_test("11"))
    # {
    #       fix_contact();
    #       setflag(5);
    # }


    # grant Route routing if route headers present
    if (has_totag())
    {
		# route("force_to");
        if (loose_route())
        {
            if ( is_method("BYE") )
            {
                setflag(1);
                setflag(3);
            }
			if ( is_method("BYE|ACK|UPDATE") )
			{
				route("force_to");
			}
			else
			{
				t_relay();
			}
            exit;
        }
        else
        {
            if ( is_method("ACK") )
            {
                if ( t_check_trans() )
                {
                    # t_relay();
					route("force_to");
                    exit;
                }
                else
                {
                    exit;
                }
            }
            sl_send_reply("404","Not here");
        }
        exit;
    }

    # CANCEL processing
    if (is_method("CANCEL"))
    {
		# route("force_to");
        if (t_check_trans())
		{
			# t_relay();
			route("force_to");
		}
        exit;
    }
	
	t_check_trans();

    # Process initial INVITE|REGISTER|OPTIONS
    if (is_method("INVITE|OPTIONS"))
    {
        if (is_method("INVITE"))
        {
            # record_route();
            if ($si=="172.10.2.20")
			{
				record_route_preset("10.0.1.129:5060","10.110.12.5:5060");
				#record_route_preset("10.0.1.129:5060");
			}
			else if (src_ip==myself)
			{
                #record_route();
				record_route_preset("10.110.12.5:5060","10.0.1.129:5060");
				set_advertised_address("10.110.12.5");
				subst_hf("Contact", "/@.*/@10.110.12.5:5060;trasport=udp>/", "a");
				subst_hf("Allow", "/^.*/INVITE, ACK, BYE, CANCEL, OPTIONS, MESSAGE, INFO, REGISTER, REFER, NOTIFY/", "a");
			}

             if ((has_body("application/sdp") || search("^Content-Type:[ ]*application/sdp")) && $si=="172.10.2.20")
             {
                fix_nated_sdp("10","172.10.2.20");
              }
        }
        t_on_reply("1");
    }
    else
    {
        sl_send_reply("404","Not here");
        exit;
    };
	
	if(is_method("INVITE"))
	{
		setflag(1); # do accouting
	}

    # if ($si=="192.168.5.115")
    # if ($si=="172.10.2.20")
    # {
    #     #$du = "sip:eno16780032:5080;transport=udp";
    #     $du = "sip:10.0.1.129:5080;transport=udp";
    # }
    # #else if ($si=="192.168.5.140")
    # else if (src_ip==myself)
    # {
    #     $du = "sip:172.10.2.20:5060;transport=udp";
    # }
	route("force_to");

    # forward the request now
    if (!t_relay())
    {
        sl_reply_error();
        break;
    };
}

route[force_to]
{
	if ($si=="172.10.2.20")
    {
		if(is_method("BYE|ACK"))
		{
			if(is_present_hf("Route"))
			{
				remove_hf_value("Route[*]");
			}
		}
        $du = "sip:10.0.1.129:5080;transport=udp";
        if (!is_method("INVITE"))
		{
			forward();
		}
    }
    else if (src_ip==myself)
    {
		if(is_method("BYE|ACK"))
		{
			#subst_hf("From", "/@10.110.12.5>/@10.226.30.1>/", "a");
			#subst_hf("To", "/@10.226.30.1>/@10.110.12.5>/", "a");
			if(is_present_hf("Route"))
			{
				remove_hf_value("Route[1]");
				remove_hf_value("Route[2]");
			}
		}
        $du = "sip:172.10.2.20:5060;transport=udp";
        if (!is_method("INVITE"))
		{
			forward();
		}
    }
}

onreply_route[1]
{
    # if (client_nat_test("11"))
    # {
    #       fix_contact();
    #       setflag(5);
    # }
    if(status =~ "(180)|(183)|2[0-9][0-9]")
	{
		if ($si=="172.10.2.20")
		{
			fix_nated_sdp("10","172.10.2.20");
		}
		else if (src_ip==myself)
		{
            fix_nated_sdp("10","10.110.12.5");
			subst_hf("Contact", "/@.*/@10.110.12.5:5060;trasport=udp>/", "a");
			subst_hf("Allow", "/^.*/INVITE, ACK, BYE, CANCEL, OPTIONS, MESSAGE, INFO, REGISTER, REFER, NOTIFY/", "a");
		}
	}
}
```

版本二：

```conf
memdbg=5
memlog=5

debug=4
log_stderror=no
log_facility=LOG_LOCAL0

fork=yes
children=4
disable_tcp=yes
listen=udp:192.168.5.104:5078
port=5078
mpath="/usr/local/lib64/kamailio/modules/"

# ------------------ module loading ----------------------------------
loadmodule "mi_fifo.so"
loadmodule "tm.so"
loadmodule "sl.so"
loadmodule "rr.so"
loadmodule "pv.so"
loadmodule "maxfwd.so"
loadmodule "textops.so"
loadmodule "siputils.so"
loadmodule "xlog.so"
loadmodule "nathelper.so"
loadmodule "nat_traversal.so"



# ----------------- setting module-specific parameters ---------------
modparam("mi_fifo", "fifo_name", "/tmp/kamailio_fifo")

# -------------------------  request routing logic -------------------

# main routing logic

route{

    /* ********* ROUTINE CHECKS  ********************************** */
    # filter too old messages
    if (!mf_process_maxfwd_header("10"))
    {
        log("LOG: Too many hops\n");
        sl_send_reply("483","Too Many Hops");
        break;
    };

    # grant Route routing if route headers present
    if (has_totag())
    {
        if (loose_route())
        {
            if ( is_method("BYE") )
	    {
		setflag(1);
		setflag(3);
	    }
            t_relay();
            break;
        }
        else
        {
            if ( is_method("ACK") )
            {
                if ( t_check_trans() )
                {
                    t_relay();
                    break;
                }
                else
                {
                    exit;
                }
            }
            sl_send_reply("404","Not here");
        }
        exit;
    }

    # CANCEL processing
    if (is_method("CANCEL"))
    {
        if (t_check_trans())
            t_relay();
        exit;
    }

    # Process initial INVITE|REGISTER|OPTIONS
    if (is_method("INVITE|REGISTER|OPTIONS"))
    {
	fix_contact();
        record_route();
        if ( $si=="121.78.72.68" )
        {
            fix_nated_sdp(2,"192.168.50.16");
            fix_nated_sdp(8,"192.168.50.16");

        }
        t_on_reply("modify_sdp");
    }
    else
    {
        sl_send_reply("404","Not here");
        exit;
    };

    if ($si=="121.78.72.68" || $si=="192.168.3.2")
	rewritehostport("192.168.5.104:5078");
    else if ($si=="192.168.5.104")
	rewritehostport("121.78.72.68");

    # forward the request now
    if (!t_relay())
    {
        sl_reply_error();
        break;
    };
}

onreply_route[modify_sdp]
{
    if((status=="183" || status=="200") && $si=="121.78.72.68")
    {
        fix_nated_sdp("10","192.168.5.127");
    }
}
```

配置开机启动

```
mkdir -p /var/run/kamailio
groupadd kamailio
adduser --system -g kamailio --shell /bin/false --home /var/run/kamailio kamailio
chown kamailio:kamailio /var/run/kamailio
```

```
#!/bin/bash
#
# Startup script for Kamailio
#
# chkconfig: 345 85 15
# description: Kamailio (OpenSER) - the Open Source SIP Server
#
# processname: kamailio
# pidfile: /var/run/kamailio.pid
# config: /etc/kamailio/kamailio.cfg
#
### BEGIN INIT INFO
# Provides: kamailio
# Required-Start: $local_fs $network
# Short-Description: Kamailio (OpenSER) - the Open Source SIP Server
# Description: Kamailio (former OpenSER) is an Open Source SIP Server released
# 	under GPL, able to handle thousands of call setups per second.
### END INIT INFO

# Source function library.
. /etc/rc.d/init.d/functions

KAM=/data/kamailio/sbin/kamailio
KAMCFG=/data/kamailio/etc/kamailio/kamailio.cfg
PROG=kamailio
HOMEDIR=/var/run/$PROG
PID_FILE=/var/run/kamailio.pid
LOCK_FILE=/var/lock/subsys/kamailio
RETVAL=0
DEFAULTS=/etc/sysconfig/kamailio
RUN_KAMAILIO=yes


# Do not start kamailio if fork=no is set in the config file
# otherwise the boot process will just stop
check_fork ()
{
    if grep -q "^[[:space:]]*fork[[:space:]]*=[[:space:]]*no.*" $KAMCFG; then
        echo "Not starting $PROG: fork=no specified in config file; run /etc/init.d/kamailio debug instead"
        exit 1
    fi
}

check_kamailio_config ()
{
        # Check if kamailio configuration is valid before starting the server
        out=$($KAM -c $OPTIONS 2>&1 > /dev/null)
        retcode=$?
        if [ "$retcode" != '0' ]; then
            echo "Not starting $PROG: invalid configuration file!"
            echo -e "\n$out\n"
            exit 1
        fi
}


start() {
	check_kamailio_config
        if [ "$1" != "debug" ]; then
            check_fork
        fi
	echo -n $"Starting $PROG: "
	daemon $KAM $OPTIONS >/dev/null 2>/dev/null
	RETVAL=$?
	[ $RETVAL = 0 ] && touch $LOCK_FILE && success
	echo
	return $RETVAL
}

stop() {
	echo -n $"Stopping $PROG: "
	killproc -p $PID_FILE
	RETVAL=$?
	echo
	[ $RETVAL = 0 ] && rm -f $LOCK_FILE $PID_FILE
}

# Load startup options if available
if [ -f $DEFAULTS ]; then
   . $DEFAULTS || true
fi

if [ "$RUN_KAMAILIO" != "yes" ]; then
    echo "Kamailio not yet configured. Edit $DEFAULTS first."
    exit 0
fi


SHM_MEMORY=$((`echo $SHM_MEMORY | sed -e 's/[^0-9]//g'`))
PKG_MEMORY=$((`echo $PKG_MEMORY | sed -e 's/[^0-9]//g'`))
[ -z "$USER" ]  && USER=kamailio
[ -z "$GROUP" ] && GROUP=kamailio
[ $SHM_MEMORY -le 0 ] && SHM_MEMORY=32
[ $PKG_MEMORY -le 0 ] && PKG_MEMORY=4

if test "$DUMP_CORE" = "yes" ; then
    # set proper ulimit
    ulimit -c unlimited

    # directory for the core dump files
    COREDIR=/tmp
    echo "$COREDIR/core.%e.sig%s.%p" > /proc/sys/kernel/core_pattern
fi

# /var/run can be a tmpfs
if [ ! -d $HOMEDIR ]; then
    mkdir -p $HOMEDIR
	chown ${USER}:${GROUP} $HOMEDIR
fi

OPTIONS="-f $KAMCFG -P $PID_FILE -m $SHM_MEMORY -M $PKG_MEMORY -u $USER -g $GROUP $EXTRA_OPTIONS"


# See how we were called.
case "$1" in
	start|debug)
		start
		;;
	stop)
		stop
		;;
	status)
		status $KAM
		RETVAL=$?
		;;
	restart)
		stop
		start
		;;
	condrestart)
		if [ -f $PID_FILE ] ; then
			stop
			start
		fi
		;;
	*)
		echo $"Usage: $PROG {start|stop|restart|condrestart|status|debug|help}"
		exit 1
esac

exit $RETVAL
```

方式二：docker 部署

在第一次运行之前，需要准备kamailio默认配置文件。 如果您已经有kamailio配置文件，则可以跳过此操作。 要准备默认配置文件，需要执行

```
docker create --name kamailio kamailio/kamailio-ci
docker cp kamailio:/etc/kamailio /etc
docker rm kamailio
```

然后备份旧的 /etc/kamailio/kamailio.cfg，替换配置文件为内容为：

```conf
#!KAMAILIO
#
# Kamailio (OpenSER) SIP Server v5.3 - default configuration script
#     - web: https://www.kamailio.org
#     - git: https://github.com/kamailio/kamailio
#
# Direct your questions about this file to: <sr-users@lists.kamailio.org>
#
# Refer to the Core CookBook at https://www.kamailio.org/wiki/
# for an explanation of possible statements, functions and parameters.
#
# Note: the comments can be:
#     - lines starting with #, but not the pre-processor directives,
#       which start with #!, like #!define, #!ifdef, #!endif, #!else, #!trydef,
#       #!subst, #!substdef, ...
#     - lines starting with //
#     - blocks enclosed in between /* */
#
# Several features can be enabled using '#!define WITH_FEATURE' directives:
#
# *** To run in debug mode:
#     - define WITH_DEBUG
#
# *** To enable mysql:
#     - define WITH_MYSQL
#
# *** To enable authentication execute:
#     - enable mysql
#     - define WITH_AUTH
#     - add users using 'kamctl'
#
# *** To enable IP authentication execute:
#     - enable mysql
#     - enable authentication
#     - define WITH_IPAUTH
#     - add IP addresses with group id '1' to 'address' table
#
# *** To enable persistent user location execute:
#     - enable mysql
#     - define WITH_USRLOCDB
#
# *** To enable presence server execute:
#     - enable mysql
#     - define WITH_PRESENCE
#
# *** To enable nat traversal execute:
#     - define WITH_NAT
#     - install RTPProxy: http://www.rtpproxy.org
#     - start RTPProxy:
#        rtpproxy -l _your_public_ip_ -s udp:localhost:7722
#     - option for NAT SIP OPTIONS keepalives: WITH_NATSIPPING
#
# *** To enable PSTN gateway routing execute:
#     - define WITH_PSTN
#     - set the value of pstn.gw_ip
#     - check route[PSTN] for regexp routing condition
#
# *** To enable database aliases lookup execute:
#     - enable mysql
#     - define WITH_ALIASDB
#
# *** To enable speed dial lookup execute:
#     - enable mysql
#     - define WITH_SPEEDDIAL
#
# *** To enable multi-domain support execute:
#     - enable mysql
#     - define WITH_MULTIDOMAIN
#
# *** To enable TLS support execute:
#     - adjust CFGDIR/tls.cfg as needed
#     - define WITH_TLS
#
# *** To enable XMLRPC support execute:
#     - define WITH_XMLRPC
#     - adjust route[XMLRPC] for access policy
#
# *** To enable anti-flood detection execute:
#     - adjust pike and htable=>ipban settings as needed (default is
#       block if more than 16 requests in 2 seconds and ban for 300 seconds)
#     - define WITH_ANTIFLOOD
#
# *** To block 3XX redirect replies execute:
#     - define WITH_BLOCK3XX
#
# *** To block 401 and 407 authentication replies execute:
#     - define WITH_BLOCK401407
#
# *** To enable VoiceMail routing execute:
#     - define WITH_VOICEMAIL
#     - set the value of voicemail.srv_ip
#     - adjust the value of voicemail.srv_port
#
# *** To enhance accounting execute:
#     - enable mysql
#     - define WITH_ACCDB
#     - add following columns to database
#!ifdef ACCDB_COMMENT
  ALTER TABLE acc ADD COLUMN src_user VARCHAR(64) NOT NULL DEFAULT '';
  ALTER TABLE acc ADD COLUMN src_domain VARCHAR(128) NOT NULL DEFAULT '';
  ALTER TABLE acc ADD COLUMN src_ip varchar(64) NOT NULL default '';
  ALTER TABLE acc ADD COLUMN dst_ouser VARCHAR(64) NOT NULL DEFAULT '';
  ALTER TABLE acc ADD COLUMN dst_user VARCHAR(64) NOT NULL DEFAULT '';
  ALTER TABLE acc ADD COLUMN dst_domain VARCHAR(128) NOT NULL DEFAULT '';
  ALTER TABLE missed_calls ADD COLUMN src_user VARCHAR(64) NOT NULL DEFAULT '';
  ALTER TABLE missed_calls ADD COLUMN src_domain VARCHAR(128) NOT NULL DEFAULT '';
  ALTER TABLE missed_calls ADD COLUMN src_ip varchar(64) NOT NULL default '';
  ALTER TABLE missed_calls ADD COLUMN dst_ouser VARCHAR(64) NOT NULL DEFAULT '';
  ALTER TABLE missed_calls ADD COLUMN dst_user VARCHAR(64) NOT NULL DEFAULT '';
  ALTER TABLE missed_calls ADD COLUMN dst_domain VARCHAR(128) NOT NULL DEFAULT '';
#!endif

####### Include Local Config If Exists #########
import_file "kamailio-local.cfg"

####### Defined Values #########

# *** Value defines - IDs used later in config
#!ifdef WITH_MYSQL
# - database URL - used to connect to database server by modules such
#       as: auth_db, acc, usrloc, a.s.o.
#!ifndef DBURL
#!define DBURL "mysql://kamailio:kamailiorw@localhost/kamailio"
#!endif
#!endif
#!ifdef WITH_MULTIDOMAIN
# - the value for 'use_domain' parameters
#!define MULTIDOMAIN 1
#!else
#!define MULTIDOMAIN 0
#!endif

# - flags
#   FLT_ - per transaction (message) flags
#	FLB_ - per branch flags
#!define FLT_ACC 1
#!define FLT_ACCMISSED 2
#!define FLT_ACCFAILED 3
#!define FLT_NATS 5

#!define FLB_NATB 6
#!define FLB_NATSIPPING 7

####### Global Parameters #########

### LOG Levels: 3=DBG, 2=INFO, 1=NOTICE, 0=WARN, -1=ERR
#!ifdef WITH_DEBUG
debug=4
log_stderror=yes
#!else
debug=2
log_stderror=no
#!endif

memdbg=5
memlog=5

log_facility=LOG_LOCAL0
log_prefix="{$mt $hdr(CSeq) $ci} "

/* number of SIP routing processes for each UDP socket
 * - value inherited by tcp_children and sctp_children when not set explicitely */
children=4

/* uncomment the next line to disable TCP (default on) */
disable_tcp=yes

/* number of SIP routing processes for all TCP/TLS sockets */
# tcp_children=8

/* uncomment the next line to disable the auto discovery of local aliases
 * based on reverse DNS on IPs (default on) */
# auto_aliases=no

/* add local domain aliases */
# alias="sip.mydomain.com"

/* uncomment and configure the following line if you want Kamailio to
 * bind on a specific interface/port/proto (default bind on all available) */
# listen=udp:10.0.0.10:5060
listen=udp:eno16780032:5078

/* life time of TCP connection when there is no traffic
 * - a bit higher than registration expires to cope with UA behind NAT */
tcp_connection_lifetime=3605

/* upper limit for TCP connections (it includes the TLS connections) */
tcp_max_connections=2048

#!ifdef WITH_TLS
enable_tls=yes

/* upper limit for TLS connections */
tls_max_connections=2048
#!endif

####### Custom Parameters #########

/* These parameters can be modified runtime via RPC interface
 * - see the documentation of 'cfg_rpc' module.
 *
 * Format: group.id = value 'desc' description
 * Access: $sel(cfg_get.group.id) or @cfg_get.group.id */

#!ifdef WITH_PSTN
/* PSTN GW Routing
 *
 * - pstn.gw_ip: valid IP or hostname as string value, example:
 * pstn.gw_ip = "10.0.0.101" desc "My PSTN GW Address"
 *
 * - by default is empty to avoid misrouting */
pstn.gw_ip = "" desc "PSTN GW Address"
pstn.gw_port = "" desc "PSTN GW Port"
#!endif

#!ifdef WITH_VOICEMAIL
/* VoiceMail Routing on offline, busy or no answer
 *
 * - by default Voicemail server IP is empty to avoid misrouting */
voicemail.srv_ip = "" desc "VoiceMail IP Address"
voicemail.srv_port = "5060" desc "VoiceMail Port"
#!endif

####### Modules Section ########

/* set paths to location of modules */
# mpath="/usr/lib/kamailio/modules/"

#!ifdef WITH_MYSQL
loadmodule "db_mysql.so"
#!endif

loadmodule "jsonrpcs.so"
loadmodule "kex.so"
loadmodule "corex.so"
loadmodule "tm.so"
loadmodule "tmx.so"
loadmodule "sl.so"
loadmodule "rr.so"
loadmodule "pv.so"
loadmodule "maxfwd.so"
loadmodule "usrloc.so"
loadmodule "registrar.so"
loadmodule "textops.so"
loadmodule "siputils.so"
loadmodule "xlog.so"
loadmodule "sanity.so"
loadmodule "ctl.so"
loadmodule "cfg_rpc.so"
loadmodule "acc.so"
loadmodule "counters.so"
loadmodule "nathelper.so"
loadmodule "nat_traversal.so"
loadmodule "dialog.so"

#!ifdef WITH_AUTH
loadmodule "auth.so"
loadmodule "auth_db.so"
#!ifdef WITH_IPAUTH
loadmodule "permissions.so"
#!endif
#!endif

#!ifdef WITH_ALIASDB
loadmodule "alias_db.so"
#!endif

#!ifdef WITH_SPEEDDIAL
loadmodule "speeddial.so"
#!endif

#!ifdef WITH_MULTIDOMAIN
loadmodule "domain.so"
#!endif

#!ifdef WITH_PRESENCE
loadmodule "presence.so"
loadmodule "presence_xml.so"
#!endif

#!ifdef WITH_NAT
loadmodule "nathelper.so"
loadmodule "rtpproxy.so"
#!endif

#!ifdef WITH_TLS
loadmodule "tls.so"
#!endif

#!ifdef WITH_ANTIFLOOD
loadmodule "htable.so"
loadmodule "pike.so"
#!endif

#!ifdef WITH_XMLRPC
loadmodule "xmlrpc.so"
#!endif

#!ifdef WITH_DEBUG
loadmodule "debugger.so"
#!endif

# ----------------- setting module-specific parameters ---------------


# ----- jsonrpcs params -----
modparam("jsonrpcs", "pretty_format", 1)
/* set the path to RPC fifo control file */
# modparam("jsonrpcs", "fifo_name", "/var/run/kamailio/kamailio_rpc.fifo")
/* set the path to RPC unix socket control file */
# modparam("jsonrpcs", "dgram_socket", "/var/run/kamailio/kamailio_rpc.sock")

# ----- ctl params -----
/* set the path to RPC unix socket control file */
# modparam("ctl", "binrpc", "unix:/var/run/kamailio/kamailio_ctl")

# ----- tm params -----
# auto-discard branches from previous serial forking leg
modparam("tm", "failure_reply_mode", 3)
# default retransmission timeout: 30sec
modparam("tm", "fr_timer", 30000)
# default invite retransmission timeout after 1xx: 120sec
modparam("tm", "fr_inv_timer", 120000)

# ----- rr params -----
# set next param to 1 to add value to ;lr param (helps with some UAs)
modparam("rr", "enable_full_lr", 0)
# do not append from tag to the RR (no need for this script)
modparam("rr", "append_fromtag", 0)

# ----- registrar params -----
modparam("registrar", "method_filtering", 1)
/* uncomment the next line to disable parallel forking via location */
# modparam("registrar", "append_branches", 0)
/* uncomment the next line not to allow more than 10 contacts per AOR */
# modparam("registrar", "max_contacts", 10)
/* max value for expires of registrations */
modparam("registrar", "max_expires", 3600)
/* set it to 1 to enable GRUU */
modparam("registrar", "gruu_enabled", 0)

# ----- acc params -----
/* what special events should be accounted ? */
modparam("acc", "early_media", 0)
modparam("acc", "report_ack", 0)
modparam("acc", "report_cancels", 0)
/* by default ww do not adjust the direct of the sequential requests.
 * if you enable this parameter, be sure the enable "append_fromtag"
 * in "rr" module */
modparam("acc", "detect_direction", 0)
/* account triggers (flags) */
modparam("acc", "log_flag", FLT_ACC)
modparam("acc", "log_missed_flag", FLT_ACCMISSED)
modparam("acc", "log_extra",
	"src_user=$fU;src_domain=$fd;src_ip=$si;"
	"dst_ouser=$tU;dst_user=$rU;dst_domain=$rd")
modparam("acc", "failed_transaction_flag", FLT_ACCFAILED)
/* enhanced DB accounting */
#!ifdef WITH_ACCDB
modparam("acc", "db_flag", FLT_ACC)
modparam("acc", "db_missed_flag", FLT_ACCMISSED)
modparam("acc", "db_url", DBURL)
modparam("acc", "db_extra",
	"src_user=$fU;src_domain=$fd;src_ip=$si;"
	"dst_ouser=$tU;dst_user=$rU;dst_domain=$rd")
#!endif

# ----- usrloc params -----
/* enable DB persistency for location entries */
#!ifdef WITH_USRLOCDB
modparam("usrloc", "db_url", DBURL)
modparam("usrloc", "db_mode", 2)
modparam("usrloc", "use_domain", MULTIDOMAIN)
#!endif

# ----- auth_db params -----
#!ifdef WITH_AUTH
modparam("auth_db", "db_url", DBURL)
modparam("auth_db", "calculate_ha1", yes)
modparam("auth_db", "password_column", "password")
modparam("auth_db", "load_credentials", "")
modparam("auth_db", "use_domain", MULTIDOMAIN)

# ----- permissions params -----
#!ifdef WITH_IPAUTH
modparam("permissions", "db_url", DBURL)
modparam("permissions", "db_mode", 1)
#!endif

#!endif

# ----- alias_db params -----
#!ifdef WITH_ALIASDB
modparam("alias_db", "db_url", DBURL)
modparam("alias_db", "use_domain", MULTIDOMAIN)
#!endif

# ----- speeddial params -----
#!ifdef WITH_SPEEDDIAL
modparam("speeddial", "db_url", DBURL)
modparam("speeddial", "use_domain", MULTIDOMAIN)
#!endif

# ----- domain params -----
#!ifdef WITH_MULTIDOMAIN
modparam("domain", "db_url", DBURL)
/* register callback to match myself condition with domains list */
modparam("domain", "register_myself", 1)
#!endif

#!ifdef WITH_PRESENCE
# ----- presence params -----
modparam("presence", "db_url", DBURL)

# ----- presence_xml params -----
modparam("presence_xml", "db_url", DBURL)
modparam("presence_xml", "force_active", 1)
#!endif

#!ifdef WITH_NAT
# ----- rtpproxy params -----
modparam("rtpproxy", "rtpproxy_sock", "udp:127.0.0.1:7722")

# ----- nathelper params -----
modparam("nathelper", "natping_interval", 30)
modparam("nathelper", "ping_nated_only", 1)
modparam("nathelper", "sipping_bflag", FLB_NATSIPPING)
modparam("nathelper", "sipping_from", "sip:pinger@kamailio.org")

# params needed for NAT traversal in other modules
modparam("nathelper|registrar", "received_avp", "$avp(RECEIVED)")
modparam("usrloc", "nat_bflag", FLB_NATB)
#!endif

#!ifdef WITH_TLS
# ----- tls params -----
modparam("tls", "config", "/etc/kamailio/tls.cfg")
#!endif

#!ifdef WITH_ANTIFLOOD
# ----- pike params -----
modparam("pike", "sampling_time_unit", 2)
modparam("pike", "reqs_density_per_unit", 16)
modparam("pike", "remove_latency", 4)

# ----- htable params -----
/* ip ban htable with autoexpire after 5 minutes */
modparam("htable", "htable", "ipban=>size=8;autoexpire=300;")
#!endif

#!ifdef WITH_XMLRPC
# ----- xmlrpc params -----
modparam("xmlrpc", "route", "XMLRPC");
modparam("xmlrpc", "url_match", "^/RPC")
#!endif

#!ifdef WITH_DEBUG
# ----- debugger params -----
modparam("debugger", "cfgtrace", 1)
modparam("debugger", "log_level_name", "exec")
#!endif

####### Routing Logic ########

# main routing logic

route{

    /* ********* ROUTINE CHECKS  ********************************** */
    # filter too old messages
    if (!mf_process_maxfwd_header("10"))
    {
        log("LOG: Too many hops\n");
        sl_send_reply("483","Too Many Hops");
        break;
    };

    force_rport();
    if (client_nat_test("11"))
    {
          fix_contact();
          setflag(5);
    }
    
    
    # grant Route routing if route headers present
    if (has_totag())
    {
        if (loose_route())
        {
            if ( is_method("BYE") )
	    {
		setflag(1);
		setflag(3);
	    }
            t_relay();
            break;
        }
        else
        {
            if ( is_method("ACK") )
            {
                if ( t_check_trans() )
                {
                    t_relay();
                    break;
                }
                else
                {
                    exit;
                }
            }
            sl_send_reply("404","Not here");
        }
        exit;
    }

    # CANCEL processing
    if (is_method("CANCEL"))
    {
        if (t_check_trans())
            t_relay();
        exit;
    }

    # Process initial INVITE|REGISTER|OPTIONS
    if (is_method("INVITE|REGISTER|OPTIONS"))
    {
        if (is_method("INVITE"))
        {
            record_route();
             if ((has_body("application/sdp") || search("^Content-Type:[ ]*application/sdp")) && $si=="192.168.5.127")
             {
                fix_nated_sdp("10","192.168.5.127");
              }
        }
        t_on_reply("1");
    }
    else
    {
        sl_send_reply("404","Not here");
        exit;
    };

    # if ($si=="192.168.5.115")
    if ($si=="192.168.5.127")
    {
	#$du = "sip:eno16780032:5080;transport=udp";
	$du = "sip:eno16780032:5080;transport=udp";
    }
    #else if ($si=="192.168.5.140")
    else if (src_ip==myself)
    {
	$du = "sip:192.168.5.127:5187;transport=udp";
    }

    # forward the request now
    if (!t_relay())
    {
        sl_reply_error();
        break;
    };
}

onreply_route[1]
{
    if (client_nat_test("11"))
    {
          fix_contact();
          setflag(5);
    }
    #if((status=="180" || status=="183" || status=="200") && $si=="192.168.5.127")
    if(status =~ "(180)|(183)|2[0-9][0-9]" && $si=="192.168.5.127")
    {
        fix_nated_sdp("10","192.168.5.127");    

    }
}
```

最后利用 docker-compose 管理 kamailio 容器，需要先安装 docker-compose，配置文件如下。

```conf
version: "2"
services:
        kamailio:
                image: kamailio/kamailio-ci:5.3
                restart: always
                container_name: kamailio
                volumes:
                        - /etc/kamailio:/etc/kamailio
                network_mode: "host"
                logging:
                        driver: "json-file"
                        options:
                                max-size: "1g"
                                max-file: "2"
                command: -m 64 -M 8
```

## Reference

- [OpenSIPS实战九：跨NAT通信](https://blog.csdn.net/I_O_fly/article/details/102811961)
- [Opensips & RTPEngine & FreeSwitch 实现FS高可用](http://www.younian.me/archives/Opensips%20&%20RTPEngine%20&%20FreeSwitch%20%E5%AE%9E%E7%8E%B0FS%E9%AB%98%E5%8F%AF%E7%94%A8)
- [how to use fix_nated_sdp("2") with multiple media types](https://lists.kamailio.org/pipermail/sr-users/2012-October/075061.html)
- [Centos 7 下安装opensips详细流程](https://www.jianshu.com/p/4f17429cfdcd)
- [FreeSWITCH 使用OpenSIPS进行负载均衡](https://blog.csdn.net/gredn/article/details/88696624)
- [FreeSWITCH折腾笔记8——使用OpenSIPS进行负载均衡](https://blog.51cto.com/908405/2235934)
- [Freeswitch 高级主题之用kamailio负载均衡](https://blog.csdn.net/voipmaker/article/details/51056322)
- [安装配置opensips过程记录](https://www.cnblogs.com/bjzhanghao/archive/2013/02/13/2910903.html#top)
- [opensips-mediaproxy](https://github.com/Izib/opensips-mediaproxy/blob/master/Install/package/opensip_rtpproxy.rm)
- [opensips 安装 rtpproxy 教程](https://blog.csdn.net/hzh_csdn/article/details/56011403)
- [ubuntu下opensips安装配置](https://blog.csdn.net/u011026329/article/details/50821679)
- [Freeswitch之RTP地址自动校正](https://blog.csdn.net/m0_46498786/article/details/104871227)
- [rtpproxy](https://github.com/sippy/rtpproxy)
- [opensips-freeSwitch负载均衡环境搭建配置.pptx](https://download.csdn.net/download/voipwangpeng/10713639)
- [OpenSIPS configuration for 2 or more FreeSWITCH installs](https://freeswitch.org/confluence/display/FREESWITCH/OpenSIPS+configuration+for+2+or+more+FreeSWITCH+installs)
- [Kamailio basic setup as proxy for FreeSWITCH](https://freeswitch.org/confluence/display/FREESWITCH/Kamailio+basic+setup+as+proxy+for+FreeSWITCH)
- [Kamailio RTP engine 模块文档](http://www.opensips.net.cn/30.html)
- [使用OpenSIPS构建电话通信系统-9SIP穿透NAT](https://www.docin.com/p-163183476.html)
- [Opensips文档之MediaProxy模块](https://www.docin.com/p-154354628.html)
- [Opensips 配置文件](https://www.docin.com/p-154393877.html)
- [开源SIP服务器OpenSIPS简介](https://blog.csdn.net/wavemcu/article/details/39270047)
- [OpenSIPS-1](https://github.com/baconbao/OpenSIPS-1)
- [freeswitch系列二 kamailio 5.0安装及实现kamailio集成freeswitch](https://blog.csdn.net/hry2015/article/details/77338341)
- [OpenSIPS 一键安装脚本-及 OpenSIPs+N个FreeSWITCH 实战技巧](https://blog.csdn.net/FSGUI/article/details/105223129)
- [cookbooks:core-cookbook:devel - sip-router.org](http://sip-router.org/wiki/cookbooks/core-cookbook/devel)
- [Asipto - SIP and VoIP Knowledge Base Site](http://kb.asipto.com/freeswitch:index)
- [Kamailio SIP Server v5.3.x (stable): Core Cookbook](https://www.kamailio.org/wiki/cookbooks/5.3.x/core#dokuwiki__top)
- [Kamailio (OpenSER) WiKi](https://www.kamailio.org/dokuwiki/doku.php)
- [Kamailio SIP Server Documentation Wiki](https://www.kamailio.org/wiki/#dokuwiki__top)
- [Kamailio Modules - v5.3.x (stable)](https://www.kamailio.org/docs/modules/stable/)
- [Install Kamailio Devel Version From Git Repository](https://kamailio.org/docs/tutorials/5.3.x/kamailio-install-guide-git/)
- [opensipsexamples](https://github.com/altanai/opensipsexamples)
- [CentOS7 Kamailio 安装指南](https://blog.csdn.net/yetyongjin/article/details/8106997)
- [kamailio 在容器环境中的部署](http://www.ctiforum.com/news/guonei/536195.html)
- [将freeswitch上的网关通过opensips注册](https://www.jianshu.com/p/563cd1164408)
- [opensips+freeswitch+mysql集群](https://www.jinchutou.com/p-88057032.html)
