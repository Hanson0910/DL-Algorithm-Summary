## CenterNet

## 1.概述：

**centernet预测中心点、宽高和偏置，无IOU、无nms，整个框架十分简介**

<div align=center>
<img src="https://pic4.zhimg.com/v2-8ad7954aef29ab84acab9cc9f0e5d7eb_r.jpg"/>
</div>
#### 

## 2. 具体实现：

### 2.1 Backbone：

**主干网络可以用Hourglass、DLA、ResNet等**



### 2.2 Head:

**输入经Backbone后输出3个分支:**

- **类别分支**
  $$
  \large \hat{Y}\in[0,1]^{\frac{W}{R} \times \frac{H}{R} \times C},R为下采样倍数，一般为4倍；W,H为原图大小，C为类别数(coco 80) \\
  $$
  

- **偏置分支**
  $$
  \large \hat{O} \in \mathcal{R}^{\frac{W}{R} \times \frac{H^{2}}{R} \times 2}
  $$

- **宽高分支**
  $$
  \large \hat{S} \in \mathcal{R}^{\frac{W}{R} \times \frac{H^{2}}{R} \times 2}
  $$



### 2.3 Sample Assignment:

**正负样本分配与[CornerNet](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Free/Key-Point/CornerNet.md)一样，确定一个高斯半径，以目标中心为高斯中心去初始化样本标签。**



### 2.4 Loss

- **分类Loss**

  **分类Loss采用focal loss**
  $$
  \large L_{k}=\frac{-1}{N} \sum_{x y c}\left\{\begin{array}{cl}
  \large \left(1-\hat{Y}_{x y c}\right)^{\alpha} \log \left(\hat{Y}_{x y c}\right) & \text { if } Y_{x y c}=1 \\
  \large \left(1-Y_{x y c}\right)^{\beta}\left(\hat{Y}_{x y c}\right)^{\alpha} & \\
  \large \log \left(1-\hat{Y}_{x y c}\right) & \text { otherwise }
  \large \end{array}\right.
  $$

- **offset Loss**

  **offset loss采用L1loss，猜测L2 loss在初始化初期训练不稳定**
  $$
  \large L_{o f f}=\frac{1}{N} \sum_{p}\left|\hat{O}_{\tilde{p}}-\left(\frac{p}{R}-\tilde{p}\right)\right|
  $$

- **宽高loss**

  **宽高loss任然采用L1 Loss**
  $$
  \large L_{\text {size }}=\frac{1}{N} \sum_{k=1}^{N}\left|\hat{S}_{p_{k}}-s_{k}\right|\\
  \large s_{k}=\left(x_{2}^{(k)}-x_{1}^{(k)}, y_{2}^{(k)}-y_{1}^{(k)}\right)\\
  \large \hat{S} \in \mathcal{R}^{\frac{W}{R} \times \frac{H}{R} \times 2}
  $$

- **mult-Loss**
  $$
  \large L_{det} = L_{k}+\lambda_{size}L_{size}+\lambda_{off}L_{off}
  $$

- 



### 2.5 inference

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB904e33fefa8e072ddafc60dc62519ccf?method=download&shareKey=25530e5478af20a0f04b819a40490b92"/>
</div>

## 3. 总结：

**CenterNet网络设计十分简单，较CornerNet实现起来容易很多，也可用于关键点检测等任务，无需nms后处理。缺点是如果相同种类的目标中心很近那么它只能回归一个目标。**