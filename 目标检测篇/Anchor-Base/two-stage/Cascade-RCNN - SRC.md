# Cascade R-CNN: Delving into High Quality Object Detection

## 概述：

**Cascade R-CNN通过级联的方式一级一级筛选出高质量的proposal，提升train和inference的精度**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB6e931f39957cefec87ebf965e3d8cd22?method=download&shareKey=99bcd4b72722bdbb14f9062f04cd45ad"/>
</div>


​                                                                                                                                      **图1**

## 提出背景

**目标检测过程中会通过IOU阈值来区分正负样本，如果IOU阈值太小则目标的噪声会很多，如果IOU阈值太大则会导致两个问题：**

- **正样本数量太少，导致网络过拟合**

- **训练阶段与前向阶段样本不匹配（训练阶段可以认为控制iou阈值来提升样本质量，但是inference阶段则要靠网络自己输出proposal，这是的质量不能得到保证，这就导致训练与inference阶段mismatch，导致结果不好。）**

  <div align=center>
  <img src="https://note.youdao.com/yws/api/personal/file/WEB66e5962eb0b4191706cf1d819f596311?method=download&shareKey=17be16d142f257b70eb4f14fa3201e07"/>
  </div>

  ​                                                                                                                             **图2**

**如上图(a)所示(实线部分)，在训练阶段的阈值为0.7时效果急剧下降，为了分析下降的原因人为把输入的质量提升再进行试验，如图(b)所实，此时阈值为0.7训练的模型的roc最高。这就是train和inference mismatch问题**



## 解决方案

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBc4e22409d3ff69801974430b6cb4d06b?method=download&shareKey=e617f338f0738c5149b4483ccce3c12a"/>
</div>

​                                                                                                                                 **图3**

**如图3(a)所示，曲线代表在不同阈值训练的模型，输入iou与输出iou的对应关系。可以得到如下结论：**

- **不同阈值训练出来的模型在输入iou在其附近时能够取得最好效果(与baseline的差)**
- **回归出的iou总是大于输入iou**

**基于以上发现，作者就想到图1(d)的级联结构，以固定阈值训练可以得到一个更好的训练分布样本，下一级输入为上一级的输出就能够得到较好的效果。**



**这种级联方式有三种选择方案：**

- **iterative bounding box regression(图1(b))，它的三个head是公用的，这会存在两个问题：**
  - **图3(c)所示，以u=0.5为例，它对于更高iou的输入是一个次优化(在input iou > 0.8是 output iou急剧变差)**
  - **随着级数的增加，bounding box的分布显著改变，如果每一级还公用参数那么显然是不合理的。**

- **Integral loss（图1(c)）,它的三个分支分别使用不同阈值{0.5,0.6,0.7}，第二、三分支负责精确分类，但是这存在两个问题：**
  - **后面两个分支由于阈值的提升，正样本数量势必急剧减小会导致过拟合发生**
  - **由于后两级高质量分分类器需要处理inference结算大量低质量的proposal，这些是它们在训练阶段没有优化过的(训练阶段有gt可以控制样本)，所以导致它的效果不好**

- **cascade rcnn（图1(d)），采用多级不同head，head结构相同但是各自参数不同，每一级的输入是前一级的输出，各级独自优化**



# 总结

**cascade通过大量试验分析，从试验角度出发设计网络，提升训练样本质量同时过拟合与mismatch的问题。试验结合理论再提出解决方案，这种方式很值得我们学习，**