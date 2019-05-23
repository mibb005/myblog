---
title: nginxhls流媒体m3u8跨域问题
date: 2019-05-23 17:24:44
tags:
 - nginx
 - hls
 - m3u8
---

# 问题

Nginx以 `HLS+m3u8`方式搭建的流媒体报`https://www.***.com/crossdomain.xml`找不到

<!---more-->

# 解决方案

在跨域的网站根目录放crossdomain.xml文件，下面是允许所有的网站（一般不采取这样的方式，我只是方便调试）均可以跨越访问资源配置如下：

```
 <?xml version="1.0"?>
 <!DOCTYPE cross-domain-policy SYSTEM "http://www.macromedia.com/xml/dtds/cross-domain-policy.dtd">
 <cross-domain-policy>
 <allow-access-from domain="*" />
 <allow-http-request-headers-from domain="*" headers="*"/>
 </cross-domain-policy>
```
