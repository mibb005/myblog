---
title: docker.gitlab
date: 2019-05-10 17:20:34
tags:
    - docker
    - gitlab
---

# 拉取镜像并启动

```
docker pull gitlab/gitlab-ce
# 拉取镜像
mkdir -p gitlab/config gitlab/logs gitlab/data
# 创建映射文件
sudo docker run -d -p 8443:443 -p 8081:80 -p 8022:22 \
--name gitlab --restart always \
--volume $PWD/gitlab/config:/etc/gitlab \
--volume $PWD/gitlab/logs:/var/log/gitlab \
--volume $PWD/gitlab/data:/var/opt/gitlab \
gitlab/gitlab-ce:latest
# 启动
8081 访问
```
