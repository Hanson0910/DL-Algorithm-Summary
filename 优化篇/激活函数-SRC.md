# 激活函数

## 概述

**激活函数对于人工神经网络模型去学习、理解非常复杂和非线性的函数来说具有十分重要的作用。它们将非线性特性引入到我们的网络中。如果不用激活函数，每一层输出都是上层输入的线性函数，无论神经网络有多少层，输出都是输入的线性组合，这种情况就是最原始的感知机（Perceptron）。没有激活函数的每层都相当于矩阵相乘。就算你叠加了若干层之后，无非还是个矩阵相乘罢了。如果使用的话，激活函数给神经元引入了非线性因素，使得神经网络可以任意逼近任何非线性函数，这样神经网络就可以应用到众多的非线性模型中。**



## 1.Sigmoid激活函数

$$
 \large \sigma (x)= \frac{1}{1+e^{-x}} \\
 \large \sigma^, (x) = \sigma (x)*(1 - \sigma (x))
$$

<div align=center>
<img src="https://pic2.zhimg.com/80/v2-595feb9c4660fdee432dcd30b8256735_720w.jpg"/>
</div>

#### 优点：

- **平滑，处处可导数**
- **求导容易**
- **能够把输入的连续实值变换为0和1之间的输出**

#### 缺点：

- **非0均值，会导致后一层的神经元将得到上一层输出的非0均值的信号作为输入, 改变了数据的分布，收敛速度变慢。**
- **容易梯度爆炸或消失**
- **存在梯度饱和区**

$$
   \large \left\{\begin{array}{l}
    \large y_{1} = w_{1} *  x_{1} \\
    \large z_{1} = \sigma(y_{1}) \\
    \large y_{2} = z_{1}*w_{2} \\
    \large z_{2} = \sigma(y_{2}) \\
    \large y_{out} = z_{2}*w_{3}
    \large \end{array}\right. \\
    \large 损失函数L_{loss}=||y_{out} - y_{truth}||^2 \\
    \large \frac{dL_{loss}}{dw_{1}}=\frac{dL_{loss}}{dy_{out}}*\frac{dy_{out}}{dz_{2}}
   \large  *\frac{dz_{2}}{dy_{2}}*\frac{dy_{2}}{dz_{1}}*\frac{dz_{1}}{dy_{1}}*\frac{dy_{1}}{dw_{1}}\\
   \large  \frac{dL_{loss}}{dw_{1}}=||y_{out} - y_{truth}||*w_{3}*\sigma^,(y_{2})*
   \large  w_{2}*\sigma^,(y_{1})*x_{1}\\
   \large  由于\sigma^,(x)在[0.0.25]之间，所以当初始化参数在[0,4]之间的时候，\\
   \large 会发生梯度消失，参数初始化在[4,~]的时候会发生梯度爆炸。
$$

- **指数操作比较耗时。**



## 2.tanh函数

**tanh为双曲正切函数。tanh和 sigmoid 相似，都属于饱和激活函数，区别在于输出值范围由 (0,1) 变为了 (-1,1)，可以把 tanh 函数看做是 sigmoid 向下平移和拉伸后的结果。**
$$
 \large \tanh(x)= \frac{e^{x}-e^{-x}}{e^{x}+e^{-x}} \\
 \large  \tanh(x)= \frac{2}{1+e^{-2x}}-1 
$$
**从公式中，可以更加清晰看出tanh与sigmoid函数的关系（平移+拉伸）。**

<div align=center>
<img src="https://pic1.zhimg.com/v2-ac5875cf045fbdf213b8b0ba67f10b30_r.jpg"/>
</div>

#### 优点：

- **平滑，处处可导数**
- **求导容易**
-  **能够把输入的连续实值变换为0和1之间的输出**
- **是0均值的，不会改变数据分布**

#### 缺点:

- **幂运算的问题仍然存在；**
- **tanh导数范围在(0, 1)之间，相比sigmoid的(0, 0.25)，梯度消失（gradient vanishing）问题会得到缓解，但仍然还会存在。**
- **同样存在梯度饱和区**



## 3.ReLU函数

**relu(Rectified Linear Unit)——修正线性单元函数：该函数形式比较简单**
$$
\large relu(x) = max(0,x)
$$

<div align=center>
<img src="https://pic2.zhimg.com/80/v2-da3babf705f525effbaab3bbbed7df51_720w.jpg"/>
</div>

#### 优点：

- **ReLU将x<0的输出置为0，就是一个去噪音，稀疏矩阵的过程。而且在训练过程中，这种稀疏性是动态调节的，网络会自动调整稀疏比例，保证矩阵有最优的有效特征**。
-  **解决了梯度消失问题，收敛速度快于Sigmoid和tanh函数**
- **计算速度快**
- **使网络变得更加稀疏**



#### 缺点:

-  **ReLU函数在0出不可导**

- **还是存在梯度爆炸的问题**

- **ReLU 强制将x<0部分的输出置为0（置为0就是屏蔽该特征），可能会导致模型无法学习到有效特征，所以如果学习率设置的太大，就可能会导致网络的大部分神经元处于‘dead’状态，所以使用ReLU的网络，学习率不能设置太大。**

- **非0均值**

  

## 4.Leaky ReLU, PReLU(Parametric Relu), RReLU(Random ReLU)函数

### 1.Leaky ReLU

$$
\large Leaky Relu(x) = \left\{\begin{array}{l} x,x>0  \\
\large                     \alpha * x, x < 0
\large                     \end{array}\right.
$$

<div align=center>
<img src="https://pic2.zhimg.com/80/v2-9b1fc63cf1058e5543285494fa26a4c1_720w.jpg"/>
</div>

#### 优点：

- **LeakyReLU的提出就是为了解决神经元”死亡“问题，LeakyReLU与ReLU很相似，仅在输入小于0的部分有差别，ReLU输入小于0的部分值都为0，而LeakyReLU输入小于0的部分，值为负，且有微小的梯度。**

#### 缺点：

- **少了Relu的稀疏性**

#### 2.ReLU

***PRelu*（参数化修正线性单元）中的作为一个可学习的参数，会在训练的过程中进行更新。**

#### 3.RReLU

***RReLU*，负值的斜率在训练中是随机的，在之后的测试中就变成了固定的了。**



## 5.ELU 函数

$$
\LARGE ELU(x) = \left\{\begin{array}{l} x,x>0  \\
                    \alpha * (e^x-1), x < 0
                    \end{array}\right.
$$

<div align=center>
<img src="https://pic2.zhimg.com/v2-fa5b4490dc4a7f698543f9d37e28b6b1_r.jpg"/>
</div>

#### 优点：

- **解决RELU非0均值，输出的分布是零均值的，可以加快训练速度。**

- **解决LeakyReLU单侧不饱和，ELU是单侧饱和的，可以更好的收敛。**

#### 缺点

- **计算复杂**
- **在0处不可导**