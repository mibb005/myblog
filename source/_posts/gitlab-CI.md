---
title: GitLab持续集成环境 windows版
date: 2019-05-13 15:14:46
tags: gitlab
---

# 搭建环境

## GitLab-Runner下载
···
https://gitlab-ci-multi-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-ci-multi-runner-windows-amd64.exe
···

##  GitLab查看项目的Runners

> 点击一个项目->Settings->Runners, 得到Url地址①和registration token②

## 构建GitLab-Runner服务

```
gitlab-ci-multi-runner-windows-amd64.exe register
根据提示，填写
1) GitLab->Runners的Url地址①
2) GitLab->Runners的registration token②
3) runner名称，这个随便写
4) 分支名，master
5) 协议方式，shell
gitlab-ci-multi-runner-windows-amd64.exe install

gitlab-ci-multi-runner-windows-amd64.exe start
```
## 修改协议config.toml文件

> 注册成功后，在文件夹中找到config.toml，在[[runners]]后面添加shell = "powershell"节点

## 构建.gitlab-ci.yml脚本

```
stages:

- build

job:

 stage: build

 script:

 - echo "Restoring NuGet Packages..." 

 #- C:\test\nuget.exe restore "ConsoleApplication1.sln" 

 - echo "Solution Build..."

 - C:\Windows\Microsoft.NET\Framework64\v4.0.30319\msbuild.exe /p:Configuration=Debug /p:Platform="Any CPU" /consoleloggerparameters:ErrorsOnly /maxcpucount /nologo /property:Configuration=Release /verbosity:quiet /p:VisualStudioVersion=10.0 "test1.sln"

 tags:
 
 except:

  - tags

```

