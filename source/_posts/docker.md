---
title: Docker开启远程进程服务以及VSCode、Idea等IDE连接使用远程
date: 2019-05-09 16:47:10
tags: docker
---
# Docker远程服务

开发环境大多使用的的是windows系统，服务器运行环境一般采用Linux系统，这时候生成镜像时用到远程连接Docker服务。

<!---more-->

## 开启Docker远程

如果只是临时使用远程docker，使用以下命令：

```
sudo dockerd -H unix:///var/run/docker.sock -H tcp://0.0.0.0:2375
```
如果使用docker启动时开启远程docker，则修改 /lib/systemd/system/docker.service 的ExecStart(不同版本的docker可能不同，处理思路类似)

```
vim /lib/systemd/system/docker.service
```

原docker.service配置中的ExecStart配置项

```
ExecStart=/usr/bin/dockerd -H unix://
```

修改为

```
ExecStart=/usr/bin/dockerd -H unix:///var/run/docker.sock -H tcp://0.0.0.0:2375
```
重启Docker配置生效

```
systemctl daemon-reload
systemctl restart docker
```
