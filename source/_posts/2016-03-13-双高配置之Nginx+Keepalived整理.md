---
layout: lay_post
title: "双高配置之Nginx+Keepalived整理"
date: 2016-03-13 20:59:00
categories: 架构
tags: 双高架构
author: lvyafei
---

## 0.概述

前几天发的一篇"系统架构总结之高可用高负载"文章里面讲到的双高策略，里面包含的详细配置，在这里整理出来以做备忘。

<!-- more -->

## 1.Nginx安装

版本:1.9.11

依赖pcre-8.38.tar.gz

<font color="blue">安装:</font>

	./configure --with-pcre=/aclome/pcre-8.38 --prefix=/aclome/nginx

	Make

	Make install

<font color="blue">配置负载策略:</font>

	http{
	......
	upstream myServer{
	 server 199.31.165.61:8080;
	 server 199.31.165.62:8080;
	}
	server{
	  listen   8001;
	  location / {
        root  html;
        index index.html index.htm;
        proxy_pass http://myServer;
	  }
	}
	......
	}


<font color="blue">启动</font>

	/sbin/nginx

<font color="blue">停止</font>

	ps -ef |grep nginx

	kill -9 <pid>

<font color="blue">访问</font>

http://199.31.165.61:8001  可以在61:8080和62:8080之间跳转

**nginx的upstream目前支持的5种方式的分配**

1、轮询（默认）

每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。

upstream backserver {

server 192.168.0.14;

server 192.168.0.15;

}

2、weight

指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。

upstream backserver {

server 192.168.0.14 weight=10;

server 192.168.0.15 weight=10;

}

3、ip_hash

每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。

upstream backserver {

ip_hash;

server 192.168.0.14:88;

server 192.168.0.15:80;

}

4、fair（第三方）

按后端服务器的响应时间来分配请求，响应时间短的优先分配。

upstream backserver {

server server1;

server server2;

fair;

}

5、url_hash（第三方）

按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。

upstream backserver {

server squid1:3128;

server squid2:3128;

hash $request_uri;

hash_method crc32;

}

在需要使用负载均衡的server中增加

proxy_pass http://backserver/ ;

upstream backserver{

ip_hash;

server 127.0.0.1:9090 down; (down 表示单前的server暂时不参与负载)

server 127.0.0.1:8080 weight=2; (weight 默认为1.weight越大，负载的权重就越大)

server 127.0.0.1:6060;

server 127.0.0.1:7070 backup; (其它所有的非backup机器down或者忙的时候，请求backup机器)

}

max_fails ：允许请求失败的次数默认为1.当超过最大次数时，返回proxy_next_upstream 模块定义的错误。

## 2.Keepalived安装

版本:1.2.19

<font color="blue">安装</font>

	./configure --disable-fwmark --prefix=/aclome/keepalived

	//使用root安装指定如下:
	
	./configure --prefix=/usr --sysconf=/etc

	make 

	make install

<font color="blue">配置</font>

**主:**

	vi /etc/keepalived/keepalived.conf

	global_defs{
	  smtp_server 127.0.0.1
	}
	vrrp_instance_VI_1{
	  state MASTER 
	  interface eth1
	  priority 100
	  ......
	  virtual_ipaddress{
	    199.31.165.65
	  }
	}

**备:**

	vi /etc/keepalived/keepalived.conf

	global_defs{
	  smtp_server 127.0.0.1
	}
	vrrp_instance_VI_1{
	  state BACKUP
	  interface eth1
	  priority 50
	  ......
	  virtual_ipaddress{
	    199.31.165.65
	  }
	}

<font color="blue">启动</font>

	sudo service keepalived start

	sudo service keepalived status

	sudo service keepalived stop 

<font color="blue">验证</font>

![验证](/images/架构/配置/keepalived验证.png)

## 3.Redis安装

版本:3.0.7

**安装:**

	Make PREFIX=/aclome/redis install

**配置:** 

	cp /aclome/redis-3.0.7/utils/redis_init_script /aclome/redis/

	vi redis_init_script

	EXEC=/aclome/redis/bin/redis-server
	CLIEXEC=/aclome/redis/bin/redis-cli

	cp /aclome/redis-0.0.7/redis.conf /aclome/redis/

	Vi redis.conf

	dir ./data

	mkdir data

	mkdir data/sentinel

**启动：**

	nohup bin/redis-server conf/redis.conf &

**停止:**

	bin/redis-cli -p 6379 shutdown


**验证状态:**

	ps -ef |grep redis

![验证](/images/架构/配置/redis验证.png)


**客户端:**

	/aclome/redis/bin/redis-cli -p 6379

连接到远程的redis服务器: bin/redis-cli -h 199.31.165.62 -p 6379 get myname

存储KEY/VALUE命令：set myname “lvyafei”

读取Value命令: get myname

删除缓存命令: del myname

批量删除缓存: ./redis-cli -p 6379 KEYS “*” | xargs ./redis-cli -p 6379 DEL

获取所有Key命令: keys *

**Redis-HA方案-Sentinel配置:**

<font color="red">Redis-server端口:6379  Redis-sentinel端口:7031</font>

>1.master-slave配置

只需要在slave上配置redis.conf即可,master不用配置

	slaveof 199.31.165.61 6379

	slave-read-only yes

在master,slave上分别启动nohup bin/redis-server conf/redis.conf &

停止master/slave命令:bin/redis-cli -p 6379 shutdown

>2.Sentinel配置

在master/slave上配置sentinel.conf（master/slave相同配置）

	#######redis-sentinel######################
	port 7031

	dir ./data/sentinel

	sentinel monitor mymaster 199.31.165.61 6379 1
	sentinel down-after-milliseconds mymaster 5000
	sentinel parallel-syncs mymaster 1
	sentinel failover-timeout mymaster 15000

**启动sentinel：**

分别在master/slave上执行命令：nohup bin/redis-sentinel conf/sentinel.conf &

**3.验证：**

	Bin/redis-cli -p 7031 sentinel masters

停止master:

在master上执行bin/redis-cli -p 6379 shutdown.观察sentinel输出.

停止sentinel:bin/redis-cli -p 7031 shutdown 

## 4.Tomcat Session共享配置

拷贝 jar包到tomcat 的libs目录下:

1.x版本：不支持Redis集群

![配置](/images/架构/配置/tomcat-session1.png)

2.x版本：支持Redis集群

![配置](/images/架构/配置/tomcat-session2.png)

配置:vi /tomcat/config/content.xml

<font color="red">注意：1.x版本的包名和2.x版本的包名不一样，且2.x中多的参数1.x版本中不支持。</font>

1.x版本配置:

![配置](/images/架构/配置/tomcat-session3.png)

2.x版本配置：

![配置](/images/架构/配置/tomcat-session4.png)

启动即可，在redis客户端验证存储的KEY/VALUE

## 5.CAS ticket共享 

复制redis-ticket-registry.jar包到 cas/WEB-INF/libs和cas/libs目录下。

编辑cas/WEB-INF/spring-configuration/ticketRegistry.xml文件

![配置](/images/架构/配置/cas-ticket.png)
