---
title: SwiftTips01 | Swift的一些语法特性
p: iOS/iOS_SwiftTips/iOS_SwiftTips01.md
date: 2020-05-10 21:59:24
tags:
- Swift语法特性
categories: SwiftTips
---

## 一、闭包

### 逃逸闭包(@escaping)
* 逃逸闭包的概念：将closure当做参数传入一个函数中，但是这个closure在函数执行完之后才在特定时机触发执行（OC的block Copy捕获作为Stack block时也是为了异步调用）
* 逃逸闭包的表现形式
``` Swift
// 1. 类实例存储属性捕获，RxSwift中Observale Observer的创建使用了大量的escaping closure
class Demo {
  let _handler: (Any) -> String

  init(_ eventHandler: @escaping (Any) -> String) {
    _handler = eventHandler
  }
}
// 2. 外部变量捕获
var completionHandlers: [() -> Void] = []
func someFunctionWithEscapingClosure(completionHandler: @escaping () -> Void) {
  completionHandlers.append(completionHandler)
}
```