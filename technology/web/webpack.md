---
title: Webpack in Practice and Principle
description: A build and bundling tool for front-end engineering
published: true
date: 2026-09-30T13:35:19.000Z
tags: webpack
editor: markdown
dateCreated: 2026-09-30T13:35:19.000Z
---

**English** · [中文](/zh/technology/web/webpack.md)

# Introduction

Unlike the development of the past, which was about page display, interaction and data presentation, more and more mature concepts and technologies are being poured into today's big front-end ecosystem.

## Front-end engineering

In the front-end field these days, how complete a project is has gradually come to be measured by the word engineering[^1]. It mainly means the combined effect of the following aspects, which together can build a fairly efficient workflow for many people working together.

### Modularization

Files with different functions are split up and organized as modules, which can be further divided into

- modularization of front-end code logic
- modularization of assets and style files

> The FEE can be divided into separate packages covering different portions of the project.

### Componentization

Split pages into components with design-pattern thinking; each component can be further made polymorphic and reused for different scenarios

### Standardization

Project standardization can be further divided into:
  
- A sensible division of the project structure
- Syntax and indentation standards at the code-writing level
- Unified documentation output for components, functions and utility libraries
- Sensible version control logic for the source code
- Color schemes and visual interaction logic for components

### Testing

Depending on the testing scenario and intent, it can be further divided into:

- Tests of the data structure and completeness of the RESTful API data fetched from the backend over the network
- Tests that UI components render correctly
- Functional tests from interactions through to complete business flows
- End-to-end smoke tests

### Automation

The automated workflow can be further divided into:

- Automated bundling (bundling locally or online)
- Automated local syntax checks and test-case coverage checks for each commit
- Automated deployment to different environments through scripts, platforms and containerization
- And continuous integration through automated pipelines in different environments, with reports on test pass rates and failures

### Environment isolation

Isolate the development, testing, canary, pre-release and production environments (exactly how many environments depends on the business), and let the pipeline run integration tests in the different environments.

## What is Webpack

So what does Webpack have to do with the engineering mentioned in the last section?

Think about it: the JavaScript we write, supported by different module systems[^2] and language specifications (ECMAScript)[^3], has to work together with CSS organized in different ways (Less, Scss) and get past lots of browser platform and compatibility problems before it can finally be output and shown as a complete front-end page.

As a bundler for project modules, backed by a rich ecosystem of plugins, Webpack can work with scripts to handle many problems such as front-end project dependencies, components and standardization, so we can get the workflow above running at a fairly small cost.

<img src="/technology/web/webpack/webpack.png" alt="webpack.png" width="95%">

# Core features

Since Webpack can solve part of the problems of front-end engineering, next we'll go through the docs and understand how to use the tool step by step, and then think about its principles and implementation in light of the use cases.

## Starting with a simple example

We start from a simple example[^4] to understand how Webpack organizes and compiles modules written in JavaScript, and finally outputs them into an HTML page.

First create a new directory and install the necessary Webpack dependencies and command-line tool. All our later examples iterate on [this project](https://github.com/L-Jovi/latte-web/tree/master/build/webpack), starting here with [`getting-started`](https://github.com/L-Jovi/latte-web/tree/master/build/webpack/getting-started).

```bash
mkdir webpack-demo && cd webpack-demo
npm init -y
npm install webpack webpack-cli --save-dev
mkdir getting-started && cd getting-started
```

Add an HTML template and source files so the directory matches the structure below.

```bash
.
├── dist
│   └── index.html
├── src
│   └── index.js
└── webpack.config.js

```

Add some simple JavaScript to `index.js`, and to simulate an outside experience similar to JQuery, bring in Lodash as a utility library dependency.

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

Although Webpack has supported running without configuration since Webpack 4, to make the later iterations easier we go on to add a `webpack.config.js` configuration.

The configuration below makes Webpack start from `src/index.js` as the project entry and look for every dependency related to this file, then convert the ES6 logic in it into JavaScript that mainstream browsers can run, and output it to `dist/bundle.js`.

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

Next, prepare the HTML template and bring in the JavaScript file above.

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

Finally, run the Webpack command with npx.

```bash
$ npx webpack
```

Open `dist/index.html`, and you'll see that the JavaScript logic above and the external dependency Lodash have both been organized by Webpack into the HTML template output, printing `Hello Webpack`.

## Asset management

The project from the last section can already turn ES6 logic into mainstream JavaScript that browsers can run, and bundles the external dependency Lodash into the final `bundle.js` as well, but in a real project we also need to handle stylesheets, images and many other kinds of asset files.

In this section we configure things to handle that; for the changes, see [`asset-management`](https://github.com/L-Jovi/latte-web/tree/master/build/webpack/asset-management).

In the `src` directory, create an image file [`icon.jpg`](https://github.com/L-Jovi/latte-web/blob/master/build/webpack/asset-management/src/icon.jpg), an XML file [`data.xml`](https://github.com/L-Jovi/latte-web/blob/master/build/webpack/asset-management/src/data.xml) and a CSS file [`style.css`](https://github.com/L-Jovi/latte-web/blob/master/build/webpack/asset-management/src/style.css), then change the logic of `src/index.js`.

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

Note that our stylesheet gives the `.hello` class the image above as its background.

```css
.hello {
  color: red;
  background: url('./icon.jpg');
}
```

You may have noticed that we've been writing ES6 all along, and Webpack, through its modules[^5] feature, can handle different types of files and JavaScript syntax following different standards; this is modularization support from Webpack's point of view. Without relying on the module abilities the ecosystem provides, Webpack natively supports these module types.

- ECMAScript modules
- CommonJS modules (the module system Node.js uses)
- AMD modules
- Assets
- WebAssembly modules

In this example, each of the three file types added above also needs Webpack configuration: change `webpack.config.js` so that the `module` property matches the different file types with regular expressions and picks the corresponding module loader to process them.

Take CSS as the example. When Webpack matches a file ending in `.css` and finds that several loaders need to process it, it calls them as a chain in **reverse** order: first `css-loader` compiles the CSS file brought in by `import './style.css'` in `index.js`, then the output is passed to the next module, `style-loader`, which adds the styles to the `<head>` tag of the final HTML template.

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

Finally, install the module-handling dependencies above in the same directory as `package.json`; once that succeeds, run `npx webpack` again and you'll see the output in `dist`.

```bash
yarn add style-loader css-loader file-loader csv-loader xml-loader -D
```

## Output management

So far we can handle the different types of files and assets brought into the project, but as the project grows, it can't always be one `src/index.js` bundled into one `dist/bundle.js` that takes care of everything; more modules and dependencies will come in, and at that point the code modules need to be split.

We call this kind of splitting output management; for this section's changes, see [`output-management`](https://github.com/L-Jovi/latte-web/tree/master/build/webpack/output-management).

To simulate an extra module referenced from the entry, we change `src/index.js` to:

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

Now, besides the third-party Lodash module, there's also a local module we wrote ourselves, `src/print.js`; here we just give it a simple print function.

```js
export default function printMe() {
  console.log('I get called from print.js!')
}
```

Up to the last section, we kept a hand-maintained `index.html` template in the bundled output directory `dist`. Now we let Webpack manage this step too: before each build, clear out every existing file in the output directory `dist`, then generate the HTML template with a plugin, linked to all of Webpack's final compiled output.

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

Note that Webpack uses the `plugins` property to call the right plugins at the right moments in the different stages of compilation. Unlike modules, the calls here run in order: first `clean-webpack-plugin` clears the existing content under `dist`, then `html-webpack-plugin` is called to generate the HTML template linked to the project's dependencies.

Install this section's new dependencies into `package.json` in the outer directory yourself, then run `npx webpack` and look at the HTML file in `dist`. By now every file in the output directory is managed by Webpack's bundling process.

## Development

## Code splitting

## Caching

## Hot Module Replacement (HMR)

## Tree shaking

## Shimming

## Plugins

## DLL

# How Webpack works

## AST

## Hooks

[^1]: [Front-end engineering - Wikipedia](https://en.wikipedia.org/wiki/Front-end_engineering)
[^2]: [Understanding (all) JavaScript module formats and tools](https://weblogs.asp.net/dixin/understanding-all-javascript-module-formats-and-tools)
[^3]: [ECMAScript - Wikipedia](https://en.wikipedia.org/wiki/ECMAScript)
[^4]: [Getting Started | webpack](https://webpack.js.org/guides/getting-started/)
[^5]: [Modules | webpack](https://webpack.js.org/concepts/modules/#what-is-a-webpack-module)