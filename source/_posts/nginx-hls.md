---
title: windows下配置nginx的m3u8点播服务
date: 2019-05-23 10:19:33
tags:
 - nginx
 - m3u8
---
# 下载ffmpeg

# 新建子目录：nginx-1.5.10\html\hls，把生成的m3u8和切片好的ts文件或目录拷贝到hls目录下  

# 打开任务管理其，杀掉ngnix.exe,重启ngnix.exe 

# 打开vlc播放器, 【打开网络串流】菜单，输入url：http://127.0.0.1/hls/playlist.m3u8 即可测试播放了
