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
	```
	* 安装依赖包
	$ sudo yum install -y yum-utils device-mapper-persistent-data lvm2
	* 配置存储仓库地址
	$ sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
	* 是否开启测试版本
	$ sudo yum-config-manager --enable docker-ce-edge
	$ sudo yum-config-manager --enable docker-ce-testing
	$ sudo yum-config-manager --disable docker-ce-edge
	* 更新yum包
	$ sudo yum makecache fast
	* 安装docker-ce
	$ sudo yum install docker-ce
	* 查询docker版本
	$ yum list docker-ce.x86_64  --showduplicates | sort -r 
	docker-ce.x86_64 17.06.0.el7 docker-ce-stable
	* 指定版本安装
	$ sudo yum install docker-ce-<VERSION>
	* 测试容器hello-world
	$ sudo docker run hello-world
	* 卸载docker-ce
	$ sudo yum remove docker-ce
	$ sudo rm -rf /var/lib/docker
	```
	3. 通过安装包直接安装
	* 下载centos7下[docker安装包](https://download.docker.com/linux/centos/7/x86_64/stable/Packages/)
	* `$ sudo yum install /path/to/package.rpm`
	* `$ sudo systemctl start docker`
	* `$ sudo docker run hello-world`
	* 卸载`$ sudo yum remove docker-ce`，`$ sudo rm -rf /var/lib/docker`

	