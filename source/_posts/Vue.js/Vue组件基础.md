---
title: Vue组件基础
categories:
  - Vue.js
tags:
  - 前端框架
  - Vue.js
toc: true
date: 2018-09-30 15:33:56
---

### 组件基础

#### 1. 什么是组件

组件是可重复使用的 Vue 实例,开发中可以把经常重复的功能封装为组件.
组件可以将整个页面尽显模块化分割.
组件分为全局组件和局部组件.

#### 2. 基本实例

全局组件的创建方式

1. 使用 Vue.extend 来创建全局的 Vue 组件

   ```html
   <body>
     <div id="app">
       // 使用组件,直接把组件的名字以HTML标签形式引入 //注意:
       HTML大小写忽略,所以驼峰命名失效,要采用 xxx-xxx来代替xxxXxx
       <my-con1></my-con1>
     </div>
   </body>
   <script>
     // 注册组件Vue.component('组件名称',组件对象)
     Vue.component(
       "myCon1",
       Vue.extend({
         // 使用Vue.extend创建全局组件
         // 通过template属性指定组件要展示的HTML结构
         template: "<h3>这是使用Vue.extend创建的组件</h3>"
       })
     );
     // 创建Vue实例
     var vm = new Vue({});
   </script>
   ```

2. ```js
   // 定义一个名为 button-counter 的新组件
   //使用Vue.component函数创建组件,该函数有两个参数, 第一个是组件的名称,第二个是以对象的形式,描述一个组件
   //因为组件是可复用的Vue实例,所以与new Vue接收相同的选项
   Vue.component("button-counter", {
     data: function() {
       return {
         count: 0
       };
     },
     template:
       '<button v-on:click="count++">You clicked me {{ count }} times.</button>'
   });
   ```

3. ```html
   <body>
     <div id="app">
       <my-con3></my-con3>
     </div>

     <template id="tmp1">
       <div>
         <h1>
           在Vue实例外部定义组件,使用template标签,这种方式有代码提示,好用,推荐
         </h1>
       </div>
     </template>
   </body>
   <script>
     // 注册组件Vue.component('组件名称',组件对象)
     Vue.component("myCon3", {
       // 使用Vue.extend创建全局组件
       // 通过template属性指定组件要展示的HTML结构
       template: "#tmp1"
     });
     // 创建Vue实例
     var vm = new Vue({});
   </script>
   ```

局部组件,在 Vue 实例内部的 components 方法内创建,只有这个 Vue 实例可以使用

```html
<body>
  <div id="app">
    <my-con></my-con>
  </div>

  <template id="tmp1">
    <div>
      <h1>
        在Vue实例外部定义组件,使用template标签,这种方式有代码提示,好用,推荐
      </h1>
    </div>
  </template>
</body>
<script>
  var vm = new Vue({
    components: {
      myCon: {
        template: "#tmp1"
      }
    }
  });
</script>
```

**注意:** 无论使用什么方式, 模板最外层只能用一个根标签包含

#### 3. 组件复用

组件的优点在于重复利用
注意:复用组件内的 data 必须是一个函数,如果是一个对象,组件之间会相互影响(多个组件会共用一个对象),使用函-数会分别管理数据
vue 提供 component 标签来有用组件

```html
<template id="tmp1">
  <div>
    <h1>在Vue实例外部定义组件,使用template标签,这种方式有代码提示,好用,推荐</h1>
  </div>
</template>

<div id="app">
  <component :is="tmp1"></component>
</div>
```

#### 4. 组件之间的通信

组件也需要数据的通信,而 vue 中子组件默认无法访问父组件的数据

##### 父子组件之 props

props 是一个单向的数据流,只允许父组件向子组件传值,可以是数值、字符、布尔值、数值、对象。

```html
<body>
  <div id="app">
    <com1 v-bind:parentmsg="msg"></com1>
  </div>

  <template id="tmp1">
    <div>
      <h1>{{ parentmsg }}</h1>
    </div>
  </template>
</body>
<script>
  var vm = new Vue({
    el: "#app",
    data: {
      msg: "hello"
    },
    components: {
      com1: {
        template: "#tmp1",
        // 定义一个参数接收
        props: ["parentmsg"]
      }
    }
  });
</script>
```
