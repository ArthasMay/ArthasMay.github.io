---
title: Node.js(三) | Web框架：express & koa
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

### express的中间件能力
* 什么是中间件能力：可以将复杂的逻辑拆分成不同模块（可以分模块和文件处理，这样可以更好的维护代码）
* express中间件的逻辑拆分模式
1. 一个个拆分的模块（函数）顺序完成
2. express中间件还有执行完可以返回执行之前中间件剩余逻辑的能力（洋葱模型）

``` js
// 未使用express的洋葱模型(串行执行中间件未返回res结果给外面层)
app.get('/game', 
  function(req, res, next) {
    if (playerwon >= 3 || sameCount == 9) {
      res.status(500)
      res.send('我不会再玩了')
      return
    }
    
    // 通过next执行后续中间件
    next()
  },

  function(req, res, next) {
    const query = req.query
    const playerAction = query.action
    
    if (!playerAction) {
      res.status(400)
      res.send()
      return
    }

    if (lastAction == playerAction) {
      sameCount++
      if (sameCount >= 3) {
        res.status(400)
        res.send('你作弊！我再也不玩了')
        sameCount = 9
        return
      } 
    } else {
      sameCount = 0
    }
    lastAction = playerAction

    // 把用户操作挂在response上传递给下一个中间件
    res.playerAction = playerAction
    next()
  },

  function(req, res) {
    const playerAction = res.playerAction
    const result = game(playerAction)

    res.status(200)
    if (result == 0) {
      res.send('平局')

    } else if (result == -1) {
      res.send('你输了')

    } else {
      res.send('你赢了')
      res.playerWon = true;
    }
    
    if (res.playerWon) {
      playerwon++;
    }
  }
)
```

### 详解express中间件的洋葱模型

``` js
// 使用express的洋葱模型
app.get('/game', 
  function(req, res, next) {
    if (playerwon >= 3 || sameCount == 9) {
      res.status(500)
      res.send('我不会再玩了')
      return
    }
    
    // 通过next执行后续中间件
    next()

    // 当后续中间件执行完之后，会执行到这个位置
    if (response.playerWon) {
      playerWinCount++;
    }
  },

  function(req, res, next) {
    const query = req.query
    const playerAction = query.action
    
    if (!playerAction) {
      res.status(400)
      res.send()
      return
    }

    if (lastAction == playerAction) {
      sameCount++
      if (sameCount >= 3) {
        res.status(400)
        res.send('你作弊！我再也不玩了')
        sameCount = 9
        return
      } 
    } else {
      sameCount = 0
    }
    lastAction = playerAction

    // 把用户操作挂在response上传递给下一个中间件
    res.playerAction = playerAction
    next()
  },

  function(req, res) {
    const playerAction = res.playerAction
    const result = game(playerAction)

    res.status(200)
    if (result == 0) {
      res.send('平局')
    } else if (result == -1) {
      res.send('你输了')
    } else {
      res.send('你赢了')
      res.playerWon = true;
    }
    
    if (res.playerWon) {
      playerwon++;
    }
  }
)
```

洋葱模型：
* 顶层中间件就像洋葱外皮
* 逻辑穿过洋葱外皮进入内核中间件
* 执行完内层中间件又可以穿出到洋葱外皮中间件

express洋葱模型的作用
在顶层next()上下记录时间，可以统计所有中间件的执行时间(处理http过程总共时间)
``` js
const start = Date.now()
next()
const end = Date.now()
```

### express洋葱模型的缺陷

处理异步非阻塞操作会失效
``` js
// 模拟非阻塞I/O的操作
setTimeout(()=> {
  response.status(200);
  if (result == 0) {
      response.send('平局')
  } else if (result == -1) {
      response.send('你输了')
  } else {
      response.send('你赢了')
      response.playerWon = true;
  }
}, 500)
// 赢三次的功能失效了
```

express的洋葱缺陷原因：express对异步的支持不完善
* express是一个普通函数(未支持异步操作)，next是阻塞执行的，所有的中间件是在一个事件循环（在chrome调试台表现是在一个函数调用栈里的）中的执行的
* setTimeout or 异步操作是在另一个（下一个）事件循环中的，设置的response.playerWon在上个时间循环中已经结束了

## koa框架

* 中间件：express中间件异步洋葱模型失效，koa是可以通过async function实现中间件的，await可以中断异步执行，实现支持异步的洋葱模型
* context：挂载req，res
* 砍掉了路由，希望路由挂到中间件里面(微内核的架构思想) koa-mount


### koa代码实例


### koa核心功能
* 


## express 和 koa的对比

