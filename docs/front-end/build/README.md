# 构建

待补充，催稿可以

（1）邮件催稿

（2）打赏，备注“催稿+内容”（通常这种方式会更有效点，毕竟收了钱不好意思再拖）

## 说明

本篇以教程形式书写，适合想要深入 js 工具链的同学学习以入门。

具体的速查可以参考教程内各汇总、参考部分。

## 为什么需要构建工具

- 允许书写 ES6+ 代码、vue template、jsx 等
- 无需考虑 css 前缀，自动添加
- 压缩混淆代码
- 压缩图片、文件等大小

总而言之，构建工具减少了重复的工作，使我们能投入更多的时间到开发工作中。

## webpack 4

webpack 4 是一个静态模块打包工具，分析依赖生成依赖图，最终根据配置生成一个或多个 bundle。

### 为什么选择 webpack

- 社区活跃度高
- 官方生态完整，社区生态丰富
- 配置灵活
- 更新迭代速度快

### 核心概念

#### 入口 entry

entry 指定 webpack 工作时从哪个文件开始分析依赖，默认值为`${PROJECT_DIR}/src/index.js`。本篇教程使用`${PROJECT_DIR}`表示项目根目录，一般认为`package.json`所处目录即为项目根目录。

注意：windows 系统的默认路径分隔符是`\`，而 unix 系统的默认路径分割符是`/`。在旧版本 windows 上建议不要直接使用`/`，而是使用`path.resolve`或`path.join`，否则可能会造成路径识别的问题。以上两种方法会在之后进行说明。

```js
// 使用 path 模块来指定路径
const path = require('path');

module.exports = {
  // 指定入口为 ${PROJECT_DIR}/src/app.js
  entry: path.resolve('src', 'app.js'),
};

```

#### 输出 output

output 指定 webpack 在哪里存放、命名创建的 bundle，主要输出文件的默认值为`${PROJECT_DIR}/dist/main.js`，其他生成文件默认放在`${PROJECT_DIR}/dist`。

```js
// 使用 path 模块来指定路径
const path = require('path');

module.exports = {
  // 指定入口为 ${PROJECT_DIR}/src/app.js
  entry: path.resolve('src', 'app.js'),
  // 指定输出为 ${PROJECT_DIR}/dist/bundle.js
  output: {
    path: path.resolve('dist'),
    filename: 'bundle.js',
  },
};

```

#### 加载器 loader

webpack 本身只能解析 js 和 json 文件，loader 增强了 webpack 的解析能力，使得 webpack 能够解析 jsx，ts，tsx，png，jpg，mp3，mp4，flv，webp 等诸多格式的文件，并将它们转换为有效模块、添加到依赖图中并供应用程序使用。

loader 有两个必需的属性，一个是`test`，用于判断需要解析的文件，另一个是`use`，指定用于解析的 loader。

```js
// 使用 path 模块来指定路径
const path = require('path');

module.exports = {
  // 指定入口为 ${PROJECT_DIR}/src/app.js
  entry: path.resolve('src', 'app.js'),
  // 指定输出为 ${PROJECT_DIR}/dist/bundle.js
  output: {
    path: path.resolve('dist'),
    filename: 'bundle.js',
  },
  module: {
    rules: [
      // 指定：在 require/import 语句中涉及到 .png 的路径都先用 url-loader 转换
      { test: /\.png$/, use: 'url-loader' },
    ],
  },
};

```

有的 loader 可能还会有额外的属性供你配置。

#### 插件 plugin

plugin 用于执行范围更广的任务，比如打包优化，资源管理，注入环境变量等。

```js
// 使用 path 模块来指定路径
const path = require('path');
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = {
  // 指定入口为 ${PROJECT_DIR}/src/app.js
  entry: path.resolve('src', 'app.js'),
  // 指定输出为 ${PROJECT_DIR}/dist/bundle.js
  output: {
    path: path.resolve('dist'),
    filename: 'bundle.js',
  },
  module: {
    rules: [
      // 指定：在 require/import 语句中涉及到 .png 的路径都先用 url-loader 转换一下
      { text: /\.png$/, use: 'url-loader' },
    ],
  },
  plugins: [
    // 指定：使用 webpack-bundle-analyzer 分析生成内容的大小及各依赖占比
    new BundleAnalyzerPlugin(),
  ],
};

```

#### 模式 mode

指定不同的模式，webpack 会有不同的表现，默认值为`production`。

```js
// 使用 path 模块来指定路径
const path = require('path');
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = {
  // 指定模式为 development，即开发模式
  mode: 'development',
  // 指定入口为 ${PROJECT_DIR}/src/app.js
  entry: path.resolve('src', 'app.js'),
  // 指定输出为 ${PROJECT_DIR}/dist/bundle.js
  output: {
    path: path.resolve('dist'),
    filename: 'bundle.js',
  },
  module: {
    rules: [
      // 指定：在 require/import 语句中涉及到 .png 的路径都先用 url-loader 转换
      { text: /\.png$/, use: 'url-loader' },
    ],
  },
  plugins: [
    // 指定：使用 webpack-bundle-analyzer 分析生成内容的大小及各依赖占比
    new BundleAnalyzerPlugin(),
  ],
};

```

### demo01 - 一个简单的 demo

前面简单地讲述了 webpack 4 的 5 个核心概念，下面开始实战。之后，示例的代码均基于 macOS，使用 linux 和 windows 的朋友请自行作相关调整，不再重复提醒。

windows 系统建议使用 git bash 或者 windows terminal，linux 系统和 macOS 建议使用 zsh + oh-my-zsh。

首先安装 nvm。nvm 是一个用于管理 node 版本的工具，免去了升级 node 版本的繁琐工作。

- [nvm for linux/macOS](https://github.com/nvm-sh/nvm)
- [nvm for windows](https://github.com/coreybutler/nvm-windows)

安装 nvm 之后，使用 nvm 来安装 node lts 版本（即长期支持版）。

```sh
nvm install --lts
```

新建一个文件夹，命名为 demo。进入到该文件夹中，初始化一个`package.json`文件。

```sh
mkdir demo && cd demo
npm init -y
```

根目录下新建一个`.npmrc`文件，用于设置 npm 镜像，加速安装依赖。

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

然后安装 webpack 相关的依赖。

```sh
npm i webpack@4 webpack-cli@3 -DE
```

创建一个内容简单的`index.js`。

```js
// ${PROJECT_DIR}/src/index.js
document.write('Hello webpack!');

```

创建一个 webpack 配置文件`webpack.config.js`。`path`模块的`resolve`可以帮助我们基于项目根目录生成任一平台的绝对路径（无需考虑路径分隔符的问题），用于定位特定的文件夹或文件。`path`模块还有一个`join`，作用与`resolve`相近，可以自行了解一下。

```js
// 使用 path 模块来指定路径
const path = require('path');

module.exports = {
  // 指定模式为 production，即生产模式
  mode: 'production',
  // 指定入口为 ${PROJECT_DIR}/src/index.js
  entry: path.resolve('src', 'index.js'),
  // 指定输出为 ${PROJECT_DIR}/dist/bundle.js
  output: {
    path: path.resolve('dist'),
    filename: 'bundle.js',
  },
};

```

目前，我们已经看到了 2 种模式：`development`和`production`。实际上，一共内置了 3 种模式，还有一种模式就是`none`。`development`和`production`模式都会启用一些内置的优化，而`none`模式没有任何优化，一般也不会使用该模式。假如不指定模式，会默认使用`production`模式。

现在，我们修改`package.json`中的`scripts`字段。

```json
{
  ...,
  "scripts": {
    "build": "webpack"
  },
  ...
}
```

接着，就可以执行命令进行构建了。执行`npm run build`之后，npm 会在`package.json`文件中寻找`scripts`中的`build`字段，找到的话会执行操作，npm 会自动在项目根目录下的`node_modules`文件夹中寻找 webpack 依赖并调用，webpack 会默认使用项目根目录下的`webpack.config.js`文件进行构建（也就是俗称的打包）。寻找的顺序是：项目根目录 -> 全局目录 -> 都没有找到，则报错终止执行。

```sh
npm run build
```

最后，我们可以在`dist`目录下看到已经生成了一个`bundle.js`文件。

但是现在还远远不够，每次都手动创建一个 html 文件并且手动引用这个`bundle.js`是难以接受的，因为这会耗费不必要的时间，而且当项目复杂度越来越高时，这个引用的成本也会越来越高。我们需要一些自动处理的手段，来帮我们自动引入这个`bundle.js`文件到 html 文件中。

我们先来安装一下相关的依赖。

```sh
npm i copy-webpack-plugin@6 html-webpack-plugin@4 clean-webpack-plugin@3 webpackbar@4 -DE
```

在项目根目录下新建一个`public`文件夹，放入`favicon.ico`（可以自己随便找一个，或者把已有的图片转成 ico 格式）和`index.html`。`index.html`如下所示。

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

紧接着，我们在`webpack.config.js`中做相关的配置。这样，我们就可以无需操心在 html 文件中引入 js 文件的问题了，而且网站图标也可以帮我们自动处理。

```js
const CopyPlugin = require('copy-webpack-plugin');
const HtmlPlugin = require('html-webpack-plugin');

module.exports = {
  ...
  plugins: [
    // 复制 ${PROJECT_DIR}/public/favicon.ico 到 ${PROJECT_DIR}/dist 目录下
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

但现在还不够，我们还需要在每次构建之前，把上一次构建的文件给删除掉，也就是删除掉`dist`文件夹，以避免可能发生的冲突。我们还需要加入一些额外的配置。

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

我们还可以加一点额外的处理，让它在打包的时候显示进度条。

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

完整的`webpack.config.js`如下所示。

```js
// 使用 path 模块来指定路径
const path = require('path');
// 使用 plugins
const { CleanWebpackPlugin: CleanPlugin } = require("clean-webpack-plugin');
const CopyPlugin = require('copy-webpack-plugin');
const HtmlPlugin = require('html-webpack-plugin');
const WebpackBar = require('webpackbar');

module.exports = {
  // 指定模式为 production，即生产模式
  mode: 'production',
  // 指定入口为 ${PROJECT_DIR}/src/index.js
  entry: path.resolve('src', 'index.js'),
  // 指定输出为 ${PROJECT_DIR}/dist/bundle.js
  output: {
    path: path.resolve('dist'),
    filename: 'bundle.js',
  },
  // 指定插件
  plugins: [
    // 显示进度条
    new WebpackBar(),
    // 移除上一次的构建文件
    new CleanPlugin(),
    // 复制 ${PROJECT_DIR}/public/favicon.ico 到 ${PROJECT_DIR}/dist 目录下
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

重新开始构建，之后可以看到进度条和简短的提示信息。最终生成的`dist`目录的结构如下。直接打开`index.html`，可以看到`Hello webpack!`。🎉恭喜，一个简单的 webpack demo 已经完成啦～

```sh
dist
├── bundle.js
├── favicon.ico
└── index.html
```

相关资料汇总：

- [webpack - mode](https://v4.webpack.js.org/configuration/mode/)
- [clean-webpack-plugin](https://github.com/johnagan/clean-webpack-plugin#readme)
- [copy-webpack-plugin](https://github.com/webpack-contrib/copy-webpack-plugin#readme)
- [html-webpack-plugin](https://github.com/jantimon/html-webpack-plugin#readme)
- [webpack-bar](https://github.com/nuxt/webpackbar)
- [webpack plugins 的顺序会影响什么吗？](https://stackoverflow.com/questions/41470771/webpack-does-the-order-of-plugins-matter)

源代码见 [modyqyw/webpack-demos/demo01](https://github.com/ModyQyW/webpack4-demos/tree/master/demo01)。

### demo02 - 再看 webpack 核心概念与基础应用

#### 入口 entry

一开始我们说到，webpack 会分析依赖生成依赖图，而分析依赖生成依赖图的起点就由`entry`指定。

![webpack示意图](https://ae01.alicdn.com/kf/Hc2861d3d0e2b4ad89bfab9c37be5ecbcK.jpg)

单页面应用 SPA 只有单入口，此时`entry`是一个字符串 string。多页面应用 MPA 有多个入口，此时`entry`是一个对象 object。目前，单页面应用比较多见，demo01 就是一个 SPA 的配置。

实际上，demo01 中`entry`的写法等同于下面的代码。也就是说，单入口也可以用多入口的写法书写，默认的 key 是`main`，你也可以自行修改。

```js
const path = require('path');

module.exports = {
  ...,
  entry: {
    main: path.resolve('src', 'index.js'),
  },
  ...
};

```

还有一个可能会跟`entry`一起使用的字段，那就是`context`。`context`可以指定`entry`的基本路径，接收的值是一个绝对路径，对于 MPA，使用`context`往往能使`entry`更简洁，下面就是一个示例。因为需要用到路径分割符，可能会造成问题，我个人并不建议这么写。

```js
const path = require('path');

module.exports = {
  ...,
  context: path.resolve('src'),
  entry: './index.js',
  ...
};

```

为了之后的方便，我们先把没有使用到`context`的`webpack.config.js`放入到`${PROJECT_DIR}/config`文件夹中，后续我们也不会使用`context`字段。无需修改代码，`path.resolve`也能保证其运作成功，但我们需要把对应的`script`修改一下来确保能够运行。

```json
{
  ...,
  "scripts": {
    "build": "webpack --config ./config/webpack.config.js"
  },
  ...
}

```

注意这里使用了`/`作为路径分隔符，可能会造成问题，我们之后会再加以改进。

相关资料汇总：

- [webpack - entry and context](https://v4.webpack.js.org/configuration/entry-context/)

#### 输出 output

分析依赖生成依赖图之后，webpack 就会开始打包，打包结果如何输出就由`output`指定。

SPA 往往只需要简单地指定`filename`和`path`，如上面示例所示。而如果要构建 MPA，除了修改`entry`，还需要利用占位符确保文件名唯一，下面代码中，`[name]`就是一个占位符，表示使用`entry`中配置的 key 来命名打包后的文件。

```js
const path = require('path');

module.exports = {
  entry: {
    main: path.resolve('src', 'main.js'),
    app: path.resolve('src', 'app.js'),
  },
  output: {
    filename: '[name].js',
    path: path.resolve('dist'),
  },
};

```

此外，还需要修改 html-webpack-plugin 的配置，让每一个入口点都有一个专属的 html 文件模板，并且还需要让每一个 html 文件模板都插入公共代码块。

MPA 在配置上相当复杂，也相当劝退新手，在这里不再展开阐述，而是以 SPA 作为示例作 webpack 打包教学。有兴趣的可以自行搜索`webpack mpa`学习。

相关资料汇总：

- [webpack - output](https://v4.webpack.js.org/configuration/output/)

#### 插件 plugin

plugins 用于增强 webpack 功能，比如优化打包文件，管理资源，注入环境变量等等，作用于整个构建过程。

前面的例子中，我们使用到了`copy-webpack-plugin`，`html-webpack-plugin`和`webpackbar`，都是相对来说比较常用的 plugins。

每个 plugin 都需要放入到`plugins`字段数组中，顺序一般不影响，具体 plugin 的配置需要去查询具体的文档。

相关资料汇总：

- [webpack - plugins](https://v4.webpack.js.org/configuration/plugins/)
- [webpack - plugins list](https://v4.webpack.js.org/plugins/)
- [常用 plugins 汇总](https://modyqyw.top/front-end/lib-toolkit-framework-and-more/#%E7%BC%96%E8%AF%91%E6%89%93%E5%8C%85)

#### 模式 mode

指定不同的模式，webpack 会自动启用不同的功能进行优化，默认值为`production`，默认取值范围为`development`，`production`和`none`。具体可以参考官方文档。

后面还会展示如何自定义`mode`。

相关资料汇总：

- [webpack - mode](https://v4.webpack.js.org/configuration/mode/)

#### 加载器 loader

由于 webpack 默认只支持解析 js 和 json 文件，所以项目中使用到的其他文件，比如图片文件，字体文件，样式文件等，就只能使用 loaders 解析，解析后文件会作为模块被 webpack 加入到依赖图中。

demo02 将会关注一些常用 loader。

相关资料汇总：

- [webpack - module](https://webpack.js.org/configuration/module/)
- [webpack - loaders list](https://v4.webpack.js.org/loaders/)
- [常用 loaders 汇总](https://modyqyw.top/front-end/lib-toolkit-framework-and-more/#%E7%BC%96%E8%AF%91%E6%89%93%E5%8C%85)

#### 新语法相关的 loader

因为 webpack 本身并不支持 es6+ 语法打包，所以我们需要使用 [babel](https://babeljs.io/) 和 [babel-loader](https://github.com/babel/babel-loader#readme)，让 webpack 能够解析 es6+ 语法。

babel 其中一个很好的作用就是转换新语法为旧语法，也就是我们常说的转译。babel-loader 使得 webpack 能够结合 babel 来使用。

首先还是要安装相关的依赖。

```sh
npm i @babel/runtime@7 core-js@3 regenerator-runtime@0.11.1 react@16.13.1 react-dom@16.13.1 -E
npm i @babel/cli@7 @babel/core@7 @babel/plugin-transform-runtime@7 @babel/preset-env@7 @babel/preset-react@7 babel-loader@8 -DE
```

其次是修改 webpack 配置。

```js
// ./config/webpack.config.js
module.exports = {
  ...
  // 指定 loader
  module: {
    rules: [
      ...
      {
        // 指定 js 和 jsx 文件
        test: /\.jsx?$/,
        // 不处理 node_modules 和 bower_components
        exclude: /(node_modules|bower_components)/,
        // 使用 babel-loader 进行处理
        use: {
          loader: 'babel-loader',
        },
      },
      ...
    ],
  },
};

```

排除 node_modules 和 bower_components 中的 js 文件能够有效地提高编译效率，同时避免可能存在的二次编译问题。

值得注意的是，loaders 和上面提到的几个核心概念都不同，所使用到的字段是`module.rules`。对于 webpack 而言，所有文件都可以视作一个模块，所以需要在`module`（模块）字段内做相关的定义。

修改完 webpack 配置后，我们还需要配置 babel，让 babel 根据我们的需求进行转译。我们在根目录下创建一个文件`babel.config.json`作为 babel 的配置文件，这也是当前 babel 官方推荐的做法。

要支持 es6+ 语法非常简单，可以直接使用 babel 官方提供的`@babel/preset-env`。它能实现智能转换，而无需提供额外的配置。

```json
{
  "presets": ["@babel/preset-env"]
}

```

但现实往往是残酷的，`@babel/preset-env`并不一定能满足项目需求，我们需要描述我们想要支持的浏览器，也就是转译后的代码能够跑在什么浏览器上。我们可以在项目根目录创建一个文件`.browserslistrc`，babel 会自动读取该文件来使用。实际上，这个文件还会被 autoprefixer，stylelint 等依赖使用，之后会进一步讲解。下面是一个示例。

```sh
> 0.2%
last 5 versions
not dead
chrome >= 70

```

`> 0.2%`表示需要支持使用率超过 0.2% 的浏览器，`last 5 versions`表示需要支持浏览器的最近 5 个版本，`not dead`表示浏览器在最近 24 个月内还得到了官方的支持（所以目前已经不应该再支持 IE 11-，要使用 babel 支持 IE 11-，需要更多的工作，这里并不会做相关的演示），`chrome >= 70`表示 chrome 的版本需要不小于 70（目前，qq 浏览器使用的内核就是 chromium 70）。上面四个条件取并集，就是需要支持的浏览器范围。更多的信息可以查看[官方文档](https://github.com/browserslist/browserslist#readme)。

描述完之后，我们还需要在转译的时候加入这些浏览器不支持，但我们项目中又使用到的特性。

这里还需要讲解 2 个新的概念再接着往下说，它们就是`shim`和`polyfill`。`shim`用于兼容，拦截调用不存在的 api 并提供抽象层，并不局限于 html/css/js，而`polyfill`是`shim`的一种，通常使用 js 提供一些浏览器本身没有的 html/css/js 新功能。我们要做的，实际上就是要自动处理`polyfill`，这也就是 babel 另一个很好的作用。

自动处理`polyfill`也可以通过配置`@babel/preset-env`来做。`@babel/preset-env`默认只有转译的配置（默认把 es6+ 语法转换成 es5 语法），不会进行 polyfill，需要进行手动配置。这里我们指定`useBuiltIns`为`usage`模式，这样比较省事，不用配置太多。

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

默认地，`@babel/preset-env`会使用`core-js`v2 和`regenerator-runtime`做 polyfill。`core-js` v3 版本要更好，这里我们指定要使用 v3 版本。

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

之后，babel 会为我们自动引入代码使用到了的但浏览器不支持的、`core-js`和`regenerator-runtime`关联的部分做 polyfill。

但是还存在一个问题，转译之后可能会使得每个文件头部都增加了相同的代码，比如使用`class`，转译之后就会在使用到`class`的文件头部都增加一串相同的代码，这些代码官方叫做`helpers`。这些重复的代码会影响最终构建包的体积，在实际开发中是难以接受的，我们可以使用`@babel/plugin-transform-runtime`来抽离这些重复的代码到一起，进而压缩最终构建包的体积。

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

因为我们已经在`@babel/preset-env`中配置了`core-js`，所以无需在`@babel/plugin-transform-runtime`中重复配置。

除了 es6+ 的语法，我们还想支持 react。类似地，我们也可以使用 babel 来解析 react 代码，只需要配置`@babel/preset-react`即可。

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
  "plugins": [["@babel/plugin-transform-runtime"]]
}

```

当然，对比起官方文档和实际大型应用开发需求，教程这部分还相当简陋，建议还是多多阅读文档多多实践。

相关资料汇总：

- [babel](https://babeljs.io/)
- [webpack - loaders - babel-loader](https://v4.webpack.js.org/loaders/babel-loader/)
- [babel-loader](https://github.com/babel/babel-loader#readme)
- [babel 教程](https://www.jiangruitao.com/docs/babel/)
- [browserslist](https://github.com/browserslist/browserslist#readme)
- [@babel/preset-env](https://babeljs.io/docs/en/babel-preset-env)
- [@babel/preset-react](https://babeljs.io/docs/en/babel-preset-react)
- [@babel/plugin-transform-runtime](https://babeljs.io/docs/en/babel-plugin-transform-runtime)
- [@babel/polyfill](https://babeljs.io/docs/en/babel-polyfill)
- [core-js](https://github.com/zloirock/core-js#readme)
- [@vue/babel-preset-app](https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/babel-preset-app)

## snowpack

## rollup

## parcel

待补充，催稿可以

（1）邮件催稿

（2）打赏，备注“催稿+内容”（通常这种方式会更有效点，毕竟收了钱不好意思再拖）

<Vssue />
