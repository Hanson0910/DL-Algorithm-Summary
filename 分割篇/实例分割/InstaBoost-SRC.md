# ****InstaBoost****

## 1.概述

**InstaBoost是一种数据增广方式，用在示例分割和目标检测上。通过把实例crop出来再随机放置在图像背景区域来达到数据增广提升检测精度的目的。**

## 2.具体实现

### 2.1 随机InstaBoost

**随机InstaBoost主要是通过随机仿射变化把crop出的实例随机安放在目标周围，仿射变换的公式如下：**
$$
\large \mathbf{H}=\left[\begin{array}{ccc}
s \cos r & s \sin r & t_{x} \\
-s \sin r & s \cos r & t_{y} \\
0 & 0 & 1
\end{array}\right]\\
\large 其中t_{x},t_{y}表示平移量，s表示缩放量，r表示旋转量。通过\left\{t_{x},t_{y},s,r\right\}4维空间可以定义一个目标自然出现频率的模型\\
\large 定义一个概率密度函数f(.)测量把目标O放置在图像I中的合理性:
\large P(x,y,s,r|I,O) = f(t_{x},t_{y},s,r|I,O),x = x_{0} + t_{x},y = y_{0}+t_{y}\\
\large 在\left\{x_{0},y_{0},1,0\right\}的概率应该为最大值(这个比较好理解，目标不动当然概率密度函数在这个点的值最大),\\
\large argmaxP(x,y,s,r)=(x_{0},y_{0},1,0)\\
\large 按常理分析，在\left\{x_{0},y_{0},1,0\right\}处的一个小的区域，概率图P(x,y,s,r|I,O)应该有一个较大响应，\\
\large 那么InstaBoost则通过在目标\left\{x_{0},y_{0},1,0\right\}处的一个小的区域范围内随机采样，最后通过仿射矩阵\mathbf{H}把目标移动到相应区域。
$$
**随机InstaBoost这样做比较直观好理解，但是也有一个明显的短板就是实例只能在原周围区域进行变换。为了补充这个短板作者又提出了** **Appearance consistency heatmap guided InstaBoost来接触只能在目标周围区域变换的限制。**



### 2.2 Appearance consistency heatmap guided InstaBoost

 **appearance consistency heatmap是用来评估在任意位置(x0,y0)转换到(x,y)的RGB空间相似性。**
$$
\large 通过把上式的概率密度函数f(.)分成3个独立的条件概率分布f_{x,y}(,),f_{s}(.),f_{r}(.)，则\\
P(x,y,s,r)=f_{xy}(t_{x},t_{y}|s,r)f_{s}(s|r)f_{r}(r)\\
\quad \large =f_{xy}(t_{x},t_{y})f_{s}(s)f_{r}(r)\\
\large Appearance\quad consistency\quad heatmap \quad M(x, y)=E[P(x, y)] \propto f_{x y}\left(t_{x}, t_{y}\right)
$$


**上式定义了appearance consistency heatmap，接下来详细讲解一下appearance consistency heatmap，它主要由三部分组成：**

- **Appearance descriptor**

  **为了测量目标在原来位置与pasted位置的相似度，首先需要定义一个目标邻近区域背景编码后的描述子，它基于如下直觉：随着距离的增加，实例在appearance consistency中对周围环境的影响越小。定义一个appearance描述算子:**
  $$
  \large \mathcal D(c_{x},c_{y}) = \left\{(C_{i}(c_{x},c_{y}),w_{i})|i\in\left\{1,2,3\right\}\right\}\\
  \large C_{i}表示轮廓区域i带权重w_{i}，给定一个实例中心(c_{x},x_{y}),i=1表示最里面的轮廓区域\\
  \large w_{1}>w_{2}>w_{3}
  $$
  

- **Appearance distance**

  **Appearance distance是用来测量一对appearance描述算子之间的距离：**
  $$
  \large d\left(\mathcal{D}_{1}, \mathcal{D}_{2}\right)=\sum_{i=1}^{3} \sum_{\left(x_{1}, y_{1}\right) \in \mathcal{C}_{1 i} \atop\left(x_{2}, y_{2}\right) \in \mathcal{C}_{2 i}} w_{i} \Delta\left(I_{1}\left(x_{1}, y_{1}\right), I_{2}\left(x_{2}, y_{2}\right)\right)\\
  \large I_{k}(x,y)表示在图像k位置(x,y)处的RGB值，\Delta表示距离，一般用欧式距离。
  $$



- **Heatmap generation**
  $$
  \large 目标原位置的appearance描述子\mathcal D_{0},图像中所有其它位置的\mathcal {D},Appearance distances通过如下方式进行归一化：\\
  \large h(x) = -log(\frac{x-m}{M-m})\\
  \large M = max(d(\mathcal D,\mathcal D_{0})),m = min(d(\mathcal D,\mathcal D_{0}))\\
  \large HeatmapH通过在背景区域的所有位置像素和原实例点(x_{0},y_{0})处的归一化距离h(.)求得,x是(x_{0},y_{0})处RGB的值，\\它上面公式感觉少了一个\large RGB的求和项，如果按照公式来那么这个值肯定是单通道的。
  $$

## 补充

- 图像matting

  在将图片前景和背景按照标注进行分离的过程中，如果完全按照标注去切割前景，前景的边界处会呈不自然的多边形锯齿状，这与 COCO 数据标注方式有关。为了解决这一问题，研究者使用《A global sampling method for alpha matting》一文中提出的方法对前景轮廓做 matting 处理，以得到与物体轮廓契合的边界。

- 图像inpainting

  在前景背景分离后，背景上会存在若干个空白区域，这些区域可以使用 inpainting 算法进行填补。论文中使用了《Navier-stokes, fluid dynamics, and image and videoinpainting》文章中提出的 inpainting 算法。

  <div align=center>
  <img src="https://note.youdao.com/yws/api/personal/file/WEB6de7eda9f331cf23d13d18204d400ab4?method=download&shareKey=786e7e8844ea89ab888c0a055771db61"/>.
  </div>

## 感想

**InstaBoost这种增广思路在之前就有，目标检测用的比较多。作者通过建模找出合适的pasted位置，总体精度能提升2个点左右，还是蛮有效的。但是从它公式来看要遍历所以背景区域像素点，还有做log运算计算量可想而知，当然作者最后通过resize图像到一个小的分辨率做运算再resize回来，确实能减少不少时间，但是感觉计算量还是不老少。**