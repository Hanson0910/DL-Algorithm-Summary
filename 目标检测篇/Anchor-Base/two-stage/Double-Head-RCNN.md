# **Double-Head RCNN：Rethinking Classifification and Localization for Object Detection**

## 概述：

**Double-Head RCNN 分析了分类head和回归head，认为主流的方式是都用全连接或则用卷积层最后接pooling再去做分类和回归，这样会降低检测精度。因此作者提出了Double-Head的思想，用全连接层去做分类且用卷积层做回归。作者认为fc层没有共享变换具有空间敏感性，而conv层则应为局部共享，在location敏感性更强。**

****

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBf9f017f50f567b442d4ba0fea5410d13?method=download&shareKey=8ab002c888bf86f31004518eb72013ad"/>
</div>



​                                                                                                                                      **图1**

## 具体实现

### 1. Backbone:

**作者论文实验中使用的ResNet-101和ResNet-50**



### 2. Neck：

**Neck部分这里即为RPN部分，可参考Faster-RCNN的RPN部分。**



### 3. Head：

- **CLS HEAD**

**cls Head如图1(c)所示，两个全连接层。**



- **Reg Head**

**Reg Head堆叠K个residual blocks， 第一个block作用是升维（256->1024)，如图2(a)所示，其余是bottleneck block,如图2(b)所示,本文在每一个bottleneck block前面添加一个non-local block，如图2(c)所示。最后 average pooling层把1024x7x7的特征pooling成1024维度再回归。**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBb98b58f80a6c9992579015d51947126e?method=download&shareKey=e3cb8efc1651fa40d3c3d318ab1ef561"/>
</div>

​																																		**图2**

- **混合任务**

**混合任务，如图1(d)所示，在cls head再接一个reg分支，在reg分支再接一个cls分支。作者认为回归头能够给分类任务提供额外的监督信号，reg-head和cls-head的bbox能够互补。**



### 5. Sample Assignment:

​	**可参考[Faster-RCNN的Sample Assignment部分](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Base/two-stage/Faster-RCNN.md)。**

### 6. Decode:

​	**可参考[Faster-RCNN的Decode部分](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Base/two-stage/Faster-RCNN.md)。**

### 7. Loss:

​	**可参考[Faster-RCNN Loss部分](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Base/two-stage/Faster-RCNN.md)。**

### 8. 总结

**fc层对全局空间信息掌握的更好，卷积能够对局部位置信息掌握更好，作为利用这个发现分别用fc和conv来进行分类和回归。然后提出混合任务，fc层再回归框，conv层再进行分类进行辅助互补。**

