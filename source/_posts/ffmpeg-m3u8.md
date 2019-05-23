---
title: ffmpeg视频m3u8切片
date: 2019-05-23 10:10:57
tags:
 - ffmpeg
---

# 首先将视频文件转为视频编码h.264，音频编码aac格式的mp4文件 

1. 使用ffprobe查看文件编码方式

```
ffprobe xxx.mp4
```

2. 如果不是mp4的，可以用如下命令进行转

```
ffmpeg -i input.mkv -acodec copy -vcodec copy out.mp4 
```

# 将mp4转为完整的ts

```
ffmpeg -i out.mp4 -c copy -bsf h264_mp4toannexb output.ts  
```

*说明*
> 为什么要用-bsf h264_mp4toannexb,主要是因为使用了mp4中的h264编码，而h264有两种封装： 
一种是annexb模式，传统模式，有startcode，SPS和PPS是在ES中；另一种是mp4模式，一般mp4、mkv、avi会没有startcode，SPS和PPS以及其它信息被封装在container中，每一个frame前面是这个frame的长度，很多解码器只支持annexb这种模式，因此需要将mp4做转换；在ffmpeg中用h264_mp4toannexb_filter可以做转换；所以需要使用-bsf h264_mp4toannexb来进行转换；

# 将ts切片，并生成m3u8文件

```
ffmpeg -i output.ts -c copy -map 0 -f segment -segment_list playlist.m3u8 -segment_time 5 output%03d.ts  
#其中segment 就是切片，-segment_time表示隔几秒进行切一个文件
```



