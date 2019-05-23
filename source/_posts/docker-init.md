---
title: docker ubuntu 安装
date: 2019-05-10 11:17:28
tags: docker
---

# 安装

```
sudo apt install docker.io
```

<!---more-->

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

# 运行交互式的容器

```
docker run -i -t microsoft/dotnet  /bin/bash
各个参数解析：
-t:在新容器内指定一个伪终端或终端。
-i:允许你对容器内的标准输入 (STDIN) 进行交互。
ctrl + D 退出
```
