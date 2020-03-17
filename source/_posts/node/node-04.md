---
title: Node.js(四) | RPC通信
p: node/node-04
date: 2019-10-21 01:49:18
tags:
- node
categories: Node.js
---

## RPC调用(站在前端的角度来说)

### RPC定义
* Remote Procedure Call(远程过程调用)

* 和Ajax有什么相同点

  * 都是两个计算机之间的网络通信（Ajax是 client <-> server, RPC是 server <-> server）
  * 需要双方约定一个数据格式

* 和Ajax有什么不同点

  * 一般不使用DNS作为寻址服务（http 一般使用DNS寻址服务，而RPC调用一般在内网，使用DNS划不来）
  * 应用层协议一般不使用http（RPC使用二进制协议，性能有优势)
  * 基于TCP或UDP协议（http都是基于tcp）

### 详解RPC

* 寻址/负载均衡
  * Ajax: 使用DNS进行寻址（TODO: 详解http协议簇）
  * RPC: 使用特有服务进行寻址（TODO: 画出RPC寻址示意图）
* TCP通讯方式（TODO：http的通讯方式演进）
  * 单工通讯
  * 半双工通讯
  * 全工通讯（实现难度和成本是增加的，全双工最难）
* 二进制协议
  * 更小的数据包体积
  * 更快的编解码速率

### BFF的RPC实现

* 寻址和负载均衡由运维消化
* BFF需要理解和实现多路复用和二进制协议

## RPC的二进制协议

### 详解Buffer 

#### Stream

* Stream 是对输入输出设备的抽象，这里的设备可以是文件、网络或内存
* Stream 是有方向的，文件和网络读取数据是输入流，写入文件是输出流
* 举个例子：
  我们现在有一大罐水需要浇一片菜地，如果我们将水罐的水一下全部倒入菜地，首先得需要有多么大的力气（这里的力气好比计算机中的硬件性能）才可搬得动。如果，我们拿来了水管将水一点一点流入我们的菜地，这个时候不要这么大力气就可完成。
* 流可以理解成水流这种抽象概念

#### Buffer

数据的移动肯定是为了处理和读取，每段时间的都会有最小或最大数据量
* 数据到达的速度比进程消耗的快，少数先达到的数据就会在等待区等候被处理
* 数据到达的速度比进程消耗的慢，先达到的数据需要等待一定量的数据到达之后才能被处理
这里的等待区就指的缓冲区（Buffer），它是计算机中的一个小物理单位，通常位于计算机的 RAM 中，对Buffer的操作就是
查询Buffer里面的数据

#### Protocol Buffer

* Protocol Buffer 和 XML、JSON一样都是结构数据序列化的工具，但它们的数据格式有比较大的区别：
  * PB序列化之后得到的数据不是可读字符串，而是二进制流
  * 其次，XML 和 JSON 格式的数据信息都包含在了序列化之后的数据中，不需要任何其它信息就能还原序列化之后的数据；但使用 Protocol Buffer 需要事先定义数据的格式(.proto 协议文件)，还原一个序列化之后的数据需要使用到这个定义好的数据格式（可以理解为解析规则的配置文件）
  * 最后，在传输数据量较大的需求场景下，Protocol Buffer 比 XML、JSON 更小（3到10倍）、更快（20到100倍）、使用 & 维护更简单；而且 Protocol Buffer 可以跨平台、跨语音使用

### Node.js中的Buffer模块

### protocol-buffers: Node.js 支持pb的库
``` js
const protobuf = require('protocol-buffers')
const schema = protobuf(fs.readFileSync(__dirname + '/test.proto', 'utf-8'))

console.log(schema)
// console.log(schema.toJSON)

const buffer = schema.Column.encode({
  id: 1,
  name: "Node.js",
  price: 80.4
})

console.log(schema.Column.decode(buffer))
```
由于js内核原因，浮点型会出现进度不标准的问题


### Node.js net搭建多路复用的 RPC 通道

#### 单工通信

#### 半双工通信
``` js
// server.js
const net = require('net')

const server = net.createServer((socket) => {
  socket.on('data', function (buffer) {
    // console.log(buffer, buffer.toString());
    const lessonid = buffer.readInt32BE()
    console.log(lessonid);
    
    
    setTimeout(() => {
      socket.write(
        Buffer.from(data[lessonid])
      )
    }, 10 + Math.random() * 1000)
  })
})

server.listen(4000)

const data = {
  136797: "01 | 课程介绍",
  136798: "02 | 内容综述",
  136799: "03 | Node.js是什么？",
  136800: "04 | Node.js可以用来做什么？",
  136801: "05 | 课程实战项目介绍",
  136803: "06 | 什么是技术预研？",
  136804: "07 | Node.js开发环境安装",
  136806: "08 | 第一个Node.js程序：石头剪刀布游戏",
  136807: "09 | 模块：CommonJS规范",
  136808: "10 | 模块：使用模块规范改造石头剪刀布游戏",
  136809: "11 | 模块：npm",
  141994: "12 | 模块：Node.js内置模块",
  143517: "13 | 异步：非阻塞I/O",
  143557: "14 | 异步：异步编程之callback",
  143564: "15 | 异步：事件循环",
  143644: "16 | 异步：异步编程之Promise",
  146470: "17 | 异步：异步编程之async/await",
  146569: "18 | HTTP：什么是HTTP服务器？",
  146582: "19 | HTTP：简单实现一个HTTP服务器"
}
```
``` js
// client.js
const net = require('net')

const socket = new net.Socket({})

socket.connect({
  host: '127.0.0.1',
  port: 4000
})

// socket.write('Hello, server!!!')

const lessonids = [
  "136797",
  "136798",
  "136799",
  "136800",
  "136801",
  "136803",
  "136804",
  "136806",
  "136807",
  "136808",
  "136809",
  "141994",
  "143517",
  "143557",
  "143564",
  "143644",
  "146470",
  "146569",
  "146582"
]

let buffer = Buffer.alloc(4)
let index = Math.floor(Math.random() * lessonids.length)
buffer.writeInt32BE(
  lessonids[index]
)
socket.write(buffer)

socket.on('data', (buffer) => {
  console.log(lessonids[index], buffer.toString())

  buffer = Buffer.alloc(4)
  index = Math.floor(Math.random() * lessonids.length)
  buffer.writeInt32BE(
    lessonids[index]
  )
  socket.write(buffer)
})
```
* 为什么半双工不能不停发包呢： 涉及到时序问题（并发请求会导致res包和req包匹配不上的问题，出现乱序返回包的问题）


#### 全双工通信

半双工 => 支持全双工
* 在请求加上时序号：解决返回包乱序的问题
* 解决粘包问题：粘包切分
  * 粘包：TCP底层机制会自动把同时发的包自动拼接起来一次发送

#### net模块

* 单工/半双工的通信通道搭建
* 全双工通信通道搭建
  * 关键在于应用层协议需要有标记包号的字段
  * 处理以下的情况，需要有标记包长的字段
    * 粘包
    * 不完整包
* 错误处理

## TODO: Buffer的大端和小端

## 参考
[一篇帮你彻底弄懂NodeJs中的Buffer](https://blog.csdn.net/qq_34629352/article/details/88037778)

