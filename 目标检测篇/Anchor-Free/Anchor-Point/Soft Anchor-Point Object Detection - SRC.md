## Soft Anchor-Point Object Detection

## 1.概述：

**Soft Anchor-Point Object Detection主要是解决Anchor-free框架中的abchor-point检测精度不高的问题。**

<div align=center>
<img src="https://pic4.zhimg.com/v2-bbec7cf563fc6c0bb981bef30bb9ca17_r.jpg"/>
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
  $$
  \large w_{l i j}=\left\{\begin{array}{ll}f\left(p_{l i j}, B\right), & \exists B, p_{l i j} \in B_{v} \\
  \large 0, & \text { otherwise }
  \large \end{array}\right.\\
  \large w_{l i j}是attention权重，p_{l i j}是实例B在第l层特征的一个位于(i,j)的正例。f\left(p_{l i j}, B\right)是一个\\
  \large 反应p_{l i j}到B边界的一个距离函数，文中用:
  \large f\left(p_{l i j}, B\right)=\left[\frac{\min \left(d_{l i j}^{l}, d_{l i j}^{r}\right) \min \left(d_{l i j}^{t}, d_{l i \large j}^{b}\right)}{\max \left(d_{l i j}^{i}, d_{l i j}^{r}\right) \max \left(d_{l i j}^{t}, d_{l i j}^{b}\right)}\right]^{\eta}\\
  \large \eta设置为1。\left(d_{l i j}^{l}, d_{l i j}^{r},d_{l i j}^{t}, d_{l i j}^{b}\right)分别代表第l层特征距离目标中心点(i,j)的左、右、上、下的距离。\\
  \large 可知但在中心点是f(p_{lij},B)为越远离中心点越小。论文中otherwise为1我觉得错了。\\
  \large 假设实例B用(c,x,y,w,h)表示,B_{v}为有效bbox,B_{v}=(c, x, y, \epsilon w, \epsilon h)，\\
  \large \epsilon是一个缩放系数，即在B一定区域内的sample作为正样本，其余为负样本。
  $$

- 



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
  
  

### 2.7 inference

**inference的时候把centerness部分的网络输出值跟scores对应的值相乘，作为最终的scores。**

****


## 3. 总结：

**FCOS网络结构十分简单，同时利用了FPN多层特征能够有效检测多尺度目标，提出的中心度度量分支可以有效滤除低质量的输出，同时在相同目标拥挤场景下FCOS的性能并好，因为他输出的特征图只能在相同类别上回归一个目标，类似于wider-face这样的场景检测性能并不会理想。**

