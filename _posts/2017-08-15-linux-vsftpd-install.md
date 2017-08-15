---
layout: post
title: linux vsftpd安装配置
categories: linux
description: 每天记录一点点，快乐工作一辈子
keywords: linux vsftpd ftp
---
>项目中部署vsftpd遇到的坑，记录下来，

## 检查是否安装vsftp

>rpm -qa | grep vsftpd
>如果出现vsftpd-*.*.*-*.*，说明已经安装 vsftp

## yum安装vsftp

```
yum -y install vsftpd
```

## rpm安装vsftp-[安装包地址](https://pkgs.org/download/vsftpd)

```
rpm -ivh vsftpd-3.0.2-21.el7.x86_64.rpm
```

## 测试安装

```
service vsftpd start
启动成功说明安装成功！
```

## 配置vsftpd

```
查找配置目录：whereis vsftpd
默认配置文件: /etc/vsftpd.conf
```

## 配置文件说明（/etc/vsftpd/）

```
1. user_list
禁止或允许使用vsftpd的用户列表文件。这个文件中指定的用户缺省情况（即在/etc/vsftpd/vsftpd，conf中设置userlist_deny=YES）下也不能访问FTP服务器，在设置了userlist_deny=NO时,仅允许user_list中指定的用户访问FTP服务器。
2. ftpusers
禁止使用vsftpd的用户列表文件。记录不允许访问FTP服务器的用户名单，管理员可以把一些对系统安全有威胁的用户账号记录在此文件中，以免用户从FTP登录后获得大于上传下载操作的权利，而对系统造成损坏。
3. vsftpd.conf     主配置文件
```

## vsftpd.conf配置

```
anonymous_enable=NO  /*禁止匿名用户访问*/
local_enable=YES    /*允许本地用户登录*/
write_enable=YES   /*本地用户的写权限*/
local_umask=022    /*使用FTP的本地文件权限*/
anon_upload_enable=YES 
anon_mkdir_write_enable=YES
dirmessage_enable=YES
xferlog_enable=YES /*启用上传/下载日志记录*/
connect_from_port_20=YES /*指定FTP服务器使用20端口进行数据传输*/
chown_username=whoever
chroot_list_enable=YES /*在chroot_list中列出的用户不允许切换到家目录的上级目录*/
chroot_list_file=/etc/vsftpd/chroot_list
listen=NO
pam_service_name=vsftpd
tcp_wrappers=YES /*不使用tcp wrapper来控制主机访问*/
userlist_deny=NO /*限定user_list配置用户*/
pasv_min_port=5555
pasv_max_port=5566
allow_writeable_chroot=YES

```

## 防火墙设置

```
iptables -nL  /*查看全部端口限制策略*/
iptables -I INPUT 5 -p tcp --dport 21 -j ACCEPT
vi /etc/sysconfig/iptables-config
添加：IPTABLES_MODULES="ip_nat_ftp"
iptables save
```

## 添加用户ftpuser

```
useradd -d /home/ftp -s /sbin/nologin ftpuser
说明:
-d 用访问的根目录
-s /sbin/nologin  禁止该用户登录
-g 创建组
-G 给用户分配组
passwd ***
user_list 中添加用户ftpuser
service vsftpd restart 
```

> 打开工具WinSCP或者filezilla工具测试连接，打完收工，so easy!!!