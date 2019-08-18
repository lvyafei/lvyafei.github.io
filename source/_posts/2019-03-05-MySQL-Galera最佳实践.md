---
layout: lay_post
title: "MySQL-Galera最佳实践"
date: 2019-03-05
categories: MySQL
tags: [数据库]
author: lvyafei
---

## 1.查看Galera集群状态

show status like 'wsrep%';
<!--more-->
### 集群完整性检查:

wsrep_cluster_state_uuid:在集群所有节点的值应该是相同的,有不同值的节点,说明其没有连接入集群.
wsrep_cluster_conf_id:正常情况下所有节点上该值是一样的.如果值不同,说明该节点被临时”分区”了.当节点之间网络连接恢复的时候应该会恢复一样的值.
wsrep_cluster_size:如果这个值跟预期的节点数一致,则所有的集群节点已经连接.
wsrep_cluster_status:集群组成的状态.如果不为”Primary”,说明出现”分区”或是”split-brain”状况.

### 节点状态检查:

wsrep_ready: 该值为ON,则说明可以接受SQL负载.如果为Off,则需要检查wsrep_connected.
wsrep_connected: 如果该值为Off,且wsrep_ready的值也为Off,则说明该节点没有连接到集群.(可能是wsrep_cluster_address或wsrep_cluster_name等配置错造成的.具体错误需要查看错误日志)
wsrep_local_state_comment:如果wsrep_connected为On,但wsrep_ready为OFF,则可以从该项查看原因.

### 复制健康检查:

wsrep_flow_control_paused:表示复制停止了多长时间.即表明集群因为Slave延迟而慢的程度.值为0~1,越靠近0越好,值为1表示复制完全停止.可优化wsrep_slave_threads的值来改善.
wsrep_cert_deps_distance:有多少事务可以并行应用处理.wsrep_slave_threads设置的值不应该高出该值太多.
wsrep_flow_control_sent:表示该节点已经停止复制了多少次.
wsrep_local_recv_queue_avg:表示slave事务队列的平均长度.slave瓶颈的预兆.

ps:最慢的节点的wsrep_flow_control_sent和wsrep_local_recv_queue_avg这两个值最高.这两个值较低的话,相对更好.

### 检测慢网络问题:

wsrep_local_send_queue_avg:网络瓶颈的预兆.如果这个值比较高的话,可能存在网络瓶

### 冲突或死锁的数目:

wsrep_last_committed:最后提交的事务数目
wsrep_local_cert_failures和wsrep_local_bf_aborts:回滚,检测到的冲突数目

## 2.Galera最佳实践

在实际应用中，Galera集群常与haproxy配合，haproxy作为单一入口，将SQL流量分配到后端Galera，注意此处是TCP级别的，而不是SQL语句级别。如果需要SQL语句级别的，可以参考看看MaxScale。相比haproxy方案，各有应用，此处不展开。实际应用的结构通常如下图所示，haproxy作为Galera Cluster的前端，2台haproxy用keepalived避免单点故障。后端的Galera集群可以是3个节点，也可以是2个节点+1个仲裁节点。

为了避免Galera写入冲突，在haproxy配置中，实际上只允许一个节点接受写入，另一个节点带有backup参数，只有当前允许写入的节点坏掉，才会自动写入另一个节点。为了充分利用另一个节点，同时做读写分离，在haproxy上监听两个端口，例如：3307用于写入，3308用于读取。

为了避免“脑裂”，Galera集群最少需要3个节点，生产环境中最好是3台独立主机，或者位于3个不同IDC的主机。如果觉得3台主机太浪费，也可以2台数据库节点+1个虚拟机仲裁节点。仲裁节点是集群的一部分，可以参与投票，但是不参与实际复制。在3个IDC场景下，如果一个IDC自身网络中断，但假如它与仲裁节点可以连通，同时其他节点也与仲裁节点可以连通的话，那仲裁节点就会作为中间人连接另两个节点通信。

## 3.参考资料

Galera读写分离HAProxy配置：https://github.com/olafz/percona-clustercheck

参考文章：http://blog.sina.com.cn/s/blog_704836f40101lixp.html