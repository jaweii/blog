title: TensorFlow.js · 起手式
author: 朱嘉伟
tags:
  - TensorFlow
  - AI
categories:
  - WEB向
date: 2018-07-26 15:22:00
---
所谓机器学习，就是将已知对应结果的`输入数据`和`对应结果` 提交到TensorFlow模型中，以训练模型。**这一步就是机器学习的过程**。

训练完成后，我们传入尚不知晓结果的`输入数据` 给模型，模型就能预言结果是什么。**这一步就是AI的应用**。

举个例子，现有澳门赌场性感荷官在线投骰子，已经投了4次：

第一次的结果是1点，

第二次的结果是3点，

第三次的结果是5点，

第四次的结果是7点。

最多几个点不是重点。

重点来了，**让模型能预测第五次是几点**。

你可能一眼就看出来了规律，甚至提炼出了可套用的公式\(函数\)：结果 = 2n-1。

但是如果写套用公式的程序，那就不是编写AI了。

事实上，我们对模型的训练，就是为了让模型推导出那个公式，而不是让程序员推导出，然后写死。

我们对模型调教的越多，模型推导出的公式就越趋近于正确。

写到这里，可以总结一点：**对模型的训练，就是让模型从数据中找出公式\(函数\)**。

有了这个函数，就能输入数据，返回结果。

# CODING

现在我们以投骰子的例子，来写出相关代码。

首先全局[安装Parcel](https://parceljs.org/getting_started.html)。

`npm init`一个项目。

然后安装TensorFlow：

```
npm install @tensorflow/tfjs --save
```

新建index.html和1.js文件，并在index.html中引入1.js。

运行`parcel index.html `启动项目。

下面开始在1.js中编写演示代码了:

```
//引入TensorFlow
import * as tf from '@tensorflow/tfjs'

// 定义线性衰退模型
const model = tf.sequential()
// add方法添加一个图层实例
// tf.layers.dense 创建一个输入输出维度为1的层
model.add(tf.layers.dense({ units: 1, inputShape: [1] }))

// 指定损失函数和优化器
model.compile({ loss: 'meanSquaredError', optimizer: 'sgd' })

// 模拟一些数据用以训练
// xs代表投骰子的第几次
// tensor2d方法定义一个4行1列的二维张量
// xs代表投骰子的第几次
const xs = tf.tensor2d([1, 2, 3, 4], [4, 1])
// ys代表每一次骰子的结果
const ys = tf.tensor2d([1, 3, 5, 7], [4, 1])

// 训练模型
async function train() {
  // for循环增加训练次数，训练的越多，预测的越准
  for (let index = 0; index < 30; index++) {
    // fit方法训练模型，epochs表示迭代训练数组的次数
    await model.fit(xs, ys, { epochs: 10 })
  }
  // 训练完成，现在我们要预测第5次是几点
  // predict方法用以预测，相当于调用机器推导出的公式，传入一个b表示第5次的1行1列张量
  // print方法打印结果到控制台
  model.predict(tf.tensor2d([5], [1, 1])).print()
}

train()


```

OK！

在浏览器控制台我们就能看到以上代码的执行结果了：

```
Tensor
[[8.7086086],]
```

预测结果为8.7086086，和正确结果9相差0.3左右。

如果我们调整代码，增加10倍训练次数：

```
for (let index = 0; index < 300; index++)
```

再来看看结果：

```
Tensor
[[8.9998331],]
```

已经很接近9了！



到这里，我们已经接触到了AI的一些皮毛了。

本文是我学习TensorFlow的第一个例子后写出，作为提炼总结，和笔记分享。



