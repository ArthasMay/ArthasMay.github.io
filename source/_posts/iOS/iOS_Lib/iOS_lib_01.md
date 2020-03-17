---
title: iOS_lib_01 | Moya的使用和源码分析
p: iOS/iOS_Lib/iOS_lib_01
date: 2020-03-10 16:37:11
tags:
---

## Moya

## Moya的使用
Moya的基本使用方法很简单:
* 创建TargetType的数据结构: 通常是enum
* 创建MoyaProvider实例, 调用request方法，传入对应的TargetType

基本的使用方法就不赘述了，这篇文章主要是记录下Moya的思想， 以及如何在App的现行架构里面引入Moya，并且方便的代码的维护和拓展，所以在这里侧重想关注下面几点
* Moya的Pipe机制和插件机制
* Moya如何定制通用业务请求
* Moya + RxSwift（RxMoya的使用）

## Moya的Pipe和Plugin

## Moya如何定制通用业务请求

在实际项目开发中，如何维护和拓展网络层代码是很必要的
* APIService的模块化，集中式管理和拓展（规范大于思想，规范就是为了团队项目在合理的层次统一管理单一功能的代码并维护和拓展）
* APIService 和 App架构设计
* 业务处理：特殊业务(token失效的code码的统一处理)、Log日志追踪、DNS优化


