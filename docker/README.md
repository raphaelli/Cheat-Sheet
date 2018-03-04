---
title: Docker Cheat Sheet
date: 2018-03-04 23:47:08
tags: 
 - Docker

---

# What's Docker? - 什么是Docker

Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的Linux机器上，也可以实现虚拟化，容器是完全使用沙箱机制，相互之间不会有任何接口。

>Docker 相关概念 - [Docker 初次见面](https://ns96.com/2018/01/01/docker-start/)

# Installation - 安装

获得最新的docker安装包。
```
$ sudo curl -sSL https://get.docker.com/ | sh 
```
安装时间较长，Why not drink a cup of coffer？

确认Docker是否安装成功。
```
$ sudo docker run hello-world
```

这个命令会下载一个测试用的镜像并启动一个容器运行它。

# Permissions - 权限相关
## 非Root用户授权
>实际的开发运维情况下，一般极少使用Root权限，所以Docker提供了一个权限组，把当前用户加入到Docker用户组中。

一共需要三条指令:
```
$ sudo groupadd docker
$ sudo gpasswd -a ${USER} docker
$ sudo service docker restart
```
三条指令的意思分别是：

- 添加docker用户组，一般会默认创建，提示已存在
- 将用户添加到docker用户组
- 重启docker服务


# Containers - 容器
粗暴来说，可以简单的把一个 Containers 容器理解为一个虚拟机，一个完全的虚拟化的环境。虽然实际上，容器(Container)之于虚拟机(Virtual Machine)就如同线程和进程。

## Lifecycle - 生命周期
- [`docker create`](https://docs.docker.com/engine/reference/commandline/create) 创建一个容器但是不启动。
- [`docker rename`](https://docs.docker.com/engine/reference/commandline/rename/) 重命名容器。
- [`docker run`](https://docs.docker.com/engine/reference/commandline/run) 创建并启动一个容器
- [`docker rm`](https://docs.docker.com/engine/reference/commandline/rm)  删除容器
- [`docker update`](https://docs.docker.com/engine/reference/commandline/update/) 更新容器 

>`docker run --rm `可以用于创建一个在容器停止之后删除的临时容器，方便测试。  
`docker run -d IMAGE [COMMAND] [AGR…] ` 创建守护式容器


`docker run -v $HOSTDIR:$DOCKERDIR` 用于映射宿主(host)的一个文件夹到 docker 容器。具体参考 [Volumes](https://github.com/wsargent/docker-cheat-sheet/#volumes)。

## Operation - 操作
- [`docker start`](https://docs.docker.com/engine/reference/commandline/start) 启动容器
- [`docker stop`](https://docs.docker.com/engine/reference/commandline/stop) 停止运行中的容器
- [`docker restart`](https://docs.docker.com/engine/reference/commandline/restart) 重启容器
- [`docker pause`](https://docs.docker.com/engine/reference/commandline/pause/) 暂停容器
- [`docker unpause`](https://docs.docker.com/engine/reference/commandline/unpause/) 恢复已暂停容器
- [`docker kill`](https://docs.docker.com/engine/reference/commandline/kill) 停止容器,区别于stop等待，kill直接停止
- [`docker attach`](https://docs.docker.com/engine/reference/commandline/attach) 附加指令到容器
- [`docker top`](https://docs.docker.com/engine/reference/commandline/top/) 查看运行中容器的进程
- [`docker exec`](https://docs.docker.com/engine/reference/commandline/exec/) 在运行的容器中启动新的进程

>守护式容器切出到后台，使用快捷键 `Ctrl+P` + `Ctrl+Q` 切出，并使用 attach 命令回到容器。