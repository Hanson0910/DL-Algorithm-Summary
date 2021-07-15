# Faster R-CNN: Towards Real-Time Object Detection with Region Proposal Networks

## 概述：

**Faster-RCNN是在Fast-RCNN基础上设计RPN层替换选来的selective search方式生成ROI，达到真正的end-to-end的训练方式，在速度和精度上都有很大提升。网络结构如下图所示：**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBd38dafd37fe793094027722946434e50?method=download&shareKey=11b70155c90c4da41ba28a6e0f4e8307"/>
</div>

​                                                                                                                                      **图1**

## 具体实现

### Backbone:

**作者论文实验中使用的时ZFNet和VGG，仅取16被下采样后的特征层作为最终RPN和RCNN的特征提取层。如图1中backobone所示。**



### Neck：

**Neck部分这里即为RPN部分，RPN网络如图1中Neck部分所示，具体如图2所示：rpn_cls_scrore、rpn_bbox_pred的大小分别为[B,N * 2,H/16,W/16],[B,N * 4,H / 16,W / 16]，N为单点anchor个数，默认为9个，H,W分别为输入的高和宽，B为batch-size大小。RPN阶段提取ROI只有正例和负例两个类别，所以分类分支的维度为2*N，同时每一个ROI需要回归中心点坐标(x,y)和宽高(w,h)，所以bbox分支维度为N *4。最终输出的proposal有scores和rois组成。**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBc93f137ae20ce47b809bae894147b707?method=download&shareKey=79acbbc71cb6dfd047415ab3388a0990"/>
</div>

​                                                                                                                              **图2**



### Head：

**RPN阶段刷选出rois在conv5_3层中经过ROI-Pooling操作把每一个roi提取的特征固定到形同大小(默认为7x7)，然后再把该特征分别经过分类和回归分支的全连接层输出类别和坐标。如图1中Head所示。**



### Sample Assignment:

- **rpn sample assignment**
  1. 对rpn proposal的scores进行排序，选取前RPN_PRE_NMS_TOP_N个，一般默认RPN_PRE_NMS_TOP_N为12000;
  2. 对前RPN_PRE_NMS_TOP_N个proposal进行nms，nms阈值为RPN_NMS_THRESH（默认为0.7);
  3. 选取nms后的前RPN_POST_NMS_TOP_N（默认为2000）个proposal;
  4. 计算前RPN_POST_NMS_TOP_N个proposal与gts的ious;
  5. iou小于RPN_NEGATIVE_OVERLAP(默认0.3)的proposal为负样本，iou大于 RPN_POSITIVE_OVERLAP(默认0.7)的proposal为正样本。需要注意的是gt与anchor最大的IOU，不 管是否大于RPN_POSITIVE_OVERLAP始终设置为正样本。其余设置为忽略样本(-1);
  6. 选择正负样本个数，由RPN_FG_FRACTION和RPN_BATCHSIZE两个变量控制，RPN_FG_FRACTION是负样本比例默认0.5,RPN_BATCHSIZE为该批次正负样本个数，默认为256。正负样本最大个数为128，当正样本大于128时，随机选取128个正样本，负样本个数始终为RPN_BATCHSIZE-实际正样本个数。
- **rcnn sample assignment**
  1. 计算2000个proposals+gts（后面还是用proposals表示）与所有gts的ious
  2. 找出每个proposal与哪个gt的iou最大，并记录其iou值，如果iou值大于FG_THRESH(默认为0.5)，则设置该proposal为正样本。
  3. 如果该iou值<FG_THRESH并 >BG_THRESH(默认为0.) 的proposal则设置为负样本
  4. 控制正负样本的个数及比例，默认设置正负样本比例为1:3且正负样本总数为256，实际正样本如果大于64则随机选取64个，如果为0则随机选取256个负样本。



### Decode

1. 宽高编码

   ```math
   \begin{aligned}
   t_{\mathrm{x}} &=\left(x-x_{\mathrm{a}}\right) / w_{\mathrm{a}}, \quad t_{\mathrm{y}}=\left(y-y_{\mathrm{a}}\right) / h_{\mathrm{a}} \\
   t_{\mathrm{w}} &=\log \left(w / w_{\mathrm{a}}\right), \quad t_{\mathrm{h}}=\log \left(h / h_{\mathrm{a}}\right) \\
   t_{\mathrm{x}}^{*} &=\left(x^{*}-x_{\mathrm{a}}\right) / w_{\mathrm{a}}, \quad t_{\mathrm{y}}^{*}=\left(y^{*}-y_{\mathrm{a}}\right) / h_{\mathrm{a}} \\
   t_{\mathrm{w}}^{*} &=\log \left(w^{*} / w_{\mathrm{a}}\right), \quad t_{\mathrm{h}}^{*}=\log \left(h^{*} / h_{\mathrm{a}}\right)
   \end{aligned}
   ```