title: ul>li*3 实现翻书动画效果
thumbnail: /img/uploads/2017/07/GIF.gif
tags:
  - 动画
id: 374
categories:
  - WEB向
date: 2017-07-07 01:43:00
---
按惯例，上GIF

![GIF](/img/uploads/2017/07/GIF.gif)

重现：[https://codepen.io/anon/pen/JJBxOm](https://link.juejin.im/?target=https%3A%2F%2Fcodepen.io%2Fanon%2Fpen%2FJJBxOm)

来看看HTML部分：

```
 <body>
     <ul>
         <li>
         <li class="anim">
         <li class="anim2">
     </ul>
 </body> 
```

由于li是inline-block元素，所有没有写li的闭合，写了的话每个li直接会有4px的间距，不写浏览器也会自动补全。

CSS部分

```    
	body {
        text-align: center;
    }
    
    ul {
        background: gray;
        width: 100%;
        padding: 20px;
        -webkit-perspective: 200;
    }
    
    li {
        list-style: none;
        height: 50px;
        width: 100px;
        padding: 0;
        margin: 0;
        display: inline-block;
        background: white;
        border-radius: 2px;
    }
    
    .anim {
        animation: anim 1s infinite;
        width: 100px;
        margin-left: -100px;
        background: white;
    }
    
    @keyframes anim {
        to {
            transform: rotateY(-360deg);
        }
    }
    
    .anim2 {
        animation: anim2 1s infinite;
        width: 100px;
        margin-left: -100px;
        background: white;
    }
    
    @keyframes anim2 {
        25% {
            transform: rotateY(0deg);
        }
        to {
            transform: rotateY(-360deg);
        }
    }
```

