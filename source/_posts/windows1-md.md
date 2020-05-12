---
title: windows包管理工具 Chocolatey
date: 2020-05-12 19:54:43
tags: windows
---

# 安装

右键开始菜单，选择以管理员身份运行PowerShell，然后粘贴以下指令：

```
Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
```

# 用法

choco install 软件名