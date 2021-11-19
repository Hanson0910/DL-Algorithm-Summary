# Dynamic R-CNN: Towards High Quality Object Detection via Dynamic Training

## 1.概述：

**Dynamic R-CNN 还是在样本选取的角度出发，认为网络在不同训练阶段porposal的分布会不一样，如果还是以一样的状态去选取样本会损害网络精度。结合cascade-rcnn，认为cascade-rcnn太耗时，提出了动态分配样本的方法，通过统计object与proposal的统计学特征来动态分配样本。**



## 2.提出背景

**这篇文章的背景跟cascade-rcnn的提出背景一样，可以参考我得cascade-rcnn，都是为了提升训练过程中的样本质量。**



## 3.解决方案

**不同于cascade-rcnn采用多级预测的方式太耗时，作者通过统计proposal的统计学特征来动态选择样本。**

### 3.1分类

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB35317a8df68f3f1887ba1af8fe9971cf?method=download&shareKey=077cc31b1ae1795a7ba01d97c250b6c5"/>
</div>

​                                                                                                                                        图1

**分类问题最好满足两个条件**

- **充分数量的正负样本**
- **精确的正样本**

**为了满足这两个条件，cascade-rcnn通过多级预测的方式，前面几级的iou阈值比较小，保证了正负样本的数量，后面的iou阈值比较大，保证了iou的精度。作者发现随着网络迭代的次数增加，正样本的数量会越来越多，这个很好理解，网络到后半段精度较高了。所以作者就利用不同训练阶段采用不同阈值的训练策略，训练前期采用低阈值，训练后期采用高阈值。低阈值来保证样本的充分性，高阈值来保证样本的精确性。动态阈值的调整策略如下：**

1. **计算mini-batch的样本与proposal的ious，选取前top-ki个值**
2. **没C论迭代统计一下这top-ki个值得均值作为iou阈值。**

**这样在前期阈值较小，后期阈值较大，可以达到预期效果。**

### 3.2回归

**Faster-RCNN得回归loss采用smooth-l1 loss，smooth-l1 loss对整个训练过程得样本都是一视同仁得，网络训练后期proposal得分布会变，如果还是一视同仁会损害模型精度，打个比方，训练后期proposal得精度越来越高，我们希望给你这些高质量的proposal更多的关怀，是他们能够更好回归，而低质量的proposal则给少一些关怀。**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBf223f76f4a797a65fe529074011864ef?method=download&shareKey=4811545646a5715cd570bdaff1687352"/>
</div>

​                                                                                                                                     图2

**作者通过统计proposal与gt的误差来动态更新一个参数，控制loss对proposal的关怀程度，如下式所示：**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB16613191bf3b9a2b66f9e7bf88ee5d5e?method=download&shareKey=ad5e122377f3fdb619a110960fd15f9c"/>
</div>







# 总结

**很难想象2020年还有RCNN相关的论文发表，而且说实话Dynamic R-CNN整体创新点并不多。通过动态阈值来选取proposal的论文还挺多的，有通过梯度的，有同归iou-balance的等等，另外就是回归loss这一块添加一个统计因子，我觉得focal loss的效果会更好一些，这些统计特征会因为目标的差异性不同应该差距还是有一些的。**