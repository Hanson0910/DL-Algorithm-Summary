# Yolo

## 1.概述

**不论是two-stage的Faster-RCNN还是one-satge的SSD,RetinaNet等，都需要精心设置anchor，这样人为的anchor设置需要大量的先验知识，anchor设置不当会极大影响检测器的性能，Yolo1采用anchor-free的方式无需设置anchor，将detection视为回归问题，仅使用一个网络同时预测bounding box的位置和类别。**

<div align=center>
<img src="https://pic2.zhimg.com/80/v2-d9f14f0d6f0371912e1ce588f88e39cd_1440w.jpg"/>
</div>


**图1**

## 2.具体实现

### 2.1. Bckbone：

**借鉴了GooLeNet设计，共包含24个卷积层，2个全链接层（前20层中用1×1 reduction layers 紧跟 3×3 convolutional layers 取代GooLeNet的 inception modules），最后一个线性层使用 leaky relu**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB14015e04ea2db9510dd62a8d1150f1c5?method=download&shareKey=4458986982f69762137a5f4176d910aa"/>
</div>

### 2.2. Neck：

**无Neck**



### 2.3. Head:

**Yolo1的Head输出一个7x7x(NumClass+ 2 + 2*4)大小的特征图，如下图所示：**

<div align=center>
<img src="https://img-blog.csdn.net/20170420213936232?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvaHJzc3R1ZHk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast"/ >
</div>

**以PASCLA-VOC数据为例，一共20类(不算负样本)，首先需要20维来表示目标属于哪一个维度，yolo默认每个grid最多预测两个目标(两个目标得是同一个类别的)，紧接着需要2个维度去表示每个预测的置信度，同时每个预测需要回归4个坐标，所以Head一共有30维度。**



### 2.4. Sample Assignment:

**Yolo1没有Anchor那么怎么去回归目标呢？yolo1采用grid的方式，把输入分割成SxS大小的grid（默认为7x7), 判断目标中心落在哪个grid里面即该grid负责预测该目标（目标中心下取整判断跟哪个grid相等），如下图所示：**

<div align=center>
<img src="https://pic4.zhimg.com/80/v2-8e0328c4effdb02961b755a9652fb5a3_1440w.jpg"/ heigth = 480 width = 720>
</div>



### 2.6. Decode部分：

**直接回归原始坐标**



### 2.7. LOSS部分：

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBe9b03190128b319dc33716ba7f220b53?method=download&shareKey=2a755e5740ffce585a4f07a075afe30c"/ >
</div>



<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB2d70b9e9089ebb82c70309a27a24bdfb?method=download&shareKey=bdd5baeaeddd649bb2a44704917b840b"/ >
</div>

## 3.总结：

1. **YOLO检测物体非常快。因为没有复杂的检测流程，只需要将图像输入到神经网络就可以得到检测结果，YOLO可以非常快的完成物体检测任务。标准版本的YOLO在Titan X 的 GPU 上能达到45 FPS。更快的Fast YOLO检测速度可以达到155 FPS。而且，YOLO的mAP是之前其他实时物体检测系统的两倍以上。**
2. **YOLO可以很好的避免背景错误，产生false positives。不像其他物体检测系统使用了滑窗或region proposal，分类器只能得到图像的局部信息。YOLO在训练和测试时都能够看到一整张图像的信息，因此YOLO在检测物体时能很好的利用上下文信息，从而不容易在背景上预测出错误的物体信息。和Fast-R-CNN相比，YOLO的背景错误不到Fast-R-CNN的一半。**
3. **YOLO可以学到物体的泛化特征。当YOLO在自然图像上做训练，在艺术作品上做测试时，YOLO表现的性能比DPM、R-CNN等之前的物体检测系统要好很多。因为YOLO可以学习到高度泛化的特征，从而迁移到其他领域。**
4. **YOLO的物体检测精度低于其他state-of-the-art的物体检测系统。**
5. **YOLO容易产生物体的定位错误。**
6. **YOLO对小物体的检测效果不好（尤其是密集的小物体，因为一个栅格只能预测2个物体）。**