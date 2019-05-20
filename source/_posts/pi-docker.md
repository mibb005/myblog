---
title: 树莓派安装docker
date: 2019-05-20 16:18:27
tags:
 - pi
 - docker
---

# 使用快捷脚本安装

```
curl -fsSL https://get.docker.com -o get-docker.sh

sh get-docker.sh
```

> 注意：

> “dpkg ”是“Debian Packager ”的简写。为 “Debian” 专门开发的套件管理系统，方便软件的安装、更新及移除。所有源自“Debian”的“Linux ”> 发行版都使用 “dpkg”，例如 “Ubuntu”、“Knoppix ”等。dpkg是Debian软件包管理器的基础，在刚才安装docker时，dpkg被中断，我们可以使用> “sudo dpkg --configure -a”命令来重新配置和释放所有的软件包。

# 将docker权限添加给普通用户

```
sudo gpasswd -a $USER docker     #将登陆用户加入到docker用户组中
newgrp docker     #更新用户组
```
