---
title: vue服务端渲染框架nuxt.js中文文档（翻译）
thumbnail: /img/uploads/2016/12/3.jpg
tags:
  - vue
  - 原创
  - 翻译
id: 237
categories:
  - WEB向
date: 2016-12-17 12:55:49
---

之前我写了个spa应用后，意识到了spa应用的一大弊病，就是SEO超差。

根据我多年的SEO经验，不做好SEO，何来流量？

没有流量，还做什么产品。

那么在百度不思进取，不抓取js内容的情况下，只能用服务端渲染了。

[https://nuxtjs.org/](https://nuxtjs.org/) | [gihub](https://github.com/nuxt/nuxt.js)

NUXT.js，一个轻量级的服务端渲染框架，vue应用专享。

#### 安装

```
npm install nuxt--save
```

添加一个脚本到你的package.json,像这样：

```
{"scripts":{"start":"nuxt"}}
```

然后，文件系统会成为主要的API。

所有的 .vue 文件会变成自动处理和渲染的路由。

填充下面代码到你的项目中的./pages/index.vue中：

```
<template><h1>Hello {{ name }}!</h1></template><script>exportdefault{data:()=>{return{name:'world'}}}</script>
```

然后运行：

```
npm start
```

然后访问：http://localhost:3000

现在，我们有了这些功能：

* 自动语法编译和打包（用babel和webpack）
* 代码热重载
* 服务端渲染和 ./pages 索引
* 静态文件服务， ./static/被映射到了 /
* 配置文件nuxt.config.js
* 通过webpace分割代码

#### 编程式使用

Nuxt是建立在ES2015之上的，这使得编码更加愉快，阅读更加清晰。Nuxt不依赖任何转换器和V8引擎实现的功能。 nuxt.js支持Node.js 4.0或更高版本（如果你的node版本低于6.5.0，则可能需要配合 -harmony-proxies 运行node）。

```
const Nuxt = require('nuxt')
const options = {
routes: [], // see examples/custom-routes
css: ['/dist/bootstrap.css'] // see examples/global-css
store: true // see examples/vuex-store
plugins: ['public/plugin.js'], // see examples/plugins-vendor
loading: false or { color: 'blue', failedColor: 'red' } or 'components/my-spinner' // see examples/custom-loading
build: {
vendor: ['axios'] // see examples/plugins-vendor
}
}
// Launch nuxt build with given options
let nuxt = new Nuxt(options)
nuxt.build()
.then(() => {
// You can use nuxt.render(req, res) or nuxt.renderRoute(route, context)
})
.catch((e) => {
// An error appended during the build
})
```

#### 中间件式使用

你也许想用自己配置的服务、自己的API，或任何自己的得意之作。那么你可以像使用一个中间件一样使用nuxt.js。

推荐你在中间节的结尾处使用它。因为它负责处理你的web应用的渲染，并且它不会调用next\(\)。

```
app.use(nuxt.render)
```

#### 渲染指定路由

```
nuxt.renderRoute('/about', context)
.then(function ({ html, error }) {
// You can check error to know if your app displayed the error page for this route
// Useful to set the correct status status code if an error appended:
if (error) {
return res.status(error.statusCode || 500).send(html)
}
res.send(html)
})
.catch(function (error) {
// And error appended while rendering the route
})
```

#### 例子

请参考examples/目录。

如果你想运行一个例子：

```
cd node_modules/nuxt/
bin/nuxt examples/hello-world
# Go to http://localhost:3000
```

你可能想提前构建，而不是运行nuxt。所以构建和开始是分开的命令。

```
nuxt buildnuxt start
```

举个例子，要通过package.json部署，推荐如下配置：

```
{"name":"my-app","dependencies":{"nuxt":"latest"},"scripts":{"dev":"nuxt","build":"nuxt build","start":"nuxt start"}}
```

然后运行，很简单吧！

提示：推荐将.nuxt推入.npmignore或.gitignore

> 原文档地址：[https://github.com/nuxt/nuxt.js](https://github.com/nuxt/nuxt.js)
>
> 官网的api还在完善中，很多api都是写着“Documentation is coming soon”
>
> 所以api就不翻了。。。


