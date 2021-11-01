# **Mask R-CNN**

## 1.概述

**Mask R-CNN是在faster RCNN基础上添加mask分支，同时提出ROI-Align来消除ROI-Pooling的量化误差，并添加FPN结构大大提升效果。**

**paper link:https://arxiv.org/pdf/1703.06870.pdf**

**code link:https://github.com/facebookresearch/Detectron**

## 2.具体实现

### 2.1 网络结构

- **backbone采用resnet系列**
- **neck部分采用FPN+RPN**
- **head为faster-rcnn的分类和回归head+rpn后面接mask head**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBcc5a3d318e2eab22d3beda07cb93d349?method=download&shareKey=98fc717284335b7c37425c88e5e821c8"/>.
</div>

### 2.2 mask branch细节

**mask分支通过对RPN阶段生成的ROI通过ROI-Align操作生成m*m大小的proposal，再把这些prorosal通过卷积成K(类别数)个维度**



### 2.3 ROI Align

**roi-align是争对roi-pooling提出，roi-pooling会产生两次量化误差：**

- **第一次是bbox在特征图位置的量化误差,例如图片大小为(225,225)，特征图16倍下采样，那么坐标(x,y)在特征图为位置为(x * 16 / 255, y * 16 / 255)，坐标会向下取整。**

- **第二次是roi-pooling的时候会把bbox划分为n*n个bin，如下图所示，一个宽高为(7，5)大小的bbox，分成2 * 2大小的bin，由于不能整除，会按下图向上取整和向下取整的方式去划分：**

  <div align=center>
  <img src="https://pic3.zhimg.com/v2-d59199c554ac7ccfbd2038317aef74f2_r.jpg"/>.
  </div>

**由于两次量化误差的存在随着下采样倍数的提升，误差偏差会变得非常大，如果是分类这类问题对位置信息不是很敏感可能体现的不是很明显，但是检测和分割这类问题对位置信息很敏感就会严重影响效果。因此提出了roi-align**

**roi align就是也是利用roi pooling的思想，但是不进行量化取整，如下图所示：**

<div align=center>
<img src="https://pic4.zhimg.com/v2-03d820b27ffca39854f0febf0ef9e37b_r.jpg"/>.
</div>

- **在每个bin里面选取n个点(对应上图右边每个bin 4个蓝色的点)**
- **然后再通过双线性插值求得这4个点的值**
- **通过对n个bin里面的数值进行max pooling or avg pooling**



### 2.4 mask traget选取

**在训练过程中mask target为roi与groud truth的交集，即只选取roi里面的mask的ground truth作为训练target。**



### 2.5 Inference

**inference的时候不同于训练时仅用rpn部分产生的proposal经过mask分支，而是通过faster-rcnn最终输出的检测框作为mask分支的输入。最后仅选取类别分支输出的指定类别那一维度的mask。**





## 3.感想

**看mask rcnn这篇论文的时间比较晚，虽然现在有各种花式网络各种新的技巧出现也丝毫不能影响mask rcnn在实例分割中的地位，添加mask分支、采用fpn、提出roi align，该文章的工作非常扎实实用，这种端对端的多任务学习也会在一定程度上对检测效果带来提升。**