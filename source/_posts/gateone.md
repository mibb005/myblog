---
title: gateone docker安装配置
date: 2019-06-13 15:13:37
tags:
 - gateone
 - docker
---

GateOne是一个能在阅读器上运转的Terminal SSH客户端，不管你在那里，只需有网，你便可以用阅读器操控你的VPS云主机，还支援右键复制/粘贴等客户端经常使用效能，包罗多窗口等，应用起来异常便宜，同时别的人也能够应用，之前也说过一品种似的东西WebSSH2，检察：WebSSH2装置教程，都挺好用的，这边就说下应用Docker快速装置GateOne，并增加SSL证明。

# 拉取镜像

```
docker pull liftoff/gateone
```

# 启动容器

```
docker run -d -p 443:8000 -h Rats --name gateone -v $PWD/gateone:/etc/gateone/conf.d liftoff/gateone gateone

参数说明：
-d/-t：决定镜像是使用Deamon（后台）模式启动，或者显示启动过程 
-p 443:8000：绑定端口，注意：GateOne强制使用SSL，8000端口为Docker容器内的固定映射端口，请只改动冒号前面的端口，不要动后面的端口号！ 
-h hostname：设置Docker容器的主机名（这个将会显示在你的浏览器标题中） 
--name gateone：设置Docker容器的名称（不是主机名），用来docker ps时识别用 
liftoff/gateone：镜像名称 
gateone：启动命令行，勿动（默认命令行会发生Python io_loop报错，故使用此命令行来避免错误）

```
