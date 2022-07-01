title: 写了个 VS Code 中实时预览 Vue/React 组件的插件
author: 朱嘉伟
date: 2022-03-16 10:35:56
tags:
---

最近年纪大了喜欢胡思乱想，前段时间突然想到能不能在VS Code中实现组件的所见即所得，于是折腾了两个月终于做出了能实现这个效果的插件，如图：

![](https://raw.githubusercontent.com/jaweii/AutoPreview/main/demo/img/webpack5_react.gif)

支持实时预览Webpack/Vite开发时下的React/Vue组件（Angular应该也能，但是我还没用过，所以没做支持）。


**使用方式**

按参考文档安装并使用插件：[https://github.com/jaweii/AutoPreview](https://github.com/jaweii/AutoPreview/blob/main/README-zh.md)

然后你就可以写组件时实时预览当前组件：

![](https://raw.githubusercontent.com/jaweii/AutoPreview/main/demo/img/1.png)


还可以把可复用组件/物料的用例整理在一个预览专用的文件里：

![](https://raw.githubusercontent.com/jaweii/AutoPreview/main/demo/img/2.png)


在OUTPUT面板切到AutoPreview来打印调试：

![](https://raw.githubusercontent.com/jaweii/AutoPreview/main/demo/img/4.png)

> 断点功能：emmm...? 我也想有...


把预览面板拖到底部来预览比较宽的组件：
![](https://raw.githubusercontent.com/jaweii/AutoPreview/main/demo/img/5.png)


**实现方式**

一开始我是只想实现Webpack+React的组件预览的，尝试了从Webpack配置着手来实现，但是发现这条路走不通，依赖关系太错综复杂了，搁置几天后想到了我以前Vue-Layout项目中组件重新挂载的思路，几番尝试后找到实现的关键，即通过Webpack和Vite都提供的import api来异步加载当前窗口文件路径的组件，然后重新挂载到预览窗口，即可实现预览。

所以其本质上和你给项目新增个路由来显示当前页面中的组件一样，只是插件自动帮你做了。

而因为使用了Webpack和Vite都提供的import api，使用这两种构建工具开发时，所有的前端框架理论上都可以实现在VS Code中渲染项目组件实现预览。


**延伸**

现在我也只是写了几个Demo来测试插件效果，对实际开发过程中是有增效还是鸡肋我也不确定，只是感兴趣就做了。

在做的过程中也有一些思考：

如果给可预览的组件分级，那么有

-   基础组件
-   物料(基础组件、元素、业务逻辑之间的组合，比如登录框)
-   页面

    这三种级别。

基础组件

通常实际项目中，基础组件是来自内部或第三方组件库，高复用，低耦合，其提供的文档已经能够预览组件效果，针对这类组件的IDE内预览似乎意义并不是很大；

物料

物料则是根据产品需求对基础组件、元素、业务逻辑进行组合的产物，这类物料有的是项目内可复用的，有的是项目内没有复用但是跨项目存在复用，有的是业务定制完全不可复用的。

实际项目的协同中，物料也是最容易被重复写的，且随着项目越来越大其会散落在各个内页，没有一个展示页来让不同开发者知道哪些物料是已经有了的，就会造成重复造物料。对于这个问题，阿里飞冰、京东JD WORK这样的开发链工具是一种解决方案，其提供的物料制作、发布、使用一条龙服务可以很大程度上避免重复造物料，是个重武器。

我想到了个轻武器的方案，即针对物料的IDE内预览，若合理约定、使用，或许也会是改善重复造物料问题的一个方案。比如约定开发者对可复用的物料导出预览，这样插件可以列出所有可预览的组件供其他开发者浏览，其他开发者开发新物料前，先在预览列表看看有没有可复用的，如果有直接参考复用或CV定制，没有再自己写。

页面

页面级则是复用性极低，不过IDE内预览H5页面倒也是种不错的体验，不用在浏览器和编辑器间切换，有点微信开发者工具的感觉。

![](https://raw.githubusercontent.com/jaweii/AutoPreview/main/demo/img/3.png)

----

还有一点看法，如果组件的预览能够优化开发体验，我想并不是因为它能预览，而是因为其强制组件作者写预览函数，预览组件，天然需要为组件props传递mock数据，而mock的数据能供自己和其他开发者参考和CV，这是其提高代码可维护性，降低协同成本的很重要的原因。

且如果预览函数能罗列出组件的多个用例，对于其他开发者来说是多么心旷神怡的事：

![](https://raw.githubusercontent.com/jaweii/AutoPreview/main/demo/img/2.png)

----

