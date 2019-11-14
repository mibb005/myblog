---
title: node打包exe
date: 2019-11-14 09:31:26
tags:
 - node
 - pkg
---

# 项目地址

```
https://github.com/zeit/pkg
```

# 安装

```
npm install -g pkg
```

# 打包

```
pkg entrance.js
```

即可打包linux,macos,win3个平台的可执行文件。entrance.js为你node项目的入口文件。

如果只想打包windows下的exe，则加上-t参数。win即为打包成windows平台下的exe文件

```
pkg -t win entrance.js
```

稍等片刻后项目目录下就会生成打包好的`entrance.exe`文件。

pkg会自动从入口文件开始查找依赖的文件并全数打包进去，无须修改项目里的任何代码。

# 其他

pkg可以根据`package.json`下的配置进行打包，默认入口文件为`bin`指向的文件。
执行
```
pkg .
```

或是

```
pkg package.json
```

即可自动按照`package.json`的配置打包。

```
//package.json
{
    //其他配置项
    "bin": "service.js",//入口文件
    "pkg": {
        "scripts": [
            "build/**/*.js"//需要打包进来的其他js文件，可添加多个
        ],
        "assets": [
            "dist/**/*"//静态文件的目录，可添加多个
        ]
    }
}    
```
注意：静态文件需要在项目中将文件的引用换成

```
path.join(__dirname, 'dist')
```

的形式，才可以正常打包，否则可能会读取不到。
