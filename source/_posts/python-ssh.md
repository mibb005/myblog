---
title: python ssh 模块使用
date: 2019-09-05 16:09:45
tags:
 - python
 - ssh
---


# 介绍

paramiko是用python语言写的一个模块，遵循SSH2协议，支持以加密和认证的方式，进行远程服务器的连接。paramiko支持Linux, Solaris, BSD, MacOS X, Windows等平台通过SSH从一个平台连接到另外一个平台。利用该模块，可以方便的进行ssh连接和sftp协议进行sftp文件传输。

# 安装

paramiko模块依赖PyCrypto模块，而PyCrypto需要GCC库编译，不过一般发行版的源里带有该模块。这里以ubuntu为例，直接借助以下命令可以直接完成安装：


```python
# pip install paramiko 
```

# 连接

使用paramiko模块有两种连接方式，一种是通过paramiko.SSHClient()函数，另外一种是通过paramiko.Transport()函数。

## 方法一：


```python
import paramiko

ssh = paramiko.SSHClient()


#这行代码的作用是允许连接不在know_hosts文件中的主机。
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())

# 使用私钥连接
# ssh.connect('10.120.48.109', port, '用户名',key_filename='私钥')

# 使用密码连接：
ssh.connect("192.168.2.247",22,"mbb", "243063036.mm")

stdin, stdout, stderr = ssh.exec_command('ls')

print(stdout.readlines())

ssh.close()
```

    ['conf\n', 'examples.desktop\n', 'frp\n', 'gateone\n', 'get_pip.py\n', 'git\n', 'gitlab\n', 'index2.py\n', 'index.py\n', 'Lite\n', 'logs\n', 'mongo\n', 'mynginx\n', 'myregistry\n', 'mysql\n', 'n2n\n', 'nbs\n', 'nginx\n', 'node\n', 'nohup.out\n', 'notebooks\n', 'opencv\n', 'opencv_python\n', 'packages-microsoft-prod.deb\n', 'portainer\n', 'privoxy-3.0.26-stable\n', 'shadowsocks-all.log\n', 'shadowsocks-all.sh\n', 'shadowsocks_python_qr.png\n', 'srs\n', 'srsnew\n', 'temp\n', 'tensorflow\n', 'test\n', 'txt\n', 'vpn-gen.env\n', 'vpnsetup.sh\n', 'web\n', 'webapp\n', 'wget-log\n', 'www\n']
    

## 方法二：


```python
import paramiko

t = paramiko.Transport(("192.168.2.247",22))

t.connect(username = "mbb", password = "243063036.mm")

# t.connect(username = "用户名", password = "口令", hostkey="密钥")

print(t)

```

    <paramiko.Transport at 0xa0ec10f0 (cipher aes128-ctr, 128 bits) (active; 0 open channel(s))>
    

# sftp

文件传输


```python
import paramiko

ssh = paramiko.SSHClient()


#这行代码的作用是允许连接不在know_hosts文件中的主机。
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())

# 使用私钥连接
# ssh.connect('10.120.48.109', port, '用户名',key_filename='私钥')

# 使用密码连接：
ssh.connect("192.168.2.247",22,"mbb", "243063036.mm")

# sftp = paramiko.SFTPClient.from_transport(t)

sftp = paramiko.SFTPClient.from_transport(ssh.get_transport())

sftp = ssh.open_sftp()
```

1.文件上传


```python
sftp.put('index.py', 'index.py')
```




    <SFTPAttributes: [ size=787 uid=1008 gid=1009 mode=0o100664 atime=1567670573 mtime=1567670585 ]>



2.文件下载


```python
sftp.get('get_pip.py', 'get_pip.py')
```

3.获取文件列表


```python
print(sftp.listdir())
```

    ['vpn-gen.env', 'conf', '.Xauthority', 'logs', 'n2n', 'temp', '.ssh', '.bashrc', 'tensorflow', 'www', 'srs', '.pip', 'examples.desktop', 'index2.py', '.cache', 'wget-log', '.profile', '.python_history', 'packages-microsoft-prod.deb', 'gitlab', 'myregistry', '.keras', '.bash_logout', 'node', 'shadowsocks-all.log', 'gateone', 'txt', 'nohup.out', 'frp', 'srsnew', 'nginx', 'opencv_python', '.vim_mru_files', 'portainer', '.bash_history', '.vscode-server', 'opencv', 'privoxy-3.0.26-stable', 'shadowsocks_python_qr.png', 'nbs', 'mysql', 'test', 'shadowsocks-all.sh', 'vpnsetup.sh', '.viminfo', '.vim', 'mongo', '.docker', 'Lite', '.local', 'get_pip.py', 'webapp', 'mynginx', '.vimrc', 'git', 'notebooks', 'web', 'index.py']
    


```python
t.close()
```

官方文档：https://paramiko-docs.readthedocs.io/en/2.6/
