## CenterNet

## 1.概述：

**centernet预测中心点、宽高和偏置，无IOU、无nms，整个框架十分简介**

<div align=center>
<img src="https://pic4.zhimg.com/v2-8ad7954aef29ab84acab9cc9f0e5d7eb_r.jpg"/>
</div>
#### 

## 2. 具体实现：

### 2.1 Backbone：

**主干网络可以用Hourglass、DLA、ResNet等**



### 2.2 Head:

**输入经Backbone后输出3个分支:**

- **类别分支**

  <div align=center>
  <img src="https://note.youdao.com/yws/api/personal/file/WEB73f83fa99070b7b37e273835e3f7863e?method=download&shareKey=ab0a6fb2b611cfd21d746273f0228861"/>
  </div>

  

- **偏置分支**

  <div align=center>
  <img src="https://note.youdao.com/yws/api/personal/file/WEB1e41292969d2e5b87845b31a141fbe8f?method=download&shareKey=22a2133b424e566c6babefc848a146b1"/>
  </div>

- **宽高分支**

  <div align=center>
  <img src="https://note.youdao.com/yws/api/personal/file/WEB44b6a4200bf25f0491db4fc0d0d1330f?method=download&shareKey=adce77644e97321b0b51d149b7c9ea09"/>
  </div>



### 2.3 Sample Assignment:

**正负样本分配与[CornerNet](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Free/Key-Point/CornerNet.md)一样，确定一个高斯半径，以目标中心为高斯中心去初始化样本标签。**



### 2.4 Loss

- **分类Loss**

  **分类Loss采用focal loss**

  <div align=center>
  <img src="https://note.youdao.com/yws/api/personal/file/WEBc16c4d83e0f74c92d7f0b1f76966258d?method=download&shareKey=6115440df66debbfa84c6cda67c946cb"/>
  </div>

- **offset Loss**

  **offset loss采用L1loss，猜测L2 loss在初始化初期训练不稳定**

  <div align=center>
  <img src="https://note.youdao.com/yws/api/personal/file/WEB5d13a0da61e636732e9ccc2e21de0e5f?method=download&shareKey=4ade98c95f5fa24646c246b14284609b"/>
  </div>

- **宽高loss**

  **宽高loss任然采用L1 Loss**

  <div align=center>
  <img src="https://note.youdao.com/yws/api/personal/file/WEB68e216c4e5ee552fca2f3d23dacd58f6?method=download&shareKey=4a3f93ed2ba313762e35a2412f13ecbb"/>
  </div>

  

- **mult-Loss**

  <div align=center>
  <img src="https://note.youdao.com/yws/api/personal/file/WEBddf91a7467762f61ae8d234b74090495?method=download&shareKey=ffebf2b6dc0af467911bc1c964ee0db4"/>
  </div>

- 



### 2.5 inference

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB904e33fefa8e072ddafc60dc62519ccf?method=download&shareKey=25530e5478af20a0f04b819a40490b92"/>
</div>

## 3. 总结：

**CenterNet网络设计十分简单，较CornerNet实现起来容易很多，也可用于关键点检测等任务，无需nms后处理。缺点是如果相同种类的目标中心很近那么它只能回归一个目标。**