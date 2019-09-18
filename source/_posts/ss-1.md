---
title: ss无法上网的排查
date: 2019-05-22 14:10:45
tags:
 - shadowsocks
---

从日志开始排查。

登录服务器端

```
$ ssh root@[IP]
```

关闭 ss，再次启动并其指定日志输出文件

```
$ ssserver -c /etc/shadowsocks.json -d stop
$ ssserver -c /etc/shadowsocks.json --log-file /var/log/shadowsocks.log -d start
```

启动后查看日志

```
$ tail -f /var/log/shadowsocks.log
```
由日志可见启动正常。

本地电脑开启ss客户端，打开Google网址。返回终端查看服务器日志，发现日志没有任何变化。
于是可以断定，是客户端向服务器端发请求的这个过程失败了。
本地电脑检查客户端配置，都没有错。感觉是端口被墙了。

于是在服务器端修改 ss 端口。

```
$ vim /etc/shadowsocks.json
{
"server":"0.0.0.0",
"server_port":8388,
"local_address": "127.0.0.1",
"local_port":1080,
"password":"helloworld",
"timeout":300,
"method":"rc4-md5"
}
```

找到 server_port 字段，修改它的值。注意不要和现有端口冲突。

重启 ss 服务,并再次查看日志输出：

```
$ ssserver -c /etc/shadowsocks.json -d stop
$ ssserver -c /etc/shadowsocks.json --log-file /var/log/shadowsocks.log -d start
$ tail -f /var/log/shadowsocks.log
```

本地电脑再次访问Google网址，发现日志变化了。同时本地电脑网址也打开了🎉。