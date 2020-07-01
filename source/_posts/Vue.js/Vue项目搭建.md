---
title: Vue项目搭建
categories:
  - Vue.js
tags:
  - 前端框架
  - Vue.js
toc: true
date: 2018-12-16 19:39:42
---
## Vue项目实战
做一个简单的Vue前端项目,练习与巩固知识点,做一个总结

#### 前期准备
node环境
github+git项目管理
#### 项目创建
1. 在github上创建一个仓库,作为本次项目仓库
然后clone到本地

2. 命令行工具初始化一个Vue项目
前提:先安装好vue脚手架(全局), 可以使用vue命令
进入项目根目录,然后执行`vue init webpack`初始化一个vue

3. 尝试启动项目
运行`npm run dev`命令,访问成功,一个vue的demo创建完成

#### 项目结构
**src项目源代码结构**
业务开发根目录
![项目结构](项目结构.png)
+ main.js --- 项目入口文件
+ App.vue --- 项目原始根组件
+ router
  + index.js --- 项目路由
+ components --- 项目中的组件
+ assests --- 项目中的图片资源文件

**config配置文件夹**
  + index.js --- 基础配置文件
  + dev.env.js --- 开发环境
  + prod.env.js --- 上线环境

**build项目打包的webpack的配置内容**
一般也不需要修改

#### .vue文件结构
.vue是一种单文件组件
```html
<!-- Vue单组件文件 -->

<!-- 组件模板 -->
<template>
  <div id="app">
    <img src="./assets/logo.png">
    <router-view/>
  </div>
</template>

<!-- 组件逻辑 -->
<script>
export default {
  name: 'App'
}
</script>

<!-- 组件样式 -->
<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
```

#### 引入fastclick第三方模块
引入fastclick包解决有些浏览器延迟300ms响应click事件的问题
`npm install fastclick --save`

#### 引入Element模板库
`npm i element-ui -S` npm安装,这样可以和webpack更好的结合
然后在main.js中导入
```js
// 引入element
import ElementUI from 'element-ui'
import 'element-ui/lib/theme-chalk/index.css'
// 全局使用
Vue.use(ElementUI);
```