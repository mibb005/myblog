---
title: Hexo的Next主题详细配置
date: 2019-05-09 15:12:23
tags: hexo
---
在 Hexo 中有两份主要的配置文件，其名称都是 _config.yml。 其中，一份位于站点根目录下，主要包含 Hexo 本身的配置；另一份位于主题目录下，这份配置由主题作者提供，主要用于配置主题相关的选项。
为了描述方便，在以下说明中，将前者称为站点配置文件， 后者称为主题配置文件。
以下所有终端执行的命令都在你的Hexo根目录下

## 基本信息配置

```
基本信息包括：博客标题、作者、描述、语言等等。
```

打开 站点配置文件 ，找到Site模块

```
title: 标题
subtitle: 副标题
description: 描述
author: 作者
language: 语言（简体中文是zh-Hans）
timezone: 网站时区（Hexo 默认使用您电脑的时区，不用写）
```

## 菜单设置

菜单包括：首页、归档、分类、标签、关于等等

我们刚开始默认的菜单只有首页和归档两个，不能够满足我们的要求，所以需要添加菜单，打开 主题配置文件 找到Menu Settings

```
menu:
  home: / || home                          //首页
  archives: /archives/ || archive          //归档
  categories: /categories/ || th           //分类
  tags: /tags/ || tags                     //标签
  about: /about/ || user                   //关于
  #schedule: /schedule/ || calendar        //日程表
  #sitemap: /sitemap.xml || sitemap        //站点地图
  #commonweal: /404/ || heartbeat          //公益404
```

看看你需要哪个菜单就把哪个取消注释打开就行了；
关于后面的格式，以archives: /archives/ || archive为例：
|| 之前的/archives/表示标题“归档”，关于标题的格式可以去themes/next/languages/zh-Hans.yml中参考或修改
||之后的archive表示图标，可以去Font Awesome中查看或修改，Next主题所有的图标都来自Font Awesome。

## Next主题样式设置

我们百里挑一选择了Next主题，不过Next主题还有4种风格供我们选择，打开 主题配置文件 找到Scheme Settings

```
# Schemes
# scheme: Muse
# scheme: Mist
# scheme: Pisces
scheme: Gemini
```
4种风格大同小异，本人用的是Gemini风格，你们可以选择自己喜欢的风格。

## 侧栏设置

侧栏设置包括：侧栏位置、侧栏显示与否、文章间距、返回顶部按钮等等

打开 主题配置文件 找到sidebar字段

```
sidebar:
# Sidebar Position - 侧栏位置（只对Pisces | Gemini两种风格有效）
  position: left        //靠左放置
  #position: right      //靠右放置

# Sidebar Display - 侧栏显示时机（只对Muse | Mist两种风格有效）
  #display: post        //默认行为，在文章页面（拥有目录列表）时显示
  display: always       //在所有页面中都显示
  #display: hide        //在所有页面中都隐藏（可以手动展开）
  #display: remove      //完全移除

  offset: 12            //文章间距（只对Pisces | Gemini两种风格有效）

  b2t: false            //返回顶部按钮（只对Pisces | Gemini两种风格有效）

  scrollpercent: true   //返回顶部按钮的百分比
```

## 头像设置

打开 主题配置文件 找到Sidebar Avatar字段

```
# Sidebar Avatar
avatar: /images/header.jpg
```
这是头像的路径，只需把你的头像命名为header.jpg（随便命名）放入themes/next/source/images中，将avatar的路径名改成你的头像名就OK啦！




