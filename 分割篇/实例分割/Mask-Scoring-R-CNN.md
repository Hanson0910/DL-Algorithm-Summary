# **Mask Scoring R-CNN**

## 1.概述

**Mask Scoring R-CNN是在Mask R-CNN基础上添加MaskIOU分支增加分类准确性。**

**paper link:https://arxiv.org/abs/1903.00241**

**code link:https://github.com/zjhuang22/maskscoring_rcnn**

## 2.具体实现

### 2.1 网络结构

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBf377c2a9f2826218dd2e15f6c5277554?method=download&shareKey=071ef2d20419fbf69859810515ecb545"/>.
</div>

**可以看出mask scoring rcnn是在mask rcnn的基础上添加了一个MaskIOU分支**

### 2.2 训练阶段

- **RPN阶段产生proposal**
- **RCNN分支训练过程**
- **mask rcnn训练过程**
- **获取目标的预测mask，然后二值化**
- **计算二值化mask与gt mask的l2 loss**

### 2.3 前向阶段

- **maskiou head的输出直接与rcnn分支的类别输出相乘作为最终的分类得分**



## 3.总结

**Mask Scoring R-CNN这篇文章总体来说中规中矩，没有什么特别的地方，添加一个额外的分支辅助训练，最终效果有一定的提升，但是我认为maskiou部分用二值化mask去做回归可能会导致网络训练过程不易收敛。**