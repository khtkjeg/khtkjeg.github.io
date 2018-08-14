---
layout: post
title: mysql主从复制和读写分离
categories: mysql
description: 每天记录一点点，快乐工作一辈子
keywords: mysql master slave proxy
---

> 以往的经验单台mysql作为独立数据库是完全不能满足实际需求，尤其是安全性，高可用性以及高并发等多方面。一般这两种中间价搭配使用的时候比较多，主从复制（Master-Slave）的方式来同步数据，再通过读写分离（MySQL-Proxy）来提升数据库的并发负载能力，这样的方案来进行部署与实施（如下图）。

![mysql-master-salve-proxy](/images/posts/mysql/mysql-master-salve-proxy.jpg)

### MYSQL安装部署

具体的安装过程，建议参考我的这一篇文章：`http://heylinux.com/archives/993.html`
值得一提的是，我的安装过程都是源码包编译安装的，并且所有的配置与数据等都统一规划到了`/opt/mysql`目录中，因此在一台服务器上安装完成以后，可以将整个mysql目录打包，然后传到其它服务器上解包，便可立即使用。

当然也可以用docker 创建mysql容器方法，更简单。

### MySQL主从复制

场景描述：
主数据库服务器：192.168.10.130，MySQL已经安装，并且无应用数据。
从数据库服务器：192.168.10.131，MySQL已经安装，并且无应用数据。

#### 主服务器上进行的操作

启动mysql服务
/opt/mysql/init.d/mysql start

通过命令行登录管理MySQL服务器
/opt/mysql/bin/mysql -uroot -p'new-password'

授权给从数据库服务器192.168.10.131
mysql> GRANT REPLICATION SLAVE ON *.* to 'rep1'@'192.168.10.131' identified by 'password';

查询主数据库状态
Mysql> show master status;
+------------------+----------+--------------+------------------+
| File | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000005 | 261 | | |
+------------------+----------+--------------+------------------+

记录下 FILE 及 Position 的值，在后面进行从服务器操作的时候需要用到。

#### 配置从服务器

修改从服务器的配置文件/opt/mysql/etc/my.cnf
将 server-id = 1修改为 server-id = 10，并确保这个ID没有被别的MySQL服务所使用。

启动mysql服务
/opt/mysql/init.d/mysql start

通过命令行登录管理MySQL服务器
/opt/mysql/bin/mysql -uroot -p'new-password'

执行同步SQL语句
mysql> change master to
master_host='192.168.10.130',
master_user='rep1',
master_password='password',
master_log_file='mysql-bin.000005',
master_log_pos=261;

正确执行后启动Slave同步进程
mysql> start slave;

主从同步检查
mysql> show slave status\G

==============================================
Slave_IO_State:
Master_Host: 192.168.10.130
Master_User: rep1
Master_Port: 3306
Connect_Retry: 60
Master_Log_File: mysql-bin.000005
Read_Master_Log_Pos: 415
Relay_Log_File: localhost-relay-bin.000008
Relay_Log_Pos: 561
Relay_Master_Log_File: mysql-bin.000005
Slave_IO_Running: YES
Slave_SQL_Running: YES
Replicate_Do_DB:
……………省略若干……………
Master_Server_Id: 1
1 row in set (0.01 sec)
==============================================

其中Slave_IO_Running 与 Slave_SQL_Running 的值都必须为YES，才表明状态正常。

如果主服务器已经存在应用数据，则在进行主从复制时，需要做以下处理：
(1)主数据库进行锁表操作，不让数据再进行写入动作
mysql> FLUSH TABLES WITH READ LOCK;

(2)查看主数据库状态
mysql> show master status;

(3)记录下 FILE 及 Position 的值。
将主服务器的数据文件（整个/opt/mysql/data目录）复制到从服务器，建议通过tar归档压缩后再传到从服务器解压。

(4)取消主数据库锁定
mysql> UNLOCK TABLES;

#### 验证主从复制

主服务器上的操作

在主服务器上创建数据库first_db
mysql> create database first_db;
Query Ok, 1 row affected (0.01 sec)

在主服务器上创建表first_tb
mysql> create table first_tb(id int(3),name char(10));
Query Ok, 1 row affected (0.00 sec)

在主服务器上的表first_tb中插入记录
mysql> insert into first_tb values (001,’myself’);
Query Ok, 1 row affected (0.00 sec)

在从服务器上查看
mysql> show databases;
=============================
+--------------------+
| Database |
+--------------------+
| information_schema |
| first_db |
| mysql |
| performance_schema |
| test |
+--------------------+
5 rows in set (0.01 sec)
=============================
数据库first_db已经自动生成

mysql> use first_db
Database chaged

mysql> show tables;
=============================
+--------------------+
| Tables_in_first_db |
+--------------------+
| first_tb |
+--------------------+
1 row in set (0.02 sec)
=============================
数据库表first_tb也已经自动创建

mysql> select * from first_tb;
=============================
+------+------+
| id | name |
+------+------+
| 1 | myself |
+------+------+
1 rows in set (0.00 sec)
=============================
记录也已经存在

由此，整个MySQL主从复制的过程就完成了，接下来，我们进行MySQL读写分离的安装与配置。

### MySQL读写分离

场景描述：
数据库Master主服务器：192.168.10.130
数据库Slave从服务器：192.168.10.131
MySQL-Proxy调度服务器：192.168.10.132

以下操作，均是在192.168.10.132即MySQL-Proxy调度服务器 上进行的。

#### 安装配置mysql-proxy

推荐采用已经编译好的二进制版本，因为采用源码包进行编译时，最新版的MySQL-Proxy对automake，glib以及libevent的版本都有很高的要求，而这些软件包都是系统的基础套件，不建议强行进行更新,下载地址`https://downloads.mysql.com/archives/proxy/`

进入到 `/usr/local/mysql-proxy` 安装目录

执行：bin/mysql-proxy -P 192.168.10.132:3306 --daemon --keepalive -b 192.168.10.130:3306 -r 192.168.10.131:3306 -s /usr/local/mysql-proxy/share/doc/mysql-proxy/rw-splitting.lua&

到此读写分离就可以运行了，建议主从用相同的用户密码，就可以直接用navicat客户端登录192.168.10.132进行读写操作测试。

### 经验分享

1. 当MySQL主从复制在 show slave status\G 时出现Slave_IO_Running或Slave_SQL_Running 的值不为YES时，需要首先通过 stop slave 来停止从服务器，然后再执行一次本文开始步骤即可恢复，但如果想尽可能的同步更多的数据，可以在Slave上将master_log_pos节点的值在之前同步失效的值的基础上增大一些，然后反复测试，直到同步OK。因为MySQL主从复制的原理其实就是从服务器读取主服务器的binlog，然后根据binlog的记录来更新数据库。

2. MySQL-Proxy的rw-splitting.lua脚本在网上有很多版本，但是最准确无误的版本仍然是源码包中所附带的lib/rw-splitting.lua脚本，如果有lua脚本编程基础的话，可以在这个脚本的基础上再进行优化；

3. MySQL-Proxy实际上非常不稳定，在高并发或有错误连接的情况下，进程很容易自动关闭，因此打开--keepalive参数让进程自动恢复是个比较好的办法，但还是不能从根本上解决问题，因此通常最稳妥的做法是在每个从服务器上安装一个MySQL-Proxy供自身使用，虽然比较低效但却能保证稳定性；

4. 一主多从的架构并不是最好的架构，通常比较优的做法是通过程序代码和中间件等方面，来规划，比如设置对表数据的自增id值差异增长等方式来实现两个或多个主服务器，但一定要注意保证好这些主服务器数据的完整性，否则效果会比多个一主多从的架构还要差；

5. MySQL-Cluster 的稳定性也不是太好；

6. Amoeba for MySQL 是一款优秀的中间件软件，同样可以实现读写分离，负载均衡等功能，并且稳定性要大大超过MySQL-Proxy，建议大家用来替代MySQL-Proxy，甚至MySQL-Cluster
