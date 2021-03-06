---
layout:     post
title:      "初学Tensorflow"
subtitle:   "最基础的知识记录"
date:       2018-11-12
author:     "Qian"
header-img: "img/in-post/post-bg-tensorflow.jpg"
tags:
    - tensorflow
---

> 初识Tensorflow


## 主要依赖

### Protocal Buffer
tensorflow使用的处理结构化数据的工具，类似xml、json，与其不同在于，使用Protocol Buffer 时需要先定义数据的格式（schema），一般保存在.proto文件中。还原一个序列化之后的数据将需要使用到这个定义好的数据格式。

与xml相比，序列化出来的数据更小，解析时间更短。

### Bazel
自动化构建工具，完成tensorflow应用的编译。

## tensorflow基础
tensor是张量，可以简单理解为多维数组，flow是流，直观地表达了张量之间通过计算相互转化的流程，通过计算图的形式来表述计算的编程系统，每一个计算都是计算图中的一个节点。

### 张量
一个张量主要保存了三个属性： 名字（ name ）、维度（ shape ）和类型（ type ）。属性名字不仅是一个张量的唯一标识符，它同样也给出了这个张量是如何计算出来的。计算图上的每一个节点代表了一个计算，计算的结果就保存在张量之中。张量的命名就可以通过“node:src_ output”的形式来给出。其中node 为节点的名称， src一output 表示当前张量来自节点的第几个输出。

张量的第二个属性是张量的维度（ shape ）。这个属性描述了一个张量的维度信息。比如shape=(2，）说明了张量是一个一维数组，这个数组的长度为2。

第三个属性是类型（type），每一个张量会有一个唯一的类型。TensorFlow会对参与运算的所有张量进行类型的检查，当发现类型不匹配时会报错。

### 会话
Tensorflow通过计算图和张量组织好数据与运算后，通过会话（session）来执行定义好的运算，会话拥有并管理TensorFlow程序运行时的所有资源，为防止程序异常中断而导致资源泄漏，一般常用python的上下文管理器来使用会话，如：

```
# 创建一个会话，并通过Python 中的上下文管理器来管理这个会话。
with tf. Session() as sess :
＃使用创建好的会话来计算关心的结果。
sess.run(...)
# 不需要再调用“Session.close()”函数来关闭会话，
# 当上下文退出时会话关闭和资源释放也自动完成了。
```

### tensorflow实现神经网络
Tensorflow中，变量（tf.Variable）的作用就是保存和更新神经网络中的参数。变量的初始化可以有多种方式：
```
# 产生一个2 x 3矩阵，矩阵中的元素是均值为0，标准差为2的随机数。
weights = tf.Variable(tf.random_normal([2,3),stddev=2))
# 生成初始值为0长度为3的变量
biases = tf.Variable(tf.zeros([3)))
```

虽然在变量定义时给出了变量初始化的方法，但这个方法并没有被真正运行，需要运行weights.initializer来赋值，若变量太多时，TensorFlow提供了一种更加便捷的方式来完成变量初始化过程，通过tf.global_variables_initializer函数实现初始化所有变量。

Tensorflow训练中每次迭代的训练数据是一个batch，如果每轮迭代中选取的数据都要通过常量来表示，那么TensorFlow的计算图将会太大。因为每生成一个常量， TensorFlow都会在计算图中增加一个节点。为了避免这个问题， TensorFlow提供了placeholder机制用于提供输入数据。placeholder相当于定义了一个位置，这个位置中的数据在程序运行时再指定。

```
import tensorf low as tf
wl = tf.Variable(tf.random normal([2 , 3], stddev=l ))
w2 = tf.Variable(tf.random normal([3, 1], stddev=l))
# 定义placeholder 作为存放输入数据的地方。这里维度也不一定要定义。
# 但如果维度是确定的，那么给出维度可以降低出错的概率。
x = tf.placeholder(tf.float32 , shape=(l , 2 ), name =” input")
a = tf.matmul (x, wl)
y = tf.matmul(a , w2)
sess = tf. Session()
init_op = tf.global_variables_initializer()
sess .run(init_op)
# 下面一行将报错： InvalidArgumentError: You must feed a value f。r placeholder
# tensor 'input_1' with dtype float and shape [1,2]
print(sess.run(y))
# 下面一行将会输出结果：［ [3. 95757794]]
print(sess.run(y,feed_dict={x: [[0.7,0.9]]}))
```

训练神经网络的全部过程，可以分为以下三个步骤：

1. 定义神经网络的结构和前向传播的输出结果。
2. 定义损失函数以及选择反向传播优化的算法。
3. 生成会话（tf.Session）并且在训练、数据上反复运行反向传播优化算法。

## 深度神经网络

### 激活函数和偏置项
如果神经网络的神经元结构的输出仅为所有输入的加权和，这导致整个神经网络是一个线性模型，因此，为将神经网络模型非线性化，加入了非线性函数（激活函数）以及偏置项（bias）
![neuron](/img/in-post/post-tensorflow/neuron.jpg "神经元结构")

图中激活函数为sigmod函数，此外其他一下常用得激活函数如ReLU、tanh函数，此外，Tensorflow还支持使用自己定义的激活函数。

### 损失函数定义
损失函数用以判断实际输出与期望的差距，交叉熵是常用的评判方法之一，刻画了两个概率分布之间的距离，但神经网络前向传播的结果并不一定为概率分布（即各种事件的概率和为1），因此，采用Softmax回归层使神经网络的输出变成一个概率分布。

例如：某模型经过Softmax 回归之后的预测答案是（0.5,0.4,0.1），那么这个预测和正确答案之间的交叉熵为：H((1,0,0),(0.5,0.4,0.1))＝-(1xlog0.5+0xlog0.4+0xlog0.1)=0.3

此外，用户可以自定义损失函数。

### 神经网络优化
神经网络通过反向传播算法（backpropagation）和梯度下降算法（gradient decent）调整神经网络中参数的取值，假设用θ表示神经网络中的参数，J(θ)表示在给定的参数取值下，训练数据集上损失函数的大小，那么整个优化过程可以抽象为寻找一个参数θ，使得J(θ)最小。目前没有一个通用的方法可以对任意损失函数直接求解最佳的参数取值，所以在实践中，梯度下降算法是最常用的神经网络优化方法。梯度下降算法会法代式更新参数θ ，不断沿着梯度的反方向让参数朝着总损失更小的方向更新。

参数的梯度可以通过求偏导的方式计算，对于参数θ，其梯度为J(θ)对θ求导，还需要定义一个学习率η（ learning rate ）来定义每次参数更新的幅度。参数更新的公式为：
![](/img/in-post/post-tensorflow/gradient_decent.jpg)

梯度下降算法不一定能达到全局最优，只有损失函数为凸函数时，才能保证达到全局最优而不是局部最优，此外，梯度下降算法的另外一个问题就是计算时间太长。因为要在全部训练数据上最小化损失，所以损失函数J(θ)是在所有训练数据上的损失和。这样在每一轮迭代中都需要计算在全部训练数据上的损失函数。

为了加速训练过程，可以使用随机梯度下降的算法(stochastic gradient descent)。这个算法优化的不是在全部训练数据上的损失函数，而是在每一轮法代中，随机优化某一条训练数据上的损失函数。虽然减少了训练时间，但神经网络的优化程度不高。

为了综合梯度下降算法和随机梯度下降算法的优缺点，在实际应用中一般采用这两个算法的折中一一每次计算一小部分训练数据的损失函数。这一小部分数据被称之为一个batch。

在迭代训练过程中，学习率既不能过大，也不能过小。为了解决设定学习率的问题， TensorFlow 提供了一种更加灵活的学习率设置方法一一指数衰减法。tf.train.exponential_decay函数实现了指数衰减学习率。通过这个函数，可以先使用较大的学习率来快速得到一个比较优的解，然后随着迭代的继续逐步减小学习率，使得模型在训练后期更加稳定。

为了避免过拟合问题， 一个非常常用的方法是正则化（regul arization）。正则化的思想就是在损失函数中加入刻画模型复杂程度的指标。假设用于刻画模型在训练数据上表现的损失函数为J(θ)，那么在优化时不是直接J(θ)，而是优化J(θ)＋λR(w)。其中R(w)刻画的是模型的复杂程度，而λ表示模型复杂损失在总损失中的比例。注意这里θ表示的是一个神经网络中所有的参数，它包括边上的权重w 和偏置项b。一般来说模型复杂度只由权重w决定。其基本的思想都是希望通过限制权重的大小，使得模型不能任意拟合训练数据中的随机噪音。

## tensorflow使用

### 模型

模型持久化保存方式，这段代码会生成的第一个文件为model.ckpt.meta，它保存了TensorFlow 计算图的结构，可以简单理解为神经网络的网络结构。第二个文件为model.ckpt，这个文件中保存了TensorFlow 程序中每一个变量的取值。最后一个文件为checkpoint文件，这个文件中保存了一个目录下所有的模型文件列表。

```
import tensorflow as tf
# 声明训练推理过程
......

# 声明tf . train . Saver 类用于保存模型。
saver= tf.train.Saver()
with tf.Session() as sess :
    sess.run(...)
    # 将模型保存到/path/to/model/model.ckpt文件。
    saver.save(sess ,"/path/to/model/model.ckpt")
```

加载持久化图的方式为：

```
import tensorflow as tf
# 直接加载持久化的图。
saver = tf.train.import_meta_graph("/path/to/model/model.ckpt/model.ckpt.meta” )
with tf.Session() as sess :
    saver.restore(sess , "/path/to/model/model.ckpt")
```