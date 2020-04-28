---
title: iOS_interview04 | iOS中多线程开发
p: iOS_interview/iOS_interview04.md
date: 2020-03-29 12:41:05
tags:
---

## iOS中多线程开发

### 线程的工作原理
CPU通过轮转时间片的机制来切换线程的调用，而线程对象的创建和切换是有资源消耗的，所以“线程爆炸”这样的多线程使用反而会影响性能

### iOS中线程的概念
iOS中线程对象有p_thread和NSThread， 这是直接的线程对象，如果要使用的话需要手动创建、管理和销毁线程，在实际的iOS开发中一般使用相关的mainThread、currentThread获取线程消息。

### GCD
GCD是iOS针对自家CPU多核架构设计的线程调度的中间件（理解类似于后端开发的线程调度器），充分利用多核性能，避免开发者调用繁琐的底层线程相关的API，开发者通过向系统管理的调度队列中提交任务，在多核硬件上同时执行代码

#### GCD中一些的概念
* 任务
  1. 同步任务会阻塞当前线程，并在当前线程执行任务
  2. 异步线程不会阻塞当前线程，具有开启线程的能力，在新的线程中执行任务
* 队列：
  1. 串行队列只能使用同一个线程，依次执行多个任务
  2. 并行队列能够使用多个线程，并在与当前线程不同的线程执行任务
* 死锁： 在串行队列中（包括主队列）嵌套同步任务

* 栅栏任务（barrier）
* 迭代任务

#### 多线程相关面试题
1. iOS开发中有多少类型的线程？分别对比 
p_thread NSThread 基于最底层Unix的mach_thread封装的
2. GCD有哪些队列，默认提供哪些队列

3. GCD有哪些方法api GCD主线程 & 主队列的关系

4. 如何实现同步，有多少方式就说多少
group barrier semaphore
5. dispatch_once实现原理

6. 什么情况下会死锁，有哪些类型的线程锁，分别介绍下作用和使用场景

7. NSOperationQueue中的maxConcurrentOperationCount默认值 
-1
8. NSTimer、CADisplayLink、dispatch_source_t 的优劣
