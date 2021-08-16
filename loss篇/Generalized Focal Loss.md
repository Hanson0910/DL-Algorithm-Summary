## **Generalized Focal Loss**

## 1.概述：

**文章争对分类问题，提出Quality Focal Loss。核心目的是解决在训练阶段和测试阶段bbox置信度与类别置信度不统一问题，训练阶段bbox置信度与类别置信度单独训练，但是在测试阶段bbox置信度又内别置信度相乘。针对回归问题，提出Distribution Focal Loss，把bbox的回归当成一个任意随机分布问题来看待。**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB2fdf09ce551e5c9c8fe8d9071c19e370?method=download&shareKey=ffb07fae4f7c62acda9d37ae314ed10f"/>
</div>



## 2. 具体实现：

### 2.1 Quality Focal Loss：

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBca7df1f17c00ed550cf3e3e1e2c2e89a?method=download&shareKey=56d230278e9213908ed704045d4191b0"/>
</div>



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

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB49bf22843744333fbbd6d8513eb16902?method=download&shareKey=764f3e3ec90f64f8b36c5cc65b555d8d"/>
</div>

**如果按照上市的预测值去做回归，这满足预测条件的情况太多收敛速度会很慢如下图所示：**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBcfdaa334ca77e14380ee150b8cbc55e4?method=download&shareKey=635c3d10ad2d50aabfb7dd4147a7a4fa"/>
</div>
<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB5280dbb79b9dddaeabfd16f4daed4433?method=download&shareKey=2f29bea2a63192a9db87e45bfbc15ea5"/>
</div>

**很第三张更符合要求，于是为了限制这种情乱使网络更加关注标签周边的值，通过简单的扩大标签周边值得概率，DFL Loss表示如下：**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBcd9a16ca717b6f090f95045e20c4070f?method=download&shareKey=a015116b4e697eb8bb6593b3cd6f350e"/>
</div>


## 3. 总结：

****

**Quality Focal Loss比较好理解，通过软化标签用IOU值替换one-hot标签，再利用focal-loss的形式组成分类loss；Distribution Focal Loss还是没有理解透彻，大致是围绕坐标标签定义一个离散空间，然后把这个离散空间通过softmax函数表示每个离散空间可能是坐标的概率，同事为了加快收敛速度只计算坐标左右两点的loss。**