---
title: 对基于vue-cli创建的项目进行webpack配置和环境配置
tags:
  - 前端
  - vue
  - webpack
  - 项目环境配置
categories:
  - 前端
  - vue
  - 对基于vue-cli创建的项目进行webpack配置和环境配置
abbrlink: 69d232dc
date: 2022-07-14 21:01:59
---


### 一、安装vue cli 并 创建一个vue项目



```bash
# 安装脚手架
npm install -g @vue/cli

# 查看版本
vue --version
```



```shell
# 创建项目 vue-demo
vue create vue-demo

# 项目创建完成后打开项目
```



### 二、基本目录结构



```
├── node_modules  			 	# 依赖包
|-- dist                    	# 打包后文件夹
├── public						# 静态文件夹，webpack不会处理
│   ├── favicon.ico
│   └── index.html				# 入口页面
├── src						    # 源码目录
│—— .env.development       	    # 环境变量和模式,根据需求自行配置
│—— .env.production
│—— .env.staging
│—— .browserslistrc             # 浏览器兼容版本配置
│—— .eslintrc.js    		  	# eslint
│—— .eslintignore    		    # 根据需要自行配置
│—— .gitignore          		# git忽略哪些文件    
│—— babel.config.js   			# babel语法编译
│—— jsconfig.json    		  	# js语言配置
├── package-lock.json
│—— package.json
├── README.md
│—— vue.config.js               # 配置文件，需自行配置
```



### 三、开发生产测试三种模式的配置



```json
// package.json

"scripts": {
    "serve": "vue-cli-service serve",
    "dev": "vue-cli-service serve",
    "build": "vue-cli-service build",
    "lint": "vue-cli-service lint"
},
```



当运行 `vue-cli-service` 命令时，所有的环境变量都从对应的环境文件中载入。

如果文件内部不包含 `NODE_ENV` 变量，它的值将取决于模式。

例如，在 `production` 模式下被设置为 `"production"`，在 `test` 模式下被设置为 `"test"`，默认则是 `"development"` 。

```
比如：我运行
npm run dev --> process.env.NODE_ENV = 'development'
npm run build --> process.env.NODE_ENV = 'production'
```


  如果在环境中有默认的 `NODE_ENV`，你应该移除它或在运行 `vue-cli-service` 命令的时候明确地设置 `NODE_ENV`。 

所以在根目录下专门创建 `三个环境配置文件`来配置开发生产测试环境下的 `环境变量`。



```
根目录下创建三个环境配置文件
│—— .env.development       	    # 开发环境
│—— .env.production             # 生产环境
│—— .env.staging                # 测试环境
```



#### .env.development

```
# 开发环境
# 用于开发人员开发时本地调试项目
# npm run dev

# 明确地设置 NODE_ENV
NODE_ENV = development

VUE_APP_ENV = 'development'

# 自定义的一些环境变量一般以 VUE_APP_
# 接口请求的基本路径
VUE_APP_BASE_URL = 'http://www.abc.com'
```






#### .env.production



 `vue-cli-service build` 会加载可能存在的 `.env`、`.env.production` 和 `.env.production.local` 文件然后构建出生产环境应用 



```
# 生产环境
# # npm run build

# 明确地设置 NODE_ENV
NODE_ENV = production

VUE_APP_ENV = 'production'

# 生产环境中的 接口请求地址
VUE_APP_BASE_URL = 'http://www.efg.com'
```






#### .env.staging



 `vue-cli-service build --mode staging` 会在 staging 模式下加载可能存在的 `.env`、`.env.staging` 和 `.env.staging.local` 文件然后构建出生产环境应用 



```json
// package.json 调整 来区分 打生产包还是测试包

"scripts": {
    "serve": "vue-cli-service serve",
    "dev": "vue-cli-service serve",
    "build:prod": "vue-cli-service build",
    "build:stage": "vue-cli-service build --mode staging",
    "lint": "vue-cli-service lint"
},
```



```
# 测试环境

NODE_ENV = production

# 可以用来区分 生产环境和测试环境
VUE_APP_ENV = 'staging'

VUE_APP_BASE_URL = 'http://www.xyz.com'
```








### 四、vue.config.js



```js
//  具体配置的官方文档  https://cli.vuejs.org/zh/config/#baseurl
// 部分配置参考此博客 https://juejin.cn/post/6998322455173398536
const path = require('path')
const CompressionPlugin = require("compression-webpack-plugin") // 开启gzip压缩
const gzipRegExp = /\.(js|css|json|txt|html|ico|svg)(\?.*)?$/i
const resolve = (dir) => path.join(__dirname, dir)



module.exports = {
  publicPath: './',  // 基本路径
  outputDir: 'dist', // 构建时的输出目录
  assetsDir: 'static', // 放置静态资源的目录
  // 可以根据开发环境来设置是否开启 检查
  // indexPath: 'index.html', 默认是dist下的index.html,如果想改为 dist/html/index.html，则 配置为 indexPath: 'html/index.html'
  lintOnSave: process.env.NODE_ENV === 'development', // 是否在保存的时候使用eslint-loader进行检查。 只在开发模式下开启。该配置只有在@vue/cli-plugin-eslint安装后才会生效。

  transpileDependencies: [], // babel-loader 默认会跳过 node_modules 依赖。数组中可以传递需要转译的第三方包名或者正则表达式
  productionSourceMap: false, // 生产环境 是否生成 source map
  parallel: require("os").cpus().length > 1, // 是否为 Babel 或 TypeScript 使用 thread-loader

  // 调整 webpack 配置
  // 如果这个值是一个函数，则会接收被解析的配置 config 作为参数
  configureWebpack: config => {

    // 开启gzip压缩
    // npm install -D compression-webpack-plugin
    // compression-webpack-plugin 提前做gzip 处理得到 .gz 文件， 当浏览器访问静态资源时，静态资源服务器根据请求头中 Accept-Encoding 字段判断请求端是否支持 gzip 解压，如果支持，那么返回 .gz 文件，否则返回原文件。
    // 需要服务端 nginx配置开启gzip
    if (process.env.NODE_ENV === 'production') {
      config.plugins.push(
        new CompressionPlugin({
          filename: "[path][base].gz",
          algorithm: "gzip", // 压缩算法/函数
          test: gzipRegExp,
          threshold: 10240, // 资源文件大于 10kB时会被压缩
          minRatio: 0.8  // 最小压缩比达到0.8时才会被压缩
        })
      )
    }

  },

  // webpack-chain API  https://github.com/neutrinojs/webpack-chain#getting-started
  chainWebpack: config => {
    config.resolve.symlinks(true) // 修复热更新失效

    // 添加别名
    config.resolve.alias
      .set('@', resolve('src'))
      .set('@assets', resolve('src/assets'))
      .set('@components', resolve('src/components'))
      .set('@views', resolve('src/views'))
      .set('@store', resolve('src/store'))
      .set('@config', resolve('src/config'))

  },

  // css: {
  //   extract: process.env.NODE_ENV === 'production', // 是否将组件中的 CSS 提取至一个独立的 CSS 文件中, 有点像 mini-css-extract-plugin
  //   // requireModuleExtension: false // 去掉文件名中的 .module。 webpack5中应该用如下写法
  //   loaderOptions: {
  //     css: {
  //       modules: {
  //         auto: () => true
  //       }
  //     }
  //   }
  // },

  // 配置 webpack-dev-server
  // https://webpack.docschina.org/configuration/dev-server/#root
  devServer: {
    open: true, // 编译后默认打开浏览器, 类似 --open
    // host: '0.0.0.0',  // 域名
    port: 8080,  // 端口
    // https: false,  // 是否https
    // disableHostCheck: true, 是否关闭用于dns重绑定的http请求的host检查 devServer默认只接收本地的请求，关闭后可以接收人任何host的请求,webpack5中被遗弃，需要替换为 historyApiFallback 和 allowedHosts
    historyApiFallback: true,
    allowedHosts: 'all',
    // // 当出现编译错误或警告时，在浏览器中显示全屏覆盖, webpack5中写法有些改变
    // overlay: {
    //   warnings: false,
    //   errors: true
    // },
    client: {
      overlay: {
        errors: true,
        warnings: false,
      },
    },
  }
}





```





