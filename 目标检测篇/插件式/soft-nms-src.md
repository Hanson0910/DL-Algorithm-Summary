# Soft nms:Improving Object Detection With One Line of Code

## 概述：

**softnms仅通过在原nms基础上改变其中一个步骤，把原来nms滤除周边检测框改为降低其检测得分来提高检测精度。**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB8439514b81340cff412ebe7340e40ad1?method=download&shareKey=357d27759092b708004be4c5659f2dae"/>
</div>                                                                                                                                      

  **图1**

## 提出背景

**目标检测会得到大批密集的检测款，为了得到最终的检测框主流的检测框架都是通过一个非极大值抑制(nms)的方式来滤除大量冗余检测框，具体步骤如下图所示：**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBe567c2e88a7f1a3f9e8cfc4c6aac4c9a?method=download&shareKey=48f04c96118773bb55d849f71374c4ff"/>
</div>   



**采用hard计算的方式(下式所示)会在现有评测指标(map,即iou阈值0.5~1.0之间的均值)中存在如下问题：**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB43aac59bfee083867eea4811df28f238?method=download&shareKey=1665c932d9bd097fa4ae12a7ec7eb0a4"/>
</div>   

- **阈值过低会增加漏检，这个比较好理解，假设阈值设置为0.3，有一个检测框与目标的iou很大，但是得分比较低，但是有一个检测框分数很高，但是压根就没有conver住目标，但是这个得分高的检测框又与得分低的检测框有一部分重叠，这时iou阈值卡的低的话这种情况会很频繁发生，导致miss rate增加**
- **阈值过高会增加误检查，检测精度=TP/TP+FP，当阈值过高时由于检测框的数量远远多于目标，所以FP增加的概率远远大于TP，所以会导致精度降低**。

**所以作者采用了一种软化得分的方式，不直接滤除检测框，而是根据IOU的大小来相应惩罚检测得分，iou越大的惩罚越严厉，iou越小惩罚越小，有两种形式：**

- **根据iou阈值来惩罚：**

  <div align=center>
  <img src="https://note.youdao.com/yws/api/personal/file/WEB8d86c9f23c569461041ef55ea0569d9f?method=download&shareKey=9da15b1ba6576b64f95dc56d047f4635"/>
  </div>  

- **通过高斯分布来连续惩罚：**

  <div align=center>
  <img src="https://note.youdao.com/yws/api/personal/file/WEB5ef5d4b892f4ba62c2063b6a50f47ad6?method=download&shareKey=bfd9223bf5656ba593e9af32bed383ad"/>
  </div>  

# 总结

**soft nms与原nms相比几乎没有额外计算量，通过软化得分的方式最后再根据得分来滤除冗余检测在mAP这种评测方式中应该会有一点提升。这个最终的得分阈值还是要通过手工去调整，这一块应该还能再优化一下。**