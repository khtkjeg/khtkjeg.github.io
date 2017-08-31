---
layout: post
title: Rsync实现内外网的同步
categories: rsync
description: 每天记录一点点，快乐工作一辈子
keywords: linux rsync
---

>由于网络的原因需要部署两套web系统，造成产生的数据文件无法实时同步，本地网络状态是内网到外网是通的，外网到内网是断开的，所以采取的策略是内网做为客户端，外网作为服务端，内网定时向外网做推拉操作，两台linux机器举例，内网IP：192.168.0.1，本地同步目录：/testSync1;外网ip：192.168.0.2，同步目录：/testSync2

# 检查两台机器是否安装rsync

>一般情况linux都是自带rsync.

```shell
yum install rsync
```

# 服务端 （192.168.0.2）

## 添加服务端配置文件

>控制台执行`vi /etc/rsyncd.conf` 创建配置文件（目录文件名可自行定义）,内容如下，可以根据自己需求添加或删除

```shell
uid = nobody
gid = nobody
use chroot = no
max connections = 10
pid file = /var/run/rsyncd.pid
lock file = /var/run/rsyncd.lock
log file = /var/run/rsyncd.log

[modeOne]
path = /testSync2
comment = web_bak file
ignore errors
read only = no
write only = no
list = no
auth users = root
secrets file = /etc/rsyncd.pass
hosts deny=192.168.0.1/22
```

## 创建服务端密码文件
>这里的密码是服务器端ssh连接密码，要与`rsyncd.conf`中的`secrets file`一致，执行如下操作

```shell
echo "root:12345" > /etc/rsyncd.pass

chmod 600 /etc/rsyncd.pass
```

## 防火墙设置
>rsync默认端口是873，通过`iptables --list`可以查看rsync是否在其中，如果不在其中可以添加端口到iptables列表中

```shell
iptables -I INPUT -p tcp --dport 873 -j ACCEPT

/etc/rc.d/init.d/iptables save
```

## 停止当前rsync服务
>配置文件改好后如果服务一直在启动是服务将配置文件启动的，需要先停止，同时删掉pid进程文件

```shell
netstat -nlp --ip //查询rsync运行状态，找到PID

kill -9 PIP
```

## 启动rsync
>在`vi /var/run/rsyncd.log`中可以看执行状态

```shell
/usr/bin/rsync --daemon --config=/etc/rsyncd.conf
```

# 客户端（192.168.0.1）

## 创建密码文件
>相比于服务端客户端不需要启动rsync服务,注意这里不需要加`root:`

```shell
echo "12345" > /etc/rsyncd.pass

chmod 600 /etc/rsyncd.pass
```

## 客户端拉取
>这里可以根据需要添加rsync附加参数

```shell
rsync -avz --progress -e 'ssh -p 22' --password-file=/etc/rsyncd.pass 
root@192.168.0.2::modeOne /testSync1
```

## 客户端推送
>

```shell
rsync -avz --progress -e 'ssh -p 22' --password-file=/etc/rsyncd.pass /testSync1/* root@192.168.0.2::modeOne
```

## 运行结果
>通过推拉可以实现双向同步，但是也有弊端，只能增加，不能删除；同时执行时依然需要手动输入密码，以为是`--password-file` 没有起作用后来去掉后执行，居然需要数据两次密码，说明`--password-file`起作用了，仔细想想可能是ssh安全策略问题,基于这个问题没办法只能硬办法添加ssh公钥免密码请求

# ssh免密码登录
>创建公钥私钥，将公钥添加到服务器端`/root/.shh/authorized_keys`中

```shell
ssh-keygen -t rsa -P

ssh-copy-id -i /root/.shh/id_rsa.pub -p 22 root@192.168.0.2

ssh -p '22' 'root@192.168.0.2'
```

# 总结
>虽然实现夸网络的同步但是仍然是一端发起的，这种情况是先拉取还是先推送都会影响结果，而且rsync参数中不能加`--delete`，因为两边都会有文件增加或删减，所以只能舍弃删除，即使一端删除仍可会被重新copy过去。
