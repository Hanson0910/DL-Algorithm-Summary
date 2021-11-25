# DL-Algorithm-Summary
**该仓库记录本人平时阅读一些论文和方法的总结，也为了督促自己多学习多分享，有些内容也会同步分享到[知乎](https://www.zhihu.com/people/feng-yi-54-69/posts),所以有些链接直接链到我的知乎账号。具体内容可点击readme下的超链接查看，链接中有些图片外链可能需要较长加载时间。如果觉得对你帮助，给我一个star吧！！！**

# 目标检测篇
## Anchor-Base

### two-stage

1. [Faster R-CNN: Towards Real-Time Object Detection with Region Proposal Networks，在Fast-RCNN基础上设计RPN层替换选来的selective search方式生成ROI，达到真正的end-to-end的训练方式](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Base/two-stage/Faster-RCNN.md)
2. [R-FCN在Faster-RCNN的基础上提出position-sensitive RoI pooling layer，不需要每一个roi单独提取特征再进行全连接操作进行分类和回归，R-FCN是全  	卷积网络能够提升训练和测试阶段的检测速度](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Base/two-stage/R-FCN.md)
3. [Double-Head RCNN：Rethinking Classifification and Localization for Object Detection,fc层对全局空间信息掌握的更好，卷积能够对局部位置信息掌握更好，作者利用这个发现分别用fc和conv来进行分类和回归。然后提出混合任务，fc层再回归框，conv层再进行分类进行辅助互补](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Base/two-stage/Double-Head-RCNN.md)
4. [Cascade R-CNN通过级联的方式一级一级筛选出高质量的proposal,提升train和inference的精度](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Base/two-stage/Cascade-RCNN.md)
5. [Libra R-CNN现有two-satge检测框架得问题分别从样本层面、特征层面和目标层面出发，提出IOU-balanced sampling，balanced feature pytamid，balanced L1 loss。主要解决选取高质量样本、更加有效融合mult-stage特征和更加关注难例的学习](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Base/two-stage/Libra%20R-CNN.md)
6. [Dynamic R-CNN 还是在样本选取的角度出发，认为网络在不同训练阶段porposal的分布会不一样，如果还是以一样的状态去选取样本会损害网络精度。结合cascade-rcnn，认为cascade-rcnn太耗时，提出了动态分配样本的方法，通过统计object与proposal的统计学特征来动态分配样本](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Base/two-stage/Dynamic-RCNN.md)

  ### ont-stage
1. [SSD:Single Shot MultiBox Detector,，通过特征金字塔能够检测不同大小的目标，没有RPN步骤，采用全卷积的方式能够极大提升检测速度。使用hard_negative_mining可以有效解决正负样本不均很的问题，同时加速收敛速度](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Base/one-stage/SSD.md)
2. [RetinaNet,采用FNP结构并利用focal-loss来解决正负样本不均衡的问题](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Base/one-stage/RetinaNet.md)
3. [EfficientDet,EfficientDet采用BIFPN结构在替身检测速度的同时增加检测精度](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Base/one-stage/EfficientDet.md)
4. [yolo4,采用CSP模块能够有效减小网络计算量](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Base/one-stage/Yolo4.md)
5. [yolo5,采用首先采用了focus层，然后设计了两种CSP结构，能够进一步减小网络计算量](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Base/one-stage/Yolo5.md)
6. [Gradient Harmonized Single-stage Detector,分析了单阶段目标检测中的样本不均衡问题，通过梯度分布来动态为不同样本分配不同的梯度。这样原本分类中占据大量梯度的负例和easy-sample就会减少它们的梯度，占据少量的very-hard sample虽然样本比较少，但是却单个梯度很大，这样也会减小它们的梯度，medium-sample就会相对增加梯度。针对回归提出了改进版的smootl-l1 loss使得在预测值和gt差值大于阈值的时候个样本梯度范数不再统一是1，能够反映不同样本的重要程度，同时针对性提出GHM-R Loss来较小very-hard sample的影响](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Base/one-stage/GHM.md)

## Anchor-Free

### Key-Point
1. [DenseBox,DenseBox的思想十分超前，采用多层特征融合，多任务学习等未后续的anchor-free打下了基础](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Free/Key-Point/Densebox.md)
2. [Yolo,Yolo没有复杂的检测流程，只需要将图像输入到神经网络就可以得到检测结果，YOLO可以非常快的完成物体检测任务](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Free/Key-Point/Yolo.md)
3. [CornerNer,将目标检测问题看作关键点回归问题，通过检测目标框的左上角和右下角两个关键点得到预测框。整个检测网络的训练是从头开始的，并不基于预训练的分类模型，这使得用户能够自由设计特征提取网络，不用受预训练模型的限制](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Free/Key-Point/CornerNet.md)
4. [RepPoints使用一个完全anchor-free的检测框架，通过deformable-conv预测9组点来代表目标的重点区域，再通过这9组点生成伪box进行回归。RepPoints达到了真正的anchor-free，整个检测框架十分简洁，超参少。](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Free/Key-Point/RepPoints-SRC.md)
5. [ATSS通过对比试验发现造成anchor-base和anchor-free精度最大的区别是在正负样本选择上，进而通过分析目标统计学特征提出自动选择正负样本的方法，可以大幅提升检测精度](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Free/Key-Point/FoveaBox.md)


### Center-Point
1. [CenterNet,centernet预测中心点、宽高和偏置，无IOU、无nms，整个框架十分简介](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Free/Anchor-Point/CenterNet.md)
2. [FASF-Feature Selective Anchor-Free Module for Single-Shot Object DetectionFeature Selective Anchor-Free Module for Single-Shot Object Detection,提出了feature selective anchor-free module来解决这个限制，让每个示例动态选择最适合自己的特征层级，并且FASF可以与anchor-based 分支协同并行工作，提高模型效果。](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Free/Anchor-Point/FASF.md)
3. [FCOS,FCOS基于FCN的逐像素目标检测算法，实现了无锚点（anchor-free）、无提议（proposal free）的解决方案，并且提出了中心度（Center—ness）的思想，同时在召回率等方面表现接近甚至超过目前很多先进主流的基于锚框目标检测算法](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Free/Anchor-Point/FCOS.md)
4. [Soft Anchor-Point Object Detection, 实例在不同特征层都会有相应只是响应的大小不一样，所以用了一个软标签使得正样本在不同特征层之间都有一定的梯度回传](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Free/Anchor-Point/Soft%20Anchor-Point%20Object%20Detection.md)
5. [YOLOX,YOLOX在YOLO的基础上把cls,pbj,box分支解耦开来，同时采用anchor-free的框架并简化OTA提出SimOTA加快训练时间](https://zhuanlan.zhihu.com/p/395554928)

## 插件式
1. [Dynamic Head: Unifying Object Detection Heads with Attentions,这是微软发表关于目标检测的论文,可以作为一个插件集成到现有目标检测框架中：One-Stage; Two-Stage; Anchor Base; Anchor Free,Dynamic Head主要解决目标检测的三个问题（我觉得第三个有点牵强）：尺度感知、空间感知、任务感知](https://zhuanlan.zhihu.com/p/381481382)
2. [Generalized Focal Loss:Learning Qualified andDistributed Bounding Boxes for Dense Object Detection,文章提出Quality Focal Loss，通过软化标签用IOU值替换one-hot标签，再利用focal-loss的形式组成分类loss；Distribution Focal Loss还是没有理解透彻，大致是围绕坐标标签定义一个离散空间，然后把这个离散空间通过softmax函数表示每个离散空间可能是坐标的概率，同事为了加快收敛速度只计算坐标左右两点的loss。](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/loss%E7%AF%87/Generalized%20Focal%20Loss.md)
3. [FreeAnchor: Learning to Match Anchors for Visual Object Detection,FreeAnchor从极大释然估计的角度出发设计优化方式，使得网络能够以一个更加灵活的方式去选择目标匹配的anchor。不同于guide-anchor从网络设计出发，FreeAnchor从优化方式出发，利用巧妙的数学技巧达到anchor自适应object的目的](https://zhuanlan.zhihu.com/p/433873412)
4. [ATSS通过对比试验发现造成anchor-base和anchor-free精度最大的区别是在正负样本选择上，进而通过分析目标统计学特征提出自动选择正负样本的方法，可以大幅提升检测精度](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Free/Key-Point/ATSS-SRC.md)
5. [CARAFE,Content-Aware ReAssembly of FEatures，这是一篇关于特征上采样的文章，其实不应该放在目标检测上，可以用在分割等其它任务中，这篇文章我觉得故事成的成分很大，分析了传统插值和deconvolution的缺点，认为传统插值精度不好，deconvolution比较费时，为此提出了一种新的上采样策略](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Base/two-stage/CARAFE-SRC.md)
6. [softnms仅通过在原nms基础上改变其中一个步骤，把原来nms滤除周边检测框改为降低其检测得分来提高检测精度](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/%E6%8F%92%E4%BB%B6%E5%BC%8F/soft-nms-src.md)


## 数据集
1. [MS-COCO](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/%E6%95%B0%E6%8D%AE%E9%9B%86/ms-coco.md)
2. [PASCAL-VOC](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/%E6%95%B0%E6%8D%AE%E9%9B%86/pascal-voc.md)



### 评测方式
1. [mAP](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/%E8%AF%84%E6%B5%8B%E6%8C%87%E6%A0%87/mAP.md)



# 网络篇
1. [GCNet通过分析No-Local(Non-local Neural Networks)网络，实现发现No-Local网络用了大量的计算量来建立的特征长程关系，但是却发现通过No-local网络来建模的全局语义信息在不同的点上几乎是相同的。对于这一现象，作者进行了改进](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%BD%91%E7%BB%9C%E7%AF%87/GCNet.md)


# 分割篇

## 实例分割

1. [Mask R-CNN是在faster RCNN基础上添加mask分支，同时提出ROI-Align来消除ROI-Pooling的量化误差，并添加FPN结构大大提升效果](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E5%88%86%E5%89%B2%E7%AF%87/%E5%AE%9E%E4%BE%8B%E5%88%86%E5%89%B2/mask-rcnn.md)
2. [Mask Scoring R-CNN是在Mask R-CNN基础上添加MaskIOU分支增加分类准确性](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E5%88%86%E5%89%B2%E7%AF%87/%E5%AE%9E%E4%BE%8B%E5%88%86%E5%89%B2/Mask-Scoring-R-CNN.md)
3. [Hybrid Task Cascade for Instance Segmentation在cascade mask rcnn基础上进行改进，充分利用detection与segmentation的联系，使得detection与segmentation由之前的并行学习改为串行学习，让segmentation能够学习到detection信息，同时也添加语义分割模块辅助训练进一步提升网路性能。](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E5%88%86%E5%89%B2%E7%AF%87/%E5%AE%9E%E4%BE%8B%E5%88%86%E5%89%B2/Hybrid%20Task%20Cascade%20for%20Instance%20Segmentation.md)
4. [InstaBoost是一种数据增广方式，用在示例分割和目标检测上。通过把实例crop出来再随机放置在图像背景区域来达到数据增广提升检测精度的目的。](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E5%88%86%E5%89%B2%E7%AF%87/%E5%AE%9E%E4%BE%8B%E5%88%86%E5%89%B2/InstaBoost.md)



## 语义分割



# 目标跟踪篇

1. [RetinaTrack: Online Single Stage Joint Detection and Tracking，RetinaTrack主要解决检测区域高度重叠的时候导致实例特征混淆，从而在RetinaNet基础上对每一个anchor用一个单独的分支去负责预测，Task-Shared和Task-Specfic部分主要是为了在减小模型参数量的基础上提升多任务学习的精度](https://zhuanlan.zhihu.com/p/269571970)

# Loss篇

## 分类Loss
1. [交叉熵Loss，多分类任务中通常用到交叉熵Loss作为代价函数，分类交叉熵损失求导更简单，损失仅与正确类别的概率有关](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/loss%E7%AF%87/%E5%88%86%E7%B1%BBloss.md)
2. [Focal-Loss，Focal-Loss能够更加关注难例](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/loss%E7%AF%87/%E5%88%86%E7%B1%BBloss.md)
3. [Gradient Harmonized Single-stage Detector,分析了单阶段目标检测中的样本不均衡问题，通过梯度分布来动态为不同样本分配不同的梯度。这样原本分类中占据大量梯度的负例和easy-sample就会减少它们的梯度，占据少量的very-hard sample虽然样本比较少，但是却单个梯度很大，这样也会减小它们的梯度，medium-sample就会相对增加梯度](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Base/one-stage/GHM.md)
4. [Bi-Tempered Logistic Loss, 可以在一定程度上解决错误样本标记带来的问题，通过改进log函数和exp函数，添加两个tempered变量减轻错误样本的影响](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/loss%E7%AF%87/%E5%88%86%E7%B1%BBloss.md)

## 回归Loss
1. [L1 Loss, 不论对于什么样的输入值，都有着稳定的梯度，不会导致梯度爆炸问题，对离群点不敏感，具有较为稳健性的解](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/loss%E7%AF%87/%E5%9B%9E%E5%BD%92%E6%8D%9F%E5%A4%B1%E5%87%BD%E6%95%B0.md)
2. [L2 Loss, 各点都连续光滑，方便求导，而且梯度值也是动态变化的，能够快速的收敛，得到较为稳定的解](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/loss%E7%AF%87/%E5%9B%9E%E5%BD%92%E6%8D%9F%E5%A4%B1%E5%87%BD%E6%95%B0.md)
3. [smooth-l1 loss, 综合了L1和L2的优点，相比于L2损失函数，对离群点、异常值使用L1，梯度变化相对更小，训练时不容易跑飞。当预测值与真是差异很小时，使用L2损失，快速得到较为稳定的解](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/loss%E7%AF%87/%E5%9B%9E%E5%BD%92%E6%8D%9F%E5%A4%B1%E5%87%BD%E6%95%B0.md)
4. [iou loss, 使用 IOU LOSS可以把回归框当成一个整体去优化](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/loss%E7%AF%87/%E5%9B%9E%E5%BD%92%E6%8D%9F%E5%A4%B1%E5%87%BD%E6%95%B0.md)
5. [GIOU Loss,预测值和 gt 不重叠时也可以优化,可以分辨不同的重叠方式](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/loss%E7%AF%87/%E5%9B%9E%E5%BD%92%E6%8D%9F%E5%A4%B1%E5%87%BD%E6%95%B0.md)
6. [DIOU Loss,直接最小化预测框与目标框之间的归一化距离，以达到更快的收敛速度。](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/loss%E7%AF%87/%E5%9B%9E%E5%BD%92%E6%8D%9F%E5%A4%B1%E5%87%BD%E6%95%B0.md)
7. [CIOU Loss, 可以在DIOU Loss基础上回归宽高比差异](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/loss%E7%AF%87/%E5%9B%9E%E5%BD%92%E6%8D%9F%E5%A4%B1%E5%87%BD%E6%95%B0.md)
8.  [Gradient Harmonized Single-stage Detector,提出改进Smooth-L1 Loss，使得smootl-l1 loss使得在预测与gt差值大于阈值的时候单个样本梯度范数不再统一是1，能够反映不同样本的重要程度，同时针对性提出GHM-R Loss来较小very-hard sample的影响。](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Base/one-stage/GHM.md)

# 优化篇

1. [正则项,加入正则项能有效避免模型过拟合同时使模型更加稀疏](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E4%BC%98%E5%8C%96%E7%AF%87/%E6%AD%A3%E5%88%99%E9%A1%B9.md)
2. [优化器，优化器就像灯塔指引网络往正确的方向收敛，一个好的优化器能够帮助网络快速收敛，跳出局部最优，训练过程更加稳定](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E4%BC%98%E5%8C%96%E7%AF%87/%E4%BC%98%E5%8C%96%E5%99%A8.md)
3. [激活函数，激活函数能为网络增加非线性使网络有更好的拟合能力](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E4%BC%98%E5%8C%96%E7%AF%87/%E6%BF%80%E6%B4%BB%E5%87%BD%E6%95%B0.md)
4. [BN,BN能够有效减小神经网络训练过程中产生的Internal Covariate Shift问题](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E4%BC%98%E5%8C%96%E7%AF%87/Batch-Normilize.md)

# 竞赛篇
#### Kaggle

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBfdbcdec88218d0ae4f0c39e8e0bad4c6?method=download&shareKey=547df1fa26838383496ff7f4a40ab081"/>
</div>

1. [Kaggle 木薯叶疾病识别银牌，top1% 29/3900](https://www.kaggle.com/c/cassava-leaf-disease-classification)
2. [Kaggle 化学结构式翻译，top4% 30/874](https://www.kaggle.com/c/bms-molecular-translation)
3. [Kaggle shopee商品识别银牌，top5%,118/2426](https://www.kaggle.com/c/shopee-product-matching)
4. [Kaggle 导管放置位置识别铜牌,top6%,99/1547](https://www.kaggle.com/c/ranzcr-clip-catheter-line-classification)
#### else
1. [ISBI-2021 RIADD 视网膜疾病分类第一名](https://riadd.grand-challenge.org/evaluation/challenge/leaderboard/)
<div align=center>
<img src="https://github.com/Hanson0910/Pytorch-RIADD/blob/main/show-img/show-img.png" width = "480" height = "240" />
</div>

# 工程应用篇

1. [轻量级人脸检测MNN模型部署](https://github.com/Hanson0910/MNNFaceDetect)
2. [libtorch多batch inference](https://zhuanlan.zhihu.com/p/386108526)
3. [关键点匹配算法SuperGlue MNN C++部署](https://github.com/Hanson0910/MNNSuperGlue)

