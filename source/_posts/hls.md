---
title: HLS协议介绍
date: 2019-05-23 16:03:09
tags:
 - hls
 - m3u8
---

今天来介绍一下HLS协议，这个协议是由苹果公司提出并推广开来的。来一段维基百科的定义。

<!--more-->

> HTTP Live Streaming（缩写是HLS）是一个由苹果公司提出的基于HTTP的流媒体网络传输协议。是苹果公司QuickTime X和iPhone软件系统的一部分。它的工作原理是把整个流分成一个个小的基于HTTP的文件来下载，每次只下载一些。当媒体流正在播放时，客户端可以选择从许多不同的备用源中以不同的速率下载同样的资源，允许流媒体会话适应不同的数据速率。在开始一个流媒体会话时，客户端会下载一个包含元数据的extended M3U (m3u8)playlist文件，用于寻找可用的媒体流。
HLS只请求基本的HTTP报文，与实时传输协议（RTP)不同，HLS可以穿过任何允许HTTP数据通过的防火墙或者代理服务器。它也很容易使用内容分发网络来传输媒体流。
苹果公司把HLS协议作为一个互联网草案（逐步提交），在第一阶段中已作为一个非正式的标准提交到IETF。但是，即使苹果偶尔地提交一些小的更新，IETF却没有关于制定此标准的有关进一步的动作。

# 协议简介

HLS协议规定：

* 视频的封装格式是TS。
* 视频的编码格式为H264,音频编码格式为MP3、AAC或者AC-3。
* 除了TS视频文件本身，还定义了用来控制播放的m3u8文件（文本文件）。

为什么苹果要提出HLS这个协议，其实他的主要是为了解决RTMP协议存在的一些问题。比如RTMP协议不使用标准的HTTP接口传输数据，所以在一些特殊的网络环境下可能被防火墙屏蔽掉。但是HLS由于使用的HTTP协议传输数据，不会遇到被防火墙屏蔽的情况（该不会有防火墙连80接口都不放过吧）。
另外于负载，RTMP是一种有状态协议，很难对视频服务器进行平滑扩展，因为需要为每一个播放视频流的客户端维护状态。而HLS基于无状态协议（HTTP），客户端只是按照顺序使用下载存储在服务器的普通TS文件，做负责均衡如同普通的HTTP文件服务器的负载均衡一样简单。
另外HLS协议本身实现了码率自适应，不同带宽的设备可以自动切换到最适合自己码率的视频播放。其实HLS最大的优势就是他的亲爹是苹果。苹果在自家的IOS设备上只提供对HLS的原生支持，并且放弃了flash。Android也迫于平果的“淫威”原生支持了HLS。这样一来flv，rtmp这些Adobe的视频方案要想在移动设备上播放需要额外下点功夫。当然flash对移动设备造成很大的性能压力确实也是自身的问题。
但HLS也有一些无法跨越的坑，比如采用HLS协议直播的视频延迟时间无法下到10秒以下，而RTMP协议的延迟最低可以到3、4秒左右。**所以说对直播延迟比较敏感的服务请慎用HLS。**

![图片alt](hls/1328846-39f999dab36167db ''图片title'')

来解释一下这张图，从左到右讲，左下方的inputs的视频源是什么格式都无所谓，他与server之间的通信协议也可以任意（比如RTMP），总之只要把视频数据传输到服务器上即可。这个视频在server服务器上被转换成HLS格式的视频（既TS和m3u8文件）文件。细拆分来看server里面的Media encoder的是一个转码模块负责将视频源中的视频数据转码到目标编码格式（H264）的视频数据，视频源的编码格式可以是任何的视频编码格式（参考《视频技术基础》）。转码成H264视频数据之后，在stream segmenter模块将视频切片，切片的结果就是index file（m3u8）和ts文件了。图中的Distribution其实只是一个普通的HTTP文件服务器，然后客户端只需要访问一级index文件的路径就会自动播放HLS视频流了。

# HLS的index文件

所谓index文件就是之前说的m3u8文本文件。

![图片alt](hls/1328846-d0df01e6b2dec3bb ''图片title'')

如上图所示，客户端播放HLS视频流的逻辑其实非常简单，先下载一级Index file，它里面记录了二级索引文件（Alternate-A、Alternate-B、Alternate-C）的地址，然后客户端再去下载二级索引文件，二级索引文件中又记录了TS文件的下载地址，这样客户端就可以按顺序下载TS视频文件并连续播放。

# 一级index文件

视频源：https://dco4urblvsasc.cloudfront.net/811/81095_ywfZjAuP/game/index.m3u8

```
#EXTM3U
#EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=1064000
1000kbps.m3u8
#EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=564000
500kbps.m3u8
#EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=282000
250kbps.m3u8
#EXT-X-STREAM-INF:PROGRAM-ID=1,BANDWIDTH=2128000
2000kbps.m3u8
```

bandwidth指定视频流的比特率，PROGRAM-ID无用无需关注，每一个#EXT-X-STREAM-INF的下一行是二级index文件的路径，可以用相对路径也可以用绝对路径。例子中用的是相对路径。这个文件中记录了不同比特率视频流的二级index文件路径，客户端可以自己判断自己的现行网络带宽，来决定播放哪一个视频流。也可以在网络带宽变化的时候平滑切换到和带宽匹配的视频流。

```
#EXTM3U
#EXT-X-PLAYLIST-TYPE:VOD
#EXT-X-TARGETDURATION:10
#EXTINF:10,
2000kbps-00001.ts
#EXTINF:10,
2000kbps-00002.ts
#EXTINF:10,
2000kbps-00003.ts
#EXTINF:10,
2000kbps-00004.ts
#EXTINF:10,

... ...

#EXTINF:10,
2000kbps-00096.ts
#EXTINF:10,
2000kbps-00097.ts
#EXTINF:10,
2000kbps-00098.ts
#EXTINF:10,
2000kbps-00099.ts
#EXTINF:10,
2000kbps-00100.ts
#ZEN-TOTAL-DURATION:999.66667
#ZEN-AVERAGE-BANDWIDTH:2190954
#ZEN-MAXIMUM-BANDWIDTH:3536205
#EXT-X-ENDLIST
```

二级文件实际负责给出ts文件的下载地址，这里同样使用了相对路径。#EXTINF表示每个ts切片视频文件的时长。#EXT-X-TARGETDURATION指定当前视频流中的切片文件的最大时长，也就是说这些ts切片的时长不能大于#EXT-X-TARGETDURATION的值。#EXT-X-PLAYLIST-TYPE:VOD的意思是当前的视频流并不是一个直播流，而是点播流，换句话说就是该视频的全部的ts文件已经被生成好了，#EXT-X-ENDLIST这个表示视频结束，有这个标志同时也说明当前的流是一个非直播流。

# 播放模式

* 点播VOD的特点就是当前时间点可以获取到所有index文件和ts文件，二级index文件中记录了所有ts文件的地址。这种模式允许客户端访问全部内容。上面的例子中就是一个点播模式下的m3u8的结构。

* Live 模式就是实时生成M3u8和ts文件。它的索引文件一直处于动态变化的，播放的时候需要不断下载二级index文件，以获得最新生成的ts文件播放视频。如果一个二级index文件的末尾没有#EXT-X-ENDLIST标志，说明它是一个Live视频流。

客户端在播放VOD模式的视频时其实只需要下载一次一级index文件和二级index文件就可以得到所有ts文件的下载地址，除非客户端进行比特率切换，否则无需再下载任何index文件，只需顺序下载ts文件并播放就可以了。但是Live模式下略有不同，因为播放的同时，新ts文件也在被生成中，所以客户端实际上是下载一次二级index文件，然后下载ts文件，再下载二级index文件（这个时候这个二级index文件已经被重写，记录了新生成的ts文件的下载地址）,再下载新ts文件，如此反复进行播放。

> 转载： https://www.jianshu.com/p/426425cad08a




