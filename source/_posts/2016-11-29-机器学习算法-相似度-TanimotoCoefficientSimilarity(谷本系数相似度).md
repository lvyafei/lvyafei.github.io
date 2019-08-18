---
layout: lay_post
title: "机器学习算法-相似度-TanimotoCoefficientSimilarity(谷本系数相似度)"
date: 2016-11-29
categories: 相似度算法
tags: 机器学习
author: bornhe
---

## 0.概述

它不关心用户对物品的具体评分值是多少，它在关心用户与物品之间是否存在关联关系。Tanimoto Coefficient依赖于用户和物品之间的这种Boolean关系作为输入。
<!-- more -->

## 1.谷本系数描述

Tanimoto Coefficient主要用于计算符号度量或布尔值度量的个体间的相似度，因为个体的特征属性都是由符号度量或者布尔值标识，因此无法衡量差异具体值的大小，只能获得“是否相同”这个结果，所以Tanimoto Coefficient只关心个体间共同具有的特征是否一致这个问题。Tanimoto Coefficient又被叫做Jaccard Coefficient，其值等于两个用户共同关联（不管喜欢还是不喜欢）的物品数量除于两个用户分别关联的所有物品数量。

![公式](/images/算法/谷本系数/公式.png)

其值介于[0, 1]之间，如果两个用户关联的物品完全相同，交集等于并集，值为1；如果没有任何关联，交集为空，值为0。
