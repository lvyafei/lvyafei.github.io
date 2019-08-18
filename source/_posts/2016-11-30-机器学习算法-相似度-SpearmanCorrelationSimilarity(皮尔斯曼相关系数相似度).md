---
layout: lay_post
title: "机器学习算法-相似度-SpearmanCorrelationSimilarity(皮尔斯曼相关系数相似度)"
date: 2016-11-30
categories: 相似度算法
tags: 机器学习
author: bornhe
---

## 0.概述

皮尔斯曼相关度的计算舍弃了一些重要信息，即真实的评分值。但它保留了用户喜好值的本质特性——排序（ordering），它是建立在排序（或等级，Rank）的基础上计算的。
<!-- more -->

## 1.皮尔斯曼相关性描述

皮尔斯曼相关性可以理解为是排列后（Rank）用户喜好值之间的Pearson相关度。《Mahout in Action》中有这样的解释：假设对于每个用户，我们找到他最不喜欢的物品，重写他的评分值为“1”；然后找到下一个最不喜欢的物品，重写评分值为“2”，以此类推。然后我们对这些转换后的值求Pearson相关系数，这就是Spearman相关系数。

## 2.举例

User1～5对Item101～103的喜好（评分）值，通过斯皮尔曼相关系数计算出的相似度为：

![例子](/images/算法/皮尔斯曼/例子.png)

我们发现，计算出来的相似度值要么是1，要么是-1，因为这依赖于用户的喜好值和User1的喜好值是否趋于“一致变化”还是呈“相反趋势变化"。

Mahout对斯皮尔曼相关系数给出了实现，具体可参考SpearmanCorrelationSimilarity，它的执行效率不是非常高，因为斯皮尔曼相关性的计算需要花时间计算并存储喜好值的一个排序（Ranks），具体时间取决于数据的数量级大小。正因为这样，斯皮尔曼相关系数一般用于学术研究或者是小规模的计算。

代码：

![代码](/images/算法/皮尔斯曼/代码.png)

考虑到Spearman Correlation的效率，可以把SpearmanCorrelationSimilarity包装一层Cache，具体做法为：

UserSimilarity similarity2 = new CachingUserSimilarity(new SpearmanCorrelationSimilarity(model), model);

这样，每次计算的结果会直接放入Cache，下一次计算的时候可以立即得到结果，而不是重新再计算一次。