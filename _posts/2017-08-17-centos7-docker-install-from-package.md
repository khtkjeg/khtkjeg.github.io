---
layout: post
title: docker本地安装
categories: Linux
description: 每天记录一点点，快乐工作一辈子
keywords: centos7 docker install package
---

>由于业务机器没有联网，所以只能下载安装包进行手动安装，个人安装选择CE版本

## 下载安装包

>[docker-ce-17.03.2.ce-1.el7.centos.x86_64.rpm](https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-17.03.2.ce-1.el7.centos.x86_64.rpm)
>[docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch.rpm](https://download.docker.com/linux/centos/7/x86_64/stable/Packages/docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch.rpm)
>[libtool-ltdl-devel-2.4.2-22.el7_3.x86_64.rpm](https://centos.pkgs.org/7/centos-updates-x86_64/libtool-ltdl-devel-2.4.2-22.el7_3.x86_64.rpm.html)
>[lib64seccomp2-2.3.2-1.mga6.x86_64.rpm](ftp://195.220.108.108/linux/mageia/distrib/cauldron/x86_64/media/core/release/lib64seccomp2-2.3.2-1.mga6.x86_64.rpm)

## rpm安装

>$ rpm -ivh *.policycoreutils-python-2.7-1.fc27.x86_64.rpm  --nodeps --force

## 启动服务

>service docker start


>由于系统版本不同自带的依赖库可能不同，导致有些依赖安装很麻烦，本文是以centos7 最小系统安装部署。