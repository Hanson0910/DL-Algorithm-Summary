

# **Gradient Harmonized Single-stage Detector**

## 1.概述

**Gradient Harmonized Single-stage Detector分析了一阶段目标检测器样本不均问题，认为大量简单的背景占领了整了训练过程，同时对比了OHEM和Focal Loss，认为OHEM直接放弃那些简单样本，这样效率是很低的；同时Focal Loss需要tune两个参数，并且Focal Loss是一个静态loss，不能适应数据的动态分布。作者提出l gradient harmonizing mechanism(GHM)--梯度协调机制，动态改变个样本梯度，降低简单样本和极难例的样本梯度，增加medium样本的梯度。**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB1b45e92c4508e01600aa7aa5ca082815?method=download&shareKey=3154e09077310f9c51e674ab6ac5b187"/>.
</div>


​																								**图1**

## 2.具体实现

### 2.1 问题描述

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBfb8171dfe956795715d8aff335c9555f?method=download&shareKey=5c75d56643029dd91ef8a313230ba2aa"/>.
</div>

***g* 为梯度的L1 范数，代表该样本该样本的贡献大小。**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB24e9da2e4fe54ea631404da8b601d61d?method=download&shareKey=e14ea15f0cd70c6445af90071ad00571"/>.
</div>

**如上图所示，为了更加清楚显示坐标轴经过log函数，可以看出大量easy-sample贡献了绝大部分梯度，还有一部分very-hard-sample贡献比较多的梯度，这些very-hard-sample可能是一些异常值，如果收敛模型被迫学习那些异常值可能对其它例子的分类不那么准确。为此提出了Gradient Density的概念**



### 2.2 Gradient Density

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBa319581f6fb723482f8d9bd31c8e13b6?method=download&shareKey=4eecef8896edb90ed585b4752e4a1dec"/>.
</div>



### 2.3 **GHM-C Loss**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBaa93d1cebfd1fcd1e8ce13ad7a3d71e8?method=download&shareKey=40aad34b34c3e2da8db6c4231315c1d9"/>.
</div>

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB44e8811bc1afb7922845c22d8fadf395?method=download&shareKey=3f6acd1ebf0beb99f83b5791013c5f45"/>.
</div>

**从上图可以看出GHM-C能够把easy-sample和very-hard-sample的梯度个将下来，其实focal-loss也可以，但是focal-loss对very-hard-sample好像没辙。**

### 2.4 Unit Region Approximation

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBb6251cb66ad68bf0249f176c5999b208?method=download&shareKey=31c45e34d7ba30767775535f41e8d24c"/>.
</div>

### 2.5 Unit Region Approximation GHM-C Loss

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB40ba8ba4dd52ac61d9fb0f59591cacd1?method=download&shareKey=753bf7f73e2213fa8e291735d20a567e"/>.
</div>

### 2.6 **GHM-R Loss**

**Faster-RCNN 回归Loss一般使用Smooth-L1 Loss，如下式子所示：**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB1ec7b0b98260975f49b80ddf865e9264?method=download&shareKey=191825d900a3fcf522c01b950cde7e62"/>.
</div>



- **改进Smooth-L1 Loss**
  
  <div align=center>
  <img src="https://note.youdao.com/yws/api/personal/file/WEB04f1e84dea98f15e7646fc1ae68ca7ef?method=download&shareKey=e7978f4b8e5401ad2e8be7d8d8509647"/>.
  </div>
  
  

如下图所示，这种回归的梯度范数与目标个数的关系，因为坐标回归只回归正例，所以分布图不想分类一样，打样样本集中在负例上。可以看出梯度占据比较大的是一些难例。所以使用GHM-R Loss优化梯度，如下式所示：

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB76b8c9e52584c89e34a9b283f34ea836?method=download&shareKey=1828b43d9f5d35a490e8dab9bf5b83cf"/>.
</div>

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB02d65c66fd75ecbf60092ab87e73726a?method=download&shareKey=1a5b3f1e682e599032d6da6191ae2b9e"/>.
</div>

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB29e0e96fb4481ef80858e571c6716804?method=download&shareKey=8373c7ceeafaedc199302852646341e0"/>.
</div>

## 3.总结

**GHM的工作十分扎实，认真分析了单阶段目标检测中的样本不均衡问题，通过梯度分布来动态为不同样本分配不同的梯度。这样原本分类中占据大量梯度的负例和easy-sample就会减少它们的梯度，占据少量的very-hard sample虽然样本比较少，但是却单个梯度很大，这样也会减小它们的梯度，medium-sample就会相对增加梯度。针对回归提出了改进版的smootl-l1 loss使得在d大于阈值的时候个样本梯度范数不再统一是1，能够反映不同样本的重要程度，同时针对性提出GHM-R Loss来较小very-hard sample的影响。**