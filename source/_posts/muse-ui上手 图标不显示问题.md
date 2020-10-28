---
title: muse-ui上手 图标不显示问题
thumbnail: /img/uploads/2016/12/IMG_0873-1024x342.jpg
tags:
  - ui
  - vue
  - 原创
  - 解决方案
id: 211
categories:
  - WEB向
date: 2016-12-03 13:46:48
---

muse-ui是我很喜欢的ui，其原生般的移动动效，和基于vue 2.0，还是很受欢迎的。

正好我在学vue，刚好拿muse-ui练手。

**遇到了图标不显示的问题，发现是谷歌字体和meterial图标库的问题，被墙了。**

官网演示效果：

<!--StartFragment -->

![3](/img/uploads/2016/12/3-640x130.png)

上手效果：

![2](/img/uploads/2016/12/2.png)

由于字体图标被墙，左右两边的图标都没显示。

**解决：**

我调试后的解决方法是，在style中@import了字体和图标的cdn库：

@import &#39;//fonts.useso.com/css?family=Roboto:300,400,500,700,400italic&#39;;

@import &#39;http://cdn.bootcss.com/material-design-icons/3.0.1/iconfont/material-icons.css&#39;;

在github上向作者提交issue，作者给的方案是：

> 实际项目中可以下载下来直接放在项目中使用。

> *[icons 下载地址](https://github.com/google/material-design-icons/tree/master/iconfont)

> *[roboto font 下载地址](https://material.google.com/resources/roboto-noto-fonts.html)&nbsp;**需要翻墙**

![4](/img/uploads/2016/12/4-640x303.png)

发现群里有人问这个的问题，就记录下来了~