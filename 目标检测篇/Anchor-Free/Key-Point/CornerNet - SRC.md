# CornerNet

## 1.概述

**将目标检测问题看作关键点回归问题，通过检测目标框的左上角和右下角两个关键点得到预测框。整个检测网络的训练是从头开始的，并不基于预训练的分类模型，这使得用户能够自由设计特征提取网络，不用受预训练模型的限制。**

<div align=center>
<img src="https://pic3.zhimg.com/v2-2353dc9fb1f6aea5be064ab9d63fd70e_r.jpg"/>
</div>

**图1**

## 2.具体实现

### 2.1主干网络：

**使用两个级联的Hourglass Network作为backbone network**

<div align=center>
<img src="https://pic2.zhimg.com/v2-98d5e9c905ac9a55c67b7550a1e6ab85_r.jpg"/>
</div>

**图2**

### 2.2Head：

**输入经过Backbone产生两张heatmaps，一张用于左上角点的预测，一张用于右下角点的预测，其中每张heatmaps分为3个分支：**

- **Heatmaps:[B,C,H/4,W/4];**

  **判断该点是哪个类别(可能有多个类别，所以解码的时候用sigmoid)；**

- **Embedding:[B,1,H/4,W/4];**

  **Embedding是看tl_corner跟br_corner的匹配情况，两个差值越小，说明这两个点越匹配；**

- **Offsets:[B,2,H/4,W/4]**

  **Offsets是预测偏置，弥补下采样误差。**

**如图2所示，在产生3个分支之前会经过一个CornerPooling的操作，如下图所示：**

<div align=center>
<img src="https://pic2.zhimg.com/v2-cc2c44d98f803f70a7dad2260f535949_r.jpg"/>
</div>

**图3**

### 2.3CornerPooling

<div align=center>
<img src="https://img-blog.csdn.net/2018101220185287?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQzODAxNjU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70"/>
</div>

**图4**

**如上图所示，因为CornerNet是预测左上角和右下角两个角点，但是这两个角点在不同目标上没有相同规律可循，如果采用普通池化操作，那么在训练预测角点支路时会比较困难。考虑到左上角角点的右边有目标顶端的特征信息（图4第一张图的头顶），左上角角点的下边有目标左侧的特征信息（图4第一张图的手），因此如果左上角角点经过池化操作后能有这两个信息，那么就有利于该点的预测，这就有了corner pooling。**

**该层有2个输入特征图，如图3所示。特征图的宽高分别用W和H表示，假设接下来要对图中红色点（坐标假设是(i,j)）做corner pooling，那么就计算(i,j)到(i,H)的最大值（对应Figure3上面第二个图），类似于找到Figure2中第一张图的左侧手信息；同时计算(i,j)到(W,j)的最大值（对应Figure3下面第二个图），类似于找到Figure2中第一张图的头顶信息，然后将这两个最大值相加得到(i,j)点的值（对应Figure3最后一个图的蓝色点）。右下角点的corner pooling操作类似，只不过计算最大值变成从(0,j)到(i,j)和从(i,0)到(i,j)。**

<div align=center>
<img src="https://img-blog.csdn.net/20181012201913296?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQzODAxNjU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70"/>
</div>

**图5**

### 2.4 Sample Assignment:

**如下图所示，如果仅把gt的角点当成正样本，那么正负样本的比例会极不均衡。cornernet选择把iou>0.7范围内的角点都作为正样本，但是不给一样的权重，按照二维高斯分布来给权重，离gt越近的角点权重越大，反之越小。**

<div align=center>
<img src="https://pic1.zhimg.com/v2-3c12cc5e21583f240d47b3d4c46533dc_r.jpg"/>
</div>

**那么如何确定高斯半径呢？如下所示：**

- **情况一，预测的框和GTbox两个角点以r为半径的圆外切**

<div align=center>
<img src="https://pic2.zhimg.com/80/v2-9947a25bd29646ee7fcf04abff11d011_720w.jpg" width=480 height=320 />
</div>

$$
\large \text { overlap }=\frac{h * w}{(h+2 r)(w+2 r)} \\
\large 4 * \text { overlap } * r^{2}+2 * \text { overlap } *(h+w) * r+(\text { overlap }-1) *(h * w)=0 \\
\large r=\frac{-b+\sqrt{b^{2}-4 a c}}{2 a}
$$

- **情况二，预测的框和GTbox两个角点以r为半径的圆内切**

  <div align=center>
  <img src="https://pic3.zhimg.com/80/v2-d83427bf1e16ef4fe29b8e283ca3545e_720w.jpg" width=480 height=320 />
  </div>

  $$
  \large \text { overlap }=\frac{(h-2 r) *(w-2 r)}{h * w} \\
  \large r=\frac{-b+\sqrt{b^{2}-4 a c}}{2 a}
  $$

- **预测的框和GTbox两个角点以r为半径的圆一个边内切，一个边外切。**

  <div align=center>
  <img src="https://pic1.zhimg.com/80/v2-abd0a5c02625618270651a43fd6276c4_720w.jpg" width=480 height=320 />
  </div>

  $$
  \large \text { overlap }=\frac{(h-r) *(w-r)}{2 * h * w-(h-r) *(w-r)} \\
  \large r=\frac{-b+\sqrt{b^{2}-4 a c}}{2 a}
  $$

  ```
  def gaussian_radius(det_size, min_overlap):
    height, width = det_size
  
    a1 = 1
    b1 = (height + width)
    c1 = width * height * (1 - min_overlap) / (1 + min_overlap)
    sq1 = np.sqrt(b1 ** 2 - 4 * a1 * c1)
    r1 = (b1 - sq1) / (2 * a1)
  
    a2 = 4
    b2 = 2 * (height + width)
    c2 = (1 - min_overlap) * width * height
    sq2 = np.sqrt(b2 ** 2 - 4 * a2 * c2)
    r2 = (b2 - sq2) / (2 * a2)
  
    a3 = 4 * min_overlap
    b3 = -2 * min_overlap * (height + width)
    c3 = (min_overlap - 1) * width * height
    # print(b3 ** 2 - 4 * a3 * c3)
    sq3 = np.sqrt(b3 ** 2 - 4 * a3 * c3)
    r3 = (b3 + sq3) / (2 * a3)
    return min(r1, r2, r3)
  ```

  

### 2.5 Sample matching:

1. **首先在类别分支中找出前topk的最大值，3*3的maxpooling操作相当于一个nms，然后再把top-left和botom-right的点索引topk个点组合成[topk,topk]的形式**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBcbc9805b7fe9455927b523440de30e67?method=download&shareKey=cd7dcdf298e529de31ef000d64375638"/>
</div>

2. **在offset预测图中根据tl_inds找出前topk个x,y的偏置**

   <div align=center>
   <img src="https://note.youdao.com/yws/api/personal/file/WEB674915ddd6527748384b55ee44121d6c?method=download&shareKey=6feb85612d277cef69fed5aacc26d83e"/>
   </div>

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB97d2b1df02397f152f89e8c2e650efff?method=download&shareKey=8a72f3614b9d53f3177463c5ede20074"/>
</div>

3. **在embdding预测图中根据tl_inds找出前topk个embdding vector**

   <div align=center>
   <img src="https://note.youdao.com/yws/api/personal/file/WEB3103b9f22d6e261c17a6ba5b732672d1?method=download&shareKey=448a4543681bbff04456e75418e2a5b3"/>
   </div>

4. **对socres和cls进行组合**

   <div align=center>
   <img src="https://note.youdao.com/yws/api/personal/file/WEB767cc6bd6dcce1b4959446375c9cbc99?method=download&shareKey=dc55273bd25bdadbb01af8b33f122df9"/>
   </div>

5. **根据tscores,clses和dists滤除不匹配的bboxes**

   <div align=center>
   <img src="https://note.youdao.com/yws/api/personal/file/WEBcd17f90fe64e3b3a37c9d5066e9cb9f4?method=download&shareKey=949b44b0828da898a52d8fc58e943aef"/>
   </div>

6. **对scores进行排序，选取前num_det(默认1000)的scores和其索引，按照其索引选择相应的bbox，最后再nms。**



### 2.6 Loss

1. **分类Loss：**

   **分类loss如下图所示，是一个改进版的focal loss**
   $$
   \large L_{cls}=\frac{-1}{N} \sum_{c=1}^{C} \sum_{i=1}^{H} \sum_{j=1}^{W}\left\{\begin{array}{cl}
   \large \left(1-p_{c i j}\right)^{\alpha} \log \left(p_{c i j}\right) & \text { if } y_{c i j}=1 \\
   \large \left(1-y_{c i j}\right)^{\beta}\left(p_{c i j}\right)^{\alpha} \log \left(1-p_{c i j}\right) & \text { otherwise }
   \large \end{array}\right.\\
   \large P_{cij}是类别C在位置(i,j)处的预测得分；y_{cij}是目标的真实标签（服从二维高斯分布）
   $$

2. **offset Loss**

   **offset loss采用smooth-l1 loss**
   $$
   \large L_{o f f}=\frac{1}{N} \sum_{k=1}^{N} \text { SmoothL1Loss }\left(\boldsymbol{o}_{k}, \hat{\boldsymbol{o}}_{k}\right)\\
   \large \boldsymbol{o}_{k}=\left(\frac{x_{k}}{n}-\left\lfloor\frac{x_{k}}{n}\right\rfloor, \frac{y_{k}}{n}-\left\lfloor\frac{y_{k}}{n}\right\rfloor\right)
   $$

3. **Embdding Loss**

   **embdding loss采用pull和push两部分，pull使得相同目标的top-left和bottom-right corner越接近，push使得不同目标的top-left和bottom-right corner越远离。**
   $$
   \large \begin{gathered}
   \large L_{p u l l}=\frac{1}{N} \sum_{k=1}^{N}\left[\left(e_{t_{k}}-e_{k}\right)^{2}+\left(e_{b_{k}}-e_{k}\right)^{2}\right], \\
   \large L_{p u s h}=\frac{1}{N(N-1)} \sum_{k=1}^{N} \sum_{j=1 \atop j \neq k}^{N} \max \left(0, \Delta-\left|e_{k}-\large \large e_{j}\right|\right)
   \large \end{gathered}\\
   \large e_{t_{k}},e_{b_{k}}分别对应top和bottom的corner feature-map值,e_{k}是e_{t_{k}},e_{b_{k}}的均值
   $$
   