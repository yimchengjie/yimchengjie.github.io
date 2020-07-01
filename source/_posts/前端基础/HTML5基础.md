---
title: HTML5基础
categories:
  - HTML
tags:
  - HTML
toc: true
date: 2018-08-13 09:15:32
---

### HTML5 基础

HTML5 是最新的 HTML 标准

#### HTML5 新特性

1. 在 HTML5 中,添加了新的 HTML 标签,比如`<article>`、`<footer>`、`<header>`、`<nav>`、`<section>`等
   - **section 标签:**
     `<section>`表示文档中的一个区域.比如章节、页眉、页脚或文档中的其他部分,一般来说会包含一个标题
     一般来说,一个`<section>`标签应该出现在文档大纲中.
   - **article 标签:**
     `<article>`标签定义独立的内容,常用在论坛帖子,报纸文章,博客条目,用户评论等独立内容项目中.
   - **nav 标签:**
     `<nav>`标签定义导航栏链接部分,描述一个包含多个链接的区域.
   - **header 标签:**
     `<header>`标签定义文档页眉,通常是一些引导和导航信息
   - **footer 标签:**
     `<footer>`标签定义 section 或 document 的页脚
   - **aside 标签:**
     `<aside>`标签标示一个页与其余页面几乎无关的内容,表现为侧边栏或者嵌入内容
   - **figure 标签:**
     `<figure>`标签规定独立的流(图片,图像,代码等)

#### HTML 视频音频

在以前,想要在网页上展示视频音频通常需要使用 flash 这样的插件,很麻烦,而 HTML5 中提供了音频视频的标准接口

##### 视频标签`<video>`

video 元素允许多个 source 子元素,可以链接不同的视频文件,浏览器自动使用第一个可使用的视频(用来引入多种格式以支持不同浏览器访问)

```html
<video>
  <source src="" type="video/mp4" />
  <source src="" type="video/webm" />
</video>
```

##### 视频字幕的使用

在`<video>`标签中使用`<track>`元素引入字幕`.vtt`格式的字幕文件

```html
<video>
  <source src="" type="video/mp4" />
  <track
    src="video_ch.vtt"
    srclang="zh"
    kind="subtitles"
    label="中文"
    default
  />
  <video></video>
</video>
```

`.vtt`内容格式

```text
WEBVTT

00:00:01.000 --> 00:00:04.000
Don't play games in class！
00:00:05.000 --> 00:00:09.000
Study hard and make progress every day！
```

##### 音频标签`<audio>`

audio 标签也支持多种 source 子标签

```html
<audio>
  <source src="" type="audio/mpeg" />
  <source src="" type="audio/wav" />
</audio>
```

#### HTML5 拖放

拖放是 HTML 标准的组成部分,任何元素都能进行拖放

1. 首先要定义元素的 draggable 属性为 true,再加上全局处理事件函数 ondragstart

   ```html
   <img draggable="true" ondragstart="drag(event)" />
   ```

2. 定义拖放数据
   每个 drag event 都有一个 dataTransfer 属性保存事件数据,setData()方法添加一个项目的拖拽数据
   `html function drag(ev){ ev.dataTransfer.setData("Text",ev.target.id); }`
3. 定义一个放置区
   ondragover 事件规定了在哪里放置拖动的数据,默认阻止,我们要开启
   `html function allowDrop(ev) { ev.preventDefault(); }`
4. 进行放置
   放置被拖动数据时,会发生 drop 事件
   `html function drop(ev) { //调用 preventDefault() 来避免浏览器对数据的默认处理 ev.preventDefault(); //通过 dataTransfer.getData("Text") 方法获得被拖的数据。该方法将返回在 setData() 方法中设置为相同类型的任何数据。 var data = ev.dataTransfer.getData("Text"); //被拖数据是被拖元素的 id ("drag1"),把被拖元素追加到放置元素（目标元素）中 ev.target.appendChild(document.getElementById(data)); }`

#### Canvas 画布

Canvas 可以用于图形表示,图像绘制,游戏制作,,需要通过 js 控制来绘制

使用来操作 HTML 图形图标

#### HTML5 表单

- datalist 元素,使用 `<datalist>` 元素来为表单小部件提供建议的、自动完成的值，并使用一些 `<option>` 子元素来指定要显示的值。然后使用 list 属性将数据列表绑定到一个文本域(通常是一个 `<input>`元素)。

  ```html
  <input type="text" name="myColor" id="myColor" list="mySuggestion" />
  <datalist id="mySuggestion">
    <option value="black"> </option>
    <option value="blue"> </option>
    <option value="green"> </option>
    <option value="red"> </option>
    <option value="white"> </option>
    <option value="yellow"> </option>
  </datalist>
  ```

- autocomplete 属性规定表单是否应该启用自动完成功能,当用户在字段开始键入时，浏览器基于之前键入过的值，应该显示出在字段中填写的选项。
- form 属性,适用所有 input 标签,让其可以在 form 域之外,但仍属于 form 的一部分.
- multiple 属性规定输入域可以选择多个值,适用于以下 input:email 和 file
- novalidate 属性,规定在表单提交时不应该验证 form 域
- pattern 属性,指定正则表达式,用于验证
- required 属性规定提交时不能为空

#### HTML5 输入类型

- input-email 类型,提交时自动验证
- input-url 类型,自动验证 url 域的值
- input-number,允许设置最大最小值和数字间隔
- input-range 类型,显示为滑动条,也有最大最小值和数字间隔
- input-Date Pickers 时间选择器
- input-search 搜索域
- input-color 颜色选择器

#### Web Storage 本地存储

由于 Cookie 的限制,HTML5 支持了两种 Web Storage:永久性的本地存储（localStorage）和会话级别的本地存储（sessionStorage）

#### HTML5 文件上传

在 HTML4 标准中文件上传控件只接受一个文件，而在新标准中，只需要设置 multiple，就支持多文件上传。按住 Ctrl 或者 Shift 即可选择多个文件。
**可以限制文件的上传类型**
使用 accept 属性,使用 accept 接受一个逗号分隔的 MIME 类型字符串。
