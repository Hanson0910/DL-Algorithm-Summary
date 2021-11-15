# **RepPoints: Point Set Representation for Object Detection**

## 1.概述

**RepPoints使用一个完全anchor-free的检测框架，通过deformable-conv预测9组点来代表目标的重点区域，再通过这9组点生成伪box进行回归。RepPoints达到了真正的anchor-free，整个检测框架十分简洁，超参少。**



## 2.网络结构



<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBfa77a1eed47e53eb35cca2a2b7e42ecc?method=download&shareKey=5f6edd3b0a77e970d82483d709e09dff"/>
</div>

​                                                                                                                                         **图1**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB65319d39207c723f4a22467749d08dad?method=download&shareKey=c06aeaa9bc942696b31e89d23479fa8d"/ width=720 height=480 >
</div>



​                                                                                                                               **图2**

**如上图所示，RepPoints网络结构分为连个分支--检测和回归分支。其中检测分支又有2个stage，stage-1生成粗糙的pseudo-box再经stage-2生成精细的pseudo-box。图1中的N为生成点对的个数（文中设置为9）。**

## 2.具体实现

### 2.1. **RepPoints**：

**RepPoints的生成是通过deformable-conv，生成N个点对（文中9对）的偏置，改偏置是相对于目标中心点而言，即通过目标中心点坐标和deformable-conv生成的偏置decode成实际的点对**。

1. **点对生成**

- **stage-1**

  **这一阶段的解码论文中没有明确的公式表明，只是说明了一下，以目标中心点为base，通过预测的9个offsets来生成9组点对：**

  <div align=center>
  <img src="https://note.youdao.com/yws/api/personal/file/WEB863a4d746eb53d0200c0c6484ede3ef5?method=download&shareKey=740592ee228c4898e73e484b94e23fb0"/>
  </div>

- **stage-2**

  <div align=center>
  <img src="https://note.youdao.com/yws/api/personal/file/WEBecbfb16db6ad95575f02ffa9f35b1caa?method=download&shareKey=a70d607348a9f73fc20279324786b87b"/>
  </div>

2. **pseudo box**

   **通过1步骤生成的 RepPoints再通过如下步骤生成pseud box**

   - **Min-max function，通过选取这些点对的最大最小值来生成pseudo box**
   - **Partial min-max function，通过选取部分点对的最大最小值来生成pseudo box（具体选哪些不太清楚，可能是为了滤除一些极端值？？？）**
   - **Moment-based function，通过计算这些点对的均值方差来生成pseud box**

### 2.2 正负样本选择

#### 2.2.1 回归正样本

- **stage-1**

  **第一阶段的正负样本选择有两步：**

  - **判断改目标在哪一个特征层：**
    
    <div align=center>
    <img src="https://note.youdao.com/yws/api/personal/file/WEB0b25399a6367ad2ef16b846cd3954835?method=download&shareKey=58ad2f564cce6386624d795765c4a35f"/>
  </div>
    
  - **然后判断gt的中心点落在哪个特征点上，则改特征层的该特征点为正样本**

- **stage-2**

  **如果第一阶段生成的pseudo box与gt的iou大于0.5就认为是正样本**

#### **2.2.2 分类正样本**

​	**分类只针对第一阶段，第一阶段生成的pseudo box如果与gt iou > 0.5就为正样本，< 0.4就为负样本，其余的为忽略样本。**




## 3.总结：

**RepPoints的核心在于它的网络结构是回归点对，然后再通过点对生成pseudo box，这样就可以省去大量的有关anchor的超参，通过玩网络自己学习这些目标的核心关键点。这个做法是非常新颖的，有一篇目标跟踪的文章也用了类似的思想，让网络自己去学习一些关键区域作为reid特征，整个框架也十分简洁。但是我觉存在的问题是由于编码方式是直接加上预测偏置，没有进行压缩，这样对于尺度变化特别大的目标检测效果可能不是太理想，网络的收敛速度可能比较慢。**

