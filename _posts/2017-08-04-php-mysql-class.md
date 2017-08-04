---
layout: post
title: php mysql链接类总结
categories: PHP
description: 每天记录一点点，快乐工作一辈子
keywords: PHP mysql
---

> 很久没有写php了,这次也是项目中的需要通过php链接mysql，于是乎就把很久以前写过的源码整理了出来直接copy过来，也算温习一下php知识吧。

## 简单连接增删改查走起：文件名class.mysql.php
```php
<?php

    class mysqlDB {
        
        private $host;
        private $port;
        private $username;
        private $password;
        private $dbname;
        private $charset;

        // 数据库信息
        function __construct( $host, $port, $username, $password, $dbname, $charset )
        {
            $this->host = $host;
            $this->port = $port;
            $this->username = $username;
            $this->password = $password;
            $this->dbname = $dbname;
            $this->charset = $charset;
        }
        
        // 查询
        function select($sql){
            $conn = mysql_connect ( $this->host.":".$this->port, $this->username, $this->password ) or die ("Could not connect");
            mysql_select_db($this->dbname);
            if(strtolower($this->charset)=='utf-8' or strtolower($this->charset)=='utf8'){
                mysql_query('SET NAMES UTF8');
            }
            //mysql_query("SET time_zone='+8:00'");
            $result = mysql_query($sql);
            $returnArr = Array();
            while ($row=mysql_fetch_array($result,MYSQL_ASSOC)){
                array_push ( $returnArr, $row );
            }
            return $returnArr;
        }
        
        // insert into 插入
        function insert($sql){
            try{
                $conn = mysql_connect ( $this->host.":".$this->port, $this->username, $this->password ) or die ("Could not connect");
                mysql_select_db($this->dbname);
                if(strtolower($this->charset)=='utf-8' or strtolower($this->charset)=='utf8'){
                    mysql_query('SET NAMES UTF8');
                }
                mysql_query("SET time_zone='+8:00'");
                $result = mysql_query($sql);
                if (!$result) {
                    return 0;
                }
                $affectRows = mysql_affected_rows();
                return 1;
            }catch(Exception $e){
                return $e;
            }
        }
        
        // 更新
        function update($sql){
            $conn = mysql_connect ( $this->host.":".$this->port, $this->username, $this->password ) or die ("Could not connect");
            mysql_select_db($this->dbname);
            if(strtolower($this->charset)=='utf-8' or strtolower($this->charset)=='utf8'){
                mysql_query('SET NAMES UTF8');
            }
            mysql_query("SET time_zone='+8:00'");
            $result = mysql_query($sql);
            $affectRows = mysql_affected_rows();
            return $affectRows;
        }
        
        // 删除
        function delete($sql){
            $conn = mysql_connect ( $this->host.":".$this->port, $this->username, $this->password ) or die ("Could not connect");
            mysql_select_db($this->dbname);
            if(strtolower($this->charset)=='utf-8' or strtolower($this->charset)=='utf8'){
                mysql_query('SET NAMES UTF8');
            }
            mysql_query("SET time_zone='+8:00'");
            $result = mysql_query($sql);
            $affectRows = mysql_affected_rows();
            return $affectRows;
        }
    }
?>
```
## 应用
```php
include_once("config.php");
include_once("class.mysql.php");

$db_host = constant("DB_HOST");
$db_port = 3306;
$db_user = constant("DB_USER");
$db_passwd = constant("DB_PASSWORD");
$db_dbname = constant("DB_NAME");
$db_charset = "utf8mb4";

$mysqlDB = new mysqlDB( $db_host, $db_port, $db_user, $db_passwd, $db_dbname, '
utf8' );

$result_select = $mysqlDB->select($sql);
```

> 打完收工，so easy!!!