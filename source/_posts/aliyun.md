---
title: 阿里云盘挂载
date: 2021-09-28 13:53:34
tags: 
---

# 准备工作

## 获取refreshToken

https://www.aliyundrive.com/sign/ 登录网页版  按f12打开控制台—application—local storage—token，即可查看refresh token


## 安装webdav工具

https://github.com/zxbu/webdav-aliyundriver.git
下载运行文件或源码编译

## 运行程序

```
java -jar webdav.jar --aliyundrive.refresh-token="your refreshToken"
```

## 参数说明

```
--aliyundrive.refresh-token
    阿里云盘的refreshToken，获取方式见下文
--server.port
    非必填，服务器端口号，默认为8080
--aliyundrive.auth.enable=true
    是否开启WebDav账户验证，默认开启
--aliyundrive.auth.user-name=admin
    WebDav账户，默认admin
--aliyundrive.auth.password=admin
    WebDav密码，默认admin
```

## 容器运行

```
docker run -d --name=webdav-aliyundriver --restart=always -p 8080:8080  -v /etc/localtime:/etc/localtime -v /etc/aliyun-driver/:/etc/aliyun-driver/ -e TZ="Asia/Shanghai" -e ALIYUNDRIVE_REFRESH_TOKEN="your refreshToken" -e ALIYUNDRIVE_AUTH_PASSWORD="admin" -e JAVA_OPTS="-Xmx1g" zx5253/webdav-aliyundriver
```

# 安装Rclone进行挂载

## 安装rclone

```
curl https://rclone.org/install.sh | sudo bash
```

## 配置rclone

```
rclone config
[root@localhost yum.repos.d]# rclone config
2021/08/12 16:28:35 NOTICE: Config file "/root/.config/rclone/rclone.conf" not found - using defaults
No remotes found - make a new one
n) New remote
s) Set configuration password
q) Quit config
n/s/q> n
name> aliyunwebdav
Type of storage to configure.
Enter a string value. Press Enter for the default ("").
Choose a number from below, or type in your own value
 1 / 1Fichier
   \ "fichier"
 2 / Alias for an existing remote
   \ "alias"
 3 / Amazon Drive
   \ "amazon cloud drive"
 4 / Amazon S3 Compliant Storage Providers including AWS, Alibaba, Ceph, Digital Ocean, Dreamhost, IBM COS, Minio, SeaweedFS, and Tencent COS
   \ "s3"
 5 / Backblaze B2
   \ "b2"
 6 / Box
   \ "box"
 7 / Cache a remote
   \ "cache"
 8 / Citrix Sharefile
   \ "sharefile"
 9 / Compress a remote
   \ "compress"
10 / Dropbox
   \ "dropbox"
11 / Encrypt/Decrypt a remote
   \ "crypt"
12 / Enterprise File Fabric
   \ "filefabric"
13 / FTP Connection
   \ "ftp"
14 / Google Cloud Storage (this is not Google Drive)
   \ "google cloud storage"
15 / Google Drive
   \ "drive"
16 / Google Photos
   \ "google photos"
17 / Hadoop distributed file system
   \ "hdfs"
18 / Hubic
   \ "hubic"
19 / In memory object storage system.
   \ "memory"
20 / Jottacloud
   \ "jottacloud"
21 / Koofr
   \ "koofr"
22 / Local Disk
   \ "local"
23 / Mail.ru Cloud
   \ "mailru"
24 / Mega
   \ "mega"
25 / Microsoft Azure Blob Storage
   \ "azureblob"
26 / Microsoft OneDrive
   \ "onedrive"
27 / OpenDrive
   \ "opendrive"
28 / OpenStack Swift (Rackspace Cloud Files, Memset Memstore, OVH)
   \ "swift"
29 / Pcloud
   \ "pcloud"
30 / Put.io
   \ "putio"
31 / QingCloud Object Storage
   \ "qingstor"
32 / SSH/SFTP Connection
   \ "sftp"
33 / Sugarsync
   \ "sugarsync"
34 / Tardigrade Decentralized Cloud Storage
   \ "tardigrade"
35 / Transparently chunk/split large files
   \ "chunker"
36 / Union merges the contents of several upstream fs
   \ "union"
37 / Uptobox
   \ "uptobox"
38 / Webdav
   \ "webdav"
39 / Yandex Disk
   \ "yandex"
40 / Zoho
   \ "zoho"
41 / http Connection
   \ "http"
42 / premiumize.me
   \ "premiumizeme"
43 / seafile
   \ "seafile"
Storage> 38
URL of http host to connect to
Enter a string value. Press Enter for the default ("").
Choose a number from below, or type in your own value
 1 / Connect to example.com
   \ "https://example.com"
url> http://127.0.0.1:8080
Name of the Webdav site/service/software you are using
Enter a string value. Press Enter for the default ("").
Choose a number from below, or type in your own value
 1 / Nextcloud
   \ "nextcloud"
 2 / Owncloud
   \ "owncloud"
 3 / Sharepoint Online, authenticated by Microsoft account.
   \ "sharepoint"
 4 / Sharepoint with NTLM authentication. Usually self-hosted or on-premises.
   \ "sharepoint-ntlm"
 5 / Other site/service or software
   \ "other"
vendor> 5
User name. In case NTLM authentication is used, the username should be in the format 'Domain\User'.
Enter a string value. Press Enter for the default ("").
user> admin
Password.
y) Yes type in my own password
g) Generate random password
n) No leave this optional password blank (default)
y/g/n> y
Enter the password:
password:
Confirm the password:
password:
Bearer token instead of user/pass (e.g. a Macaroon)
Enter a string value. Press Enter for the default ("").
bearer_token> 
Edit advanced config?
y) Yes
n) No (default)
y/n> 
--------------------
[aliyunwebdav]
type = webdav
url = http://127.0.0.1:8080
vendor = other
user = admin
pass = *** ENCRYPTED ***
--------------------
y) Yes this is OK (default)
e) Edit this remote
d) Delete this remote
y/e/d> 
Current remotes:

Name                 Type
====                 ====
aliyunwebdav         webdav

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q>  q
```

## 挂载到本地linux

```
#新建本地文件夹,位置可以自己选
mkdir /data/aliyunwebdav
#挂载
rclone mount DriveName:Folder LocalFolder --cache-dir /tmp --allow-other --vfs-cache-mode writes --allow-non-empty

DriverName是你在配置rclone的时候设置的名字,Folder没有需要求的话填/即可,LocalFolder是你本地挂载的地址,/tmp比较特殊,上传时缓存目录,其他类型挂载一般时不需要这个参数的,默认/tmp地址即可,除非你的系统特殊
```

## 设置开机自启

```
#将后面修改成你上面手动运行命令中，除了rclone的全部参数
command="mount DriveName:Folder LocalFolder --cache-dir /tmp --allow-other --vfs-cache-mode writes --allow-non-empty"
#以下是一整条命令，一起复制到SSH客户端运行
cat > /etc/systemd/system/rclone.service <<EOF
[Unit]
Description=Rclone
After=network-online.target

[Service]
Type=simple
ExecStart=$(command -v rclone) ${command}
Restart=on-abort
User=root

[Install]
WantedBy=default.target
EOF
```

## 启动

```
重启：systemctl restart rclone
停止：systemctl stop rclone
状态：systemctl status rclone
```

## windwos

新建两个文件，分别为`rclone.bat`和`rclone.vbs`

1. `rclone.bat`中写入上述挂载命令：
```
rclone mount onedrive:/ E: --cache-dir D:\Onedrive --vfs-cache-mode writes &
```
2. rclone.vbs设置开机自动调用cmd运行rclone.bat文件并退出cmd，写入如下代码：
```
CreateObject("WScript.Shell").Run "cmd /c D:/rclone.bat",0
```
将rclone.bat文件放到D盘（或其他的盘）目录下，我放在了D盘，所以在rclone.vbs中该文件的路径就是D:/rclone.bat

将rclone.vbs文件放到windows系统启动项目录下，在文件夹的路径框中输入
`%USERPROFILE%\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup`
即可进入启动项目录