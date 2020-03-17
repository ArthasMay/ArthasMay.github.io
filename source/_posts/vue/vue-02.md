---
title: Vue01 | Vue细节实现：封装高可用的Axios模块
p: vue/vue-02
date: 2019-11-18 14:03:05
tags:
- Vue
- Axios
categories: Vue
---

### 为什么要对Axios二次封装？
Axios其实是对 Node的Http以及对XMLHttpRequest的封装，使用了Es6中的Promise的异步调用的方式，其API对开发者来说其实已经是很友善了。而在项目中对Axios的二次封装主要是为了桥接项目的业务，因为Axios本身只是抽象的调用层，项目中的一些具体业务并未处理，所以需要根据后端的的架构（前后端鉴权方式，业务Code码层面的设计）、API设计风格（最好是面向规范的Restful设计）以及面对复杂的异步调用（由于没有BFF的数据裁剪，一般公司前后端分离之后复杂的串行和并行是经常要面对的）等问题。

#### Vue项目的网络层设计

1. 网络请求代码的聚合和复用，支持模块化的扩展（类似于vuex的模块化的横向拓展）
2. 对用户状态方式的处理（常见的两种后端鉴权方式：（1）cookie & session （2）JWT）
3. 对服务端设计的业务code码的集中式处理
4. 面对标准RESTFUL的api的请求方式设计
5. 自动重试机制分层设计 & 复杂异步的调用的api设计（支持async/await函数）
6. 如何设计生产环境和开发环境等设置
7. 底层信息的Cache(localStorage & sessionStorage)

``` 
// 整个网络层涉及的文件层级
api
└── login               // 对应的业务模块
    └── index.js
└── dashboard
    └── index.js
├── base.js             // 基础的设置
└── index.js            // 导出的出口文件
utils
└── network             // 对axios的封装
    ├── fetch.js
    └── request.js
├── .env.production     // Vue项目生产和测试环境的配置文件
├── .env.development
├── .env.test 
└── .vue.config.js      // Vue 配置文件
```

#### fetch.js的设计

* JWT / Session & Cookie的鉴权方式
* 处理status & code的异常提示处理
* 处理断网的全局UI组件处理（是从BOM中获取网络状态）

``` js
import axios from 'axios'
import router from "@/router/router.js";
import store from "@/store/store.js";
// TODO: 为什么在js文件内需要单独引用message，但是在Vue文件内可以使用this.$message这种内部函数调用，研究下怎么实现一个可以全局使用的Vue组件（即通过Vue.use()注册） 
import { Message } from 'element-ui'

// Warning: 默认打开的是JWT的鉴权方式，若要修改为Session & Cookie的方式来保持会话状态，请将 tokenStyle 修改为： var tokenStyle = 'Session_Cookie' 
var tokenStyle = 'JWT' 

/*
 * @Description: 提示函数
 */
const tip = msg => {
  Message.error(msg)
}

/*
 * @Description: 跳转登录页
 * 携带当前页面路由，为了登录完成后返回当前页面
 */
const toLogin = () => {
  router.replace({
    path: '/login',
    query: {
      redirect: router.currentRoute.fullPath
    }
  })
}

/*
 * @Description: 处理常见的几种http协议规范中status的错误
 */
const statusErrorHandle = (status, other) => {
  // 状态码判断
  switch(status) {
    case 401: // 401: 未登录状态，跳转登录页
      toLogin()
      break
    case 403: // 403: token过期，清除token并跳转登录页
      tip('登录过期，请重新登录')
      // 后面会重新对持久层封装， 在后面层重构这部分代码
      localStorage.removeItem("token");
      store.commit("loginSuccess", null);
      setTimeout(() => {
        toLogin()
      }, 1000)
      break
    case 404: // 404: 请求不存在
      tip('请求的资源不存在')
      break
    default:
      console.log(other)
  }
}

/*
 * @Description: 业务层Code的错误处理，这边服务端的code设计错误
 */
const codeErrorHandler = (errRes) => {
  let { code, msg } = errRes
  switch (code) {
    case 1403:
      tip(msg);
      localStorage.removeItem("token");
      store.commit("loginSuccess", null);
      setTimeout(() => {
        toLogin();
      }, 1000);
      break;
    default:
      console.log("Unkown code type 😭😭😭");
      tip(msg);
  }
}

var instance = axios.create({
  timeout: 30000
})


/*
 * @Description: 请求拦截器
 * 每次请求前，如果存在token则在请求中携带token
 */
instance.interceptors.request.use(
  config => {
    if (tokenStyle === 'JWT') {
      const token = store.state.token;
      token && (config.headers.Authorization = token);
    }
    return config
  }
)

/*
 * @Description: 响应拦截器
 */
instance.interceptors.response.use(
  // 请求成功
  res => {
    if (res.status === 200) {
      if (res.data.code === 200) {
        return Promise.resolve(res.data);
      } else {
        codeErrorHandler(res.data);
        return Promise.reject(res.data);
      }
    } else {
      return Promise.reject(res.data);
    }
  },
  // 请求失败
  error => {
    const { response } = error;
    if (response) {
      // 请求已经发出，但是不在2xx的范围
      statusErrorHandle(response.status, response.message);
      return Promise.reject(response);
    } else {
      // 处理断网的情况
      // eg:请求超时或断网时，更新state的network状态
      // network状态在app.vue中控制着一个全局的断网提示组件的显示隐藏
      // 关于断网组件中的刷新重新获取数据，会在断网组件中说明
      if (!window.navigator.onLine) {
        store.commit('changeNetwork', false);
      } else {
        return Promise.reject(error);
      }
    }
  }
);

export default instance;
```
 
 ##### JWT和Session_Cookie的理解和使用
 
 
