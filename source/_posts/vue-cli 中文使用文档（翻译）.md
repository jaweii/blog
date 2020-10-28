---
title: vue-cli 中文使用文档（翻译）
thumbnail: /img/uploads/2016/10/1-KgEvwnPZwvfEL5mSi4Cyg-1024x320.jpg
tags:
  - vue
  - 原创
  - 翻译
id: 155
categories:
  - WEB向
date: 2016-10-28 21:40:38
---

搜了下，发现没有中文版的，就自己动手了。

> 本内容是对照官方[https://github.com/vuejs/vue-cli](https://github.com/vuejs/vue-cli)的进行的翻译的，有不足之处请查阅原文档。

# vue-cli v2.4.0

一个简单的vue脚手架命令行工具。

### 安装

安装前需要：[Node.js](https://nodejs.org/en/)\(&gt;=4.x,建议5.x\)和[Git](https://git-scm.com/)。

```
npm install-g vue-cli
```

### 使用方法

```
vue init <template-name> <project-name>
```

例子：

```
vue init webpack my-project
```

上面的命令会从[vuejs-templates/webpack](https://github.com/vuejs-templates/webpack)获取模板，输出提示信息后，会在`./my-project/`目录生成项目。

### 官方模板

官方模板的作用是提供一套自有的，功能完备的开发工具配置，以便用户尽可能快的应用到实际代码中。而且，这些模板根据你的应用的构建方式，和你使用的Vue.js以外的库，可以随意变通。

所有的官方模板都存储在[vuejs-templates organization](https://github.com/vuejs-templates)。当有新的模板添加到仓库中，你将可以使用`vue init <template-name><project-name>`命令来使用这个模板。你也可以运行`vue list`命令查看所有可用的官方模板。

最近可用的模板包括：

* [webpack](https://github.com/vuejs-templates/webpack) – 一个功能全面的Webpack + vue-loader设置，包括hot reload（热加载）、linting（代码检查）、testing（测试）和css extraction（css提取）。

* [webpack-simple](https://github.com/vuejs-templates/webpack-simple) – 一个简单的Webpack + vue-loader设置，用于快速制作原型。

* [browserify](https://github.com/vuejs-templates/browserify) – 一个功能全面的Browserify + vueify设置，包含hot reload（热加载）, linting（代码检查）& unit testing（单元测试）。

* [browserify-simple](https://github.com/vuejs-templates/browserify-simple) – 一个简单的 Browserify + vueify设置，用于快速制作原型。

* [simple](https://github.com/vuejs-templates/simple) – 单页HTML应用中尽可能简单的Vue设置。

### 常用模板

要让官方模板使每个人都感到满意是不太可能的。你可以简单的fork一个官方的模板，然后通过`vue-cli`使用它：

```
vue init username/repo my-project
```

其中的username/repo是你fork的GitHub仓库的的简写标记。

简写标记也传递给了[download-git-repo](https://github.com/flipxfx/download-git-repo)，所以你也可以通过`bitbucket:username/repo`使用Bitbucket仓库，还可以通过`username/repo#branch`操作你的tags或branches。

如果你是从私人仓库中用–clone标识下载，cli将会用到git clone命令，这样会用到你的SSH keys。

### 本地模板

你也可以使用本地文件系统中的模板，而不是用GitHub里的。

```
vue init~/fs/path/to-custom-template my-project
```

### 从零开始编写自定义模板

* 一个模板仓库必须有一个保存模板文件的template目录。
* 一个模板仓库可以有一个用于模板的元数据文件，它可以是 meta.js 或 meta.json 。它可以包含以下字段：

> 1. prompts:用来收集用户的选项数据；
> 2. filters: 用来渲染符合条件的文件；
> 3. completeMessage: 当模板生成完毕后会有消息提示。你可以在这里写入自定义指令。

#### `prompts`

元数据中的 `prompts` 字段用来为用户提示消息。每一个条目的key是一个变量名，value是一个 [Inquirer.js question 对象](https://github.com/SBoudrias/Inquirer.js/#question)。

> ps.\(Inquirer 是常规交互式命令行用户接口的集合，提供给 Node.js 一个方便嵌入，漂亮的命令行接口。Inquirer 会简化询问终端用户问题，解析，验证答案，提供错误反馈等等功能\)

例子：

```
{"prompts":{"name":{"type":"string","required": true,"message":"Project name"}}}
```

在所有prompts完成后，模板中的所有文件，将使用prompt的结果作为数据，使用[Handlebars](http://handlebarsjs.com/)模板引擎渲染。



#### `conditional prompts`（条件提示）

你可以添加一个 `when` 字段，使prompt支持条件判断， `when` 字段应该是一个用来触发自己的prompt的key。

例如：

```
{"prompts":{"lint":{"type":"confirm","message":"Use a linter?"},"lintConfig":{"when":"lint","type":"list","message":"Pick a lint config","choices":["standard","airbnb","none"]}}}
```

上面 `lintConfig` 的prompt只会在用户对 `lint` 的prompt回应“yes”时才会被触发。



pre-registered Handlebars Helpers\(预注册Handlebars功能函数\)

两种常用的预注册功能函数是，`if_eq` 和 `unless_eq`：

```
{{#if_eq lintConfig"airbnb"}};{{/if_eq}}
```

Custom Handlebars Helpers\(自定义Handlebars功能函数\)

你也许想用元数据中的helpers属性注册额外的Handlebars功能函数。其对象的key就是helper的名字：

```
module.exports={helpers:{lowercase: str=> str.toLowerCase()}}
```

注册完成后，可以像下面这样使用：

```
{{ lowercase name}}
```

### 文件过滤器

元数据文件中的 `filters` 字段应该是一个包含了文件过滤规则的对象，没个条目中，key为[minimatch glob pattern](https://github.com/isaacs/minimatch)，value字段应该是一个用来触发自己的prompt的key。

例如：

```
{"filters":{"test/**/*":"needTests"}}
```

如果用户对 `needTests` prompt的回应是yes，就会生成被测试的文件。

注意，minimatch的 `dot` 选项是设置为true的，所以glob模式默认也会匹配dotfiles。

### meta.{js,json}中的其他可用选项

* `destDirName` – 目标路径

```
{"completeMessage":"To get started:\n\n  cd {{destDirName}}\n  npm install\n  npm run dev"}
```

* `inPlace`– 生成目录到当前模板

```
{
"completeMessage": "{{#inPlace}}To get started:\n\n npm install\n npm run dev.{{else}}To get started:\n\n cd {{destDirName}}\n npm install\n npm run dev.{{/inPlace}}"
}
```

`vue-cli` 可以用 [`download-git-repo`](https://github.com/flipxfx/download-git-repo) 工具下载特定版本的官方模板， `download-git-repo` 允许你通过在 `#` 后提供一个你希望的分支名字，来从指定仓库中获取指定的分支。

其格式必须是这样的：

```
vue init<template-name>#<branch-name><project-name>
```

例子：

安装 webpack-simple模板 [`1.0`](https://github.com/vuejs-templates/webpack-simple/tree/1.0)版本的分支：

```
vue init webpack-simple#1.0 mynewproject
```
