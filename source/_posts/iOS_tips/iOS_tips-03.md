---
title: iOS03 | 大话组件化
p: iOS_tips/iOS_tips-03.md
date: 2019-12-04 10:17:56
tags: 
---

## 什么是组件化开发

## 为什么需要组件化开发

## 组件化技术预研

### CocoaPods私有仓库

CocoaPods私有仓库的搭建是组件化开发一个很重要的技术点，除非是完全迁移到Swift + Carthage的团队，但是现阶段动态化的框架方案极度依赖OC的runtime能力，所以暂时CocoaPods的私有库还是主流(当然Swift私有库也是可以通过pod的framework的方式管理)。

#### CocoaPods tools规划
1. RTPrivateSpecs仓库: 管理所有私有库的podspec文件
2. RTNetwork、RTKit这类仓库: 存储私有库源码
3. 通过 GUI 管理私有仓库的 update、push和打tag等私有库的CI

#### 创建pods私有库的流程

##### 一、创建spec Repo:
1. 在Gitlab上创建Pods私有仓库: RTPrivateSpecs    
这个仓库是用来保存未来所有私有库.podspec文件，包括版本的管理

2. 添加私有库 RTPrivateSpecs 的私有Spec Repo     
  * .cocoapods/repos 中的master是官方的Spec Repo
  * 执行添加Spec Repo的命令
``` bash
# XXPrivateSpecs: 私有Spec Repo的仓库name
# https://github.com/xxx/mySpecs.git: 私有Spec Repo的仓库地址，如果团队要使用SSH, 请确保所有人已经配置好ssh
$pod repo add XXPrivateSpecs https://github.com/xxx/mySpecs.git
```

3. 创建私有库源码库:
  * 创建私有源码库的两种方式:
``` bash
# 根据Pods官方模板创建:
$pod lib create RTKit
# 自定义源码库，添加.podspec 
$pod spsc create RTKit
```

4. spec文件的配置:
spec文件是Ruby的脚本文件，这边只列举一些常用的配置
``` ruby
Pod::Spec.new do |s|
  # 私有库名
  s.name         = 'RTKit'
  # 摘要介绍
  s.summary      = 'Network layer for iOS.'
  # 版本号: 这个必须和source的tag对应好
  s.version      = '0.0.1'
  # 这个必须在git仓库配置的，不然后面执行pod repo push的时候是不会验证通过的
  s.license      = { :type => 'MIT', :file => 'LICENSE' }
  s.authors      = { 'arthas' => 'arthasmay@gmail.com' }
  s.social_media_url = 'https://arthasmay.github.io'
  s.homepage     = 'https://github.com/../RTKit'
  s.platform     = :ios, '7.0'
  s.ios.deployment_target = '10.0'
  # 源码仓库，最好是https的，防止团队有成员没有配置ssh
  s.source       = { :git => 'git@code.xxx.com:xxx/rtnetwork.git', :tag => s.version.to_s }
  
  s.requires_arc = true
  s.source_files = '*.{h,m}'
  s.public_header_files = '*.{h}'
  
  s.frameworks = 'UIKit', 'CoreFoundation' 

end
```

踩坑的地方：
  * [Podsepc语法说明](https://www.jianshu.com/p/75e19c92df50)
  * MIT license 中必须和 s.author 对应，arthas\<arthasmay@gmail.com\>，我觉得不需要
  * 后面会拆分成submodules，这块的内容后面补充

5. 上传私有库源码
``` bash
# 将本地代码关联和上传远程仓库
$git init
$git remote add origin git@code.xxx.com:xxx/rtcao.git
$git add . 
$git commit -m "Commit tag 0.0.1"
$git push -u origin master
# 打上pods版本version对应的tag
$git tag 0.0.1
$git push --tags
```

6. 校验.podspec文件

``` bash
# 在podspec所在目录执行 or 指定校验文件
$pod spec lint //[xxx]
```

7. 将依赖库的spec文件push到spec仓库
``` bash
# 仓库名 spec文件名字
$pod repo push RTPrivateSpecs RTNetwork.podspec
```
注意：
如果创建的用来存储podspec的仓库处于无文件的空状态，那在pod repo push的时候会碰到下面的问题
```
创建一个空的仓库后，git pull/push 报错,报错信息如下
Your configuration specifies to merge with the ref 'refs/heads/master' from the remote, but no such ref was fetched.
这是正常的，在本地目录添加一个文件，commit 后 push 即可。
```
[错误解决参考](https://segmentfault.com/a/1190000012787100)

8. 验证私有库的使用

``` bash
# 查看私有库的文件树：
$tree -L 2
# 文件树：
.
├── README.md
└── RTKit
    └── 0.0.1
```

9. 项目验证
在Podfile文件添加：
``` ruby
# podfile写入依赖
source https://code.xxx.com/xxx/rtprivatespecs.git
pod 'RTKit'
```
执行pod install

##### Pods私有仓库的升级(文档待完善)

#### Pods相关GUI工具(工具开发待完成)


### 动态化方案

#### 路由：逻辑(页面和业务逻辑的分发)


#### hybird开发：Web & flutter的跨平台开发


### 组件化的颗粒度选择



