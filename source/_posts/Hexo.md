---
title: Hexo用法
date: 2019-05-09 14:35:44
tags: hexo
categories: hexo
---
输入命令

``` bash
$ npm install -g hexo-cli
```

<!---more-->

依旧用hexo -v查看一下版本

至此就全部安装完了。

接下来初始化一下hexo
``` bash
hexo init myblog
```
这个myblog可以自己取什么名字都行，然后

cd myblog //进入这个myblog文件夹
``` bash
npm install
```
新建完成后，指定文件夹目录下有：

node_modules: 依赖包
public：存放生成的页面
scaffolds：生成文章的一些模板
source：用来存放你的文章
themes：主题
** _config.yml: 博客的配置文件**
``` bash
hexo g
hexo server
```
打开hexo的服务，在浏览器输入localhost:4000就可以看到你生成的博客了。

新建文档
```
$ hexo new {title}
```