title: Parse服务的手机验证码用户注册/登录教程
author: 朱嘉伟
tags:
  - Parse
  - ''
categories:
  - WEB向
date: 2018-02-02 00:09:00
---
> 夜深人静，硬睡不着，写个教程。

有的产品为了简化用户的注册或登录，会采取验证码注册/登录的方式。

我写了个Parse结合云代码实现手机验证码注册/登录的示范，在下面云代码中，我们定义了两个云函数，`requestSmsCode`和`signUpOrlogInWithMobilePhone`。

函数一接收一个手机号码参数，用来下发验证码；函数二接收一个手机号码和验证码参数，用来确认验证验证码，验证成功后，判断用户是否存在，不存在则注册并返回sessionToken，存在则登录并返回sessionToken。

此代码中应用了ES6的特性，如果复制使用下面代码，你需要使用Babel或Node.js &gt; 7.6版本，我用的v8.9.3。

另外，本例子中使用的是Leancloud的验证码服务，因为Leancloud的短信签名对个人开发者也很开放，不像阿里云、腾讯云对个人开发者要求备案、授权书等。所以，你需要先注册/登录Leancloud，创建应用，将下面的appId、appKey替换成你的。并且，为ParseServer安装leancloud-storage依赖。

```js
// \parse-server-example-master\cloud\main.js
var moment = require('moment')
var AV = require('leancloud-storage')
var APP_ID = '你的leancloud app id'
var APP_KEY = '你的leancloud app client key'
AV.init({
  appId: APP_ID,
  appKey: APP_KEY
})

//获取验证码
/**
 *@params phoneNumber:string
 */
Parse.Cloud.define('requestSmsCode', function(req, res) {
  let phoneNumber = req.params.phoneNumber
  AV.Cloud.requestSmsCode({
    mobilePhoneNumber: phoneNumber
  }).then(function() {
    //调用成功
    res.success('发送成功')
  }, function(err) {
    //调用失败
    res.error('发送失败')
  });
})

//使用短息验证码注册或登录系统，用户不存在时自动注册。
/**
 *@params phoneNumber:string
 *@params code:string
 */
Parse.Cloud.define('signUpOrlogInWithMobilePhone', function(req, res) {
  let phoneNumber = req.params.phoneNumber,
    code = req.params.code
  if (!phoneNumber || !code) {
    return res.error('手机号码或验证码不能为空')
  }
  AV.Cloud.verifySmsCode(code, phoneNumber).then(function() {
    let query = new Parse.Query(Parse.User)
    query.equalTo('phone', phoneNumber)
    query.limit(1)
    query.include(['smsCode'])
    return query.find({ useMasterKey: true })
  }).then(async function([user]) {
    if (user) {
      //登录
      return Parse.User.logIn(user.get('phone'), user.get('smsCode').get('code')).then(user => {
        return res.success(user.getSessionToken())
      })
    } else {
      //注册
      let user = new Parse.User()
      user.set('username', phoneNumber)
      // 使用验证码作为密码
      user.set('password', code)

      // 因password不可见，固保存到SmsCode，并禁止任何人读写，仅用作云代码登录用。
      // 若担心黑客使用监听网络拿到验证码登录，可以写相关触发器拒绝6位数密码登录
      let SmsCode = new Parse.Object('SmsCode')
      let codeACL = new Parse.ACL()
      codeACL.setPublicReadAccess(false)
      codeACL.setPublicWriteAccess(false)
      SmsCode.setACL(codeACL)
      SmsCode.set('code', code)
      SmsCode = await SmsCode.save()

      user.set('smsCode', SmsCode)
      user.set('phone', phoneNumber)
      user.set('phoneVerified', true)
      return user.signUp().then(user => {
        return res.success(user.getSessionToken())
      })
    }
  }).catch(res.error)
})

```

OK，云函数写好，运行ParseServer服务后，下一步就是在客户端调用者两个云函数了。

在需要请求验证码下发的函数里，调用：

```js
//发送验证码
Parse.Cloud.run('requestSmsCode', { phoneNumber: this.cellphoneNumber.text }).then(res => {
	Pop.toast('验证码已发送')
},err => {
	Pop.toast(`发送失败(${err})`)
})
```

用户输入验证码后，通常就是点击注册、登录、验证之类的按钮，在对应的事件中调用：

```js
// 登录，获取用户信息
Parse.Cloud.run('signUpOrlogInWithMobilePhone',{phoneNumber:this.cellphoneNumber.text, code:this.SmsCode.text}).then(async (sessionToken) => {
	//更新用户session
	const user = await Parse.User.become(sessionToken)
	//关闭loading
	Pop.closeLoading()

	//登录成功关闭登录框
	Pop.toast('登录成功')

	this.close()
},err => {
	console.log(err)
	Pop.closeLoading()
	Pop.toast(`登录失败${err}`)
})
```

OK，在这个接口调用成功后就可以返回sessionToken，然后调用`Parse.User.become` 就可以更新本地当前用户了。然后你可以通过`Parse.User.current()`调用。

如果，你想使用其他的短信平台，比如阿里云，更改两个云函数、增加验证码缓存即可，我写了个大概的示例：

```js
const util = require('./util')
const SMSClient = require('@alicloud/sms-sdk')

// ACCESS_KEY_ID/ACCESS_KEY_SECRET 根据实际申请的账号信息进行替换
const accessKeyId = 'ACCESS_KEY_ID'
const secretAccessKey = 'ACCESS_KEY_SECRET'
const smsSignName = '五星',
    smsTemplateCode = 'SMS_123666602'

//初始化sms_client
let smsClient = new SMSClient({ accessKeyId, secretAccessKey })
let smsCodeCache = {}


Parse.Cloud.define('hello', function(req, res) {
    res.success('Hi')
})

//获取验证码，阿里云短信接口
/**
 *@params phoneNumber:string
 */
Parse.Cloud.define('requestSmsCode', function(req, res) {
    let phoneNumber = req.params.phoneNumber
    let code = util.getSmsCode()
    smsCodeCache[phoneNumber] = code

    //发送短信
    smsClient.sendSMS({
        PhoneNumbers: phoneNumber,
        SignName: smsSignName, //阿里云短信服务签名名称
        TemplateCode: smsTemplateCode, //阿里云短信服务	模版CODE
        TemplateParam: '{"code":"' + code + '"}'
    }).then(function(res) {
        let { Code } = res
        if (Code === 'OK') {
            res.success('发送成功')
        } else {
            res.error(res)
        }
    }, function(err) {
        res.error(err)
    })
})

//使用短息验证码注册或登录系统，用户不存在时自动注册。
/**
 *@params phoneNumber:string
 *@params code:string
 */
Parse.Cloud.define('signUpOrlogInWithMobilePhone', function(req, res) {
    let phoneNumber = req.params.phoneNumber,
        code = req.params.code
    if (!phoneNumber || !code) {
        return res.error('手机号码或验证码不能为空')
    }
    if (!smsCodeCache[phoneNumber]) {
        return res.error('未获取验证码')
    }
    if (smsCodeCache[phoneNumber] === code) {

		//这里写上类似leancloud验证成功后那部分代码的操作

    } else {
        res.error('验证失败')
    }
})


```

本方法十分安全，不必有安全顾虑。

来看看我以前个人业余项目的使用效果：

![](/img/GIF_Parse_login_with_phone_number.gif)

教程结束，不谢。