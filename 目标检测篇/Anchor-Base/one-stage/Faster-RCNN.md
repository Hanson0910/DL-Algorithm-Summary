# Faster R-CNN: Towards Real-Time Object Detection with Region Proposal Networks

## 概述：

**Faster-RCNN是在Fast-RCNN基础上设计RPN层替换选来的selective search方式生成ROI，达到真正的end-to-end的训练方式，在速度和精度上都有很大提升。**

## 具体实现

### Backbone:

**作者论文实验中使用的时ZFNet和VGG，仅取16被下采样后的特征层作为最终RPN和RCNN的特征提取层。**

### Neck：

**Neck部分这里即为RPN部分，RPN网络如下图所示：rpn_cls_scrore、rpn_bbox_pred的大小分别为[B,N * 2,H/16,W/16],[B,N * 4,H / 16,W / 16]，N为单点anchor个数，默认为9个，H,W分别为输入的高和宽。B为batch-size大小。最终输出的proposal有scores和rois组成。**

![image](https://note.youdao.com/yws/api/personal/file/WEBc93f137ae20ce47b809bae894147b707?method=download&shareKey=79acbbc71cb6dfd047415ab3388a0990)

