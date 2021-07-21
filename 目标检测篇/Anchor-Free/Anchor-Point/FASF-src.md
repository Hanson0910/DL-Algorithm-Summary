## FASF-Feature Selective Anchor-Free Module for Single-Shot Object DetectionFeature Selective Anchor-Free Module for Single-Shot Object Detection

## 1.概述：

**目前基于anchor的目标检测方法，大多采用不同的level预测不同尺度的instance，而分配规则往往是人为设计的，这导致anchor的匹配策略可能不是最优的。本文提出了feature selective anchor-free module来解决这个限制，让每个示例动态选择最适合自己的特征层级，并且FASF可以与anchor-based 分支协同并行工作，提高模型效果。在one-stage模型基础上添加一个anchor-freen部分，论文以retinanet作为例子如下图所示。**

<div align=center>
<img src="https://img-blog.csdnimg.cn/20200102225600745.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0NodW5mZW5neWFueXVsb3Zl,size_16,color_FFFFFF,t_70"/>
</div>

## 2. 具体实现：

### 2.1 Backbone：

**主干网络参考[RetinaNet主干网络](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Base/one-stage/RetinaNet.md)**

### 2.2 Neck:

**Neck部分参考[RetinaNet Neck](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Base/one-stage/RetinaNet.md)**

### 2.3 Head:

**输入经Backbone后输出3个分支:**

- **Anchor Base分支**
  
  **Anchor Base分支参考[RetinaNet Head](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Base/one-stage/RetinaNet.md)**
  
- **Anchor Free分支**
  
  **FASF在RetinaNet基础上添加了一个Anchor-Free分支，anchor-free分支由两部分组成：类别分支、回归分支。类别分支C个通道(C为类别总数，回归分支由4通道组成：(cx,cy,w,h))。**

### 2.4 Sample Assignment:

**Anchor-Base分支的正负样本分配与[RetinaNet](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Base/one-stage/RetinaNet.md)一样，还是依靠IOU来分配正负样本，这里不做详细介绍。Anchor-Free部分的正负样本分配如下图所示：**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB97b61cb385ebac081494fa09a5f45d01?method=download&shareKey=a30ff7a5f2b2f9060f6f5ef4726b45c7"/>
</div>

$$
\large 实例I原始坐标信息为b=[x,y,w,h],x,y为实例的中心点，w,h为实例的宽和高。\\
\large I在第l层特征P_{l}的坐标为b^l_{p}=[x^l_{p},y^l_{p},w^l_{p},h^l_{p}],b^l_{p} = b/2^l\\
\large 以b^l_{p}为base，建立effective box和ignoring box分别表示为\\
\large b^l_{e}=[x^l_{e},y^l_{e},w^l_{e},h^l_{e}],b^l_{i}=[x^l_{i},y^l_{i},w^l_{i},h^l_{i}]
$$

$$
\large effective box和ignoring box的建立规则是，是以base的中心点为中心，\\
\large base的宽高*\epsilon_{e}的区域为有效box区域，base的宽高*\epsilon_{i}的区域为忽略box区域。
\large \epsilon_{e}默认值为0.2,\epsilon_{i}默认值为0.5。
$$

**如上图所示，这是目标在第L层特征图的实例，红色区域为正样本区域对应类别的特征图该区域标签为1，灰色区域为忽略区域，对应类别的特征图该区域标签为-1，其余为负例子，对应标签为0。两个同一类别的不同实例如果在同一特征层重叠，则优先考虑目标小的实例。**

### 2.5 Encode-Decode:

**Anchor-Base分支的encode-decode与[RetinaNet](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Base/one-stage/RetinaNet.md)一样，这里主要讨论Anchor-Free分支的编解码部分**

- **Encode**
  $$
  \large 仅回归有效区域的框,对于每一个在b^l_{e}的点(i,j)，用一个4维度向量d^l_{i,j}表示，\\
  \large d^l_{i,j}=[d^l_{t_{i,j}},d^l_{l_{i,j}},d^l_{b_{i,j}},d^l_{r_{i,j}}]\\
  \large d^l_{t},d^l_{l},d^l_{b},d^l_{r}分别表示原始框与当前位置(i,j)的顶部，左部，底部，右部的距离。\\
  $$

  $$
  \large 蓝色为实例I的box区域，红色为有效区域，两个小黑点代表有效区域里面的点(i,j)，\\
  \large 灰色箭头就是(i,j)顶部，左部，底部，右部距离[d^l_{t_{i,j}},d^l_{l_{i,j}},d^l_{b_{i,j}},d^l_{r_{i,j}}]。\\
  \large 最终还需要把这4维向量除以S即d^l_{i,j} / S,\\
  \large S是一个归一化参数，为了防止训练的时候不收敛。论文里面把S设置为4
  $$

  

  <div align=center>
  <img src="https://note.youdao.com/yws/api/personal/file/WEBe5110879413eb90795af71ba9b1f2bc1?method=download&shareKey=e2102227bba53e545fe52112f9c448c6"/>
  </div>

- **Decode**

$$
\large 假设第l层位置(i,j)的网络输出的4-dim向量表示为\\
\large \left[\hat{o}_{t_{i, j}}, \hat{o}_{l_{i, j}}, \hat{O}_{b_{i, j}}, \hat{O}_{r_{i, j}}\right],然后乘以归一化系数S(默认为4)，为\left[S \large \hat{o}_{t_{i, j}}, S \hat{o}_{l_{i, j}}, S \hat{o}_{b_{i, j}}, S \hat{o}_{r_{i, j}}\right]，\\
\large top-left和right-bottom的值表\large 示为：\left.\left(i-S \hat{o}_{t_{i, j}}, j-S \hat{o}_{l_{i, j}}\right) \text { and }\left(i+S \hat{o}_{b_{i, j}}, j+S \large \hat{o}_{r_{i, j}}\right]\right)，\\
\large 最后再把该值乘以2^l即为实际坐标
$$



### 2.4 Loss

**Anchor-Base分支的Loss与[RetinaNet](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Base/one-stage/RetinaNet.md)一样，下面主要讨论Anchor-Free部分的Loss**

- **分类 Loss**

  **分类Loss采用focal-loss**
  
- **回归loss**

  **回归Loss采用IOU Loss**
  

### 2.6 online feature selecetion：

$$
\large 实例I在第l层特征的分类和回归loss分别用L^I_{FL}(l)和L^I_{IOU}{l}表示，然后它的有效box的平均loss\\
\large \begin{aligned}L_{F L}^{I}(l) &=\frac{1}{N\left(b_{e}^{l}\right)} \sum_{i, j \in b_{e}^{l}} F L(l, i, j) \\
\large L_{I o U}^{I}(l) &=\frac{1}{N\left(b_{e}^{l}\right)} \sum_{i, j \in b_{e}^{l}} \operatorname{IoU}(l, i, j)\end{aligned}\\
\large N\left(b_{e}^{l}\right)代表有效区域的像素点个数，最后看该实例I在哪层特征的loss最低\\
\large 就认为该层适合该实例\\
\large l^{*}=\arg \min _{l} L_{F L}^{I}(l)+L_{I o U}^{I}(l)\\
\large 在训练得时候只更新该实例对应最优层部分的loss，在inference的时候不用feature\\
\large selection模块，因为训练的时候已经保证了该实例在其最优层能够取得最大得分
$$





### 2.7 inference

**在anchor-free在每个特征层选取前1k的得分对应的类别和位置，然后同0.05的得分阈值滤除部分预测值，最后和anchor-base的预测组合再一起使用nms得出最终的预测。**


## 3. 总结：

**FASF网络能够兼容anchor-base网络，与anchor-base联合训练互补提高网络性能，利用在线特征选取模块能够有效为每个实例分配最优特征层.**

