---
layout: post
title: SVN在linux下的搭建配置
categories: svn
description: 每天记录一点点，快乐工作一辈子
keywords: linux,svn
---

> Linux 下svn通过`svnserve -d -r /svndir`命令创建基础镜像，生成`conf`  `db`  `format`  `hooks`  `locks`  `README.txt`等文件和目录，其中conf下`authz`  `passwd`  `svnserve.conf`三个配置文件，authz主要是配置用户权限，passwd主要是配置用户和密码，svnserve.conf为基础服务配置。

### 权限说明

打开镜像目录下conf下的`authz`文件，groups为用户组设置，其中[]内为项目名称命名文件夹

```shell
[groups]
admin = luming,fengdeen
xtkf = luming,fengdeen
[/]
@admin = rw
[/test]
@xtkf = rw
```

### 用户设置

打开镜像目录下conf下的passwd文件，格式如下

```shell
[users]
luming  = luming123
fengdeen = fengdeen123
```

### 基础服务配置

打开镜像目录下conf下的svnserve.conf文件,其中svndir为镜像路径

```shell
anon-access = none
auth-access = write
password-db = /svndir/conf/passwd
authz-db = /svndir/conf/authz
realm = /svndir

```

### 操作流程

1. 启动命令：`svnserve -d -r /svndir`
2. 停止命令：`ps –ef | grep svn` 查询当前进程是否存在，如果存在需要 `kill -9 进程ID结束`
3. 客户端链接：`svn://121.40.135.43/test`