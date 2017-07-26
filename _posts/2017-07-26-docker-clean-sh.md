---
layout: post
title: docker快速清理整理
categories: Docker
description: 每天记录一点点，快乐工作一辈子
keywords: docker
---

> 每天记录一点点，快乐工作一辈子

## 清理容器产生的日志（多为控制台输出日志）
```shell
 #!/bin/bash
echo "================== start clean docker containers logs ================"
logs=$(find /var/lib/docker/containers/ -name *-json.log)
for log in $logs
	do
		echo "ckeab logs : $log"
		cat /dev/null > $log
	done
echo "================== end clean docker containers logs =================="
```
## 杀死容器的进程
```shell
docker kill $(docker ps -a -q)
```
## 删除所有已经停止的容器
```shell
docker rm $(docker ps -a -q)
```
## 删除所有未打 dangling 标签的镜像
```shell
docker rmi $(docker images -q -f dangling=true)
```
## 删除所有镜像
```shell
docker rmi $(docker images -q)
```