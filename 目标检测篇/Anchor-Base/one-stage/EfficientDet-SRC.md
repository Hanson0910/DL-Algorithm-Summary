

# EfficientDet

## 1.概述

**EfficientDet主要从FPN的结构出发，分析了不同种类的FPN网络结构，提出BIFPN的网络结构在提高检测精度的同时也能提升检测速度。作者观察到 PANet 的效果比 FPN ，NAS-FPN 要好，就是计算量更大；作者从 PANet 出发：**

- **移除掉了只有一个输入的节点。这样做是假设只有一个输入的节点相对不太重要。这样把 PANet 简化。**
- **在相同 level 的输入和输出节点之间连了一条边，假设是能融合更多特征，有点 skip-connection 的意味**。
- **PANet 只有从底向上连，自顶向下两条路径，作者认为这种连法可以作为一个基础层，重复多次。**

<div align=center>
<img src="https://pic4.zhimg.com/v2-225cc89e2308de82aa2267a9a944762f_r.jpg"/>.
</div>


**图1**

## 2.具体实现

### 2.1. Bckbone：

**EfficientDet的主干网络为EfficientNet，EfficientNet的网络性能如下图所示：**



<div align=center>
<img src="https://img-blog.csdnimg.cn/2020070319555365.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2JibGluZ2JibGluZw==,size_16,color_FFFFFF,t_70"/>.
</div>


**图2**

### 2.2. Neck：

**EfficientDet的Neck部分为BIFPN，BIFPN的机构如图1(d)所示，多层BIFPN的网路结构如下图所示：**

<div align=center>
<img src="https://pic1.zhimg.com/v2-680448fbf8cf5de53fa7b6a7b0a41f6c_r.jpg"/>.
</div>

图3
$$
\large BIFPN名称双向特征金字塔，主要思想是学习一个可学的标量 w_{i} 来加权每一个FPN层，作者认为不同输入对应的FPN层重要程度不一样。\\
\large 如果对每一个FPN分支仅学习一个权重，那么这个权重是不可控的(比如极大或则极小)，\\ 
\large 导致学习过程不可控。一个自然的想法是用softmax归一化，但是softmax函数很耗时，\\
\large 所以用了如下结构替换:\\
\large O=\sum_{i} \frac{w_{i}}{\epsilon+\sum_{j} w_{j}} \cdot I_{i}
\large I_{i}为特征图
$$
**以P6特征为例。BIFPN的表达式如下**：
$$
\begin{aligned}
\large P_{6}^{t d} &=\operatorname{Con} v\left(\frac{w_{1} \cdot P_{6}^{i n}+w_{2} \cdot \operatorname{Resize}\left(P_{7}^{i n}\right)}{w_{1}+w_{2}+\epsilon}\right) \\
\large P_{6}^{\text {out }} &=\operatorname{Conv}\left(\frac{w_{1}^{\prime} \cdot P_{6}^{i n}+w_{2}^{\prime} \cdot P_{6}^{t d}+w_{3}^{\prime} \cdot \text { Resize }\left(P_{5}^{\text {out }}\right)}{w_{1}^{\prime}+w_{2}^{\prime}+w_{3}^{\prime}+\epsilon}\right)
\end{aligned}\\
\large P_{6}^{t d}是P_{6}的中间特征。
$$

$$
\large BIFPN宽度: W_{bifpn}=64*(1.35^φ),φ表示Efficient的深度(D0,D1,D2...D7).\\
\large BIFPN深度: D_{bifpn}=3+φ \\
\large Box/class prediction network宽度：W_{pred} = W_{bifpn}.\\
\large Box/class prediction network深度：D_{\text {box }}=D_{\text {class }}=3+\lfloor\phi / 3\rfloor
$$

```
   # Weight
    self.p6_w1 = nn.Parameter(torch.ones(2, dtype=torch.float32), requires_grad=True)
    self.p6_w1_relu = nn.ReLU()
    self.p5_w1 = nn.Parameter(torch.ones(2, dtype=torch.float32), requires_grad=True)
    self.p5_w1_relu = nn.ReLU()
    self.p4_w1 = nn.Parameter(torch.ones(2, dtype=torch.float32), requires_grad=True)
    self.p4_w1_relu = nn.ReLU()
    self.p3_w1 = nn.Parameter(torch.ones(2, dtype=torch.float32), requires_grad=True)
    self.p3_w1_relu = nn.ReLU()

    self.p4_w2 = nn.Parameter(torch.ones(3, dtype=torch.float32), requires_grad=True)
    self.p4_w2_relu = nn.ReLU()
    self.p5_w2 = nn.Parameter(torch.ones(3, dtype=torch.float32), requires_grad=True)
    self.p5_w2_relu = nn.ReLU()
    self.p6_w2 = nn.Parameter(torch.ones(3, dtype=torch.float32), requires_grad=True)
    self.p6_w2_relu = nn.ReLU()
    self.p7_w2 = nn.Parameter(torch.ones(2, dtype=torch.float32), requires_grad=True)
    self.p7_w2_relu = nn.ReLU()

    self.attention = attention


    P7_0 -------------------------> P7_2 -------->
       |-------------|                ↑
                     ↓                |
    P6_0 ---------> P6_1 ---------> P6_2 -------->
       |-------------|--------------↑ ↑
                     ↓                |
    P5_0 ---------> P5_1 ---------> P5_2 -------->
       |-------------|--------------↑ ↑
                     ↓                |
    P4_0 ---------> P4_1 ---------> P4_2 -------->
       |-------------|--------------↑ ↑
                     |--------------↓ |
    P3_0 -------------------------> P3_2 -------->
        """
```



### 2.3. Head:

**EfficientDet的Head部分跟RetinaNet一样分为cls和reg两部分，如图3所示**



### 2.4. Sample Assignment:

**EfficientDet的Sample Assignment跟RetinaNet方式一样，t把iou < 0.4作为负样本， > 0.5作为正样本，其余为难例忽略。**



### 2.5. Anchor生成

**EfficientDet的anchor生成跟RetinaNet一样:**

- **ratios:[0.5,1,2]**
- **scales:[[2^0, 2^(1.0 / 3.0), 2^(2.0 / 3.0)]]**



### 2.6. Decode部分：

**EfficientDet decode部分跟[ssd decode部分一样](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Base/one-stage/SSD.md)**



### 2.7. LOSS部分：

**retinanet把iou < 0.4作为负样本， > 0.5作为正样本，其余为难例忽略，分类loss用focal loss，回归loss用smooth-L1 loss。分类loss计算代码。**

**focal loss部分会在分类loss部分单独总结。**





