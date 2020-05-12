---
title: NodeJs - Express - apidoc 自动生成API文档
date: 2019-11-13 14:58:55
tags:
 - node
 - apidoc
---

# 安装

```
npm i apidoc -g   #全局安装
```

# 配置

方式一：根目录配置apidoc.json

```
{
  "name": "example",
  "version": "0.1.0",
  "description": "apiDoc basic example",
  "title": "Custom apiDoc browser title",
  "url" : "https://api.github.com/v1"
}
```

方式二：项目package.json配置api-doc

```
{
  "name": "helo",
  "version": "0.0.0",
  "private": true,
  "scripts": {
    "start": "node ./bin/www"
  },
  "dependencies": {
    "cookie-parser": "~1.4.3",
    "debug": "~2.6.9",
    "ejs": "~2.5.7",
    "express": "~4.16.0",
    "http-errors": "~1.6.2",
    "morgan": "~1.9.0"
  },
  "devDependencies": {
    "express-session": "^1.15.6",
    "mysql": "^2.15.0"
  },
  "apidoc": {  //配置api-doc
    "title": "接口文档", //Api-Doc的网页Title
    "url": "http://localhost:3000" //Api测试需要这个地址，地址必须正确
  }
}
```

然后通过在项目的public文件夹下面新建一个apidoc目录。接着，我们需要编写router里的代码，创建一个api目录，里面编写一个User.js接口的东西。

范例：

```
let express = require('express');
let router = express.Router();

/**
 * @api {post} /api/user/submit-login 用户登录
 * @apiDescription 用户登录
 * @apiName submit-login
 * @apiGroup User
 * @apiParam {string} loginName 用户名
 * @apiParam {string} loginPass 密码
 * @apiSuccess {json} result
 * @apiSuccessExample {json} Success-Response:
 *  {
 *      "success" : "true",
 *      "result" : {
 *          "name" : "loginName",
 *          "password" : "loginPass"
 *      }
 *  }
 * @apiSampleRequest http://localhost:3000/api/user/submit-login
 * @apiVersion 1.0.0
 */
router.post('/submit-login', function (req, res, next) {
    let loginName = req.body.loginName;
    let loginPass = req.body.loginPass;
    res.json({
        success: true,
        result: {
            name: loginName,
            password: loginPass
        }
    });
});

module.exports = router;
```

# 命令

```
apidoc -i routes/ -o public/apidoc/
```
