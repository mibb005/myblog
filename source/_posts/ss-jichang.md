---
title: 搭建飞机场教程
date: 2019-05-22 14:10:45
tags:
 - shadowsocks
---

# 第一步：安装宝塔Web管理面板

SSH登陆服务器，sodu su获得系统管理权限，输入下面代码进行宝塔面板安装：

```
wget -O install.sh http://download.bt.cn/install/install-ubuntu.sh && bash install.sh
```

安装完毕后登陆管理后台安装LNMP环境，数据库推荐MYSQL5.6+，如果因内存小可以选5.5版本，PHP环境必须选7.1，ftp看个人需要，phpMyAdmin为了便于管理数据库也是必装的，选了4.7版本但实际安装会变成4.4版本，估计是宝塔的Bug，强迫症的可以删除这项安装任务，待后面手动安装4.7版本的，安装过程时间较长需要耐心等待，时间有点长，可以去喝杯茶，吃个饭，泡个澡，回来后可能还没装好，哈哈哈。



下一步新建一个站点，这里需要你去申请一个域名，可以去申请一个免费域名freenom.com这里可以去申请，但是建议还是去买一个域名吧。数据库暂时先不创建，网站目录自己设定，这里简单设置为ssrpanel。

点击软件管理，找到php7.1设置，在禁用函数里面找一下proc开头的函数，找到后都删除。Debian环境下装了几次这两个函数都不在列表里面，Centos6系统安装宝塔后应该会有，如果实际操作删除成功就在php服务里面重启一下php。

然后需要在php7.1设置里面安装扩展fileinfo，如下图:

# 第二步：ssrpanel前端安装

返回到shell命令行模式，进入刚才建立的站点根目录

`cd /www/wwwroot/ssrpanel `  说明下这里的ssrpanel是你的站名，这个不是唯一的哦，牢记，你前面写的是什么就填什么！！！
下载ssrpanel源代码（如果没有系统默认没有安装git的话，先用`apt-get install git`命令安装一下）：

```
git clone https://github.com/ssrpanel/ssrpanel.git tmp && mv tmp/.git . && rm -rf tmp && git reset --hard
```

下一步先回到宝塔面板添加ssrpanel所需的数据库，数据库名和密码自行设定，需要注意的是访问权限改为所有人，这样后端才能有权限远程访问前端数据库

在宝塔面板的文件管理中找到`www/wwwroot/ssrpanel` sql目录下的db.sql，下载到本地



点数据库，找到前面创建的数据库，点管理进入`phpMyAdmin`，选Import，点浏览找到本地的db.sql文件，最后点Go导入数据库



点文件找到`/www/wwwroot/ssrpanel/config`目录下的database.php文件点编辑，打开编辑窗口找到mysql设置的地方修改成刚才创建的数据库名称、用户名和密码，修改完毕点保存后关闭





需要配置邮件发送的在同目录下找到mail.php文件，编辑设置SMTP信息，邮箱我觉得不用去设置了所以这里就不设置了。

OK，数据库配置好了回到Shell继续安装前端，依次输入以下命令：

```

php composer.phar install

cp .env.example .env

php artisan key:generate

chown -R www:www storage/

chmod -R 777 storage/

chmod -R 777 public/upload/

```

回到宝塔面板中，点击所建网站——设置，找到伪静态这里，填写如下规则后保存：

```

location / {
    try_files $uri $uri/ /index.php$is_args$args;
}
```


找到网站目录，把运行目录改为/public，保存



至此前端安装结束，打开浏览器访问域名。由于作者的更新，需要执行 php artisan upgradeUserPassword (默认用户名密码是admin/admin)：


登陆进去之后记得修改管理员密码和默认的SSR连接密码，然后在设置里面根据需要做相应的调整。

# 第三步:ssrpanel后端安装

SSH登陆服务器安装libsodium，让后端支持更多的加密方式（以下命令一个个输入）：

```

cd /root

apt-get install build-essential

wget https://github.com/jedisct1/libsodium/releases/download/1.0.15/libsodium-1.0.15.tar.gz

tar xf libsodium-1.0.15.tar.gz && cd libsodium-1.0.15

./configure && make -j2 && make install

echo /usr/local/lib > /etc/ld.so.conf.d/usr_local_lib.conf
```

ldconfig
安装shadowsocksr后端程序：

```
cd /root

git clone -b manyuser https://github.com/shadowsocksrr/shadowsocksr.git

cd shadowsocksr

./setup_cymysql.sh

./initcfg.sh
修改userapiconfig.py的接口为glzjinmod，修改完毕按Ctrl+X，输入Y回车退出并保存

nano userapiconfig.py


修改user-config.json，将connect_verbose_info的值改为1

nano user-config.json


修改usermysql.json，将数据库信息改为前面自己创建的数据库信息，记得修改node_id的值为1，这个值是要和前端面板的节点ID相对应的

nano usermysql.json
```


回到宝塔面板，在安全里面放行3306数据库端口和10000后面ssrpanel需要用到的端口，这里设置放行10000至10500的端口



返回shell运行下面命令测试后端看是否正常，出现如下信息说明数据库连接正常

python server.py


运行正常后可以按Ctrl+C退出，运行下面的命令放入后台运行，如果前后端在同一台服务器上的可以运行./logrun.sh，这样管理后台能看到日志分析：

./run.sh
运行说明, 有几种方式
python server.py 用于调错的
./run.sh 无日志后台运行
./logrun.sh 有日志后台运行
./stop.sh 停止运行
./tail.sh 在有日志后台运行的情况下显示运行信息

现在进入ssrpanel前端面板，在节点管理——节点分组列表——新增一个“所有可见”的分组名称，可见级别选“倔强青铜”，这里只是为了演示，分组名称和对应的级别都是可以根据自己需要设定的



然后在节点管理——节点列表——新增节点，根据自己需要和实际设置一下，所属分组选“所有可见”



至此正常情况下前后端都能正常运行了，可以新建一个账号或是注册一个账号进行测试。

为了运行更可靠，可以用Supervisor守护进程启动ssr，先安装supervisor

apt-get install supervisor -y
写入配置

nano /etc/supervisor/conf.d/ssr.conf
写入以下内容

[program:ssr]
command=python /root/shadowsocksr/server.py 
autorestart=true
autostart=true
user=root
重启Supervisor服务。

/etc/init.d/supervisor restart

重启ssr

supervisorctl restart ssr
查看Supervisor服务运行状态

supervisorctl status
如果遇到问题，可以检查日志

supervisorctl tail -f ssr stderr
如果使用supervisor进程守护，需要修改文件/etc/default/supervisor：

nano /etc/default/supervisor
添加一行

ulimit -n 1024000
『代码升级』
如果代码有更新可以进入网站根目录运行以下命令，更新前最好利用宝塔面板做好网站和数据库备份：

cd /www/wwwroot/ssrpanel
git pull
数据库升级可以用https://github.com/ssrpanel/ssrpanel/tree/master/sql/update里面的升级指令，打开phpMyAdmin，在ssrpanel数据库里面用SQL命令执行进行升级。以/sql/update/20171020.sql为例，在浏览器输入网址https://github.com/ssrpanel/ssrpanel/blob/master/sql/update/20171020.sql打开这个文件，将指令复制。随后打开宝塔面板数据库ssrpanel的phpMyAdmin管理，在SQL区域黏贴指令，点Go运行导入即可，如图：





以此类推，只要是在初始安装过后的数据库升级按文件日期先后用这个方式对数据库进行升级。

这篇教程其实也谈不上什么教程，只是整理一下便于自己今后使用而已。本篇教程全部基于bebian系统。centos以及乌班图系统的勿扰。如果你还有不会的，可以留言提出！

>转载：https://qukashing.com/?p=9