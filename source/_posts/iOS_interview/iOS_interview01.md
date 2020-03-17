---
title: iOS_interview01 | UI相关
p: iOS_interview/iOS_interview01
date: 2020-03-01 15:18:33
tags:
---

## 1. UIView 和 CALayer


## 2. iOS事件传递机制和响应链机制

### (1) 事件的传递
  1. 事件的产生：
    * 发生事件（touch, motion, remote 讨论的最多的是touch），系统会把产生的touch event 加入到UIApplication管理的队列中（FIFO）
    * UIApplication将事件队列中最前面的事件取出分发，通常是分发给keywindow
    * keywindow会在视图层级里面寻找最合适的视图来处理触摸事件，找到合适的视图就会调用该视图的touches相关方法来处理时间
  2. 事件的传递
    * touch event从父控件传递给子控件
    * UIApplication -> UIWindow -> 寻找处理时间最合适的view
    * 不能传递touch event的情况：父控件不能传递，子控件不能传递
      (1) 不允许交互 userInteractionEnabled
      (2) hidden
      (3) alpha < 0.01
  3. 寻找最合适的View
    * keywindow接受Application传递的事件，判断自己能否接受事件，如果能判断touch point是否在window之上
    * window能响应事件并且touch point在自身之上，从后往前遍历子控件，寻找最合适的view来响应事件
    * 找到子控件，重复上面步骤（1.判断是否响应事件， 2.判断触摸点位置）
    * 从后往前循环遍历子视图，直到找到合适的view，若未找到，自己就是最合适的view
  4. 底层原理的解析：UIView的两个重要Api
  ``` swift
    open func hitTest(_ point: CGPoint, with event: UIEvent?) -> UIView?
    open func point(inside point: CGPoint, with event: UIEvent?) -> Bool
  ```
    * func hitTest(_ point: CGPoint, with event: UIEvent?) -> UIView?
    1. 非override的话：返回最适合的子控件, 子控件不是的话返回自己
    2. override的话：选择返回需要响应的子控件
    3. 返回nil，本身那个和子控件都不是fitview，判断兄弟视图，如果兄弟视图也不是, superview
    
    * open func point(inside point: CGPoint, with event: UIEvent?) -> Bool
    1. 判断touch point是否在控件的响应范围内
     
    * 利用上面一个Api可以做一些控件的热区限制
  5. 写一个可以热区扩大的按钮

### 响应链
* UIApplication UIWindow UIViewController UIView 都是UIResponder的子类
* 在时间分发中找到fitview后，fitview通过responder chain: view -> viewcontroller -> window -> application 寻找可以响应事件的对象，处理事件。若到了最上层application也没有处理，该事件给抛弃，从event queue里面删除
* touches
``` swift
// UIResponder的方法
open func touchesBegan(_ touches: Set<UITouch>, with event: UIEvent?)
open func touchesMoved(_ touches: Set<UITouch>, with event: UIEvent?)
open func touchesEnded(_ touches: Set<UITouch>, with event: UIEvent?)
open func touchesCancelled(_ touches: Set<UITouch>, with event: UIEvent?)
```
未override时：调用super touches相关方法将事件传递到responder chain的上一层
overrider：定制响应操作， 如果在override的过程中未调用super touches，响应链不再往上层传递
* gesture 和 button 都是对touch event的响应
* next responder属性
```
open var next: UIResponder? { get }
```
1. 可以构建基于UIViewController的多层级视图之间轻量级通信机制，减少代理和通知的使用。
2. 但是在使用过程中发现，如果window上添加子视图遮盖ViewController的话，点击上面window的视图，ViewController上如果有基于响应链的通信，也是会触发，从而出现Bug

任务：实现上面通信机制的swift版本，并且验证是否有Bug和bug问题所在

## iOS渲染原理
