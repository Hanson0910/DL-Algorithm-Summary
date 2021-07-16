# 优化器

## 概述：

**优化器在DL中起着灯塔的作用，指引参数往正确的方向更新，一个好的优化器能够使模型收敛速度加快，跳出局部最优，训练过程平稳等**



## 定义：

$$
\Large 1.待优化参数:w\\
\Large 2.迭代次数: t\\
\Large 3.一阶动量:m _ { t } = \phi ( g _ { 1 } , g _ { 2 } , \cdots , g _ { t } )\\
\Large 4.二阶动量:V _ { t } = \psi ( g _ { 1 } , g _ { 2 } , \cdots , g _ { t } )\\
\Large 5.当前时刻下降梯度: \Delta w_{t}= -\alpha \cdot m_{t}/ \sqrt{V_{t}} \\
\Large 6.根据下降梯度参数进行更新: w_{t+1}=w_{t} + \Delta w_{t}
$$



## SGD

$$
\Large SGD算法没有动量的概率，一阶动量m_{t} = g_{t};二阶动量V_{t} = I^2;\\
\Large \Delta w_{t} = -\alpha * g_{t}\\
\Large w_{t+1} = w_{t} + \Delta w_{t}
$$



#### 优点：

- **SGD法由于每次仅仅采用mini-batch来迭代，计算速度很快。**

#### 缺点：

- **下降速度慢，训练容易震荡**
- **可能陷入局部最优点**



# SGDM：SGD with Nesterov Moment 

**为了抑制SGD的震荡，SGDM认为梯度下降过程可以加入惯性即历史时刻的梯度信息。**
$$
\Large m_{t} = \beta_{1} * m_{t-1} + (1-\beta_{1}) * g_{t}\\
\Large \Delta w_{t} = -\alpha*m_{t}\\
\Large w_{t+1} = w_{t} + \Delta w_{t}\\
\Large 例如：\\
\Large m_{1} = \beta_{0}*m_{0}+(1-\beta_{1})*g_{1},m_{0}=0\\
\Large m_{1} = (1-\beta_{1})*g_{1}\\
\Large m_2 = \beta_{1}*m_{1}+(1-\beta_{1})*g_{1}=\beta_{1}*((1-\beta_{1})*g_{1}) + (1-\beta_{1})*g_{2}\\
\Large m_3 = \beta_{1}*m_{2}+(1-\beta_{1})*g_{3} = \beta_{1}*(\beta_{1}*((1-\beta_{1})*g_{1}) + (1-\beta_{1})*g_{2})+(1-\beta_{1})*g_{3}\\
\Large 从上式可以看出离当前时刻越远的惯性，它对当前时刻的m_{t}影响呈指数衰减。只有越近时刻的惯性对当前时刻的m_{t}影响越大。
$$

#### 优点：

- **动量的存在使得 SGD 算法在训练的过程中更加的稳定。**

#### 缺点：

- **没有解决SGD容易陷入局部最优的问题**



# NAG：SGD with Nesterov Acceleration

**SGD 还有一个问题是困在局部最优的沟壑里面震荡。想象一下你走到一个盆地，四周都是略高的小山，你觉得没有下坡的方向，那就只能待在这里了。可是如果你爬上高地，就会发现外面的世界还很广阔。因此，我们不能停留在当前位置去观察未来的方向，而要向前一步、多看一步、看远一些。 NAG全称Nesterov Accelerated Gradient，是在SGD、SGD-M的基础上的进一步改进，改进点在于定义1。我们知道在时刻t的主要下降方向是由累积动量决定的，自己的梯度方向说了也不算，那与其看当前梯度方向，不如先看看如果跟着累积动量走了一步，那个时候再怎么走。因此，NAG在步骤1，不计算当前位置的梯度方向，而是计算如果按照累积动量走了一步，那个时候的下降方向：**
$$
\Large g_{t}= \nabla f( \omega _{t}- \alpha \cdot m_{t-1}) \\
 \Large m_{t} = \beta_{1} * m_{t-1} + (1-\beta_{1}) * g_{t}\\
\Large \Delta w_{t} = -\alpha*m_{t}\\
\Large w_{t+1} = w_{t} + \Delta w_{t}
$$

#### 优点：

- **收敛快**
- **在一定程度上解决了陷入局部最优的问题**

#### 缺点：

- **学习率未自适应**



# AdaGrad

**此前我们都没有用到二阶动量。二阶动量的出现，才意味着“自适应学习率”优化算法时代的到来。SGD及其变种以同样的学习率更新每个参数，但深度神经网络往往包含大量的参数，这些参数并不是总会用得到。对于经常更新的参数，我们已经积累了大量关于它的知识，不希望被单个样本影响太大，希望学习速率慢一些；对于偶尔更新的参数，我们了解的信息太少，希望能从每个偶然出现的样本身上多学一些，即学习速率大一些。**
$$
\Large 二阶动量就是迄今为止该维度上所有梯度的和 \\
\Large V_{t}= \sum _{ \tau =1}^{t}g_{ \tau }^{2} \\
 \Large m_{t} = g_{t} (这时还没用到一阶动量)\\
 \Large \Delta w_{t} = -\alpha*m_{t}  / \sqrt{V_{t}} \\
 \Large w_{t+1} = w_{t} + \Delta w_{t}\\
 \Large 从上式可以看出实际上学习率\alpha变成了\alpha / \sqrt{V_{t}},\\
 \Large 一般为了避免分母为0，会在分母上加一个小的平滑项。
 \Large 所以参数更新的越频繁\Large \sqrt{V_{t}}就越大，\Delta w_{t}就变化越小。
$$

#### 优点：

- **引入二阶动量可以对不同参数进行不同速度的改变，达到自适应学习率的目的**

#### 缺点：

- **由于二阶动量是单调递增的，所以会使得学习率单调递减至0，可能会使得训练过程提前结束，即便后续还有数据也无法学到必要的知识。**



# AdaDelta / RMSProp

**由于AdaGrad单调递减的学习率变化过于激进，我们考虑一个改变二阶动量计算方法的策略：不累积全部历史梯度，而只关注过去一段时间窗口的下降梯度。这也就是AdaDelta名称中Delta的来历。修改的思路很简单。前面我们讲到，指数移动平均值大约就是过去一段时间的平均值，因此我们用这一方法来计算二阶累积动量：**
$$
\Large V_{t}= \beta _{2}*V_{t-1}+(1- \beta _{2})g_{t}^{2} \\
\Large  m_{t} = g_{t} (这时还没用到一阶动量)\\
\Large \Delta w_{t} = -\alpha*m_{t}  / \sqrt{V_{t}} \\
\Large w_{t+1} = w_{t} + \Delta w_{t}\\
$$

#### 优点：

- **使用加权平均使得学习率不会单调递减至0，可以加快收敛速度。**



# Adam

**Adam和Nadam的出现就很自然而然了——它们是前述方法的集大成者。我们看到，SGD-M在SGD基础上增加了一阶动量，AdaGrad和AdaDelta在SGD基础上增加了二阶动量。把一阶动量和二阶动量都用起来，就是Adam了——Adaptive + Momentum。**
$$
\Large m_{t}= \beta _{1} \cdot m_{t-1}+(1- \beta _{1}) \cdot g_{t} \\
 
\Large V _ { t } = \beta _ { 2 } * V _ { t - 1 } + ( 1 - \beta _ { 2 } ) g _ { t } ^ { 2 } \\
 
\Large \Delta w_{t} = -\alpha*m_{t}  / \sqrt{V_{t}} \\
 
\Large w_{t+1} = w_{t} + \Delta w_{t}\\
 
\Large \beta_{1}一般取0.9，\beta_{2}一般取0.999初始化的时候m_{0} = 0,V_{0}=0，所以在训练初期m_{t},V_{t}都会接近0，因此我们常常\\
\Large 根据下式进行误差修正：\tilde { m } _ { t } = m _ { t } / ( 1 - \beta _ { 1 } ^ { t } )tilde{V}_{t}=V_{t}/(1- \beta _{2}^{t})
$$


# Nadam

**Adam是集大成者，但它居然遗漏了Nesterov, Nadam就是加上了Nesterov按照NAG的步骤1：**
$$
\Large g _ { t } = \nabla f ( w _ { t } - \alpha \cdot m _ { t - 1 } / \sqrt { V _ { t } } )\\
 \Large m_{t}= \beta _{1} \cdot m_{t-1}+(1- \beta _{1}) \cdot g_{t} \\
 
 \Large V _ { t } = \beta _ { 2 } * V _ { t - 1 } + ( 1 - \beta _ { 2 } ) g _ { t } ^ { 2 } \\
 
 \Large \Delta w_{t} = -\alpha*m_{t}  / \sqrt{V_{t}} \\
 
 \Large w_{t+1} = w_{t} + \Delta w_{t}\\
$$