## Soft Anchor-Point Object Detection

## 1.概述：

**Soft Anchor-Point Object Detection主要是解决Anchor-free框架中的abchor-point检测精度不高的问题。**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBf0ab50d50d2cb2a5b40f99a6791520ad?method=download&shareKey=c713b28f5b4e6a4699a91cf0f35f3e37"/>
</div>



## 2. 具体实现：

### 2.1 Backbone：

**------**

### 2.2 Neck:

**FPN部分作为网络Neck部分**

### 2.3 Head:

**FCOS Head部分分为两个分支：类别分支、回归分支**

### 2.4 Encode-Decode：

**Encode-Decode部分可参考[FCOS](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Free/Anchor-Point/FCOS.md)**


### 2.4 Sample Assignment:

**Soft Anchor-Point Object Detection的核心在于样本分配上，认为anchor-point精度有待提高的原因：**

- **所有围绕每一个实例的正例都给一样权重去回归，这样会照成很多得分高但是位置不精确，位置精确但是得分低的情况**
- **没有充分利用FPN层，anchor-base中每一个实例按照其IOU可能会分配到不同的FPN层中，FCOS则仅分配到对应大小的特征层，FASF则按照loss大小分配到最合适的特征层，他们都只分配的一个层。**

**针对上述现象提出了如下解决方案：**

**如下图所示，作者认为实例被某一特征层激活，那么相邻的特征层改区域也被激活了，特征层之间离的越远相应激活的越小。所以同一实例可以被不同层级特征表示，但是赋予的权重不一样。因此设计了特征选择网络。**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB3f52eef9bbba92636c132fb59f911039?method=download&shareKey=5217d9166db2e7f07267d73c0cf7b5af"/>
</div>

- **对围绕每一个实例的正例，在计算loss的时候加一个权重系数，离实例中心越近的正例权重越大，越远的权重越小。**
  
  <div align=center>
  <img src="https://note.youdao.com/yws/api/personal/file/WEB9b26865a73dd752fdfc892f540b3246a?method=download&shareKey=079afde8a94ccd6f50ce2c6a59a5456d"/>
  </div>

### 2.5 Loss

- **分类Loss**

  **分类Loss采用Focal Loss**

- **回归Loss**

  **回归Loss采用IOU Loss**

- **实例权重Loss**

  **2.4节中提到的实例分配不同权重，在训练的时候需要为其分配标签，标签的分配形式如下：**
  
  **如下图所示，每一个实例通过roi-align在每个特征层之间提取特征，最后concate一起，通过soft-max函数输出看那一层的概率。这个监督信号参照FASF，看该实例在那一层loss最低则认为改实例属于那一层，就可以生成one-hot标签。例如一共3层特征，实例I在第1层特征loss最大，则one-hot标签为[0,1,0]。**
  
  <div align=center>
  <img src="https://note.youdao.com/yws/api/personal/file/WEB5c0d4644fbfa8d6135a38400817b77d1?method=download&shareKey=ce00f151d24e5a5562ab32731f4dd748"/>
  </div>
  **最终loss采用交叉熵loss,总的loss形式为：**
  
  <div align=center>
  <img src="https://note.youdao.com/yws/api/personal/file/WEB34b6d1ea05dd8ecc433a6bfa5053bb67?method=download&shareKey=e20c4519b224891b9311bab6fcfbdc94"/>
  </div>

### 2.7 inference

**inference的时候把centerness部分的网络输出值跟scores对应的值相乘，作为最终的scores。**

****


## 3. 总结：

**Soft Anchor-Point Object Detection与FASF不同点在于FASF利用loss来判断该实例最符合哪一层特征，就把该实例分配到对应特征层，其余特征层军不回归该实例，Soft Anchor-Point 则认为不同实例在不同特征层都会有相应只是响应的大小不一样，所以用了一个软标签为实例在不同特征层之间都有一定的梯度回传。我认为这样的方式很好，相当于一个soft-label，避免网络过度执迷于某层特征的回归起到一定程度的过拟合作用。**