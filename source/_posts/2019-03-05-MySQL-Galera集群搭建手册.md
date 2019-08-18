---
layout: lay_post
title: "MySQL-Galera集群搭建手册"
date: 2019-03-05
categories: MySQL
tags: [数据库]
author: lvyafei
---

## 1.安装

首先配置yum源。在/etc/yum.repos.d/ 的目录下编辑gelare.repo文件。
<!--more-->

```xml
[galera]
name = Galera
baseurl = http://releases.galeracluster.com/galera-3/DIST/RELEASE/ARCH
gpgkey = http://releases.galeracluster.com/GPG-KEY-galeracluster.com
gpgcheck = 1

[mysql-wsrep]
name = MySQL-wsrep
baseurl =  http://releases.galeracluster.com/mysql-wsrep-VERSION/DIST/RELEASE/ARCH
gpgkey = http://releases.galeracluster.com/GPG-KEY-galeracluster.com
gpgcheck = 1
```

注意需要替换掉相应的参数：
VERSION是mysql-wsrep的版本，如5.7；
DIST是操作系统，如centos；
RELEASE是操作系统版本，如6；
ARCH是系统架构，如x86_64；
视自己的环境不同作出相应的更改。
源配置好之后，就可以安装了，很简单，只需执行以下命令：

```shell
yum install galera-3 mysql-wsrep-5.7
```

*手动安装：*

使用 (yum install -y 包名)命令进行安装。
  安装顺序：
    1、	mysql-wsrep-libs-compat-5.7-5.7.23-25.15.el7.x86_64.rpm
    2、	mysql-wsrep-common-5.7-5.7.23-25.15.el7.x86_64.rpm
    3、	mysql-wsrep-libs-5.7-5.7.23-25.15.el7.x86_64.rpm
    4、	mysql-wsrep-devel-5.7-5.7.23-25.15.el7.x86_64.rpm
    5、	mysql-wsrep-client-5.7-5.7.23-25.15.el7.x86_64.rpm
    6、	mysql-wsrep-server-5.7-5.7.23-25.15.el7.x86_64.rpm
    7、	mysql-wsrep-5.7-5.7.23-25.15.el7.x86_64.rpm
    8、	mysql-wsrep-test-5.7-5.7.23-25.15.el7.x86_64.rpm（该包在安装时会提示没有	mysql-wsrep-server的依赖，使用rpm 加上–nodeps参数安装，该安装可忽略）
    9、galera-3-25.3.24-2.el7.x86_64

## 2.初始化

1.使用命令：mysqld --initialize 初始化mysql
2.查看默认密码: grep 'temporary password' /var/log/mysqld.log
3.更改权限：chown -R mysql:mysql /var/lib/mysql
4.启动mysql服务:service mysqld start
  进入mysql：mysql -uroot -p
  执行修改密码：alter user user() identified by "新密码";
5.开放远程登录授权
	GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
	flush privileges;
6.去掉Postfix,这个可能跟MySQL配置有冲突: yum remove postfix -y
7.关闭防火墙(生产上可开放端口)：
	setenforce 0 && systemctl stop firewalld
8.关闭mysql服务：
	service mysqld stop （注意mysql5.7是mysqld）

## 3.配置文件

1) 修改my.cnf

```shell
vim /etc/my.cnf，在末尾增加：!includedir /etc/my.cnf.d/
```

2) 修改wsrep.cnf

```shell
vi /etc/my.cnf.d/wsrep.cnf
```

```prop
[mysqld]
server-id=133 #每个节点一个唯一的ID
default-storage-engine=innodb
innodb_autoinc_lock_mode=2
innodb_locks_unsafe_for_binlog=1
query_cache_size=0
query_cache_type=0
bind-address=0.0.0.0
wsrep_provider=/usr/lib64/galera-3/libgalera_smm.so
wsrep_provider_options='gmcast.listen_addr=tcp://192.168.88.133:4567' #修改为本节点地址
wsrep_cluster_name="my_wsrep_cluster" #集群名称，所有节点配置为同一个
wsrep_cluster_address=gcomm://192.168.88.133:4567,192.168.88.128:4567 #节点中所有节点地址
wsrep_node_name=node133 #node名称，每个节点名称唯一
wsrep_node_address='192.168.88.133' #本节点地址
wsrep_node_incoming_address='192.168.88.133' #本节点地址
wsrep_slave_threads=1
wsrep_certify_nonPK=1
wsrep_max_ws_rows=131072
wsrep_max_ws_size=1073741824
wsrep_debug=0
wsrep_convert_LOCK_to_trx=0
wsrep_retry_autocommit=1
wsrep_auto_increment_control=1
wsrep_drupal_282555_workaround=0
wsrep_causal_reads=0
wsrep_notify_cmd=
wsrep_sst_method=rsync
wsrep_sst_auth=root:2464323 #数据量库用户名密码
wsrep_sst_donor='node133,node128' #节点中所有节点的node名称
```

## 4.启动

任意选取一个节点，执行：

```shell
/usr/bin/mysqld_bootstrap
```

然后登陆到mysql ，执行：

```shell
show status like "wsrep_cluster_size";
```

不出意外的话，就会显示1，表明这个集群内现在只有一个节点.

其他的节点依次执行：

```shell
service mysqld start
```

然后再查看wsrep_cluster_size数量。

## 5.集群重启：

在集群停止之后重启时，先在每个几点上使用mysqld_safe --wsrep-recover查看节点的Recovered position的值，选择值最大的一个作为第一个几点启动，如果启动失败，则在将作为第一个启动节点的服务器上编辑grastate.dat文件（可以使用find命令查找位置），将safe_to_bootstrap的值设为1，使用/usr/bin/mysqld_bootstrap命令启动该节点，其他节点启动与以前一致。

**查看配置**
```shell
cat /etc/my.cnf.d/wsrep.cnf
```

**修改启动项**
```shell
vim /var/lib/mysql/grastate.dat
safe_to_bootstrap 改为1
```

## 6.错误排查

查看mysql日志位置：

```mysql
show variables like 'log_error';
```