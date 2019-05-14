---
title: docker.core
date: 2019-05-14 14:33:19
tags:
---

# 拉取microsoft/dotnet镜像

```
ocker pull microsoft/dotnet
```

# 运行镜像

使用docker run <image>可以启动镜像，通过指定参数-it以交互模式（进入容器内部）启动。依次执行以下命令：

```
//启动一个dotnet镜像
$ docker run -it microsoft/dotnet
//创建项目名为HelloDocker.Web的.NET Core MVC项目
dotnet new mvc -n HelloDocker.Web
//进入HelloDocker.Web文件夹
cd HelloDocker.Web
//启动.NET Core MVC项目
dotnet run
```
键盘按住Ctrl+C即可关闭应用，输入exit即可退出当前容器。

# 挂载宿主机项目到容器中

在启动Docker镜像时，Docker允许我们通过使用-v参数挂载宿主机的文件到容器的指定目录下。换句话说，就相当于宿主机共享指定文件供容器去访问

```
// 命令中的`\`结合`Enter`键构成换行符，允许我们换行输入一个长命令。
$ docker run -it \
-v $HOME/demo/HelloDocker.Web:/app \
microsoft/dotnet:latest
```
上面的命令就是把$HOME/demo/HelloDocker.Web文件夹下的文件挂载到容器的\app目录下。

# Dockerfile

```
//确保进入我们创建的MVC项目目录中去
$ cd $HOME/demo/HelloDocker.Web
//使用touch命令创建Dockerfile
$ touch Dockerfile
//使用vi命令编辑Dockerfile
vi Dockerfile
```
进入VI编辑界面后，复制以下代码，使用shift + Ins命令即可粘贴。然后按ESE退出编辑模式，按shift + :，输入wq即可保存并退出编辑界面。

```
FROM microsoft/dotnet:latest
WORKDIR /app
COPY . /app
RUN dotnet restore
EXPOSE 5000
ENV ASPNETCORE_URLS http://*:5000
ENTRYPOINT ["dotnet","run"]
```

上面的命令我依次解释一下：

使用FROM指定容器使用的镜像
使用WORKDIR指定工作目录
使用COPY指令，复制当前目录（其中.即代表当前目录）到容器中的/app目录下
使用RUN命令指定容器中执行的命令
使用EXPOSE指定容器暴露的端口号
使用ENV指定环境参数，上面用来告诉.NETCore项目在所有网络接口上监听5000端口
使用ENTRYPOINT制定容器的入口点

Dockerfile就绪，我们就可以将我们当前项目打包成镜像以分发部署。
使用docker build -t <name> <path>指令打包镜像：

```
 docker build -t hellodocker.web .
```

# 推送镜像到仓库

到Docker Hub注册个账号，然后我们把本地打包的镜像放到自己账号下的仓库

注册完毕后，执行docker login：

再执行docker push：

镜像按<user>/<repo>格式来命名

```
docker tag hellodocker.web shengjie/hellodocker.web:v1
```

