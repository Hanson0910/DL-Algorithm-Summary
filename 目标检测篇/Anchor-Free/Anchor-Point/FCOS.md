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

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB37368b773136626d851fe377a7a0fdea?method=download&shareKey=91d192527d4aaba17d381c20dce8240b"/>
</div>



<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB61d63937debeed53a80f5711b73dce02?method=download&shareKey=7951a04560e9664796d7b71af84c19c1"/>
</div>

### 2.4 Sample Assignment:

**与FASF不一样，FCOS把所有在目标区域的点都认为是正样本，其余为负样本。为了使不同层级的特征回归对应目标，不同于anchor-base依据iou来把目标分配到对应的FPN层,FCOS分配规则如下:**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB2da9d4a1794bc9812ef2e64f96978fb5?method=download&shareKey=77cf9687cdd98994215a73817ed84ddb"/>
</div>

**值得注意的是，即便是使用了多层预测，仍然会存在一个位置(x,y)可能存在多个相同类别的实例都为正例，这时简单的选取(x,y)属于面积较小的那个实例。如下图所示：黑色和蓝色区域是同一层两个相同类别的实例，红点即在黑色区域也在蓝色区域里面，这时我们把红点分配给面积较小的那个区域。**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB51cdb5be43acbf341726528b6213d8c3?method=download&shareKey=78d6ac69cd670de0ef87acdec3e2ac25"/>
</div>

### 2.5 中心度度量Center-Ness:

**依据上诉的做法会产生很多原理目标中心的低质量框，这也很好理解，因为把目标宽区域所有的点都作为正样本，然后边界据这些点的偏置，这样负责回归正样本的数量会比较多，难免有错误的，为此提出了中心度度量center-ness。**
<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB36261225e04ae558966ca60c1c5d69ca?method=download&shareKey=43c7b936f1b69b7434d84c6bb3656ae8"/>
</div>

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

