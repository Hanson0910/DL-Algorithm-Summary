## FCOS

## 1.概述：

**算法是一种基于FCN的逐像素目标检测算法，实现了无锚点（anchor-free）、无提议（proposal free）的解决方案，并且提出了中心度（Center—ness）的思想，同时在召回率等方面表现接近甚至超过目前很多先进主流的基于锚框目标检测算法。**

<div align=center>
<img src="https://pic4.zhimg.com/v2-bbec7cf563fc6c0bb981bef30bb9ca17_r.jpg"/>
</div>


## 2. 具体实现：

### 2.1 Backbone：

**------**

### 2.2 Neck:

**FPN部分作为网络Neck部分**

### 2.3 Head:

**FCOS Head部分分为三个分支：类别分支、回归分支和中心度度量分支**

### 2.4 Encode-Decode：

$$
\large \begin{aligned}
\large 输入的gt集合为\left\{B_{i}\right\},B_{i}=\left(x_{0}^{(i)}, y_{0}^{(i)}, x_{1}^{(i)},y_{1}^{(i)}, c^{(i)}\right) \\
\large \left(x_{0}^{(i)}, y_{0}^{(i)}\right),\left(x_{1}^{(i)}, y_{1}^{(i)}\right)分别表示左上和右下点的坐标,c_{i}为第i个类被。\\
\large (x,y)为改box里面的点坐标。
\large \end{aligned}
$$


$$
\large \begin{aligned}每一个正样本回归一个四维向量，t ^ { * } = ( l ^ { * } , t ^ { * } , r ^ { * } , b ^ { * } ),\\
\large l ^ { * } , t ^ { * } , r ^ { * } , b ^ { * }分别是left,top,right,booton到点(x,y)的距离 \\
\large l^{*}=x-x_{0}^{(i)}, & t^{*}=y-y_{0}^{(i)}, \\
\large r^{*}=x_{1}^{(i)}-x, \quad b^{*}=y_{1}^{(i)}-y .
\end{aligned}
$$

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB61d63937debeed53a80f5711b73dce02?method=download&shareKey=7951a04560e9664796d7b71af84c19c1"/>
</div>

### 2.4 Sample Assignment:

**与FASF不一样，FCOS把所有在目标区域的点都认为是正样本，其余为负样本。为了使不同层级的特征回归对应目标，不同于anchor-base依据iou来把目标分配到对应的FPN层,FCOS分配规则如下:**
$$
\large ( l ^ { * } , t ^ { * } , r ^ { * } , b ^ { * } ) >= m _ { i }且( l ^ { * } , t ^ { * } , r ^ { * } , b ^ { * } ) < m _ { i+1}则改特征属于第i层FPN，其余层的改box不做回归。\\
\large m _ { i }是第i层的最大距离。各层的最大距离为m _ { 3 },m _ { 4 },m _ { 5 },m _ { 6 },m _ { 7 }分别为0，64，128，256，512，+无穷\\
$$
**值得注意的是，即便是使用了多层预测，仍然会存在一个位置(x,y)可能存在多个相同类别的实例都为正例，这时简单的选取(x,y)属于面积较小的那个实例。如下图所示：黑色和蓝色区域是同一层两个相同类别的实例，红点即在黑色区域也在蓝色区域里面，这时我们把红点分配给面积较小的那个区域。**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB51cdb5be43acbf341726528b6213d8c3?method=download&shareKey=78d6ac69cd670de0ef87acdec3e2ac25"/>
</div>

### 2.5 中心度度量Center-Ness:

**依据上诉的做法会产生很多原理目标中心的低质量框，这也很好理解，因为把目标宽区域所有的点都作为正样本，然后边界据这些点的偏置，这样负责回归正样本的数量会比较多，难免有错误的，为此提出了中心度度量center-ness。**
$$
\large \text { centerness }^{*}=\sqrt{\frac{\min \left(l^{*}, r^{*}\right)}{\max \left(l^{*}, r^{*}\right)} \times \frac{\min \large \left(t^{*}, b^{*}\right)}{\max \left(t^{*}, b^{*}\right)}}
$$
**上式是训练centerness的编码，可见点越靠近目标中心centerness的值越高，inference的时候把centerness部分的网络输出值跟scores对应的值相乘，作为最终的scores。认为越靠近目标中心的回归越靠谱。centerness即理解为靠近目标中心度的一种度量。**

### 2.6 Loss

- **分类Loss**

  **分类Loss采用Focal Loss**

- **回归Loss**

  **回归Loss采用IOU Loss**

- **中心度度量Loss**

  **Center-Ness Loss采用BCE Loss**

### 2.7 inference

**inference的时候把centerness部分的网络输出值跟scores对应的值相乘，作为最终的scores。**

****


## 3. 总结：

**FCOS网络结构十分简单，同时利用了FPN多层特征能够有效检测多尺度目标，提出的中心度度量分支可以有效滤除低质量的输出，同时在相同目标拥挤场景下FCOS的性能并好，因为他输出的特征图只能在相同类别上回归一个目标，类似于wider-face这样的场景检测性能并不会理想。**

