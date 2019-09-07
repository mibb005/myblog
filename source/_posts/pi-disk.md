---
title: 树莓派—raspbian软件源
date: 2019-09-07 14:00:46
tags:
 - pi
---


```
sudo apt-get update
```

# 一键换源
直接执行以下两行命令，即可替换将官方默认软件源替换为中科大或清华镜像源。
2018.05.18更新：新的默认源为raspbian.raspberrypi.org
因此一键换源相应改为

```
sudo sed -i 's#://raspbian.raspberrypi.org#s://mirrors.ustc.edu.cn/raspbian#g' /etc/apt/sources.list 
sudo sed -i 's#://archive.raspberrypi.org/debian#s://mirrors.ustc.edu.cn/archive.raspberrypi.org/debian#g' /etc/apt/sources.list.d/raspi.list
```

或

```
sudo sed -i 's#://raspbian.raspberrypi.org#s://mirrors.tuna.tsinghua.edu.cn/raspbian#g' /etc/apt/sources.list
sudo sed -i 's#://archive.raspberrypi.org/debian#s://mirrors.tuna.tsinghua.edu.cn/raspberrypi#g' /etc/apt/sources.list.d/raspi.list
```

