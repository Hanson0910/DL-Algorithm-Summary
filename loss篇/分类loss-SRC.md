# 交叉熵损失函数
#### 概念
- **信息量**

    信息奠基人香农（Shannon）认为“信息是用来消除随机不确定性的东西”，也就是说衡量信息量的大小就是看这个信息消除不确定性的程度。信息量的大小与信息发生的概率成反比。概率越大，信息量越小。概率越小，信息量越大。设某一事件发生的概率P(x)，其信息量表示为：
    $$
    I(x) = -logp(x)
    $$
    
- **信息熵**

    信息熵也被称为熵，用来表示所有信息量的期望。
    $$
    H ( X ) = - \sum _ { i = 1 } ^ { n } P ( x _ { i } ) \log ( P ( x _ { i } ) ) ,X = ( x _ { 1 } , x _ { 2 } , \ldots , x _ { n } )
    $$

- **KL散度**

    KL散度用来衡量两个概率分布之间的相似度。
    $$
    D_{KL}(P||Q)=\sum_{i=1}^{n}P(x_{i})\frac{log(P(x_{i}))}{log(Q(x_{i})}\\
    P(x_{i})表示真实概率分布、Q(x_{i})表示预测概率分布，\\Q(x_{i})越接近P(x_{i}),KL散度越小。
    $$

- **交叉熵**

    KL散度拆开形似为:
    $$
    D_{KL}(P||Q)=\sum_{i=1}^{n}P(x_{i})\frac{log(P(x_{i}))}{log(Q(x_{i})}\\=\sum_{i=1}^{n}P(x_{i})\log(P(x_{i}))-\sum_{i=1}^{n}P(x_{i})\log(Q(x_{i}))\\
    \sum_{i=1}^{n}P(x_{i})\log(P(x_{i}))为信息熵；\\
    \sum_{i=1}^{n}P(x_{i})\log(Q(x_{i}))为交叉熵；\\
    $$
    由于信息熵不变，所以交叉熵越小，预测概率的分布和原始分布越接近，
    所以优化的时候按交叉熵来优化即可。

- **求导**
    $$
    分类问题中，P(x_{i})为目标one-hot形似的标签，这里用y_{i}表示，Q(x_{i})为\\softmax函数的输出用a_{i}表示，交叉熵形式为\\
    L=-\sum_{j}y_{i}lna_{j}\\
    a_{i} = \frac{e^{z^i}}{\sum_{k}e^{z^k}}
    $$
    
    $$
    \left\{\begin{array}{ll}
        \frac{dL}{dz^i}=\frac{d(-\sum_{j}y_{i}lna_{j})}{da_{j}}.\frac{da_{j}}{dz^i}=-\frac{dL}{da_{j}}.\frac{da_{j}}{dz^i}\\
        =-\sum_{j}\frac{y_{j}}{a_{j}}.\frac{da_{j}}{dz_{j}}\\
        =-(\sum_{i=j}\frac{y_{i}}{a_{i}}.\frac{da_{i}}{dz_{i}}+\sum_{i!=j}\frac{y_{j}}{a_{j}}.\frac{da_{j}}{dz_{i}})\\
        \frac{da_{i}}{dz_{i}}=a_{i}.(1-a_{i}),i==j\\
        \frac{da_{j}}{dz_{i}}=-a_{j}.a_{i},i!=j\\
        \frac{dL}{dz^i}=a_{i}.\sum_{j}y_{j}-y_{i}\\
        因为y_{i}是one-hot标签，所以\frac{dL}{dz^i}=a_{i}-y_{i}
        \end{array}\right.
    $$
$$
- **注意点** \\
a_{i} = \frac{e^{z^i}}{\sum_{k}e^{z^k}},当z^i过大时会造成内存上溢出，过小时会造成内存下溢，为了避免内存上溢。\\通常z^i会减去一个最大值\\
softmax(z^i-max)=\frac{e^{z^i-max}}{\sum_{k}e^{z^k-max}},\\
ln(a_{i}) = ({z^i-max})-ln(\sum_{k}e^{z^k-max})
$$




# Focal-loss

## 概述

**Focal-Loss可以在一定程度上降低样本类别不均衡带来的精度损失，通过简单easy-sample的loss值，提升hard-sample的loss值(相对提升)来促使网络更加关注难例。**

<div align=center>
<img src="https://images2018.cnblogs.com/blog/1055519/201808/1055519-20180818170840882-453549240.png"/>
</div>


**公式如下式所示：**
$$
\large \mathrm{L}_{f l}= \begin{cases}-\alpha\left(1-y^{\prime}\right)^{\gamma} \log y^{\prime} \quad, & y=1 \\ -(1-\alpha) y^{\prime \gamma} \log \left(1-y^{\prime}\right), & y=0\end{cases}\\
\large 从上公式可以看出，focal-loss是从BCEloss的基础上添加了两部分:\gamma和\alpha，以y=1为例，当y^{\prime}越大时认为该样本比较容易学习就降低\\
\large 该样本的loss值，当y^{\prime}越小时就认为该样本越难学习就相对增加该样的权重(就是减小的少了)。
$$




# Bi-Tempered Logistic Loss

## 概述

**Bi-Tempered Logistic Loss可以在一定程度上解决错误样本标记带来的问题，通过改进log函数和exp函数，添加两个tempered变量减轻错误样本的影响。**

**常见样本标注错误一般分两种情况：**

- **远离分类边界面的样本标注错误：**

  **这种情况《In *Proceedings of the 25th international conference on Machine learning*》证明了像如下图所示的convex losses，原理分界面的错误样本标注会极大增加loss值，由于softmax loss的无界性，会导致loss值会非常大。**

<div align=center>
<img src="https://img-blog.csdnimg.cn/20210503101558842.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxOTE3Njk3,size_16,color_FFFFFF,t_70#pic_center"/>
</div>



​		**解决这个问题的方法是使得softmax-loss有界，因此需要改进softmax-loss，改进形式如下：**

​		
$$
\large \log _{t}(x):=\frac{1}{1-t}\left(x^{1-t}-1\right)
$$
​		**当limit t 趋近1 的时候上式子就退化成了普通的log函数，函数图像如下所示：**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB2231d2a2fc8970cf757ef5ad2b6c3c7e?method=download&shareKey=400bd3de9b8769d0abfab3e8a7549045"/>
</div>

​	    **可以看出改进后的函数变得有界，能够缓解极端错误标签对网络的影响**

​		

- **靠近分类面的样本标注错误**

  **exp的短尾效应会迫使分类器扩大至噪声样本边界，重尾效应能够显著解决该问题，参考论文《Nan Ding and S. V. N. Vishwanathan. *t*-logistic regression. In *Proceedings of the 23th***

  ***International Conference on Neural Information Processing Systems*, NIPS’10, pages 514–522,**

  **Cambridge, MA, USA, 2010》**

  **通过改进exp函数可以减轻轻尾效应：**
  $$
  \large \exp _{t}(x):=[1+(1-t) x]_{+}^{1 /(1-t)}
  $$
  

  <div align=center>
  <img src=" https://note.youdao.com/yws/api/personal/file/WEB3d48d2f993c3ffdb56e9adeccb835315?method=download&shareKey=75ac28692a74df9d4d9ad2c3ca044f16"/>
  </div>

  