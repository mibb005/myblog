---
title: node.js express 配置模块config-lite的用法
date: 2019-11-15 09:31:27
tags:
 - node
 - Express
---

# 安装

```
npm i config-lite --save 
```

# 使用方法

配置文件的示例路径：项目文件夹/config/default.js:

```
'use strict';
 
module.exports = {
    name: 'zz',
    port: 2000,
    
    /** mysql settings */
    mysql: {
        host: '127.0.0.1',
        port: 3306,
        user: 'root',
        db: 'auto_mon',
        pass: '000000',
        char: 'utf8mb4'
    },
    debug: true
}
```

调用  index.js:

```
import config=require('config-lite')  //先引入配置模块
 
//config便是配置对象，通过config.port config.mysql调用其配置属性
console.log(config.name); 
console.log(config.port); 
```

# 为什么要使用配置模块？

不管是小项目还是大项目，将配置与代码分离是一个非常好的做法。我们通常将配置写到一个配置文件里，如 config.js 或 config.json，并放到项目的根目录下。但通常我们都会有许多环境，如本地开发环境、测试环境和线上环境等，不同的环境的配置不同，我们不可能每次部署时都要去修改引用config.test.js 或者 config.production.js。config-lite 模块正是你需要的。

config-lite是一个轻量的读取配置文件的模块。config-lite 会根据环境变量（NODE_ENV）的不同从当前执行进程目录下的 config 目录加载不同的配置文件。如果不设置NODE_ENV，则读取默认的 default 配置文件，如果设置了NODE_ENV，则会合并指定的配置文件和 default 配置文件作为配置，config-lite 支持 .js、.json、.node、.yml、.yaml 后缀的文件。

如果程序以NODE_ENV=test node app启动，则通过require('config-lite')会依次降级查找config/test.js、config/test.json、config/test.node、config/test.yml、config/test.yaml并合并 default 配置;

如果程序以NODE_ENV=production node app启动，则通过require('config-lite')会依次降级查找config/production.js、config/production.json、config/production.node、config/production.yml、config/production.yaml并合并 default 配置。
