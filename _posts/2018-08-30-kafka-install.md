---
layout: post
title: kafka安装和配置
categories: kafka
description: 每天记录一点点，快乐工作一辈子
keywords: kafka zookeeper
---

## 环境配置

+ 操作系统：Cent OS 7 
+ Kafka版本：0.9.0.0 
+ Kafka官网下载：[https://mirrors.cnnic.cn/apache/kafka/](https://mirrors.cnnic.cn/apache/kafka/)  
+ JDK版本：1.7.0_51

## 操作过程

+ 下载：`curl -L -O http://mirrors.cnnic.cn/apache/kafka/0.9.0.0/kafka_2.10-0.9.0.0.tgz ` 
+ 解压：`tar zxvf kafka_2.10-0.9.0.0.tgz ` 

## kafka目录结构

+ /bin 操作kafka的可执行脚本，还包含windows下脚本 
+ /config 配置文件所在目录 
+ /libs 依赖库目录 
+ /logs 日志数据目录，目录kafka把server端日志分为5种类型，分为:server,request,state，log-cleaner，controller 

## 配置程序

1. 修改server.properties文件 

+ broker.id：确保每个节点该值不同 
+ port：kafka提供其他节点或者client访问的端口 
+ log.dir：kafka数据文件存放的位置 
+ advertised.listeners=PLAINTEXT://10.14.84.221:9092 
+ log.cleaner.enable：设置是否启动日志清理过程 
+ zookeeper.connect：设置zookeeper地址与端口 

2. 修改zookeeper.properties文件（只在启动zookeeper的机器上配置） 

+ dataDir：设置zookeeper持久化数据存放路径 
+ clientPort：设置zookeeper的端口号 

## 启功程序 

1. 启动zookeeper 
选kafka机器节点中的一个节点启动zookeeper，命令为 
```
./bin/zookeeper‐server‐startup.sh ./config/zookeeper.properties
```
在所有节点中启动kafka，命令为 
```
./bin/kafka‐server‐startup.sh ./config/server.properties
```

## 遇到问题

1. Failed add leader to partition
有些kafka服务器所有的端口没有开放，在centos7中使用firewall-cmd命令开放相应端口
  

### 可能会出现的问题

java\scala版本不兼容 