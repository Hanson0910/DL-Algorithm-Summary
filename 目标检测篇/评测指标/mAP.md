### Pascal-mAP

- ##### 相关定义
    
mAP: mean Average Precision, 即各类别AP的平均值;
    AP: PR曲线下面积; 
    PR曲线: Precision-Recall曲线段；
    目标检测中的召回、进度定义如下图所示：

![image](https://note.youdao.com/yws/api/personal/file/WEB32b532e547a2140d3b39415f1945f23d?method=download&shareKey=3ad63af42397bbb84584fef9fddfd46c)

- ### P-R曲线
    Precision-Recall曲线，简称P-R曲线，其横轴是召回率，纵轴是精确率。

- ### AP计算
    AP即PR曲线的面积，计算方式有:
    - 积分求解
    - 插值求解

- ### VOC-AP计算

- P-R曲线图

    **对于某类下全部的真实目标，将IOU>=0.5的作为检测出来的目标，取不同的confidence 阈值计算对应的precision和recall。**
![image](https://note.youdao.com/yws/api/personal/file/WEBc4a87e63f151aa09b000df4b8ba7e0c4?method=download&shareKey=82b5f9d0b1c0112da4c2924c876d557e)
- #### 2010之前
    取recall >= [0,0.1,0,2,0,3,0,4,0.5,0.6,0.7,0.8,0.9,1.0]共11个点时的precision最大值，然后AP就是这11个precision的平均值.如下图红色点所示:

![image](https://note.youdao.com/yws/api/personal/file/WEBdfdc186999291fa106abc89310986f6a?method=download&shareKey=a40d40ac4998f7f54fec2ff5d7bccbd0)

假设召回为0时的precision即取在recall>=0时的最大精度作为该点的precision，召回为0.1时的precision即取在recall>=0.1时的最大精度作为该点的precision。

- #### 2010之后
    不再是对召回率在[0,1]之间的均匀分布的11个点，而是对每个不同的recall值都计算一个precision，然后再求平均。每个点的precision计算与之前一样，都取recall>=该点时的最大precision作为该点的precision。如下图红色部分所示:

![image](https://note.youdao.com/yws/api/personal/file/WEB3de0a00b9747160a163685de6be5c670?method=download&shareKey=ab508664f99d237810d6b7718ae8e637)


- ### mAP
    mAP就是多个类别AP的平均。



### coco-mAP

**在评测时，COCO评估了在不同的交并比(IoU)[0.5:0.05:0.95]共10个IoU下的AP，并且在最后以这些阈值下的AP平均作为结果，记为mAP@[.5, .95]。 而在Pascal VOC中，检测结果只评测了IOU在0.5这个阈值下的AP值。 因此相比VOC而言，COCO数据集的评测会更加全面：不仅评估到物体检测模型的分类能力，同时也能体现出检测模型的定位能力。因此在IoU较大如0.8时，预测框必须和真实的框具有很大的重叠比才能被视为正确。**