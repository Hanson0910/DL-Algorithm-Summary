# 交叉熵损失函数
#### 概念
- **信息量**

    信息奠基人香农（Shannon）认为“信息是用来消除随机不确定性的东西”，也就是说衡量信息量的大小就是看这个信息消除不确定性的程度。信息量的大小与信息发生的概率成反比。概率越大，信息量越小。概率越小，信息量越大。设某一事件发生的概率P(x)，其信息量表示为：

    <div align=center>
    <img src="https://note.youdao.com/yws/api/personal/file/WEB43865a50e667357d7e3c76c6e5ca2925?method=download&shareKey=cc7412b513b9082f1b5e0a914b839cf0"/>
    </div>

- **信息熵**

    信息熵也被称为熵，用来表示所有信息量的期望。

    <div align=center>
    <img src="https://note.youdao.com/yws/api/personal/file/WEB505be12d9b1a508cb0de63b417f5b0f4?method=download&shareKey=6b75738730cfb3e3d9c4c371659593a9"/>
    </div>

- **KL散度**

    KL散度用来衡量两个概率分布之间的相似度。
    
    <div align=center>
    <img src="https://note.youdao.com/yws/api/personal/file/WEB379be4d981ba3fcce75c63b7af0b1d07?method=download&shareKey=e018141b3d97eab0a27ea090138f220c"/>
    </div>

- **交叉熵**

    KL散度拆开形似为:
    
    <div align=center>
    <img src="https://note.youdao.com/yws/api/personal/file/WEB120a9162659849b4353c86730d6766d8?method=download&shareKey=d2f05aefbf8ee31660ec9087811dd006"/>
    </div>
    
    由于信息熵不变，所以交叉熵越小，预测概率的分布和原始分布越接近，
    所以优化的时候按交叉熵来优化即可。

- **求导**
  
    <div align=center>
    <img src="https://note.youdao.com/yws/api/personal/file/WEB58f679cdda0c6d980b6061aa2d3605c5?method=download&shareKey=6def5966c6ae6ac11c80a0aa0df0c502"/>
    </div>
    
    <div align=center>
    <img src="https://note.youdao.com/yws/api/personal/file/WEBecf8c8b46adf066b1ad6d7af3551eda8?method=download&shareKey=c4e86c520003b76da19706dbe2a2c753"/>
    </div>
    
- **注意点**

    <div align=center>
    <img src="https://note.youdao.com/yws/api/personal/file/WEBd1606f00dd274a37853c99cee3c42612?method=download&shareKey=02eee18c45f3ef900a3cec81a2060bfb"/>
    </div>

# Focal-loss

## 概述

**Focal-Loss可以在一定程度上降低样本类别不均衡带来的精度损失，通过简单easy-sample的loss值，提升hard-sample的loss值(相对提升)来促使网络更加关注难例。**

<div align=center>
<img src="https://images2018.cnblogs.com/blog/1055519/201808/1055519-20180818170840882-453549240.png"/>
</div>

**公式如下式所示：**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB1eabd88577dfced7b75af4772b96bfbf?method=download&shareKey=f40a750e9137a938f7994d307425c10e"/>
</div>

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

  