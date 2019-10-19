---
title: Node.js(二) | 异步编程
p: node/node-02
date: 2019-10-19 15:53:48
tags:
- node
categories: Node.js
---

## 异步：非阻塞I/O

### 非阻塞I/O定义
* I/O 即Input/Output, 一个系统的输入和输出
* 阻塞I/O和阻塞I/O的区别在于系统接收输入再到输出期间，能不能接收其他输入

### Node架构设计

![Node.js架构图](node-02/node_arch.png)

* 系统边界：左边和LibUV（event queue + event loop）在一个node线程里面
* 其他c++工作线程
* node将任务分发给其他线程

## 异步：异步编程之callback

### Node回调函数格式规范

* error-first callback
* Node-style callback

所有callback函数都要遵循一个参数格式：第一个参数是error， 后面的参数才是结果

### 为什么需要遵循error-first范式
``` js
try {
  interview(function() {
    console.log('smile');
  })
} catch (e) {
  console.log('cry', e);
}

function interview(callback) {
  setTimeout(() => {
    if (Math.random() < 0.1) {
      callback('success')
    } else {
      throw new Error('fail')
    }
  }, 500);
}
```

* try-catch机制：try-catch是根据当前函数调用栈从栈顶往下抛的，throw 抛出的错误从栈顶往下寻找try-catch(被try-catch包裹)，如果throw的error没有被补货，就会抛到全局，node.js程序会崩溃
* 为什么上面的error会被抛到全局（throw 是在interview函数里面，被trycatch包裹的）：是因为事件循环，node里面每一个事件循环都是一个全新的调用栈，throw error是在setTimeOut函数里面，是在另一个事件循环被回调的，所以这里的try-catch是不能捕获到抛出的error的，被抛到了node.js全局的
* node回调规范： 第一个参数不为空，代表这次非阻塞I/O(异步任务调用)失败了

``` js node.js 回调规范
interview(function(res) {
  if (res instanceof Error) {
    return console.log('cry');
  }
  console.log('smile');
})

function interview(callback) {
  setTimeout(() => {
    if (Math.random() < 0.8) {
      ** 错误的时候，cb的第一个参数为null **
      callback(null, 'success')
    } else {
      callback(new Error('fail')) 
    }
  }, 500);
}
```

### 回调函数的异步流程控制的问题：回调地狱
* 串行：回调地狱
* 并行：计数器判断代码（冗余代码）

``` js 异步串行调用 callback hell
 interview(function(err) {
  if (err) {
    return console.log('cry at 1st round');
  }
  
  interview(function(err) {
    if (err) {
      return console.log('cry at 2st round');
    }

    interview(function (err) {
      if (err) {
        return console.log('cry at 3st round');
      }

      console.log('smile');
    })
  })
})
```

``` js 异步并行调用
var count = 0
interview(function(err) {
  if (err) {
    return console.log('cry')
  }
  count ++

  if (count === 2) {
    console.log('smile')
  }
})

interview(function(err) {
  if (err) {
    return console.log('cry')
  }
  count++
  if (count === 2) {
    console.log('smile')
  }
})

function interview(callback) {
  setTimeout(() => {
    if (Math.random() < 0.8) {
      callback(null, 'success')
    } else {
      callback(new Error('fail'))
    }
  }, 500);
}
```
* 之前在永辉写iOS的方式就是这样的并行调用处理，代码看起来好丑
* TODO: 后面可以在iOS里面看下异步流程控制的解决方案：阿里的coroutine框架coobjc

npm社区的解决方案：
* async.js(思路可以参考下)
* thunk 编程方式

## 异步：事件循环

## 异步：异步编程之Promise

## 异步：异步编程者async/await



