

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

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBe36b2f17d7ebe7f3769dc65175e06b5b?method=download&shareKey=2b186472a0d54c84ed45b39b1410d06c"/>.
</div>