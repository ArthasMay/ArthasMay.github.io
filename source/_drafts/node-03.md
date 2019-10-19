---
title: Node.js(二) | Web框架：express & koa
p: node/node-03
date: 2019-10-19 20:44:38
tags:
- node
categories: Node.js
---

## express框架

### express官方列举的features

* Robust routing
* Focus on high performance
* Super-high test coverage
* HTTP helpers (redirection, caching, etc)
* View system supporting 14+ template engines
* Content negotiation
* Executable for generating applications quickly

解读
1. Robust routing：健壮的路由
  * 后端路由系统：一个请求包进来，服务器会把这个请求包分发给不同的逻辑单元处理，这个分发的过程就是路由
  * express 可以快速实现路由

2. Focus on high performance：高性能 & Super-high test coverage 单元测试覆盖
  * 设计思路关注高性能，舍弃了一些复杂的功能
  * 完整的单元测试功能覆盖

3. HTTP helpers (redirection, caching, etc)：处理重定向和缓存
   Content negotiation：内容协商
  * 重定向的规范：
  ```
  1. server返回302状态码
  2. 带上location：redirect url的header
  ```
  * cache 304的缓存机制：（**TODO：必须知道**）
  * Content negotiation 识别accept头，返回符合accept的数据格式

4. View system supporting 14+ template engines：模板引擎

5. Executable for generating applications quickly：脚手架

1，3，4 功能是为了减轻开发负担
5 为了快速上手，对于大项目价值不大

### express核心功能：路由
* request/response简化
  * request: pathname、query等
  * response: send()、json()、jsonp()等


