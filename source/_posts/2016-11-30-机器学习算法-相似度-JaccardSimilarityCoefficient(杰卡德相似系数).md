---
layout: lay_post
title: "机器学习算法-相似度-JaccardSimilarityCoefficient(杰卡德相似系数)"
date: 2016-11-30
categories: 相似度算法
tags: 机器学习
author: cnblog
---

## 0.概述

两个集合A和B的交集元素在A，B的并集中所占的比例，称为两个集合的杰卡德相似系数，用符号J(A,B)表示。杰卡德相似系数是衡量两个集合的相似度一种指标。与杰卡德相似系数相反的概念是杰卡德距离(Jaccard distance)。杰卡德距离可用如下公式表示：1-J(A,B)
<!-- more -->

## 1.公式

![公式](/images/算法/杰卡德相似系数/公式.png)

## 2.Octave实例

![octave](/images/算法/杰卡德相似系数/octave.png)

![octave_result](/images/算法/杰卡德相似系数/octave_result.png)