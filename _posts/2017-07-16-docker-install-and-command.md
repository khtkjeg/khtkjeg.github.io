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
```
$ sudo yum remove docker docker-common docker-selinux docker-engine
```
2. 安装docker CE
安装依赖包
```
$ sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```
配置存储仓库地址
```
$ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```
是否开启测试版本
```
$ sudo yum-config-manager --enable docker-ce-edge
$ sudo yum-config-manager --enable docker-ce-testing
$ sudo yum-config-manager --disable docker-ce-edge
```
更新yum包
```
$ sudo yum makecache fast
```
安装docker-ce
```
$ sudo yum install docker-ce
```
查询docker版本
```
$ yum list docker-ce.x86_64  --showduplicates | sort -r
docker-ce.x86_64 17.06.0.el7 docker-ce-stable
```
指定版本安装
```
$ sudo yum install docker-ce-<VERSION>
```
测试容器hello-world
```
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
```
# docker images 
REPOSITORY               TAG             IMAGE ID        CREATED         VIRTUAL SIZE
ubuntu                   14.10           2185fd50e2ca    13 days ago     236.9 MB
…
```
其中我们可以根据`REPOSITORY`来判断这个镜像是来自哪个服务器，如果没有 / 则表示官方镜像，类似于`username/repos_name`表示`Github`的个人公共库，类似于`regsistory.example.com:5000/repos_name`则表示的是私服。
`IMAGE ID`列其实是缩写，要显示完整则带上`--no-trunc`选项
2. 在docker index中搜索image（search）,Usage: `docker search TERM`
```
# docker search seanlo
NAME                DESCRIPTION           STARS     OFFICIAL   AUTOMATED
seanloook/centos6   sean's docker repos         0
```
搜索的范围是官方镜像和所有个人公共镜像。NAME列的 / 后面是仓库的名字。
3. 从`docker registry server` 中下拉image,Usage: `docker pull [OPTIONS] NAME[:TAG]`
```
# docker pull centos
```
上面的命令需要注意，在docker v1.2版本以前，会下载官方镜像的centos仓库里的所有镜像，而从v.13开始官方文档里的说明变了：`will pull the centos:latest image, its intermediate layers and any aliases of the same id`，也就是只会下载tag为latest的镜像（以及同一images id的其他tag）。
也可以明确指定具体的镜像：
```
# docker pull centos:centos6
```
当然也可以从某个人的公共仓库（包括自己是私人仓库）拉取，形如`docker pull username/repository<:tag_name>`
```
# docker pull seanlook/centos:centos6
```
如果你没有网络，或者从其他私服获取镜像，形如`docker pull registry.domain.com:5000/repos:<tag_name>`
```
# docker pull dl.dockerpool.com:5000/mongo:latest
```
4. 推送一个image或repository到registry（push）
与上面的pull对应，可以推送到Docker Hub的Public、Private以及私服，但不能推送到Top Level Repository。
```
# docker push seanlook/mongo
# docker push registry.tp-link.net:5000/mongo:2014-10-27
```
registry.tp-link.NET也可以写成IP，172.29.88.222。
在repository不存在的情况下，命令行下push上去的会为我们创建为私有库，然而通过浏览器创建的默认为公共库。