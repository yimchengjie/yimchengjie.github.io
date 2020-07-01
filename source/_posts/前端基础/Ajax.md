---
title: Ajax
date: 2018-03-05 16:25
categories:
  - 前端基础
tags:
  - Ajax
  - 异步请求
toc: true
---

## AJAX

### 1.Ajax 的作用

获取服务器的数据

### 2.Ajax 的效果

在不刷新整个页面的情况下,通过一个 url 地址获取服务器的数据,然后进行页面的局部刷新. 异布加载,

### 3.小结

Ajax 的全称: Asychronous JavaScript And XML,就是使用 js 代码获取服务器数据.

### 4.Ajax 的使用

Ajax 简单的来说,就是一个异布的 JavaScript 请求,用来获取后台服务端的数据,而并不是整个界面进行跳转.

在元素 js 中来实现 AJax 主要的类就是 XMLHttpRequest,它的使用一般有四个步骤;

1. 创建 XMLHttpRequest 对象
2. 准备发送网络请求
3. 开始发送网络请求
4. 指定回调函数

**注意:**

1. 需要注意兼容处理.低版本浏览器不支持 XMLHttpRequest 对象,需要创建 ActiveXObject 对象;
2. 指定请求方式,请求地址以及指定是否异步刷新
3. 执行发送,**POST 请求方式**时,数据不写在地址中,放在请求数据体中.需要发送给服务器,同时设置请求头
4. 异步的原理是通过请求浏览器进行网络数据的请求

### 5. 数据格式

#### 1. Xml 数据格式

Xml 数据格式是将数据以标签的方式进行组装,必须以`<?xml version="1.0" encoding="utf-8" ?>`开头,标签必须成对出现,xml 是一个通用标准,任何人都知道如何解析它;
**缺点:**体积太大,传播慢,元数据太多,解析不方便,目前使用很少

#### 2. JSON 数据格式

Json 数据格式类似于 js 中的对象方式,通过 key-value 的形式组装,是一个通用的标准,任何人都知道如何解析它;
**优点:**体积小,传输快,解析方便

#### 3. 解析 Xml 数据格式

获取 Xml 对象,在通过 getElementsBtTagName 获取标签内元素

```js
var result = xhr.responseXML;
var books = result
  .getElementsByTagName("booklist")[0]
  .getElementsByTagName("book");
var newHtml = document.getElementById("bookContariner").innerHtml;
for (var i = 0; i < books.length; i++) {
  var itemBook = books[i];
  var name = itemBook.getElementsByTagName("name")[0].textContent;
  var author = itemBook.getElementsByTagName("author")[0].textContent;
  var desc = itemBook.getElementsByTagName("desc")[0].textContent;
  var tempHtml =
    "<tr><td>" + name + "</td><td>" + author + "</td><td>" + desc + "</td><td>";
  newHtml += tempHtml;
}
document.getElementById("bookContariner").innerHtml = newHtml;
```

#### 4. 解析 Json 数据格式

获取 Json 对象,再通过对象直接获取对象的属性

```js
var result = xhr.responseTest;
//responseTest获取的是字符串,要转换成JSON对象
result = JSON.parse(result);
var newHtml = document.getElementById("bookContariner").innerHtml;
for (var i = 0; i < result.length; i++) {
var item = result[i];
var name = item.name;
var author = item.author;
var desc = item.desc;
var tempHtml ="<tr><td>" + name + "</td><td>" + author + "</td><td>" + desc + "</td><td>";
newHtml += tempHtml;
}
document.getElementById("bookContariner").innerHtml = newHtml;
```

### 6.封装 Ajax

```js
function myAjax(type, url, params, callback, async) {
  var xhr = null;
  if (window.XMLHttpRequest) {
    xhr = new XMLHttpRequest();
  } else {
    xhr = new ActiveXObject("Microsoft.XMLHTTP");
  }
  if (type == "get") {
    if (params && params != "") {
      url += "?" + params;
    }
  }
  xhr.open(type, url, async);
  if (type == "get") {
    xhr.send(null);
  } else if (type == "post") {
    xhr.setRequestHeader("Contend-Type", "application/x-www-from-urlencoded");
    xhr.send(params);
  }
  if (async) {
    xhr.onreadystatechange = function() {
      if (xhr.readyState == 4) {
        if (xhr.readyState == 200) {
          var result = null;
          if (dataType == "json") {
            result = xhr.responseText;
            result = JSON.parse(result);
          } else if (dataType == "xml") {
            result = xhr.responseXML;
          } else {
            result = xhr.responseText;
          }
          if (callback) {
            callback(result);
          }
        }
      }
    };
  } else {
    if (xhr.readyState == 4) {
      if (xhr.readyState == 200) {
        var result = null;
        if (dataType == "json") {
          result = xhr.responseText;
          result = JSON.parse(result);
        } else if (dataType == "xml") {
          result = xhr.responseXML;
        } else {
          result = xhr.responseText;
        }
        if (callback) {
          callback(result);
        }
      }
    }
  }
}
```

### 7 用 jQuery 实现 Ajax

jQuery 对 Ajex 操作进行了封装,在 jQuery 中最底层的方法是`$.ajax()`,第二层是`$.load()`、`$.get()` 、`$.post()` 第三层是`$.getJSON()`、`$.getScript()`

- `$.ajax()`用法: + **type**：指定数据提交的方式 + **url**：提交数据的路径 + **cache**:是否存在缓存 + **data**：向后台发送的数据 + **dataType**：服务器端返回的数据类型，比如：xml，text，json, html，script + **success**：响应成功后执行的函数 + **error**：响应失败后执行的函数

```javascript
var subData={
    name:'张三'
} //提交的数据
$.ajax({
    url : "IndexController/getIndexImages.html",
    type : "POST",
    async : true,//表示进行异步获取
    data:subData,//提交的数据
    dataType : 'json',
    contentType : 'application/x-www-form-urlencoded;charset=UTF-8', //contentType很重要
    success : function(result) {
        var obj = $.parseJSON(result);
        //在这里对返回的数据进行处理
    }
    error:function(){
       //请求失败执行这个
    }
});
```
