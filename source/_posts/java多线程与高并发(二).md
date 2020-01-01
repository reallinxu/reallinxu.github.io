---
title: "java多线程与高并发---(二)JUC"
tags:
  - java多线程与高并发
id: 1001
categories:
  - java多线程与高并发
date: 2020-1-1 16:00:00
---

## volatile
1. 保证线程可见性
2. 禁止指令重排序
3. 单例模式双重检查，还是要加volatitle，问题可能会出在指令重排序上。 jvm new对象分为三步，申请内存--->构建成员变量-->赋值给对象，加了volatile才能保证赋值完成才会返回new出的对象。
4. 锁细化，前后都有逻辑，只有中间几步需要加锁，不加在方法，而加在这几步上
5. 锁粗化，很多个锁的时候直接整体锁，锁争用减少
6. 只能保证可见性，不能保证原子性，因为写的时候不能保证，比如count++内部指令是分几步的，高并发读的时候还是会出错。还是得用sychoronised


## CAS(Compare and set)
1. 无锁优化 自旋
2. juc中Atomic开头都是CAS保证的
3. ABA问题  加version版本号 AtomicStampedReference解决 基础类型不影响，但是如果Object等引用，过程中改变了引用中的值，就会有问题 比如你的女朋友跟你复合，但是中间已经经历了很多，已经不是原来的女朋友了。。。

## unsafe类
1. 直接操作内存 allocateMemory 分配内存
freeMemory 释放内存
2. 直接生产类实例
3. 直接操作类或实例变1量
4. CAS相关操作
5. jdk1.8是不可以直接用的，11已经可以直接使用

## LongAdder
分段锁

## ReentrantLock
1. lock 只能unlock，否则无法停止
2. unlock 一般放finally里
3. trylock
4. lockInterruptibly 跟lock相比，他可以调用interrupt()方法进行打断
5. new ReentrantLock(true) 为公平锁，默认不公平

## ReentrantLock vs sychoronised
1. cas vs sync
2. trylock
3. lockinterruptibly
4. 公平和非公平
5. 可以替代synchronized

## CountDownLatch
1. 相当于一个门栓，wait等倒数，等变成0时打开，用来等待线程结束，比join更灵活
2. await():等待变为0
3. countDown()：值减一

## CyclicBarrier
await等待线程达到一个指定数量开始放行，人满发车，可以定义一个到达后执行的线程，比如需要等待其他线程完成后才执行


## phaser
按阶段执行
1. 自定义一个类继承Phaser,重写onAdvance方法，数量到达后自动调用，第一个参数为第几个阶段(从0开始)，第二个参数为此阶段剩余多少线程，arriveAndDeregister会移除注册
2. bulkRegister(n) 执行到达一个阶段的线程数量
3. arriveAndAwaitAdvance() 到达后等待，等指定数量后执行
4. arriveAndDeregister()
解除注册，和调用此方法的线程无关系。
5. register() 注册一个

## ReadWriteLock
读写锁，即共享锁和排他锁
1. readLock() 获取读锁
2. WriteLock() 获取写锁

## Semaphore
1. new Semaphore(n) 指定信号量，允许n个线程同时运行
2. new Semaphore(n，true) 第二个参数为true为公平，默认不公平
2. acquire() 取得信号量，信号量会减1
3. release() 释放信号量，信号量加1
4. 作用是限流，一般也用Guava RateLimiter

## Exchanger
1. exchange(String) 两个线程间交换数据,返回交换后的数据
2. exhange可以设置交换超时时间







