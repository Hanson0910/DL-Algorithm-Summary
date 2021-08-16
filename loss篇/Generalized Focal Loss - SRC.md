## **Generalized Focal Loss**

## 1.概述：

**文章争对分类问题，提出Quality Focal Loss。核心目的是解决在训练阶段和测试阶段bbox置信度与类别置信度不统一问题，训练阶段bbox置信度与类别置信度单独训练，但是在测试阶段bbox置信度又内别置信度相乘。针对回归问题，提出Distribution Focal Loss，把bbox的回归当成一个任意随机分布问题来看待。**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB2fdf09ce551e5c9c8fe8d9071c19e370?method=download&shareKey=ffb07fae4f7c62acda9d37ae314ed10f"/>
</div>



## 2. 具体实现：

### 2.1 Quality Focal Loss：

$$
\large Q F L ( \sigma ) = - | y - \sigma | ^ { \beta } ( ( 1 - y ) \log ( 1 - \sigma ) + y \log ( \sigma ) ) \\
\large \sigma为预测值,y为标签。值得注意的是这里的标签不再是one-hot标签，而是改预测框和gt的iou值。这个很好理解，相当于软化标签。
$$



****

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBcf814a0373f6543445381e39287939e0?method=download&shareKey=ecd98c71ac10502e933bf71fde6db247"/>
</div>

### 2.2 **Distribution Focal Loss**：

**Distribution Focal Loss提出的目的是为了在一定程度上反应目标框的不确定程度。如下图所示：**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB5f17c38d688adc91dd68e6e09be25ab5?method=download&shareKey=4bac53e07d155f886f8aa4d7676ae097"/>
</div>

**可以看出上下左右四条边，上下左三条边还是不容易混淆的，但是右边这条边就比较容易混淆了。普通的标签做法就是按照狄拉克分布来标记即认为在标注出置信度最高其余都为0，也有一些做法是按照高斯分布，对每一条边学习额外2个参数(高斯的均值和方差)，但是这里有一个先验知识即这些目标框是按照高斯分布。这两种做法都有一定的局限性，作为认为目标狂应该是General分布的。**
$$
\large \hat { y } = \int _ { - \infty } ^ { + \infty } P ( x ) x d x = \int _ { y _ { 0 } } ^ { y _ { n } } P ( x ) x d x\\
\large y_0<=y<=y;\hat {y}是预测值，且y_0<=\hat {y} <=y;\\
\large 为了离散化把y_0,y_n分成n等分\{ y _ { 0 } , y _ { 1 } , \ldots , y _ { i } , y _ { i + 1 } , \ldots , y _ { n - 1 } , y _ { n } \}\\
\large \sum _{i=0}^{n}P(y_{i})=1 \\
\large \hat { y } = \sum _ { i = 0 } ^ { n } P ( y _ { i } ) y _ { i }\\
\large 一般情况下P(y_i)用softmax函数得到。y_0=0,y_1=1,y_2=2,....,y_n=n
$$
**如果按照上市的预测值去做回归，这满足预测条件的情况太多收敛速度会很慢如下图所示：**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBcfdaa334ca77e14380ee150b8cbc55e4?method=download&shareKey=635c3d10ad2d50aabfb7dd4147a7a4fa"/>
</div>

$$
\large 上图三种形式都满足 \sum _{i=0}^{n}P(y_{i})=1 \\
$$

**很第三张更符合要求，于是为了限制这种情乱使网络更加关注标签周边的值，通过简单的扩大标签周边值得概率，DFL Loss表示如下：**
$$
\large DFL(S_{i},S_{i+1})=-((y_{i+1}-y) \log (S_{i})+(y-y_{i}) \log (S_{i+1})).\\
\large P(x)用S(.)表示。y _ { i } \leq y \leq y _ { i + 1 }表示标签得左右俩个点。
$$


## 3. 总结：

****

**Quality Focal Loss比较好理解，通过软化标签用IOU值替换one-hot标签，再利用focal-loss的形式组成分类loss；Distribution Focal Loss还是没有理解透彻，大致是围绕坐标标签定义一个离散空间，然后把这个离散空间通过softmax函数表示每个离散空间可能是坐标的概率，同事为了加快收敛速度只计算坐标左右两点的loss。**