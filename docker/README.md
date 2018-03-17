
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

# Info - 信息

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

## 执行命令

* [`docker exec`](https://docs.docker.com/engine/reference/commandline/exec) 在容器中执行命令。

比如，进入正在运行的容器，在名为 foo 的容器中打开一个新的 shell 进程: `docker exec -it foo /bin/bash`.

# Images - 镜像

镜像是[docker 容器的模板](https://docs.docker.com/engine/understanding-docker/#how-does-a-docker-image-work)。

## 生命周期

* [`docker images`](https://docs.docker.com/engine/reference/commandline/images) 查看所有镜像。
* [`docker import`](https://docs.docker.com/engine/reference/commandline/import) 从压缩文件中创建镜像。
* [`docker build`](https://docs.docker.com/engine/reference/commandline/build) 从 Dockerfile 创建镜像。
* [`docker commit`](https://docs.docker.com/engine/reference/commandline/commit) 为容器创建镜像，如果容器正在运行则会临时暂停。
* [`docker rmi`](https://docs.docker.com/engine/reference/commandline/rmi) 删除镜像。
* [`docker load`](https://docs.docker.com/engine/reference/commandline/load) 通过 STDIN 从压缩包加载镜像，包括镜像和标签(images and tags) (0.7 起).
* [`docker save`](https://docs.docker.com/engine/reference/commandline/save) 通过 STDOUT 保存镜像到压缩包，包括所有的父层，标签和版本(parent layers, tags & versions) (0.7 起).

## 信息

* [`docker history`](https://docs.docker.com/engine/reference/commandline/history) 查看镜像历史记录。
* [`docker tag`](https://docs.docker.com/engine/reference/commandline/tag) 给镜像命名打标(tags) (本地或者仓库)。

## 清理

虽然你可以用 `docker rmi` 命令来删除指定的镜像，但是这里有个称为 [docker-gc](https://github.com/spotify/docker-gc) 的工具，它可以以一种安全的方式，清理掉那些不再被任何容器使用的镜像。

## 加载/保存镜像

从文件中加载镜像:
```
docker load < my_image.tar.gz
```
保存既有镜像:
```
docker save my_image:my_tag | gzip > my_image.tar.gz
```

## 导入/导出容器

从文件中将容器作为镜像导入:
```
cat my_container.tar.gz | docker import - my_image:my_tag
```

导出既有容器:
```
docker export my_container | gzip > my_container.tar.gz
```

## 加载被保存的镜像和导入作为镜像导出的容器之间的不同

通过 `load` 命令来加载镜像，会创建一个新的镜像，并继承原镜像的所有历史。
通过 `import` 将容器作为镜像导入，也会创建一个新的镜像，但并不包含原镜像的历史，因此生成的镜像会比使用加载方式生成的镜像要小。

# Networks - 网络

Docker 有[网络(networks)](https://docs.docker.com/engine/userguide/networking/)功能。我并不是很了解它，所以这是一个扩展本文的好地方。这里有篇笔记指出，这是一种可以不使用端口来达成 docker 容器间通信的好方法。详情查阅[通过网络来工作](https://docs.docker.com/engine/userguide/networking/work-with-networks/)。

## 生命周期

* [`docker network create`](https://docs.docker.com/engine/reference/commandline/network_create/)
* [`docker network rm`](https://docs.docker.com/engine/reference/commandline/network_rm/)

## 信息

* [`docker network ls`](https://docs.docker.com/engine/reference/commandline/network_ls/)
* [`docker network inspect`](https://docs.docker.com/engine/reference/commandline/network_inspect/)

## 链接

* [`docker network connect`](https://docs.docker.com/engine/reference/commandline/network_connect/)
* [`docker network disconnect`](https://docs.docker.com/engine/reference/commandline/network_disconnect/)

你可以为[容器指定 IP 地址](https://blog.jessfraz.com/post/ips-for-all-the-things/):

```
# 使用你自己的子网和网关创建一个桥接网络
docker network create --subnet 203.0.113.0/24 --gateway 203.0.113.254 iptastic

# 基于以上创建的网络，运行一个nginx容器并指定ip
$ docker run --rm -it --net iptastic --ip 203.0.113.2 nginx

# 在其他地方使用curl访问这个ip（假设这是一个公网ip）
$ curl 203.0.113.2
```

# Registry & Repository - 仓管中心和仓库

仓库(repository)是*被托管(hosted)*的已命名镜像(tagged images)集合，这组镜像用于构建容器文件系统。

仓管中心(registry)是一个*托管服务(host)* -- 一个服务，用于存储仓库和提供 HTTP API，以便[管理上传和下载仓库](https://docs.docker.com/engine/tutorials/dockerrepos/)。

Docker.com 把它自己的[索引](https://hub.docker.com/)托管到了它的仓管中心，那里有数量众多的仓库。不过话虽如此，这个仓管中心[并没有很好的验证镜像](https://titanous.com/posts/docker-insecurity)，所以如果你很担心安全问题的话，请尽量避免使用它。

* [`docker login`](https://docs.docker.com/engine/reference/commandline/login) 登入仓管中心。
* [`docker logout`](https://docs.docker.com/engine/reference/commandline/logout) 登出仓管中心。
* [`docker search`](https://docs.docker.com/engine/reference/commandline/search) 从仓管中心检索镜像。
* [`docker pull`](https://docs.docker.com/engine/reference/commandline/pull) 从仓管中心拉去镜像到本地。
* [`docker push`](https://docs.docker.com/engine/reference/commandline/push) 从本地推送镜像到仓管中心。

## 本地仓管中心

你可以创立一个本地的仓管中心，通过使用 [docker distribution](https://github.com/docker/distribution) 工程，细节请查看 [本地发布(local deploy)](https://github.com/docker/docker.github.io/blob/master/registry/deploying.md) 介绍。  

也可以参考 [邮件列表](https://groups.google.com/a/dockerproject.org/forum/#!forum/distribution)。

# Dockerfile

[配置文件](https://docs.docker.com/engine/reference/builder/)。当你执行 `docker build` 的时候会根据该配置文件设置 Docker 容器。远优于使用 `docker commit`。

下面是一些常用的编写 Dockerfile 的编辑器和语法高亮模块︰
* 如果你使用 [jEdit](http://jedit.org)，我为 [Dockerfile](https://github.com/wsargent/jedit-docker-mode) 做了个语法高亮模块。
* [Sublime Text 2](https://packagecontrol.io/packages/Dockerfile%20Syntax%20Highlighting)
* [Atom](https://atom.io/packages/language-docker)
* [Vim](https://github.com/ekalinin/Dockerfile.vim)
* [Emacs](https://github.com/spotify/dockerfile-mode)
* [TextMate](https://github.com/docker/docker/tree/master/contrib/syntax/textmate)
* 如果要找更全面的关于编辑器或者 IDE 的内容，请看 [当 Docker 遇上 IDE](https://domeide.github.io/)

## 指令

* [.dockerignore](https://docs.docker.com/engine/reference/builder/#dockerignore-file)
* [FROM](https://docs.docker.com/engine/reference/builder/#from) 为其他指令设置基础镜像(Base Image)。
* [MAINTAINER](https://docs.docker.com/engine/reference/builder/#maintainer) 为生成的镜像设置作者字段。
* [RUN](https://docs.docker.com/engine/reference/builder/#run) 在当前镜像的基础上生成一个新层并执行命令。
* [CMD](https://docs.docker.com/engine/reference/builder/#cmd) 设置容器默认执行命令。
* [EXPOSE](https://docs.docker.com/engine/reference/builder/#expose) 告知 Docker 容器在运行时所要监听的网络端口。注意：并没有实际上将端口设置为可访问。
* [ENV](https://docs.docker.com/engine/reference/builder/#env) 设置环境变量。
* [ADD](https://docs.docker.com/engine/reference/builder/#add) 将文件，文件夹或者远程文件复制到容器中。缓存无效。尽量用 `COPY` 代替 `ADD`。
* [COPY](https://docs.docker.com/engine/reference/builder/#copy) 将文件或文件夹复制到容器中。
* [ENTRYPOINT](https://docs.docker.com/engine/reference/builder/#entrypoint) 将一个容器设置为可执行。
* [VOLUME](https://docs.docker.com/engine/reference/builder/#volume) 为外部挂载卷标或其他容器设置挂载点(mount point)。
* [USER](https://docs.docker.com/engine/reference/builder/#user) 设置执行 RUN / CMD / ENTRYPOINT 命令的用户名。
* [WORKDIR](https://docs.docker.com/engine/reference/builder/#workdir) 设置工作目录。
* [ARG](https://docs.docker.com/engine/reference/builder/#arg) 定义编译时(build-time)变量。
* [ONBUILD](https://docs.docker.com/engine/reference/builder/#onbuild) 添加触发指令，当该镜像被作为其他镜像的基础镜像时该指令会被触发。
* [STOPSIGNAL](https://docs.docker.com/engine/reference/builder/#stopsignal) 设置通过系统向容器发出退出指令。
* [LABEL](https://docs.docker.com/engine/userguide/labels-custom-metadata/) 将键值对元数据(key/value metadata)应用到你的镜像，容器，或者守护进程。 


# Layers - 层

Docker 的版本化文件系统是基于层的。就像[git的提交或文件变更系统](https://docs.docker.com/engine/userguide/storagedriver/imagesandcontainers/)一样。

注意: 如果你使用 [aufs](https://en.wikipedia.org/wiki/Aufs) 作为你的文件系统，当删除一个容器的时候，Docker 并不一定能成功删除的文件卷标！更多详细信息请参阅 [PR 8484](https://github.com/docker/docker/pull/8484)。

# Links - 链接

链接(Links)[通过 TCP/IP 端口](https://docs.docker.com/userguide/dockerlinks/)实现了 Docker 容器之间的通讯。[链接到 Redis](https://docs.docker.com/examples/running_redis_service/) 和 [Atlassian](https://blogs.atlassian.com/2013/11/docker-all-the-things-at-atlassian-automation-and-wiring/) 是两个可用的例子。你还可以[通过 hostname 关联链接](https://docs.docker.com/engine/userguide/networking/default_network/dockerlinks/#/updating-the-etchosts-file)。

注意: 如果你希望容器之间**只**通过链接进行通讯，在启动 docker 守护进程的时候请添加参数 `-icc=false` 来禁用内部进程通讯。

如果你有一个名为 CONTAINER 的容器(通过 `docker run --name CONTAINER` 指定) 并且在 Dockerfile 中，它的端口暴露为:

```
EXPOSE 1337
```

然后，我们创建另外一个名为 LINKED 的容器:

```
docker run -d --link CONTAINER:ALIAS --name LINKED user/wordpress
```

然后 CONTAINER 的端口和别名将会以如下的环境变量出现在 LINKED 中:

```
$ALIAS_PORT_1337_TCP_PORT
$ALIAS_PORT_1337_TCP_ADDR
```

之后你就可以通过这种方式来链接它了。

要删除链接，通过命令 `docker rm --link`。

通常，docker 服务之间的链接，是"服务发现"的一个子集，如果你打算在生产中大规模使用 Docker，这将是一个很大的问题。请参阅[The Docker Ecosystem: Service Discovery and Distributed Configuration Stores](https://www.digitalocean.com/community/tutorials/the-docker-ecosystem-service-discovery-and-distributed-configuration-stores)获得更多细节。

# Volumes - 卷标 

Docker 的卷标(volumes)是一个[free-floating 文件系统](https://docs.docker.com/engine/tutorials/dockervolumes/)。它们不应该链接到特定的容器上。好的做法是如果可能，应当把卷标挂载到[纯数据容器(data-only containers)](https://medium.com/@ramangupta/why-docker-data-containers-are-good-589b3c6c749e)上。

## 生命周期

* [`docker volume create`](https://docs.docker.com/engine/reference/commandline/volume_create/)
* [`docker volume rm`](https://docs.docker.com/engine/reference/commandline/volume_rm/)

## 信息

* [`docker volume ls`](https://docs.docker.com/engine/reference/commandline/volume_ls/)
* [`docker volume inspect`](https://docs.docker.com/engine/reference/commandline/volume_inspect/)

卷标在不能使用链接(只有 TCP/IP )的情况下非常有用。例如，如果你有两个 docker 实例需要通讯并在文件系统上留下记录。

你可以一次性将其挂载到多个 docker 容器上，通过 `docker run --volumes-from`。

因为卷标是独立的文件系统，它们通常被用于存储各容器之间的瞬时状态。也就是说，你可以配置一个无状态临时容器，关掉之后，当你有第二个这种临时容器实例的时候，你可以从上一次保存的状态继续执行。

查看[卷标进阶](http://crosbymichael.com/advanced-docker-volumes.html)来获取更多细节。Container42 [非常有用](http://container42.com/2014/11/03/docker-indepth-volumes/)。

你可以[将宿主 MacOS 的文件夹映射为 docker 卷标](https://docs.docker.com/engine/tutorials/dockervolumes/#mount-a-host-directory-as-a-data-volume):

```
docker run -v /Users/wsargent/myapp/src:/src
```

你也可以用远程 NFS 卷标，如果你觉得你[有足够勇气](https://docs.docker.com/engine/tutorials/dockervolumes/#/mount-a-shared-storage-volume-as-a-data-volume)。

可还可以考虑运行一个纯数据容器，像[这里](http://container42.com/2013/12/16/persistent-volumes-with-docker-container-as-volume-pattern/)所说的那样，提供可移植数据。

# Exposing ports - 暴露端口

通过宿主容器暴露输入端口是相当[繁琐，但有效](https://docs.docker.com/engine/reference/run/#expose-incoming-ports)的。

这种方式可以将容器端口映射到宿主端口上(只使用本地主机(localhost)接口)，通过使用 `-p`:

```
docker run -p 127.0.0.1:$HOSTPORT:$CONTAINERPORT --name CONTAINER -t someimage
```

你可以告诉 Docker 容器在运行时监听指定的网络端口，通过使用 [EXPOSE](https://docs.docker.com/engine/reference/builder/#expose):

```
EXPOSE <CONTAINERPORT>
```

但是注意 EXPOSE 并不会暴露端口，你需要用参数 `-p` 。比如说你要在 localhost 上暴露容器的端口:

```
iptables -t nat -A DOCKER -p tcp --dport <LOCALHOSTPORT> -j DNAT --to-destination <CONTAINERIP>:<PORT>
```

如果你是在 Virtualbox 中运行 Docker，那么你需要转发端口(forward the port)，使用 [forwarded_port](https://docs.vagrantup.com/v2/networking/forwarded_ports.html)。它可以用于在 Vagrantfile 上配置暴露端口段，这样你就可以动态的映射它们了:

```
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  ...

  (49000..49900).each do |port|
    config.vm.network :forwarded_port, :host => port, :guest => port
  end

  ...
end
```

如果你忘记你将什么端口映射到宿主容器上的话，使用 `docker port` 来查看它:

```
docker port CONTAINER $CONTAINERPORT
```

# 安全(Security)

这节准备讨论一些关于 Docker 安全性的问题。[安全](https://docs.docker.com/articles/security/)这章讲述了更多细节。

首先第一件事: Docker 是有 root 权限的。如果你在 `docker` 组，那么你就有[ root 权限](http://reventlov.com/advisories/using-the-docker-command-to-root-the-host)。如果你暴露了 docker unix socket 给容器，意味着你赋予了容器[宿主的 root 权限](https://www.lvh.io/posts/dont-expose-the-docker-socket-not-even-to-a-container.html)。Docker 不应该是你唯一的防御措施。

## 安全提示

为了最大的安全性，你应该会考虑在虚拟机上运行 Docker 。这是直接从 Docker 安全团队拿来的资料 -- [slides](http://www.slideshare.net/jpetazzo/linux-containers-lxc-docker-and-security) / [notes](http://www.projectatomic.io/blog/2014/08/is-it-safe-a-look-at-docker-and-security-from-linuxcon/)。然后，可以使用 AppArmor / seccomp / SELinux / grsec 之类的来[限制容器的权限](http://linux-audit.com/docker-security-best-practices-for-your-vessel-and-containers/)。更多细节，请查阅 [Docker 1.10 security features](https://blog.docker.com/2016/02/docker-engine-1-10-security/)。

Docker 镜像 id 属于[敏感信息](https://medium.com/@quayio/your-docker-image-ids-are-secrets-and-its-time-you-treated-them-that-way-f55e9f14c1a4) 所以它不应该向外界公开。你应该把他们当成密码来对待。

参考 [Docker Security Cheat Sheet](https://github.com/konstruktoid/Docker/blob/master/Security/CheatSheet.adoc)中 - 作者是 [Thomas Sjögren](https://github.com/konstruktoid) - 关于如何提高容器安全的建议。

下载[docker 安全测试脚本](https://github.com/docker/docker-bench-security)，下载[白皮书](https://blog.docker.com/2015/05/understanding-docker-security-and-best-practices/) 以及订阅[邮件列表](https://www.docker.com/docker-security) (不幸的是 Docker 并没有独立的邮件列表，只有 dev / user)。

你应该远离那些使用编译版本 grsecurity / pax 的不稳定内核，比如 [Alpine Linux](https://en.wikipedia.org/wiki/Alpine_Linux)。如果在产品中用了 grsecurity ，那么你应该考虑使用有[商业支持](https://grsecurity.net/business_support.php)的[稳定版本](https://grsecurity.net/announce.php)，就像你对待 RedHat 那样。它要 $200 每月，对于你的运维预算来说不值一提。

从 docker 1.11 开始，你可以轻松的限制在容器中可用的进程数，以防止 fork bombs。 这要求 linux 内核 >= 4.3 并且要在内核配置中打开 CGROUP_PIDS=y 。

```
docker run --pids-limit=64
```

同时，从 docker 1.11 开始，你也可以限制进程有再获取新权限的能力了。该功能是 linux 内核从 version 3.5 开始就拥有的。你可以从[这篇博客](http://www.projectatomic.io/blog/2016/03/no-new-privs-docker/)中阅读到更多关于这方面的内容。

```
docker run --security-opt=no-new-privileges
```

参考 [Docker Security Cheat Sheet](http://container-solutions.com/content/uploads/2015/06/15.06.15_DockerCheatSheet_A2.pdf) (它是个 PDF 版本，搞得非常难用，所以拷贝出来了) 的 [容器解決方案](http://container-solutions.com/is-docker-safe-for-production/):

关闭内部进程通讯:

```
docker -d --icc=false --iptables
```

设置容器为只读:

```
docker run --read-only
```

通过 hashsum 来验证卷标:

```
docker pull debian@sha256:a25306f3850e1bd44541976aa7b5fd0a29be
```

设置卷标为只读:

```
docker run -v $(pwd)/secrets:/secrets:ro debian
```

在 Dockerfile 中定义并运行一个用户，避免在容器中以 root 身份操作:

```
RUN groupadd -r user && useradd -r -g user user
USER user
```

## 用户命名空间(User Namespaces)

还可以通过使用 [user namespaces](https://s3hh.wordpress.com/2013/07/19/creating-and-using-containers-without-privilege/) -- 这已经是 1.10 内建功能了，但默认情况下是不启用的。

要在 Ubuntu 15.10 中启用用户命名空间 ("remap the userns")，请[跟着这篇博客的例子](https://raesene.github.io/blog/2016/02/04/Docker-User-Namespaces/)来做。

# Tips

来源:

* [15 Docker Tips in 5 minutes](http://sssslide.com/speakerdeck.com/bmorearty/15-docker-tips-in-5-minutes)

## 最后的 Ids

```
alias dl='docker ps -l -q'
docker run ubuntu echo hello world
docker commit `dl` helloworld
```

## 带命令行的提交 (需要 Dockerfile)

```
docker commit -run='{"Cmd":["postgres", "-too -many -opts"]}' `dl` postgres
```

## 获取 IP 地址

```
docker inspect `dl` | grep IPAddress | cut -d '"' -f 4
```

或者安装 [jq](https://stedolan.github.io/jq/):

```
docker inspect `dl` | jq -r '.[0].NetworkSettings.IPAddress'
```

或者用[go 模板](https://docs.docker.com/engine/reference/commandline/inspect)

```
docker inspect -f '{{ .NetworkSettings.IPAddress }}' <container_name>
```

## 获取端口映射

```
docker inspect -f '{{range $p, $conf := .NetworkSettings.Ports}} {{$p}} -> {{(index $conf 0).HostPort}} {{end}}' <containername>
```

## 通过正则获取容器

```
for i in $(docker ps -a | grep "REGEXP_PATTERN" | cut -f1 -d" "); do echo $i; done`
```

## 获取环境设定

```
docker run --rm ubuntu env
```

## 强迫关闭正在运行的容器

```
docker kill $(docker ps -q)
```

## 删除旧容器

```
docker ps -a | grep 'weeks ago' | awk '{print $1}' | xargs docker rm
```

## 删除停止容器

```
docker rm -v `docker ps -a -q -f status=exited`
```

## 删除 dangling 镜像

```
docker rmi $(docker images -q -f dangling=true)
```

## 删除所有镜像

```
docker rmi $(docker images -q)
```

## 删除 dangling 卷标

Docker 1.9 开始:

```
docker volume rm $(docker volume ls -q -f dangling=true)
```

1.9.0 中，过滤器 `dangling=false` 居然 _没_ 用 - 它会被忽略然后列出所有的卷标。

## 查看镜像依赖

```
docker images -viz | dot -Tpng -o docker.png
```

## Docker 容器瘦身  [Intercity 博客](http://bit.ly/1Wwo61N)

- 在当前运行层(RUN layer)清理 APT

这应当和其他 apt 命令在同一层中完成。
否则，前面的层将会保持原有信息，而你的镜像则依旧臃肿。

```
RUN {apt commands} \
  && apt-get clean \  
  && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
```

- 压缩镜像
```
ID=$(docker run -d image-name /bin/bash)
docker export $ID | docker import – flat-image-name
```

- 备份
```
ID=$(docker run -d image-name /bin/bash)
(docker export $ID | gzip -c > image.tgz)
gzip -dc image.tgz | docker import - flat-image-name
```

## 监视运行中容器的系统资源利用率

检查某个单独容器的 CPU, 内存, 和 网络 i/o 使用情况，你可以:

```
docker stats <container>
```

按 id 列出所有的容器:

```
docker stats $(docker ps -q)
```

按名称列出所有容器:

```
docker stats $(docker ps --format '{{.Names}}')
```

按指定镜像名称列出所有容器:

```
docker ps -a -f ancestor=ubuntu
```