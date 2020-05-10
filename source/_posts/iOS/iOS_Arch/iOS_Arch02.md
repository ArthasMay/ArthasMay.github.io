---
title: iOS架构思考&实践02 | App页面层级结构的组织
p: iOS/iOS_Arch/iOS_Arch02
date: 2020-03-04 13:38:14
tags:
---

## 总结
搭建一个高可用，易拓展的App的架构，对之后的业务升级和产品开发的效率都是有着积极作用的。从这篇文章开始，笔者将根据多年的开发经验总结和技术博客的认知理解，尝试搭建一个完整的app架构。
上篇文章我根据自己的理解，将App架构拆分为表现层和服务层，表现层侧重于App的UI和交互，是业务代码的直接承载层，设计优秀的表象层能够更好的服务于业务的成长

## 表现层

### 表现层的涉及内容

* 页面的组织结构和解耦: 不管是MVVM还是MVC，ViewController都是App架构最基本的单元，页面的切换和业务的流转都是以ViewController为基础，View、ViewModel、model都是围绕ViewController组织代码的
* 设计模式的选择: MVVM 数据的双向绑定（数据驱动UI，响应UI交互）
* Page的设计: Page状态管理(网络状态、上拉刷新和下拉加载更多、loading、空白占位、网络层的接入、页面数据缓存策略的设计以及骨架屏等)
* 基本Page样式的构建: 业务中常用的瀑布流（CollectionView）、表单页面(TableView)、更加复杂的嵌套结构和性能优化(IGListKit)
* Widgets的梳理: UI设计规范和高复用的UI组件库(Loading、Toast、Float、Dialog)
* 动画: 转场动画的设计和动画kit的设计
* hybird技术: flutter 和 web的容器设计
* 通信: 页面之内是通过数据双向绑定，页面之间（组件化组件之间）
* 渲染性能

## 解决千变万化的导航栏

如果从事iOS开发时间够长的话，你一定曾经导航栏虐过，特别是近两年App对UI的追求和设计，越来越多的App对页面进行了定制化设计，更改NavigationBar的交互(渐变，模糊等等)。但是如果只对UINavigationController进行简单的使用和组织，后面由于navigationbar的更改导致的问题会让你头疼不已
``` Swift
edgesForExtendedLayout
automaticallyAdjustsScrollViewInsets -> scrollV.contentInset :(UIScrollView.contentInsetAdjustmentBehavior -> UIScrollView.adjustedContentInset)
extendedLayoutIncludesOpaqueBars
navigationBar.isTranslucent
```
我相信上面的几个方法和属性都曾经为难过你，还有iOS11开始的留海。所以相信我，你需要对你项目中的导航栏设计一下

####
``` swift
一、
UIVIewController.edgesForExtendedLayout = .none（isTranslucent = true / false时都是）
非全屏布局
UIViewController.view的大小就是不会往bar下面延伸(UIViewController视图层级查看是从导航栏下面开始的)
1. 隐藏navigationbar，view的大小会根据navigationbar, tabbar的隐藏与否变化
2. 由于之前的navigationbar是一个导航栈只拥有一个，所以在各个页面切换的更改全局导航栏，会导致view的大小上上下下，而且侧滑返回的动画不好处理
3. edgesForExtendedLayout = .none， View的子视图ScrollView的contentInset不会变化，即该设置属性无效

二、
isTranslucent = true 时：
UIVIewController.edgesForExtendedLayout = UIRectEdgeAll(默认)
全屏布局
1. UIVIewController.contentInsetAdjustmentBehavior = true/ UIScrollView.contentInsetAdjustmentBehavior = .auto
自动更改scrollview的contentInset
只要上面有透明的bar(NavigatiobBar, TabBar, StatusBar), View的子视图是ScrollView就会变化，当ScrollView的Y > NavigationBar的Bottom, ScrollView的contentInset也是会偏移的，引起Bug

2. UIVIewController.contentInsetAdjustmentBehavior = false/ UIScrollView.contentInsetAdjustmentBehavior = .false
不调整ScrollView的内边距contentInset

三、extendedLayoutIncludesOpaqueBars = false（默认）-> 穿过不透明（有透明度）的bar
只在isTranslucent = false时设置有效，也是影响UIViewController的大小
extendedLayoutIncludesOpaqueBars = true 这时候contentInsetAdjustmentBehavior也会影响ScrollView的contentInset
```

总结：
1. edgesForExtendedLayout 在 isTranslucent = true(默认)时影响是否全屏布局
.all全屏是默认 .none变小
2. 三、extendedLayoutIncludesOpaqueBars 在 isTranslucent = false 时影响是否全屏布局
false非全屏默认 true全屏
3. UIVIewController.contentInsetAdjustmentBehavior = true/ UIScrollView.contentInsetAdjustmentBehavior = .auto 管理透明bar对ScrollView的contentInset的影响，非全屏（上面两种情况）下一般无表现

### 解决方法
1. 每一个ViewController拥有一个独立的UINavigationBar: 千变万化的导航栏最大的痛点就是每个RootNavigationController是共享一个bar的
2. 使用全屏布局，让UIViewController的View的frame和ScrollView的内容遵循一个规范，尺寸统一处理后不变

### RTRootNavigationController的引入
独立导航栏的处理方法很多: 
1. 隐藏全局导航，自定义View替代，这种方案简单，但是需要完全模仿一个完整的系统导航栏效果，处理粗糙的话就会细节上不过关，譬如左滑返回的阴影效果消失
2. RTRootNavigationController是github上找到的优雅的解决方案，设计精巧，代码质量也很高
### RTRootNavigationController的思路解析
```
- RTRootNavigationController
-- RTContainerController
--- RTContainerNavigationController
---- RTWrappedViewController
```

* RTRootNavigationController 的导航栈里面不能直接管理UINavigationController，所以在RTContainerNavigationController外面包一层RTContainerController容器
* RTWrappedViewController的NavigatonController和bar是RTContainerNavigationController的，导航行为代理给RTRootNavigationController管理

### 根据源码整理了一个Swift版本(但是还有一些待确定的细节，后续会做一个源码解析的总结)

### 关于pushReplace这类的方法
笔者在使用VueRouter的时候，发现pushReplace这类的方法，在实际开发的页面切换中是很有价值的：譬如引导页切换App的tab的时候，直接设置window.root有点简单粗暴的，pushReplace这类的方法是有价值的。iOS通过管理NavigationController管理的导航栈ViewControllers来在pop的时候模拟pushReplace，其实在push的时候控制对Page的解耦来说更有价值，特别是在使用路由的时候。

## 路由的引入

### 路由的作用
不管是前端还是后端，router的作用实质上市一样的，即是用来进行逻辑分发，在前端主要表现为Page的切换，在后端则作为逻辑分发的中间件。
前端引入路由可以进一步的进行解耦拆分：
* 由可组织控制的自定义scheme规则来管理页面的切换跳转
* 业务加大后，若要使用业务组件化，又路由层分发业务组件

### 路由的选择
RTURLNavigator: 基于Swift的轻量级路由
* 完全遵守URL协议
* 可以在此开源库的基础上进行业务层路由的组织

## App的主要组织结构和Router的结合
由于使用了RTRootNavigationController，需要对URLNavigator中进行页面切换的部分需要重写





