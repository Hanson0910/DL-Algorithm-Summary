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
<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBeeb89630ef3fff1e9d9450c3c72c453f?method=download&shareKey=9b412cfcc32237f58b9aa284244744a8"/>
</div>

**如上图所示，这是目标在第L层特征图的实例，红色区域为正样本区域对应类别的特征图该区域标签为1，灰色区域为忽略区域，对应类别的特征图该区域标签为-1，其余为负例子，对应标签为0。两个同一类别的不同实例如果在同一特征层重叠，则优先考虑目标小的实例。**

### 2.5 Encode-Decode:

**Anchor-Base分支的encode-decode与[RetinaNet](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Base/one-stage/RetinaNet.md)一样，这里主要讨论Anchor-Free分支的编解码部分**

- **Encode**

  <div align=center>
  <img src="https://note.youdao.com/yws/api/personal/file/WEBf702702f4bbdd43046075fea252c1b1e?method=download&shareKey=7f50ed7c6dc2b378398bad344648d482"/>
  </div>

  

  <div align=center>
  <img src="https://note.youdao.com/yws/api/personal/file/WEBe5110879413eb90795af71ba9b1f2bc1?method=download&shareKey=e2102227bba53e545fe52112f9c448c6"/>
  </div>

- **Decode**

  

  <div align=center>
  <img src="https://note.youdao.com/yws/api/personal/file/WEB8cdb992ac0b8f83cc869c21947fd2ecd?method=download&shareKey=69aa146300238ab478f209f1f14f294c"/>
  </div>

### 2.4 Loss

**Anchor-Base分支的Loss与[RetinaNet](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Base/one-stage/RetinaNet.md)一样，下面主要讨论Anchor-Free部分的Loss**

- **分类 Loss**

  **分类Loss采用focal-loss**
  
- **回归loss**

  **回归Loss采用IOU Loss**
  

### 2.6 online feature selecetion：

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB6e4cd4df4b2dba3db5995a1c9a97975b?method=download&shareKey=5a6bd69adbea8e669f1711d51cae9408"/>
</div>

### 2.7 inference

**在anchor-free在每个特征层选取前1k的得分对应的类别和位置，然后同0.05的得分阈值滤除部分预测值，最后和anchor-base的预测组合再一起使用nms得出最终的预测。**


## 3. 总结：

**FASF网络能够兼容anchor-base网络，与anchor-base联合训练互补提高网络性能，利用在线特征选取模块能够有效为每个实例分配最优特征层.**

