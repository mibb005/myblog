---
title: winsw说明
date: 2019-11-12 14:34:48
tags:
 - windows
---

# 一、WinSW介绍

官方介绍如下：

WinSW is an executable binary, which can be used to wrap and manage a custom process as a Windows service.

现实生活中，我们使用windows系统的电脑的时候，可能会遇到这么一种情况：想把一些应用程序添加为开机启动项。对于有图形界面的应用程序，一般不存在问题。但是如果想运行命令行应用程序，就不是那么方便了。一种笨办法就是写个bat，放到启动文件夹里，就可以开机启动了。开机之后，你就会发现，这样会一直显示着一个CMD窗口，而且这个窗口不能关，关了程序就停了。

其实Windows系统自带后台程序管理的功能，也就是我们经常用到的服务。但是Windows的服务只有程序的开发者在写程序的时候引用到这个功能，我们才能利用服务来控制程序的启动和关闭。对于一般的命令行程序来说，没办法利用服务。

今天要介绍的WinSW，它就是一个可以将Windows上的任何一个程序注册为服务的工具。同样也可以进行卸载该服务。

# 二、WinSW使用

## 1、工具下载

这个工具可以在GitHub上【https://github.com/kohsuke/winsw/releases】进行下载。当然，在GitHub上也可以获取到工具的源码【https://github.com/kohsuke/winsw】。

下载完之后最好把文件改成一个比较短小的名字，例如：WinSW.exe 方便后面输入命令时使用。

## 2、修改配置文件

我们需要编写一个和程序同名的XML文件作为WinSW的配置文件。大体的格式如下：

这里只是简单介绍使用WinSW的流程，配置项实际也不止那么几个，在第三部分有关于配置文件的详细解释。

## 3、注册服务

配置文件编写完之后，将配置文件与WinSW.exe放在同一目录中。注意对应WinSW.exe的配置文件名称应该是WinSW.xml。此时，WinSW.exe、WinSW.xml以及你的应用程序应该都是在同一目录中。然后用管理员权限打开一个命令提示符窗口，cd进入到应用程序所在目录，可以通过输入下面的命令来进行控制应用程序对应的服务：

winsw install 安装服务

winsw uninstall 卸载服务

winsw start 开启服务

winsw stop 停止服务

winsw restart 重新启动服务

winsw status 检查服务的当前状态

安装服务命令执行后，如果返回值为0，就表示服务已经安装成功。此时在windows服务的窗口，就能看到你刚才安装的服务了。

# 三、WinSW配置说明（翻译xmlConfigFile.md）

## 1、配置文件的文件结构

配置文件是一个XML格式文件，其根元素必须是“service”节点。

## 2、配置文件中的环境变量表达式

XML配置文件中，可以包含形如“%Name%”的环境变量表达式。如果发现这种情况，将会自动被变量的实际值替换掉。如果引用未定义的环境变量，则不会进行替换。

此外，服务包装器本身定义了一个名为“BASE”的环境变量，它指向一个包含重命名为“winsw.exe”的文件的目录。这在引用同一目录中的其他文件的时候是很有用的。由于这本身就是一个环境变量，所以这个值也可以从服务包装器启动的子进程中访问。

## 3、配置项说明

（1）id

指定在Windows系统内部使用的识别服务的ID。在系统中安装的所有服务中，这必须是唯一的，它应该完全由字母数字字符组成。

（2）name

服务的简短名称，它可以包含空格和其他字符。尽量简短，就像“id”一样，在系统的所有服务名称中，也要保持唯一。

（3）description

该服务可读描述。当选中该服务时，它将显示在Windows服务管理器中。

（4）executable

该元素指定要启动的可执行文件。它可以是绝对路径，也可以指定可执行文件的名称，然后从环境变量“PATH”中搜索（需要注意的是，服务经常在不同的用户账户中运行，因此它可能需要有不同于你设置在环境变量Path中的路径）。

（5）startmode

该元素指定Windows服务的启动模式。它可以是下列值之一：开机、系统、自动或手动。有关详细信息，请参阅MSDN【https://msdn.microsoft.com/en-us/library/aa384896%28v=vs.85%29.aspx】。默认值是“Automatic”。

（6）delayedAutoStart

这个布尔选项允许在定义“自动”启动模式时延时启动。关于延时启动模式，可参阅【https://blogs.technet.microsoft.com/askperf/2008/02/02/ws2008-startup-processes-and-delayed-automatic-start】。

请注意，延时启动模式在早于Windows 7和Windows Server 2008的操作系统中可能失效。在这种情况下，Windows服务安装可能会失败。

（7）depend

指定该服务所依赖的其他服务的id。当服务“X”依赖于服务“Y”时，“X”只能在“Y”运行后运行。可以使用多个元素来指定多个依赖项。

（8）logging

关于服务进程的日志以及错误信息，有单独的一个配置说明文档【WinSW Logging and Error Reporting】，咱们下次再详细说。

（9）argument

该元素指定要传递给可执行文件的参数。

如果有必要，Winsw会给每一个参数外加引号（“”），所以为了避免双重引号，尽量不要在参数中使用引号。

为了向后兼容，可以使用“arguments”来指定单个元素中的整个命令行。

（10）stopargument/stopexecutable

请求停止服务时，winsw通过调用终止进程的API函数来立即终结服务。然而，如果存在“stopargument”元素，winsw将通过使用”stopargument“作为参数，来启动“executable“元素（或者是”stopexecutable“元素）中配置的进程，来代替调用终止进程的API函数。期望通过这种方式来优雅的关闭服务进程。

然后，Winsw将等待两个进程自行退出，然后向Windows报告该服务已经终止。

当你使用“stopargument”元素时，你必须使用“startargument”元素代替“argument”元素。参见下面的完整示例：

catalina.sh

jpda

run

catalina.sh

stop

注意，元素的名称是“startargument”，而不是“startarguments”。因此，要指定多个参数，你必须指定多个元素。

（11）stoptimeout

当服务被要求停止时，winsw首先尝试调用GenerateConsoleCtrlEvent 方法（类似于Ctrl+C），然后等待长达15秒的时间，让进程自行退出。

如果这样做了，进程关闭还是失败了（或者如果进程没有控制台），

然后winsw会调用终止进程的API函数来立即终止服务。

这个可选元素允许您改变这个“15秒”的值，这样您就可以控制winsw等待服务进程自行关闭的时间。

如何指定时间期限，可参考下面的“onfailure”元素的设置：

10sec

（12）env

如果有必要，可以使用这个可多次指定的可选元素，为子进程设置环境变量。其语法是:

（13）interactive

如果指定了这个可选的元素，那么该服务将被允许与桌面交互，比如显示一个新的窗口和对话框。

如果你的程序需要GUI，那么设置如下：

请注意，自从引入UAC（Windows Vista及之后的版本）以来，服务不再被允许与桌面交互。

在这些操作系统中，所有这一切都是为了允许用户切换到一个单独的窗口来与服务交互。

（14）beeponshutdown

这个可选元素是为了在服务关闭时发出简单的音调。

这个特性应该只用于调试，因为一些操作系统和硬件不支持这种功能。

（15）download

这个可选元素可以多次指定，让服务包装器从URL检索资源，并将其作为文件放置在本地。这个操作是在服务启动时运行的，在“executable”指定的应用程序启动之前。

对于需要身份验证的服务器，必须根据认证的类型指定一些参数。只有基本身份验证需要额外的子参数。支持的身份验证类型是:

none：默认类型，不能指定

sspi：微软认证，包括Kerberos、NTLM等。

basic：基本身份认证，子参数包括：

1）user=“UserName”

2）password=“Password”

3）unsecureAuth=“true”：default=“false”

当传输协议是未加密的HTTP数据传输时，参数“unsecureAuth”才生效。这是一个安全漏洞，因为凭证是用明文发送的！对于SSPI认证，这是不相关的，因为身份验证令牌是加密的。

对于使用HTTPS传输协议的目标服务器这是必须的，因为服务器发出的CA证书是客户端信任的。这通常是服务器在Internet上的情况。当一个组织在内网中使用自发布的CA时，情况可能不是这样。在这种情况下，有必要将CA导入到Windows客户端的证书MMC。可以看一下微软的说明（https://technet.microsoft.com/en-us/librar/cc754841.aspx）。自发布CA必须被导入到计算机的受信任的根证书颁发机构。

默认情况下，如果操作失败（例如，无效的下载地址），“download”命令也不会在服务启动时失败。为了在这种情况下强制下载失败，可以指定“failOnError”布尔属性。示例如下：

这是开发自动更新服务的另一个有用的构建块。

（16）onfailure

这个可选的可重复元素控制了winsw启动的进程失败后的行为（例如，执行带有非零退出码的退出操作）。

例如，上面的配置使服务在第一次故障后10秒内重新启动，在第二次失败后20秒重新启动，如果服务再次失败，Windows将重新启动。

每个元素都包含一个强制的“action”属性，它控制着Windows SCM将要做什么，以及可选的“延迟”属性，它控制着延迟，直到采取行动为止。“action”属性的值包含以下几种：

restart：重启服务

reboot：重启系统

none：什么也不做，就让服务那么停着

延迟属性的可能后缀是sec/secs/min/mins/hours/day/days。如果未设置，延迟属性默认为0。

如果服务持续失败的次数超出了配置的“onfailure”的数量，那么最后的操作将会被重复。因此，如果只是想要自动重启服务，只需指定一个“onfailure”元素即可：

（17）resetfailure

这个可选元素控制了Windows SCM重新设置故障计数的时间。

例如，如果指定``，并且您的服务持续运行超过一个小时，那么故障计数将重置为零。

这会影响故障操作的行为（见上面的“onfailure”）。

换句话说，这是你认为服务已经成功运行的持续时间。

默认值为1天。

（18）Service account

有可能需要指定服务运行所需要的的useraccount（和密码）。要做到这一点，可以按照下面的方法指定一个“serviceaccount”元素：

YOURDOMAIN

useraccount

Pa55w0rd

true

“”是可选的。如果设置为“true”，则会自动给上面列出的用户设置“Allow Log On As A Service”的权限。

可以使用（组）管理服务帐户 附加'$'到帐户名称，并删除'password'元素：

YOURDOMAIN

gmsa_account$

true

（19）Working directory

有些服务需要使用指定的工作目录运行。

要做到这一点，要指定一个像这样的“workingdirectory”元素：

C:\application

（20）priority

可选地指定服务流程的调度优先级（类似于Unix）。可能的值有“idle”、“belownormal”、“normal”、“abovenormal”、“high”、“realtime”（大小写不敏感）。

idle

指定高于正常值的优先级会产生意想不到的后果。请参阅MSDN文章ProcessPriorityClass Enumeration来了解详细信息。

这个特性主要是为了在较低的优先级上启动一个进程，以避免干扰计算机的交互使用。

（21）stopparentprocessfirst

可选地指定服务关闭的顺序。

如果“true”，父进程首先关闭。

当主进程是一个控制台时，这是很有用的，它可以响应Ctrl+C命令，并优雅地关闭子进程。

true
