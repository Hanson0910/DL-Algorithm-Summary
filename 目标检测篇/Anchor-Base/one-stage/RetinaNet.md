# RetinaNet

## 1.概述

**RetinaNet和大多数的one stage算法相同，直接对图片进行边框的回归，这导致了在一开始回归的时候，算法产生了大量的anticipate anchor（two stage 算法产生anchor的方式是通过region proposal的方式产生1k～2k的边框），这些anchor大部分都不包含object，即作者提到的easy negativate。 因此anchor导致了正负样本的不均衡。 正负样本不均衡主要有以下两个问题：**

- **在网络进行训练时，一些easy negativate 样本对loss不起作用，网络收敛速度很慢。**
- **由于存在大量的easy negativate 样本，因此在loss回归的过程，easy negativate样本将会覆盖掉真正有益的收敛方向，导致模型精度下降。**
**基于上面的分析，作者提出了一种对新型的loss，这种loss能够对不同的easy，hard样本进行权重的赋值。使得loss更加倾向于学习一些hard样本。**

<div align=center>
<img src="https://perper.site/images/article/retina-frame.png"/>
</div>

**图1**

## 2.具体实现

### 2.1. Bckbone：

**RetinaNet以ResNet为Backbone**

<div align=center>
<img src="https://pic1.zhimg.com/80/v2-4c40148cd684b26b7fcade1ea018af7c_720w.jpg"/>.
</div>

**图2**

### 2.2. Neck：

**RetinaNet在以ResNet为backbone的基础上最终输出了五层feature map，在这五层feature map进行anchor的选取。首先由Resnet 最后的三层C3，C4，C5产生P3，P4，P5，然后在C5的后面接着生成了P6，P7。如图2所示**



### 2.3. Head:

**RetinaNet的Head分为两部分一个分类分支和一个回归分支。如图1所示**



### 2.4. Sample Assignment:

**RetinaNet的Sample Assignment跟SSD方式，参考[SSD Sample Assignment](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Base/one-stage/SSD.md)**



### 2.5. Anchor生成

**RetinaNet的anchor生成比SSD稍微简单点:**

- **ratios:[0.5,1,2]**
- **scales:[[2^0, 2^(1.0 / 3.0), 2^(2.0 / 3.0)]]**

**每层featuremap的每个点产生num-ratios * scales个anchor。**

```
def generate_anchors(base_size=16, ratios=None, scales=None):
    """
    Generate anchor (reference) windows by enumerating aspect ratios X
    scales w.r.t. a reference window.
    """

    if ratios is None:
        ratios = np.array([0.5, 1, 2])

    if scales is None:
        scales = np.array([2 ** 0, 2 ** (1.0 / 3.0), 2 ** (2.0 / 3.0)])

    num_anchors = len(ratios) * len(scales)

    # initialize output anchors
    anchors = np.zeros((num_anchors, 4))

    # scale base_size
    anchors[:, 2:] = base_size * np.tile(scales, (2, len(ratios))).T

    # compute areas of anchors
    areas = anchors[:, 2] * anchors[:, 3]

    # correct for ratios
    anchors[:, 2] = np.sqrt(areas / np.repeat(ratios, len(scales)))
    anchors[:, 3] = anchors[:, 2] * np.repeat(ratios, len(scales))

    # transform from (x_ctr, y_ctr, w, h) -> (x1, y1, x2, y2)
    anchors[:, 0::2] -= np.tile(anchors[:, 2] * 0.5, (2, 1)).T
    anchors[:, 1::2] -= np.tile(anchors[:, 3] * 0.5, (2, 1)).T
    return anchors
```



### 2.6. Decode部分：

**RetinaNet decode部分跟[ssd decode部分一样](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Base/one-stage/SSD.md)**



### 2.7. LOSS部分：

**retinanet把iou < 0.4作为负样本， > 0.5作为正样本，其余为难例忽略，分类loss用focal loss，回归loss跟ssd一样。分类loss计算代码。**

**focal loss部分会在分类loss部分单独总结。**





