# BN

## 1.概述

**深层网络训练的过程中，由于网络中参数变化而引起内部结点数据分布发生变化的这一过程被称作Internal Covariate Shift。Internal Covariate Shift的存在使得上层网络需要不停调整来适应输入数据分布的变化，导致网络学习速度的降低；网络的训练过程容易陷入梯度饱和区，减缓网络收敛速度**



## 2.如何减缓Internal Covariate Shift

### 白化：

- **白化（Whitening）是机器学习里面常用的一种规范化数据分布的方法，主要是PCA白化与ZCA白化。白化是对输入数据分布进行变换，进而达到以下两个目的**

  1. **使得输入特征分布具有相同的均值与方差。其中PCA白化保证了所有特征分布均值为0，方差为1；而ZCA白化则保证了所有特征分布均值为0，方差相同；**

  2. **去除特征之间的相关性。**

- 白化缺点:
  1. **白化过程计算成本太高**;
  2. **白化过程由于改变了网络每一层的分布，因而改变了网络层中本身数据的表达能力。底层网络学习到的参数信息会被白化操作丢失掉。**

### BN：

**白化计算过程比较复杂，BN就简化一点:**

1. **单独对每个特征进行normalizaiton,让每个特征都有均值为0，方差为1的分布;**
2. **既然白化操作减弱了网络中每一层输入数据表达能力，那就再加个线性变换操作，让这些数据再能够尽可能恢复本身的表达能力。**

**BN优点：**

- **BN使得网络中每层输入数据的分布相对稳定，加速模型学习速度；**
- **BN使得模型对网络中的参数不那么敏感，简化调参过程，使得网络学习更加稳定；**
- **BN允许网络使用饱和性激活函数（例如sigmoid，tanh等），缓解梯度消失问题；**
- **BN具有一定的正则化效果:在Batch Normalization中，由于我们使用mini-batch的均值与方差作为对整体训练样本均值与方差的估计，尽管每一个batch中的数据都是从总体样本中抽样得到，但不同mini-batch的均值与方差会有所不同，这就为网络的学习过程中增加了随机噪音，与Dropout通过关闭神经元给网络训练带来噪音类似，在一定程度上对模型起到了正则化的效果。**





## 3.公式：

$$
 \tilde{x}= \frac{x- \mu }{ \sqrt{ \sigma ^{2}+ \varepsilon }}\\
  \mu = \frac{1}{N} \sum _{i}^{N}x_{i} , \sigma ^{2}= \frac{1}{N} \sum _{i}^{N}(x_{i}- \mu )^{2} \\
  N为batchsize,x_{i}为batch的每一个样本，在具体实现时\\(如果网络的输入是二维的图像),一般BN层的前面一层的卷积输出为(N,C,H,W),\\C,H,W分别为特征图的通道、宽、高。那么每一个样本可以表示为x_{c,i,j}，\\BN实际上是对各个x_{c,i,j}分别进行归一化，因此计算得到的\mu 也相应的有C,H,W个。
$$

**为了这些数据再能够尽可能恢复本身的表达能力，会另外学历两个参数**
$$
y = \gamma \cdot \tilde{x} + \beta \\特别地，当\gamma^2 = \sigma^2,\beta = \mu时，以实现等价变换\\（identity transform）并且保留了原始输入特征的分布信息。
$$
**测试阶段：测试阶段可能不是分batch测试的，并且在模型推断时常常只输入一个样本，那么就不能再计算测试阶段的batch的均值和方差，这就需要用到训练集的数据。训练时计算的均值和方差会不断累积下来，通过移动平均的方法来近似得到整个样本集的均值和方差。**
$$
\tilde{ \mu }= \mu _{n}= \alpha \mu _{n-1}+(1- \alpha ) \cdot \frac{1}{N} \sum _{i}^{N}x_{i,n} \\
 x_{i,n}表示第n的batch的一个样本，\mu_{n}表示第n个batch的均值，同理\sigma^2也可以同样得到。
$$




## 4.求导：

$$
\frac{dl}{d\gamma} = \sum _{i}^{N}\frac{dl}{dy_{i}}\cdot \frac{dy_{i}}{d_{\gamma}}=\sum _{i}^{N}\frac{dl}{dy_{i}}\cdot \tilde{x_{i}}\\
\frac{dl}{d\beta} = \sum _{i}^{N}\frac{dl}{dy_{i}}\cdot \frac{dy_{i}}{d_{\beta}}=\sum _{i}^{N}\frac{dl}{dy_{i}}\\
\frac{dl}{dx_{i}}=
\frac { \partial l } { \partial \tilde { x } _ { i } } \cdot \frac { \partial \tilde { x } _ { i } } { \partial x _ { i } } + \frac { \partial l } { \partial \sigma _ { B } ^ { 2 } } \cdot \frac { \partial \sigma _ { B } ^ { 2 } } { \partial x _ { i } } + \frac { \partial l } { \partial \mu _ { B } } \cdot \frac { \partial \mu _ { B } } { \partial x _ { i } }\\
1.\frac { \partial l } { \partial \tilde { x } _ { i } } = \frac { \partial l } { \partial y _ { i } } \cdot \frac { \partial y _ { i } } { \partial \tilde { x } _ { i } } = \frac { \partial l } { \partial y _ { i } } \cdot \gamma\\
2. \frac { \partial l } { \partial \sigma _ { B } ^ { 2 } } = \sum _ { i = 1 } ^ { m } \frac { \partial l } { \partial \tilde { x } _ { i } } \cdot \frac { \partial \tilde { x } _ { i } } { \partial \sigma _ { B } ^ { 2 } } = \sum _ { i = 1 } ^ { m } \frac { \partial l } { \partial \tilde { x } _ { i } } \cdot ( x _ { i } - \mu _ { B } ) \cdot \frac { - 1 } { 2 } ( \sigma _ { B } ^ { 2 } + \varepsilon ) ^ { - 3 / 2 }\\
3. \frac { \partial l } { \partial \mu _ { B } } = \frac { \partial l } { \partial \sigma _ { B } ^ { 2 } } \cdot \frac { \partial \sigma _ { B } ^ { 2 } } { \partial \mu _ { B } } + \sum _ { i = 1 } ^ { m } \frac { \partial l } { \partial \tilde { x } _ { i } } \cdot \frac { \partial \tilde { x } _ { i } } { \partial \mu _ { B } }= \frac { \partial l } { \partial \sigma _ { B } ^ { 2 } } \cdot \frac { 1 } { m } \sum _ { i = 1 } ^ { m } - 2 ( x _ { i } - \mu _ { B } ) + \sum _ { i = 1 } ^ { m } \frac { \partial l } { \partial \tilde { x } _ { i } } \cdot \frac { - 1 } { \sqrt { \sigma _ { B } ^ { 2 } + \varepsilon } }\\
\frac{dl}{dx_{i}}= \frac { \partial l } { \partial \tilde { x } _ { i } } \cdot \frac { 1 } { \sqrt { \sigma _ { B } ^ { 2 } + \varepsilon } } + \frac { \partial l } { \partial \sigma _ { B } ^ { 2 } } \cdot \frac { 2 ( x _ { i } - \mu _ { B } ) } { m } + \frac { \partial l } { \partial \mu _ { B } } \cdot \frac { 1 } { m }
$$

