---
layout: post
title: Elasticsearch 数据备份迁移
categories: elasticsearch
description: 每天记录一点点，快乐工作一辈子
keywords: linux,elasticsearch
---

> ES数据迁移在数据量很小的情况下，`elasticdump`工具就能实现，但是量级到白G或T级的时候，受限于查询分页，导致后面越来越慢直到异常。官网提供了使用快照的方法进行备份，在新的集群中直接恢复的方法。


### 旧集群创建备份仓库

修改配置文件`elasticsearch.yml`,添加如下配置(需要在旧集群的每个节点上添加)：

```shell
path.repo: /home/es/elasticsearch-5.6.2/back_data
```

创建快照仓库命令：

```shell
curl -XPUT http://10.14.84.47:9200/_snapshot/my_backup -d '
{
    "type": "fs",
    "settings": {
        "location": "/home/es/elasticsearch-5.6.2/back_data",
"compress": true
    }
}
'
```

查看仓库

```shell
curl -XGET http://10.14.84.47:9200/_snapshot/my_backup?pretty
```


### 创建快照备份

创建所有索引的快照

```shell
curl -XPUT http://10.14.84.47:9200/_snapshot/my_backup/snapshot_20180606
```

创建指定索引的快照

```shell
curl -XPUT http://10.14.84.47:9200/_snapshot/my_backup/snapshot_20180606 -d '
{
    "indices": "cimiss"
}
'
```

如果成功返回结果显示{"accepted":true}

### 查看备份

数据量大，恢复需要一定时间，可以通过命令查看状态

```shell
 curl -XGET http://10.14.84.47:9200/_snapshot/my_backup/snapshot_20180606?pretty
```

###　新集群添加备份仓库

修改配置文件`elasticsearch.yml`,添加如下配置(需要在新集群的每个节点上添加)：

```shell
path.repo: /home/es/elasticsearch-5.6.2/back_data
```

### 备份数据操作

旧的集群备份出来的数据，要逐一拷贝到新集群的公共文件夹。也可以通过在新集群中创建共享文件夹的方式，将旧集群上的每个节点备份目录挂载。

在10.14.84.227创建共享目录，执行`vim /etc/exports`

```shell
/home/es/elasticsearch-5.6.2/back_data *(insecure,rw,no_root_squash,sync,anonuid=500,anongid=500)
```

重启启动新集群机器的NFS服务  `systemctl restart nfs`

在旧集群中挂载

```shell
mount -t nfs 10.14.84.47:/home/es/elasticsearch-5.6.2/back_data /home/es/elasticsearch-5.6.2/back_data -o proto=tcp -o nolock
```

### 执行恢复

```shell
curl -XPOST http://10.14.84.227:9200/_snapshot/my_backup/snapshot_20180606/_restore
```

状态查询

```shell
curl -XGET http://10.14.84.227:9200/_snapshot/my_backup/snapshot_20180606/_status 
```

```shell
curl  http://10.14.84.227:9200/_cat/indices?v
```

> 亲测可行