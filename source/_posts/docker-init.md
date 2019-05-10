---
title: docker ubuntu 安装
date: 2019-05-10 11:17:28
tags:
---

# 安装

```
sudo apt install docker.io
```

# 启动

```
sudo systemctl start docker
sudo systemctl enable docker
```

# 将docker权限添加给普通用户

```
sudo gpasswd -a $USER docker     #将登陆用户加入到docker用户组中
newgrp docker     #更新用户组
```
