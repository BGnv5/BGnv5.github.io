---
title: 史上最详细的hive安装教程，避免踩坑系列
comments: true
date: 2020-03-31 17:34:51
tags: hive
categories: hive
description: hive安装教程
---

网上安装教程很多，照着安装还会出现各种各样的错误，这篇文章手把手教你安装hive，对安装时常见的几种错误也进行了汇总，让你能够轻松避免踩坑~

<!--more-->

#### Hive 安装部署

##### derby版 Hive

前提：安装配置好 Jdk 和 hadoop,并启动 hdfs。

1. 下载安装包,并上传到/opt/software下

地址：[https://mirrors.tuna.tsinghua.edu.cn/apache/hive/]

2.  解压hive压缩包到/opt/module下：
```
tar -zxvf apache-hive-1.2.1-bin.tar.gz -C  ../module
```

3.  将文件重命名为hive-1.2.1文件：
```
mv apache-hive-1.2.1-bin/ hive-1.2.1
```

4. 进入bin目录直接启动
```
[root@hadoop100 software]# cd ../module/hive-1.2.1/
[root@hadoop100 hive-1.2.1]# bin/hive
```

5. 测试一下，查看数据库，出现如下所示即安装成功！
```
hive> show databases;
OK
default
Time taken: 1.63 seconds, Fetched: 1 row(s)
```

**注意：**

如果启动hive时如果报:

```
Call From hadoop /192.168.1.100 to hadoop :9000 failed on connection
```
解决方案：

1. 进入hadoop/bin目录下，格式化namenode

```Shell
[root@hadoop100 module]# cd hadoop-2.7.1/bin
[root@hadoop100 bin]# hadoop namenode -format
```
2. 然后进入hadoop/sbin 执行 sh stop-all.sh

```Shell
[root@hadoop100 bin]# cd ../sbin
[root@hadoop100 sbin]# sh stop-all.sh
```
3. 最后重新启动 sh start-all.sh即可

```
[root@hadoop100 sbin]# sh start-all.sh
```

缺点：使用内置 derby版，不同路径启动hive，每个hive拥有一套自己的元数据，无法共享。

##### Mysql版 Hive

###### 安装并配置MySQL

1. 查看是否安装MySql
```
rpm -qa|grep -i mysql
```
- 如果已经安装过,卸载安装过MySQL:
```
rpm -ev --nodeps mysql-libs-5.1.71-1.el6.x86_64
```
- 删除其他配置
```
rm -rf /usr/my.cnf
rm -rf /root/.mysql_sercret
```

2. 下载并安装MySQL官方的 Yum Repository

```
wget -i -c http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm
```
3. 使用 yum 进行安装

```
yum -y install mysql57-community-release-el7-10.noarch.rpm
```

4. 安装MySQL（完成后就会覆盖掉之前的mariadb）
```
yum -y install mysql-server mysql
```

5. 启动MySql服务
```
[root@hadoop100 ~]# systemctl start mysqld.service
```

启动mysql时报错：
```
Job for mysqld.service failed because the control process exited with error code. See "systemctl status mysqld.service" and "journalctl -xe" for details.
```

解决方法：

- 修改/var/lib/mysql 文件的权限
```
[root@hadoop100 ~]# chmod -R 777 /var/lib/mysql
```
- 删除/var/lib/mysql 下的所有文件
```
[root@hadoop100 ~]# rm -rf /var/lib/mysql/*
```
- 重新启动mysql服务
```
[root@hadoop100 ~]# systemctl start mysqld.service
```
6. 查看MySQL运行状态,出现下图所示即启动成功
```
[root@hadoop100 ~]# systemctl status mysqld.service
```
```
● mysqld.service - MySQL Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   Active: active (running) since 二 2020-03-31 10:29:00 CST; 24s ago
     Docs: man:mysqld(8)
           http://dev.mysql.com/doc/refman/en/using-systemd.html
  Process: 4054 ExecStart=/usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid $MYSQLD_OPTS (code=exited, status=0/SUCCESS)
  Process: 4005 ExecStartPre=/usr/bin/mysqld_pre_systemd (code=exited, status=0/SUCCESS)
 Main PID: 4057 (mysqld)
   CGroup: /system.slice/mysqld.service
           └─4057 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid

3月 31 10:28:51 hadoop100 systemd[1]: Starting MySQL Server...
3月 31 10:29:00 hadoop100 systemd[1]: Started MySQL Server.
```
7. 登入MySQL
```
[root@hadoop100 ~]# mysql -u root -p
```

连接MySQL报错：

```
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)
```

解决方法：

- 查看root用户的密码
```
[root@hadoop100 ~]# grep "password" /var/log/mysqld.log
```

```
2020-03-30T17:20:05.867355Z 1 [Note] A temporary password is generated for root@localhost: Qsye(dZPq
2020-03-30T17:30:01.493042Z 2 [Note] Access denied for user 'root'@'localhost' (using password: YES)
2020-03-30T17:30:43.027818Z 3 [Note] Access denied for user 'root'@'localhost' (using password: YES)
2020-03-30T17:30:58.234454Z 4 [Note] Access denied for user 'root'@'localhost' (using password: YES)
2020-03-30T17:31:29.410990Z 5 [Note] Access denied for user 'root'@'localhost' (using password: YES)
2020-03-30T17:31:43.170470Z 6 [Note] Access denied for user 'root'@'localhost' (using password: YES)
2020-03-30T17:33:41.321386Z 8 [Note] Access denied for user 'root'@'localhost' (using password: YES)
2020-03-30T17:50:55.265575Z 11 [Note] Access denied for user 'root'@'localhost' (using password: YES
2020-03-30T19:27:01.508362Z 0 [Note] Shutting down plugin 'validate_password'
2020-03-30T19:27:04.113128Z 0 [Note] Shutting down plugin 'sha256_password'
2020-03-30T19:27:04.113136Z 0 [Note] Shutting down plugin 'mysql_native_password'
2020-03-31 04:30:50 3851 [Note] Shutting down plugin 'sha256_password'
2020-03-31 04:30:50 3851 [Note] Shutting down plugin 'mysql_old_password'
2020-03-31 04:30:50 3851 [Note] Shutting down plugin 'mysql_native_password'
2020-03-31 04:45:55 6906 [Note] Shutting down plugin 'sha256_password'
2020-03-31 04:45:55 6906 [Note] Shutting down plugin 'mysql_old_password'
2020-03-31 04:45:55 6906 [Note] Shutting down plugin 'mysql_native_password'
2020-03-31 04:55:55 8956 [Note] Shutting down plugin 'sha256_password'
2020-03-31 04:55:55 8956 [Note] Shutting down plugin 'mysql_old_password'
2020-03-31 04:55:55 8956 [Note] Shutting down plugin 'mysql_native_password'
2020-03-31 05:06:10 11074 [Note] Shutting down plugin 'sha256_password'
2020-03-31 05:06:10 11074 [Note] Shutting down plugin 'mysql_old_password'
2020-03-31 05:06:10 11074 [Note] Shutting down plugin 'mysql_native_password'
2020-03-31 05:15:11 13422 [Note] Shutting down plugin 'sha256_password'
2020-03-31 05:15:11 13422 [Note] Shutting down plugin 'mysql_old_password'
2020-03-31 05:15:11 13422 [Note] Shutting down plugin 'mysql_native_password'
2020-03-31 05:25:12 15960 [Note] Shutting down plugin 'sha256_password'
2020-03-31 05:25:12 15960 [Note] Shutting down plugin 'mysql_old_password'
2020-03-31 05:25:12 15960 [Note] Shutting down plugin 'mysql_native_password'
2020-03-31 09:39:35 2385 [Note] Shutting down plugin 'sha256_password'
2020-03-31 09:39:35 2385 [Note] Shutting down plugin 'mysql_old_password'
2020-03-31 09:39:35 2385 [Note] Shutting down plugin 'mysql_native_password'
2020-03-31T02:28:54.650273Z 1 [Note] A temporary password is generated for root@localhost: lts2L)L<h
2020-03-31T02:30:15.184424Z 2 [Note] Access denied for user 'root'@'localhost' (using password: NO)
2020-03-31T02:30:24.888102Z 3 [Note] Access denied for user 'root'@'localhost' (using password: NO)
2020-03-31T02:30:44.372240Z 4 [Note] Access denied for user 'root'@'localhost' (using password: YES)
2020-03-31T02:34:40.134792Z 5 [Note] Access denied for user 'root'@'localhost' (using password: YES)
2020-03-31T02:34:49.375071Z 6 [Note] Access denied for user 'root'@'localhost' (using password: YES)
```
- 根据查找到的密码重新登入即可
```
[root@hadoop100 ~]# mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 12
Server version: 5.7.29
```

8. 修改密码（这里用户名为root，密码为BGnv5_Oooh）
```
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'BGnv5_Oooh';
```
**注意:** 密码设置必须要大小写字母数字和特殊符号（,/';:等）,不然不能配置成功

9. 使用mysql数据库
```
mysql> use mysql;
```

10. 把在所有数据库的所有表的所有权限赋值给位于所有IP地址的root用户
```
mysql> update user set host = '%' where user = 'root';
```

11. 配置远程连接
```
mysql> grant all privileges on *.* to root@'%'identified by 'BGnv5_Oooh' with grant option;
```
12. 刷新MySQL的系统权限相关表
```
mysql> flush privileges;
mysql> exit;
```
###### 安装并配置hive

1. 下载hive压缩包并解压到/opt/module下：

```
tar -zxvf apache-hive-1.2.1-bin.tar.gz -C  ../module
```

2. 配置HIVE环境变量

```
[root@hadoop100 ~]vim /etc/profile

##Hive
export HIVE_HOME=/opt/module/hive-1.2.1
export PATH=$PATH:$HIVE_HOME/bin

[root@hadoop100 ~]# source /etc/profile
```

3. 进入hive/conf 目录下
```
[root@hadoop100 ~]# cd /opt/module/hive-1.2.1/conf/
```
- 配置hive-env.sh文件

```
[root@hadoop100 conf]# cp hive-env.sh.template hive-env.sh
[root@hadoop100 conf]# vim hive-env.sh

# Set HADOOP_HOME to point to a specific hadoop install directory
export HADOOP_HOME=/opt/module/hadoop-2.7.1

# Hive Configuration Directory can be controlled by:
export HIVE_CONF_DIR=/opt/module/hive-1.2.1/conf
```

- 配置 hive-site.xml文件(默认不存在)

```Shell
[root@hadoop100 conf]# vim hive-site.xml

<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
    <property>
            <name>javax.jdo.option.ConnectionURL</name>
            <value>jdbc:mysql://localhost/metastore?createDatabaseIfNotExist=true</value>
    </property>

    <property>
            <name>javax.jdo.option.ConnectionDriverName</name>
            <value>com.mysql.jdbc.Driver</value>
    </property>
    <property>
            <name>javax.jdo.option.ConnectionUserName</name>
            <value>root</value>
    </property>
    <property>
            <name>javax.jdo.option.ConnectionPassword</name>
            <value>BGnv5_Oooh</value>
    </property>
</configuration>
```
4. 将驱动包mysql-connector-java-5.1.38.jar 放在hive/lib目录下

```
[root@hadoop100 conf]# cd /opt/module/hive-1.2.1/lib
```
5. 对MySQL 进行初始化
```
[root@hadoop100 conf]# schematool -dbType mysql -initSchema
```

6. 启动hadoop集群
```
[root@hadoop100 conf]# cd /opt/module/hadoop-2.7.1/
[root@hadoop100 hadoop-2.7.1]# sbin/start-all.sh
```

7. 启动Hve
```
[root@hadoop100 conf]# hive
```

8. 测试hive

- 在hive/conf目录下创建一个BGnv5数据库，并查看数据库
```
hive> create database BGnv5;
OK
Time taken: 1.317 seconds
```
```
hive> show databases;
OK
bgnv5
default
Time taken: 0.664 seconds, Fetched: 2 row(s)
```
- 切换到主目录,在主目录处启动hive，并查看数据库

```
[root@hadoop100 conf]# cd 
```
```
[root@hadoop100 ~]# hive
```
```
hive> show databases;
OK
bgnv5
default
Time taken: 1.565 seconds, Fetched: 2 row(s)
```
可以观察到不论是在哪里启动hive，数据都可以共享，MySQL版hive安装成功。

