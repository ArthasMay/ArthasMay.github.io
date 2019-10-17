---
title: node-01 | CommonJS模块规范 
p: node/node-01
date: 2019-10-17 12:51:47
tags:
- node 
---

## CommonJS规范

### (script标签)加载脚本的缺点
* 脚本变多时，需要手动管理加载程序
* 不同脚本之间的逻辑调用，需要通过全局变量的方式
* 没有html怎么办？ node.js

### CommonJS模块规范
* Node.js应用并推广，CommonJS是同步加载的，因为server的脚本文件都是在本地，同步加载的block的影响不大
* 后续也影响到浏览器端js的书写，借助webpack

### require & exports / module.exports

```
  # require是有返回值的，exports or module.exports
  var lib = require('./lib')
  
  # exports可以认为是module的默认的属性，没有声明module.exports = {/* 对象、函数对象 */}，require的输出# 就是exports对象
  exports.hello = 'world' 
  
  # module.exports = {}, require返回的就是后面导出的对象，exports可以被理解为覆盖了
```

### Webpack怎么实现CommonJS模块规范

```bash
# 安装全局webpack，
$ webpack --devtool none --mode development --target node index.js
```

### 查看dist/main.js的源码

```
(function(modules) { // webpackBootstrap
  // The module cache
  var installerModules = {};
  
  function __webpack_require__(moduleId) {// moduleId就是path
    // Check if module is in cache
    if (installerModules[moduleId]) {
      return installerModules[moduleId].exports;
    }
    // Create a new module(and put it into the cache)
    var module = installerModules[moduleId] = {
      i: moduleId,
      l: false,
      exports: {}
    }

    // Execute the module function         
    modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
    
    // Flag the module as loaded
    module.l = true;
    
    // Return the exports of the module
    return module.exports;
  }

  // expose the modules object (__webpack_modules__)
  __webpack_require__.m = modules;
  
  // expose the module cache
  __webpack_require__.c = installerModules;

 	// define getter function for harmony exports
  __webpack_require__.d = function(exports, name, getter) {
    if(!__webpack_require__.o(exports, name)) {
      Object.defineProperty(exports, name, { enumerable: true, get: getter });
    }
  };

  // define __esModule on exports
  __webpack_require__.r = function(exports) {
    if(typeof Symbol !== 'undefined' && Symbol.toStringTag) {
      Object.defineProperty(exports, Symbol.toStringTag, { value: 'Module' });
    }
    Object.defineProperty(exports, '__esModule', { value: true });
  };

 	__webpack_require__.t = function(value, mode) {
    if(mode & 1) value = __webpack_require__(value);
    if(mode & 8) return value;
    if((mode & 4) && typeof value === 'object' && value && value.__esModule) return value;
    var ns = Object.create(null);
    __webpack_require__.r(ns);
    Object.defineProperty(ns, 'default', { enumerable: true, value: value });
    if(mode & 2 && typeof value != 'string') for(var key in value) __webpack_require__.d(ns, key, function(key) { return value[key]; }.bind(null, key));
    return ns;
  }

  // getDefaultExport function for compatibility with non-harmony modules
  __webpack_require__.n = function(module) {
    var getter = module && module.__esModule ?
    function getDefault() { return module['default']; } :
    function getModuleExports() { return module; };
    __webpack_require__.d(getter, 'a', getter);
  	return getter;
  };

  // Object.prototype.hasOwnProperty.call
  __webpack_require__.o = function(object, property) { return Object.prototype.hasOwnProperty.call(object, property); };

  // __webpack_public_path__
  __webpack_require__.p = "";
   
  // Load entry module and return exports
  return __webpack_require__(__webpack_require__.s = "./index.js");
})
({
  "./index.js": (function(module, exports, __webpack_modules__) {
    console.log('srart require')
    var lib = __webpack_require__(/*! ./lib.js */ "./lib.js")
    console.log('end require', lib)
    lib.additional = 'test'
  }),
  "./lib.js": (function(module, exports) {
    console.log('hello, geekbang')
    exports.hello = 'world'
    exports.add = function(a, b) {
      return a + b
    }
    exports.geekbang = { hello: 'world' }
      module.exports = function minus(a, b) {
    }
    
    setTimeout(() => {
      console.log(exports);
    }, 2000)
  })
})
```
1. index.js 和 lib.js 都被打包到函数里，key为文件名
  * CommonJS规范要求每个函数有独立的作用域，js创建作用域的方法就是函数（什么是作用域，参考浏览器原理）
2. __webpack_require__ 函数就是require的实现
3. webpack打包的源码解读
  * (funcntion(modules){})({}) webpack打包的js脚本的结构


