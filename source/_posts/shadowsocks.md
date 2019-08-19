---
title: Centos 搭建ss服务器
date: 2019-08-19 16:12:25
tags:
 - Centos
 - shadowsocks
---

# 安装pip

yum install python-setuptools && easy_install pip

# 安装 shadowsocks

pip install shadowsocks

# 配置

vi /etc/shadowsocks.json

```
{

"server":"你的ip地址",

"server_port":443,

"local_address":"127.0.0.1",

"local_port":1080,

"password":"MyPass",

"timeout":600,

"method":"rc4-md5"

}
```

# 启动服务

ssserver -c /etc/shadowsocks.json -d start

# 停止服务

ssserver -c /etc/shadowsocks.json -d stop
