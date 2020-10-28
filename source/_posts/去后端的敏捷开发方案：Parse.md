title: 去后端的敏捷开发方案：Parse
author: 朱嘉伟
tags:
  - Parse
  - 原创
categories:
  - WEB向
date: 2018-01-22 21:13:00
---
就在昨天，我完成了Parse JavaScript SDK指南的翻译，并在指南中加入了一些教程、示例，以保证新手能够快速上手。

我在QQ群里面贴出[地址](https://parse-zh.buzhundong.com/)宣传了下，然后发现大部分人听都没听说过Parse，果然，我是站在潮流之巅的弄潮儿啊hhhhhh~

也有管理员看了，觉得Parse很棒，与是我就获得了两个群的管理员资格。

那么，Parse到底是什么？

三个字就可以概括：去后端。

Parse服务其实已经推出很多年了，在4年前被Facebook 8500万美元收购，在去年被Facebook开源，但是在国内几乎一直无人问津。

国内同类的服务平台，即BasS\(Backend as a Service\)服务平台主流的有Leancloud、Bmob。

其中Leancloud我使用过，Bmob也注册过。Leancloud达到免费阈值以后的收费方案是每天最低消费30元，Bmob达到免费阈值以后的收费方案有99元/月的套餐。对于原型开发阶段的应用，其服务的免费额度使用还是绰绰有余的。

就像刚才说的，BaaS服务就是去后端，也就是说，后端几乎完全不需要你自己开发了，数据库、用户系统、安全系统、Hook回调、API查询等都为你搭建好了。而且，比你搭建的还好。你只需要在项目中集成SDK，然后直接调用方法传参就可以完成前后端交互。

打个比方，我要写一个按钮的用户注册方法：

```js
//集成SDK
import parse from 'parse'
parse.serverURL = 'http://localhost:2018/parse'
parse.initialize('myAppId', '123456')

//在按钮点击实践中调用注册方法：
button.on('click',function(username,password){
let user = new parse.User()
user.set('username', '000001')
user.set('password', 'lzhlmcl,yhblsqt')
user.set('email', 'xxxxx@qq.com')
user.signUp().then(user => {
alert('注册成功')
}).catch(console.error)
})
```

完成！

只需要十几行代码，不需要写后端和接口。当上面代码弹出注册成功后，后台还会发送验证邮箱的邮件给用户，然后后台数据库就会出现新注册的用户。

怎么样，so easy吧。

简直就是，前端抢后端饭碗系列。

当然了，使用Leancloud、Bmob这样的商业服务平台，数据是存储在他们的服务器上的。使用Parse自己搭建后端，则是保存在自己的服务器上。

对比一下：

Parse开源，数据库绝对私有，完全免费，这是Parse的优点。

Leancloud、Bmob则是本地化做的非常优秀，比如对微信接口、短信验证的集成，这是他们的优点。

上述优点，双方都不具备对方的优点，所以，让你选的话，你选胸大的还是腿长的？

我一般是结合业务，如果我觉得这个应用就是玩玩，肯定做不大，为了省去维护和服务器维护，我会选择使用Leancloud，Bmob不熟练。如果说这个应用以后会有很大访问量，打算认真做做，预期会超过L、B的免费额度，出于平穷限制，我会选择Parse。

不管你选择哪个，Parse其实都值得你掌握的。

身为一个前端，如果你会使用L、B，首先也是不错的，至少开发效率很高了；不过，依赖于第三方商业平台，还是不够独立，算不得全栈开发者。如果你会使用Parse，你就相当于快速掌握了一种后端技术栈，算得上是全栈了。

我为什么这么推荐Parse，并把Parse的文档翻译中文，整合详细的教程和实例呢？Parse是一个开源项目，也不是我的项目，我就是非常喜欢这类框架，你知道，发现一个好东西，总是特别想推荐给朋友们的。

接下来，我还会继续写几个教程和例子，虽然，不是非常必要，有的人文档过一遍就会用了。

最后附上[Parse JavaScript SDK中文文档地址](https://parse-zh.buzhundong.com/)。





