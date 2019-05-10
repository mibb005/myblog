---
title: docker.portainer
date: 2019-05-10 11:28:31
tags: docker
---

# 安装

```
docker pull portainer/portainer
```

# 启动

```
$ docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock \
   -v $PWD/portainer:/data \
   --name portainer --restart=always \
   portainer/portainer
说明:
-p 9000:9000 : portainer的端口映射为9000
-v /var/run/docker.sock:/var/run/docker.sock： 映射本地docker路径，即可管理本地docker
-v /home/docker/portainer:/data: 实现数据持久化，将portainer数据映射到本地。
```
