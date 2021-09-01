---
title: "RocketMq事务回查"
id: 2101
categories:
  - rocketmq
date: 2020-04-23 20:17:01
tags: 问题系列
---
## RocketMq事务回查

### 问题

rocketmq发送事务消息后，如果事务没有返回实际结果COMMIT/ROLLBACK，就会触发到消息回查，消息回查的时间是多少呢？消息回查是怎么做的呢？



### 开搞

1. 部署个本地rocketmq： 略

2. 新建个springboot项目：略

3. 根据官网加依赖，随便改改代码如下：

   ```java
   package com.example.study;
   
   import org.apache.rocketmq.client.producer.LocalTransactionState;
   import org.apache.rocketmq.client.producer.TransactionListener;
   import org.apache.rocketmq.common.message.Message;
   import org.apache.rocketmq.common.message.MessageExt;
   
   import java.util.Date;
   
   /**
    * 事务监听
    * 
    * @author reallinxu
    */
   public class TransactionListenerImpl implements TransactionListener {
   
       @Override
       public LocalTransactionState executeLocalTransaction(Message msg, Object arg) {
           return LocalTransactionState.UNKNOW;
       }
   
       @Override
       public LocalTransactionState checkLocalTransaction(MessageExt msg) {
           System.out.println("我回来回查啦：" + new Date());
           return LocalTransactionState.UNKNOW;
       }
   }
   ```

   ```java
   package com.example.study;
   
   import org.apache.rocketmq.client.exception.MQClientException;
   import org.apache.rocketmq.client.producer.TransactionListener;
   import org.apache.rocketmq.client.producer.TransactionMQProducer;
   import org.apache.rocketmq.client.producer.TransactionSendResult;
   import org.apache.rocketmq.common.message.Message;
   import org.apache.rocketmq.remoting.common.RemotingHelper;
   
   import java.io.UnsupportedEncodingException;
   import java.util.Date;
   import java.util.concurrent.*;
   
   /**
    * 消息发送者
    *
    * @author reallinxu
    */
   public class TransactionProducer {
       public static void main(String[] args) throws MQClientException, InterruptedException, UnsupportedEncodingException {
           TransactionListener transactionListener = new TransactionListenerImpl();
           TransactionMQProducer producer = new TransactionMQProducer("please_rename_unique_group_name");
           ExecutorService executorService = new ThreadPoolExecutor(2, 5, 100, TimeUnit.SECONDS, new ArrayBlockingQueue<Runnable>(2000), new ThreadFactory() {
               @Override
               public Thread newThread(Runnable r) {
                   Thread thread = new Thread(r);
                   thread.setName("client-transaction-msg-check-thread");
                   return thread;
               }
           });
           producer.setNamesrvAddr("localhost:9876");
           producer.setExecutorService(executorService);
           producer.setTransactionListener(transactionListener);
           producer.start();
           Message msg = new Message("test", "tag", "key",
                   ("Hello RocketMQ ").getBytes(RemotingHelper.DEFAULT_CHARSET));
           TransactionSendResult transactionSendResult = producer.sendMessageInTransaction(msg, null);
           System.out.println(transactionSendResult);
           System.out.println("发送消息完成：" + new Date());
           // 休息休息，为了后面看回查日志
           Thread.sleep(1000000);
           producer.shutdown();
       }
   }
   ```




### 实际结果现象

![结果](/imgs/结果.png)

这是运行了几次后的结果，可以看出以下几个现象

   + 从第一次回查后，后续的回查时间都是间隔1min
   + 第一次运行后，后面的运行都是按照第一次运行的回查秒数一起进行回查
   + 运行后的第一次回查在一定时间范围内不会跟随第一次回查，会跟着1min后的一次回查



### 带着现象看源码

首先进rocketmq源码，既然checkLocalTransaction是实现了TransactionListener的方法，那自然从这里作为入口，看看是哪里调用的，顺腾摸瓜。

![事务1](/imgs/事务1.png)

首先找到DefaultMQProducerImpl#checkTransactionState，然后看到这里是将方法加到线程池checkExecutor中运营，那我们继续往前找，从哪里调用了这个方法。

![事务2](/imgs/事务2.png)

一路往前找到ClientRemotingProcessor#processRequest，这里就不能再往前了，再往前就是netty了，那是从哪里发送过来的呢，这里正好有一个枚举，我们根据枚举入手找到发送的地方。

![事务3](/imgs/事务3.png)

找到了Broker2Client#checkProducerTransactionState为发送check回查触发的地方。

![事务4](/imgs/事务4.png)

老规矩，顺腾摸瓜，找到了真正执行的地方TransactionalMessageServiceImpl#check方法，上图最上方可以看出，从RMQ_SYS_TRANS_HALF_TOPIC这个topic中拿出Set<MessageQueue>，遍历消息去check事务，中间会涉及到一些校验，回查次数校验等都是在这个方法里，注意其中removeMap会过滤掉一些消息，从RMQ_SYS_TRANS_OP_HALF_TOPIC（在半消息被commit或者rollback处理后，会存储到Topic为RMQ_SYS_TRANS_OP_HALF_TOPIC的队列中，标识半消息已被处理）中最新的offset开始拉一次32条消息，如果offeset小于当前的offset则处理过放入doneOpOffset，未处理过的放入removeMap。当removeMap有的说明已经处理过了直接过滤掉，放入doneOpOffset。

![事务5](/imgs/事务5.png)

![事务6](/imgs/事务6.png)

继续往前我们就可以看到，每次执行完默认会等待1min（transactionCheckMax参数）执行下一次，默认6s（transactionTimeOut参数）为事务检查的最小时间，默认最大检查次数为15次（transactionCheckMax参数）



### 搞定

经过上面的流程下来，我们再去看发现的现象，都可以找到对应的答案，是不是很容易就能找到？本文没有过多源码讲解，大家可以根据代码自行进行阅读（其实就是自己懒，有些也懒得看），好久没发博客了，水一波，bye~



