---
title: 树莓派安装docker nextcloud
date: 2019-05-21 14:38:20
tags:
 - docker
 - pi
 - nextcloud
---

# 查找nextcloud

```
docker search nextcloud
```

# 拉去

```
docker pull nextcloud
```

# 启动

```
docker run -d --restart=always --name nextcloud -p 8080:80 -v $PWD/docker/nextcloud:/data nextcloud
```

