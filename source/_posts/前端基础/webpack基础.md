---
title: webpack基础
categories:
  - 前端基础
tags:
  - webpack
  - 打包管理
toc: true
date: 2018-10-13 14:22:50
---

#### webpack

webpack 是一个现代的 JavaScript 应用程序的静态模块打包器,当 webpack 处理应用程序时,它会递归的构建一个依赖关系,然后将模块打包成一个或多个 bundle
webpack 有四个核心概念:entry,output,loader,plugins

- entry
  entry(入口)之时 webpack 应该使用哪个模块,来作为构建内部依赖的开始

  ```js
  //单个入口语法
  const config = {
    entry: "./src/main.js"
  };
  //对象语法
  const config = {
    app: "./src/main.js",
    vendors: "./src/vendors.js"
  };
  ```

- output
  output 属性会告诉 webpack 在哪里输出 bundle,以及如何命名,默认设置./list

  ```js
  const config={
    entry:"./src/main.js"
    output:{
      filename:"bundle.js",
      path:path.resolve(_dirname,'dist')
    }
  }
  ```

- loader
  loader 让 webpack 可以处理那些非 JavaScript 的文件,例如开发 ES6 时,通过 loader 将 ES6 语法转成 ES5

  ```js
  const config = {
    entry: "./src/main.js",
    output: {
      filename: "bundle.js",
      path: path.resolve(__dirname, "dist")
    },
    module: {
      rules: [
        {
          test: /\.js$/,
          exclude: /node_modules/,
          loader: "babel-loader",
          options: [(presets: ["env"])]
        }
      ]
    }
  };
  ```

- plugins
  loader 被用于转换某些类型的模块,而插件则可以做更多的事,包括打包优化,压缩,定义环境变量等等

  ```js
  // 通过 npm 安装
  const HtmlWebpackPlugin = require("html-webpack-plugin");
  // 用于访问内置插件
  const webpack = require("webpack");

  const config = {
    module: {
      rules: [
        {
          test: /\.js$/,
          exclude: /node_modules/,
          loader: "babel-loader"
        }
      ]
    },
    plugins: [new HtmlWebpackPlugin({ template: "./src/index.html" })]
  };
  ```

#### webpack 基本配置

```js
const path = require('path');

module.exports = {
  mode: "development", // "production" | "development"
  // 选择 development 为开发模式， production 为生产模式
  entry: "./src/main.js",
  output: {
    filename: "bundle.js",
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: "babel-loader",
        options: [
          presets: ["env"]
        ]
      }
    ]
  },
  plugins: [
    ...
  ]
}
```

#### webpack 新建项目

前提有 node.js 环境

1. 进入项目根目录执行 npm init -y
   创建默认 package.json 文件, -y 表示使用默认配置
2. npm install webpack webpack-cli --save-dev
   将 webpack 安装到本地项目,可以看到目录中会下载一些文件
3. 新建项目结构
   ![webpack项目结构](项目结构.png)
   app 存放原始项目,public 存放之后生成的浏览器可用的 js 代码,以及一个 index.html
4. 修改 index.html 文件

   ```html
   <!DOCTYPE html>
   <html>
     <head>
       <meta charset="utf-8" />
       <title></title>
     </head>
     <body>
       <div id="root"></div>
     </body>
     <!-- 之后webpack打包会生成的bundle.js -->
     <script src="bundle.js"></script>
   </html>
   ```

5. 编写 Geeter.js,改文件为项目文件

   ```js
   module.exports = function() {
     var greet = document.createElement("div");
     greet.textContent = "Hello Webpack";
     return greet;
     // 随便写一个
   };
   ```

6. 在 main.js 中引入

   ```js
   // 引入js
   const greeter = require("./Greeter");
   // 查询id为root的标签,在这个标签中拼接Greeter返回的内容
   document.querySelector("#root").appendChild(greeter());
   ```

7. 打包 webpack app/main.js -o public/bundle.js
   注意 webpack 要全局安装
   打包完,项目内就会出现 bundle.js
   在运行 index.html 就 ok 啦

### 总结

Webpack 是一款前端的模块化打包工具,就像 java 可以把项目打包为 jar,webpack 可以将模块化的前端打包成一个 js
webpack 需要 node.js 的环境,
