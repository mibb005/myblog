---
title: vs code 通过ssh 远程连接 linux
date: 2019-08-08 14:09:31
tags:
 - ssh
 - vscode
---

# vscode 安装 `Remote Development` 插件

# 配置 Remote-ssh

  配置文件一般放在用户目录下的 .ssh

```
Host AliServer
	HostName 192.168.2.247    
	User     mbb
	IdentityFile C:\Users\Tmac\.ssh\id_rsa
```

# 配置windows下的ssh

cd 到 .ssh

执行 ssh-keygen 生成秘钥

# linux下配置ssh

将windows生成的公钥文件 id_rsa.pub 复制到 用户目录下 .ssh

执行  `cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys`

