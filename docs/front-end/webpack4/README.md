# webpack4

## 说明

- 形式：本部分以教程形式书写。
- 适用：本教程适合想要深入 js 工具链的读者学习以入门`webpack4`。默认读者已经了解 npm，有原生 js 和`react`/`vue`开发经验。
- 思路：本教程书写思路是`是什么 -> 为什么 -> 怎么做`和`为什么 -> 是什么 -> 怎么做`，力求解决实际配置中的各类问题。
- 结构：本教程会以单页应用作示例，前面部分着重关注于基本使用，后面部分涉及原理，更多相关信息在章节末尾列写，可以自行查阅资料作进一步的学习。
- 环境：本篇教程默认使用 macOS，zsh，[oh-my-zsh](https://ohmyz.sh/)，[node](https://nodejs.org/en/) v12，[vscode](https://code.visualstudio.com/) 和 [chrome](https://www.google.com/chrome/browser/index.html)。另外使用 vscode 的 [live server 插件](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer)来测试构建后的文件，做法可参考其文档。
- 约定：本教程使用`${PROJECT_DIR}`表示项目根目录，一般认为`package.json`所处目录即为项目根目录。
- 范围：本教程不考虑 IE 11- 的浏览器。IE 11- 已经在 24 个月内没有得到官方支持，不应该再使用。此外，要支持 IE 11-，还要考虑怎么支持 es5+ 乃至 es3+ 的语法和特性，相当耗费时间。

## webpack 是什么

`webpack`是一个静态模块构建工具，分析依赖生成依赖图，最终根据配置进行构建。

## 为什么要使用构建工具

- 能支持新语法 syntax、新特性 feature
- 能自动处理图片、文件等资产文件
- 能使用 css 预处理器，自动添加 css 前缀
- 能压缩混淆
- 能帮助做构建文件的版本管理

总而言之，构建工具减少了重复的工作，使我们能投入更多的时间到开发工作中。

## 为什么选择 webpack

- 社区活跃度高
- 官方生态完整，社区生态丰富
- 配置灵活

## 基本概念

### 模块 module

`webpack`中，一切文件都可以被解析成模块`module`。

### 入口 entry

`entry`指定`webpack`工作时从哪个`.js`文件开始分析依赖，默认值为`${PROJECT_DIR}/src/index.js`。这个文件，也会被称为入口文件或入口模块。

```js
// 使用 path 模块来指定路径
const path = require('path');

module.exports = {
  // 指定 entry 为 ${PROJECT_DIR}/src/app.js
  entry: path.resolve('src', 'app.js'),
};

```

`path`是 node 的内置模块，我们可以在`webpack`的配置文件中使用`require`语句引用该模块。`path.resolve`是`path`模块内置的方法，它能将提供的字符串参数拼接起来，形成一个绝对路径，用于指定`entry`的值。

`path.join`与`path.resolve`用法、作用相似，使用频率也比较高。二者之间的主要区别是`path.join`仅仅拼接给出的字符串并返回，而`path.resolve`会解析并返回一个绝对路径。

下面给出示例帮助理解二者之间的主要区别。

```js
path.join('/a', '/b'); // string /a/b
path.resolve('/a', '/b'); // string /b

path.join("a", "b1", "..", "b2"); // string a/b2
path.resolve("a", "b1", "..", "b2"); // string ${PROJECT_DIR}/a/b2
```

### 输出 output

`output`可以指定`webpack`存放所有输出文件的路径`output.path`和输出的`.js`文件的路径`output.filename`。

`entry`对应的输出文件默认为`${PROJECT_DIR}/dist/main.js`。

```js
// 使用 path 模块来指定路径
const path = require('path');

module.exports = {
  // 指定 entry 为 ${PROJECT_DIR}/src/app.js
  entry: path.resolve('src', 'app.js'),
  // 指定 output 目录为 ${PROJECT_DIR}/dist
  // 指定 entry 对应的输出文件为 ${PROJECT_DIR}/dist/bundle.js
  output: {
    path: path.resolve('dist'),
    filename: 'bundle.js',
  },
};

```

### 加载器 loader

`webpack`本身只能解析`.js`和`.json`文件，`loader`增强了`webpack`的解析能力，使得 `webpack`能够解析`.jsx`，`.ts`，`.tsx`，`.css`等文件、将它们转换为模块、添加到依赖图中并供应用程序使用。

`module.rules`数组中的每一项中都是处理模块的规则，规则中会声明用到的`loader`。每一项有两个必需的属性，一个是`test`，用于指定需要解析的文件，值往往是一个正则表达式，另一个是`use`，指定用于解析的`loader`。

```js
// 使用 path 模块来指定路径
const path = require('path');

module.exports = {
  // 指定 entry 为 ${PROJECT_DIR}/src/app.js
  entry: path.resolve('src', 'app.js'),
  // 指定 output 目录为 ${PROJECT_DIR}/dist
  // 指定 entry 对应的输出文件为 ${PROJECT_DIR}/dist/bundle.js
  output: {
    path: path.resolve('dist'),
    filename: 'bundle.js',
  },
  module: {
    rules: [
      {
        // png 文件
        test: /\.png$/,
        // 使用 url-loader 处理
        use: 'url-loader',
      },
    ],
  },
};

```

有的`loader`可能还会与`plugin`有关联，也可能有额外的属性供你配置，具体信息要查看对应`loader`的文档。

### 插件 plugin

`plugin`用于执行范围更广的任务，比如打包优化，资源管理，注入环境变量等。

```js
// 使用 path 模块来指定路径
const path = require('path');
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = {
  // 指定 entry 为 ${PROJECT_DIR}/src/app.js
  entry: path.resolve('src', 'app.js'),
  // 指定 output 目录为 ${PROJECT_DIR}/dist
  // 指定 entry 对应的输出文件为 ${PROJECT_DIR}/dist/bundle.js
  output: {
    path: path.resolve('dist'),
    filename: 'bundle.js',
  },
  module: {
    rules: [
      {
        // png 文件
        test: /\.png$/,
        // 使用 url-loader 处理
        use: 'url-loader',
      },
    ],
  },
  plugins: [
    // 分析生成包大小
    new BundleAnalyzerPlugin({
      analyzerMode: 'static',
      defaultSizes: 'stat',
    }),
  ],
};

```

### 模式 mode

指定不同的模式，`webpack`会自动启用不同的优化，默认值为`production`，即生产模式。

模式一共有三种：`production`，`development`，`none`，优化程度由高到低为：`production`>`development`>`none`。

`none`不会启用优化，我们一般不会使用，而`production`和`development`的默认优化往往不能满足项目要求，还需要我们进一步定制。

```js
// 使用 path 模块来指定路径
const path = require('path');
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = {
  // 指定 mode 为 development，即开发模式
  mode: 'development',
  // 指定 entry 为 ${PROJECT_DIR}/src/app.js
  entry: path.resolve('src', 'app.js'),
  // 指定 output 目录为 ${PROJECT_DIR}/dist
  // 指定 entry 对应的输出文件为 ${PROJECT_DIR}/dist/bundle.js
  output: {
    path: path.resolve('dist'),
    filename: 'bundle.js',
  },
  module: {
    rules: [
      {
        // png 文件
        test: /\.png$/,
        // 使用 url-loader 处理
        use: 'url-loader',
      },
    ],
  },
  plugins: [
    // 分析生成包大小
    new BundleAnalyzerPlugin({
      analyzerMode: 'static',
      defaultSizes: 'stat',
    }),
  ],
};

```

### module，chunk，bundle

`bundle`指的是最终输出的一个或多个文件，也就是最终得到的代码块。

`chunk`则是打包过程中的代码块，它是某些`module`的封装，也可以称为某些`module`的集合。构建结束后，`chunk`就呈现为`bundle`。

一个`entry`会有一个或多个`chunk`，但最终只会生成一个`bundle`，但是这个`bundle`可能会包含多个文件。这是因为我们可能会把引用到的`.css`、`.js`文件分拆出来，也可能会添加`.map`文件。

## demo01 - 一个简单的 demo

前面简单地讲述了`webpack`的几个基本概念，下面开始实战。

首先安装 [nvm](https://github.com/nvm-sh/nvm)。nvm 是一个用于管理 node 版本的工具，使用 nvm 能免去频繁更换 node 版本的繁琐。

```sh
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash
```

如果你已经安装了 node，你也可以考虑完全卸载 node 之后安装 nvm。如果没有频繁更换 node 版本的需求，可以直接安装 node。

安装 nvm 之后，使用 nvm 来安装 node v12（写下这篇教程时 node 的长期支持版本）。

```sh
nvm install 12
```

新建一个文件夹，命名为`demo`。进入到该文件夹中，使用`npm`初始化，这将会生成一个默认的`package.json`文件。

```sh
mkdir demo # 新建一个名为 demo 的文件夹
cd demo # 进入该文件夹中
npm init -y # npm 初始化
```

根目录下新建一个`.npmrc`文件。该文件用于配置`npm`，比如指定依赖源等，这里我们指定依赖源为国内的淘宝源，这样安装依赖的速度会更快一点。

```sh
registry=https://registry.npm.taobao.org
disturl=https://npm.taobao.org/dist
sass_binary_site=https://npm.taobao.org/mirrors/node-sass/
phantomjs_cdnurl=https://npm.taobao.org/mirrors/phantomjs/
electron_mirror=https://npm.taobao.org/mirrors/electron/
electron_builder_binaries_mirror=https://npm.taobao.org/mirrors/electron-builder-binaries/
chromedriver_cdnurl=https://npm.taobao.org/mirrors/chromedriver
operadriver_cdnurl=https://npm.taobao.org/mirrors/operadriver
selenium_cdnurl=https://npm.taobao.org/mirrors/selenium
node_inspector_cdnurl=https://npm.taobao.org/mirrors/node-inspector
fsevents_binary_host_mirror=http://npm.taobao.org/mirrors/fsevents/

```

然后安装相关依赖。

```sh
npm i webpack@4 webpack-cli@3 copy-webpack-plugin@6 html-webpack-plugin@4 clean-webpack-plugin@3 webpackbar@4 friendly-errors-webpack-plugin@1 -DE
```

创建一个内容简单的`${PROJECT_DIR}/src/index.js`。

```js
document.write('Hello webpack!');

```

创建一个`webpack`配置文件`${PROJECT_DIR}/webpack.config.js`。不特意指定配置文件时，`webpack`会默认使用`${PROJECT_DIR}/webpack.config.js`。

```js
// 使用 path 模块来指定路径
const path = require('path');

module.exports = {
  // 指定 mode
  mode: 'production',
  // 指定 entry
  entry: path.resolve('src', 'index.js'),
  // 指定 output
  output: {
    path: path.resolve('dist'),
    filename: 'bundle.js',
  },
};

```

我们再来修改`package.json`中的`scripts`字段。这样，我们就能通过命令调用`webpack`构建。

```json
{
  ...,
  "scripts": {
    "build": "webpack"
  },
  ...
}
```

接着，就执行命令开始构建吧。

```sh
npm run build
```

构建结束后，我们可以看到，在`dist`目录下已经生成了一个`bundle.js`文件。但是我们还不能直接使用这个`bundle.js`文件，要使用它，我们还需要手动创建一个`.html`文件并且引用。

在实际开发中，每次构建后都耗费时间做这样的重复工作是难以忍受的。我们需要一些自动处理的手段，来帮我们自动生成`.html`文件并引入这个`bundle.js`文件。

在项目根目录下新建一个`public`文件夹，放入`favicon.ico`（可以自己随便找一个，或者把已有的图片转成 ico 格式）和`index.html`。

`index.html`如下所示。

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
    <link href="favicon.ico" type="image/x-icon" />
    <title>demo01</title>
  </head>
  <body></body>
</html>

```

接着，我们在`${PROJECT_DIR}/webpack.config.js`中配置`copy-webpack-plugin`和`html-webpack-plugin`，让它们来帮忙处理在`.html`文件中引入`.js`文件以及网站图标的问题。

```js
const CopyPlugin = require('copy-webpack-plugin');
const HtmlPlugin = require('html-webpack-plugin');

module.exports = {
  ...
  plugins: [
    // 复制 ${PROJECT_DIR}/public/favicon.ico
    new CopyPlugin({
      patterns: [{ from: path.resolve('public', 'favicon.ico') }],
    }),
    // 使用 ${PROJECT_DIR}/public/index.html 作为模板
    new HtmlPlugin({
      title: 'demo01',
      template: path.resolve('public', 'index.html'),
    }),
  ],
  ...
};

```

`copy-webpack-plugin`会帮我们把网站图标复制到输出目录下（在这个例子中，就是`${PROJECT_DIR}/dist`），而`html-webpack-plugin`会使用`${PROJECT_DIR}/public/index.html`作为承载`bundle.js`和网站 icon 的模板。

重新构建，构建结束后，我们会发现`${PROJECT_DIR}/dist`下出现了三个文件：`favicon.ico`，`index.html`和`bundle.js`，而`index.html`中还自动引入了`bundle.js`。

一切都很完美，但不能忽视的是，每次构建前都应该删除上一次构建的结果，也就是删除`${PROJECT_DIR}/dist`，否则可能会导致新的构建和旧的构建产生冲突、`${PROJECT_DIR}`占用空间越来越大不利于部署等问题。

我们可以使用`clean-webpack-plugin`来帮我们解决这个问题，默认地，`clean-webpack-plugin`会帮我们自动删除上一次构建的结果。

```js
const { CleanWebpackPlugin: CleanPlugin } = require('clean-webpack-plugin');

module.exports = {
  ...
  plugins: [
    // 移除上一次的构建文件
    new CleanPlugin(),
    ...
  ],
  ...
};

```

让`webpack`在打包的时候显示进度条也是常见的做法，这能够在一定程度上降低等待的焦虑度。

```js
const WebpackBar = require('webpackbar');

module.exports = {
  ...
  plugins: [
    // 显示进度条
    new WebpackBar(),
    ...
  ],
  ...
};

```

假如打包出现了问题，`webpack`将会输出一长串错误信息，这不利于快速地确认问题。使用`friendly-errors-webpack-plugin`可以使输出的错误信息更加友好。

```js
const FriendlyErrorsPlugin = require('friendly-errors-webpack-plugin');

module.exports = {
  ...
  plugins: [
    // 显示友好的错误信息
    new FriendlyErrorsPlugin(),
    ...
  ],
  ...
};

```

完整的`webpack.config.js`如下所示。

```js
// 使用 path 模块来指定路径
const path = require('path');
// 使用 plugin
const { CleanWebpackPlugin: CleanPlugin } = require("clean-webpack-plugin');
const CopyPlugin = require('copy-webpack-plugin');
const HtmlPlugin = require('html-webpack-plugin');
const WebpackBar = require('webpackbar');
const FriendlyErrorsPlugin = require('friendly-errors-webpack-plugin');

module.exports = {
  // 指定 mode
  mode: 'production',
  // 指定 entry
  entry: path.resolve('src', 'index.js'),
  // 指定 output
  output: {
    path: path.resolve('dist'),
    filename: 'bundle.js',
  },
  // 指定 plugin
  plugins: [
    // 显示友好的错误信息
    new FriendlyErrorsPlugin(),
    // 显示进度条
    new WebpackBar(),
    // 移除上一次的构建文件
    new CleanPlugin(),
    // 复制 ${PROJECT_DIR}/public/favicon.ico
    new CopyPlugin({
      patterns: [{ from: path.resolve('public', 'favicon.ico') }],
    }),
    // 使用 ${PROJECT_DIR}/public/index.html 作为模板
    new HtmlPlugin({
      title: 'demo01',
      template: path.resolve('public', 'index.html'),
    }),
  ],
};

```

重新开始构建，之后可以看到进度条和简短的提示信息。最终生成的`dist`目录的结构如下。使用`live server`来查看效果，可以看到`Hello webpack!`。

```sh
dist
├── bundle.js
├── favicon.ico
└── index.html
```

🎉恭喜，一个简单的 webpack demo 已经完成啦～

参考源代码见 [modyqyw/webpack-demos/demo01](https://github.com/ModyQyW/webpack4-demos/tree/master/demo01)。

## demo02 - 再看 webpack 核心概念与基础应用

### 入口 entry

一开始我们说到，`webpack`会分析依赖生成依赖图，而分析依赖生成依赖图的起点就由`entry`指定。

![webpack示意图](https://ae01.alicdn.com/kf/Hc2861d3d0e2b4ad89bfab9c37be5ecbcK.jpg)

单页面应用 SPA 只有单入口，此时`entry`是一个字符串 string 或一个对象 object。多页面应用 MPA 有多入口，此时`entry`只能是是一个对象 object。demo01 就是一个 SPA 的配置。

单入口也可以写成多入口的形式，demo01 中`entry`的写法等同于下面的代码。单入口默认的 key 是`main`。

```js
const path = require('path');

module.exports = {
  ...,
  entry: {
    main: path.resolve('src', 'index.js'),
  },
  ...,
};

```

`context`可能会跟`entry`一起使用，它可以指定`entry`的基本路径。对于 MPA，使用`context`往往能使`entry`更简洁。下面是一个示例。

```js
const path = require('path');

module.exports = {
  ...,
  context: path.resolve('src'),
  entry: './index.js',
  ...,
};

```

考虑到实际开发时必要的工程化，我们需要单独维护`webpack`配置相关的文件。我们先把没有使用到`context`的`webpack.config.js`移动到到`${PROJECT_DIR}/config`文件夹中，再指定`entry`的 key 为`app`。

```js
// ${PROJECT_DIR}/config/webpack.config.js
const path = require('path');

module.exports = {
  ...,
  entry: {
    app: path.resolve('src', 'index.js'),
  },
  ...,
};

```

`webpack`会默认使用`${PROJECT_DIR}/webpack.config.js`作为配置文件，我们移动配置文件后，就需要修改命令、手动指定配置文件了。

```json
{
  ...,
  "scripts": {
    "build": "webpack --config ./config/webpack.config.js"
  },
  ...
}

```

### 输出 output

分析依赖生成依赖图之后，`webpack`就会开始构建，构建结果如何输出就由`output`指定。

SPA 可以只简单地指定`filename`和`path`，如上面示例所示。而如果要构建 MPA，除了修改`entry`，还需要利用占位符确保文件名唯一。

下面代码中，`filename: '[name].js'`表示使用`entry`中配置的 key 来命名对应`entry`构建出来的文件。

```js
const path = require('path');

module.exports = {
  entry: {
    main: path.resolve('src', 'main.js'),
    app: path.resolve('src', 'app.js'),
  },
  output: {
    path: path.resolve('dist'),
    filename: '[name].js',
  },
};

```

此外，还需要修改`html-webpack-plugin`的配置，让每一个入口点都有一个专属的`.html`文件模板，并且还需要让每一个`.html`文件模板都插入公共代码块。

MPA 在配置上相对复杂，本篇教程只会以 SPA 作为示例，感兴趣的话可以自行搜索相关资料学习 MPA 的 `webpack`构建配置。

我们修改一下`output`的配置，使得`entry`指定的文件构建完毕后的命名跟随`entry`的 key。

```js
const path = require('path');

module.exports = {
  ...,
  entry: {
    app: path.resolve('src', 'index.js'),
  },
  output: {
    path: path.resolve('dist'),
    filename: '[name].js',
  },
  ...,
};

```

### 插件 plugin

`plugin`用于增强`webpack`功能，比如优化打包文件，管理资源，注入环境变量等等，作用于整个构建过程。

前面的例子中，我们使用到了`copy-webpack-plugin`，`html-webpack-plugin`等相对来说比较简单的`plugin`。

每个`plugin`都需要放入到`plugins`字段数组中，顺序一般不影响，具体的配置需要去查询具体的文档。

### 加载器 loader

由于`webpack`默认只支持解析`.js`和`.json`文件，所以项目中使用到的其他文件，比如图片文件、字体文件、样式文件等，就只能使用`loader`解析。

下面将会关注一些常用的`loader`。

### js 新语法和新特性相关的 loader

因为`webpack`本身并不支持解析 es6+ 语法，所以要使用 es6+ 语法，我们就需要使用`babel`和`babel-loader`。

`babel`一个主要作用就是转换新语法为旧语法，也就是我们常说的语法转换，比如把箭头函数的写法转换成`function`的写法。`babel-loader`使得`webpack`在构建过程中能使用`babel`处理相关的文件。

首先还是要安装相关的依赖。

```sh
npm i @babel/runtime@7 core-js@3 regenerator-runtime@0.11.1 react@16.13.1 react-dom@16.13.1 -E
npm i @babel/cli@7 @babel/core@7 @babel/plugin-transform-runtime@7 @babel/preset-env@7 @babel/preset-react@7 babel-loader@8 @types/react@16 @types/react-dom@16 -DE
```

其次是修改`webpack`配置。不要忘记，对于`webpack`而言，所有文件都可以视作一个模块，所以需要在`module.rules`中配置`loader`。

```js
// ${PROJECT_DIR}/config/webpack.config.js
module.exports = {
  ...
  // 指定 loader
  module: {
    rules: [
      ...,
      {
        // js 和 jsx 文件
        test: /\.jsx?$/,
        // 不处理 node_modules 和 bower_components
        exclude: /(node_modules|bower_components)/,
        // 使用 babel-loader 进行处理
        use: [{ loader: 'babel-loader' }],
      },
      ...
    ],
  },
};

```

不处理`node_modules`和`bower_components`中的`.js`文件能够有效地提高编译效率，同时避免可能存在的重新编译问题。

修改完`webpack`配置后，我们还需要配置`babel`，让它根据我们的需求进行转译。我们创建`${PROJECT_DIR}/babel.config.json`作为`babel`的配置文件。

要支持 es6+ 语法非常简单，可以直接使用`babel`官方提供的`@babel/preset-env`。它能智能转换 es6+ 语法到 es5 语法，无需提供额外的配置。

```json
{
  "presets": ["@babel/preset-env"]
}

```

但现实往往是残酷的，`@babel/preset-env`默认的配置通常不能准确满足项目需求。一个比较常见的项目需求是支持特定的浏览器和特定的浏览器版本，比如 ie 11。

这个时候，我们就需要先向`@babel/preset-env`说明目标浏览器（我们想要支持的浏览器），也就是语法转换后的代码能够跑在什么浏览器上。

怎么说明？我们可以创建`${PROJECT_DIR}/.browserslistrc`文件。`.browserslistrc`文件是一个特殊的文件，依赖于`browserslist`，它的内容说明了项目的目标浏览器。如果存在该文件，它的内容就会被`@babel/preset-env`读取并使用。

我们为`.browserslistrc`添加以下内容。

```sh
> 0.2%
last 3 versions
not dead
chrome >= 70
firefox >= 68
edge >= 81
safari >= 13
ie >= 11

```

- `> 0.2%`表示需要支持使用率超过 0.2% 的浏览器。
- `last 3 versions`表示需要支持浏览器的最近 3 个版本。
- `not dead`表示浏览器在最近 24 个月内还得到过官方的支持。
- `chrome >= 70`表示 chrome 的版本需要不小于 70。
- `firefox >= 68`表示 firefox 的版本需要不小于 68。
- `edge >= 81`表示 edge 的版本需要不小于 81。
- `safari >= 13`表示 safari 的版本需要不小于 13。
- `ie >= 11`表示 ie 的版本需要不小于 11。

上面的条件取并集，就是需要支持的浏览器范围。

向`@babel/preset-env`说明完目标浏览器之后，`babel`在转换语法时会更加精准，它会把目标浏览器不支持的 es6+ 语法转换成 es5 语法，同时保留目标浏览器支持的 es6+ 语法。

但是如果转换后的代码中存在浏览器不支持的特性，比如`Promise`，那该怎么办呢？这个时候，`babel`的另一个作用，自动补齐特性，就很好地解决了这个问题。

polyfill 指的是能够提供一些浏览器本身没有的新特性的 js 代码包。我们可以配置`babel`自动引入 polyfill 来自动补齐目标浏览器的特性。

`@babel/preset-env`默认只会转换语法，我们需要手动配置来启用自动补全特性的功能。这里我们指定`useBuiltIns`为`usage`模式，这样做的好处是`@babel/preset-env`会为我们自动按需引入 polyfill，省去了不少麻烦。

```json
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "useBuiltIns": "usage"
      }
    ]
  ]
}

```

默认地，`@babel/preset-env`会使用`core-js` v2 和`regenerator-runtime`做 polyfill。但是`core-js` v3 提供的 polyfill 更多，同时减少了全局污染，现在一般建议使用 v3。这里我们就指定要使用 v3 版本。

```json
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "useBuiltIns": "usage",
        "corejs": { "version": 3, "proposals": true }
      }
    ]
  ]
}

```

之后，`babel` 会为我们自动引入`core-js` v3 和`regenerator-runtime`中和项目代码关联的部分，自动补全浏览器特性。

但使用这样的配置构建出来的代码还不能投入到生产环境中。自动补全浏览器特性之后可能会使得每个文件头部都增加了相同的代码，比如多个文件都使用了`Promise`，转译之后就会在这些文件的头部都引入相同的`Promise`相关的 polyfill。这些重复的代码会影响最终构建包的体积，在实际开发中是难以接受的。

我们可以使用`@babel/plugin-transform-runtime`来抽离这些重复的代码并放到一起，进而压缩最终构建包的体积。

```json
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "useBuiltIns": "usage",
        "corejs": { "version": 3, "proposals": true }
      }
    ]
  ],
  "plugins": ["@babel/plugin-transform-runtime"]
}

```

除了 es6+ 的语法，我们还想支持`react`语法。类似地，我们也可以使用`babel`来解析`react`代码，只需要根据文档配置`@babel/preset-react`即可。

```json
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "useBuiltIns": "usage",
        "corejs": { "version": 3, "proposals": true }
      }
    ],
    "@babel/preset-react"
  ],
  "env": {
    "development": {
      "presets": [["@babel/preset-react", { "development": true }]]
    }
  },
  "plugins": ["@babel/plugin-transform-runtime"]
}

```

之后可以修改`${PROJECT_DIR}/src/index.js`，使用`react`，`react-dom`，`Promise`以测试我们的配置。

```js
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';

class App extends React.Component {
  componentDidMount() {
    new Promise((resolve) => {
      setTimeout(() => {
        document.title = 'Hello World!';
        resolve();
      }, 5000);
    });
  }

  render() {
    return (
      <div className="container">
        <p>Hello Webpack!</p>
      </div>
    );
  }
}

ReactDOM.render(<App />, document.getElementById('root'));

```

你也可以使用函数式组件 FC 书写。

```js
import React, { useEffect } from 'react';
import ReactDOM from 'react-dom';
import './index.scss';

const App = () => {
  useEffect(() => {
    new Promise((resolve) => {
      setTimeout(() => {
        document.title = 'Hello World!';
        resolve();
      }, 5000);
    });
  }, []);

  return (
    <div className="container">
      <p>Hello Webpack!</p>
    </div>
  );
};

ReactDOM.render(<App />, document.getElementById('root'));

```

之后构建并运行测试即可。

如果正常，可以看到页面上会出现`Hello Webpack!`的文字，5 秒左右标签页的标题被修改成`Hello World!`。

我们查看`${PROJECT_DIR}/dist/report.html`会看到`es.promise.js`也加入了`bundle`中，这是因为我们给出的目标浏览器包含了`ie >= 11`，而 IE 11 是不支持`Promise`的。

可能你还会有疑虑，那已经支持了`Promise`特性的浏览器会再度引入这部分 polyfill 吗？不会，polyfill 会聪明地先判断浏览器环境，如果不支持这部分特性再引入。

当然，对比起官方文档和实际大型应用开发需求，教程这部分还相当简陋，建议还是多多阅读文档多多实践。

### 样式相关的 loader

因为`webpack`本身并不支持解析`.css`，`.sass`和`.scss`文件，所以我们需要使用`loader`去解析。

`.less`和`.styl`的配置也相差不大，自己参考对应`loader`的文档摸索即可。

首先还是要安装相关的依赖。

```sh
npm i zent@8 -E
npm i style-loader@1 css-loader@3 sass@1 sass-loader@9 resolve-url-loader@3 babel-plugin-zent@2 -DE
```

`css-loader`能够解析`.css`文件成 css 模块，`style-loader`能够将 css 模块嵌入到文件中。

我们先在`${PROJECT_DIR}/src/index.js`引入`.css`文件。

```js
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';

const App = () => (
  <div className="container">
    <p>Hello Webpack!</p>
  </div>
);

ReactDOM.render(<App />, document.getElementById('root'));

```

然后创建`${PROJECT_DIR}/src/index.css`，内容如下所示。

```css
*,
::before,
::after {
  box-sizing: border-box;
  padding: 0;
  margin: 0;
}

html,
body {
  min-width: 1280px;
  min-height: 100%;
}

.container {
  width: 100%;
}

```

最后，我们修改一下`webpack`配置，增加对`.css`文件的解析。

```js
module.exports = {
  ...,
  // 指定 loader
  module: {
    rules: [
      ...,
      {
        // css 文件
        test: /\.css$/,
        // 先使用 css-loader 再使用 style-loader 处理
        use: [{ loader: 'style-loader' }, { loader: 'css-loader' }],
      },
      ...
    ],
  },
};

```

默认地，`.js`文件引用了 css，转换后的 css 模块会被嵌入到`.js`文件中，然后再生成标签嵌入到`<head>`标签中，你也可以通过修改`style-loader`地配置来自定义这一行为。

重新构建并运行，我们可以在浏览器控制台中看到，样式被插入到`<head>`标签中，内容与我们书写的一致，并且已经起了作用。

值得注意的是，如果要使用多个`loader`处理某种文件，`loader`的顺序应该是从后往前的，上面的示例中，会先调用`css-loader`处理`.css`文件，再调用`style-loader`做进一步处理。

要处理`.sass`和`.scss`文件，又有少许的不同。因为`sass-loader`会把`.sass`和`.scss`文件转换成`.css`文件，而`.css`文件的处理步骤就跟上面一致。所以，我们首先需要复制粘贴，并在最后加上相应的`.loader`配置。

```js
module.exports = {
  ...,
  // 指定 loader
  module: {
    rules: [
      ...,
      {
        // css 文件
        test: /\.css$/,
        // 先使用 css-loader 再使用 style-loader 处理
        use: [{ loader: 'style-loader' }, { loader: 'css-loader' }],
      },
      {
        // sass/scss 文件
        test: /\.s[ac]ss$/,
        // 依次使用 sass-loader，css-loader 和 style-loader 处理
        use: [
          { loader: 'style-loader' },
          { loader: 'css-loader' },
          { loader: 'sass-loader' },
        ],
      },
      ...,
    ],
  },
};

```

由于`sass-loader`会处理`@import`语句，所以我们还需要配置`css-loader`，说明在`css-loader`之前有多少`loader`会处理`@import`语句。

```js
module.exports = {
  ...,
  // 指定 loader
  module: {
    rules: [
      ...,
      {
        // css 文件
        test: /\.css$/,
        // 先使用 css-loader 再使用 style-loader 处理
        use: [{ loader: 'style-loader' }, { loader: 'css-loader' }],
      },
      {
        // sass/scss 文件
        test: /\.s[ac]ss$/,
        // 依次使用 sass-loader，css-loader 和 style-loader 处理
        use: [
          { loader: 'style-loader' },
          {
            loader: 'css-loader',
            options: {
              importLoaders: 1,
            },
          },
          { loader: 'sass-loader' },
        ],
      },
      ...,
    ],
  },
};

```

由于 sass 并没有提供 url 重写的功能，我们还需要配置`resolve-url-loader`，否则可能会在实际使用出现 url 指向不正确的问题。注意：`resolve-url-loader`并不会处理`@import`语句，所以无需再修改`css-loader`的`importLoaders`配置。

```js
module.exports = {
  ...,
  // 指定 loader
  module: {
    rules: [
      ...,
      {
        // css 文件
        test: /\.css$/,
        // 先使用 css-loader 再使用 style-loader 处理
        use: [{ loader: 'style-loader' }, { loader: 'css-loader' }],
      },
      {
        // sass/scss 文件
        test: /\.s[ac]ss$/,
        // 依次使用 sass-loader，resolve-url-loader，css-loader 和 style-loader 处理
        use: [
          { loader: 'style-loader' },
          {
            loader: 'css-loader',
            options: {
              importLoaders: 1,
            },
          },
          { loader: 'resolve-url-loader' },
          { loader: 'sass-loader' },
        ],
      },
      ...,
    ],
  },
};

```

我们把`index.css`重命名为`index.scss`并修改`${PROJECT_DIR}/src/index.js`中的引入。重新构建并测试，一切正常。

我们再来试着添加并使用`zent`。首先修改`${PROJECT_DIR}/src/index.js`，加入一个简单的带图标的按钮。

```js
import React from 'react';
import ReactDOM from 'react-dom';
import { Button, Icon } from 'zent';
import './index.scss';

const App = () => (
  <div className="container">
    <p>Hello Webpack!</p>
    <Button type="primary">
      <Icon type="youzan" />
      Hello Zent!
    </Button>
  </div>
);

ReactDOM.render(<App />, document.getElementById('root'));

```

之后修改`${PROJECT_DIR}/babel.config.json`，根据`zent`的官网说明加入按需加载的优化。

```json
{
  ...,
  "plugins": [
    "@babel/plugin-transform-runtime",
    [
      "zent",
      {
        "libraryName": "zent",
        "noModuleRewrite": false,
        "automaticStyleImport": true,
        "useRawStyle": true
      }
    ]
  ]
}

```

重新构建并测试，我们能看到一个蓝色的按钮，按钮内左边是有赞的图标，文字是`Hello Zent`。

可能有人会问，为什么不用`antd`作示例。第一是因为我认为`zent`使用的 scss 的语法比`antd`使用的 less 的更接近 css，第二是因为`antd`曾经出过圣诞彩蛋事件，目前对我来说没有可信度。

### 资产相关的 loader

一般称项目使用到的图片、字体、音频、视频等为项目资产。

最常用的处理资产的`loader`就是`file-loader`和`url-loader`。`url-loader`是`file-loader`的升级版，增加了文件大小的上限配置，达到大小上限时会自动使用`file-loader`，没达到大小上限时，会把文件转换成 base64 数据并硬编码进代码中。

```sh
npm i file-loader@6 url-loader@4 -DE
```

安装依赖之后，直接修改配置文件即可。

```js
module.exports = {
  ...,
  // 指定 loader
  module: {
    rules: [
      ...,
      {
        // 图片文件
        test: /\.(png|jpg|jpeg|gif)$/,
        // 使用 url-loader 处理
        use: [
          {
            loader: 'url-loader',
            options: {
              // 8 MB 上限
              limit: 8192,
              // 放入 ${output.path}/img 文件夹中
              outputPath: 'img',
              // 使用 ${output.path}/img 文件夹中的图片
              publicPath: 'img',
            },
          },
        ],
      },
      {
        // 字体文件
        test: /\.(woff|woff2|eot|ttf|otf)$/,
        // 使用 url-loader 处理
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 8192,
              outputPath: 'fonts',
              publicPath: 'fonts',
            },
          },
        ],
      },
    ],
    ...,
  },
  ...,
}
```

注意：`outputPath`和`publicPath`是和项目中配置的输出路径`output.path`相关的，在这个项目里，就是和`${PROJECT_DIR}/dist`相关的。我们把图片和字体放入各自的文件夹中，就是为了区分开不同类型的文件。

为什么要转换成 base64 数据并硬编码进代码呢？一方面，直接硬编码进代码可以避免在读取该部分文件时的页面闪烁，提高用户体验，另一方面也可以减少网络请求，降低服务器压力。

如果 base64 数据太多太大，加载网页的速度依旧会变慢，所以不是所有文件都适宜转换成 base64 数据。

在`${PROJECT_DIR}/src/assets`中放入一个图片文件（我这里放入了`webpack.png`），然后在`${PROJECT_DIR}/src/index.js`中引入并使用它。

```js
import React from 'react';
import ReactDOM from 'react-dom';
import { Button, Icon } from 'zent';
import iconWebpack from './assets/webpack.png';
import './index.scss';

const App = () => (
  <div className="container">
    <p>Hello Webpack!</p>
    <img src={iconWebpack} />
    <Button type="primary">
      <Icon type="youzan" />
      Hello Zent!
    </Button>
  </div>
);

ReactDOM.render(<App />, document.getElementById('root'));

```

在`${PROJECT_DIR}/src/assets`中放入一个字体文件（我这里放入了阿里普惠体的字体文件`Alibaba-PuHuiTi-Regular.ttf`），然后在`${PROJECT_DIR}/src/index.scss`中引入并使用它。

```scss
@font-face {
  font-family: "Alibaba PuHuiTi";
  src: url("./assets/Alibaba-PuHuiTi-Regular.ttf") format("ttf");
}

*,
::before,
::after {
  box-sizing: border-box;
  padding: 0;
  margin: 0;
}

html,
body {
  min-width: 1280px;
  min-height: 100%;
  font-family: "Alibaba PuHuiTi", -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, 'Noto Sans', sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', 'Noto Color Emoji';
}

.container {
  width: 100%;
}

```

重新构建，可以看到`${PROJECT_DIR}/dist`目录下额外多出了两个文件夹`fonts`和`img`，里面分别是一个字体文件和一个图片文件，名字被修改成一串字符串，这个我们称为文件指纹，会在之后做进一步的解释。测试时一切正常。

`url-loader`和`file-loader`只会处理`.js`中引用的图片，如果我们在`.html`里直接引用呢？那就只能使用`html-loader`来处理了。这种情况较为少见，可以自行查阅相关资料学习。

### 模式 mode

指定不同的模式，`webpack`会自动启用不同的功能进行优化，默认值为`production`，默认取值范围为`development`，`production`和`none`。

现在，我们每一次要查看代码效果，都需要执行`npm run build`，然后用`live server`启动。这相当地麻烦，尤其是在开发过程中，这么做会耗费不必要的时间，而且开发时也不应该使用`production`模式，而应该使用`development`模式。

`webpack-dev-server`帮我们解决了这个问题。使用`webpack-dev-server`可以不刷新浏览器就看到我们开发时修改代码后的结果（这也就是我们常说的热更新），也不会生成文件放到`dist`目录下（会把生成文件放到内存中）。

```sh
npm i cross-env@7 webpack-bundle-analyzer@3 webpack-dev-server@3 webpack-merge@5 -DE
```

我们还需要根据环境来调用不同的构建配置。基于可维护性考虑，我们应该拆分出不同环境的构建配置文件，最终根据环境暴露出对应环境的构建配置。

首先修改`package.json`。`cross-env`可以修改`process.env.NODE_ENV`的值，进而供我们确认环境。

```json
{
  ...,
  "scripts": {
    "dev": "cross-env NODE_ENV=development webpack-dev-server --config ./config/webpack.config.js",
    "build": "cross-env NODE_ENV=production webpack --config ./config/webpack.config.js"
  },
  ...
}

```

接着，我们把原本`${PROJECT_DIR}/config/webpack.config.js`中除`mode`之外的内容抽离出来，放入`${PROJECT_DIR}/config/webpack.base.js`中，作为基础配置。

再新建两个配置文件如下。

`${PROJECT_DIR}/config/webpack.dev.js`：

```js
const { merge } = require('webpack-merge');
const baseConfig = require('./webpack.base.js');

module.exports = merge(baseConfig, {
  mode: 'development',
  devServer: {
    hot: true,
    quiet: true,
  },
  devtool: 'eval-cheap-source-map',
});

```

- `hot: true`表示开启`webpack-dev-server`的热更新。
- `quiet: true`表示减少构建输出的信息显示。

`devtool`可以帮助调试，这里指定为`eval-cheap-source-map`，有兴趣可以自行查阅相关资料作进一步学习，结尾处也有给出参考文章。

`webpack-merge`会帮助我们自动合并相关字段的配置，这样就使得`webpack-dev-server`也会使用基础配置中的`plugin`和`loader`。

`${PROJECT_DIR}/config/webpack.prod.js`也十分类似，指定了`mode`，`devtool`还有额外的 `plugin`。额外的`plugin`会被`webpack-merge`使用，与基础配置组合。

```js
const { merge } = require('webpack-merge');
const baseConfig = require('./webpack.base.js');
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = merge(baseConfig, {
  mode: 'production',
  devtool: 'cheap-source-map',
  plugins: [
    new BundleAnalyzerPlugin({
      analyzerMode: 'static',
      defaultSizes: 'stat',
    }),
  ],
});

```

最后修改`${PROJECT_DIR}/config/webpack.config.js`，让它在不同环境时暴露不同的构建配置。

```js
const devConfig = require('./webpack.dev.js');
const prodConfig = require('./webpack.prod.js');

if (process.env.NODE_ENV === 'development') {
  module.exports = devConfig;
} else {
  module.exports = prodConfig;
}

```

最后分别执行`npm run dev`和`npm run build`做测试，一切正常。最终项目目录如下所示。

```sh
.
├── babel.config.json
├── config
│   ├── webpack.base.js
│   ├── webpack.config.js
│   ├── webpack.dev.js
│   └── webpack.prod.js
├── dist
│   ├── app.js
│   ├── app.js.map
│   ├── favicon.ico
│   ├── fonts
│   ├── img
│   ├── index.html
│   └── report.html
├── package-lock.json
├── package.json
├── public
│   ├── favicon.ico
│   └── index.html
└── src
    ├── assets
    ├── index.js
    └── index.scss
```

对于`react`，还可以加入`react-hot-loader`进一步提升使用体验。有兴趣可自行查阅相关资料学习。

🎉恭喜，你的第二个 webpack demo 已经完成啦～

参考源代码见 [modyqyw/webpack-demos/demo02](https://github.com/ModyQyW/webpack4-demos/tree/master/demo02)。

## demo03 - 优化以贴近实际工程

### 使用文件指纹做版本管理

人的指纹是特殊的，不存在完全相同的人的指纹，依靠人的指纹可以确定唯一的人。文件指纹的用途和人的指纹相近，可以用于版本管理。

常用的文件指纹有三类：

- `hash` - 与整个项目的构建有关，只要项目中文件有修改，值就会变化。特别地，对于图片、字体等可以被`url-loader`和`file-loader`处理的文件，`hash`表示的是文件内容，与整个项目的构建无关。因为起不到缓存效果和管理版本的作用，所以不建议用于资产文件外的文件。
- `chunkhash` - 根据不同的`chunk`生成`hash`，通常会把不常变动的公共库单独抽离，然后对业务代码使用`chunkhash`，这样改动业务代码不会影响公共库，客户端只需更新业务代码对应的`chunk`。
- `contenthash` - 根据文件内容生成`hash`。`.js`文件常常会引用`.css`文件，如果使用`chunkhash`，会导致修改`.js`文件、没有修改`.css`文件的时候，最终构建完发现`.css`文件的`hash`也变化了，所以 css 文件一般使用`contenthash`。

我们先来修改`${PROJECT_DIR}/config/webpack.base.js`，为图片和字体文件添加文件指纹。

```js
module.exports = {
  ...,
  module: {
    rules: [
      ...,
      {
        test: /\.(png|jpg|jpeg|gif)$/,
        use: [
          {
            loader: 'url-loader',
            options: {
              ...,
              name: '[name].[contenthash:8].[ext]',
            },
          },
        ],
      },
      {
        test: /\.(woff|woff2|eot|ttf|otf)$/,
        use: [
          {
            loader: 'url-loader',
            options: {
              ...,
              name: '[name].[contenthash:8].[ext]',
            },
          },
        ],
      },
      ...,
    ],
  },
};

```

- `[name]`表示使用文件本身的命名。
- `[contenthash:8]`表示使用`contenthash`的前 8 位，也可以写成`[hash:8]`，结果将会是一样的，这是因为`url-loader`和`file-loader`将会以同样的方式处理`contenthash`和`hash`，这是文件指纹中的一个特例。
- `[ext]`表示使用文件本身的后缀。

需要注意的是，要在生产模式下为`.css`文件添加文件指纹，就不能使用`style-loader`，这是因为`style-loader`会把`.css`文件嵌入到`.js`文件中，我们无法得到单独的`.css`文件，自然也就无法添加文件指纹了。

要解决这个问题，我们需要添加一个依赖`mini-css-extract-plugin`，使用它在项目生产环境中分离`.css`文件，然后让`style-loader`只在开发环境中起作用。

```sh
npm i mini-css-extract-plugin@0 -DE
```

我们再把`${PROJECT_DIR}/config/webpack.base.js`中关于 css 的部分都放入`${PROJECT_DIR}/config/webpack.dev.js`中。

现在，完整的`${PROJECT_DIR}/config/webpack.base.js`如下所示，包含了`entry`，`plugin`和`loader`。其中，图片文件和字体文件的处理都使用了`contenthash`的前 8 位。

注意：我们只是移除了其中关于 css 的部分，给出完整的文件只是谨慎起见，避免移除错误。

```js
const path = require('path');
const { CleanWebpackPlugin: CleanPlugin } = require('clean-webpack-plugin');
const CopyPlugin = require('copy-webpack-plugin');
const HtmlPlugin = require('html-webpack-plugin');
const WebpackBar = require('webpackbar');
const FriendlyErrorsPlugin = require('friendly-errors-webpack-plugin');

module.exports = {
  entry: {
    app: path.resolve('src', 'index.js'),
  },
  plugins: [
    new FriendlyErrorsPlugin(),
    new WebpackBar(),
    new CleanPlugin(),
    new CopyPlugin({
      patterns: [{ from: path.resolve('public', 'favicon.ico') }],
    }),
    new HtmlPlugin({
      title: 'demo03',
      template: path.resolve('public', 'index.html'),
    }),
  ],
  module: {
    rules: [
      {
        test: /\.jsx?$/,
        exclude: /(node_modules|bower_components)/,
        use: [{ loader: 'babel-loader' }],
      },
      {
        test: /\.(png|jpg|jpeg|gif)$/,
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 8192,
              outputPath: 'img',
              publicPath: 'img',
              name: '[name].[contenthash:8].[ext]',
            },
          },
        ],
      },
      {
        test: /\.(woff|woff2|eot|ttf|otf)$/,
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 8192,
              outputPath: 'fonts',
              publicPath: 'fonts',
              name: '[name].[contenthash:8].[ext]',
            },
          },
        ],
      },
    ],
  },
};

```

完整的`${PROJECT_DIR}/config/webpack.dev.js`如下所示，除去基本的配置外，还声明了`mode`，`webpack-dev-server`，`devtool`和`loader`。在这里，我们使用了`style-loader`。

```js
const { merge } = require('webpack-merge');
const baseConfig = require('./webpack.base.js');

module.exports = merge(baseConfig, {
  mode: 'development',
  devServer: {
    hot: true,
    open: true,
  },
  devtool: 'eval-cheap-source-map',
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [{ loader: 'style-loader' }, { loader: 'css-loader' }],
      },
      {
        test: /\.s[ac]ss$/,
        use: [
          { loader: 'style-loader' },
          {
            loader: 'css-loader',
            options: {
              importLoaders: 1,
            },
          },
          { loader: 'resolve-url-loader' },
          { loader: 'sass-loader' },
        ],
      },
    ],
  },
});

```

我们再来修改`${PROJECT_DIR}/config/webpack.prod.js`，不使用`style-loader`而是使用`mini-css-extract-plugin`，并为输出的`.js`文件还有`.css`文件添加文件指纹。

首先用`mini-css-extract-plugin`附带的`loader`替换掉原本使用的`style-loader`。我们还要指定`publicPath`，用于指定要读取的`.css`文件所处的文件夹。把`.css`文件放入到特定的文件夹中，有利于区分开不同类型的文件。

```js
module.exports = merge(baseConfig, {
  ...,
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          {
            loader: MiniCssExtractPlugin.loader,
            options: {
              publicPath: 'css',
            },
          },
          { loader: 'css-loader' },
        ],
      },
      {
        test: /\.s[ac]ss$/,
        use: [
          {
            loader: MiniCssExtractPlugin.loader,
            options: {
              publicPath: 'css',
            },
          },
          {
            loader: 'css-loader',
            options: {
              importLoaders: 1,
            },
          },
          { loader: 'sass-loader' },
        ],
      },
    ],
  },
});

```

接着，把`mini-css-extract-plugin`加入到`plugins`中，并指定输出文件名。注意：在前面我们已经指定要使用`${PROJECT_DIR}/dist/css`文件夹内的`.css`文件，在这里我们需要把文件夹名也添加上去，让`.css`文件输出到`${PROJECT_DIR}/dist/css`目录下，否则仍然会直接输出到`${PROJECT_DIR}/dist`目录下，进而导致引用错误。

```js
module.exports = merge(baseConfig, {
  ...,
  plugins: [
    ...,
    new MiniCssExtractPlugin({
      filename: 'css/[name].[contenthash:8].css',
    }),
    ...,
  ],
  ...,
});

```

要为`entry`对应的输出文件添加文件指纹非常简单，只需要直接使用`chunkhash`即可。

完整的`${PROJECT_DIR}/config/webpack.prod.js`如下所示。其中，`.css`文件的处理都使用了`contenthash`的前 8 位。

```js
const path = require('path');
const { merge } = require('webpack-merge');
const baseConfig = require('./webpack.base.js');
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;
const MiniCssExtractPlugin = require('mini-css-extract-plugin');

module.exports = merge(baseConfig, {
  mode: 'production',
  devtool: 'cheap-source-map',
  output: {
    path: path.resolve('dist'),
    filename: '[name].[chunkhash:8].js',
  },
  plugins: [
    new BundleAnalyzerPlugin({
      analyzerMode: 'static',
      defaultSizes: 'stat',
    }),
    new MiniCssExtractPlugin({
      filename: 'css/[name].[contenthash:8].css',
    }),
  ],
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          {
            loader: MiniCssExtractPlugin.loader,
            options: {
              publicPath: 'css',
            },
          },
          { loader: 'css-loader' },
        ],
      },
      {
        test: /\.s[ac]ss$/,
        use: [
          {
            loader: MiniCssExtractPlugin.loader,
            options: {
              publicPath: 'css',
            },
          },
          { loader: 'css-loader' },
          { loader: 'resolve-url-loader' },
          { loader: 'sass-loader' },
        ],
      },
    ],
  },
});

```

这仅仅是最简单的处理，在实际应用中，我们还可能会遇到更复杂的情况需要我们来处理。

### 移除 js 中的注释以压缩 js

如果我们构建之后打开`${PROJECT_DIR}/dist/js/app.[chunkhash:8].js`，就会发现大量 js 代码堆叠到一起，这是正常的压缩现象。但是，文件中还存在有一些注释，这是生产环境中不需要的，我们还需要手动配置来去除掉这些注释。

`webpack`默认在生产环境下使用`terser-webpack-plugin`来压缩`.js`文件，我们只需要做进一步的配置即可。

虽然安装`webpack`依赖会一并安装该依赖，但是我们通常会显式安装我们所需要的依赖，避免可能的版本问题。

```sh
npm i terser-webpack-plugin@3 -DE
```

我们无需从头配置`terser-webpack-plugin`，而是修改`webpack`原本的`terser-webpack-plugin`配置，所以我们是在`optimization`字段中（而不是在`plugins`字段中）使用`terser-webpack-plugin`。

```js
const TerserPlugin = require('terser-webpack-plugin');

module.exports = {
  ...,
  optimization: {
    minimize: true,
    minimizer: [
      new TerserPlugin({
        terserOptions: {
          output: {
            comments: false,
          },
        },
        extractComments: false,
      }),
    ],
  },
  ...,
};

```

简单地配置`terser-webpack-plugin`，就可以使得最终输出的结果没有注释。

这个配置也是`terser-webpack-plugin`文档中提供的例子，而这只是`terser-webpack-plugin`功能的冰山一角。感兴趣的话可以自行查阅相关资料学习。

### 压缩 html

要压缩 html，我们可以使用之前用到的`html-webpack-plugin`。除了我们之前用到的指定模板的功能，它还有压缩 html 的功能，而且是默认开启的。

如果我们需要修改`html-webpack-plugin`的压缩选项，我们只需要为`${PROJECT_DIR}/config/webpack.base.js`中的`html-webpack-plugin`的配置添加一个`minify`字段，然后写入自己的配置即可。

下面是默认的`html-webpack-plugin`的压缩选项。

```js
const HtmlPlugin = require('html-webpack-plugin');

module.exports = {
  ...,
  plugins: [
    ...,
    new HtmlPlugin({
      title: 'demo03',
      template: path.resolve('public', 'index.html'),
      minify: {
        collapseWhitespace: true,
        removeComments: true,
        removeRedundantAttributes: true,
        removeScriptTypeAttributes: true,
        removeStyleLinkTypeAttributes: true,
        useShortDoctype: true,
      },
    }),
    ...,
  ],
  ...,
};

```

- `collapseWhitespace: true`表示减少 html 中不必要的空白。
- `removeComments: true`表示移除 html 中的注释。
- `removeRedundantAttributes: true`表示移除标签上使用了默认值的属性，比如`<input type="text" />`，可以移除`type="text`。
- `removeScriptTypeAttributes: true`表示移除`<script>`标签上的`type="text/javascript"`。
- `removeStyleLinkTypeAttributes: true`表示移除`<style>`和`<link>`标签上的`type="text/css"`。
- `useShortDoctype: true`表示使用较短的 html 格式声明。

一般不会再需要手动配置，如果有这方面需求，可以翻看文档并做修改。注意：写入的自己的配置不会与默认配置组合使用（默认配置会被覆盖），所以必须确保写入的自己的配置是完整的。

### 使用 postcss 处理 css

`postcss`是一个处理 css 的工具。因为`postcss`能通过不同的插件实现各类对 css 的操作（包括补齐特性、压缩等），不少人称它为 css 界的`babel`。

还是要先安装相关的依赖。

```sh
npm i postcss@7 postcss-loader@3 autoprefixer@9 postcss-preset-env@6 cssnano@4 -DE
```

要怎么使用`postcss`呢？很简单，在`webpack`配置文件中加入`postcss-loader`，之后创建一个`${PROJECT_DIR}/postcss.config.js`文件供`postcss`读取使用即可。

注意：由于`postcss-loader`会处理`@import`语句，所以还需要修改`css-loader`的`importLoaders`配置。

`${PROJECT_DIR}/config/webpack.dev.js`：

```js
const { merge } = require('webpack-merge');
const baseConfig = require('./webpack.base.js');

module.exports = merge(baseConfig, {
  ...,
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          { loader: 'style-loader' },
          {
            loader: 'css-loader',
            options: {
              importLoaders: 1,
            },
          },
          { loader: 'postcss-loader' },
        ],
      },
      {
        test: /\.s[ac]ss$/,
        use: [
          { loader: 'style-loader' },
          {
            loader: 'css-loader',
            options: {
              importLoaders: 2,
            },
          },
          { loader: 'postcss-loader' },
          { loader: 'resolve-url-loader' },
          { loader: 'sass-loader' },
        ],
      },
    ],
  },
  ...,
});

```

`${PROJECT_DIR}/config/webpack.prod.js`：

```js
const { merge } = require('webpack-merge');
const baseConfig = require('./webpack.base.js');

module.exports = merge(baseConfig, {
  ...,
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          {
            loader: MiniCssExtractPlugin.loader,
            options: {
              publicPath: 'css',
            },
          },
          {
            loader: 'css-loader',
            options: {
              importLoaders: 1,
            },
          },
          { loader: 'postcss-loader' },
        ],
      },
      {
        test: /\.s[ac]ss$/,
        use: [
          {
            loader: MiniCssExtractPlugin.loader,
            options: {
              publicPath: 'css',
            },
          },
          {
            loader: 'css-loader',
            options: {
              importLoaders: 2,
            },
          },
          { loader: 'postcss-loader' },
          { loader: 'resolve-url-loader' },
          { loader: 'sass-loader' },
        ],
      },
    ],
  },
  ...,
});

```

`${PROJECT_DIR}/postcss.config.js`：

```js
module.exports = {};

```

空文件不会让`postcss`起任何作用。要让`postcss`对 css 作特定的处理，就需要使用插件。

浏览器厂商们有时会给实验性的或者非标准的 css 属性添加前缀，这样就可以让开发者进行试验，同时也不会使得标准化之后现有代码被破坏。

由于存在浏览器厂商自实现某些实验性的属性、停止更新浏览器导致没有浏览器跟随标准等情况，所以为 css 属性添加特定浏览器的前缀也带有了 polyfill 的意味。

手动添加前缀是相当麻烦的一件事情，使用`autoprefixer`插件可以让`postcss`自动为我们补全浏览器的样式前缀。

`${PROJECT_DIR}/postcss.config.js`：

```js
module.exports = {
  plugins: [
    require('autoprefixer'),
  ],
};

```

无需额外的配置，就是这么简单。`autoprefixer`会自动寻找目标浏览器的说明（就是`${PROJECT_DIR}/.browserslistrc`，希望你还没有忘记），并且根据目标浏览器自动地添加前缀。

而要处理某些 css 的新语法和新特性，我们就需要用到另外一个插件`postcss-preset-env`。和`@babel/preset-env`类似，它可以为我们处理 css 的某些新语法和新特性，而且，它还内置了`autoprefixer`！

我们可以把`autoprefixer`换成`postcss-preset-env`，同样的，无需额外的配置。`${PROJECT_DIR}/.browserslistrc`也会被自动地读取并使用，此时，`postcss`会根据目标浏览器自动添加属性前缀、处理相对稳定的新语法和新特性。

```js
module.exports = {
  plugins: [
    require('postcss-preset-env'),
  ],
};

```

默认地，`postcss-preset-env`会自动处理 stage 2+ 的新语法和新特性，你可以在它的[官方网站](https://preset-env.cssdb.org/features)中查阅。

最后，`cssnano`插件可以帮助我们压缩`.css`文件并且去除掉多余的注释，用法也同样很简单。但要注意：只有在生产环境下才需要压缩并去除注释，所以我们在生产环境时再引入`cssnano`。

这里我们参考官方文档的配置，使用`cssnano-preset-default`并配置移除所有注释。

```js
module.exports = {
  plugins: [
    require('postcss-preset-env'),
    process.env.NODE_ENV === 'production' &&
      require('cssnano')({
        preset: ['default', { discardComments: { removeAll: true } }],
      }),
  ],
};

```

如果你在加入`cssnano`前查看过生产环境下构建出来的`.css`文件，你会发现文件已经被压缩过了，只是注释没有被去掉。这是因为目前我们所有的`.css`文件都是由`.scss`编译而来，而在生产环境下，`sass`依赖会给`sass-loader`指定`outputStyle: 'compressed'`，这就导致了我们得到的`.css`文件已经被压缩过一次，但注释仍然还在。

加入`cssnano`插件仍然是有必要的，因为`.css`文件依旧没有被压缩，而`.css`和`.scss`文件的注释也没有被去掉，`cssnano`可以很好地完成这部分工作。

到此为止，`postcss`已经能自动处理我们的 css 代码中用到的新语法和新特性，会自动添加属性前缀，能压缩并移除注释了。

### 基础依赖的处理

项目内往往有一些比较基础的依赖，比如`vue`，`react`，`react-dom`等。默认地，`webpack`会把这些基础依赖放入`entry`对应的输出文件中，进而影响最终构建的大小。又因为这些基础依赖往往比较稳定，不会经常更新，打包进`entry`对应的输出文件中会出现业务代码变化、基础依赖没有变化、但用户需要重新拉取包含了基础依赖的代码的情况。

我们可以使用公共 cdn 来加载这些依赖，解决上面提到的两个问题。首先要在`${PROJECT_DIR}/config/webpack.prod.js`配置`externals`，向`webpack`说明无需添加到构建包中的依赖。

```js
module.exports = {
  ...,
  externals: {
    react: 'React',
    'react-dom': 'ReactDOM'
  },
  ...,
};

```

`externals`是一个对象 object，它的键就是`webpack`需要排除的依赖名，稍后我们再说明对应的值。上面给出的配置会让`webpack`构建时排除`react`和`react-dom`两个依赖，不把它们加入到最终构建包中。

之后，还需要手动加入`react`和`react-dom`的公共 cdn 链接，使得项目能够使用到`react`和`react-dom`这两个依赖，否则构建之后将无法运行。

这里使用了 [jsdelivr](https://www.jsdelivr.com/) 这个公共 cdn，你也可以使用 [unpkg](https://unpkg.com/) 这个比较常用的公共 cdn。

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
    <link href="favicon.ico" type="image/x-icon" />
    <title>demo03</title>
  </head>
  <body>
    <div id="root" />
    <script src="https://cdn.jsdelivr.net/npm/react@16.13.1/umd/react.production.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/react-dom@16.13.1/umd/react-dom.production.min.js"></script>
  </body>
</html>

```

注意：公共 cdn 链接指定的依赖版本，要与项目内使用的依赖版本一致，否则可能导致开发和生产环境的表现不一致。`react`的 cdn 链接中暴露了`React`这个变量，而`react-dom`的 cdn 链接中暴露了`ReactDOM`这个变量，对应地，我们要把这两个变量指定为对应键的值。

我们尝试构建一下。最终构建的文件中，没有`react`和`react-dom`的存在。运行测试正常。我们也可以借助`html-webpack-externals-plugin`来实现类似的功能，这里不再做相应的展开演示。

但是更多时候，比起相信公共 cdn，相信自己更好。不使用公共 cdn，我们可以自行把这些基础依赖抽离出来统一放置。这么做要比使用公共 cdn 更好，无需手动指定、更新公共 cdn 的依赖版本，也无需考虑公共 cdn 的稳定性。

`webpack`已经内置了这部分优化，但还需要进一步配置。我们要做的就是配置`optimization.splitChunks`，让这部分内置的优化更紧贴我们的项目。

```js
// ${PROJECT_DIR}/config/webpack.prod.js
module.exports = merge(baseConfig, {
  mode: 'production',
  ...,
  optimization: {
    ...,
    splitChunks: {
      chunks: 'all',
    },
    ...,
  },
  ...,
});

```

- 使用`optimization.splitChunks`，表示我们想要手动配置地分离`chunk`。
- 指定`chunks: 'all'`，表示我们想要把所有引入的库从已有的业务代码中分离出来。

具体需要怎么分离呢？一个常见的配置是，项目内的组件库单独成一个`chunk`，然后`node_modules`文件夹内同步引入的其他依赖单独成一个`chunk`，最后是项目内封装的自定义组件（也就是页面公共组件）单独成一个`chunk`。异步引入的其他依赖，`webpack`默认另行打包出一个`bundle`。

我们通过`optimization.cacheGroups`来配置。首先是项目内的组件库`zent`单独成一个`chunk`。

```js
// ${PROJECT_DIR}/config/webpack.prod.js
module.exports = merge(baseConfig, {
  mode: 'production',
  ...,
  optimization: {
    ...,
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        zent: {
          name: 'chunk-zent',
          priority: 30,
          test: /[\\/]node_modules[\\/]_?zent(.*)/,
        },
      },
    },
    ...,
  },
  ...,
});

```

接着是`node_modules`文件夹内同步引入的其他依赖。

```js
// ${PROJECT_DIR}/config/webpack.prod.js
module.exports = merge(baseConfig, {
  mode: 'production',
  ...,
  optimization: {
    ...,
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        ...,
        vendors: {
          name: 'chunk-vendors',
          test: /[\\/]node_modules[\\/]/,
          priority: 20,
          chunks: 'initial',
        },
      },
    },
    ...,
  },
  ...,
});

```

- `vendors`的`priority`设置得比`zent`的`priority`低，因此，`zent`会优先生成一个`chunk`，而`vendors`对应的`chunk`不会再包含`zent`。
- 设置`vendors.chunks`为`initial`，意味着`chunk-vendors`只会包含代码中同步引入的部分，异步引入的部分会加入到`${PROJECT_DIR}/dist/app.[chunkhash:8].js`中。

最后则是页面公共组件。

```js
// ${PROJECT_DIR}/config/webpack.prod.js
module.exports = merge(baseConfig, {
  mode: 'production',
  ...,
  optimization: {
    ...,
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        ...,
        components: {
          name: 'chunk-components',
          test: path.resolve('src', 'components'),
          minChunks: 2,
          priority: 10,
          reuseExistingChunk: true,
        },
      },
    },
    ...,
  },
  ...,
});

```

- `minChunks`表示最小引用次数，这里设置为 2，即`chunk-components`内的代码都被其他`.js`中的代码引用过 2 次或以上。
- `reuseExistingChunk: true`表示如果`chunk-components`包含了已经被分离出来的部分（某些代码已经被分进了自定义`chunk`中），这些部分会被复用而不再打包进`chunk-components`中。

我们可以构建一下，看看效果。查看构建文件可以发现，`html-webpack-plugin`已经自动引入了各个`chunk`，无需我们操心。

下面是我在构建后的截图。

![有 splitChunks 构建效果图](https://ae01.alicdn.com/kf/U9cbdde6f3a1f4b728dcb3f9902ac9300C.jpg)

- 列`Asset`给出了打包出来的各个文件的位置和名称。
- 列`Size`给出了打包出来的各个文件的大小。
- 列`Chunks`和`Chunk Names`是我们所要关注的重点。
  - `Chunk Names`一共三个：`app`，`chunk-zent`和`chunk-vendors`。
  - `app`就是我们设置的`entry`键值，也就是说，`entry`本身就会生成一个`chunk`。
  - `chunk-zent`和`chunk-vendors`是我们配置后从`chunk app`中分离出来的`chunk`，位置和命名会跟随`output.filename`（也就是`${PROJECT_DIR}/dist/[name]:[chunkhash:8].js`）。
  - `react`和`react-dom`已经被加入到`chunk-vendors`中。
  - 我们如果修改`${PROJECT_DIR}/src/index.js`，就会发现只有`app`对应的文件指纹发生了变化，而`chunk-zent`和`chunk-vendors`的文件指纹没有发生变化。这样就使得这两个部分能有效地缓存，减少请求时间。
  - 没有`chunk-components`，这是因为我们目前没有使用任何的页面公共组件。

没有`splitChunks`的构建后的截图如下所示。

![没有 splitChunks 构建效果图](https://ae01.alicdn.com/kf/Uabea4a17c0224557bf1213cd32339ea01.jpg)

可以看到，如果不使用`splitChunks`，几乎所有的代码都会挤到`app.[chunkhash:8].js`中，在比较大的项目中，文件就会变得非常大。如果不更新基础库，用户就要耗费大量时间在获取包含了基础库代码的文件上。而使用了`splitChunks`，在不更新基础库的前提下，用户只需要获取包含了最新业务代码的相关文件（也就是`app`相关的文件），缩短了获取的时间。

但使用了`splitChunks`后，另一个问题也暴露出来了，那就是所有`.js`文件都放到了`${PROJECT_DIR}/dist`目录下，这并不利于管理。

和前面对字体、图片、`.css`文件的配置类似，我们可以让`.js`文件都放入特定的文件夹中。我们修改`output.filename`，使得所有的`.js`文件都会放入`${PROJECT_DIR}/dist/js`文件夹中。

```js
module.exports = merge(baseConfig, {
  ...,
  output: {
    path: path.resolve('dist'),
    filename: 'js/[name].[chunkhash:8].js',
  },
  ...,
});

```

### 使用 gzip 压缩资产文件

### 代码分割和动态引入

### 使用 eslint 检验 js 代码

### 使用 stylelint 检验 scss 代码

### 优化日志

### 构建分析

### 减少 webpack 信息输出

你可能会注意到，运行`npm run build`输出的信息，要比`npm run dev`输出的信息多得多。这是因为我们控制了`webpack-dev-server`输出的信息，类似地我们也可以控制`webpack`输出的信息。

要控制`webpack`输出的信息很简单，只需要在`${PROJECT_DIR}/config/webpack.prod.js`中设置`stats`字段。

```js
module.exports = {
  ...,
  stats: 'minimal',
  ...,
};

```

`stats`用于控制显示哪些信息，默认为`normal`。我们修改成`minimal`，就可以达到和`webpack-dev-server`的配置一样的效果。

🎉恭喜，你的第三个 webpack demo 已经完成啦～

参考源代码见 [modyqyw/webpack-demos/demo03](https://github.com/ModyQyW/webpack4-demos/tree/master/demo03)。

待补充，催稿可以

（1）邮件催稿

（2）打赏，备注“催稿+内容”（通常这种方式会更有效点，毕竟收了钱不好意思再拖）

## demo04 - 编写可维护的配置

### 配置设计

### 使用 eslint 检验配置

### 使用 prettier 格式化配置

### 冒烟测试

### 单元测试

### 持续集成

🎉恭喜，你的第四个 webpack demo 已经完成啦～

参考源代码见 [modyqyw/webpack-demos/demo04](https://github.com/ModyQyW/webpack4-demos/tree/master/demo04)。

## demo05 - 背后的原理

待补充，催稿可以

（1）邮件催稿

（2）打赏，备注“催稿+内容”（通常这种方式会更有效点，毕竟收了钱不好意思再拖）

## 参考

- [nodejs 12.x - path](https://nodejs.org/dist/latest-v12.x/docs/api/path.html)
- [webpack](https://v4.webpack.js.org)
- [理解 webpack chunk](https://juejin.im/post/5d2b300de51d45775b419c76)
- [clean-webpack-plugin](https://github.com/johnagan/clean-webpack-plugin#readme)
- [copy-webpack-plugin](https://github.com/webpack-contrib/copy-webpack-plugin#readme)
- [friendly-errors-webpack-plugin](https://github.com/geowarin/friendly-errors-webpack-plugin#readme)
- [html-webpack-plugin](https://github.com/jantimon/html-webpack-plugin#readme)
- [webpack-bar](https://github.com/nuxt/webpackbar)
- [webpack plugins 的顺序会影响什么吗？](https://stackoverflow.com/questions/41470771/webpack-does-the-order-of-plugins-matter)
- [常用 plugins 汇总](https://modyqyw.top/front-end/lib-toolkit-framework-and-more/#%E7%BC%96%E8%AF%91%E6%89%93%E5%8C%85)
- [常用 loaders 汇总](https://modyqyw.top/front-end/lib-toolkit-framework-and-more/#%E7%BC%96%E8%AF%91%E6%89%93%E5%8C%85)
- [babel](https://babeljs.io/)
- [babel-loader](https://github.com/babel/babel-loader#readme)
- [babel 教程](https://www.jiangruitao.com/docs/babel/)
- [browserslist](https://github.com/browserslist/browserslist#readme)
- [@babel/preset-env](https://babeljs.io/docs/en/babel-preset-env)
- [@babel/preset-react](https://babeljs.io/docs/en/babel-preset-react)
- [@babel/plugin-transform-runtime](https://babeljs.io/docs/en/babel-plugin-transform-runtime)
- [@babel/plugin-transform-runtime 到底是什么](https://zhuanlan.zhihu.com/p/147083132)
- [@babel/polyfill](https://babeljs.io/docs/en/babel-polyfill)
- [@vue/babel-preset-app](https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/babel-preset-app)
- [core-js](https://github.com/zloirock/core-js#readme)
- [regenerator-runtime](https://github.com/facebook/regenerator/tree/master/packages/regenerator-runtime)
- [style-loader](https://github.com/webpack-contrib/style-loader#readme)
- [css-loader](https://github.com/webpack-contrib/css-loader#readme)
- [less](http://lesscss.org/)
- [less-loader](https://github.com/webpack-contrib/less-loader#readme)
- [sass](https://sass-lang.com/)
- [sass-loader](https://github.com/webpack-contrib/sass-loader#readme)
- [resolve-url-loader](https://github.com/bholloway/resolve-url-loader#readme)
- [stylus](https://stylus-lang.com/)
- [stylus-loader](https://github.com/shama/stylus-loader#readme)
- [zent](https://youzan.github.io/zent/zh/)
- [file-loader](https://github.com/webpack-contrib/file-loader#readme)
- [url-loader](https://github.com/webpack-contrib/url-loader#readme)
- [html-loader](https://github.com/webpack-contrib/html-loader#readme)
- [cross-env](https://github.com/kentcdodds/cross-env)
- [webpack-merge](https://github.com/survivejs/webpack-merge#readme)
- [react-hot-loader](https://github.com/gaearon/react-hot-loader#readme)
- [webpack 文件指纹策略](https://jkfhto.github.io/2019-10-18/webpack/webpack-%E6%96%87%E4%BB%B6%E6%8C%87%E7%BA%B9%E7%AD%96%E7%95%A5%EF%BC%9Achunkhash%E3%80%81contenthash%E5%92%8Chash/)
- [mini-css-extract-plugin](https://github.com/webpack-contrib/mini-css-extract-plugin#readme)
- [terser-webpack-plugin](https://github.com/webpack-contrib/terser-webpack-plugin/#readme)
- [postcss](https://postcss.org/)
- [CSS 3 中 -webkit-, -moz-, -o-, -ms- 这些私有前缀什么时候可以移除？](https://www.zhihu.com/question/20597072)
- [autoprefixer](https://github.com/postcss/autoprefixer#readme)
- [postcss-preset-env](https://github.com/csstools/postcss-preset-env#readme)
- [cssnano](https://cssnano.co/)
- [阮一峰 - JavaScript Source Map 详解](http://www.ruanyifeng.com/blog/2013/01/javascript_source_map.html)
- [潘嘉晨 - 手摸手，带你用合理的姿势使用webpack4（上）](https://juejin.im/post/5b56909a518825195f499806)
- [潘嘉晨 - 手摸手，带你用合理的姿势使用webpack4（上）](https://juejin.im/post/5b5d6d6f6fb9a04fea58aabc)
- [stylelint](https://stylelint.io/)

## 致谢

- [XuXianTao](https://github.com/XuXianTao) - 阅读了初稿并提供了弥足珍贵的建议和意见

<Vssue />
