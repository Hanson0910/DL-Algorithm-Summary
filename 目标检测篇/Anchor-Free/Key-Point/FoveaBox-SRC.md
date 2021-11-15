# **FoveaBox: Beyound Anchor-Based Object Detection**

## 1.概述

**FoveaBox也是一个anchor-free的key-point检测框架，通过预测类别eatmap作为目标概率图，然后再回归中心点偏置。与FCOS等其它anchor-free检测框架的区别是它把目标邻接FPN level也作为正样本，同时它在定义positive area的时候并没有像其它框架一样定义ignore area。**



## 2.网络结构



<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBe94051ba1f167678b0b1e6482048f3b0?method=download&shareKey=dac407a041943a239d50a3fc17fd8ebf"/>
</div>


​                                                                                                                                         **图1**



**如上图所示，FoveaBox采用FPN结构，没级FPN分别预测分类跟回归两个分支。**

## 2.具体实现

### 2.1. ** *Object Occurrence Possibility***：

**文中提到的Object Occurrence Possibility其实就是正样本概率图，通过如下方式定义正样本：**
$$
\begin{aligned}
& \large x_{1}^{\prime}=\frac{x_{1}}{s_{l}}, \quad y_{1}^{\prime}=\frac{y_{1}}{s_{l}}, \quad x_{2}^{\prime}=\frac{x_{2}}{s_{l}}, \quad y_{2}^{\prime}=\frac{y_{2}}{s_{l}} \\
& \large c_{x}^{\prime}=0.5\left(x_{2}^{\prime}+x_{1}^{\prime}\right), \quad \large c_{y}^{\prime}=0.5\left(y_{2}^{\prime}+y_{1}^{\prime}\right) \\
&\large w^{\prime}=x_{2}^{\prime}-x_{1}^{\prime}, \quad h^{\prime}=y_{2}^{\prime}-y_{1}^{\prime}\\
&\large (x_{1},y_{1},x_{2},y_{2})为gt坐标，s_{l}为下采样倍数
\end{aligned}
$$

$$
\begin{aligned}
\large &x_{1}^{p o s}=c_{x}^{\prime}-0.5 \sigma w^{\prime}, & y_{1}^{p o s}=c_{y}^{\prime}-0.5 \sigma h^{\prime} \\
\large &x_{2}^{p o s}=c_{x}^{\prime}+0.5 \sigma w^{\prime}, & y_{2}^{p o s}=c_{y}^{\prime}+0.5 \sigma h^{\prime}
\large \\
\large &\sigma为常系数
\end{aligned}
$$

**由上式可以看出，正样本区域为目标中心点一定范围类的区域，如下图所示：**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB381b06c5bf9eca55f19f389d46f8c0d2?method=download&shareKey=bdb389fc41a67429e90c4fe1edc22345"/>
</div>

​                                                                                                                                        **图2**

### 2.2  Scale Assignment

**由于有多级FPN，文章在分配目标到对应特征层的时候按一定大小区间分配，只要在这个大小区间内的目标都为正样本，不在这个大小区间内的目标该区域为ignore area**
$$
\large \left[r_{l} / \eta, r_{l} \cdot \eta\right]\\
\large r_{l}为P_{3}到P_{7}的basic scale分别为32到512，\eta为常系数，文中设为2\\
$$
**通过上式的target分配到不同的FPN层上，有如下好处：**

- **邻接的pyramid feature往往有大致相同的语义特征**
- **这样分配会增加正样本的数量，使训练过程更加稳定。**



### 2.3 Box encode

$$
\begin{aligned}
&\large t_{x_{1}}=\log \frac{s_{l}(x+0.5)-x_{1}}{r_{l}}, \\
&\large t_{y_{1}}=\log \frac{s_{l}(y+0.5)-y_{1}}{r_{l}}, \\
&\large t_{x_{2}}=\log \frac{x_{2}-s_{l}(x+0.5)}{r_{l}}, \\
&\large t_{y_{2}}=\log \frac{y_{2}-s_{l}(y+0.5)}{r_{l}} .\\
& \large (x,y)是R^{pos}的点，(x_{1},y_{1},x_{2},y_{2})是gt的坐标,s_{l}是下采样倍数，r_{l}是basicscale
\end{aligned}
$$

**从上式可以看出box encode其实就是计算gt与pos点偏置(图2所示)再求log。**



## 3.Inference

**Inferense阶段分为如下步骤：**

- **以阈值为0.05滤除一部分低置信度的候选**
-  **在每一个prediction layer选取top-1000的候选**
- **NMS with threshold 0.5**


## 3.总结：

**FoveaBox的检测框架十分简洁，目前看下来就两个超参。与FCOS和CenterNet不同就是它是预测的4个点，然后利用了邻接FPN特征，同质化还是比较明显的。FPN这一块我觉得Soft Anchor-Point Object Detection的处理更好一些。**

