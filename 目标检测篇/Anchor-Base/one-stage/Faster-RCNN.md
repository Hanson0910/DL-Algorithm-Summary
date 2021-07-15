# Faster R-CNN: Towards Real-Time Object Detection with Region Proposal Networks

## 概述：

**Faster-RCNN是在Fast-RCNN基础上设计RPN层替换选来的selective search方式生成ROI，达到真正的end-to-end的训练方式，在速度和精度上都有很大提升。网络结构如下图所示：**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBd38dafd37fe793094027722946434e50?method=download&shareKey=11b70155c90c4da41ba28a6e0f4e8307"/>
</div>

​                                                                                                                                      **图1**

## 具体实现

### 1. Backbone:

**作者论文实验中使用的时ZFNet和VGG，仅取16被下采样后的特征层作为最终RPN和RCNN的特征提取层。如图1中backobone所示。**



### 2. Neck：

**Neck部分这里即为RPN部分，RPN网络如图1中Neck部分所示，具体如图2所示：rpn_cls_scrore、rpn_bbox_pred的大小分别为[B,N * 2,H/16,W/16],[B,N * 4,H / 16,W / 16]，N为单点anchor个数，默认为9个，H,W分别为输入的高和宽，B为batch-size大小。RPN阶段提取ROI只有正例和负例两个类别，所以分类分支的维度为2*N，同时每一个ROI需要回归中心点坐标(x,y)和宽高(w,h)，所以bbox分支维度为N *4。最终输出的proposal有scores和rois组成**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBc93f137ae20ce47b809bae894147b707?method=download&shareKey=79acbbc71cb6dfd047415ab3388a0990"/>
</div>

​                                                                                                                              **图2**



### 3. Head：

**RPN阶段刷选出rois在conv5_3层中经过ROI-Pooling操作把每一个roi提取的特征固定到形同大小(默认为7x7)，然后再把该特征分别经过分类和回归分支的全连接层输出类别和坐标。如图1中Head所示。**



### 4. Sample Assignment:

- **rpn sample assignment**
  1. 对rpn proposal的scores进行排序，选取前RPN_PRE_NMS_TOP_N个，一般默认RPN_PRE_NMS_TOP_N为12000;
  2. 对前RPN_PRE_NMS_TOP_N个proposal进行nms，nms阈值为RPN_NMS_THRESH（默认为0.7);
  3. 选取nms后的前RPN_POST_NMS_TOP_N（默认为2000）个proposal;
- **rcnn sample assignment**
  1. 计算2000个proposals+gts（后面还是用proposals表示）与所有gts的ious
  2. 找出每个proposal与哪个gt的iou最大，并记录其iou值，如果iou值大于FG_THRESH(默认为0.5)，则设置该proposal为正样本。
  3. 如果该iou值<FG_THRESH并 >BG_THRESH(默认为0.) 的proposal则设置为负样本
  4. 控制正负样本的个数及比例，默认设置正负样本比例为1:3且正负样本总数为256，实际正样本如果大于64则随机选取64个，如果为0则随机选取256个负样本。



### 5. Decode:

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBbd29d7a9fc1366d5526dc1a49fb87928?method=download&shareKey=2c4a6940abd2a04648becf90d0f0e4f0"/>
</div>

从该编码方式来看，编码后的值被压缩有助于网络快速收敛。

### 6. Loss:

Faster-RCNN分为RPN和RCNN两个部分，两个部分单独优化，RCNN部分上面讨论了选取256个mini-batch的proposals，RPN部分选取前RPN_PRE_NMS_TOP_N个proposals后通过下面也选取mini-bach个proposals，mini-bach的数量由RPN_BATCHSIZE控制:

1. 计算前RPN_POST_NMS_TOP_N个proposal与gts的ious;
2. iou小于RPN_NEGATIVE_OVERLAP(默认0.3)的proposal为负样本，iou大于 RPN_POSITIVE_OVERLAP(默认0.7)的proposal为正样本。需要注意的是gt与anchor最大的IOU，不 管是否大于RPN_POSITIVE_OVERLAP始终设置为正样本。其余设置为忽略样本(-1);
3. 选择正负样本个数，由RPN_FG_FRACTION和RPN_BATCHSIZE两个变量控制，RPN_FG_FRACTION是负样本比例默认0.5,RPN_BATCHSIZE为该批次正负样本个数，默认为256。正负样本最大个数为128，当正样本大于128时，随机选取128个正样本，负样本个数始终为RPN_BATCHSIZE-实际正样本个数。

两部分的Loss公式为:

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB41697d7dbc87f275135cf40b079cd4e1?method=download&shareKey=709e7d57c2c7b1012ad9f898fa19cef4"/>
</div>

**分类loss用的交叉熵loss，回归loss用的smooth-L1 loss。**

### 7. 总结

Faster-RCNN的作者从RCNN,Fast-RCNN一步一个脚印，做的工作非常扎实，RPN的提出使得网络能够完全以end-to-end的方式进行训练，anchor的提出为后来anchor-base系列的目标检测奠定了基础。现在各种主干网络，loss，FPN等提出使得Faster-RCNN在原来基础上又能够有一个很大的进步。
