---
title: Electron+Vue入门
categories:
  - Vue.js
tags:
  - 前端框架
  - Electron
  - Vue.js
toc: true
date: 2020-02-07 16:24:19
---

## Electron+Vue入门

------------

### 了解Electron

Electron是由Github开发，**用HTML，CSS和JavaScript来构建跨平台桌面应用程序**的一个开源库。 Electron通过将Chromium和Node.js合并到同一个运行时环境中，并将其打包为Mac，Windows和Linux系统下的应用来实现这一目的。

Electron使用Web页面来作为桌面应用的GUI,所以可以把它看作成一个被 JavaScript 控制的，精简版的 Chromium 浏览器

### Electron+Vue结合使用

Vue的开发环境,node.js,npm就可以进行Electron开发

#### 使用Electron-vue框架

Electron-vue是基于electron和vue结合搭建的开发脚手架

```shell
vue init simulatedgreg/electron-vue my-project
```

![脚手架构建项目](/脚手架构建项目.png)

#### Electron主进程和渲染进程

Electron项目启动后,会先找到`package.json`中的main,找到主进程`main.js`,在主进程中运行的脚本通过创建Web页面来展示用户界面.Web页面运行在渲染进程. 一个Electron应用总是有且只有一个主进程.

主要模块:

![主要模块](/主要模块.png)
