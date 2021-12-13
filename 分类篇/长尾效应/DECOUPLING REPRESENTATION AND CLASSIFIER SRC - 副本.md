

# DECOUPLING REPRESENTATION AND CLASSIFIER

## 1.概述

**目标分类过程中的长尾效应是一个很常见的问题，这篇文章通过试验发现类别较多的类特征权重值要大一些，类别较少的特征权重值要小一些。基于以上发现作责提出了一种特征归一化的方式来解决长尾效应问题。**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB102913dbc8dfa50cc4dc05d77739fa2e?method=download&shareKey=b66109951bf2d8b1f7d0b0009021c5d8"/>.
</div>

## 2.具体实现

**常见的长尾效应解决方案有：**

- **class-balancing strategies**
- **loss re-weighting**
- **data re-sampling**
-  **transfer learning**

**Decoupling在上面基础上分为两步来解决长尾效应：**

1. **在长尾数据上训练一个分类模型，然后再fix住backbone，再利用Class-balanced sampling finetune 分类head，也可以利用渐近训练的方式，类别j的采样概率如下式所示：**
   $$
   \large p_{j}^{\mathrm{PB}}(t)=\left(1-\frac{t}{T}\right) \large p_{j}^{\mathrm{IB}}+\frac{t}{T} p_{j}^{\mathrm{CB}}, \\
   \large t为当前epoch，T为总epoch，p_{j}^{IB}为instance-balance采样概率，p_{j}^{CB}\\
   \large 为class-balance采样策略
   $$
   

**采样公式为：**
$$
\large p_{j}=\frac{n_{j}^{q}}{\sum_{i=1}^{C} n_{i}^{q}}\\
\large n_{j}^{q}为类别j的个数，C为类别数，q为参数，
instance-sample时，q取1，class-sample时，q取0
$$

2. 权重归一化

   根据上图左半部分可以看到，样本数越多的类别权重会大一些，这样会使样本多的内别响应更大一些，样本数越小的类别权重会小一些。通过如下式进行权重归一化：
   $$
   \large \widetilde{w_{i}}=\frac{w_{i}}{\left\|w_{i}\right\|^{\tau}}
   $$
   

