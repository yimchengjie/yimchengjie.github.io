---
title: Vue基本指令
categories:
  - Vue.js
tags:
  - 前端框架
  - Vue.js
toc: true
date: 2018-09-15 19:40:33
---

### Vue 基本指令

#### 1:v-cloak

能够解决插值表达式闪烁问题

#### 2:v-text

和`{{text}}`功能一样,但是没有闪烁问题
`{{text}}`可以在插值前后有内容,v-text 会覆盖元素中原本的内容

#### 3:v-html

将数据已 HTML 格式输出

#### 4:v-bind

v-bind: 是 Vue 中提供的用于绑定属性的指令
v-bind 可以缩写为`:`
用法:

- v-bind:title(需要绑定的属性)
- 可简写为:title(需要绑定的属性)
- 绑定元素设置 class 样式
  1. 用数组`:class="['class1','class2']"`
  2. 三元表达式`:class="['class1','class2',flag?'class3':'']"`
  3. 数组中嵌套对象`:class="['class1','class2',{'class3':boolean}]"`
  4. 直接使用对象`:class="{class1:true,class2:true,class3:false}"`(可以在 data 中定义`classobj:{class1:true,class2:true,class3:false}`然后在属性中直接饮用`:class="classobj"`)

#### 5:v-on

v-on:是 Vue 中提供的用于绑定事件的指令
v-on 可以缩写为`@`
用法:

- v-on:click="show"
  需要在 Vue 对象中定义 show 方法

  ```js
  method:{
      show:function(){
          alert("hello");
      }
  }
  ```

- 时间修饰符
  1. .stop:阻止冒泡;
  2. .prevent:阻止默认事件的发生;
  3. .capture:捕获冒泡,有该修饰符的 dom 元素会先执行，如果有多个，从外到内依次执行，然后再按自然顺序执行触发的事件。
  4. .self:将事件绑定到自身,只有自身能被触发,通常用于避免冒泡事件的影响;
  5. .once:设置事件只能触发一次,比如按钮的点击;
  6. .passive:用于对 DOM 的默认事件进行性能优化,比如超出最大范围的滚动条滚动;
  7. .native:把 vue 组件转化成一个普通的 HTML 标签,对普通的 HTML 标签是没有任何作用的;

#### 6:v-modle

v-modle:是 Vue 中用于数据双向绑定的指令
**注意:** **_只能运用在表单元素中_**

v-bind 只能实现数据单向绑定,从 M=>V

#### 7:v-for

v-for 是循环迭代指令

- 用法:
  1. 迭代数组
  
      ```html
      <ul>
        <li v-for="(item i) in list">
          索引:{{i}}----姓名:{{item.name}}----年龄:{{item.age}}
        </li>
      </ul>
      ```

  2. 迭代对象中的属性
  
      ```html
      <!-- 循环遍历对象的属性 -->
      <div v-for="(val,key,i) in list">
        {{val}}---{{key}}---{{i}}
      </div>
      ```
  
  3. 迭代数字

      ```html
      <p v-for="i in 10">{{i}}</p>
      ```

- v-for 中 key 的使用注意(key 相当于主键,是唯一的)
  1. v-for 循环时,key 属性只能用 number 或者 string
  2. key 在使用的时候必须使用 v-bind 绑定来指定 key 的值

#### 8:v-if

v-if 是条件渲染指令,v-if 有更高的切换渲染开销.如果在运行时条件不太可能改变，则使用 v-if 较好.

- 用法

  ```html
  <div id="app-3">
    <p v-if="seen">现在你看到我了</p>
  </div>
  ```

#### 9:v-show

v-show 有更高的初始渲染开销

- 用法

  ```html
  <div v-show="ifShow">show</div>
  ```
