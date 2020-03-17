---
title: iOS_interview02 | Runtime
p: iOS_interview/iOS_interview02
date: 2020-03-01 20:12:53
tags:
---

## 1. runtime

### 1.1 什么是runtime：我理解的runtime
首先解释下编译时和运行时
编译时: compiler 将高级语言编译为机器可执行的机器码
运行时: 分配内存（创建对象）和 方法调用(objc_msgSend的消息机制)
runtime是一组c/c++/汇编编写的运行时库，负责运行在运行时执行oc代码(OC是C的超集，OC对象的本质是C的结构体指针，OC方法的调用本质也是C函数的调用，runtime其实就类似提供OC向C的转译执行)
动态创建类和对象，进行消息传递和转发

### 1.2 runtime的作用：runtime能干什么
* 在运行时决定方法调用：编译时只记录方法的符号，并不在乎方法的调用。在运行时，根据方法符号寻找到方法实现，执行方法实现
解释：
1. 在Swift中类似于C++的调用，方法是存在vtable中（虚方法调用），在运行的时候就决定了调用什么方法，所以Swift宣称比OC快。
2. 那有人要问了，在实际用Swift开发iOS的时候，runtime还是可以用的，这是因为Apple到现在还没有使用swift书写任何的系统库，只是在程序中装载了Swift的最基本的实现（基本数据结构、实现），然后对iOS的Api进行了Swift化。在xcode编译时, 继承NSObject的部分仍按照oc规则编译，在Swift4之后开启优化，新增方法只有@objc标记的编译为oc规则的机器码，否则默认不记录oc类和方法符号，无法使用runtime。
3. liba：这个在dyld加载过程导入的动态库就是runtime库

* 提供method swizzle，动态创建对象，给分类绑定属性，遍历方法和属性，消息转发的OC的黑魔法能力
1. method sizzle: 替换系统方法实现：埋点，更改系统实现，解决一些bug，更改系统UI的（UITextfield）的样式， avoid crash
2. 动态创建对象：jspatch 面向切面
3. 遍历对象属性和成员变量：json 快速转model
4. Category 关联属性
5. 消息转发

## 2. 源码中runtime的一些结构体概念实现和原理

### 2.1 源码解析objc_msgSend的过程（消息机制）
``` C
OBJC_EXPORT id _Nullable
objc_msgSend(id _Nullable self, SEL _Nonnull op, ...)
```
* 先从缓存找: 汇编编写
* 没有缓存: 从方法列表里面找
* 是找方法的实现IMP: SEL 其实是方法的符号id（实际是一个唯一的字符串）

### 2.2 一些结构体

#### 2.2.1 类对象：objc_class 
``` C
// runtime.h
// objc_class
struct objc_class {
    Class _Nonnull isa  OBJC_ISA_AVAILABILITY;

#if !__OBJC2__
    // 父类指针
    Class _Nullable super_class                              OBJC2_UNAVAILABLE;
    const char * _Nonnull name                               OBJC2_UNAVAILABLE;
    long version                                             OBJC2_UNAVAILABLE;
    long info                                                OBJC2_UNAVAILABLE;
    // 实例大小
    long instance_size                                       OBJC2_UNAVAILABLE;
    // 实例列表
    struct objc_ivar_list * _Nullable ivars                  OBJC2_UNAVAILABLE;
    // 方法列表
    struct objc_method_list * _Nullable * _Nullable methodLists                    OBJC2_UNAVAILABLE;
    // cache, 方法调用缓存列表
    struct objc_cache * _Nonnull cache                       OBJC2_UNAVAILABLE;
    // 协议列表
    struct objc_protocol_list * _Nullable protocols          OBJC2_UNAVAILABLE;
#endif

} OBJC2_UNAVAILABLE;
/* Use `Class` instead of `struct objc_class *` */

// objc.h
typedef struct objc_class *Class;
```

#### 2.2.2 实例：objc_object
``` c
// objc.h
/// Represents an instance of a class.
struct objc_object {
    Class _Nonnull isa  OBJC_ISA_AVAILABILITY;
};

/// A pointer to an instance of a class.
typedef struct objc_object *id;
```
