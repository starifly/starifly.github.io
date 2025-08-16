# Rocky Linux 安装 Gitea


```bash
dnf install -y sqlite
wget -O /usr/local//bin/gitea https://dl.gitea.com/gitea/1.23.1/gitea-1.23.1-linux-amd64
chmod +x /usr/local/bin/gitea
groupadd --system gitea
adduser --system --shell /bin/bash --home-dir /var/lib/gitea --create-home  --gid gitea gitea
mkdir -p /var/lib/gitea/{custom,data,log}
chown -R gitea:gitea /var/lib/gitea
```

<!--more-->

```bash
[root@master ~]# cat  /etc/systemd/system/gitea.service
[Unit]
Description=Gitea (Git with a cup of tea)
After=network.target

[Service]
RestartSec=2s
Type=simple
User=gitea
Group=gitea
WorkingDirectory=/var/lib/gitea/
ExecStart=/usr/local/bin/gitea web --config /var/lib/gitea/custom/conf/app.ini
Restart=always
Environment=USER=gitea HOME=/var/lib/gitea GITEA_WORK_DIR=/var/lib/gitea

[Install]
WantedBy=multi-user.target
```

```bash
systemctl daemon-reload
systemctl start gitea
systemctl enable gitea
```


---

> 作者: [starifly](https://github.com/starifly)  
> URL: https://starifly.github.io/posts/rocky-linux-install-gitea/  

