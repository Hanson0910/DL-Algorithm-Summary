# SSD:Single Shot MultiBox Detector

## 概述：

[Paper-Link](https://arxiv.org/abs/1512.02325)

[Code-Link](https://github.com/balancap/SSD-Tensorflow)

**SSD从YOLO中继承了将detection转化为regression的思路，一次完成目标定位与分类；基于Faster RCNN中的Anchor，提出了相似的Prior box；加入基于特征金字塔（Pyramidal Feature Hierarchy）的检测方式，即在不同感受野的feature map上预测目标。**

<div align=center>
<img src="https://pic3.zhimg.com/v2-cad15a190c94ac354cfa3616ba305bce_r.jpg"/>
</div>

**图1**

## 具体实现

### 1. Backbone

**作责使用VGG16作为主干网络，在VGG16上做了一些修改将VGG16的两个全连接层转换成了普通的卷积层conv6和conv7,之后又接了多个卷积conv8_1，conv8_2，conv9_1，conv9_2，conv10_1，conv10_2, 最后用一个Global Average Pool来变成1x1的输出conv11_2,如下图所示：**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB743ac9ffe90838434b2fcea1fd12c97f?method=download&shareKey=e3c257eaace2180e7a775531bc0fb763"/>
</div>

**图2**



### 2.Neck

**如图2所示，SSD的neck部分为特征金字塔部分，抽取Conv4_3、Conv7、Conv8_2、Conv9_2、Conv10_2、Conv11_2层的feature map，然后分别在这些feature map层上面的每一个点构造不同尺度大小的prior box，然后分别进行检测和分类。个特征层的大小分别是输入的8、16、32、64、128、256倍下采样。**



### 3.Head

Head在Neck提取6层感受野不一样的特征基础上分别回归两个分支：分类和回归。如下所示:

```
SSDBoxHead(
  (predictor): SSDBoxPredictor(
    (cls_headers): ModuleList(
      (0): Conv2d(512, 84, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1))
      (1): Conv2d(1024, 126, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1))
      (2): Conv2d(512, 126, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1))
      (3): Conv2d(256, 126, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1))
      (4): Conv2d(256, 84, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1))
      (5): Conv2d(256, 84, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1))
    )
    (reg_headers): ModuleList(
      (0): Conv2d(512, 16, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1))
      (1): Conv2d(1024, 24, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1))
      (2): Conv2d(512, 24, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1))
      (3): Conv2d(256, 24, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1))
      (4): Conv2d(256, 16, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1))
      (5): Conv2d(256, 16, kernel_size=(3, 3), stride=(1, 1), padding=(1, 1))
    )
  )
```

**假设每层每个特征点分配N个prior bboxes（Conv4_3：4个；Conv7：6个；Conv8_2：6个；Conv9_2：6个；Conv10_2：4个；Conv11_2：4个），则cls分支的维度：N x NumCls；reg分支的维度为N x 4。**



### 4.Sample Assignment

**在输入为300x300大小的情况下，ssd一共会产生8732个priors：其中Conv4_3：38x38x4=5776；Conv7：19x19x6 = 2166；Conv8_2：10x10x6; Conv9_2: 5x5x6 = 150；Conv10_2：3x3x6 = 54；Conv11_2：1x1x4 = 4。**

1. **计算priors与所有gt的ious，并判断该prior与哪个gt的iou最大就把该prior分配该gt；**
2. **判断iou的值，如果小于0.5则认为该prior为负样本，否则为正样本；**

#### hard_negative_mining：

**前面为priors分配了正负标签，因为实际图片中gt数量较小，所以8732个priors中会产生严重的正负样本不均衡的情况，为此ssd提出在线难例挖掘的方式，通过对loss排序，前top k的loss的priors作为训练样本，认为loss大的样本为难训练的样本需要特殊关照，loss小的样本为简单样本，不用花太多精力去学习。具体过程如下：**

首先计算loss，交叉熵公式为：

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBc0a1529d0cf6bb539ee0c6b4d58e3f85?method=download&shareKey=d186b41a6724e7fe490e1faa77c1fa1c"/>
</div>

因为我们要计算每个prior的loss，为不是计算总loss，因为label是one-hot形似，实际上每部分的loss就是：



<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB4ea013ffcf01d158a1a5597ce0767364?method=download&shareKey=936eda76af797e873bdd5c294f5dd426"/>
</div>

可以用pytorch的函数实现:

​		

```
loss = -F.log_softmax(confidence, dim=2)[:, :, 0]
```

其中confidence是网络的类别输出，大小为[B,8732,num_class];算完loss只需取前N个loss最大部分即可:

```
def hard_negative_mining(loss, labels, neg_pos_ratio):
    pos_mask = labels > 0
    num_pos = pos_mask.long().sum(dim=1, keepdim=True)
    num_neg = num_pos * neg_pos_ratio

    loss[pos_mask] = -math.inf
    _, indexes = loss.sort(dim=1, descending=True)
    _, orders = indexes.sort(dim=1)
    neg_mask = orders < num_neg
    return pos_mask | neg_mask
```

neg_pos_ratio默认为3，即正负样本比例为1：3。首先获取num_neg的个数：num_neg = num_pos * neg_pos_ratio，然后对loss通过一个降序和升序可获得前

num_neg最大的loss的缩索引，如下例所示：

比如loss为[0.4,0.8.0.2,0.5,0.7,0.3]，降序排列的索引为[1,4,3,0,5,2]。再对[1,4,3,0,5,2]按升序排列，并获取对应索引：[3,0,5,2,1,4]。 假设我们选取前2个loss最大值，那么只需判断[3,0,5,2,1,4]是否小于2，这里mask为[false,true,false,false,true,false]。通过mask可以看出我们去了第0和4个loss，分别是0.8,0.7正好是前2个最大值。

**通过hard_negative_mining就可以选取任意比例的正负样本了，而且负样本还是特别需要关注的难例，这样就在一定程度上较小的正负样本不均衡的问题并且可以加速网络收敛速度。**



### 5.Decode

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB8b7f8bb1fd0e4a0bd8617fca92226724?method=download&shareKey=5776f827efc65f5d8f4668d13f9bd3cf"/>
</div>

**ssd decode方式跟faster-rcnn一样，只不过在实际代码中，center会除以center_variance（默认0.1）,w,h会除以size_variance（默认0.2），猜测这样做的目的是为了放大编码后的值，使得在计算loss的时候不至于太小，加速收敛。**

```
def convert_boxes_to_locations(center_form_boxes, center_form_priors, center_variance, size_variance):
    # priors can have one dimension less
    if center_form_priors.dim() + 1 == center_form_boxes.dim():
        center_form_priors = center_form_priors.unsqueeze(0)
    return torch.cat([
        (center_form_boxes[..., :2] - center_form_priors[..., :2]) / center_form_priors[..., 2:] / center_variance,
        torch.log(center_form_boxes[..., 2:] / center_form_priors[..., 2:]) / size_variance
    ], dim=center_form_boxes.dim() - 1)
```



### 6.Loss

**SSDloss的计算方式与faster-rcnn一样，把不同特征层收集到的priors concate一起，分类用 交叉熵loss，回归用smooth-l1 loss。**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBe9c1916831dd0be862a15bc11a8f8163?method=download&shareKey=9847c874f15b337017cc1c785f74e436"/>
</div>

N为匹配到priors的数量，a默认为1。



### 7.补充

1. anchor 生成：ssd 采用5种分辨率特征预测目标，分别是[8, 16, 32, 64, 100, 300]倍下采样，对应的feature map大小分别是[38, 19, 10, 5, 3, 1]。影响anchor生成的分别是：

   - min_sizes：[30, 60, 111, 162, 213, 264]
   - max_sizes：[60, 111, 162, 213, 264, 315]
   - aspect_ratios：[[2], [2, 3], [2, 3], [2, 3], [2], [2]]

   看代码最直观能感受到是如何生成anchor的：

   ```
          priors = []
           for k, f in enumerate(self.feature_maps):
               scale = self.image_size / self.strides[k]
               for i, j in product(range(f), repeat=2):
                   # unit center x,y
                   cx = (j + 0.5) / scale
                   cy = (i + 0.5) / scale
   
                   # small sized square box
                   size = self.min_sizes[k]
                   h = w = size / self.image_size
                   priors.append([cx, cy, w, h])
   
                   # big sized square box
                   size = sqrt(self.min_sizes[k] * self.max_sizes[k])
                   h = w = size / self.image_size
                   priors.append([cx, cy, w, h])
   
                   # change h/w ratio of the small sized box
                   size = self.min_sizes[k]
                   h = w = size / self.image_size
                   for ratio in self.aspect_ratios[k]:
                       ratio = sqrt(ratio)
                       priors.append([cx, cy, w * ratio, h / ratio])
                       priors.append([cx, cy, w / ratio, h * ratio])
   ```

   ### 8. 总结

   **SSD是非常经典的one-stage的目标检测方法，通过特征金字塔能够检测不同大小的目标，没有RPN步骤，采用全卷积的方式能够极大提升检测速度。使用hard_negative_mining可以有效解决正负样本不均很的问题，同时加速收敛速度。**