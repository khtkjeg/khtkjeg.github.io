---
layout: post
title: elasticsearch II 集群搭建
categories: elasticsearch
description: 每天记录一点点，快乐工作一辈子
keywords: elasticsearch claster
---

> 之前在[elasticsearch I](http://luming.men/2017/09/21/linux-elasticsearch-I/) 中介绍了如何搭建单机版的es和注意的事项，今天介绍一下集群的搭建方法，很简单，因为es原生就是分布式

## 部署安装说明

* 将安装好的文件夹拷贝一份到其它节点，`elasticsearch-5.6.1/data`下的数据可以不用拷贝
* 按照`elasticsearch I`中方法创建指定用户,修改`max_map_count `,`limits.conf`
* 修改`elasticsearch-5.6.1/config/elasticsearch.yml`配置文件

```shell
# ---------------------------------- Cluster --------------------------------
# Use a descriptive name for your cluster:
 cluster.name: bigData-xtkf
# ------------------------------------ Node ---------------------------------
# Use a descriptive name for the node:
 node.name: node-4
# ---------------------------------- Network ---------------------------------
# Set the bind address to a specific IP (IPv4 or IPv6):
 network.host: 192.168.1.4 
# --------------------------------- Discovery --------------------------------
# Pass an initial list of hosts to perform discovery when new node is started:
# The default list of hosts is ["127.0.0.1", "[::1]"]
 discovery.zen.ping.unicast.hosts: ["192.168.1.4:9300", "192.168.1.2:9300","192.168.1.3:9300","192.168.1.4:9300"]
```

* 其中`cluster.name` 为集群名称，必须保证所有节点都要一致

## 新节点找不到主节点

* 网络防火墙原因：`iptables -I INPUT 5 -p tcp --dport 9300 -j ACCEPT`

## kibana管理

![claster](/images/posts/elasticsearch/claster.png)

## 总结

> 动态扩充集群的时候需要重启主节点。
