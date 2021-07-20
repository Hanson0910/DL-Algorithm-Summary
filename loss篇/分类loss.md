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
    
- **注意点**
    $$
    a_{i} = \frac{e^{z^i}}{\sum_{k}e^{z^k}},当z^i过大时会造成内存上溢出，过小时会造成内存下溢，为了避免内存上溢。\\通常z^i会减去一个最大值\\
    softmax(z^i-max)=\frac{e^{z^i-max}}{\sum_{k}e^{z^k-max}},\\
    ln(a_{i}) = ({z^i-max})-ln(\sum_{k}e^{z^k-max})
    $$
    

