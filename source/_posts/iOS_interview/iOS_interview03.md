---
title: iOS_interview03 | RunLoop详解
p: iOS_interview/iOS_interview03.md
date: 2020-03-24 11:19:47
tags:
---

## 大纲
1. 问题背景
2. RunLoop的认知
3. RunLoop的使用
4. RunLoop在性能优化层面的使用

## 问题背景

之前因为面试或多或少接触过RunLoop，也在滑动列表的NSTimer使用时了解过RunLoop的Mode，但是其实对RunLoop的认知还是比较浅层的。最近在接触了Node.js的EventLoop的工作原理，也在阅读YYAsyncLayer源码进行页面性能优化的过程中看到YY大神在RunLoop的Waiting | Exit之前的异步渲染操作，忽然感觉到底层的知识框架不应该知识理论，而是需要在底层的认识之上服务于我们的代码和框架。所以在此以一个专题拓展的方式对RunLoop进行一些再学习，提高认知，以期在未来也可以更好的使用。

## RunLoop的认知

### 什么是RunLoop

RunLoop是iOS/MacOS中EventLoop模型的具体实现，是和线程紧密相关的基础组件。

一般来讲，一个线程单次只能执行一个任务，线程在执行完之后就会退出。如果需要一个机制，线程在处理完事件之后并不退出，而是在等待后面的消息时间，那么我们通常需要实现下面的代码逻辑:
``` Swift
func loop() {
  initialize()
  do {
    var message = get_next_message()
    process_message(message)
  } while (message != quit)
}
// 但是上面的代码在等待消息的CPU时候是“忙”的状态，runloop需要实现的是在没有消息的时候是一个闲置的等待，减少对CPU资源的等待
```
上面的代码模型就是EventLoop: Node.js中的EventLoop和iOS/MacOS中的RunLoop，这个模型实现的关键点在于管理事件/消息，如何在线程没有消息的时候处于休眠以减少对资源的占用，在有消息的时候立即被唤醒。

所以RunLoop的定义: runloop实际上就是一个对象，负责管理事件和消息
1. 提供一个入口函数来执行上面的EventLoop的逻辑，线程在执行完入口函数后会一直处于“接受消息->处理->等待”的循环中，直到收到quit的事件退出循环。
2. 苹果提供了开源版本的CFRunLoopRef，iOS/MacOS中额外提供NSRunLoop（但是线程不安全）也有Swift可以使用的跨平台版本

### RunLoop和线程之间的关系

首先iOS开发中遇到的线程对象：pthread和NSThread，之前官方文档是说NSThread是基于pthread的封装，但是后面删除了文档，有可能二者都是基于最底层Unix的mach_thread封装的，两者的关系是一一对应（虽然不能转换）
RunLoop是基于pthread来管理的

iOS中不能直接创建RunLoop对象，提供了CFRunLoopGetMain() 和 CFRunLoopGetCurrent()两个函数来自动获取RunLoop对象

获取RunLoop对象注释代码
``` C
/// 全局的Dictionary: key 是 pthread_t, value 是 CFRunLoopRef
static CFMutableDictionaryRef loopsDic;
/// 访问loopsDic时的锁
static CFSpinLock_t loopsLock;

/// 获取一个 pthread 对应的 runloop
CFRunLoopRef _CGRunLoopGet(pthread_t thread) {
  OSSpinLockLock(&loopslock);

  if (!loopsDic) {
    /// 第一次进入时，初始化全局Dic，并为主线程创建一个 RunLoop
    loopsDic = CFDictionaryCreateMutable();
    CFRunLoopRef mainloop = _CFRunLoopCreate();
    CFDictionarySetValue(loopsdic, pthread_main_thread_np(), mainloop);
  }

  /// 直接从 Dictionary 中获取RunLoop
  CFRunLoopRef loop = CFDinctionaryGetValue(loopsdic, thread);

  if (!loop) {
    /// 获取不到，创建一个新的 RunLoop
    loop = _CFRunLoopCreate();
    CFDictionarySetValue(loopsdic, thread, loop);
    /// 注册一个回调函数，当线程销毁时，也顺便销毁对应的 
    RunLoop_CFSetTSD(..., thread, loop, __CFFinalizeRunLoop);
  }
  OSSpinLockUnLock(&loopsLock);
  return loop
}

CFRunLoopRef CFRunLoopGetMain() {
    return _CFRunLoopGet(pthread_main_thread_np());
}
 
CFRunLoopRef CFRunLoopGetCurrent() {
    return _CFRunLoopGet(pthread_self());
}
```

* 线程和RunLoop是一一对应的，其对应关系保存在全局的Dictionary loopsdic中
* 子线程创建后默认是没有runloop的，只有在主动获取CFRunLoopGetCurrent()时才会创建
* RunLoop的创建只在第一次获取的时候，随着线程销毁而销毁
* mainloop（主线程的loop）在 App的主函数入口main()中创建，在App运行期间不会销毁，除了主线程的RunLoop对象可以在任意一个线程内部获取，其余都只能在当前线程内部获取对应的RunLoop

### RunLoop对外的接口
CFRunLoopRef
CFRunLoopModeRef
CFRunLoopSourceRef
CFRunLoopTimerRef
CGRunLoopObsercerRef


### iOS中RunLoop实现的功能

#### @autoreleasepool的实现

App启动后，runloop注册了两个observer, 回调函数都是_wrapRunLoopWithAutoreleasePoolHandler()
observer1:_objc_autoreleasePoolPush() 
observer2:_objc_autoreleasePoolPush() , _objc_autoreleasePoolPop()

在主线程的代码，事件回调和Timer回调都是给主线程创建的runloop包围着，不需要显式创建autoreleasepool，不会出现内存泄漏

#### 事件响应
iOS注册了一个source1（基于mach-port的）用来接收系统事件，回调函数是__IOHIDEventSystemClientQueueCallback()。
当一个硬件事件（touch、lock、motion）发生后， IOKit.framework生成一个IOHIDEvnet的事件并有springboard接受。SpringBoard只接受按键（锁屏/静音等）、touch、加速和接近传感器等event，随后通过mach-port转发给需要的app进程。随后注册的source1唤起runloop，触发回调函数，并调用_UIApplicationHandleEventQueue()进行应用内事件分发。

_UIApplicationHandleEventQueue()会把IOHIDEvent包装成UIEvent进行处理和分发，其中包括识别 UIGesture/处理屏幕旋转/发送给 UIWindow 等。通常事件比如 UIButton 点击、touchesBegin/Move/End/Cancel 事件都是在这个回调中完成的

#### 手势识别
当上面的 _UIApplicationHandleEventQueue() 识别了一个手势时，其首先会调用 Cancel 将当前的 touchesBegin/Move/End 系列回调打断。随后系统将对应的 UIGestureRecognizer 标记为待处理。

苹果注册了一个 Observer 监测 BeforeWaiting (Loop即将进入休眠) 事件，这个Observer的回调函数是 _UIGestureRecognizerUpdateObserver()，其内部会获取所有刚被标记为待处理的 GestureRecognizer，并执行GestureRecognizer的回调。

当有 UIGestureRecognizer 的变化(创建/销毁/状态改变)时，这个回调都会进行相应处理。
#### 界面更新

CA的事务是在主线程RunLoop的beforeWait | exit 之前提交事务给render server（单独的一条渲染进程）

当在操作 UI 时，比如改变了 Frame、更新了 UIView/CALayer 的层次时，或者手动调用了 UIView/CALayer 的 setNeedsLayout/setNeedsDisplay方法后，这个 UIView/CALayer 就被标记为待处理，并被提交到一个全局的容器去。

苹果注册了一个 Observer 监听 BeforeWaiting(即将进入休眠) 和 Exit (即将退出Loop) 事件，回调去执行一个很长的函数：
_ZN2CA11Transaction17observer_callbackEP19__CFRunLoopObservermPv()。这个函数里会遍历所有待处理的 UIView/CAlayer 以执行实际的绘制和调整，并更新 UI 界面。

#### 定时器

NSTimer 其实就是 CFRunLoopTimerRef，他们之间是 toll-free bridged 的。一个 NSTimer 注册到 RunLoop 后，RunLoop 会为其重复的时间点注册好事件。例如 10:00, 10:10, 10:20 这几个时间点。RunLoop为了节省资源，并不会在非常准确的时间点回调这个Timer。Timer 有个属性叫做 Tolerance (宽容度)，标示了当时间点到后，容许有多少最大误差。

如果某个时间点被错过了，例如执行了一个很长的任务，则那个时间点的回调也会跳过去，不会延后执行。就比如等公交，如果 10:10 时我忙着玩手机错过了那个点的公交，那我只能等 10:20 这一趟了。

CADisplayLink 是一个和屏幕刷新率一致的定时器（但实际实现原理更复杂，和 NSTimer 并不一样，其内部实际是操作了一个 Source）。如果在两次屏幕刷新之间执行了一个长任务，那其中就会有一帧被跳过去（和 NSTimer 相似），造成界面卡顿的感觉。在快速滑动TableView时，即使一帧的卡顿也会让用户有所察觉。Facebook 开源的 AsyncDisplayLink 就是为了解决界面卡顿的问题，其内部也用到了 RunLoop，这个稍后我会再单独写一页博客来分析。

#### PerformSelecter
当调用 NSObject 的 performSelecter:afterDelay: 后，实际上其内部会创建一个 Timer 并添加到当前线程的 RunLoop 中。所以如果当前线程没有 RunLoop，则这个方法会失效。

当调用 performSelector:onThread: 时，实际上其会创建一个 Timer 加到对应的线程去，同样的，如果对应线程没有 RunLoop 该方法也会失效。

#### GCD

当调用 dispatch_async(dispatch_get_main_queue(), block) 时，libDispatch 会向主线程的 RunLoop 发送消息，RunLoop会被唤醒，并从消息中取得这个 block，并在回调 __CFRUNLOOP_IS_SERVICING_THE_MAIN_DISPATCH_QUEUE__() 里执行这个 block。但这个逻辑仅限于 dispatch 到主线程，dispatch 到其他线程仍然是由 libDispatch 处理的。

#### 网络请求


## RunLoop在性能优化层面的使用

~系统的Core Animation层提交给CPU的计算图层的事务（transaction）~ 准确的说是提交给独立的render server的进程，就是在监听RunLoop的kCFRunLoopBeforeWaiting | kCFRunLoopExit（在休眠和退出之前）时提交的

ASDK(Texture) 和 YYKit中的 YYAsyncLayer就是和Core Animation（QuartzCore/UIKit）的设计模式，实现了一套类似的界面更新机制：即在主线程RunLoop中添加一个Observer，监听了 kCFRunLoopBeforeWaiting 和 kCFRunLoopExit 事件，在收到回调时，遍历所有之前放入队列的待处理的事务，然后一一执行。

这里由于ASDK设计更加复杂，YYAsyncLayer的源码比较少，可以简单明了的揭示runloop在UIKit优化过程的使用。

### YYAsyncLayer介绍

YYAsyncLayer是YY大神YYKit中的实现一组关于异步绘制的组件，是为了列表的优化而设计的
* YYAsyncLayer：继承于CALayer，用来异步渲染layer内容的
* YYSentinel：线程安全计数，用于多线程使用场景
* YYTransaction：在主线程RunLoop中添加一个Observer，监听了 kCFRunLoopBeforeWaiting 和 kCFRunLoopExit 事件， 主要是在主线程进入休眠之前触发预设的方法(但是这边的优先级 < Core Animation提交的事务)

在本文中主要是探讨RunLoop的使用，所以这里主要是看怎么在MainLoop中添加Observer，监听MainLoop休眠事件，触发回调处理事件，而这部分涉及的异步渲染则会在之后由优化的文章来介绍。

参照CoreAnimation和ASDK的Runloop Work Distribution: 设置 UIView 的 frame、修改 CALayer 的透明度、为视图添加一些动画；这些操作最终会被 CALayer 捕获，并通过 CATransaction 提交到一个中间状态去（CATransaction 的文档中有提到这写内容，但并不完整）。当上面的所有操作结束后，Runloop 即将进入休眠或退出时，关注该事件的 Observer 都会得到通知，这时 CA 注册的那个 Observer 就会在回调中把所有的中间状态合并提交到 GPU 去显示；如果此处有动画，CA 会通过 DisplayLink 等机制多次触发相关流程
YYTransaction就是为了方便YYAsyncLayer在绘制过程中将上面捕捉到的中间状态合并到TransactionSet中, 在主线程runloop即将进入休眠时统一处理更新这些中间转态，而不是单次调用阻塞线程的其他操作

设置YYTransaction的observer优先级0xFFFFFF < CA的observer的优先级200 0000，所以YYTransaction提交的任务是不会影响CA提交的事务


``` Swift
/// Swift 版本的 YYTransaction
```
