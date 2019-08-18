---
layout: lay_post
title: "机器学习算法-相似度-PearsonCorrelationSimilarity(皮尔森相似度)"
date: 2016-11-30
categories: 相似度算法
tags: 机器学习
author: bornhe
---

## 0.概述

皮尔森相关系数反应了两个变量之间的线性相关程度，它的取值在[-1, 1]之间。当两个变量的线性关系增强时，相关系数趋于1或-1；当一个变量增大，另一个变量也增大时，表明它们之间是正相关的，相关系数大于0；如果一个变量增大，另一个变量却减小，表明它们之间是负相关的，相关系数小于0；如果相关系数等于0，表明它们之间不存在线性相关关系。
<!-- more -->

## 1.皮尔森相似度描述

用数学公式表示，皮尔森相关系数等于两个变量的协方差除于两个变量的标准差。

![数学公式](/images/算法/皮尔森相关/数学公式.png)

## 2.协方差(Covariance)

在概率论和统计学中用于衡量两个变量的总体误差。如果两个变量的变化趋于一致，也就是说如果其中一个大于自身的期望值，另一个也大于自身的期望值，那么两个变量之间的协方差就是正值；如果两个变量的变化趋势相反，则协方差为负值。

![协方差](/images/算法/皮尔森相关/协方差.png)

其中u表示X的期望E(X), v表示Y的期望E(Y)

## 3.标准差(Standard Deviation)

标准差是方差的平方根

![标准差](/images/算法/皮尔森相关/标准差.png)

## 4.方差(Variance)

在概率论和统计学中，一个随机变量的方差表述的是它的离散程度，也就是该变量与期望值的距离。即方差等于误差的平方和的期望

![方差](/images/算法/皮尔森相关/方差.png)

## 5.缺点

基于皮尔森相关系数的相似度有两个缺点：

(1) 没有考虑（take into account）用户间重叠的评分项数量对相似度的影响；

(2) 如果两个用户之间只有一个共同的评分项，相似度也不能被计算

![例子](/images/算法/皮尔森相关/例子.png)

上表中，行表示用户（1～5）对项目（101～103）的一些评分值。直观来看，User1和User5用3个共同的评分项，并且给出的评分走差也不大，按理他们之间的相似度应该比User1和User4之间的相似度要高，可是User1和User4有一个更高的相似度1。

同样的场景在现实生活中也经常发生，比如两个用户共同观看了200部电影，虽然不一定给出相同或完全相近的评分，他们之间的相似度也应该比另一位只观看了2部相同电影的相似度高吧！但事实并不如此，如果对这两部电影，两个用户给出的相似度相同或很相近，通过皮尔森相关性计算出的相似度会明显大于观看了相同的200部电影的用户之间的相似度。

Mahout对基于皮尔森相关系数的相似度给出了实现，它依赖一个DataModel作为输入。

![mahout](/images/算法/皮尔森相关/mahout.png)

同时，Mahout还针对缺点(1)进行了优化，只需要在构造PearsonCorrelationSimilarity时多传入一个Weighting.WEIGHTED参数，就能使有更多相同评分项目的用户之间的相似度更趋近于1或-1。

UserSimilarity similarity1 = new PearsonCorrelationSimilarity(model);

double value1 = similarity1.userSimilarity(1, 5);

UserSimilarity similarity2 = new PearsonCorrelationSimilarity(model, Weighting.WEIGHTED);

double value2 = similarity2.userSimilarity(1, 5);

结果：

Similarity of User1 and User5: 0.944911182523068

Similarity of User1 and User5 with weighting: 0.9655694890769175