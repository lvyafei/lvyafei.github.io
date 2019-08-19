---
layout: lay_post
title: "MQ-重试(RETRY)-死信(DLQ)"
date: 2019-06-10
categories: 中间件
tags: MQ
author: lvyafei
---

## 1.RocketMQ消息消费失败的处理策略

Consumer从Broker拉取到消息之后进行消费，但是消费并不一定都是顺利的，不可避免会遇到一些异常情况，这种情况下RocketMQ提供了怎样的处理机制？以PushConsumer为例来看下与之相关的操作。
<!--more-->

## 2.Consumer消费异常处理流程

在Consumer使用的时候需要注册MessageListener，对于PushConsumer来说需要注册MessageListenerConcurrently，其中消费消息的接口会返回处理状态，分别是，

ConsumeConcurrentlyStatus.CONSUME_SUCCESS，消费成功
ConsumeConcurrentlyStatus.RECONSUME_LATER，推迟消费

MessageListener是在ConsumeMessageConcurrentlyService中被调用的，可以看到上述两个状态会分别映射到CMResult定义的枚举值，

CMResult.CR_SUCCESS，消费成功
CMResult.CR_LATER，推迟消费
CMResult.CR_ROLLBACK，事务消息回滚
CMResult.CR_COMMIT，事务消息投递
CMResult.CR_THROW_EXCEPTION，消费过程异常
CMResult.CR_RETURN_NULL，消费结果状态为null
消息消费的结果会在ConsumeMessageConcurrentlyService.processConsumeResult中进行处理。

从代码看返回ConsumeConcurrentlyStatus.RECONSUME_LATER状态之后的处理策略是将该组消息发送回Broker，等待后续消息。发送回的消息会设置重试Topic，重试Topic命名为："%RETRY%" + Consumer组名。原先实际的Topic会暂存到消息属性当中，以及设置delayLevel以及reconsumeTimes。

Consumer消费的时候可以设置consumeMessageBatchMaxSize来控制传入MessageLisenter的消息数量，这里的失败处理策略是，其中只要有一条消息消费失败就认为全部失败，这一批消息都会发送回Broker。因此consumeMessageBatchMaxSize这个值的设置需要注意，否则容易出现消息重复消费问题。

## 3.Broker消费失败消息处理流程

Broker端对应的处理位于SendMessageProcessor.consumerSendMsgBack方法中。对于Consumer发送失败返回的消息，Broker会将其放入重试Topic中。

重试消息的重新投递逻辑与延迟消息一致，等待delayLevel对应的延时一到，Broker会尝试重新进行投递处理。

DelayLevel对应的延时级别是固定的，RocketMQ对应的配置为MessageStoreConfig.messageDelayLevel，默认的级别为，

1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h
鉴于RocketMQ的实现机制，可以去调整每一个级别对应的时间，但可以看出一是时间精度不够细，二是级别为固定级别。

**自定义消费失败处理**

Consumer对消费失败可以进行一定程度的介入，默认失败后的级别设置为msg.getReconsumeTimes() + 1，如果Consumer明确知道未来可以成功消费的时间，那么就可以主动去设置重试次数与重试级别来进行控制。

**顺序消息非顺序消息的差异**
在消息失败处理上，顺序消息与非顺序消息是有明显差异的。对于顺序消息来说，如果消费失败后将其延迟消费，那么顺序性实际就被破坏掉了。

所以顺序消息消费失败的话，消息消费不会再推进，知道失败的消息消费成功为止。

## 4.死信
RocketMQ中的消息无法无限次重新消费，当然了，手动修改重试次数是可以的，不介入的话不行。当重试次数超过所有延迟级别之后。消息会进入死信，死信Topic的命名为：%DLQ% + Consumer组名。

进入死信之后的消息肯定不会再投递了，不过可以通过接口去查询当前RocketMQ中私信队列的消息。如果在上层实现自有命令，那么可以将消息从死信中移出并重新投递。

## 5.参考说明

转载：
http://blog.soliloquize.org/2018/09/02/RocketMQ-%E6%B6%88%E6%81%AF%E6%B6%88%E8%B4%B9%E5%A4%B1%E8%B4%A5%E7%9A%84%E5%A4%84%E7%90%86%E7%AD%96%E7%95%A5/