# ATSS:**Bridging the Gap Between Anchor-based and Anchor-free Detection via Adaptive Training Sample Selection**

## 1.概述

**ATSS通过对比试验发现造成anchor-base和anchor-free精度最大的区别是在正负样本选择上，进而通过分析目标统计学特征提出自动选择正负样本的方法，可以大幅提升检测精度。**

## 2.正文

### 2.1试验分析

**Anchor-base和anchor-free主要有三个不同点，以RetinaNet和FCOS为例，主要不同点如下：**

1. **每个点Anchor的数量，RetinaNet在每个特征点预先定义一组anchor，FCOS则是每个特征点为一个anchor point**
2. **定义正负样本的形式，RetinaNet通过IOU来定义正负样本，FCOS则通过目标中心一定区域来定义正负样本**
3. **回归的方式，RetinaNet回归目标的anchor的bounding box，FCOS则是回归重点坐标和宽高。**



**为了验证上面哪个差异是造成anchor-base和anchor-free差别的主要原因，作者首先通过把retinanet的正负样本选择策略与FCOS一样，再把FCOS的正负样本选择策略与RetinaNet一样做了对比试验，试验结果如下图所示：**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBf3ae6845b00d8b99cd26987849cfe137?method=download&shareKey=8411fbeeb9a57d25971278f3db822dea"/>
</div>

​                                                                                                                                   图1

**如上图所示，如果RetinaNet改成与FCOS一样的正负样本选取，map由37.0->37.8，如果FCOS改成与RetinaNet一样的正负样本选取，map由37.8->36.9.通过如上试验表明正负样本选取是照成anchor-base和anchor-free差异的一个很大因素，同时也证实了回归方式的影响不大。**

### 2.2 **Adaptive Training Sample Selection**



**通过试验分析了由于正负样本的选择对anchor-base和anchor-free照成较大的差异，作者在此基础上提出了一个新的样本分配策略：**

1. **在每一个层特征金字塔，选取离目标中心点L2距离最近的top-k个anchor box，假设一共有L层，则每个object一共有L*k个anchor box。**
2. **计算这些anchor-box与object的iou用Dg表示，并计算他们的均值与方差mg和vg**
3. **计算iou阈值tg = mg+vg**
4. **选取iou大于tg的anchor box**
5. **如果该anchor-box被多个object选中，则把该anchor box分配给iou最高的object**

**通过上面的步骤来选取正负样本，可以选取大约0.2*k*l的正样本，与目标的 scale，aspect ratio等因素无关。这与retinanet和fcos不同，他们传统的策略越大的目标可能分配更多的正样本。而且这中方式只有一个超参K，试验表明超参K对结果并不敏感。**

**RetinaNet采用这种策略可以提升2.3个点，FCOS则有两个版本：**

- **在原FCOS基础上仅用中线点采用的方式，选取top-k个正例，保留了原来FPN层的样本的分配策略。**
- **完全采用ATSS的样本分配策略，用anchor-point改成anchor-box的回归，anchor-box的大小为8*S，aspec ratio为1:1，S为下采样步数。**

**试验结果如下图所示：**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB1122107d0db620d0818a3e84270685bf?method=download&shareKey=49e6782820c17dbe2df64c58ee660869"/>
</div>

​                                                                                                                                 图2

**其中base-anchor的大小和aspect-ratio试验结果如下图所示：**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB57c95bbe5eb9ce31733d3793cc880e30?method=download&shareKey=210b7b8290affaa517003f7e36a96bbc"/>
</div>

## 3.总结

**ATSS通过以RetinaNet和FCOS的对比试验发现照成了anchor-base和anchor-free结果差异的因素，RetinaNet和FCOS我觉得是比较有代表性的算法，能够验证作者的试验。其实我一直也有类似的想法，anchor-base通过iou来选择样本确实有一定的局限性，FreeAnchors也分析这些局限性，通过center-point再选择anchor我觉得更加合理一些，哪怕前期iou比较小，后面学习的过程应该能纠正。把FCOS进行一番魔改确实精度提升了不老少，但是没有anchor-free的感觉了，整体框架复杂性提升了一下。整体来时ATSS提点还是十分明显的，复杂性也不是很高，超参很少，能够灵活嵌套在anchor-base和anchor-free中。**

