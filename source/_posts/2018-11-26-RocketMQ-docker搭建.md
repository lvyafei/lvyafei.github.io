---
layout: lay_post
title: "RocketMQ-docker搭建"
date: 2018-11-26
categories: 中间件
tags: MQ
author: lvyafei
---

## 0.基础准备

在Docker中搭建RocketMQ集群环境。
<!-- more -->

jdk1.8下载地址：

https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html

maven 下载地址:

http://archive.apache.org/dist/maven/maven-3/3.5.2/binaries/apache-maven-3.5.2-bin.tar.gz

rocketmq-4.3.2下载地址：

https://github.com/apache/rocketmq/archive/rocketmq-all-4.3.2.tar.gz

rocketmq-console下载:

https://github.com/apache/rocketmq-externals/archive/rocketmq-console-1.0.0.zip

配置maven镜像仓库地址:

```xml
<mirrors>
    <mirror>
          <id>alimaven</id>
          <name>aliyun maven</name>
          <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
          <mirrorOf>central</mirrorOf>        
    </mirror>
</mirrors>
```
参考资料(rocketmq集群介绍)：

https://blog.csdn.net/leexide/article/details/80035470

https://roc-wong.github.io/blog/2018/11/RocketMQ-at-a-glance.html

关闭防火墙:

```shell
# systemctl stop firewalld
```

## 1.RocketMQ 集群环境搭建

### 1.1 RocketMQ编译

java,maven环境变量首先配置正确。

然后更改本地maven的镜像仓库地址,最后进入到RocketMQ的源代码文件夹中,执行编译

```shell
# mvn -Prelease-all -DskipTests clean install -U
```

执行完后，将 rocketmq-rocketmq-all-4.3.2/distribution/target 目录下的 apache-rocketmq 文件夹 copy 到 /opt 或应用存放的目录中。

### 1.2 更改默认的JVM配置

修改namesrv的配置：

```shell
# vim bin/runserver.sh
...
JAVA_OPT="${JAVA_OPT} -server -Xms4g -Xmx4g -Xmn2g -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
...
```
修改broker的配置：

```shell
# vim /bin/runbroker.sh
...
JAVA_OPT="${JAVA_OPT} -server -Xms8g -Xmx8g -Xmn4g"
...
```

### 1.3 broker配置修改

broker的配置文件在 config/broker.conf中，其中 config/2m-2s-async 为多Master多Slave模式(异步复制)集群配置，config/2m-2s-sync为多Master多Slave模式(同步双写)集群配置，config/2m-noslave 为多Master集群。每种集群模式的优缺点见参考资料。

每种配置的关键配置项，如下：

```properties
#集群名称(相同名称的brokerName,根据brokerId来区别主从)
brokerName=broker-a|broker-b

#0表示Master,>0表示Slave
brokerId=0

#Broker在集群的角色(和borkerId对应)
#- ASYNC_MASTER 异步复制Master
#- SYNC_MASTER 同步双写Master
#- SLAVE
brokerRole=ASYNC_MASTER

#刷盘方式(2m-2s-async与2m-2s-sync的区别点就是这个配置)
#- ASYNC_FLUSH 异步刷盘
#- SYNC_FLUSH 同步刷盘
flushDiskType=ASYNC_FLUSH
```

以上内容修改完毕后，还需要添加以下配置:

```properties
# Broker的IP地址
brokerIP1 = xxxxxx

# NameServer地址列表，多个nameServer地址用分号隔开
namesrvAddr = 10.29.183.66:9876;10.29.183.81:9876;10.29.183.82:9876;10.29.183.83:9876

# 是否自动创建Topic,若关闭，客户端在使用一个未事先创建好的Topic会报错，打开改配置则会自动创建
autoCreateTopicEnable = true
```

### 1.4 启动namesrv,broker

**启动namesrv：**

```shell
# nohup sh bin/mqnamesrv > /dev/null 2>&1 &
```

查看日志:

```shell
# tail -f ~/logs/rocketmqlogs/namesrv.log
```

**启动broker:**

方式一：手动指定namesrv地址:
```shell
# nohup sh bin/mqbroker -n localhost:9876 > /dev/null 2>&1 &
```

方式二:指定配置文件(推荐):
```shell
# nohup sh bin/mqbroker -c conf/broker.conf > /dev/null 2>&1 &
```

查看日志：

```shell
# tail -f ~/logs/rocketmqlogs/broker.log
```

### 1.5 停止namesrv,broker

```shell
# sh bin/mqshutdown namesrv

# sh bin/mqshutdown broker
```
### 1.6 测试Producer,Consumer

首先设置namesrv的地址

```shell
# export NAMESRV_ADDR=localhost:9876

```
测试发送消息:

```shell
sh bin/tools.sh org.apache.rocketmq.example.quickstart.Producer
```

测试接受消息:
```shell
sh bin/tools.sh org.apache.rocketmq.example.quickstart.Consumer
```

### 1.7 mqadmin功能一览

```txt
   # sh bin/mqadmin

   updateTopic          Update or create topic
   deleteTopic          Delete topic from broker and NameServer.
   updateSubGroup       Update or create subscription group
   deleteSubGroup       Delete subscription group from broker.
   updateBrokerConfig   Update broker's config
   updateTopicPerm      Update topic perm
   topicRoute           Examine topic route info
   topicStatus          Examine topic Status info
   topicClusterList     get cluster info for topic
   brokerStatus         Fetch broker runtime status data
   queryMsgById         Query Message by Id
   queryMsgByKey        Query Message by Key
   queryMsgByUniqueKey  Query Message by Unique key
   queryMsgByOffset     Query Message by offset
   printMsg             Print Message Detail
   printMsgByQueue      Print Message Detail
   sendMsgStatus        send msg to broker.
   brokerConsumeStats   Fetch broker consume stats data
   producerConnection   Query producer's socket connection and client version
   consumerConnection   Query consumer's socket connection, client version and subscription
   consumerProgress     Query consumers's progress, speed
   consumerStatus       Query consumer's internal data structure
   cloneGroupOffset     clone offset from other group.
   clusterList          List all of clusters
   topicList            Fetch all topic list from name server
   updateKvConfig       Create or update KV config.
   deleteKvConfig       Delete KV config.
   wipeWritePerm        Wipe write perm of broker in all name server
   resetOffsetByTime    Reset consumer offset by timestamp(without client restart).
   updateOrderConf      Create or update or delete order conf
   cleanExpiredCQ       Clean expired ConsumeQueue on broker.
   cleanUnusedTopic     Clean unused topic on broker.
   startMonitoring      Start Monitoring
   statsAll             Topic and Consumer tps stats
   allocateMQ           Allocate MQ
   checkMsgSendRT       check message send response time
   clusterRT            List All clusters Message Send RT
   getNamesrvConfig     Get configs of name server.
   updateNamesrvConfig  Update configs of name server.
   getBrokerConfig      Get broker config by cluster or special broker!
   queryCq              Query cq command.
   sendMessage          Send a message
   consumeMessage       Consume message
```

### 1.8 broker.conf主要配置

```properties
#===================主要配置===============

#所属集群名字
brokerClusterName=rocketmq-cluster

#broker名字，注意此处不同的配置文件填写的不一样
brokerName=broker-a|broker-b

#0表示Master，>0表示Slave
brokerId=0

#nameServer地址，分号分割
namesrvAddr=192.168.1.101:9876;192.168.1.102:9876

#限制的消息大小
maxMessageSize=65536
#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000

#Broker 的角色
#- ASYNC_MASTER 异步复制Master
#- SYNC_MASTER 同步双写Master
#- SLAVE
brokerRole=ASYNC_MASTER
#刷盘方式
#- ASYNC_FLUSH 异步刷盘
#- SYNC_FLUSH 同步刷盘
flushDiskType=ASYNC_FLUSH

#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4

#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true

#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true

#Broker 对外服务的监听端口
listenPort=10911

#============持久化配置=================

#删除文件时间点，默认凌晨 4点
deleteWhen=04
#文件保留时间，默认 48 小时
fileReservedTime=120

#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=/data/rocketmq/store
#commitLog 存储路径
storePathCommitLog=/data/rocketmq/store/commitlog
#消费队列存储路径存储路径
storePathConsumeQueue=/data/rocketmq/store/consumequeue
#消息索引存储路径
storePathIndex=/data/rocketmq/store/index
#checkpoint 文件存储路径
storeCheckpoint=/data/rocketmq/store/checkpoint
#abort 文件存储路径
abortFile=/data/rocketmq/store/abort

#===================线程池配置====================
checkTransactionMessageEnable=false
#发消息线程池数量
sendMessageThreadPoolNums=128
#拉消息线程池数量
pullMessageThreadPoolNums=128
```

## 2.Docker使用指南

### 2.1 docker常用命令

```shell
# yum install -y docker  //安装docker

# service docker start  //启动docker

# service docker status //查看docker服务状态

# service docker stop  //停止docker服务

# systemctl reload docker //刷新docker配置
```

```shell
# docker ps -a  //检查docker运行状态

# docker start rocketmq  //启动容器

# docker stop rocketmq  //停止容器

# docker rm 容器ID/Name  //删除容器

# docker rm `docker ps -a -q` //删除所有容器
```

```shell
# docker images //查看本地镜像列表

# docker rmi 镜像名称 //删除镜像

# docker rmi $(docker images -q) //删除所有镜像
```

### 2.2 编辑docker仓库地址

编辑仓库地址: vim /etc/docker/daemon.json

```json
{
"registry-mirrors": ["https://registry.docker-cn.com"],
"insecure-registries":["10.25.24.161:5000","10.29.139.47:5000","10.29.139.47:5001"]
}
```
参数详细说明：https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-user-namespace-options

### 2.3 获取基础镜像

```shell
# docker pull 10.29.139.47:5001/base/java
```

从docker hub上获取centos基础镜像

```shell
# docker pull centos:latest
```

### 2.4 docker run 启动容器

启动容器

```shell
# docker run ...
```

--name:容器实例名称

--privileged:使用该参数，container内的root拥有真正的root权限。

-td：后台运行容器（--tty，--detach）

-it 创建一个交互式的容器

-p 映射端口(本机的端口：映射的容器的端口)

-v 挂载目录/root/software:/software(本地目录：容器目录)，在创建前容器是没有software目录的，docker 容器会自己创建

--privileged=true 关闭安全权限，否则你容器操作文件夹没有权限

>资源限制：

--ulimit memlock=-1：容器使用内存不受限制

```txt
docker容器资源限制默认：

–default-ulimit nofile=131072 
–default-ulimit memlock=128849018880 
–default-ulimit core=-1 
–default-ulimit nproc=-1 
–default-ulimit stack=-1
```
>网络设置：

--net：网络指定

```txt
docker容器网络选择：

1. host模式： docker run 使用 --net=host指定
docker使用的网络实际上和宿主机一样

2. container模式：使用 --net=container:container_id/container_name
多个容器使用共同的网络，看到的ip是一样的。

3. none 模式: 使用 --net=none指定
这种模式下，不会配置任何网络。

4. bridge模式: 使用 --net=bridge指定
默认模式，不会指定。此模式会为每个容器分配一个独立的network namespace
```

使用示例:

启动镜像，指定端口映射:

```shell
# docker run --name rocketmq-tmp -p 98761:98761 --privileged --ulimit memlock=-1 --net=host -td rockemq-base1 /usr/sbin/init
```

启动一个镜像，映射目录:
```shell
# docker run -it -p 8070:8080 -v /root/software:/software --privileged=true  docker.io/centos /bin/bash
```

### 2.5 docker exec 进入容器

方式一：exec

```shell
# docker exec -it kungfu-devel bash
```

方式二：attach

注意：如果从这个stdin中exit，会导致容器的停止。

```shell
# docker attach cd24b1147c40

# docker attach mycentos
```

### 2.6 docker commit 保存镜像

保存镜像：
```shell
docker commit afcaf46e8305 rocketmq-cluster
```

查看镜像的详细信息：

```shell
docker inspect centos-vim:afcaf46e8305
```

### 2.6 docker push 推送镜像到远端

标记与推送：
```shell
# docker tag docker.io/centos 10.29.139.47:5001/jiagou_registry/centos7

# docker push 10.29.139.47:5001/jiagou_registry/centos7
```

需要先登录:

```shell
# docker login -u你的docker账号 -p你的docker密

Login Succeeded
```

### 2.7 docker build 从dockerfile构建镜像

```shell
# docker build -t image_name -f ./Dockerfile .
```

## 3.使用Docker搭建RocketMQ集群

### 3.1 构建基础镜像 rocketmq:base

1.拉取基础镜像

docker pull centos:latest

docker tag docker.io/centos 10.29.139.47:5001/jiagou_registry/centos7

docker tag 10.29.139.47:5001/jiagou_registry/centos7 centos7

2.启动容器

docker run --name rocketmq --privileged --ulimit memlock=-1 -v /root/soft:/data/soft --net=host -td centos7 /usr/sbin/init

3.在容器中配置java,maven环境

```txt
JAVA_HOME=/opt/jdk1.8.0_191
M2_HOME=/opt/apache-maven-3.5.2
ROCKETMQ_HOME=/opt/apache-rocketmq
```

4.编译rocketmq代码,将编译过后的文件copy到/opt下，最终rocketmq的路径为：/opt/apache-rocketmq

5.在/opt/apache-rocketmq/config/目录下,创建namesrv.conf配置文件

```shell
# vim namesrv.conf

listenPort=9877
```

6.修改.bashrc文件

```shell
vim ~/.bashrc
.....
if [ -f /etc/bashrc ]; then
        . /etc/bashrc
source /etc/profile
fi
....
```

7.保存镜像

在centos7的容器中，执行exit方法，退出容器

查看刚才容器的id

```shell
# docker ps -a
```

保存，

```shell
# docker commit d4bd7510bdb2 rocketmq:base
```

### 3.2 构建namesrv镜像 rocketmq:namesrv

创建目录：/root/dockerfiles/rocketmq-namesrv,进入目录执行，

```shell
# vim Dockerfile

FROM rocketmq:base

ENV JAVA_HOME=/opt/jdk1.8.0_191
ENV M2_HOME=/opt/apache-maven-3.5.2
ENV ROCKETMQ_HOME=/opt/apache-rocketmq
ENV PATH=$PATH:$JAVA_HOME/bin:$M2_HOME/bin
ENV CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

CMD cd /opt/apache-rocketmq/ \
 && /opt/apache-rocketmq/bin/mqnamesrv -c /opt/apache-rocketmq/conf/namesrv.conf
```

在Dockerfile的目录下，执行

```shell
# docker build -t rocketmq:namesrv -f ./Dockerfile .
```

### 3.3 构建broker镜像 rocketmq:broker

创建目录：/root/dockerfiles/rocketmq-broker,进入目录执行，

```shell
# vim Dockerfile

FROM rocketmq:base

ENV JAVA_HOME=/opt/jdk1.8.0_191
ENV M2_HOME=/opt/apache-maven-3.5.2
ENV ROCKETMQ_HOME=/opt/apache-rocketmq
ENV PATH=$PATH:$JAVA_HOME/bin:$M2_HOME/bin
ENV CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar

CMD cd /opt/apache-rocketmq/ \
 && /opt/apache-rocketmq/bin/mqbroker -c /config/broker.conf
```

在Dockerfile的目录下，执行

```shell
# docker build -t rocketmq:broker -f ./Dockerfile .
```

### 3.4 启动namesrv

```shell
# docker run -d -p 9877:9877 --name rmqnamesrv rocketmq:namesrv
```

### 3.5 启动broker

创建以下配置目录：

/opt/mqconfig/broker-a-config/broker.conf

```shell
vim broker.conf

#所属集群名字
brokerClusterName=rocketmq-cluster

#broker名字，注意此处不同的配置文件填写的不一样
brokerName=broker-a

#0表示Master，>0表示Slave
brokerId=0

brokerIP1 = 10.29.183.66
#nameServer地址，分号分割
namesrvAddr = 10.29.183.66:9877

#限制的消息大小
maxMessageSize=65536
#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000

#Broker 的角色
#- ASYNC_MASTER 异步复制Master
#- SYNC_MASTER 同步双写Master
#- SLAVE
brokerRole=ASYNC_MASTER
#刷盘方式
#- ASYNC_FLUSH 异步刷盘
#- SYNC_FLUSH 同步刷盘
flushDiskType=ASYNC_FLUSH

#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4

#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true

#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true

#Broker 对外服务的监听端口
listenPort=20911

============持久化配置=================

#删除文件时间点，默认凌晨 4点
deleteWhen=04
#文件保留时间，默认 48 小时
fileReservedTime=120

#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=/data/rocketmq/store
#commitLog 存储路径
storePathCommitLog=/data/rocketmq/store/commitlog
#消费队列存储路径存储路径
storePathConsumeQueue=/data/rocketmq/store/consumequeue
#消息索引存储路径
storePathIndex=/data/rocketmq/store/index
#checkpoint 文件存储路径
storeCheckpoint=/data/rocketmq/store/checkpoint
#abort 文件存储路径
abortFile=/data/rocketmq/store/abort
```
/opt/mqconfig/broker-a-s-config/broker.conf

```shell
vim broker.conf

#所属集群名字
brokerClusterName=rocketmq-cluster

#broker名字，注意此处不同的配置文件填写的不一样
brokerName=broker-a

#0表示Master，>0表示Slave
brokerId=1

brokerIP1 = 10.29.183.66
#nameServer地址，分号分割
namesrvAddr = 10.29.183.66:9877

#限制的消息大小
maxMessageSize=65536
#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000

#Broker 的角色
#- ASYNC_MASTER 异步复制Master
#- SYNC_MASTER 同步双写Master
#- SLAVE
brokerRole= SLAVE
#刷盘方式
#- ASYNC_FLUSH 异步刷盘
#- SYNC_FLUSH 同步刷盘
flushDiskType=ASYNC_FLUSH

#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4

#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true

#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true

#Broker 对外服务的监听端口
listenPort=30911

============持久化配置=================

#删除文件时间点，默认凌晨 4点
deleteWhen=04
#文件保留时间，默认 48 小时
fileReservedTime=120

#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=/data/rocketmq/store
#commitLog 存储路径
storePathCommitLog=/data/rocketmq/store/commitlog
#消费队列存储路径存储路径
storePathConsumeQueue=/data/rocketmq/store/consumequeue
#消息索引存储路径
storePathIndex=/data/rocketmq/store/index
#checkpoint 文件存储路径
storeCheckpoint=/data/rocketmq/store/checkpoint
#abort 文件存储路径
abortFile=/data/rocketmq/store/abort
```

/opt/mqconfig/broker-b-config/broker.conf

```shell
vim broker.conf

#所属集群名字
brokerClusterName=rocketmq-cluster

#broker名字，注意此处不同的配置文件填写的不一样
brokerName=broker-b

#0表示Master，>0表示Slave
brokerId=0

brokerIP1 = 10.29.183.66
#nameServer地址，分号分割
namesrvAddr = 10.29.183.66:9877

#限制的消息大小
maxMessageSize=65536
#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000

#Broker 的角色
#- ASYNC_MASTER 异步复制Master
#- SYNC_MASTER 同步双写Master
#- SLAVE
brokerRole=ASYNC_MASTER
#刷盘方式
#- ASYNC_FLUSH 异步刷盘
#- SYNC_FLUSH 同步刷盘
flushDiskType=ASYNC_FLUSH

#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4

#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true

#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true

#Broker 对外服务的监听端口
listenPort=40911

============持久化配置=================

#删除文件时间点，默认凌晨 4点
deleteWhen=04
#文件保留时间，默认 48 小时
fileReservedTime=120

#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=/data/rocketmq/store
#commitLog 存储路径
storePathCommitLog=/data/rocketmq/store/commitlog
#消费队列存储路径存储路径
storePathConsumeQueue=/data/rocketmq/store/consumequeue
#消息索引存储路径
storePathIndex=/data/rocketmq/store/index
#checkpoint 文件存储路径
storeCheckpoint=/data/rocketmq/store/checkpoint
#abort 文件存储路径
abortFile=/data/rocketmq/store/abort
```

/opt/mqconfig/broker-b-s-config/broker.conf

```shell
vim broker.conf

#所属集群名字
brokerClusterName=rocketmq-cluster

#broker名字，注意此处不同的配置文件填写的不一样
brokerName=broker-b

#0表示Master，>0表示Slave
brokerId=1

brokerIP1 = 10.29.183.66
#nameServer地址，分号分割
namesrvAddr = 10.29.183.66:9877

#限制的消息大小
maxMessageSize=65536
#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000

#Broker 的角色
#- ASYNC_MASTER 异步复制Master
#- SYNC_MASTER 同步双写Master
#- SLAVE
brokerRole= SLAVE
#刷盘方式
#- ASYNC_FLUSH 异步刷盘
#- SYNC_FLUSH 同步刷盘
flushDiskType=ASYNC_FLUSH

#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4

#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true

#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true

#Broker 对外服务的监听端口
listenPort=50911

============持久化配置=================

#删除文件时间点，默认凌晨 4点
deleteWhen=04
#文件保留时间，默认 48 小时
fileReservedTime=120

#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=/data/rocketmq/store
#commitLog 存储路径
storePathCommitLog=/data/rocketmq/store/commitlog
#消费队列存储路径存储路径
storePathConsumeQueue=/data/rocketmq/store/consumequeue
#消息索引存储路径
storePathIndex=/data/rocketmq/store/index
#checkpoint 文件存储路径
storeCheckpoint=/data/rocketmq/store/checkpoint
#abort 文件存储路径
abortFile=/data/rocketmq/store/abort
```

启动master-a:
```shell
# docker run -d -p 20911:20911 -p 20909:20909 -v=/opt/mqconfig/broker-a-config:/config --name rmqbroker-a rocketmq:broker
```

启动slave-a:
```shell
# docker run -d -p 30911:30911 -p 30909:30909 -v=/opt/mqconfig/broker-a-s-config:/config --name rmqbroker-a-s rocketmq:broker
```

启动master-b:
```shell
# docker run -d -p 40911:40911 -p 40909:40909 -v=/opt/mqconfig/broker-b-config:/config --name rmqbroker-b rocketmq:broker
```

启动slave-b:
```shell
# docker run -d -p 50911:50911 -p 50909:50909 -v=/opt/mqconfig/broker-b-s-config:/config --name rmqbroker-b-s rocketmq:broker
```

### 3.6 启动console

拉取镜像：

```shell
# docker pull styletang/rocketmq-console-ng
```

启动容器，指定--link,-p

```shell
# docker run -d -e "JAVA_OPTS=-Drocketmq.namesrv.addr=rmqnamesrv:9877 -Dcom.rocketmq.sendMessageWithVIPChannel=false" --link rmqnamesrv:rmqnamesrv -p 8080:8080 --name rmqconsole -t docker.io/styletang/rocketmq-console-ng
```

### 3.7 检查服务状态

查看容器状态:

```shell
# docker ps -a
```

浏览器访问：

http://10.29.183.66:8080/

![1](/images/rocketmq/1.png)

![2](/images/rocketmq/2.png)

![3](/images/rocketmq/3.png)

## 4.docker瘦身

参考网址：

http://dockone.io/article/8174