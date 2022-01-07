

# You Only Look Once for Panoptic Driving Perception

## 1.概述

**yolop是一个多任务学习框架，包含目标检测和车道线道路分割。**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB3122e9aff919bacaeac1e324408fabb5?method=download&shareKey=d5bf0575f77636f0f43579494776544e"/>.
</div>



**图1**

## 2.具体实现

### 2.1. Bckbone：

**Yolop的backbone为CSPDarknet**



### 2.2. Neck：

**yolop的neck部分为FPN+SPP**



### 2.3. Head:

#### 2.3.1 Detection Head

**Detection Head与yolo4一样，添加了PANet module**

#### 2.3.2 Segment Head

**Segment Head是在FPN8倍下采样层基础上上采样至输入分辨率，输出两通道**



### 2.4 Loss

#### 2.4.1 Det Loss

**detection loss分为如下几部分**

- **class loss 采用focal loss**
- **obj loss采用focal loss**
- **box loss采用CIOU loss**

#### 2.4.2 Segment loss

**segment loss采用ce loss + iou loss**



## 3.总结

**yolop属于一个组合式的论文整体没有什么创新，一个典型的多任务学习范式，应该很多人都尝试过这种方法，可能别人是在retinanet、efficientdet等基础上进行多任务学习。工程属性还是很强的，看试验结果在Jetson TX2上运行时间可以达到24ms/fps（没找到输入是多大）**



