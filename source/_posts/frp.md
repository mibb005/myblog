---
title: frp安装配置
date: 2019-05-21 16:55:49
tags:
 - frp
---

# Frp的下载地址

```
https://github.com/fatedier/frp/releases

Frp的官方中文文档: 

https://github.com/fatedier/frp/blob/master/README_zh.md
```

<!---more-->

# 服务端frps.ini

```
[common]
bind_port = 7080 #客户端和服务端之间通信的端口
```

# 客户端frpc.ini

```
[common]
server_addr = X.X.X.X  #服务端IP
server_port = 7080  #与服务端bind_port一致

[ssh]
# tcp | udp | http | https | stcp | xtcp, default is tcp
type = tcp
local_ip = 127.0.0.1
local_port = 8080
remote_port = 6080
```

# 启动

```
./frps -c ./frps.ini #服务端启动(linux)
frpc.exe -c frpc.ini #客户端启动(windows)
```

# 制作为服务

```
# 需要先 cd 到 frp 解压目录.

# 复制文件
cp frps /usr/local/bin/frps
mkdir /etc/frp
cp frps.ini /etc/frp/frps.ini

# 编写 frp service 文件，以 centos7 为例,适用于 debian
vim /usr/lib/systemd/system/frps.service
# 内容如下
[Unit]
Description=frps
After=network.target

[Service]
TimeoutStartSec=30
ExecStart=/usr/local/bin/frps -c /etc/frp/frps.ini
ExecStop=/bin/kill $MAINPID

[Install]
WantedBy=multi-user.target

# 启动 frp 并设置开机启动
systemctl enable frps
systemctl start frps
systemctl status frps

# 部分服务器上,可能需要加 .service 后缀来操作,即:
systemctl enable frps.service
systemctl start frps.service
systemctl status frps.service
```


