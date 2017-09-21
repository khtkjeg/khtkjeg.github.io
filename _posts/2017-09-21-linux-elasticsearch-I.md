---
layout: post
title: elasticsearch I 安装部署测试
categories: elasticsearch
description: 每天记录一点点，快乐工作一辈子
keywords: elasticsearch
---

> 项目中的实践，起初想用Hbase做数据数据存储和查询工具，于是就搭建了单机模式，同时入了随机数据200多万（样本较小），同时要实现多条件复杂过滤查询，同时得排序分页等简单功能，通过验证设计好rowkey指定查询还是挺快的但是加上过滤器速度就不行了，后来想到了搜索引擎elasticsearch简称ES，同样部署单机模式，对比搜索性能完爆Hbase

## 部署安装说明

* 部署环境：`linux centos7 jdk8`
* 安装包  ：[地址](https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.6.1.tar.gz)
* 解压    ：`tar -zxvf elasticsearch-5.6.1.tar.gz`
* 创建用户：`useradd es`,`passwd es` (ES默认是禁止以root用户启动)
* 切换用户：`su es`
* 修改文件夹所属目录：`chmod -R es elasticsearch-5.6.1 `
* 启动ES：`cd elasticsearch-5.6.1/bin`,`./elasticsearch`
* 后台方式启动：`./elasticsearch -d`
* 查看服务进程：`jps`
* 重启服务进程

```shell
jps | grep Elasticsearch | awk {'print$1'} | xargs kill -9 
./elasticsearch -d
```

## 测试请求

* `curl -X GET http://localhost:9200` 测试成功，返回结果

```json
{
  "name" : "R3Dz-y8",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "K9Ma1r39SBGvhTKiMwCJkA",
  "version" : {
    "number" : "5.6.1",
    "build_hash" : "667b497",
    "build_date" : "2017-09-14T19:22:05.189Z",
    "build_snapshot" : false,
    "lucene_version" : "6.6.1"
  },
  "tagline" : "You Know, for Search"
}
```

## 远程访问失败

> 错误主要分为三部分 1、访问权限受限，2、vm.max_map_count 设置太小，3、文件描述和ES的大小不匹配

* 修改配置文件`elasticsearch-5.6.1/config/elasticsearch.yml`,找到 network.host 将前面#去掉 注意加个空格否则报错 ，同时修改地址为0.0.0.0 同样前面加空格

```
# ---------------------------------- Network
#
# Set the bind address to a specific IP (IPv4 or IPv6):
#
 network.host: 0.0.0.0
#
# Set a custom port for HTTP:
#
#http.port: 9200
#
# For more information, consult the network module documentation.
#
```

* 修改vm.max_map_count（虚拟内存）

root用户下执行： `vi /etc/sysctl.conf`

```shell
vm.max_map_count=200000
```

保存配置修改：`sysctl -p`

* 文件descriptors修改

root用户下执行：`vi /etc/security/limits.conf`,添加配置

```shell
es soft nofile 65536 (es为用户名)
es hard nofile 65536
```

* 测试请求

linux：`curl -X GET http://192.168.1.2:9200`
浏览器：`http://192.168.1.2:9200`

## 推荐客户端谷歌浏览器插件

* [Sense](http://www.cnplugins.com/down/predown.aspx?fn=1412/www.cnplugins.com_lhjgkmllcaadmopgmanpapmpjgmfcfig_0_9_0_.crx&aid=2935)

![sense](/images/posts/elasticsearch/sense.png)

## 总结

> elasticsearch相比于Hbase存在很多有点，最直接的就是查询速度完爆Hbase，目前测试数据4400万，匹配查询取100条速度在100ms以内，而且是搭建的单机模式速度基本满足要求，还有ES磁盘存储量远远小于Hbase。
