

# **Gradient Harmonized Single-stage Detector**

## 1.概述

**Gradient Harmonized Single-stage Detector分析了一阶段目标检测器样本不均问题，认为大量简单的背景占领了整了训练过程，同时对比了OHEM和Focal Loss，认为OHEM直接放弃那些简单样本，这样效率是很低的；同时Focal Loss需要tune两个参数，并且Focal Loss是一个静态loss，不能适应数据的动态分布。作者提出l gradient harmonizing mechanism(GHM)--梯度协调机制，动态改变个样本梯度，降低简单样本和极难例的样本梯度，增加medium样本的梯度。**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB1b45e92c4508e01600aa7aa5ca082815?method=download&shareKey=3154e09077310f9c51e674ab6ac5b187"/>.
</div>


​																								**图1**

## 2.具体实现

### 2.1 问题描述

**ce loss为：**
$$
\large L_{CE}(p,p^{*})= \begin{cases} - \log (p)ifp^{*}=1 \\\\ - \log (1-p)ifp^{*}=0 \end{cases}\\
 \large p \in \left[ 0,1 \right] 为预测概率,这里p = sigmoid(x),p ^ { * } \in \{ 0,1 \} 为ground-truth,
$$


**求导为：**
$$
\large L_{C E}\left(p, p^{*}\right)= \begin{cases}-\log (p) & \text { if } p^{*}=1 \\ -\log (1-p) & \text { if } p^{*}=0\end{cases}
$$


$$
\large g=\left|p-p^{*}\right|= \begin{cases}1-p & \text { if } p^{*}=1 \\ p & \text { if } p^{*}=0\end{cases}
$$
***g* 为梯度的L1 范数，代表该样本该样本的贡献大小。**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB24e9da2e4fe54ea631404da8b601d61d?method=download&shareKey=e14ea15f0cd70c6445af90071ad00571"/>.
</div>

**如上图所示，为了更加清楚显示坐标轴经过log函数，可以看出大量easy-sample贡献了绝大部分梯度，还有一部分very-hard-sample贡献比较多的梯度，这些very-hard-sample可能是一些异常值，如果收敛模型被迫学习那些异常值可能对其它例子的分类不那么准确。为此提出了Gradient Density的概念**



### 2.2 Gradient Density

$$
\large G D ( g ) = \frac { 1 } { l _ { \epsilon } ( g ) } \sum _ { k = 1 } ^ { N } \delta _ { \epsilon } ( g _ { k } , g )\\
\large g_k表示第k个样本的梯度，N表示样本的个数
$$

$$
\begin{aligned}
&\delta_{\epsilon}(x, y)= \begin{cases}1 & \text { if } y-\frac{\epsilon}{2}<=x<y+\frac{\epsilon}{2} \\
0 & \text { otherwise }\end{cases} \\
&l_{\epsilon}(g)=\min \left(g+\frac{\epsilon}{2}, 1\right)-\max \left(g-\frac{\epsilon}{2}, 0\right)
\end{aligned}\\
\large \epsilon表示长度，GD(g)即为：所有样本梯度落在以梯度g为中心，\\ 
\large \epsilon为长度的区间的样本个数，再除以样本区间的长度。表示为单位梯度区间内符合的样本个数。
$$

$$
\large \beta _{i}= \frac{N}{GD(g_{i})}\\
\large \beta_{i}为梯度密度协调参数。即梯度密度越大，\beta_{i}越小。
$$



### 2.3 **GHM-C Loss**

$$
\large L _ { G H M - C } = \frac { 1 } { N } \sum _ { i = 1 } ^ { N } \beta _ { i } L _ { C E } ( p _ { i }, p _ { i } ^ { * } ) = \sum _ { i = 1 } ^ { N } \frac { L _ { C E } ( p _ { i } ,p _ { i } ^ { * } ) } { G D ( g _ { i } ) }
$$

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB44e8811bc1afb7922845c22d8fadf395?method=download&shareKey=3f6acd1ebf0beb99f83b5791013c5f45"/>.
</div>

**从上图可以看出GHM-C能够把easy-sample和very-hard-sample的梯度个将下来，其实focal-loss也可以，但是focal-loss对very-hard-sample好像没辙。**



### 2.4 Unit Region Approximation

$$
\large 2.3中的计算量太大，为log(N^2),所以采用近似计算的方式。\\
\large 把区间\epsilon分成M份，每一份的长度为\epsilon，此时的\epsilon不是原先的\epsilon。
\large \\M=\frac {1}{\epsilon} \\
\large r_{j}是索引为j的uint \quad reign \quad r_{j}=[(j-1) \epsilon, j \epsilon)\\
\large R_{j}表示在r_{j}区间内的样本数，\operatorname{ind}(g)=t \text { s.t. }(t-1) \epsilon<=g<t \epsilon 表示梯度g的uint \quad reign \quad 索引\\
\large 近似的梯度密度函数为:
\large \hat{G D}(g)=\frac{R_{i n d(g)}}{\epsilon}=R_{i n d(g)} M\\
\large 从上式可以看出，在uint-region内的梯度密度，不同uint-region梯度密度不相等，\\
\large 相当于把原来梯度密度GD(g)分为了M份，每一份是一个bin，\\
\large 跟直方图的概念相似。这样计算量为log(MN)
$$



### 2.5 Unit Region Approximation GHM-C Loss

$$
\large \hat{\beta _{i}}= \frac{N}{\hat{GD(g_{i})}}\\
$$

$$
\begin{aligned}
\large \hat{L}_{G H M-C} &=\frac{1}{N} \sum_{i=1}^{N} \hat{\beta}_{i} L_{C E}\left(p_{i}, p_{i}^{*}\right) \\
\large &=\sum_{i=1}^{N} \frac{L_{C E}\left(p_{i}, p_{i}^{*}\right)}{\hat{G} D\left(g_{i}\right)}
\end{aligned}
$$

### 2.6 **GHM-R Loss**

**Faster-RCNN 回归Loss一般使用Smooth-L1 Loss，如下式子所示：**
$$
\large L _ { r e g } = \sum _ { i \in \{ x , y , w , h \} } S L _ { 1 } ( t _ { i } - t _ { i } ^ { * } )\\
\large t = (t_x,t_y,t_w,t_h),t^* = (t_x^*,t_y^*,t_w^*,t_h^*)分别是标签和预测值
$$

$$
\large S L_{1}(d)= \begin{cases}\frac{d^{2}}{2 \delta} & \text { if }|d|<=\delta \\ |d|-\frac{\delta}{2} & \text { otherwise }\end{cases}\\
\large d = t-t^*
$$

$$
\large \frac{\partial S L_{1}}{\partial t_{i}}=\frac{\partial S L_{1}}{\partial d}= \begin{cases}\frac{d}{\delta} & \text { if }|d|<=\delta \\ \operatorname{sgn}(d) & \text { otherwise }\end{cases}\\
\large \delta 为常系数，一般设置为1/9 \\
\large 从上式可以看出,当\delta<|d|是，梯度为 | \frac{ \partial SL_{1}}{ \partial t_{i}}|=1 ，\\
\large 从梯度范数可以看出，并不能依赖梯度范数来给不同样本的梯度做优化。\\
\large 一个可选项是直接用|d|来优化样本梯度，但是|d|的值可能很大导致梯度爆炸。
$$

- **改进Smooth-L1 Loss**
  $$
  \large ASL_{1}(d)= \sqrt{d^{2}+ \mu ^{2}}- \mu\\
  \large 当d很小时，ASL_{1}(d)近似与二次函数，当d很大时近似与一次函数。\\
  \large  \frac{ \partial ASL_{1}}{ \partial d}= \frac{d}{ \sqrt{d^{2}+ \mu ^{2}}} \\
  \large 梯度范数的范围为[0,1),\mu 为常数设置为2。\\
  \large  梯度范数 gr=| \frac{d}{ \sqrt{d^{2}+ \mu ^{2}}}| 
  $$
  

如下图所示，这种回归的梯度范数与目标个数的关系，因为坐标回归只回归正例，所以分布图不想分类一样，打样样本集中在负例上。可以看出梯度占据比较大的是一些难例。所以使用GHM-R Loss优化梯度，如下式所示：
$$
\begin{aligned}
\large L_{G H M-R} &=\frac{1}{N} \sum_{i=1}^{N} \beta_{i} A S L_{1}\left(d_{i}\right) \\
\large &=\sum_{i=1}^{N} \frac{A S L_{1}\left(d_{i}\right)}{G D\left(g r_{i}\right)}
\end{aligned}
$$


<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBe6b33c9b1e39a8aa7ba5d1052e547267?method=download&shareKey=997a325c128eb230e2f848fb443ac3c7"/>.
</div>

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB29e0e96fb4481ef80858e571c6716804?method=download&shareKey=8373c7ceeafaedc199302852646341e0"/>.
</div>

## 3.总结

**GHM的工作十分扎实，认真分析了单阶段目标检测中的样本不均衡问题，通过梯度分布来动态为不同样本分配不同的梯度。这样原本分类中占据大量梯度的负例和easy-sample就会减少它们的梯度，占据少量的very-hard sample虽然样本比较少，但是却单个梯度很大，这样也会减小它们的梯度，medium-sample就会相对增加梯度。针对回归提出了改进版的smootl-l1 loss使得在d大于阈值的时候个样本梯度范数不再统一是1，能够反映不同样本的重要程度，同时针对性提出GHM-R Loss来较小very-hard sample的影响。**