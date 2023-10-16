---
title: freeswitch使用kamailio进行负载均衡
author: starifly
date: 2020-10-02T17:52:15+08:00
lastmod: 2020-10-02T17:52:15+08:00
categories: [freeswitch,kamailio]
tags: [freeswitch,kamailio]
draft: true
slug: freeswitch-uses-kamailio-for-load-balancing
---

## 安装 kamailio

如果要用数据库存储负载等信息，编译的时候需要带上 db 模块

```
make FLAVOUR=kamailio include_modules="db_mysql dialplan" cfg
make
make install
```

## 配置 mysql 数据库

进入数据库所在的服务器，创建如下账号：

```
mysql> grant all privileges on kamailio.* to kamailio@'%' identified by 'kamailiorw';
mysql> flush privileges;
```

修改数据库配置，/usr/local/etc/kamailio/kamctlrc

```conf
DBENGINE=MYSQL
DBHOST=192.168.1.10
DBPORT=3306
DBNAME=kamailio
DBRWUSER="kamailio"
DBRWPW="kamailiorw"
```

## 配置 kamailio

dispacher 模块有一些参数需要注意，默认下一跳地址的配置支持数据库和文本文件。

如果用 mysql 数据库存储，需要启用下面两行：

```conf
modparam("dispatcher", "db_url", DBURL)
modparam("dispatcher", "table_name", "dispatcher")
```

如果用文本文件，则需要启用如下配置：

```conf
modparam("dispatcher", "list_file", "/usr/local/etc/kamailio/dispatcher.list")
```

完整配置如下：

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
listen=udp:192.168.5.140:5078

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
loadmodule "db_mysql.so"
loadmodule "dispatcher.so"

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

# ----- dispatcher params -----
#modparam("dispatcher", "db_url", DBURL)
#modparam("dispatcher", "table_name", "dispatcher")
#modparam("dispatcher", "list_file", "/usr/local/etc/kamailio/dispatcher.list")
#modparam("dispatcher", "flags", 2)
#modparam("dispatcher", "dst_avp", "$avp(AVP_DST)")
#modparam("dispatcher", "grp_avp", "$avp(AVP_GRP)")
#modparam("dispatcher", "cnt_avp", "$avp(AVP_CNT)")
#modparam("dispatcher", "sock_avp", "$avp(AVP_SOCK)")
#modparam("path", "use_received", 1)

# 使用数据库时启用
modparam("dispatcher", "db_url", "mysql://kamailio:kamailiorw@192.168.1.10/kamailio")
modparam("dispatcher", "table_name", "dispatcher")
# 使用文本文件时启用
#modparam("dispatcher", "list_file", "/usr/local/etc/kamailio/dispatcher.list")
modparam("dispatcher", "flags", 2)
modparam("dispatcher", "xavp_dst", "_dsdst_")
modparam("dispatcher", "xavp_ctx", "_dsctx_")

modparam("dispatcher", "ds_ping_interval", 20)
modparam("dispatcher", "ds_probing_threshold", 3)
modparam("dispatcher", "ds_probing_mode", 2)
modparam("dispatcher", "ds_ping_reply_codes", "class=2;code=403;code=404;code=407;code=484;class=3")

####### Routing Logic ########

# main routing logic

request_route {

	# per request initial checks
	route(REQINIT);

	# CANCEL processing
	if (is_method("CANCEL")) {
		if (t_check_trans()) {
			route(RELAY);
		}
		exit;
	}

	# handle retransmissions
	if (!is_method("ACK")) {
		if(t_precheck_trans()) {
			t_check_trans();
			exit;
		}
		t_check_trans();
	}

	# handle requests within SIP dialogs
	route(WITHINDLG);

	### only initial requests (no To tag)

	# record routing for dialog forming requests (in case they are routed)
	# - remove preloaded route headers
	remove_hf("Route");
	if (is_method("INVITE|SUBSCRIBE")) {
		record_route();
	}

	# account only INVITEs
	if (is_method("INVITE")) {
		setflag(FLT_ACC); # do accounting
	}

	# handle presence related requests
	route(PRESENCE);

	# handle registrations
	route(REGISTRAR);

	if ($rU==$null) {
		# request with no Username in RURI
		sl_send_reply("484","Address Incomplete");
		exit;
	}

	# dispatch destinations
	route(DISPATCH);
}


route[RELAY] {
	if (!t_relay()) {
		sl_reply_error();
	}
	exit;
}

# Per SIP request initial checks
route[REQINIT] {
	if (!mf_process_maxfwd_header("10")) {
		sl_send_reply("483","Too Many Hops");
		exit;
	}

	if(!sanity_check("1511", "7")) {
		xlog("Malformed SIP message from $si:$sp\n");
		exit;
	}
}

# Handle requests within SIP dialogs
route[WITHINDLG] {
	if (!has_totag()) {
		return;
	}

	# sequential request withing a dialog should
	# take the path determined by record-routing
	if (loose_route()) {
		if (is_method("BYE")) {
			setflag(FLT_ACC); # do accounting ...
			setflag(FLT_ACCFAILED); # ... even if the transaction fails
		}
		route(RELAY);
	}

	if (is_method("SUBSCRIBE") && uri == myself) {
		# in-dialog subscribe requests
		route(PRESENCE);
	}

	if ( is_method("ACK") ) {
		if ( t_check_trans() ) {
			# non loose-route, but stateful ACK;
			# must be ACK after a 487 or e.g. 404 from upstream server
			t_relay();
			exit;
		} else {
			# ACK without matching transaction ... ignore and discard.
			exit;
		}
	}

	sl_send_reply("404","Not here");
	exit;
}

# Handle SIP registrations
route[REGISTRAR] {
	if(!is_method("REGISTER"))
		return;

	sl_send_reply("404", "No registrar");
	exit;
}

# Presence server route
route[PRESENCE] {
	if(!is_method("PUBLISH|SUBSCRIBE"))
		return;

	sl_send_reply("404", "Not here");
	exit;
}

# Dispatch requests
route[DISPATCH] {
	# round robin dispatching on gateways group '1'
	if(!ds_select_dst("0", "4")) {
		send_reply("404", "No destination");
		exit;
	}
	xlog("--- SCRIPT: going to <$ru> via <$du> (attrs: $xavp(_dsdst_=>attrs))\n");
    # 设置超时时间为2秒，便于快速跳转下一个地址
	t_set_fr(0,2000);
	t_on_failure("RTF_DISPATCH");
	route(RELAY);
	exit;
}

# Try next destionations in failure route
failure_route[RTF_DISPATCH] {
	if (t_is_canceled()) {
		exit;
	}
	# next DST - only for 500 or local timeout
	if (t_check_status("500")
			or (t_branch_timeout() and !t_branch_replied())) {
		ds_mark_dst("ip");
		if(ds_next_dst()) {
			xlog("--- SCRIPT: retrying to <$ru> via <$du> (attrs: $xavp(_dsdst_=>attrs))\n");
			t_set_fr(0,2000);
			t_on_failure("RTF_DISPATCH");
			route(RELAY);
			exit;
		}
	}
}
```

## 新建数据并调试

如果使用文本文件的方式，往 dispatcher.list 文件中添加数据,其中数字 0 是组 ID，对应函数 ds_select_dst 第一个参数。

```
0 sip:192.168.1.20:5080
```

如果使用数据库方式，则要往数据库中插入数据，其中 setid 字段是路由脚本中需要用到得组 ID。

```
mysql> insert into dispatcher(setid,destination,priority,flags,description) values(0,'sip:192.168.1.20:5080', 1, 0, 'fs1');
```

数据创建完后，用 `kamcmd dispatcher.reload` 重新加载，用 `kamcmd dispatcher.list` 列出数据。

到此，如果一切没问题就可以进行电话测试了。

## Reference

- [Freeswitch 高级主题之用kamailio负载均衡](https://blog.csdn.net/voipmaker/article/details/51056322)
- [freeswitch系列二 kamailio 5.0安装及实现kamailio集成freeswitch](https://blog.csdn.net/hry2015/article/details/77338341)
- [Load balancer using dispatcher module](https://github.com/altanai/kamailioexamples/tree/master/Loadbalancer_SIP_proxy)
- [DISPATCHER Module](https://www.kamailio.org/docs/modules/stable/modules/dispatcher.html#idm683)
- [Opensips & RTPEngine & FreeSwitch 实现FS高可用](http://www.younian.me/archives/Opensips%20&%20RTPEngine%20&%20FreeSwitch%20%E5%AE%9E%E7%8E%B0FS%E9%AB%98%E5%8F%AF%E7%94%A8)
- [FreeSWITCH折腾笔记8——使用OpenSIPS进行负载均衡](https://blog.51cto.com/908405/2235934)
- [opensips-freeSwitch负载均衡环境搭建配置.pptx](https://download.csdn.net/download/voipwangpeng/10713639)
- [opensips+freeswitch+mysql集群](https://www.jinchutou.com/p-88057032.html)
- [FreeSwitch+Opensips+NFS文件共享集群安装配置操作指导书](https://wenku.baidu.com/view/fe57344a76a20029bc642d6b.html#)
