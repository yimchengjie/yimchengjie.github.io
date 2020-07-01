---
title: Vue实例详解
categories:
  - Vue.js
tags:
  - 前端框架
  - Vue.js
toc: true
date: 2018-10-20 10:20:51
---
## Vue属性
每一个Vue实例都拥有一些属性，比如之前的`el`、`data`、`methods`等
还有一些其他的属性
#### el属性
用于绑定id，指示Vue的作用范围
#### data属性
Vue的数据属性，用来定义Vue中的数据
#### methods
方法属性，一般的逻辑方法都写在这里
#### computed
计算属性，定义计算属性，也是函数方法，和methods类似，但是基于它的依赖缓存，可以说computed性能更好，但不需要缓存时，可以使用methods
#### watch
监听属性，可以通过watch来响应数据的变化，每当监听的变量改变，就调用方法函数
#### components
组件属性,用于注册组件
注意:在一个Vue实例内部注册的组件,只有这个Vue的作用范围内可以使用
#### directives
自定义指令
注意:在一个Vue实例内部注册的自定义指令,只有这个Vue的作用范围内可以使用

## Vue方法
Vue实例内部定义了8个方法,涵盖了一个Vue的生命周期
#### 初始化
+ beforeCreate()
  + Vue实例被创建后,但为空,内部数据和方法都还没有初始化
+ created()
  + 内部数据和方法初始化已经完成
+ beforeMount()
  + Vue模板已经编译好,但还没有挂载到页面
+ mounted()
  + 模板挂载完成

#### 更新
+ beforeUpdate()
  + 页面的数据还是旧的数据,没有同步为最新的data数据
+ updated()
  + 页面的数据已经更新到最新的data数据了

#### 销毁
+ beforeDestory()
  + Vue即将销毁,但当前的数据和方法都还是可用状态
+ destoryed()
  + Vue已经销毁,所有数据和方法都不可用了