---
layout: post
title: docker安装及常用命令说明
categories: Docker
description: docker在程序开发部署过程中提供了很多便捷，是程序员必备的工具之一
keywords: docker
---
  docker安装可以遵循[官网](https://docs.docker.com/engine/installation/)的说明一步一步进行，注意的是linux下对内核有一定的要求,目前新推出两个版本EE和CE，EE是企业版增加一些收费定制功能，这里作为个人用户主要安装CE版本即可。

## 常见centos7下安装

1. 监测是否安装过docker，如果安装进行卸载
```shell
$ sudo yum remove docker docker-common docker-selinux docker-engine
```
2. 安装docker CE
安装依赖包
```shell
$ sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```
配置存储仓库地址
```shell
$ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```
是否开启测试版本
```shell
$ sudo yum-config-manager --enable docker-ce-edge
$ sudo yum-config-manager --enable docker-ce-testing
$ sudo yum-config-manager --disable docker-ce-edge
```
更新yum包
```shell
$ sudo yum makecache fast
```
安装docker-ce
```shell
$ sudo yum install docker-ce
```
查询docker版本
```shell
$ yum list docker-ce.x86_64  --showduplicates | sort -r
docker-ce.x86_64 17.06.0.el7 docker-ce-stable
```
指定版本安装
```shell
$ sudo yum install docker-ce-<VERSION>
```
测试容器hello-world
```shell
$ sudo docker run hello-world
```
3. 通过安装包直接安装
* 下载centos7下[docker安装包](https://download.docker.com/linux/centos/7/x86_64/stable/Packages/)
* `$ sudo yum install /path/to/package.rpm`
* `$ sudo systemctl start docker`
* `$ sudo docker run hello-world`
* 卸载`$ sudo yum remove docker-ce`，`$ sudo rm -rf /var/lib/docker`

## docker常用命令

* 容器生命周期管理 —  `docker [run|start|stop|restart|kill|rm|pause|unpause]`
* 容器操作运维     —  `docker [ps|inspect|top|attach|events|logs|wait|export|port]`
* 容器rootfs命令   —  `docker [commit|cp|diff]`
* 镜像仓库         —  `docker [login|pull|push|search]`
* 本地镜像管理     —  `docker [images|rmi|tag|build|history|save|import]`
* 其他命令         —  `docker [info|version]`

1. 列出机器上的镜像（images）
```shell
# docker images 
REPOSITORY               TAG             IMAGE ID        CREATED         VIRTUAL SIZE
ubuntu                   14.10           2185fd50e2ca    13 days ago     236.9 MB
…
```
其中我们可以根据`REPOSITORY`来判断这个镜像是来自哪个服务器，如果没有 / 则表示官方镜像，类似于`username/repos_name`表示`Github`的个人公共库，类似于`regsistory.example.com:5000/repos_name`则表示的是私服。
`IMAGE ID`列其实是缩写，要显示完整则带上`--no-trunc`选项
2. 在docker index中搜索image（search）,Usage: `docker search TERM`
```shell
# docker search seanlo
NAME                DESCRIPTION           STARS     OFFICIAL   AUTOMATED
seanloook/centos6   sean's docker repos         0
```
搜索的范围是官方镜像和所有个人公共镜像。NAME列的 / 后面是仓库的名字。
3. image操作
```shell
# Usage: docker pull [OPTIONS] NAME[:TAG]
# docker pull centos
```
上面的命令需要注意，在docker v1.2版本以前，会下载官方镜像的centos仓库里的所有镜像，而从v.13开始官方文档里的说明变了：`will pull the centos:latest image, its intermediate layers and any aliases of the same id`，也就是只会下载tag为latest的镜像（以及同一images id的其他tag）。
也可以明确指定具体的镜像：
```shell
# docker pull centos:centos6
```
当然也可以从某个人的公共仓库（包括自己是私人仓库）拉取，形如`docker pull username/repository<:tag_name>`
```shell
# docker pull seanlook/centos:centos6
```
如果你没有网络，或者从其他私服获取镜像，形如`docker pull registry.domain.com:5000/repos:<tag_name>`
```shell
# docker pull dl.dockerpool.com:5000/mongo:latest
```
删除镜像
```shell
# docker rmi <image_id/image_name ...>
```
4. 推送一个image或repository到registry（push）
与上面的pull对应，可以推送到Docker Hub的Public、Private以及私服，但不能推送到Top Level Repository。
```shell
# docker push seanlook/mongo
# docker push registry.tp-link.net:5000/mongo:2014-10-27
```
registry.tp-link.NET也可以写成IP，172.29.88.222。
在repository不存在的情况下，命令行下push上去的会为我们创建为私有库，然而通过浏览器创建的默认为公共库。
5. 通过image镜像创建相应的容器container
`docker run`后面跟启动参数很多，如：
* `-a stdin` 指定标准输入输出内容类型，可选 STDIN/STDOUT / STDERR 三项
* `-d` 后台运行容器，并返回容器ID
* `-i` 以交互模式运行容器，通常与 -t 同时使用
* `-t` 为容器重新分配一个伪输入终端，通常与 -i 同时使用
* `--name="nginx-lb"` 为容器指定一个名称
* `--dns 8.8.8.8` 指定容器使用的DNS服务器，默认和宿主一致
* `--dns-search example.com` 指定容器DNS搜索域名，默认和宿主一致
* `--link` 容器间通信
* `-v` 本地路径:容器路径，数据卷挂载，解决掉电或者容器重启产生的数据丢失
* `-w` 进入容器后当前路径
* `-e` 配置容器中的环境变量
* `--add-host master:10.10.243.192` 添加主机IP映射
* `-p 127.0.0.1:4000:5000/udp` ，ip不加即用默认主机ip,4000为宿主机端口，5000为容器中端口，udp/tcp为传输方式，添加端口映射
* `-P` 随机分配端口号
6. container操作
* `$ docker stop Name/ID`  停止容器
* `$ docker start Name/ID` 启动容器
* `$ docker kill Name/ID` 停止容器进程
* `$ docker restart Name/ID` 重启容器
* `$ docker rm -f Name/ID` 删除容器，其中-f 为强制性
* `$ docker ps` 默认显示当前正在运行中的container
* `$ COMMNAD` 经常通过ps来找到CONTAINER_ID
* `$ docker ps -a` 查看包括已经停止的所有容器
* `$ docker ps -l` 显示最新启动的一个容器（包括已停止的）
7. 将一个容器固化一个新的镜像
* 当我们在制作自己的镜像的时候，会在container中安装一些工具、修改配置，如果不做commit保存起来，那么container停止以后再启动，这些更改就消失了。
* `docker commit <container> [repo:tag]`
* 注意：后面的repo:tag可选，只能提交正在运行的container，即通过docker ps可以看见的容器

8. `dockerfile`创建新的镜像

* `docker build [OPTIONS] PATH | URL | -`
* 上面的PATH或URL中的文件被称作上下文，build image的过程会先把这些文件传送到docker的服务端来进行的。
* 如果PATH直接就是一个单独的Dockerfile文件则可以不需要上下文；如果URL是一个Git仓库地址，那么创建image的过程中会自动git clone一份到本机的临时目录，它就成为了本次build的上下文。无论指定的PATH是什么，Dockerfile是至关重要的，请参考Dockerfile Reference。
* 请看下面的例子。
```shell
FROM seanlook/nginx:bash_vim
EXPOSE 80
ENTRYPOINT /usr/sbin/nginx -c /etc/nginx/nginx.conf && /bin/bash
Sending build context to Docker daemon 73.45 MB
Sending build context to Docker daemon 
Step 0 : FROM seanlook/nginx:bash_vim
 ---> aa8516fa0bb7
Step 1 : EXPOSE 80
 ---> Using cache
 ---> fece07e2b515
Step 2 : ENTRYPOINT /usr/sbin/nginx -c /etc/nginx/nginx.conf && /bin/bash
 ---> Running in e08963fd5afb
 ---> d9bbd13f5066
Removing intermediate container e08963fd5afb
Successfully built d9bbd13f5066
```