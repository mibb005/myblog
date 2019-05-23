---
title: Hexo中添加本地图片
date: 2019-05-23 16:36:23
tags:
 - hexo
---

* 把主页配置文件_config.yml 里的post_asset_folder:这个选项设置为true

* 在你的hexo目录下执行这样一句话npm install hexo-asset-image --save

* 运行hexo n "xxxx"来生成md博文时，/source/_posts文件夹内除了xxxx.md文件还有一个同名的文件夹

* 最后在xxxx.md中想引入图片时，先把图片复制到xxxx这个文件夹中，然后只需要在xxxx.md中按照markdown的格式引入图片：

![你想输入的替代文字](图片名.jpg)

