title: '使用RNN生成文本 [TF2.0] [译]'
author: 朱嘉伟
tags:
  - AI
  - TensorFlow
categories:
  - 成长ing
date: 2019-07-23 19:55:00
---

>  原著地址:https://www.tensorflow.org/beta/tutorials/text/text_generation

本教程将演示如何使用RNN(Recurrent nerual networks, 既回归神经网络)生成文本，我们将使用到Shakespeare(莎士比亚)的作品来训练模型(数据集来自[The Unreasonable Effectiveness of Recurrent Neural Networks](http://karpathy.github.io/2015/05/21/rnn-effectiveness/ "The Unreasonable Effectiveness of Recurrent Neural Networks")，实现传入字符串“Shakespear”给预测函数，其能够预测下一个字符“e”。通过重复调用预测函数，生成长文本。

> Note: Enable GPU acceleration to execute this notebook faster. In Colab: Runtime > Change runtime type > Hardware acclerator > GPU. If running locally make sure TensorFlow version >= 1.11.


本教程包含源代码，主要使用了[tf.keras](https://www.tensorflow.org/programmers_guide/keras "tf.keras")和[eager execution](https://www.tensorflow.org/programmers_guide/eager "eager execution")。下面例子，便是用本教程中的模型训练了30个epochs后，通过输入字母“Q”生成的文本：

> QUEENE:
I had thought thou hadst a Roman; for the oracle,
Thus by All bids the man against the word,
Which are so weak of care, by old care done;
Your children were in your holy love,
And the precipitation through the bleeding throne.
BISHOP OF ELY:
Marry, and will, my lord, to weep in such a one were prettiest;
Yet now I was adopted heir
Of the world's lamentable day,
To watch the next way with his father with his face?
ESCALUS:
The cause why then we are all resolved more sons.
VOLUMNIA:
O, no, no, no, no, no, no, no, no, no, no, no, no, no, no, no, no, no, no, no, no, it is no sin it should be dead,
And love and pale as any will to that word.
QUEEN ELIZABETH:
But how long have I heard the soul for this world,
And show his hands of life be proved to stand.
PETRUCHIO:
I say he look'd on, if I must be content
To stay him from the fatal of our country's bliss.
His lordship pluck'd from this sentence then for prey,
And then let us twain, being the moon,
were she such a case as fills m

有一些句子符合语法，但大多数都是无意义的。看来模型还没有学会字词的含义，但我们要知道：
- 模型是基于字符的，训练时，模型并不知道怎么去拼写单词。
- 输出结果的结构类似于剧本，通常以已名字开始，还有其中所有的大写字母都是来自于数据集中的。
- 正如下面所演示的，模型只用了很小的batch(~~100 characters each~~)来训练，但依然能够生成这么长的，结构一致的文本。

## 安装

### 导入TensorFlow和其他库

```python
from __future__ import absolute_import, division, print_function, unicode_literals

!pip install -q tensorflow-gpu==2.0.0-beta1
import tensorflow as tf

import numpy as np
import os
import time
```
### 下载莎士比亚数据集

```python
path_to_file = tf.keras.utils.get_file('shakespeare.txt', 'https://storage.googleapis.com/download.tensorflow.org/data/shakespeare.txt')
```

### 读入数据

先来看看这个文本：
```python
# Read, then decode for py2 compat.
text = open(path_to_file, 'rb').read().decode(encoding='utf-8')
# length of text is the number of characters in it
print ('Length of text: {} characters'.format(len(text)))
```

```
Length of text: 1115394 characters
```

```python
# Take a look at the first 250 characters in text
print(text[:250])
```

```
First Citizen:
Before we proceed any further, hear me speak.  

All:
Speak, speak.

First Citizen:
You are all resolved rather to die than to famish?

All:
Resolved. resolved.

First Citizen:
First, you know Caius Marcius is chief enemy to the people.

```

```python
# The unique characters in the file
vocab = sorted(set(text))
print ('{} unique characters'.format(len(vocab)))
```

```
65 unique characters
```

## 处理文本

### 文本向量化

在开始训练前，我们需要把字符替代成数字表示，即文本向量化。为了方便查找，我们创建两个映射表，一个通过字符得到数字，另一个通过数字得到字符。
```python
# Creating a mapping from unique characters to indices
char2idx = {u:i for i, u in enumerate(vocab)}
idx2char = np.array(vocab)

text_as_int = np.array([char2idx[c] for c in text])
```
现在我们可以通过数字来表示字符了。注意，我们的字符索引是从0开始的。
```python
print('{')
for char,_ in zip(char2idx, range(20)):
    print('  {:4s}: {:3d},'.format(repr(char), char2idx[char]))
print('  ...\n}')
```

```
{
  'X' :  36,
  'i' :  47,
  'Y' :  37,
  'r' :  56,
  '$' :   3,
  'o' :  53,
  'j' :  48,
  'u' :  59,
  'v' :  60,
  'A' :  13,
  'G' :  19,
  'l' :  50,
  'B' :  14,
  'y' :  63,
  'E' :  17,
  ':' :  10,
  '\n':   0,
  'w' :  61,
  'F' :  18,
  'h' :  46,
  ...
}
```
```python
# Show how the first 13 characters from the text are mapped to integers
print ('{} ---- characters mapped to int ---- > {}'.format(repr(text[:13]), text_as_int[:13]))
```

```
'First Citizen' ---- characters mapped to int ---- > [18 47 56 57 58  1 15 47 58 47 64 43 52]
```

### 预测任务
给定一个或一串字符，下一个字符最有可能是什么？这就是我们训练模型去完成的任务。
既然RNN内部状态依赖于之前看到的文本，我们将预测结果再输入进去，会得到什么字符？

### 创建样本序列和目标序列
我们把文本数据分割成多个样本序列，每一个输入序列有`seq_length`个来自文本数据中的字符，对应的目标序列也有相同长度的字符（除了被shift到数组最右边的序列）。
举个例子，假设`seq_length`为4，我们的文本数据为“Hello”，那么样本序列为“Hell”，目标序列为“ello”。
我们可以使用[tf.data.Dataset.from_tensor_slices](https://www.tensorflow.org/versions/r2.0/api_docs/python/tf/data/Dataset#from_tensor_slices "tf.data.Dataset.from_tensor_slices")把文本向量转换成字符索引。
```python
# The maximum length sentence we want for a single input in characters
seq_length = 100
examples_per_epoch = len(text)//seq_length

# Create training examples / targets
char_dataset = tf.data.Dataset.from_tensor_slices(text_as_int)

for i in char_dataset.take(5):
  print(idx2char[i.numpy()])
```

```
F
i
r
s
t
```

`batch`方法可以让我们轻松的把字符集转换成指定长度的序列。
```python
sequences = char_dataset.batch(seq_length+1, drop_remainder=True)

for item in sequences.take(5):
  print(repr(''.join(idx2char[item.numpy()])))
```

```
'First Citizen:\nBefore we proceed any further, hear me speak.\n\nAll:\nSpeak, speak.\n\nFirst Citizen:\nYou '
'are all resolved rather to die than to famish?\n\nAll:\nResolved. resolved.\n\nFirst Citizen:\nFirst, you k'
"now Caius Marcius is chief enemy to the people.\n\nAll:\nWe know't, we know't.\n\nFirst Citizen:\nLet us ki"
"ll him, and we'll have corn at our own price.\nIs't a verdict?\n\nAll:\nNo more talking on't; let it be d"
'one: away, away!\n\nSecond Citizen:\nOne word, good citizens.\n\nFirst Citizen:\nWe are accounted poor citi'
```

我们创建一个函数，将每一个序列都映射成输入序列和目标序列组成的样本：
```python
def split_input_target(chunk):
    input_text = chunk[:-1]
    target_text = chunk[1:]
    return input_text, target_text

dataset = sequences.map(split_input_target)
```

打印第一个样本的输入序列和目标序列：
```python
for input_example, target_example in  dataset.take(1):
  print ('Input data: ', repr(''.join(idx2char[input_example.numpy()])))
  print ('Target data:', repr(''.join(idx2char[target_example.numpy()])))
```

```
Input data:  'First Citizen:\nBefore we proceed any further, hear me speak.\n\nAll:\nSpeak, speak.\n\nFirst Citizen:\nYou'
Target data: 'irst Citizen:\nBefore we proceed any further, hear me speak.\n\nAll:\nSpeak, speak.\n\nFirst Citizen:\nYou '
```

每一个向量索引都被作为一个时间步处理，比如在时间步0，模型接收到“F”，并尝试预测下一个字符为“i”。在下一个时间步，模型接收到到“i”，又开始尝试预测下一个字符，只不过这次还会额外考虑之前时间步的上下文。
```
for i, (input_idx, target_idx) in enumerate(zip(input_example[:5], target_example[:5])):
    print("Step {:4d}".format(i))
    print("  input: {} ({:s})".format(input_idx, repr(idx2char[input_idx])))
    print("  expected output: {} ({:s})".format(target_idx, repr(idx2char[target_idx])))
```

```
Step    0
  input: 18 ('F')
  expected output: 47 ('i')
Step    1
  input: 47 ('i')
  expected output: 56 ('r')
Step    2
  input: 56 ('r')
  expected output: 57 ('s')
Step    3
  input: 57 ('s')
  expected output: 58 ('t')
Step    4
  input: 58 ('t')
  expected output: 1 (' ')
```

### 创建训练

我们使用了`tf.data`把文本分割成了可管理的序列，在我们开始训练前，我们还需要打乱样本，以及打包成batch。
```python
# Batch size
BATCH_SIZE = 64

# Buffer size to shuffle the dataset
# (TF data is designed to work with possibly infinite sequences,
# so it doesn't attempt to shuffle the entire sequence in memory. Instead,
# it maintains a buffer in which it shuffles elements).
BUFFER_SIZE = 10000

dataset = dataset.shuffle(BUFFER_SIZE).batch(BATCH_SIZE, drop_remainder=True)
```

```
<BatchDataset shapes: ((64, 100), (64, 100)), types: (tf.int64, tf.int64)>
```

## 构建模型
在这个例子中，我们用了3层网络定义了一个[tf.keras.Sequential](https://www.tensorflow.org/versions/r2.0/api_docs/python/tf/keras/Sequential "tf.keras.Sequential")模型：
- [tf.keras.layers.Embedding](https://www.tensorflow.org/versions/r2.0/api_docs/python/tf/keras/layers/Embedding "tf.keras.layers.Embedding"): 输入层，一个可训练的查询表，它会把每一个字符索引映射成具有`embedding_dim`个维度的向量。
- [tf.keras.layers.GRU](https://www.tensorflow.org/versions/r2.0/api_docs/python/tf/keras/layers/GRU "tf.keras.layers.GRU") RNN的其中一种，你也可以使用改成LSTM层。
- [tf.keras.layers.Dense](https://www.tensorflow.org/versions/r2.0/api_docs/python/tf/keras/layers/Dense "tf.keras.layers.Dense") 输出层。

```python
# Length of the vocabulary in chars
vocab_size = len(vocab)

# The embedding dimension
embedding_dim = 256

# Number of RNN units
rnn_units = 1024
```

```python
def build_model(vocab_size, embedding_dim, rnn_units, batch_size):
  model = tf.keras.Sequential([
    tf.keras.layers.Embedding(vocab_size, embedding_dim,
                              batch_input_shape=[batch_size, None]),
    tf.keras.layers.LSTM(rnn_units,
                        return_sequences=True,
                        stateful=True,
                        recurrent_initializer='glorot_uniform'),
    tf.keras.layers.Dense(vocab_size)
  ])
  return model
```

```python
model = build_model(
  vocab_size = len(vocab),
  embedding_dim=embedding_dim,
  rnn_units=rnn_units,
  batch_size=BATCH_SIZE)
```

## 测试模型
现在我们来检查下模型的行为。
首先检查输出的shape：
```python
for input_example_batch, target_example_batch in dataset.take(1):
  example_batch_predictions = model(input_example_batch)
  print(example_batch_predictions.shape, "# (batch_size, sequence_length, vocab_size)")
```
```
(64, 100, 65) # (batch_size, sequence_length, vocab_size)
```
从上面我们可以看到，输入样本的序列长度为100，而模型可以在任意序列长度上运行。

```python
model.summary()
```

```
Model: "sequential"
_________________________________________________________________
Layer (type)                 Output Shape              Param #   
=================================================================
embedding (Embedding)        (64, None, 256)           16640     
_________________________________________________________________
lstm (LSTM)                  (64, None, 1024)          5246976   
_________________________________________________________________
dense (Dense)                (64, None, 65)            66625     
=================================================================
Total params: 5,330,241
Trainable params: 5,330,241
Non-trainable params: 0
_________________________________________________________________
```

要从模型中获得实际预测，我们需要从输出分布中采样，得到实际的字符索引。此分布由字符集上的logits定义。

> Note: It is important to sample from this distribution as taking the argmax of the distribution can easily get the model stuck in a loop.

用第一个batch中的样本测试：
```python
sampled_indices = tf.random.categorical(example_batch_predictions[0], num_samples=1)
sampled_indices = tf.squeeze(sampled_indices,axis=-1).numpy()
```
在每一个时间步，这会给我们下一个字符索引的预测：
```
sampled_indices
```

```
array([61, 64,  9, 63, 27, 56,  1, 12, 50, 48, 26, 11, 14, 22, 25, 24, 43,
       45, 48,  7, 59, 52, 25, 19, 63, 18, 40, 10, 61, 35, 39, 24, 26, 53,
       64, 64, 32, 23, 27,  9, 12, 58, 46, 49,  5, 23, 18, 58, 40, 11, 16,
       35, 54, 31, 24, 39, 21, 29, 22, 28, 18, 10, 57,  2, 41, 42, 25, 29,
       23, 19, 26, 55, 56,  1, 61, 17, 62, 33, 17, 47, 17, 58, 18, 38, 35,
       17, 31,  4, 19, 39,  9, 50, 47, 33, 15, 28, 24,  4, 26, 58])
```

将字符索引转换成字符：
```python
print("Input: \n", repr("".join(idx2char[input_example_batch[0]])))
print()
print("Next Char Predictions: \n", repr("".join(idx2char[sampled_indices ])))
```

```
Input: 
 "ce the earthquake now eleven years;\nAnd she was wean'd,--I never shall forget it,--\nOf all the days "

Next Char Predictions: 
 "wz3yOr ?ljN;BJMLegj-unMGyFb:wWaLNozzTKO3?thk'KFtb;DWpSLaIQJPF:s!cdMQKGNqr wExUEiEtFZWES&Ga3liUCPL&Nt"
```

## 训练模型
现在，问题可已堪称是标准分类的问题。给定之前的RNN状态，和当前时间步的输入序列，预测下一个字符的类型。

### 添加优化器和损失函数
损失函数[ tf.keras.losses.sparse_categorical_crossentropy ](https://www.tensorflow.org/versions/r2.0/api_docs/python/tf/keras/losses/sparse_categorical_crossentropy " tf.keras.losses.sparse_categorical_crossentropy ")适用于当前问题，因为它它经过预测的最后一个维度应用。
因为我们的模型返回logits，我们需要设置`from_logits`变量。
```python
def loss(labels, logits):
  return tf.keras.losses.sparse_categorical_crossentropy(labels, logits, from_logits=True)

example_batch_loss  = loss(target_example_batch, example_batch_predictions)
print("Prediction shape: ", example_batch_predictions.shape, " # (batch_size, sequence_length, vocab_size)")
print("scalar_loss:      ", example_batch_loss.numpy().mean())
```

```
Prediction shape:  (64, 100, 65)  # (batch_size, sequence_length, vocab_size)
scalar_loss:       4.1740375
```

[tf.keras.Model.compile](https://www.tensorflow.org/versions/r2.0/api_docs/python/tf/keras/Model#compile "tf.keras.Model.compile")方法配置训练过程，优化器使用[tf.keras.optimizers.Adam ](https://www.tensorflow.org/versions/r2.0/api_docs/python/tf/keras/optimizers/Adam "tf.keras.optimizers.Adam ")。

```python
model.compile(optimizer='adam', loss=loss)
```

### 配置存档
使用[tf.keras.callbacks.ModelCheckpoint ](https://www.tensorflow.org/versions/r2.0/api_docs/python/tf/keras/callbacks/ModelCheckpoint "tf.keras.callbacks.ModelCheckpoint ")可以在训练过程中进行存档：

```python
# Directory where the checkpoints will be saved
checkpoint_dir = './training_checkpoints'
# Name of the checkpoint files
checkpoint_prefix = os.path.join(checkpoint_dir, "ckpt_{epoch}")

checkpoint_callback=tf.keras.callbacks.ModelCheckpoint(
    filepath=checkpoint_prefix,
    save_weights_only=True)
```

### 开始训练

我们使用10个epochs来训练模型：
```python
EPOCHS=10
```

```python
history = model.fit(dataset, epochs=EPOCHS, callbacks=[checkpoint_callback])
```

```
Epoch 1/10
172/172 [==============================] - 8s 46ms/step - loss: 2.5909
Epoch 2/10
172/172 [==============================] - 7s 40ms/step - loss: 1.8915
Epoch 3/10
172/172 [==============================] - 7s 41ms/step - loss: 1.6430
Epoch 4/10
172/172 [==============================] - 7s 39ms/step - loss: 1.5057
Epoch 5/10
172/172 [==============================] - 7s 39ms/step - loss: 1.4211
Epoch 6/10
172/172 [==============================] - 7s 39ms/step - loss: 1.3591
Epoch 7/10
172/172 [==============================] - 7s 39ms/step - loss: 1.3077
Epoch 8/10
172/172 [==============================] - 7s 40ms/step - loss: 1.2614
Epoch 9/10
172/172 [==============================] - 7s 40ms/step - loss: 1.2171
Epoch 10/10
172/172 [==============================] - 7s 39ms/step - loss: 1.1737
```

## 生成文本

### 恢复上次存档

为了保持预测步骤简洁，`batch_size`设置为1。
由于RNN状态是使用的时间步到时间步的传递方法，一旦训练完成，模型值接收固定的`batch_size`。
要让模型接收不同的`batch_size`，我们需要重新训练模型，并从存档恢复数据。
```python
tf.train.latest_checkpoint(checkpoint_dir)
```

```
'./training_checkpoints/ckpt_10'
```

```python
model = build_model(vocab_size, embedding_dim, rnn_units, batch_size=1)

model.load_weights(tf.train.latest_checkpoint(checkpoint_dir))

model.build(tf.TensorShape([1, None]))
```

```python
model.summary()
```

```
Model: "sequential_1"
_________________________________________________________________
Layer (type)                 Output Shape              Param #   
=================================================================
embedding_1 (Embedding)      (1, None, 256)            16640     
_________________________________________________________________
lstm_1 (LSTM)                (1, None, 1024)           5246976   
_________________________________________________________________
dense_1 (Dense)              (1, None, 65)             66625     
=================================================================
Total params: 5,330,241
Trainable params: 5,330,241
Non-trainable params: 0
_________________________________________________________________
```

### 循环预测

下面代码生成这样的文本：
- 它选择一个开头的字符串，初始化RNN状态，并设置要生成的字符数量，然后开始。
- 根据开头的字符串和RNN状态，得到下一个字符的预测分布。
- 然后，用分类分布来计算预测的字符索引，并将其作为我们下一次训练的输入序列。
- 模型返回的RNN状态被反馈到了模型中，所以它有了更多的上下文信息，而不再是一个字符。在预测下一个字符时，更新后的RNN状态又被反馈到了模型中。这就是它的学习方式。

观察生成的文本，我们可以看出模型知道何时大写、分段、模仿莎士比亚风格的词汇。使用这么小的epoch，它没有学会生成连贯的句子。

```python
def generate_text(model, start_string):
  # Evaluation step (generating text using the learned model)

  # Number of characters to generate
  num_generate = 1000

  # Converting our start string to numbers (vectorizing)
  input_eval = [char2idx[s] for s in start_string]
  input_eval = tf.expand_dims(input_eval, 0)

  # Empty string to store our results
  text_generated = []

  # Low temperatures results in more predictable text.
  # Higher temperatures results in more surprising text.
  # Experiment to find the best setting.
  temperature = 1.0

  # Here batch size == 1
  model.reset_states()
  for i in range(num_generate):
      predictions = model(input_eval)
      # remove the batch dimension
      predictions = tf.squeeze(predictions, 0)

      # using a categorical distribution to predict the word returned by the model
      predictions = predictions / temperature
      predicted_id = tf.random.categorical(predictions, num_samples=1)[-1,0].numpy()

      # We pass the predicted word as the next input to the model
      # along with the previous hidden state
      input_eval = tf.expand_dims([predicted_id], 0)

      text_generated.append(idx2char[predicted_id])

  return (start_string + ''.join(text_generated))
```

```
print(generate_text(model, start_string=u"ROMEO: "))
```

```
ROMEO: vend me die,
That I your fiends are not believed too:
Fear our mercy mouth joys, that the Encouse bed,
Thrans, you would be were back with yore the nothing,
Without his bonds.

KING RICHARD III:
O, that we fell,
S dear drught's come, THAMESSE:
Hence, heavens!

LEONTES:
Goddeven,
In Oxford, kill thou, much reason substatite
Andake' the poor of her eyes king from leave to big threet that far
fear, or extremition with them.

First Servant:
The con is called this fitch dight certre for this
York doth thou shadefully openslual friends: these perfect-face
To pass me with a good destinity;
Nor never proud heart, the grass seal i' the eye,
And make no Romans, yet not nd minK:
It wise, come, trodgen, speak in relord;
For he comes Clarence' is gone, begin his true here patience;
O, so hath two-solte ouchs it mother
Than in thy company this reversion and
The longer man, e'er who beneved your formmer, as I have it dead,
A' the is profes, our generation, leage to not:
He was the duke deceifl to rea
```

要提高生成质量最简单的方法，就是训练的更久（尝试EPOCHS=30）。

你也可以体验用不同的开头字符来生成文本，或尝试其他RNN层来提高模型的精度，或调整`temprature`参数来生成更多或更少的预测。

## 进阶：自定义训练

以上的训练过程很简单，但没有给你太多控制。
现在你已经知道如何手动运行模型，现在让我们解构循环，自己来实现它。比如，实现*curriculum learning*来帮助使模型的输出更加稳定。
我们将使用[tf.GradientTape](https://www.tensorflow.org/versions/r2.0/api_docs/python/tf/GradientTape "tf.GradientTape")来跟踪梯度，你可以阅读[eager execution guide](https://www.tensorflow.org/guide/eager "eager execution guide")深入了解。

程序原理：
- 首先，通过[ tf.keras.Model.reset_states](https://www.tensorflow.org/versions/r2.0/api_docs/python/tf/keras/Model#reset_states " tf.keras.Model.reset_states")初始化RNN状态。
- 然后，逐批遍历数据集，并计算每次的预测结果。
- 打开[tf.GradientTape](https://www.tensorflow.org/versions/r2.0/api_docs/python/tf/GradientTape "tf.GradientTape")，计算上下文中的预测结果和损失函数。
- 用`tf.GradientTape.grads`计算相对模型变量的梯度和损失值。
- 最后，用优化器的`tf.train.Optimizer.apply_gradients`方法向下一步。

```python
model = build_model(
  vocab_size = len(vocab),
  embedding_dim=embedding_dim,
  rnn_units=rnn_units,
  batch_size=BATCH_SIZE)
```

```python
optimizer = tf.keras.optimizers.Adam()
```

```python
@tf.function
def train_step(inp, target):
  with tf.GradientTape() as tape:
    predictions = model(inp)
    loss = tf.reduce_mean(
        tf.keras.losses.sparse_categorical_crossentropy(
            target, predictions, from_logits=True))
  grads = tape.gradient(loss, model.trainable_variables)
  optimizer.apply_gradients(zip(grads, model.trainable_variables))

  return loss
```

```python
# Training step
EPOCHS = 10

for epoch in range(EPOCHS):
  start = time.time()

  # initializing the hidden state at the start of every epoch
  # initally hidden is None
  hidden = model.reset_states()

  for (batch_n, (inp, target)) in enumerate(dataset):
    loss = train_step(inp, target)

    if batch_n % 100 == 0:
      template = 'Epoch {} Batch {} Loss {}'
      print(template.format(epoch+1, batch_n, loss))

  # saving (checkpoint) the model every 5 epochs
  if (epoch + 1) % 5 == 0:
    model.save_weights(checkpoint_prefix.format(epoch=epoch))

  print ('Epoch {} Loss {:.4f}'.format(epoch+1, loss))
  print ('Time taken for 1 epoch {} sec\n'.format(time.time() - start))

model.save_weights(checkpoint_prefix.format(epoch=epoch))
```

```
WARNING: Logging before flag parsing goes to stderr.
W0711 02:52:32.027381 140346967226112 util.py:244] Unresolved object in checkpoint: (root).optimizer
W0711 02:52:32.028622 140346967226112 util.py:244] Unresolved object in checkpoint: (root).optimizer.iter
W0711 02:52:32.029642 140346967226112 util.py:244] Unresolved object in checkpoint: (root).optimizer.beta_1
W0711 02:52:32.030770 140346967226112 util.py:244] Unresolved object in checkpoint: (root).optimizer.beta_2
W0711 02:52:32.031696 140346967226112 util.py:244] Unresolved object in checkpoint: (root).optimizer.decay
W0711 02:52:32.032530 140346967226112 util.py:244] Unresolved object in checkpoint: (root).optimizer.learning_rate
W0711 02:52:32.033109 140346967226112 util.py:244] Unresolved object in checkpoint: (root).optimizer's state 'm' for (root).layer_with_weights-0.embeddings
W0711 02:52:32.033605 140346967226112 util.py:244] Unresolved object in checkpoint: (root).optimizer's state 'm' for (root).layer_with_weights-2.kernel
W0711 02:52:32.034740 140346967226112 util.py:244] Unresolved object in checkpoint: (root).optimizer's state 'm' for (root).layer_with_weights-2.bias
W0711 02:52:32.035699 140346967226112 util.py:244] Unresolved object in checkpoint: (root).optimizer's state 'm' for (root).layer_with_weights-1.cell.kernel
W0711 02:52:32.036309 140346967226112 util.py:244] Unresolved object in checkpoint: (root).optimizer's state 'm' for (root).layer_with_weights-1.cell.recurrent_kernel
W0711 02:52:32.036845 140346967226112 util.py:244] Unresolved object in checkpoint: (root).optimizer's state 'm' for (root).layer_with_weights-1.cell.bias
W0711 02:52:32.037351 140346967226112 util.py:244] Unresolved object in checkpoint: (root).optimizer's state 'v' for (root).layer_with_weights-0.embeddings
W0711 02:52:32.037847 140346967226112 util.py:244] Unresolved object in checkpoint: (root).optimizer's state 'v' for (root).layer_with_weights-2.kernel
W0711 02:52:32.038431 140346967226112 util.py:244] Unresolved object in checkpoint: (root).optimizer's state 'v' for (root).layer_with_weights-2.bias
W0711 02:52:32.039076 140346967226112 util.py:244] Unresolved object in checkpoint: (root).optimizer's state 'v' for (root).layer_with_weights-1.cell.kernel
W0711 02:52:32.039675 140346967226112 util.py:244] Unresolved object in checkpoint: (root).optimizer's state 'v' for (root).layer_with_weights-1.cell.recurrent_kernel
W0711 02:52:32.040273 140346967226112 util.py:244] Unresolved object in checkpoint: (root).optimizer's state 'v' for (root).layer_with_weights-1.cell.bias
W0711 02:52:32.040812 140346967226112 util.py:252] A checkpoint was restored (e.g. tf.train.Checkpoint.restore or tf.keras.Model.load_weights) but not all checkpointed values were used. See above for specific issues. Use expect_partial() on the load status object, e.g. tf.train.Checkpoint.restore(...).expect_partial(), to silence these warnings, or use assert_consumed() to make the check explicit. See https://www.tensorflow.org/alpha/guide/checkpoints#loading_mechanics for details.

Epoch 1 Batch 0 Loss 4.174699306488037
Epoch 1 Batch 100 Loss 2.333345890045166
Epoch 1 Loss 2.1393
Time taken for 1 epoch 9.043232679367065 sec

Epoch 2 Batch 0 Loss 2.170747995376587
Epoch 2 Batch 100 Loss 1.8737974166870117
Epoch 2 Loss 1.7977
Time taken for 1 epoch 6.3676910400390625 sec

Epoch 3 Batch 0 Loss 1.6878454685211182
Epoch 3 Batch 100 Loss 1.6373414993286133
Epoch 3 Loss 1.6188
Time taken for 1 epoch 6.33354115486145 sec

Epoch 4 Batch 0 Loss 1.5183608531951904
Epoch 4 Batch 100 Loss 1.5071923732757568
Epoch 4 Loss 1.5104
Time taken for 1 epoch 6.215446472167969 sec

Epoch 5 Batch 0 Loss 1.4184167385101318
Epoch 5 Batch 100 Loss 1.4265903234481812
Epoch 5 Loss 1.4379
Time taken for 1 epoch 6.28309440612793 sec

Epoch 6 Batch 0 Loss 1.3484241962432861
Epoch 6 Batch 100 Loss 1.3697052001953125
Epoch 6 Loss 1.3760
Time taken for 1 epoch 6.283656597137451 sec

Epoch 7 Batch 0 Loss 1.2933669090270996
Epoch 7 Batch 100 Loss 1.3211030960083008
Epoch 7 Loss 1.3210
Time taken for 1 epoch 6.146169424057007 sec

Epoch 8 Batch 0 Loss 1.2436736822128296
Epoch 8 Batch 100 Loss 1.2792930603027344
Epoch 8 Loss 1.2719
Time taken for 1 epoch 6.354868650436401 sec

Epoch 9 Batch 0 Loss 1.1980918645858765
Epoch 9 Batch 100 Loss 1.2407312393188477
Epoch 9 Loss 1.2255
Time taken for 1 epoch 6.278747797012329 sec

Epoch 10 Batch 0 Loss 1.1543728113174438
Epoch 10 Batch 100 Loss 1.194383144378662
Epoch 10 Loss 1.1759
Time taken for 1 epoch 7.029285907745361 sec
```









