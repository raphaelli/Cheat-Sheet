
>访问博客[Docker Cheat Sheet](https://ns96.com/2018/03/04/docker-cheat-sheet/) 带目录模式查看！

# What's Docker? - 什么是Docker
![logo](img/logo.jpg)

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

>如果你想整合容器到[宿主进程管理(host process manager)](https://docs.docker.com/engine/admin/host_integration/)，那么以 `-r=false` 启动守护进程(daemon)然后使用 `docker start -a`。

守护式容器切出到后台，使用快捷键 `Ctrl+P` + `Ctrl+Q` 切出，并使用 attach 命令回到容器。

如果你想通过宿主暴露容器的端口(ports)，请看[暴露端口](#exposing-ports)一节。

故障 docker 实例的重启策略在[这里](http://container42.com/2014/09/30/docker-restart-policies/)。


## 容器限制

### CPU 限制

你可以限制 CPU，包括使用所有 CPU 的百分比，或者使用特定内核数。

比如，你可以设置 [`cpu-shares`](https://docs.docker.com/engine/reference/run/#/cpu-share-constraint) 。这个设置看起来有点奇怪 -- 1024 的意思是 100% CPU，因此如果你希望容器使用全体 CPU 内核的 50%，应将其设置为 512。更多信息，请查阅 https://goldmann.pl/blog/2014/09/11/resource-management-in-docker/#_cpu :

```
docker run -ti --c 512 agileek/cpuset-test
```

你可以只对某些 CPU 内核使用 [`cpuset-cpus`](https://docs.docker.com/engine/reference/run/#/cpuset-constraint)]。请参阅 https://agileek.github.io/docker/2014/08/06/docker-cpuset/ 获取更多细节以及一些不错的视频:

```
docker run -ti --cpuset-cpus=0,4,6 agileek/cpuset-test
```

注意，Docker 在容器内仍然可以看到所有的 CPU -- 虽然它只是用了其中一部分。请查阅 https://github.com/docker/docker/issues/20770 获取更多细节。

### 内存限制

你同样可以在 Docker 设置[内存限制](https://docs.docker.com/engine/reference/run/#/user-memory-constraints) :

```
docker run -it -m 300M ubuntu:14.04 /bin/bash
```

### 能力(Capabilities)

Linux 的 capability 可以通过使用 `cap-add` 和 `cap-drop` 设置。请参阅 https://docs.docker.com/engine/reference/run/#/runtime-privilege-and-linux-capabilities 获取更多细节。这有助于提高安全性。

如需要挂载基于 FUSE 文件系统，你需要同时结合 --cap-add 和 --device 使用:

```
docker run --rm -it --cap-add SYS_ADMIN --device /dev/fuse sshfs
```

授予对单个设备访问权限:

```
docker run -it --device=/dev/ttyUSB0 debian bash
```

授予所有设备访问权限:

```
docker run -it --privileged -v /dev/bus/usb:/dev/bus/usb debian bash
```

有关容器特权的更多详情请参考[这里](https://docs.docker.com/engine/reference/run/#/runtime-privilege-and-linux-capabilities)

## Info - 信息

- [`docker ps`](https://docs.docker.com/engine/reference/commandline/ps) 查看运行中的所有容器。
- [`docker logs`](https://docs.docker.com/engine/reference/commandline/logs) 从容器中获取日志。(你也可以使用自定义日志驱动，不过在 1.10 中，它只支持 `json-file` 和 `journald`)
- [`docker inspect`](https://docs.docker.com/engine/reference/commandline/inspect) 查看某个容器的所有信息(包括 IP 地址)。
- [`docker events`](https://docs.docker.com/engine/reference/commandline/events) 从容器中获取事件(events)。
- [`docker port`](https://docs.docker.com/engine/reference/commandline/port) 查看容器的公开端口。
- [`docker top`](https://docs.docker.com/engine/reference/commandline/top) 查看容器中活动进程。
- [`docker stats`](https://docs.docker.com/engine/reference/commandline/stats) 查看容器的资源使用情况统计信息。
- [`docker diff`](https://docs.docker.com/engine/reference/commandline/diff) 查看容器的 FS 中有变化文件信息。

常用指令：

* `docker ps -a` 查看所有容器，包括正在运行的和已停止的。

* `docker stats --all` 显示正在运行的容器列表 

## Import&Export - 导入&导出

* [`docker cp`](https://docs.docker.com/engine/reference/commandline/cp) 在容器和本地文件系统之间复制文件或文件夹。

* [`docker export`](https://docs.docker.com/engine/reference/commandline/export) 将容器的文件系统切换为压缩包(tarball archive stream)输出到 STDOUT。

