---
title: Webpack 应用和原理
description: 前端工程化构建和打包工具
published: true
date: 2021-01-14T14:05:57.042Z
tags: webpack
editor: markdown
dateCreated: 2021-01-11T15:39:33.917Z
---

# 引入

不同于以往对页面展示交互和数据表现的开发，越来越完善的概念和技术被注入到目前的大前端生态中。

## 前端工程化

当下前端领域，逐渐被工程化[^1]一词衡量项目的完整性，主要指下面几个方面的综合作用，可以构建出相对高效的多人协作流程。

### 模块化

不同功能的文件拆分，组织为模块，可以进一步划分为

- 前端代码逻辑的模块化
- 资源和样式文件的模块化

> The FEE can be divided into separate packages covering different portions of the project.

### 组件化

用设计模式的思路拆分页面为组件，每个组件可以针对不同场景进一步做多态和复用

### 规范化

项目规范化，可以进一步划分为：
  
- 项目结构的合理划分
- 代码编写层面的语法和缩进规范
- 组件、函数和工具库的统一文档输出
- 合理的代码源版本控制逻辑
- 组件的配色和交互视觉逻辑

### 测试

结合不同的测试场景和意图，可以进一步划分为：

- 与后端网络交互获取的 RESTful API 数据结构和完整性测试
- 界面组件表现的正确性测试
- 交互到完整业务流程的功能性测试
- 端到端的冒烟测试

### 自动化

自动化流程，可以进一步划分为：

- 自动化打包流程（本地或线上打包）
- 单次提交的自动化本地语法检查和测试用例覆盖检测
- 通过脚本、平台和容器化技术自动化部署到不同环境
- 以及不同环境的自动化流水线持续集成，测试通过率、失败错误信息报告

### 环境隔离

将开发、测试、灰度、预发布和生产环境做隔离（具体区分几个环境视业务场景而定），并使流水线能够在不同的环境中做集成测试。

## 什么是 Webpack

那么 Webpack 和上一节提及的工程化又有什么关系？

试想一下，我们编写的 JavaScript，可能在不同的模块化[^2]和语言规范（ECMAScript）[^3]支持下，与不同开发组织形式的 CSS（Less、Scss）相互配合，克服诸多浏览器平台和兼容性问题，才能顺利输出展示为完整的前端页面。

而 Webpack 作为项目模块的打包工具，依赖丰富的生态插件支持，可以配合脚本处理前端项目依赖、组件和规范化等诸多问题，我们可以以较小的代价完成上述流程的运作。

![webpack.png](/technology/web/webpack/webpack.png =95%x)

# 核心功能

既然 Webpack 可以解决前端工程化的部分问题，接下来我们便结合文档，一步步理解该工具的使用方式，然后结合使用场景思考其原理和实现方法。

## 从一个简单的用例开始

从一个简单的示例[^4]开始理解 Webpack 如何组织并编译 JavaScript 实现的模块，最终输出到 HTML 页面中。

首先创建一块新的目录区域并安装必要的 Webpack 依赖和命令行工具，我们后面所有的用例都会基于[该项目](https://github.com/L-Jovi/latte-web/tree/master/build/webpack)迭代，这里先从 [`getting-started`](https://github.com/L-Jovi/latte-web/tree/master/build/webpack/getting-started) 开始。

```bash
mkdir webpack-demo && cd webpack-demo
npm init -y
npm install webpack webpack-cli --save-dev
mkdir getting-started && cd getting-started
```

添加 HTML 模板和源码文件同步为下面的目录结构。

```bash
.
├── dist
│   └── index.html
├── src
│   └── index.js
└── webpack.config.js

```

添加一些简单的 JavaScript 内容到 `index.js` 中，同时为了模拟与 JQuery 相似的外部体验，这里引入了 Lodash 依赖作为工具库。

```bash
npm install --save lodash
```

```js
import _ from 'lodash'

function component() {
  var element = document.createElement('div')

  element.innerHTML = _.join(['Hello', 'Webpack'], ' ')

  return element
}

document.body.appendChild(component())
```

虽然 Webpack 4 开始就支持无配置执行，为便于后面的功能迭代，我们继续添加配置 `webpack.config.js` 的内容。

下面的配置内容会使 Webpack 从 `src/index.js` 开始作为项目入口开始寻找所有与此文件相关联的依赖，然后将其中的 ES6 规范逻辑转换为主流浏览器可以执行的 JavaScript 并输出到 `dist/bundle.js`。

```js
const path = require('path')

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  }
}
```

接着准备 HTML 模板，引入上述的 JavaScript 文件。

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>Getting Started</title>
  </head>
  <body>
    <script src="./bundle.js"></script>
  </body>
</html>
```

最后使用 npx 调用 Webpack 命令运行。

```bash
$ npx webpack
```

打开 `dist/index.html`，可以看到上述的 JavaScript 逻辑和外部依赖 Lodash 均被 Webpack 组织到 HTML 模板中输出，打印出 `Hello Webpack`。

## 资源管理

上一节我们实现的项目已经可以处理 ES6 规范的逻辑到浏览器可以执行的主流 JavaScript，并将依赖的外部模块 Lodash 一并打包到最终的 `bundle.js` 中，但是真实的项目环境，我们还需要处理样式表和图片等诸多类型的资源文件。

这一节我们针对该问题进行配置和处理，改动点可以参考 [`asset-management`](https://github.com/L-Jovi/latte-web/tree/master/build/webpack/asset-management)。

在 `src` 目录中分别创建一个图片文件 [`icon.jpg`](https://github.com/L-Jovi/latte-web/blob/master/build/webpack/asset-management/src/icon.jpg)，一个 XML 文件 [`data.xml`](https://github.com/L-Jovi/latte-web/blob/master/build/webpack/asset-management/src/data.xml) 和一个 CSS 文件 [`style.css`](https://github.com/L-Jovi/latte-web/blob/master/build/webpack/asset-management/src/style.css)，然后改动 `src/index.js` 的逻辑。

```js
import _ from 'lodash'
import './style.css'
import Icon from './icon.jpg'
import Data from './data.xml'

function component() {
  var element = document.createElement('div')

  element.innerHTML = _.join(['Hello', 'webpack'], ' ');
  element.classList.add('hello')

  var myIcon = new Image()
  myIcon.src = Icon

  element.appendChild(myIcon)

  console.log(Data)

  return element
}

document.body.appendChild(component())
```

注意我们的样式文件中对 `.hello` 类名引入了上述图片作为背景图。

```css
.hello {
  color: red;
  background: url('./icon.jpg');
}
```

你可能已经意识到了，我们一直使用 ES6 作为开发语言，而 Webpack 通过其模块[^5]功能可以对不同类型的文件和不同规范化的 JavaScript 语法做处理，即为 Webpack 视角的模块化支持，如果不依赖生态提供的模块能力，Webpack 原生就支持下面几种模块类型。

- ECMAScript 模块
- CommonJS 模块（Node.js 使用的模块系统）
- AMD 模块
- 资源文件
- WebAssembly 模块

本例中，还需要分别对上面添加的三种文件类型做 Webpack 配置处理，修改 `webpack.config.js` 通过 `module` 属性针对不同类型文件做正则表达式匹配并选择依赖对应的模块 loader 去处理。

这里以 CSS 处理为例，Webpack 匹配到后缀为 `.css` 的文件时，发现需要处理的 loader 是多个，便会以**相反**的顺序链式调用，首先通过 `css-loader` 对 `index.js` 中 `import './style.css'` 引入的 CSS 文件进行编译，然后将输出的结果传递给下一个模块 `style-loader`，添加样式到最终的 HTML 模板 `<head>` 标签中。

```js
const path = require('path')

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          'style-loader',
          'css-loader',
        ]
      },
      {
        test: /\.(png|svg|jpg|gif)$/,
        use: [
          'file-loader',
        ]
      },
      {
        test: /\.(woff|woff2|eot|ttf|otf)$/,
        use: [
          'file-loader',
        ]
      },
      {
        test: /\.(csv|tsv)$/,
        use: [
          'csv-loader'
        ]
      },
      {
        test: /\.xml$/,
        use: [
          'xml-loader'
        ]
      }
    ]
  }
}
```

最后在 `package.json` 同层级的目录中安装上述模块处理依赖，成功后再次执行 `npx webpack` 就可以看到 `dist` 中输出的结果了。

```bash
yarn add style-loader css-loader file-loader csv-loader xml-loader -D
```

## 输出管理

虽然到目前为止，我们已经可以处理项目中引入的不同类型文件和资源，但是随着项目体量的增长，不可能永远都是一个 `src/index.js` 经过打包后生成 `dist/bundle.js` 包揽一切，会有更多的模块和依赖被引入，这个时候，就需要将代码模块进行拆分。

我们将这种拆分行为称为输出管理，本节改动点可以参考 [`output-management`](https://github.com/L-Jovi/latte-web/tree/master/build/webpack/output-management)。

为了模拟增加的入口引用模块，我们修改 `src/index.js` 为：

```js
import _ from 'lodash'
import printMe from './print.js'

function component() {
  var element = document.createElement('div')
  var btn = document.createElement('button')

  element.innerHTML = _.join(['Hello', 'webpack'], ' ')

  btn.innerHTML = 'Click me and check the console!'
  btn.onclick = printMe

  element.appendChild(btn)

  return element
}

document.body.appendChild(component())
```

这样，不仅有从第三方依赖的 Lodash 模块，又多出来了我们自己实现的本地模块 `src/print.js`，这里我们简单提供一个打印功能即可。

```js
export default function printMe() {
  console.log('I get called from print.js!')
}
```

到上一节为止，我们都是在打包后的输出目录 `dist` 中放置一个手动维护的 `index.html` 模板，现在我们让这个流程也被 Webpack 管理起来，每次打包前先清理输出目录 `dist` 中所有现存的文件，然后通过插件生成 HTML 模板关联 Webpack 最终编译的所有输出结果。

```js
const path = require('path')
const { CleanWebpackPlugin } = require('clean-webpack-plugin')
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
  entry: {
    app: './src/index.js',
    print: './src/print.js',
  },

  output: {
    filename: '[name].bundle.js',
    path: path.resolve(__dirname, 'dist')
  },

  plugins: [
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      title: 'Output Management'
    })
  ]
}
```

需要注意，Webpack 通过 `plugins` 属性控制在编译的不同生命周期适时调用对应的插件，与模块不同，这里的调用是顺序进行，即为先依赖 `clean-webpack-plugin` 清理 `dist` 下的现存内容，然后调用 `html-webpack-plugin` 生成 HTML 模板关联项目依赖。

自行在外层目录安装本节新增的依赖到 `package.json`，然后运行 `npx webpack`，观察 `dist` 中的 HTML 文件，至此输出目录中的所有文件都已经被 Webpack 打包流程所管理。

## 开发

## 代码切分

## 缓存

## 热模块替换（HMR）

## 代码裁剪

## 填充

## 插件

## DLL

# Webpack 原理

## AST

## Hooks

[^1]: [Front-end engineering - Wikipedia](https://en.wikipedia.org/wiki/Front-end_engineering)
[^2]: [Understanding (all) JavaScript module formats and tools](https://weblogs.asp.net/dixin/understanding-all-javascript-module-formats-and-tools)
[^3]: [ECMAScript - Wikipedia](https://en.wikipedia.org/wiki/ECMAScript)
[^4]: [Getting Started | webpack](https://webpack.js.org/guides/getting-started/)
[^5]: [Modules | webpack](https://webpack.js.org/concepts/modules/#what-is-a-webpack-module)