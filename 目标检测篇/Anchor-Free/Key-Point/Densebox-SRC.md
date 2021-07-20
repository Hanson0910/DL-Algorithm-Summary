# DenseBox

## 1.概述

**DenseBox是一个不需要产生proposal且可以进行端到端地训练的FCN_Based的物体检测器，且更加关注那么尺寸小和严重遮挡的目标。所以在人脸数据集MALF进行验证模型性能。**

1. **无Landmark分支的网路结构：**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB082a6e70fdd75a6b1269a238bf14812d?method=download&shareKey=69cb9b4833cc69544b62211201e01507"/>
</div>

2. **带Landmark分支的网络结构：**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB6d6d9fd833222102a13b74fcffd08202?method=download&shareKey=e649603d63d5f80fb3add3cb6bf9a0a3"/ width=720 height=480 >
</div>




**图1**

## 2.具体实现

### 2.1. Bckbone：

**基于VGG19进行的改进，整个网络包含16层卷积，前12层由VGG19初始化，输出conv4_4后接4个1x1的卷积，前两个卷积产生1-channel map用于类别分数，后两个产生4-channel map用于预测相对位置。最后一个1x1的卷积担当这全连接层的作用**


### 2.2. Neck：

**无Neck**



### 2.3. Head:

**DenseBox Head部分包含三部分，第一部分为：分类分支，第二部分为：回归分支，第三部分为：Landmark 分支。最终特征图的大小为(W/4,H/4), W,H为输入大小。**



### 2.4. Sample Assignment:

1. **以目标中心为中点，把目标的高resize大约50个像素点，可以得出resize的倍数；**
2. **按照该resize倍数resize整张图，以目标中心为中点crop 240 * 240大小的图片patch;**
3. **以目标在输出特征图中面积的0.3倍长度作半径，在该半径里面都为正样本，其余为负样本;**
4. **如果该patch里面包含多个目标，如果其它目标落在相对于patch中心目标(即以该目标中心进行crop patch的目标)的一定尺度范围内(0.8~1.25)就认为是正目标;**
5. **正样本附近2个像素（输出特征图空间的两个像素点）点的目标设置为忽略目标，不进行任何方向传播。**
6. **采用了在线难例挖掘来均衡正负样本，可以参考[SSD在线难例挖掘部分](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Base/one-stage/SSD.md)**




### 2.6. Decode部分：

**直接目标的中心点宽高下采样4倍即可。**



### 2.7. LOSS部分：

- **分类Loss:**
  $$
  \large \mathcal{L}_{c l s}\left(\hat{y}, y^{*}\right)=\left\|\hat{y}-y^{*}\right\|^{2}\\
  $$
  

- **回归Loss:**
  $$
  \large \mathcal{L}_{l o c}\left(\hat{d}, d^{*}\right)=\sum_{i \in\{t x, t y, b x, b y\}}\left\|\hat{d}_{i}-d_{i}^{*}\right\|^{2}
  $$
  

- **det Loss**
  $$
  \large M\left(\hat{t}_{i}\right)= \begin{cases}0 & f_{i g n}^{i}=1 \text { or } f_{\text {sel }}^{i}=0 \\ 1 & \text { otherwise }\end{cases}
  $$

  $$
  \large \mathcal{L}_{d e t}(\theta)=\sum_{i}\left(M\left(\hat{t}_{i}\right) \mathcal{L}_{c l s}\left(\hat{y}_{i}, y_{i}^{*}\right)+\lambda_{l o c}\left[y_{i}^{*}>0\right] M\left(\hat{t}_{i}\right) \mathcal{L}_{l o c}\left(\hat{d}_{i}, d_{i}^{*}\right)\right)
  $$

  $$
  \large \hat{y},y^*分别表示预测类被和实际类别；\hat{d},d*分别表示预测的bbox，和实际bbox，(x_{c},y_{c},w,h)形式，\lambda_{loc}默认为3\\
  $$

  

- **with Landmark Loss**
  $$
  \large \mathcal{L}_{\text {full }}(\theta)=\lambda_{\text {det }} \mathcal{L}_{\text {det }}(\theta)+\lambda_{\text {lm }} \mathcal{L}_{l m}(\theta)+\mathcal{L}_{r f}(\theta)\\
  \large \mathcal{L}_{\text {lm }}为landmark loss，默认16个landmark点，\mathcal{L}_{\text {rf}}是landmark的分类loss16个点代表16类，跟类别loss一样。
  $$
  

## 3.总结：

**DenseBox的思想非常超前，采用了多层特征融合，FPN的雏形，在线难例挖掘，直接回归中心点和宽高并设置正样本区域半径和忽略半径，这未后来的anchor-free目标检测打下基础，同时可以回归landmark在人脸检测领域有很强的应用性。缺点是只能检测一类目标，当然后期也可以设计成检测多类。**