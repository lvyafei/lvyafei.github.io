---
layout: lay_post
title: "使用Archetype创建Maven项目骨架"
date: 2020-05-09
categories: archetype
tags: maven
author: lvyafei
---

## 1.Archetype插件介绍

Archetype插件提供的4个可以直接使用的goal

```
archetype:create 已经过时不用了
archetype:generate 这是Archetype插件最常用的功能，用以从模板创建一个Maven项目
archetype:create-from-project 用以从一个Maven项目创建模板
archetype:crawl 在一个指定的Maven库中查找可以的模板，并更新模板目录
```
<!--more-->

## 2.自定义Archetype模板

从原有项目创建archtype项目，进入到原有项目的根目录执行命令：

```
# mvn archetype:create-from-project 
```

此时会在项目target下生成一些文件,再生成archetype模板

```
# cd target/generated-sources/archetype/
```

然后执行插入到本地repository中 

```
# mvn install 
```

## 3.在Idea中添加Archetype

```
<groupId>zts.intelli.app</groupId>
<artifactId>zts-intelli-app-parent-archetype</artifactId>
<version>1.16.2-SNAPSHOT</version>
```

## 4.使用Archetype创建项目骨架

```
group: zts.transactions.business
artif: zts-transactions-business-parent
version：1.0.1-SNAPSHOT
```

## 5.删除Archetypes

自定义添加的Archetypes需要移除，需要手动删除文件

```
C:\Users${user}.IntelliJIdea${version}\system\Maven\Indices\UserArchetypes.xml 
```

## 6.参考文章：

Apache Maven项目提供的Archetype插件详解:
https://blog.csdn.net/taiyangdao/article/details/51249367

Maven 自定义archeType：
https://www.jianshu.com/p/724a9fa7b37a