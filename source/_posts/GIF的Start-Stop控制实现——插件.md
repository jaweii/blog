title: GIF的Start/Stop控制实现——插件
author: 朱嘉伟
thumbnail: /img/poster/giffer.jpg
tags:
  - 原创
categories:
  - WEB向
date: 2018-04-28 18:13:00
---
最近项目中遇到了需要控制GIF图片的播放、停止的需求，查了资料发现并没有现成的API，好在可以用Canvas重画来截取帧。

因为是Vue的项目，所以封装成了Vue的插件。

# 效果

![](/img/20180428.gif)

# 使用方法

将下面代码保存到Giffer.js：

```js
// Giffer.js
const plugin = {
  install(Vue) {
    Vue.directive('gif', {
      bind: plugin.update,
      componentUpdated: plugin.update,
      updated: plugin.update
    })
  },
  async update(el, { value }, { context }) {
    el.__proto__.$play = _ => { }
    if (!value) return
    const originSrc = el.getAttribute('data-origin-src')
    if (!isGifImage(el) && !originSrc) return
    //Waiting for the render to finish
    await new Promise(resolve => setTimeout(resolve, 0))
    const autoplay = value.autoplay
    freezeGif(el)
    el.__proto__.$play = plugin.play.bind(el)
    el.__proto__.$stop = plugin.stop.bind(el)
    if (!autoplay) return
    else el.$play()
  },
  play() {
    const originSrc = this.getAttribute('data-origin-src')
    this.setAttribute('src', originSrc)
  },
  stop() {
    freezeGif(this)
  }
}

export default plugin

function isGifImage(el) {
  return /^(?!data:).*\.gif/i.test(el.src)
}

function freezeGif(el) {
  var canvas = document.createElement('canvas')
  var w = canvas.width = el.width * 10
  var h = canvas.height = el.height * 10
  const src = el.src
  if (isGifImage(el)) el.setAttribute('data-origin-src', src)
  canvas.getContext('2d').drawImage(el, 0, 0, w, h)
  el.src = canvas.toDataURL("image/gif")
}
```

然后在项目中引入：

```
import Giffer from './plugin/Giffer'
Vue.use(Giffer)
```

然后为需要控制的img标签添加v-gif指令：

```
//指令接受一个options对象：包含一个autoplay键值，功能类似于video的autoplay属性
<img v-gif="{autoplay:false}" ref="gif">
```

然后img标签就具备了`$play`和`$stop`方法，可以这样使用：

```
this.$refs.gif.$play()
this.$refs.gif.$stop()
```

---

小小的功能，就不发布到npm了~