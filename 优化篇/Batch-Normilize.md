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

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBbad4d65abe385345409d9f10c299631c?method=download&shareKey=6bb15c2191a1161d7b6ae177ed773f5f"/>
</div>

**为了这些数据再能够尽可能恢复本身的表达能力，会另外学历两个参数**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBd350451acada6b2ab7b61574c6a272fe?method=download&shareKey=4ea352244190b7ecc3ecb99710b479b5"/
</div>

**测试阶段：测试阶段可能不是分batch测试的，并且在模型推断时常常只输入一个样本，那么就不能再计算测试阶段的batch的均值和方差，这就需要用到训练集的数据。训练时计算的均值和方差会不断累积下来，通过移动平均的方法来近似得到整个样本集的均值和方差。**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBf5b64d290a9162d96361a3254542f501?method=download&shareKey=09369aab2a2642670b7676ab9a1a1883"/
</div>





## 4.求导：

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB3bdb2f8ffa99a0e7fa8a846e73f9fe70?method=download&shareKey=74bda50b2f2e0735b79b34736adcfa76"/>
</div>

