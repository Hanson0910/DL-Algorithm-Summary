# L1 Loss
#### 公式

$$
\large L_{L1}=\sum_{i=0}^{n}\left|y_{i}-y_{i}^{p}\right| / n
$$

#### 优点
不论对于什么样的输入值，都有着稳定的梯度，不会导致梯度爆炸问题，对离群点不敏感，具有较为稳健性的解。

#### 缺点
在中心点是折点，不能求导，不方便求解。其导数为常数，导致训练后期，预测值与真是差异很小时，损失函数的导数绝对值仍然为1，而如果learning rate不变，损失函数将在稳定值附近波动，难以继续收敛达到更高精度


# L2 Loss
#### 公式

$$
\large L_{L2}=\sum_{i=0}^{n}\left|y_{i}-y_{i}^{p}\right|^2 / n
$$

#### 优点
各点都连续光滑，方便求导，而且梯度值也是动态变化的，能够快速的收敛，得到较为稳定的解

#### 缺点
真实值和预测值的差值大于1时，会放大误差；而当差值小于1时，则会缩小误差，这是平方运算决定的。MSE对于较大的误差>1给予较大的惩罚，较小的误差<1给予较小的惩罚。也就是说，对离群点比较敏感，受其影响较大。所以训练初期，一旦出现预测值和真实值差异很大时，就很容易出现梯度爆炸。

# SmoothL1 Loss
#### 公式

$$
\large S m o o t h_{L 1}(x)=\left\{\begin{array}{ll}
0.5 x^{2} & \text { if }|x|<1 ; \\
\large |x|-0.5 & \text { else. }
\large \end{array} \quad x=y_{i}-y_{i}^{p}\right.
$$

#### 优点
综合了L1和L2的优点，相比于L2损失函数，对离群点、异常值使用L1，梯度变化相对更小，训练时不容易跑飞。当预测值与真是差异很小时，使用L2损失，快速得到较为稳定的解。

#### 缺点
L1,L2 loss也有相同缺点，就是他们都是分别回归宽高和中心点，没有把box当成一个整体看待。

# IOU Loss
#### 公式
<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBccd473fa86512c074201d65f5f34a729?method=download&shareKey=c81d6a2474bee24cdf6b55d928c24204"/>
</div>



#### 优点
如下图所示，L1 和 L2 的边界框 L1 和 L2 loss 是相同的，但是他们的 IOU 却相差很大：

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB3af3c8f2a84be45836272464af3e6fd7?method=download&shareKey=4277f9d266afe397ac058fde020e60ed"/>
</div>

如果宽高和中心点使用分别优化的话就会损失他们之间的相关性，实际上目标回归框之间存在相关性，并不是相互独立的，使用 IOU LOSS可以把回归框当成一个整体去优化，如上图最右边的回归情况最好，最左边的回归情况最差，但是它们的 LS Loss 确实相同的，所以不合理。

#### 缺点
- 预测值和 gt 不重叠，iou 始终为 0 且无法优化
- IOU 无法辨别不同的对其方式，如下图所示，下图三个的iou都是一样的

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBea7cefc86c1f0f45d71d6a853ee04586?method=download&shareKey=944a734313d039cf03db780a385a7c5a"/>
</div>

# GIOU LOSS

#### 公式
<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB5453d579a3efaf40e70d90137e951717?method=download&shareKey=a145d3b270d474e7a35e0101f2d121da"/>
</div>

$$
\large B^p = (x_{1}^p,y_{1}^p,x_{2}^p,y_{2}^p),B^g = (x_{1}^g,y_{1}^g,x_{2}^g,y_{2}^g)
$$

1. $$
   \large 计算B^g的面积A^g = (x_{2}^g - x_{1}^g) * (y_{2}^g - y_{1}^g)
   $$

   



2. $$
   \large 计算B^p的面积A^p = (x_{2}^p - x_{1}^p) * (y_{2}^p - y_{1}^p)
   $$

   3.

$$
\large 计算B^p,B^g重叠面积:

\large \begin{aligned}
\large x_{1}^{I}=\max \left(x_{1}^{p}, x_{1}^{g}\right), x_{2}^{I}=\min \left(x_{2}^{p}, x_{2}^{g}\right) \\
\large y_{1}^{I}=\max \left(y_{1}^{p}, y_{1}^{g}\right), x_{2}^{I}=\min \left(y_{2}^{p}, y_{2}^{g}\right) \\
\large I=\left\{\begin{array}{cc}
\large \left(x_{2}^{I}-x_{1}^{I}\right) *\left(y_{2}^{I}-y_{1}^{I}\right) & x_{2}^{I}>x_{1}^{I}, y_{2}^{I}>y_{1}^{I} \\
\large 0 & \text { otherwise }
\large \end{array}\right.
\large \end{aligned}
$$

4. 

$$
\large 找到课可以包含B^p,B^g最小的box，B^c:
\large \begin{aligned}
\large  x_{1}^{c} &=\min \left(x_{1}^{p}, x_{1}^{g}\right), x_{2}^{c}=\max \left(x_{2}^{p}, x_{2}^{g}\right) \\
\large y_{1}^{c} &=\min \left(y_{1}^{p}, y_{1}^{g}\right), x_{2}^{c}=\max \left(y_{2}^{p}, y_{2}^{g}\right)
\large \end{aligned}
$$

5. 

$$
\large 计算IOU,IOU = I / U = I / (A^P+A^g-I)
$$

6. 

$$
\large 计算GIOU, GIOU = IOU - (A^c - U) / A^c
$$

7. ​	
   $$
   \large 计算GIOU Loss，L_{giou} = 1-GIOU
   $$

#### 优点
- 预测值和 gt 不重叠时也可以优化
- 可以辨别不同的对其方式

#### 缺点
- 收敛速度比较慢

- Faster RCNN，MaskRCNN这种涨点少了些。猜测原因在于Faster RCNN，MaskRCNN本身的Anchor很多，出现完全无重合的情况比较少，这样GIOU和IOU Loss就无明显差别，所以提升不是太明显。

- 网络容易选择学习扩大面积来减小loss，而不是移动box位置去覆盖gt，如下图所示，当box被gt覆盖时，不管怎么移动box，iou和Giou loss的值都不变。

  <div align=center>
  <img src="https://www.pianshen.com/images/440/140a34dafe5aff0e861d4f236f7c00a0.png"/>
  </div>



# DIOU Loss
#### 公式

$$
\large D I o U=\operatorname{IoU}-\frac{\rho^{2}\left(b, b^{g t}\right)}{c^{2}}\\
\large b,b^{gt}分别代表预测宽和真实框的中心点，\rho表示欧式距离，c表示最小闭合曲线的欧式距离。\\
\large L_{DIou} = 1 - DIoU
$$

<div align=center>
<img src="https://pic3.zhimg.com/80/v2-5189de83711cd7bc66bcf4db685d03c6_720w.jpg"/>
</div>

#### 优点
1. 直接最小化预测框与目标框之间的归一化距离，以达到更快的收敛速度。
2. 使回归在与目标框有重叠甚至包含时更准确、更快。
3. 对于目标框包裹预测框的这种情况，DIoU Loss可以收敛的很快，而GIoU            Loss此时退化为IoU Loss收敛速度较慢

#### 缺点
没有考虑到形状差异


# CIOU Loss
#### 公式

$$
\large D I o U=\operatorname{IoU}-\frac{\rho^{2}\left(b, b^{g t}\right)}{c^{2}} + \alpha*v\\
\large \alpha=\frac{v}{(1-I o U)+v}\\
\large v=\frac{4}{\pi^{2}}\left(\arctan \frac{w^{g t}}{h^{g t}}-\arctan \frac{w}{h}\right)^{2}
\large \alpha是个平衡参数，不参与梯度计算。
$$

$$
\large 直接看括号里的arctan\frac{w}{h}根据反三角函数，就是矩形对角线的倾斜角度.,
\large 也就是反应预测框跟真实框的宽高比相似度。\\
\large \frac{4}{\pi^{2}}是一个归一化常数，arctan\frac{w}{h}的范围是\\
\large (0,\frac{\pi}{2}),平方后就是(0,\frac{\pi^{2}}{4})
$$



#### 优点
- 可以在DIOU Loss基础上回归宽高比差异