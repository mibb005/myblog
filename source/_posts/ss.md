---
title: Linux 安装 Shadowsocks 客户端 并全局代理
date: 2019-09-02 15:17:11
tags:
 - Ubuntu
 - Shadowsocks
---

# 安装 Shadowsocks

shadowsocks 客户端和服务器端是同一个安装包，两个脚本不同，客户端模式是 sslocal，服务器端模式是 ssserver

## CentOS 下搭建

```
# 安装pip
yum install python-setuptools && easy_install pip
# 安装shadowsocks
pip install shadowsocks
```

## Ubuntu 下搭建

```
# 安装pip
apt-get install python-pip
# 安装shadowsocks
pip install shadowsocks
```

sslocal 运行 shadowsocks 客户端

```
nohup sslocal -s your_server_ip -p your_server_port -l 1080 -k your_server_passwd -t 600 -m rc4-md5 > /dev/null 2>&1 &
```
解释

```
# your_server_ip: 服务器 IP
# your_server_port: 服务器端口
# your_server_passwd: SS 密码
# rc4-md5: 加密方式
```
或创建一个 shadowsocks 的配置文件

```
vim /etc/shadowsocks.json
{
    "server":"your_server_ip",      # ss 服务器 IP
    "server_port":your_server_port, # 端口
    "local_address": "127.0.0.1",   # 本地 IP
    "local_port":1080,              # 本地端口
    "password":"your_server_passwd",# 连接 ss 密码
    "timeout":300,                  # 等待超时
    "method":"rc4-md5",             # 加密方式
    "fast_open": false,             # true 或 false。
    "workers": 1                    # 工作线程数
}
```

解释（使用时删除井号及后面文字）

```
fast_open: 如果你的服务器 Linux 内核在3.7+，可以开启 fast_open 以降低延迟。
开启方法： echo 3 > /proc/sys/net/ipv4/tcp_fastopen 开启之后，
将 fast_open 的配置设置为 true 即可
```

以 json 文件来运行启动 sslocal

```
nohup sslocal -c /etc/shadowsocks.json > /dev/null 2>&1 &
```

增加开启自动启动，执行：

```
echo " nohup sslocal -c /etc/shadowsocks.json > /dev/null 2>&1 &" /etc/rc.local
```

执行 `ps aux | grep sslocal | grep -v “grep”` 可查看后台 sslocal 是否运行。

# 安装 Privoxy

上述安好了 shadowsocks，但它是 socks5 代理，我门在 shell 里执行的命令，发起的网络请求现在还不支持 socks5 代理，只支持 http／https 代理。为了我门需要安装 privoxy 代理，它能把电脑上所有 http 请求转发给 shadowsocks。

```
wget http://www.privoxy.org/sf-download-mirror/Sources/3.0.26%20%28stable%29/privoxy-3.0.26-stable-src.tar.gz
tar -zxvf privoxy-3.0.26-stable-src.tar.gz
cd privoxy-3.0.26-stable
```

若上述地址无法连接可替换为本站链接 http://blog.liuguofeng.com/wp-content/uploads/2018/07/privoxy-3.0.26-stable-src.tar.gz

新建一个 privoxy 用户用来运行

```
useradd privoxy
```
编译安装

```
autoheader && autoconf
./configure
make && make install
```

配置

```
vim /usr/local/etc/privoxy/config
```

确保下面两句没有被注释

```
listen-address 127.0.0.1:8118      # 8118 是默认端口，不用改，下面会用到
forward-socks5t / 127.0.0.1:1080   # 这里的端口写 shadowsocks 的本地端口
```

# 使用 privoxy-gfwlist

获取 privoxy-gfwlist

```
wget https://raw.github.com/zfl9/gfwlist2privoxy/master/gfwlist2privoxy 
bash gfwlist2privoxy '127.0.0.1:1080'
cp  gfwlist.action /usr/local/etc/privoxy
echo 'actionsfile gfwlist.action' >> /usr/local/etc/privoxy/config
```

启动 privoxy

```
privoxy --user privoxy /usr/local/etc/privoxy/config
```

配置 /etc/profile

```
vim /etc/profile
```

输入内容

```
export http_proxy=http://127.0.0.1:8118   #这里的端口和上面 privoxy 中的保持一致
export https_proxy=http://127.0.0.1:8118
export ftp_proxy=http://127.0.0.1:8118    #ftp可以不用
export no_proxy="localhost, 127.0.0.1, ::1, aliyuncs.com, qiniuapi.com, qiniu.com, qiniucdn.com"
```

刷新

```
source /etc/profile
```

测试

```
curl https://www.google.com
```

若不能访问，重启机器然后依次

```
nohup sslocal -c /etc/shadowsocks.json /dev/null 2>&1 &
privoxy --user privoxy /usr/local/etc/privoxy/config
```