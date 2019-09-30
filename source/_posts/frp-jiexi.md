---
title: 关于内网穿透frp程序的分析
date: 2019-09-30 15:17:03
tags:
 - go
 - frp
---

目前市面上实现动态域名解析的服务商花生壳,半开源的ngrok都是使用内网穿透技术来实现的,内网穿透可以解决什么样的需求呢?

* 有台小破电脑,想用来做一台服务器,希望出了家门也能够访问,迫于服务商不提供固定IP,不能简单的配置网关端口映射达到目的,这时我们可以采用内网穿透技术,让外网可以畅游内网.
* 目前物联网的概念很火,买了台用于监控室内防盗的监控设备,同样希望在任何时候都可以看见家里的状态,这样我们可以使用推流的方式向外服务器推送视频流,这样的话就会一直存在视频的传送.这里我们也可以使用内网穿透,实现需要时才推流

# 关于Frp

项目开源地址github,当然这不是我的作品,我只是伸手党.该程序使用golang实现.

> frp 是一个可用于内网穿透的高性能的反向代理应用，支持 tcp, udp, http, https 协议。

## frp 的作用

* 利用处于内网或防火墙后的机器，对外网环境提供 http 或 https 服务。
* 对于 http, https 服务支持基于域名的虚拟主机，支持自定义域名绑定，使多个域名可以共用一个80端口。
* 利用处于内网或防火墙后的机器，对外网环境提供 tcp 和 udp 服务，例如在家里通过 ssh 访问处于公司内网环境内的主机。

# 原理分析

## 内网端程序分析

内网程序的入口在frp/cmd/frpc下,通过入口函数,可以看见第一步其实就是读取配置文件的信息,具体怎么个读法这里就不细细展开,感兴趣的读者可以自行查阅.

程序的真正的开始执行其实是在frp/client/server.go#Run()函数中,这里调用了frp/client/control.go#Run()函数,这个函数中首先执行的是和服务端程序的对接(授权),然后开启多个goroutine,分别用于控制保持与服务端长连接,控制读写消息.

### 登录过程

登录的代码在frp/client/control.go#login()函数中,首先

```
conn, err := net.ConnectServerByHttpProxy(           
                                config.ClientCommonCfg.HttpProxy,  
                                config.ClientCommonCfg.Protocol,//协议:tcp 或 kcp
                                fmt.Sprintf("%s:%d",config.ClientCommonCfg.ServerAddr,
                                            config.ClientCommonCfg.ServerPort))
```

获取到一个conn,这里的conn可以看成是对net.Conn的一个封装,这里的实现就是建立一个tcp/kcp的连接,从而获得一个socket句柄,供后续使用.接着往下看,执行一个msg.WriteMsg(conn, ctl.loginMsg),其目的是向上文的socket句柄写一个loginMsg,loginMsg就是对配置文件中设置privilege_token = 的加密封装,既然这里授权登录的请求发了出去,接下来就是等待授权结果,如果授权结果没有返回错误那么就表示登录成功了.这里继续保持socket句柄.

这里在谈谈它对发送接收消息的结构封装

```
[第一字节为消息类型][第二字节为消息长度][后续内容为消息体 json格式]
```

### 开启消息

在登录成功之后会开启一些goroutine

```
    go ctl.controler()  //控制代理的服务的状态监听,当服务死掉后重新发起重连
    go ctl.manager()    //心跳检查与发起重连 从chan中读取消息
    go ctl.writer()     //从chan中读取消息并向服务端发送
    go ctl.reader()     //从stocket中读取消息并解析,然后发送到chan
```

这里我们以ssh登录为例进行说明,下面是reader()的源码摘要

```
    encReader := crypto.NewReader(ctl.conn, []byte(config.ClientCommonCfg.PrivilegeToken))
    for {
        if m, err := msg.ReadMsg(encReader); err != nil {
            ... //异常处理
        } else {
            fmt.Println("新消息")
            ctl.readCh <- m
        }
    }
```

我们可以看到其实reader()函数就是纯读取消息,然后通过chan把消息转发出去,如果不这样的话,处理就有需要单独开启线程来处理,比较麻烦.那么我们就看看到底在哪里读取消息,在manager()中,存在一下代码

```
   case rawMsg, ok := <-ctl.readCh:
            if !ok {
                return
            }
            switch m := rawMsg.(type) {
            case *msg.ReqWorkConn:
                go ctl.NewWorkConn()
            case *msg.NewProxyResp:
                ... //  http等代理部分
            case *msg.Pong:
                ctl.lastPong = time.Now()
                ctl.Debug("receive heartbeat from server")
            }
        }
```

果不其然, 没有在reader()中开启的线程,在manager()中开启了.因为ssh是tcp协议实现,所以在NewWorkConn会执行

```
    if pxy, ok := ctl.proxies[startMsg.ProxyName]; ok {
        workConn.Debug("start a new work connection, localAddr: %s remoteAddr: %s", workConn.LocalAddr().String(), workConn.RemoteAddr().String())
        go pxy.InWorkConn(workConn) //pxy是tcpProxy
    } else {
        workConn.Close()
    }
```

也就是会执行一下的函数

```
func (pxy *TcpProxy) InWorkConn(conn frpNet.Conn) {
    //HttpProxy & HttpsProxy 都是执行这个函数
    HandleTcpWorkConnection(&pxy.cfg.LocalSvrConf, pxy.proxyPlugin, &pxy.cfg.BaseProxyConf, conn)
}
```

在HandleTcpWorkConnection()函数的末尾,调用了frpIo.Join(localConn, remote)实现双方的通信,这个函数的实现也是非常别致,

```
// Join two io.ReadWriteCloser and do some operations.
func Join(c1 io.ReadWriteCloser, c2 io.ReadWriteCloser) (inCount int64, outCount int64) {
    var wait sync.WaitGroup
    pipe := func(to io.ReadWriteCloser, from io.ReadWriteCloser, count *int64) {
        defer to.Close()
        defer from.Close()
        defer wait.Done()

        buf := pool.GetBuf(16 * 1024)
        defer pool.PutBuf(buf)
        *count, _ = io.CopyBuffer(to, from, buf)
    }

    wait.Add(2)
    go pipe(c1, c2, &inCount)
    go pipe(c2, c1, &outCount)
    wait.Wait()

    return
}
```

这个函数的功能就是实现两个数据流的数据交换功能.比如ssh连接时,服务器会将客户的请求转发到内网的客户端机子上,客户端机子得到一个来自于服务端的conn,同时客户端根据服务端发来的数据,同本地的ssh服务建立一个conn,之后的过程就是这两个conn之间的一个通信(通过上文的Join方法),也就是frp不在参与他们之间的一个通信。


## 服务端程序分析

相对于客户端程序，服务端程序的功能就比较的单一,就是实现一个连接的转发。程序的入口位于/cmd/frps/main.go,开始的步骤和客户端一样，都是解析配置文件，最后调用了/frp/server/server.go#Run(),这里面就调用一个HandleListener(...)函数，里面是实现也比较的简单，一个标准的tcp阻塞函数，当有新的连接来的时候开启一个goroutine,在这个goroutine中获取消息的内容,再根据不同的消息类型进行不同的处理,这里只有两种类型，分别为Login和NewWorkConn,很明显Login就是对客户端的授权处理,而NewWorkConn就是客户端发起的服务。下面分别分析。

### Login Auth过程

登录授权的入口在/frp/server/server.go#RegisterControl(...),验证的参数包括版本号,时间戳,private key,验证成功之后向客户端发送一个LoginResp的消息，并且生成一个id给这个连接。接下进行的操作和客户端的操作差不多。

```
    go ctl.writer()  //发送消息到客户端
    for i := 0; i < ctl.poolCount; i++ {
        ctl.sendCh <- &msg.ReqWorkConn{}   //与客户端开启多个连接
    }

    go ctl.manager()  //从ctl.readCh中读取并处理消息
    go ctl.reader()   //从conn中读取消息扔到ctl.rendCh中
    go ctl.stoper()   //当客户端断开了连接之后，关闭这个客户端的所有连接
```

这里就不细讲了，逻辑基本和客户端的实现一致。

### NewWorkConn 消息的处理

当收到新的连接的时候,也就是收到NewWorkConn这个消息时,会执行RegisterWorkConn(...)方法，这个方法的作用就是把这个conn加入到服务端的一个pool中,供后续使用。

# 总结

frp的大体流程就是上文描述的,当然还有许多的细节点,这里只是大体描述程序的流程,如果要深入的理解还需要自己去详细的阅读源码。