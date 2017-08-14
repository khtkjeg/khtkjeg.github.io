---
layout: post
title: Flume部署手册
categories: Flume
description: 每天记录一点点，快乐工作一辈子
keywords: Flume
---

## 安装流程
- **解压安装包**
- **修改flume配置文件**
- **启动flume**

<!-- more --> 

## 修改配置文件
### 配置文件位置
./conf/flume-conf.properties

### 配置文件模板
    aK.sources = tailSrc
    aK.channels = memoryChannel
    aK.sinks =  kafkaSink 
   
    aK.sources.tailSrc.type = exec
    #set monitor file commadnd
    aK.sources.tailSrc.command =java -jar ./soft mytail.jar C:/CAPPILog/%tY%tm%tdCAPPI.txt
    aK.sources.tailSrc.channels = memoryChannel
    aK.sources.tailSrc.restart = true
    aK.sources.tailSrc.restartThrottle = 10000
    
    aK.channels.memoryChannel.type = memory
    aK.channels.memoryChannel.capacity = 100
    
    aK.sinks.kafkaSink.type = org.apache.flume.sink.kafka.KafkaSink
    #set topic
    aK.sinks.kafkaSink.topic = radarobs
    #set broker list
    aK.sinks.kafkaSink.brokerList = 10.10.243.124:9092,10.10.243.192:9092,10.10.243.215:9092
    aK.sinks.kafkaSink.channel = memoryChannel
     
    aK.sinks.loggerSink.channel = memoryChannel
    aK.sinks.loggerSink.type = logger

### 需要修改的地方
> **aK.sources.tailSrc.command：**产生日志输出流的命令，一般为java -jar ./soft/mytail.jar logspath。其中logspath中用%tY代替年份，%tm代替月份，%td代替天。
> **aK.sinks.kafkaSink.topic：**指定kafka中的主题
> **aK.sinks.kafkaSink.brokerList：**制定kafka集群中各个server的列表

## 启动flume
### Win7启动命令###

    ./bin/flume-ng.cmd agent -n aK -c conf -f ./conf/flume-conf.properties
### Linux启动命令###

    ./bin/flume-ng agent -n aK -c conf -f ./conf/flume-conf.properties
### 其他版本windows启动命令###

    cd /d %~dp0
    cd ..
    java -Xms128m -Xmx512m -Dlog4j.configuration=file:///%cd%\conf\log4j.properties -cp .\lib\* org.apache.flume.node.Application -f .\conf\flume-conf.properties -n aK

> 打完收工，so easy!!!