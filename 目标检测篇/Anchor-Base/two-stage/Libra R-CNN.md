# **Libra R-CNN: Towards Balanced Learning for Object Detection**

## 概述：

**paper link：https://arxiv.org/pdf/1904.02701.pdf**

**code link:	https://github.com/OceanPang/Libra_R-CNN**

**Libra R-CNN现有two-satge检测框架得问题分别从样本层面、特征层面和目标层面出发，提出IOU-balanced sampling，balanced feature pytamid，balanced L1 loss。主要解决选取高质量样本、更加有效融合mult-stage特征和更加关注难例的学习。**


​                                                                                                                           

## 具体实现

### 1. Sample level imbalance:

**在网络训练过程中，难例是能够十分有效提升训练结果的，但是一般random sampling往往会取到更到的easy sample,为了解决这个问题，有两个常用的方案：**

- **OHEM,OHEM通过对loss进行排序，认为loss较大的是难例，按照一定比例选取top-k的难例就可以了。但是OHEM存在一个问题就是noise label非常敏感，而且有一个较大的内存和计算成本。**
- **Focal Loss，Focal loss在two-stage阶段框架中表现效果并不是很突出，由于在RPN阶段大量简单样本在这一阶段被滤掉了。**



**争对上面两种情况，作者提出了 IoU-balanced Sampling**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB156b22b7cb6f0dbe0da21893ba5e70a0?method=download&shareKey=c83a19a2cf8e29dadf6e2d0e6873c6ad"/>
</div>


**如上图所示，作者发现，60%的难例出现在iou>0.05的情况下，而random sampling则仅提供30%的难例子。假设一共要选N个负例，其中总共有M个负例，那么在random sampling情况下每个样本选择的概率为P=N/M。为了提升难例的概率，按照iou均分成K个bin，要选的N个负例在K个bin里面是等概率的，如下式所示(MK表示在这个IOU区间内候选框的个数)：**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB790070c45c9f09383e71eb255b14a582?method=download&shareKey=f5b1f6b0f8ef458301d7165d3f0fb332"/>
</div>



<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB064552f43904d2088a110cf56ba2cad3?method=download&shareKey=6b7ffa744532981e0e446de136f5f6af"/>
</div>

**从上图可以看出，通过iou balance选出的负例跟靠近目标一些，大都集中在目标周围。需要注意的是iou balance选取的是负例，正例还是按正常去选取。**



### 2.**Feature level imbalance**:

**常见的特征融合方案一般为FPN top-down的方式融合，PANet在词基础上添加了botton-up的融合路径，但是这两样融合方式都存在一个问题，那就是融合特征跟关注相邻的特征层，不相邻的特征层会随着层数的增加不断稀释。为此提出了Feature level imbalance，包含如下四步：**

- **rescaling，把不同层的特征resale到同以分辨率（一般resale到8倍下采样分辨率），最后进行相加，此时不同层的贡献是一样的**
- **integrating，resale的特征在逆向回原来的分辨率再与原特征进行相加**
- **refining，融合的特征需要进行一些卷积消化一下，作者对比了卷积和 non-local module，最终选择 non-local module，采用Gaussian non-local attention**
- **strengthening，这个不知道，论文好像没有具体提出。**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBd5b5a8204b356916e4b6237d113a2490?method=download&shareKey=6085013b3356ff255e2c08ba32c80d3d"/>
</div>

### 3.Balanced L1 Loss

**Balanced L1 Loss主要是解决多任务学习过程中loss权重分配的一个问题，检测任务一般分为分类跟回归两个任务，loss = loss-cls + a * loss-reg，这个a就是一个平衡系数，我们可以通过改变a的值平衡分类跟回归loss。但是随着a的增加，作者发现模型对outliers更加敏感，outliers可以认为是难例，为了分类问题的准确性如果把过多注意力放在难例上（负例），难免导致分类结果不佳，那么如果去平衡outliers和inliers，作者因此提出了一个Balanced L1 Loss：**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBabb3511b71a58c8640ce862862d6fff4?method=download&shareKey=01981b358ef72703f3c9e54c37c5acb0"/>
</div>

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB590a401c89fa0b3946b0ae835b2ad776?method=download&shareKey=f3b135a66e05ef5cfe3660340d507ec0"/>
</div>

**如图所示，在regression误差较小的时候，梯度能够分配一个较大值。**

**Balanced L1 Loss给我的感觉就是，如果像普通的loss = loss-cls + a * loss-reg这种方式，通过改变a的值来均衡loss，这样会导致强迫网络去学习难的负例子（但是随着a的增加，作者发现模型对outliers更加敏感），但是这些难负例可能对分类没什么难度，分类跟希望学习到一些难正例，所以希望把注意力往inliers多放点，所以就有了这种改进，regression误差小的肯定就是在正例附近的，加大这一部分梯度使得网络多学习一点。这给人一中违背直觉的感觉，按理说我们需要把更多注意力刚在hard-sample上，这里却把更多注意力放在的easy-sample上，但是它的试验效果确实验证这这样做有用，而且能提升0.4个点。作者说的也不是很清楚，如果要强行解释的话可能跟第一步iou balance sample有关，第一不已经刷选出了hard sample，如果再往hard sample放更多的精力分类部分吃不消，分类更希望多一些正例（我瞎说的。。。。）**



## 总结：

**Libra R-CNN分别从样本、特征和目标三个层面上出发去分析，通过iou balance sample的策略选取更有价值的学习样本，同时通过更合理的特征融合方式去融合多级特征，最后设计一个Balanced L1 Loss来均衡loss。除了最后一个Balanced L1 Loss让我有点反直觉之外，其余两部分的工作还是很扎实的，分析了其余方式的利弊再提出自己的解决方案，确实能够提升不少网络性能。**
