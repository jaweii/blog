title: H5实现iPhoneX上的Animoji
thumbnail: /img/20180617-0.jpg
author: 朱嘉伟
tags:
  - 原创
  - 3D
  - 创意
categories:
  - WEB向
date: 2018-06-17 16:41:00
---
先上GIF：

![](/img/20180617-1.gif)



前段时间突发奇想，想到前端是不是可以利用摄像头设备增强页面的交互能力。

比如我们见过有些页面设计，页面元素会随着鼠标的移动而移动，结合元素的层次感，可以形成视差效果。

那么，如果能获取到人脸在屏幕前的移动变化，就可以做到页面元素跟随人脸变化，而不是跟随鼠标变化，这可能是种更棒的视差体验。

于是动手实践后，就有了这个效果：

![](/img/20180617-2.gif)


做出来后，我又想到，如果把这个应用在3D游戏里，比如CS，就可以实现一个场景：玩家在屏幕前把脑袋往右边伸，屏幕里的那个躲在转角处的土匪人物，就能看到左边更大的视野。

想想还不错吧？

然而我不是做游戏的，这个idea我发挥不了，如果有做3D页游的程序猿看到，可以试试看。

又过了一两个星期，我又想到了iPhoneX上的animoji，也可以通过这样的方式实现。

于是动手实践后，就有了你最先看到的那张图的效果。

但是我的3D这方面还不是很擅长，所以没能实现让那坨便便的表情跟随人的表情变化。

# Live Demo

[视差DEMO](https://jaweii.github.io/AR-showcase/dist/index.html)

[Animoji](https://jaweii.github.io/AR-showcase/dist/animoji.html)

# 源码

源码仓库：https://github.com/jaweii/AR-showcase

源码克隆下来后，安装依赖项:

```
npm install
```

以及安装parcel到全局：

```
npm install -g parcel-bundler
```

启动命令：

```
// 启动Animoji demo
npm run dev:animoji

// 启动视差Demo
npm run dev
```

# 实现方式

通过计算机的摄像设备捕捉画面，使用[Clmtrackr.js](https://github.com/auduno/clmtrackr)识别人脸，3D引擎使用的Three.js。

Clmtrackr.js可以捕捉到人脸在屏幕中的各个位置：


![](/img/20180617-3.png)

在项目启动后，应用会不断的获取上图\[41\]点的坐标，当位置变化后，3D物体也做相应的变化。

