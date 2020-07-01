---
title: Vue路由
categories:
  - Vue.js
tags:
  - 前端框架
  - Vue.js
toc: true
date: 2018-11-02 10:51:54
---
## Vue路由
Vue路由允许我们通过不同的URL访问不同的内容
以前都是在服务端对路由进行划分,不同路由处理不同的请求
现在前端的路由可以实现在不重新请求页面的情况下,改变URL展示不同的页面内容

利用Vue.js + vue.router可以实现单页应用
### 传统多页应用和单页应用对比
**多页应用**
+ **优点**: 首页响应快
+ **缺点**: 页面切换慢

**单页应用**
+ **优点**: js动态切换页面,无需再请求html文件
+ **缺点**: 首页加载慢

### Vue.router
`vue.router`是Vue官方的路由插件
路由引入的就是模板
#### `<rout-link>`
使用`<rout-link>`标签,引入定义好的路由
#### `<router-view>`
`<router-view>`区域用来显示路由匹配到的组件

### 路由的创建
定义路由
```js
const routes = [
  { path: '/foo', component: Foo },
  { path: '/bar', component: Bar }
]
```
创建路由实例
```js
const router = new VueRouter({
  el:"#app",
  routes:routes
})
```

