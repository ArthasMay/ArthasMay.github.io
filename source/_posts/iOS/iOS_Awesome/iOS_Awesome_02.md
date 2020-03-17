---
title: iOS开发原理 | GCD
p: iOS/iOS_Awesome/iOS_Awesome_01.md
date: 2019-12-16 10:52:56
tags: 
- iOS原理
Categories: iOS

# GCD

## 什么是GCD

* GCD是Apple开发的为了充分利用iOSCPU多核编程的解决方案
* 主要用于优化应用程序以支持多核处理器以及其他对称多处理系统
* GCD是一个在线程池的基础上执行并发任务

## 为什么使用GCD

* GCD 可以使用多核并行计算
* GCD 充分利用CPU的多核
* GCD 会自动管理线程的生命周期（创建线程、任务调度、销毁线程）
* 不用管理线程，只需要任务队列

## GCD的任务和队列

### 任务
* 任务: 就是执行操作的意思，也就是在线程中执行的那段代码
* 执行任务分为同步执行和异步执行
* 异步执行具有开线程的能力，但是并不一定开启新的线程，只有在并发队列异步执行任务才具有开线程的能力

### 队列

* 队列是执行任务的等待队列，用来存放任务的队列
* 遵循FIFO原则的线性表
* 分为 Serial Dispatch Queue 和 Concurrent Dispatch Queue
  1. 串行队列：每次只有一个任务被执行，让任务一个接着一个地执行。（只开启一个线程，一个任务执行完毕后，再执行下一个任务）
  2. 并发队列：可以让多个任务并发（同时）执行。（可以开启多个线程，并且同时执行任务）

### GCD的使用

1. 创建一个队列
2. 将任务追加到队列中，然后GCD根据任务类型执行任务

#### 创建队列
```
// 创建串行队列
dispatch_queue_t sdQueue = dispatch_queue_create("com.arthas.testSerial", DISPATCH_QUEUE_SERIAL);
// 创建并行队列
dispatch_queue_t cdQueue = dispatch_queue_create("com.arthas.testConcurrent", DISPATCH_QUEUE_CONCURRENT);
```

#### GCD默认提供的串行主队列(Main Dispatch Queue)

* 所有主队列的任务都是放在主线程执行
* dispatch_get_main_queue获取主队列

#### GCD提供的全局并发队列(Global Dispatch Queue)

* dispatch_get_global_queue获取全局并发队列
* 队列优先级: 一般使用DISPATCH_QUEUE_PRIORITY_DEFAULT

##### 队列和任务的组合方式
* 同步执行 + 主队列: 死锁卡主不执行
* Tips
  1. 主线程中 调用 主队列 + 同步执行 会导致死锁， 因为主队列中追加的同步任务和主线程两者之间相互等待，阻塞了主队列， 最终造成了主队列所在的线程（主线程）死锁
  2. 在非主线程调用 主队列 + 同步执行 不会阻塞主队列，不会导致死锁问题， 结果是：不会开启线程，串行执行任务

##### 队列嵌套
* 串行队列（同步/异步）嵌套当前串行队列（同步/异步）会导致串行队列中追加的任务和原有的任务相互等待，阻塞串行队列，最终导致该串行多列所在线程（子线程）死锁的问题