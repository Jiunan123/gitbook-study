# webpack

## 什么是webpack

1. 基于node.js的构建工具
2. 模块 --构建--> 静态资源(.js, .css, .jpg, .png)
3. 编译是构建的核心：把不能在浏览器端直接运行的模块编译（.sass -> css, 图片压缩，.vue .jsx => .js, .js(es6 -> es5)）

## 安装

- 局部安装（推荐，为了各个项目差异化）

- 全局安装（意义不大）

  ```shell
  npm install webpack webpack-cli
  
  # 在本路径的node_modules下查找可执行的webpack
  npx webpack -v 
  
  # 不使用下面的命令，因为webpack是局部安装的
  # webpack -v 
  
  # 打包（入口文件src/index.js），使用默认配置的打包方式（webpack.config.js）
  npx webpack
  
  # 使用自定义配置打包方式，可以配置到package.json中的scripts下
  npx webpack --config <filePath>
  ```

  package.json

  ```json
  {
    "scripts": {
      "dev": "webpack --config build/webpack.dev.conf.js"
    }
  }
  
  // webpack 也是走的本项目下的node_modules
  ```

  webpack.config.js

  ```js
  // 由于output的输出路径要求绝对路径，使用该模块，根据相对路径组装绝对路径
  const path = require("path");
  const HtmlWebpackPlugin = require("html-webpack-plugin");
  const MiniCss = require("mini-css-extract-plugin");
  const { CleanWebpackPlugin } = require("clean-webpack-plugin");
  
  module.exports = {
      /**
       * 执行打包任务的入口位置
       * entry 支持string, arr, object
       * string 为单入口：用于单页面项目（SPA）
       * object 为多入口：用于多页面项目（多入口一定为多出口）
       */
      // entry: "./src/index.js", // 默认 ./src/index.js，绝对路径 or 相对路径
      entry: {
          main: "./src/index.js",
          login: "./src/login.js",
      },
      // 输出资源文件的信息（存储位置、文件名称）
      output: {
          path: path.resolve(__dirname, "./dist"), // 输出文件的存储位置，要求绝对路径
          /** 输出文件名
           * filename: 支持字符串，支持占位符 [name]
           * [name] 与 entry 的结构一对一
           * 不同chunks使用相同的文件名会明明冲突，报错
           */
          // filename: "main.js",
          filename: "[name].js",
      },
      // 打包模式
      mode: "development", // development, production none
      // 配置loader中模块的查找路径
      resolveLoader: {
          modules: ["node_modules", "./my-loaders"],
      },
      // 配置loader
      module: {
          // 匹配规则
          rules: [
              {
                  test: /\.css$/,
                  use: ["style-loader", "css-loader"],
              },
              {
                  test: /\.less$/, // 以css后缀的文件,
                  /**
                   * 使用哪些模块编译处理
                   * 执行顺序，从右到左
                   * 如下：先编译（css-loader），然后应用(style-loader)
                   */
                  // use: [
                  //     // "style-loader", // style 内联方式引入
                  //     MiniCss.loader, // 通过<link href=""></link>引入独立抽出来的CSS文件，HtmlWebpackPlugin会帮忙引入
                  //     {
                  //         loader: "css-loader", // 序列化css文件
                  //         options: {
                  //             modules: true, // css模块化？
                  //         }
                  //     },
                  //     "postcss-loader",
                  //     "less-loader",
                  // ],
                  use: ["my-style-loader", "my-css-loader", "my-less-loader"], // 自定义loader
              },
              {
                  test: /\.js$/,
                  /**
                   * use: String ｜ Object | Array
                   * String: path | 模块名
                   * Object: { loader, options, ...}
                   * Array: [多个loader]
                   */
                  // use: path.resolve(__dirname, "./my-loaders/my-loader.js"),
                  use: {
                      /**
                       * loader: path or module
                       * 若在resolveLoader配置默认的查找路径，就可以使用自定义模块，否则需要绝对路径
                       */
                      loader: "my-loader",
                      // loader: path.resolve(__dirname, "./my-loaders/my-loader.js"),
                      options: {
                          name: '---------测试模块',
                      }
                  }
              }
          ]
      },
      plugins: [
          /**
           * 作用：自动生成html文件，引入bundle文件，压缩html
           * 一个实例对应一个html, 需要多个则应new多个实例
           */
          new HtmlWebpackPlugin({
              // 精准配置
              template: "./src/index.html",
              filename: "index.html", // 输出的文件名
              chunks: ['main', 'login'], // 设置scripts要包含哪些bundle 
          }),
          /**
           * 把css生成独立文件，并存放在dist中，不使用style内联方式, 在module中还需要配置loader
           */
          new MiniCss({
              filename: "style/index.css", // 抽离出来的文件名称
          }),
          new CleanWebpackPlugin(),
      ],
  }
  ```
  
  ```js
  // index.js
  import "./other";
  import "./main.css"; // 这个文件在css-loader & style-loader的作用下，被嵌入到<head><style>...内容...</style></head>
  
  console.log('index');
  ```
  
  ```css
  // main.css
  body {
      background-color: grey;
  }
  ```
  
  
  
  ### 不同版本
  
  - 3.x ：繁杂配置
  - 4.x ：零配置（噱头），是低配置

## 概念区分

### bundle chunks chunk module

- 构建后产生的资源文件（比如index.js等）为bundle文件
- 一个bundle文件对应一个chunks(chunk组)
- 一个chunks对应至少一个module
- 一个module对应至少一个chunk（在不考虑公共模块的时候，module对应一个chunk）

## plugin

plugin 安装完，在webpack.config.js中引入使用

```sh
# 使用指定模版动态生成html文件（自动引入打包生成的资源包等）
npm install html-webpack-plugin -D
# 默认安装的最新的(目前是5), 与webpack4不兼容，可以指定版本安装
npm install html-webpack-plugin@4 -D
# 自动删除生成目录的文件，dist保持最新
npm install clean-webpack-plugin -D
# 将css资源打包成独立的文件（配合html-webpack-plugin 还能自动使用<link>方式引入该css资源文件）
npm install mini-css-extract-plugin@1.6.2 -D
```

## loader

Webpack 默认只能解析 js、json。对于其它类型的文件需要配置loader

模块功能建议单一，所以现在第三方提供的模块都是单一功能，比如css-loader & style-loader 分别只做单一事情。

```sh
# 默认安装最新，webpack4不兼容最新的。
npm install css-loader@5.2.6 -D
npm install style-loader@2.0.0 -D
# sass,less...xx表示版本
npm install sass sass-loader@xx -D
npm install less less-loader@7.3.0 -D
# postcss
npm install postcss postcss-loader@4.2.0 -D
# postcss的插件，非webpack插件，所以与webpack没有兼容问题
npm install autoprefixer -D
# postcss的插件, 压缩CSS代码（去掉空白符等）
npm install cssnano -D
```

### postcss

[postcss文档](https://github.com/postcss/postcss/blob/main/docs/plugins.md)

- 一个使用js工具和插件，转换CSS代码的工具（集合）
- 作用：
  - 检查CSS，支持CSS变量和mixins,编译尚未被浏览器广泛支持的先进CSS语法，内联图片等功能
  - 支持插件： 两种配置方式, postcss.config.js、.postcssrc.js

`postcss.config.js`

```js
module.exports = {
    plugins: [require("autoprefixer"), require("cssnano")], // 使用CSS厂商自动化配置功能
}
```

`.postcssrc.js`

```js
module.exports = {
  "plugins": {
    "postcss-import": {},
    "postcss-url": {},
    // to edit target browsers: use "browserslist" field in package.json
    "autoprefixer": {},
    "postcss-pxtorem": {
      rootValue: 16, // 160
      propList: ["*"]
    }
  }
}

```

### browserslist

声明一段浏览器合集，用到该集合的工具，会根据该配置，针对性输出兼容性的代码。例如一以下工具：

- Autoprefixer
- Babel preset-env

```json
// package.json
{
  "browserslist": [
    "> 1%", // 全球市场占用率 > 1%的浏览器
    "last 2 versions", // 兼容浏览器的最近两个大版本
    // ""last 1 version", // 单数
    "not ie <= 8"
  ]
}
```

`.browserslistrc`

```
>1%
last 2 versions
```

配置可在package.json中配置，也可在`.browserslistrc`中配置

- 兼容浏览器的大版本例如：ie大版本11，10，9...；chrome版本 96.0.4664.55,则大版本为96。

- 查看浏览器列表：`npx browserslist "last 2 versions, >1%"`：

  - and_xx: 表示安卓系列的浏览器，and_ff 为火狐
  - ios_xx: 表示苹果手机端的浏览器，ios_saf 为safari
  - bb: 黑莓


### 自定义loader

`my-style-loader.js`

```js
/**
 * style-loader:
 * 将序列化后的css插入到DOM的head中，采用<style>内联</style>
 */
module.exports = function (content) {
    return `
        var styleElement = document.createElement('style');
        styleElement.innerHTML = ${content};
        document.head.append(styleElement);
    `;
}
```

`my-css-loader.js`

```js
/**
 * css-loader:
 * 把css序列化
 */
module.exports = function (content) {
    return JSON.stringify(content);
}
```

`my-less-loader.js`

```js
const less = require("less");

/**
 * less-loader
 * 依赖less模块
 * 将less文件编译成css文件
 */
module.exports = function (content) {
    // less模块自带的API
    less.render(content, (error, output) => {
        const cssInfo = output.css;
        this.callback(error, cssInfo);
    })
}
```

`my-loader.js`

```js
/**
 * 自定义模块规则：
 * 1. 返回内容不能使用箭头函数, 因为loaderAPI的方法挂载到this的对象上
 * 2. 必须有返回值（将处理后的文件内容返回给webpack):
 *  返回值形式有三种：
 *  return、this.callback、this.async()()
 *  return 与  callback 方式差别：单信息与多信息返回
 *  后面两种callback差别：同步与异步
 */
module.exports = function (content, map, meta) {
    console.log(this.query); // 打印信息在npm run build时的终端上
    let newContent = content.replace("loader", "mLoader");
    // 方式1
    // return content.replace("loader", "mLoader"); // 把文件中第一个"loader"字符替换成"mLoader"
    /**
     * 方式2
     * 输入的参数为4个
     * 1. 错误信息 或者 null
     * 2. String or Buffer，传递给下一个loader的内容
     * 3. 可选 contentMap：第三个参数必须是一个可以被 this module 解析的 content map
     * 4. 可选 meta,第四个参数，会被 webpack 忽略，可以是任何东西（例如一些元数据）
     */
    // this.callback(null, newContent);
    // 方式3
    let callback = this.async(); // 告诉 loader-runner 这个 loader 将会异步地回调。返回 this.callback
    setTimeout(() => {
        callback(null, newContent);
    }, 10)
}
```

`webpack.config.js`

```js
// 由于output的输出路径要求绝对路径，使用该模块，根据相对路径组装绝对路径
const path = require("path");

module.exports = {
    // ...其它配置省略
    // 配置loader中模块的查找路径
    resolveLoader: {
        modules: ["node_modules", "./my-loaders"],
    },
    // 配置loader
    module: {
        // 匹配规则
        rules: [
            {
                test: /\.less$/, // 以css后缀的文件,
                /**
                 * 使用哪些模块编译处理
                 * 执行顺序，从右到左
                 * 如下：先编译（css-loader），然后应用(style-loader)
                 */
                use: ["my-style-loader", "my-css-loader", "my-less-loader"], // 自定义loader
            },
            {
                test: /\.s[ac]ss$/, // 文件后缀还是需要scss，否则会报错
                use: ["style-loader", "css-loader", "sass-loader"],
            },
            {
                test: /\.js$/,
                /**
                 * use: String ｜ Object | Array
                 * String: path | 模块名
                 * Object: { loader, options, ...}
                 * Array: [多个loader]
                 */
                // use: path.resolve(__dirname, "./my-loaders/my-loader.js"),
                use: {
                    /**
                     * loader: path or module
                     * 若在resolveLoader配置默认的查找路径，就可以使用自定义模块，否则需要绝对路径
                     */
                    loader: "my-loader",
                    // loader: path.resolve(__dirname, "./my-loaders/my-loader.js"),
                    options: {
                        name: '---------测试模块',
                    }
                }
            }
        ]
    },
}
```

### sass-loader的坑

当样式文件名以.sass为后缀的时候，若报如下错误，则需将.sass 替换成 .scss

```scss
// 文件名 home.sass
h1 {
    color: blue;
}
```

````text
ERROR in ./src/assets/styles/home.sass (./node_modules/_css-loader@5.2.7@css-loader/dist/cjs.js!./node_modules/_sass-loader@7.3.1@sass-loader/dist/cjs.js!./src/assets/styles/home.sass)
Module build failed (from ./node_modules/_sass-loader@7.3.1@sass-loader/dist/cjs.js):

h1 {
  ^
      Expected newline.
  ╷
1 │ h1 {
  │    ^
  ╵
  stdin 1:4  root stylesheet
      in /Users/apple/1study/3project/webpack/webpack01/src/assets/styles/home.sass (line 1, column 4)
 @ ./src/assets/styles/home.sass 2:12-176 9:17-24 13:15-22
 @ ./src/index.js
````

另外，vue的`<style>`也可能有相同问题，需要将``lang="scss"``

```vue
<style lang="sass">
h1 {
    color: blue;
}
</style>
```



### 静态资源处理

#### webpack4 静态资源处理与压缩

**【注】** webpack 5 有内置的模块处理静态资源，不再使用file-loader, url-loader等

静态资源：图片、第三方字体等

处理：资源压缩、优化

图片资源使用场景：

- HTML <img>
- css background-image
- js 处理 Dom

```sh
# 用于静态资源处理，将src中的资源文件拷贝到输出的目标目录中
npm install file-loader -D
# file-loader的加强版，依赖file-loader, 
# 多出功能：将图片资源转化成url(base64)，目的：减少图片的请求次数。
npm install url-loader -D
# 图片压缩loader：npm 方式安装，会有依赖出现问题，且没有提示错误，但在编译时报错
cnpm install image-webpack-loader
```



`webpack.config.js`

```js
// 使用file-loader
module.exports = {
  module: {
        // 匹配规则
        rules: [
            {
                test: /\.(png|jpe?g|gif|webp)$/,
                /**
                 * file-loader:
                 * 用于静态资源处理，将src中的资源文件拷贝到输出的目标目录中
                 * 并处理资源路引用问题
                 */
                use: {
                    loader: "file-loader",
                    options: {
                        name: "[name].[ext]", // [ext] 是后缀的占位符
                        // 输出目录定义 dist/images
                        outputPath: "assets/images",
                        /**
                         * css模块引用资源路径（以css文件输出的目录作为当前目录，去定位资源路径。）
                         * 问题：若资源是嵌套在HTML文件中的时候，怎么办？【解决方案】统一一下处理方式，要嘛处理成独立文件，要嘛嵌套在html里面
                         * 资源做了目录管理（输出可以指定去其它路径），导致资源引入出现了问题
                         */ 
                        publicPath: "../images",
                    }
                }
            }
        ]
    },
}
```

```js
// 使用url-loader
// 使用file-loader
module.exports = {
  module: {
        // 匹配规则
        rules: [
            {
                test: /\.(png|jpe?g|gif|webp)$/,
                /**
                 * file-loader:
                 * 用于静态资源处理，将src中的资源文件拷贝到输出的目标目录中
                 * 并处理资源路引用问题
                 */
                use: [{
                    loader: "url-loader", // file-loader加强版
                    options: {
                        name: "[name].[ext]", // [ext] 是后缀的占位符
                        outputPath: "assets/images",
                        publicPath: "../images",
                        limit: 4 * 1024, // 当图片资源大于4kb时，则作为独立文件，而不转成base64直接加入到css文件中。
                    },

                }, {
                    loader: "image-webpack-loader", // 压缩图片，测试的例子就从16KB压缩到8KB
                }]
        ]
    },
}
```



src 样式文件中 background-image 的 URL 参数是以该样式文件目录位置作为当前位置，去定位相对于资源文件的路径

由于mini-css-extract-plugin 会把style资源打包成独立的文件，并指定输出位置

被编译后的style资源上的引用路径若是保持src中style对资源引用的路径，可能会匹配不到资源

publicPath: 该配置则直接替换源文件中url前缀，

例如：源文件的url =  ../images/ttttt/123132/logo.png

publicPath = "./images"

则编译后输出文件的url = ./images/logo.png，即最后一个 / 之前的符号全部都是前缀，会被publicPath直接替换。

#### webpack5 静态文件处理

`webpack.config.js`

```js
const path = require("path");

module.exports = {
    entry: {
        main: "./src/index.js"
    },
    output: {
        filename: "[name].js",
        path: path.resolve(__dirname, "./dist"),
    },
    mode: "development",
    module: {
        rules: [{
            test: /\.png$/,
            /**
             * type: asset,asset/inline,asset/resource
             * asset: 默认情况下，(maxSize 字段)
             * 资源 >8kb,则生成独立文件（asset/resource），
             * 资源<=8kb,则转成base64（asset/inline）
             */
            type: "asset",
            generator: {
                filename: 'static/[name][ext][query]',
            },
            parser: {
                dataUrlCondition: {
                    maxSize: 4 * 1024, // 默认8KB,
                }
            }
        }]
    }
}
```

## 其它

### npm 配置

在根目录上配置`.npmrc`，用于统一npm源

```npmrc
registry=https://registry.npm.taobao.org
```

该方案比直接使用cnpm好。不知道为什么。。。
