---
title: ES6基础
categories:
  - 前端基础
tags:
  - JavaScript
  - ES6
toc: true
date: 2018-09-25 14:06:54
---

## ES6 基础

ES6 全称 ECMAScript6.0 是 JavaScript 的下一个版本标准,但现在已经有很多浏览器支持了 ES6

JavaScript 只是 Oracle 公司注册的一个商标,它的正式名称就是 ECMAScript

### 关键字let和const

ES6 新增了两个重要的关键字:`let`和`const`
`let`声明的变量只在 let 所在代码块有效
`const`声明一个只读常量,不可修改

#### 变量的结构赋值

```js
//es5中对变量的赋值
var data = { userName: "aaaa", password: 123456 };
var userName = data.userName;
var password = data.password;
console.log(userName);
console.log(password);
var data1 = ["aaaa", 123456];
var userName1 = data1[0];
var password1 = data1[1];
console.log(userName1);
console.log(password1);

//es6中的解构赋值
const { userName, password } = { userName: "aaaa", password: 123456 };
console.log(userName);
console.log(password);
const [userName1, password1] = ["aaaa", 123456];
console.log(userName1);
console.log(password1);
```

#### ES6 模块化

ES6 引入了模块化，其设计思想是在编译时就能确定模块的依赖关系，以及输入和输出的变量。
ES6 的模块化分为导出（export） 与导入（import）两个模块。

- import 命令特性
  - **只读**:可以改写 import 变量类型为对象的属性值，不能改写 import 变量类型为基本类型的值。
  - **单例模式**
  - **静态执行模式**:不支持 import 的值为变量或者表达式
