---
layout:     post
title:      "docker学习记录"
subtitle:   "整理之前的笔记"
date:       2018-11-19
author:     "Qian"
header-img: "img/in-post/post-bg-docker.png"
tags:
    - docker
---

<!-- TOC -->

- [初步了解](#初步了解)
    - [docker官网介绍](#docker官网介绍)
    - [什么是容器？——————A standardized unit of software](#什么是容器a-standardized-unit-of-software)
- [Docker安装及初步使用](#docker安装及初步使用)
    - [使用存储库安装：](#使用存储库安装)
    - [初步使用](#初步使用)
        - [基础运行命令](#基础运行命令)
        - [容器使用](#容器使用)
        - [镜像使用](#镜像使用)
    - [认识Dockerfile](#认识dockerfile)
        - [说明](#说明)
        - [Dockerfile的命令关键字](#dockerfile的命令关键字)
- [docker技术](#docker技术)
    - [Registry](#registry)
    - [网络](#网络)
    - [存储](#存储)

<!-- /TOC -->

# 初步了解

## docker官网介绍
Docker是一家支持容器发展的公司，也是唯一一家能解决混合云中每个应用的容器平台提供商，当今的企业面临数字化转型的压力，但在面临要求合理组织日益多样化的云、数据中心和应用程序体系结构的情况下，还受到现有应用程序和基础架构的制约。 Docker实现了应用程序和基础架构与开发人员和IT运营商之间的真正独立性，从而发挥其潜力，并创建更好的协作和创新模式。

## 什么是容器？——————A standardized unit of software
容器是一个轻量级的、独立的、可执行的软件,包括运行它所需的一切条件:代码,运行时,系统工具,系统库,设置。可用于Linux和Windows应用程序,无论哪种环境，软件都运行的是相同的容器。容器能够隔离软件环境,例如隔离开发环境和环境之间的差异,有助于减少团队在相同的基础设施运行不同软件之间的冲突。

![](/img/in-post/post-docker/docker1.png)

容器的三个特点是轻量级、标准化、高安全性。

与虚拟机相比，拥有类似的资源隔离分配的优势，但具体功能并不一样，因为容器虚拟化的是操作系统而不是硬件，因此更加方便和高效。

一个image的结构分为两部分，以Linux操作系统而言，有内核空间（kenel）和用户空间（rootfs，包括根目录下的/dev、/bin等），对于base image（ubuntu、centos等）而言，底层之间使用的host主机的kernel，因此只需要提供用户空间即可。

![](/img/in-post/post-docker/docker2.png)

# Docker安装及初步使用

[docker官网安装](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/#install-docker-ce-1)

## 使用存储库安装：

```
# 建立存储库

#安装以下软件包，以允许通过HTTPS使用存储库
$ sudo apt-get update
$ sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
#添加docker官网GPG密钥
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
#使用以下命令设置稳定存储库
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

# 安装docker-ce

#更新 apt 包
$ sudo apt-get update
#安装最新版本的docker ce或安装某一具体版本的docker ce
#安装最新版本
$ sudo apt-get install docker-ce
#安装具体版本，一般用与生产
$ sudo apt-get install docker-ce=<VERSION>
#注：使用以下命令查看docker ce可用版本
$ apt-cache madison docker-ce
#验证是否正确安装
#运行hello-world镜像
$ sudo docker run hello-world
#注：如果出现“Unable to find image 'hell0-world:latest' locally”，可使用以下命令下载镜像，再重新运行hello-world
$ sudo docker run -it ubuntu bash
#配置docker启动(一般安装后系统已默认)
$ sudo systemctl enable docker
#设置用户使用docker权限
$ sudo chmod u+s /usr/bin/docker
```

## 初步使用
### 基础运行命令

`docker run ubuntu:15.10 /bin/echo "Hello world"`

`docker执行 运行的镜像 容器中执行的命令`

交互式容器参数：

+ -t:在新容器内指定一个伪终端或终端。

+ -i:允许你对容器内的标准输入 (STDIN) 进行交互。
		
+ e.g.：docker run -i -t ubuntu:15.10 /bin/bash

后台启动容器参数：

+ -d：在后台运行容器并输出容器id
		
+ e.g.：docker run -d ubuntu:15.10 /bin/sh -c "while true; do echo hello world; sleep 1; done"

容器查看与关闭：

```
#查看正在运行的容器：
$ docker ps
#查看所有的容器：
$ docker ps -a
#查看容器内的标准输出：
$ docker logs dockerID
#停止容器
& docker stop dockerID
```

### 容器使用

运行web应用：

```
-P：将容器内部使用的网络端口映射到我们使用的主机上。
e.g.：docker run -d -P training/webapp python app.py
-p：绑定指定端口。
e.g.：docker run -d -p 5000:5000 training/webapp python app.py
e.g.：docker run -d -p 127.0.0.1:5001:5002 training/webapp python app.py
```

其他命令：
```
#查看容器端口
$ docker port dockerID
#查看容器的进程
$ docker top dockerID
#检查容器的进程
$ docker inspect dockerID
#删除容器
$ docker rm dockerID
#停止容器进程
$ docker stop dockerID
#删除所有容器
$ docker rm $(docker ps -a -q)
#停止所有容器进程
$ docker stop $(docker ps -a -q)
#删除所有未使用容器的容器层
$ docker volume prune
#容器已后台启动后，进入终端
$ docker exec dockerID /bin/bash
```

### 镜像使用

基础命令：
```
#列出镜像
$ docker images
#获取镜像
$ docker pull dockerNAME
#查找镜像
$ docker search dockerNAME
```

镜像更新：
```
e.g.：docker commit -m="has update" -a="runoob" e218edb10161 runoob/ubuntu:v2
-m:提交的描述信息
-a:指定镜像作者
e218edb10161：容器ID
runoob/ubuntu:v2:指定要创建的目标镜像名
```
构建镜像：

第一步：创建一个Dockerfile文件

第二步：构建镜像：
```
e.g.：docker build -t runoob/centos:6.7 ./create_image/
-t ：指定要创建的目标镜像名
./create_image/ ：Dockerfile 文件所在目录
```
第三步：查看镜像，并用新镜像创建容器
```
$ docker images
$ docker run -t -i runoob/centos:6.7  /bin/bash
```

docker配置代理（有些镜像来自于google hub，如kubernetes所用镜像）：

对于这种情况，一种解决办法是在dockerhub中找到对应的所需镜像版本，从dockerhub pull，或者配置代理

```
# 在/etc/systemd/system/创建docker.service.d文件夹
$ sudo mkdir -p /etc/systemd/system/docker.service.d

# 在/etc/systemd/system/docker.service.d/文件夹内创建http-proxy.conf，并编辑
$ sudo touch /etc/systemd/system/docker.service.d/http-proxy.conf
# cat <<EOF >>/etc/systemd/system/docker.service.d/http-proxy.conf
> [Service]
> Environment="HTTP_PROXY=http://ip:port/" "NO_PROXY=localhost,127.0.0.1,docker-registry.somecorporation.com[私有仓库]"
> EOF

# 重启
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```

## 认识Dockerfile
### 说明

在构建镜像过程中，Dockerfile先获取一个基础镜像，并在这之上不断通过运行文件内的命令添加“容器层”，最终得到有实际可用环境的镜像。

当镜像启动成为容器时，只有容器层可写：

1. 添加文件时，在容器层添加；
2. 读取文件时，从上往下在各层寻找，一旦找到，打开并读入内容；
3. 修改文件时，从上往下在各层寻找，一旦找到，复制到容器层，进行修改；
4. 删除文件时，从上往下在各层寻找，一旦找到，在容器层记录删除操作。

![](/img/in-post/post-docker/docker3.png)


### Dockerfile的命令关键字

```
FROM			基于哪个镜像
RUN				安装软件用
MAINTAINER		镜像创建者
CMD				container启动时执行的命令
ENTRYPOINT		container启动时执行的命令
USER			使用哪个用户跑container
EXPOSE			container内部服务开启的端口
ENV				用来设置环境变量
VOLUME			可以将本地文件夹或者其他container的文件夹挂载到container中
WORKDIR			切换目录用，可以多次切换(相当于cd命令)
COPY/ADD	    添加本地的目录或者文件到镜像
```

# docker技术

## Registry
保存和分发镜像的最直接方法就是使用 Docker Hub。Docker Hub 是 Docker 公司维护的公共 Registry。用户可以将自己的镜像保存到 Docker Hub 免费的 repository 中。如果不希望别人访问自己的镜像，也可以购买私有 repository。

下面介绍如何用 Docker Hub 存取我们的镜像。

1. 首先得在 Docker Hub 上注册一个账号。我的Docker Hub账号：liqian950624
2. 在 Docker Host 上登录。docker login -u liqian950624
3. 修改镜像的 repository 使之与 Docker Hub 账号匹配。Docker Hub 为了区分不同用户的同名镜像，镜像的 registry 中要包含用户名，完整格式为：[username]/xxx:tag
我们通过 docker tag 命令重命名镜像。

    docker tag httpd liqian950624/httpd:lq
4. 通过 docker push 将镜像上传到 Docker Hub。 
Docker 会上传镜像的每一层。因为 liqian950624/httpd:v1 这个镜像实际上跟官方的 httpd 镜像一模一样，Docker Hub 上已经有了全部的镜像层，所以真正上传的数据很少。同样的，如果我们的镜像是基于 base 镜像的，也只有新增加的镜像层会被上传。如果想上传同一 repository 中所有镜像，省略 tag 部分就可以了。

    docker push liqian950624/httpd:lq

5. 登录 https://hub.docker.com，在Public Repository 中就可以看到上传的镜像。

Docker Hub 虽然非常方便，但还是有些限制，比如：

1. 需要 internet 连接，而且下载和上传速度慢。
2. 上传到 Docker Hub 的镜像任何人都能够访问，虽然可以用私有 repository，但不是免费的。
3. 安全原因很多组织不允许将镜像放到外网。

解决方案就是搭建本地的 Registry。

启动registry容器

    docker run -d -p 5000:5000 -v /myregistry:/var/lib/registry registry:2

-v 将容器 /var/lib/registry 目录映射到 Host 的 /myregistry，用于存放镜像数据。

通过docker tag重命名镜像，使之与registry匹配

    docker tag liqian950624/httpd:lq registry.example.net:5000/liqian950624/httpd:lq

repository 的完整格式为：[registry-host]:[port]/[username]/xxx:[tag]，只有 Docker Hub 上的镜像可以省略 [registry-host]:[port] 。

通过push上传镜像

## 网络

![](/img/in-post/post-docker/docker4.jpg)

docker自动创建了none, host, bridge三个网络，docker0为bridge，可以通过命令brctl show查看网络接口，以及bridge name。

可以通过bridge驱动创建类似默认的 bridge 网络，可以通过--subnet和--gateway指定子网和网关

    docker network create --driver bridge my_net
    docker network create --driver bridge --subnet 172.22.16.0/24 --gateway 172.22.16.1 my_net2
    #创建bridge后，可以指定bridge或指定ip来启动容器
    docker run -it --network=my_net2 busybox
    docker run -it --network=my_net2 --ip 172.16.16.8 busybox

## 存储
Docker为容器提供了两种存放数据的资源：
1. 由storage driver管理的镜像层和容器层
2. Data Volume

分层结构使镜像和容器的创建、共享以及分发变得非常高效，而这些都要归功于 Docker storage driver。正是 storage driver 实现了多层数据的堆叠并为用户提供一个单一的合并之后的统一视图。

Ubuntu 用的 AUFS，底层文件系统是 extfs，各层数据存放在 /var/lib/docker/aufs。

对于某些容器，直接将数据放在由 storage driver 维护的层中是很好的选择，比如那些无状态的应用。无状态意味着容器没有需要持久化的数据，随时可以从镜像直接创建。但对于另一类应用这种方式就不合适了，它们有持久化数据的需求，容器启动时需要加载已有的数据，容器销毁时希望保留产生的新数据，也就是说，这类容器是有状态的。

Data Volume 本质上是 Docker Host 文件系统中的目录或文件，能够直接被 mount 到容器的文件系统中。Data Volume 有以下特点：
1. Data Volume 是目录或文件，而非没有格式化的磁盘（块设备）。
2. 容器可以读写 volume 中的数据。
3. volume 数据可以被永久的保存，即使使用它的容器已经销毁。

Data Volume 分为两种方式，一种为bind mount，是将host上已存在的目录文件mount到容器，通过-v的形式：`<host path>:<container path>`;另一种方式是将data打包到镜像中，启动一个volume容器（专门用来存放data静态文件的容器），再将volume容器提供给其他容器，再容器迁移过程中，数据可以跟随一起迁移。