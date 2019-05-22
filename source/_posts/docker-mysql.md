---
title: docker安装mysql
date: 2019-05-22 09:32:18
tags:
---

# docker pull mysql

# 运行

```
docker run -p 3306:3306 --name mymysql -v $PWD/conf:/etc/mysql/conf.d -v $PWD/logs:/logs -v $PWD/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql
```

命令说明：
* -p 3306:3306：将容器的 3306 端口映射到主机的 3306 端口。
* -v -v $PWD/conf:/etc/mysql/conf.d：将主机当前目录下的 conf/my.cnf 挂载到容器的 /etc/mysql/my.cnf。
* -v $PWD/logs:/logs：将主机当前目录下的 logs 目录挂载到容器的 /logs。
* -v $PWD/data:/var/lib/mysql ：将主机当前目录下的data目录挂载到容器的 /var/lib/mysql 。
* -e MYSQL_ROOT_PASSWORD=123456：初始化 root 用户的密码。


# 解决 错误 ER_NOT_SUPPORTED_AUTH_MODE: Client does not support authentication protocol requested by server; consider upgrading MySQL client

```
docker exec -it mysql(这里的mysql是指你启动时的容器名称) bash

mysql -uroot -p

ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '123456';
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';
SELECT plugin FROM mysql.user WHERE User = 'root';
```
