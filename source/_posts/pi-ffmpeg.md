---
title: 树莓派使用ffmpeg说明
date: 2019-05-22 14:40:11
tags:
 - pi
 - ffmpeg
---

# 编译ffmpeg

```
https://www.jianshu.com/p/dec9bf9cffc9
```

# 用法

```
su - lyj
#切换用户lyj

cd /ffmpeg_build/rpilib
#切换目录

LD_LIBRARY_PATH=./mylib
#添加临时环境变量

./ffmpeg -v
#执行当前目录下的ffmpeg

```
