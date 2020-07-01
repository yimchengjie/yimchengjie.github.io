---
title: Vue概述
categories:
  - Vue.js
tags:
  - 前端框架
  - Vue.js
toc: true
date: 2018-09-13 10:37:09
---

## 概述

### 什么是 Vue.js

- Vue.js 是目前最火的前端框架,React 是最流行的一个前端框架(React 除了开发网站,还可以开发手机 APP,Vue 语法也可以用于手机开发,需要借助于 Weex)
- Vue.js 是前端的主流框架之一,和 Angular.js,React.js 一起,并成为前端三大主流框架
- Vue.js 是一套构建用户界面的框架,**只关注视图层**,它易于上手,还便于第三方库或既有项目整合.(Vue 有配套的第三方类库,可以整合起来做大型项目的开发)
- 前端的主要工作,负责 MVC 中的 V 层,主要工作就是制作前端页面效果.

### 为什么要学习前端框架

- 提高开发效率
  - 提高开发效率的发展历程:原生 JS->jQuery 类库->前端模板引擎->Angular.js/Vue.js(能够帮助我们减少不必要的 DOM 操作,提高渲染效率;双向数据绑带[通过框架提供的指令,前端程序员只需要关心业务逻辑,不必关心 DOM 是如何渲染了]);
  - 在 Vue.js 中,一个核心的概念,就是让用户不再操作 DOM 元素,解放了用户的双手,让程序员可以更多的关注业务逻辑

### 框架和库的区别

- 框架: 是一套完整的解决方案;对项目的侵入性较大,项目如果需要更换框架,则需要重新架构整个项目.
  - node 中的 express
- 库: 提供某一个小功能,对项目的侵入性小,如果某个库无法完成某些需求,可以很容易的切换到其他库实现需求.

### 后端(Node)中的 MVC 与前端中的 MVVM 之间的区别

- MVC 是后端的分层开发概念
- MVVM 是前端视图层的概念,主要关注与视图层分层,也就是说:MVVM 把前端的视图层,把每个页面分成了 M,V 和 VM 层,VM 是 MVVM 思想的核心,因为 VM 层是 M 与 V 之间的调度者. - M 保存的是每个页面中单独的数据 - V 就是每个页面的 HTML 结构 - VM 是一个调度者,分割了 V 和 M,每当 V 层需要获取或保存数据的时候,都需要 VM 做中间处理. - 前端页面使用 MVVM 的思想,主要是为了使开发更方便,VM 提供了数据的双向绑定
  ![mvc与mvvm](MVC与MVVM.png)

### Vue 中和 MVVM 之间的对应关系

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="ie=edge" />
    <title>Document</title>
    <!-- 1.导入Vue的包 -->
    <!-- 开发环境版本，包含了有帮助的命令行警告 -->
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
  </head>
  <body>
    <!-- MVVM中的V层 -->
    <!-- new的Vue实例,会控制这个元素中的内容 -->
    <div id="app">
      <p>{{msg}}</p>
    </div>
  </body>
  <script>
    // 2.创建一个Vue的实例
    // 当我们导入包之后,在浏览器的内存中,就多了一个Vue的构造函数
    // 这个vue对象就是MVVM中的调度者
    // 其中的date就是MVVM中的M 数据
    var vue = new Vue({
      el: "#app", //标书,当前我们new的这个vue实例,要控制页面上的哪个区域
      data: {
        //data属性中,存放的是el中要用到的数据
        msg: "Hello Vue.js" //通过Vue提供的指令,很方便的就能把数据渲染到页面上,程序员不需要操作DOM元素了
      }
    });
  </script>
</html>
```
