---
layout: lay_post
title: "机器学习算法-分类-NeuralNetwork(神经网络)"
date: 2016-11-24
categories: 分类算法
tags: 机器学习
author: lvyafei
---

## 0.概述

当我们面对一个有许多特征和复杂假设函数的问题时，神经网络提供了一种替代的方式来执行机器学习。例如，对拥有大量像素的图片进行分类识别时，大量的特征值(每个像素)和复杂的假设函数(特征值的N次方组合)使其它监督学习算法执行效率和表现上都不太良好,神经网络能够提供一个高效的学习模型。
<!-- more -->

## 1.假设函数

![假设函数](/images/算法/神经网络/hfunction.png)

![假设函数-向量化](/images/算法/神经网络/vector.png)

![多分类](/images/算法/神经网络/multi-class.png)

## 2.代价函数

![代价函数](/images/算法/神经网络/costfunction.png)

## 3.向后传播

![向后传播](/images/算法/神经网络/backpropagation.png)

## 4.向后传播-实现

![向后传播](/images/算法/神经网络/backpropagation-impl.png)

## 5.向前-向后传播示意图

![向后-向前](/images/算法/神经网络/backpropagation-pic.png)

## 6.其他

![other](/images/算法/神经网络/other.png)