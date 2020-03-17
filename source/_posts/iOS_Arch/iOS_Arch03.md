---
title: iOS_Arch02 | 设计模式和数据驱动
p: iOS_Arch/iOS_Arch03
date: 2020-03-12 16:16:44
tags:
---

## 前言：
这么多年的iOS开发，也或多或少接触过使用了MVVM的中型App，但是过往接触的iOS项目其实或多或少只是遵守了若干的MVVM架构的思想拆分，而在数据流扭转，双向绑定等方面是没有实现的。
* MVVM: 为了解决MVC中Controller的问题，需要将Controller的功能代码拆分到ViewModel层
* 数据流扭转和驱动(View和Model的双向绑定，VM作为数据流处理的黑盒子)

## 双向绑定
其实在Vue这样成熟的MVVM框架中，实现双向绑定的基础也离不开观察者模式。而在iOS中可以采用的成熟的观察者模式有两种方式:
* KVO
* 函数响应式

## RxSwift

## RxSwift + MVVM + ObjectMapper 
RxSwift + MVVM + ObjectMapper 构建MVVM和数据驱动层的主题架构，PromiseKit + SwiftyJSON为业务拓展做的补充
