---
layout: lay_post
title: "机器学习算法-相似度-HammingDistance(汉明距离)"
date: 2016-11-30
categories: 相似度算法
tags: 机器学习
author: cnblogs
---

## 0.概述

两个等长字符串s1与s2之间的汉明距离定义为将其中一个变为另外一个所需要作的最小替换次数。例如字符串“1111”与“1001”之间的汉明距离为2。
<!-- more -->

## 1.应用场景

信息编码（为了增强容错性，应使得编码间的最小汉明距离尽可能大）。

![公式](/images/算法/汉明距离/公式.png)

## 2.Octave实例

![octave](/images/算法/汉明距离/octave.png)

![octave_result](/images/算法/汉明距离/octave_result.png)