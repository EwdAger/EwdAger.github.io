---
title: LSTM原理及Keras中实现
tags:
  - Python
  - 深度学习
  - LSTM
  - Keras
abbrlink: e4e448be
categories: 深度学习笔记
date: 2019-12-07 11:03:00
cover: https://gvoidy-1251878576.cos.ap-chengdu.myqcloud.com/LSTM.jpg
---

# LSTM 原理


LSTM(Long Short-Term Memory) 即长短期记忆，适合于处理和预测时间序列中间隔和延迟非常长的重要事件。其中的内部机制就是通过四个门调节信息流，了解序列中哪些数据需要保留或丢弃。

![Long Short-Term Memory](https://pic4.zhimg.com/80/v2-e4f9851cad426dfe4ab1c76209546827_hd.jpg)

## 通俗的原理

假设你在网上查看淘宝评论，以确定你是否想购买生活物品。你将首先阅读评论，然后确定是否有人认为它是好的或是否是坏的。

![某麦片评论](https://miro.medium.com/max/1664/1*YHjfAgozQaghcsEvsBEu2g.png)

当你阅读评论时，你的大脑下意识地只会记住重要的关键词。你会选择“惊人”和“完美均衡的早餐”这样的词汇。你不太关心“this”，“give”，“all”，“should”等。如果你的朋友第二天问你评论说什么，你不可能一字不漏地记住它。但你可能还记得主要观点，比如“肯定会再次购买”。其他的话就会从记忆中逐渐消失。

这基本上就是LSTM或GRU的作用。它可以学习只保留相关信息来进行预测，并忘记不相关的数据。在这种情况下，你记住的词让你判断它是好的。


## 核心概念

![LSTM原理](https://gvoidy-1251878576.cos.ap-chengdu.myqcloud.com/LSTM.jpg)

LSTM 的核心概念是细胞状态，三个门和两个激活函数。细胞状态充当高速公路，在序列链中传递相关信息。门是不同的神经网络，决定在细胞状态上允许那些信息。有些门可以了解在训练期间保持或忘记那些信息。

### 激活函数 Tanh

![Tanh squishes值介于-1和1之间](https://miro.medium.com/max/950/1*iRlEg1GBKRzGTre5aOQUCg.gif)

用于调节流经神经网络的值，限制在-1和1之间，防止梯度爆炸

![没有tanh函数的变换](https://miro.medium.com/max/950/1*LgbEFcGiUpseZ--M7wuZhg.gif)

![经过tanh函数的变换](https://miro.medium.com/max/950/1*gFC2bTg3uihp1klknWU0qg.gif)


### 激活函数 Sigmoid

![Sigmoid squishes值介于0和1之间](https://miro.medium.com/max/950/1*rOFozAke2DX5BmsX2ubovw.gif)

与激活函数 Tanh不同，他是在0和1之间取值。这有助于更新或忘记数据，因为任何数字乘以0都是0，这会导致值小时或被"遗忘"。而任何数字乘1都是相同的值。网络可以通过这种方法了解那些数据不重要或那些数据重要。


### 遗忘门

遗忘门决定应该丢弃或保留那些信息。来自先前隐藏状态的信息和来自当前输入的信息通过sigmoid函数传递。值接近0和1之间，越接近0意味着忘记，越接近1意味着要保持。

![遗忘门操作](https://miro.medium.com/max/950/1*GjehOa513_BgpDDP6Vkw2Q.gif)


### 输入门

输入门可以更新细胞状态，将先前的隐藏状态和当前输入分别传递sigmoid函数和tanh函数。然后将两个函数的输出相乘。

![输入门](https://miro.medium.com/max/950/1*TTmYy7Sy8uUXxUXfzmoKbA.gif)

### 细胞状态

细胞状态逐点乘以遗忘向量（遗忘门操作得到），然后与输入门获得的输出进行逐点相加，将神经网络发现的新值更新为细胞状态。

![细胞状态](https://miro.medium.com/max/950/1*S0rXIeO_VoUVOyrYHckUWg.gif)


### 输出门

输出门可以决定下一个隐藏状态应该是什么，并且可用于预测。首先将先前的隐藏状态和当前的输入传给sigmoid函数，然后将新修改的细胞状态传递给tanh函数，最后就结果相乘。输出的是隐藏状态，然后将新的细胞状态和新的隐藏状态移动到下一个时间序列中。

![输出门](https://miro.medium.com/max/950/1*VOXRGhOShoWWks6ouoDN3Q.gif)


## 数学描述

从上述图解操作，我们可以轻松的理解LSTM的数学描述。

![数学描述](https://pic2.zhimg.com/80/v2-556c74f0e025a47fea05dc0f76ea775d_hd.jpg)

1. $c^t = z^f \odot c^{t-1} + z^i \odot z$

	其中$c^t$为当前细胞状态，$z^f$为遗忘门，$z^i$和$z$为输入门中两个操作。表示LSTM的**遗忘阶段**，对上一节点传进来的输入进行选择性忘记。

2. $h^t = z^o \odot tanh (c^t)$

	其中$h^t$表示当前隐藏状态，$z^o$表示输出门中前一操作。表示LSTM的**选择记忆阶段**，对输入的$x^t$进行选择记忆。哪些重要则着重记录下来，哪些不重要，则少记一些。

3. $y^t = \sigma (W^ \prime h^t)$

	表示LSTM的**输出阶段**，通过当前隐藏状态$h^t$一些变化得到。


# Keras 中 LSTM 的实现

## 加载依赖库

```python
from keras.models import Sequential
from keras.layers.core import Dense, Activation, Dropout
from keras.layers.recurrent import LSTM
```
- `models` 是 Keras 神经网络的核心。这个对象代表这个我们所定义的神经网络：它有层、激活函数等等属性和功能。我们进行训练和测试也是基于这个`models`。 `Sequetial` 表示我们将使用层堆叠起来的网络，这是Keras中的基本网络结构。
- `Dense, Activation, Dropout` 这些是神经网络里面的核心层，用于构建整个神经网络。`Dense` 实际上就是 Fully-connected 层；`Activation`是激活函数，它会通过Relu, Softmax 等函数对上一层产生的结果进行修改；当神经元过多的时候，可能效果并不好，因为容易导致过拟合的现象，`Dropout`是将上一层神经元进行随机丢弃，有助于解决过拟合的问题。
- `LSTM` 是经典的RNN神经网络层。

## 数据准备

因为 LSTM 是预测时间序列，即比如通过前19个数据去预测第20个数据。所有每次喂给LSTM的数据也必须是一个滑动窗口。

![滑动窗口](http://resuly.me/img/in_post/2017/08/Rnn_blog_sequence.png)

而这其中的19个数据就是我们训练集X的一个样本，第20个为训练集Y样本。也就是说，我们用前19个值，去预测第20个值，然后对比预测至与第20个的真实值。通过这样的误差不断的优化我们模型。

```python
for index in range(len(data) - sequence_length):
        result.append(data[index: index + sequence_length])
    result = np.array(result)
```

当然这是一维样本的情况，如果我们的数据如下。

x1|x2|x3|x4|y
--|--|--|--|--
0.363986149|0.333333333|0.713178295|0.724661696|191
0.411312043|0.333333333|0.666666667|0.731013532|190
0.357445171|0.166666667|0.627906977|0.621375311|189
0.166602539|0.333333333|0.573643411|0.662386081|188
0.402077722|0.416666667|0.589147287|0.704501519|187
0.330511735|0.25|0.651162791|0.652720243|186


那么滑动窗口将是一个矩阵形式。假设滑动窗口为3行，则训练集X的第一个样本是

x1|x2|x3|x4
--|--|--|--
0.363986149|0.333333333|0.713178295|0.724661696
0.411312043|0.333333333|0.666666667|0.731013532
0.357445171|0.166666667|0.627906977|0.621375311

去预测的是第4行的y。然后对比预测至与第4行y的真实值。通过这样的误差不断的优化我们模型。


## 创建模型

```python
model = Sequential()
model.add(LSTM(units=100, input_shape=(30, 40), return_sequences=True))
model.add(Dropout(0.2))

model.add(LSTM(units=50, return_sequences=False))
model.add(Dropout(0.2))

model.add(Dense(units=1))
model.add(Activation("linear"))
model.compile(loss="mse", optimizer="adam", metrics=['mae', 'mape'])
```

实例化模型后我们添加了两层隐含层（LSTM层）和一个输出期望（Dense层），激活函数设置为线性（linear)，其中每完成一层计算丢弃20%的数据（Dropout层）防止过拟合。最后我们使用 MSE 进行误差计算，优化函数选择 adam，评价指标为 mae 和 mape

## 参数介绍

### `return_sequences=True/False`

![return_sequence](http://resuly.me/img/in_post/2017/08/2f64696167732e6a706567.jpg)

`return_sequence=True`为多对多的关系（上图第5种情况），`return_sequence=False`为多对一的关系（上图第3种情况）


我们这里建立了两层LSTM，第一层因为我们希望把每一次的输出信息都输入到下一层LSTM作为训练数据（箭头向上），也同时也将信息传给下一个自己作为输入数据（箭头向右）。而第二层连接Dense层，只期望一个输出。所以第一层为多对多的关系，第二层为多对一的关系。

### `input_shape`

LSTM 的输入是一个三维数组，尽管他的`input_shape`为二维，但我们的输入必须也是(批次大小, 时间步长, 单元数)即每批次输入LSTM的样本数，时间步长，训练集的列数。

![输入演示](https://pic4.zhimg.com/50/v2-e9b2e0e4a6ba9bc48a6b660fd5b56f44_hd.webp)

`input_shape=(time_steps, input_dim)`只接受时间步长和单元数是因为可以自动切分批次大小，如果需要固定批次大小，可以通过`batch_input_shape=(batch_size，time_steps，input_dim)`参数实现。


### `units`

指设置的细胞单元数量，也可当做输出维度（因为在不考虑细胞状态输出的情况下，每一个细胞单元只有一个隐藏关系的输出）。比如，我们这里设置的输入为`input_shape=(30, 40)`,而我们设置的单元数为100，那么我们最终输出就是(100,4)了。

### `Dense`

`Dense`层接受上一层传递过来的输出数据，然后与激活函数结合真实值进行loss计算和优化等操作，设置的单元数`units`同上也可当做输出维度。

## 训练模型

```python
fit_result = model.fit(trainX, trainY, epochs=100, batch_size=200)
```
跟机器学习算法一样，也是输入训练集X、Y，`epochs`为循环训练次数，`batch_size`为单次训练的样本数。

## 预测结果

```python
predicted = model.predict(testX)
```
与训练模型时喂数据一致，输入一个testX数组，testX[0]为一个滑动窗口所有的样本，例如一维数组前19个，预测的结果是第20个。那么predicted[0]就为预测的第20个结果。

同理，predicted就返回了一个与testY一样大的数组。


# 参考文章
[Illustrated Guide to LSTM’s and GRU’s: A step by step explanation](https://towardsdatascience.com/illustrated-guide-to-lstms-and-gru-s-a-step-by-step-explanation-44e9eb85bf21)
[一文了解LSTM和GRU背后的秘密（绝对没有公式）](https://yq.aliyun.com/articles/647550?utm_content=m_1000017933)
[人人都能看懂的LSTM](https://zhuanlan.zhihu.com/p/32085405)
[使用Keras中的RNN模型进行时间序列预测](http://resuly.me/2017/08/16/keras-rnn-tutorial/)
[用「动图」和「举例子」讲讲 RNN](https://zhuanlan.zhihu.com/p/36455374)
[Understanding Input and Output shapes in LSTM | Keras](https://medium.com/@shivajbd/understanding-input-and-output-shape-in-lstm-keras-c501ee95c65e)