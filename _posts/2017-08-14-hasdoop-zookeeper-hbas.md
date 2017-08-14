---
layout: post
title: 大数据平台配置hadoop+zookeeper+hbase
categories: 大数据
description: 每天记录一点点，快乐工作一辈子
keywords: 大数据
---

========================
# 安装之前的一些准备 #
## linux类系统安装 ##
centos 1.7
## JDK 安装##
java 1.8.0_51
## 用户和用户组的创建及密码设置##
`sudo groupadd hadoop`  
`sudo useradd -g hadoop hadoop`  
`sudo passwd hadoop`

修改/etc/sudoers文件,在root ALL=(ALL:ALL) ALL下添加如下内容
` hadoop ALL=(ALL:ALL)   ALL`
## SSH集群配置 ##

1. 设置每台机器的主机名,例如10.10.243.192的/etc/hostname文件就加` master`,以及/etc/hosts文件,加上下面三条

        10.10.243.192 master
        10.10.243.215 slave1
        10.10.243.124 slave2
<!-- more --> 
1. 安装ssh服务

    `sudo apt-get install ssh openssh-server `

2.  建立ssh无密码登陆本机
   
    转换成hadoop用户：`sudo su - hadoop`
    
    创建ssh-key：`ssh-keygen -t rsa -P ""`

    进入~/.ssh/目录也就是/home/hadoop/.ssh/：
    `cd ~/.ssh`

    复制`id_rsa.pub  `到  `authorized_keys`授权文件中：
    `cp ~/.ssh/id_rsa.pub ~/.ssh/authorized_keys`
1. 修改文件夹权限
   
    权限要求：hadoop目录755,.ssh目录700,authorized_key文件600
    `chmode -R /home/hadoop 755`
1. 登录localhost或者其他服务器   
    `ssh master`
    `ssh localhost`   `ssh slave2`

# Hadoop #
## 介绍 ##
Hadoop是一个由Apache基金会所开发的分布式系统基础架构。



> 用户可以在不了解分布式底层细节的情况下，开发分布式程序。充分利用集群的威力进行高速运算和存储。

> Hadoop实现了一个分布式文件系统Hadoop Distributed File System简称HDFS。HDFS有高容错性的特点，并且设计用来部署在低廉的硬件上,而且它提供高吞吐量来访问应用程序的数据，适合那些有着超大数据集的应用程序。

> Hadoop的框架最核心的设计就是:HDFS和MapReduce。HDFS为海量的数据提供了存储,则MapReduce为海量的数据提供了计算。


## 安装 ##
1. 先配置一台master，然后分发到slave1和slave2,slave节点记得配置环境变量

2. 下载解压hadoop-2.7.1

        tar -zxvf hadoop-2.7.1.tar.gz
3. 修改环境变量配置文件/etc/profile

        export HADOOP_HOME=/home/hadoop/hadoop
        export PATH=.:$HADOOP_HOME/bin:$PATH      //改了之后必须要source /etc/profile一下
4. 修改hadoop/conf/下几个文件

    hadoop-env.sh
    
         export JAVA_HOME=/usr/java/jdk1.8.0_51
         export HADOOP_PID_DIR=/home/hadoop/Hadoop/tmp/pids
    
    yarn-env.sh
  
        export YARN_PID_DIR=/home/hadoop/Hadoop/tmp/pids
    core-site.xml

        <property>
             <name>fs.default.name</name>
             <value>hdfs://master:9000</value>
        </property>
        <property>
             <name>hadoop.tmp.dir</name>
             <value>/home/hadoop/hadoop/tmp</value>
        </property>

    hdfs-site.xml     
        
        <property>
            <name>dfs.data.dir</name>
            <value>/home/hadoop/hadoop/tmp/data</value>
        </property>
        <property>
            <name>dfs.replication</name>
            <value>3</value>
        </property>
        <property> 
            <name>dfs.name.dir</name> 
            <value>/home/hadoop/hadoop/tmp/name </value> 
        </property>
    
    slaves文件
                    
        slave1
        slave2
5. 复制到其他节点

        scp -r /home/hadoop/hadoop hadoop@slave1:/home/hadoop
        scp -r /home/hadoop/hadoop hadoop@slave2:/home/hadoop

## 使用 ##
1. 运行hadoop
    
    注意格式化hdfs文件系统

        hadoop namenode –format

    在主节点上启动hadoop

        /home/hadoop/hadoop/sbin/start-all.sh

2. 检测hadoop是否启动成功
    
    Jps命令查询
          
        [hadoop@master ~]jps //出现NameNode和ResourceMangger
        [hadoop@slave1 ~]jps //出现DateNode和NodeMangger
        [hadoop@slave2 ~]jps //出现DateNode和NodeMangger

    web界面查询

        http://10.10.243.192:50070/

    写入操作

        hadoop fs -put /home/hadoop/hadoop/logs/SecurityAuth-hadoop.audit /

# Zookeeper #
## 介绍 ##
ZooKeeper是一个分布式的，开放源码的分布式应用程序协调服务，是Google的Chubby一个开源的实现，是Hadoop和Hbase的重要组件。
> 它是一个为分布式应用提供一致性服务的软件，提供的功能包括：配置维护、名字服务、分布式同步、组服务等。

> ZooKeeper的目标就是封装好复杂易出错的关键服务，将简单易用的接口和性能高效、功能稳定的系统提供给用户。


## 安装 ##



1.  先配置一台master，然后分发到slave1和slave2,slave节点记得配置环境变量 

2. 下载解压zookeeper-3.4.6

        tar -zxvf zookeeper-3.4.6.tar.gz

3. 修改环境变量配置文件/etc/profile

        ZOOKEEPER_HOME=/home/hadoop/zookeeper
        export PATH=.:${HADOOP_HOME}/sbin:${ZOOKEEPER_HOME}/bin:${ZOOKEEPER_HOME}/conf:$PATH

4. 修改zookeeper/conf/下的文件

       zoo.cfg

        tickTime=2000
        initLimit=10
        syncLimit=5
        dataDir=/home/hadoop/hadoop/tmp/zookeeper
        clientPort=2222
        server.1=master:2888:3888 //在master的dataDir目录下新建myid文件并写入1
        server.2=slave1:2888:3888 //在slave1的dataDir目录下新建myid文件并写入2
        server.3=slave2:2888:3888 //在slave2的dataDir目录下新建myid文件并写入3
5. 复制到从节点，设置从节点的环境变量

        scp -r /home/hadoop/zookeeper hadoop@slave1:/home/hadoop
        scp -r /home/hadoop/zookeeper hadoop@slave2:/home/hadoop

## 使用 ##
1. 启动
  
        /home/hadoop/zookeeper/bin/zkServer.sh start    //三台机器都需要启动
2. 验证

        jps                          //有 QuorumPeerMain服务就算成功
        zkCli.sh -server 10.10.243.192:2222    //zookeeper客户端
        zkServer.sh status                    //查看本机器在zookeeper集群中的角色，一般就是leader和follower

# Hbase #
## 介绍 ##
HBase是一个分布式的、面向列的开源数据库。



> 该技术来源于Google论文“Bigtable：一个结构化数据的分布式存储系统”。就像Bigtable利用了Google文件系统（File System）所提供的分布式数据存储一样，HBase在Hadoop之上提供了类似于Bigtable的能力。

> HBase是Apache的Hadoop项目的子项目。HBase不同于一般的关系数据库，它是一个适合于非结构化数据存储的数据库。另一个不同的是HBase基于列的而不是基于行的模式。

## 安装 ##

1. 先配置一台master，然后分发到slave1和slave2,slave节点记得配置环境变量 

2. 下载解压hbase-1.1.1
   
        tar -zxvf hbase-1.1.1.tar.gz

3. 修改环境变量配置文件/etc/profile

         export HBASE_HOME=/home/hadoop/hbase
         export HBASE_PID_DIR=/home/hadoop/Hadoop/tmp/pids
         export PATH=.:${HBASE_HOME}/bin:$PATH

4. 修改hbase/conf/下几个文件

       hbase-env.sh

        export HBASE_CLASSPATH=/home/hadoop/hadoop/conf:/home/hadoop/hbase/libs
        export HBASE_OPTS="-XX:+UseConcMarkSweepGC"
        export HBASE_MANAGES_ZK=false

       hbase-site.xml
          
        <property>
              <name>hbase.rootdir</name>
              <value>hdfs://master:9000/hbase</value>
        </property>
        <property>
              <name>hbase.cluster.distributed</name>
              <value>true</value>
        </property>
        <property>
              <name>hbase.master</name>
              <value>hdfs://master:6000</value>
        </property>
        <property>
              <name>hbase.tmp.dir</name>
              <value>/home/hadoop/hadoop/tmp/hbase</value>
        </property>
        <property>
              <name>hbase.zookeeper.quorum</name>
              <value>master,slave1,slave2</value>
        </property>
        <property>
              <name>hbase.zookeeper.property.clientPort</name>
              <value>2222</value>
        </property>
        <property> 
             <name>hbase.zookeeper.property.dataDir</name> 
             <value>/home/hadoop/zookeeper/tmp</value> 
        </property>

      regionservers

        slave1
        slave2

5. 把hadoop/conf/hdfs-site.xml文件拷贝至hbase的conf文件夹下

6. 把zookeeper/conf/zoo.cfg拷贝至/home/hdp/hadoop/conf/文件夹下

7. 替换hbase中的包为目前hadoop的包

       `vim f.sh`

        //以下是脚本内容作用是把hbase中自带的hadoop-2.5.1的相关jar包替换成我们hadoop集群的对应版本
        find -name "hadoop*jar" | sed 's/2.5.1/2.7.1/g' | sed 's/\.\///g' > f.log
        rm ./hadoop*jar
        cat ./f.log | while read Line
        do
        find /home/grid/hadoop-2.7.1 -name "$Line" | xargs -i cp {} ./
        done
        rm ./f.log

    `./f.sh`  运行脚本就可以完成替换,很方便

8. 删除重复的包slj4
      
     由于hadoop和hbase中都有slf4j-log4j12-1.7.5.jar会冲突，随便删掉哪一个

9. 复制到从节点,设置从节点的环境变量

        scp -r /home/hadoop/zookeeper hadoop@slave1:/home/hadoop
        scp -r /home/hadoop/zookeeper hadoop@slave2:/home/hadoop

## 使用 ##

1. 启动
   
        /home/hadoop/hbase/bin/start-hbase.sh   //在master上面启动就行
2. 验证

        [hadoop@master ~]jps //出现HMaster
        [hadoop@slave1 ~]jps //出现RegionServer
        [hadoop@slave2 ~]jps //出现RegionServer
        
3. 使用 
        
        habse shell



# 一些注意的地方 #

## 权限问题 ##
          
    .ssh目录权限：700
    authorized_key:600
    /home/hadoop:755

## 格式化HDFS ##

     在bin/hadoop namenode -format 前必须将各节点tmp文件清空！

## source问题 ##
     如果每次都需要source profile:直接在.bashrc文件中加入source /etc/profile 这行语句


## 替换Jar包 ##
   一定要替换，亲测脚本最方便

## hadoop、zookeeper、hbase等集群要求服务器时间完全一致 ##
   就算服务器的时间相差几秒，亲测还有会报错，所以要设置服务器的时间，或者配置时钟服务器
## 所有集群的启动顺序和关闭顺序 ##


   启动之前理应先关闭所有集群

     /home/hadoop/hbase/bin/stop-hbase.sh
     /home/hadoop/zookeeper/bin/zkServer.sh stop
     /home/hadoop/hadoop/sbin/stop-all.sh
     
     /home/hadoop/hadoop/sbin/start-all.sh
     /home/hadoop/zookeeper/bin/
     /home/hadoop/hbase/bin/start-hbase.sh
## 配置文件 ##

   配置的路径，如果文件夹不存在有的需要自己新建下

## 编码连接hbase的时候是通过连接zookeeper的 ##

   需要注意zookeeper是需要主机名配置的，所以发起请求的主机需要在hosts文件中配置集群的所有服务器以及相关的主机名

## nodejs连接数据库 ##
       var hbase = require('hbase-rpc-client')
       var client = hbase({
       zookeeperHosts: ["10.10.243.124:2222"],
       zookeeperRoot: "/hbase",
       rootRegionZKPath: "/meta-region-server",
       rpcTimeout: 30000,
       pingTimeout: 30000,
       callTimeout: 5000

> 打完收工，so easy!!!