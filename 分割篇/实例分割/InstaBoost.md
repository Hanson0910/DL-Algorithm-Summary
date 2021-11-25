# ****InstaBoost****

## 1.概述

**InstaBoost是一种数据增广方式，用在示例分割和目标检测上。通过把实例crop出来再随机放置在图像背景区域来达到数据增广提升检测精度的目的。**

## 2.具体实现

### 2.1 随机InstaBoost

**随机InstaBoost主要是通过随机仿射变化把crop出的实例随机安放在目标周围，仿射变换的公式如下：**
<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB611a3e2dd26afadafc1c12fc8080b014?method=download&shareKey=c93a68864749c0cba9a03735336554c2"/>.
</div>

**随机InstaBoost这样做比较直观好理解，但是也有一个明显的短板就是实例只能在原周围区域进行变换。为了补充这个短板作者又提出了** **Appearance consistency heatmap guided InstaBoost来接触只能在目标周围区域变换的限制。**



### 2.2 Appearance consistency heatmap guided InstaBoost

 **appearance consistency heatmap是用来评估在任意位置(x0,y0)转换到(x,y)的RGB空间相似性。**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB27e6cb432261abd9ccdd3457f12d8f02?method=download&shareKey=eeeb6122388600c8bb3e99f7b4473537"/>.
</div>


**上式定义了appearance consistency heatmap，接下来详细讲解一下appearance consistency heatmap，它主要由三部分组成：**

- **Appearance descriptor**

  **为了测量目标在原来位置与pasted位置的相似度，首先需要定义一个目标邻近区域背景编码后的描述子，它基于如下直觉：随着距离的增加，实例在appearance consistency中对周围环境的影响越小。定义一个appearance描述算子:**
  
  <div align=center>
  <img src="https://note.youdao.com/yws/api/personal/file/WEBef70956df6df3898fc69a2fbf3b75109?method=download&shareKey=82e4cdd71fa9fc0f3892b2a251daf8c7"/>.
  </div>
  
  

- **Appearance distance**

  **Appearance distance是用来测量一对appearance描述算子之间的距离：**
  
  <div align=center>
  <img src="https://note.youdao.com/yws/api/personal/file/WEBd73e110f2ed1c6f7b11c14b890c12873?method=download&shareKey=563a150dc2b9bd374012e35802aa78c2"/>.
  </div>

- **Heatmap generation**
  
  <div align=center>
  <img src="https://note.youdao.com/yws/api/personal/file/WEBcc77844e29aa9c156c5132f08523d742?method=download&shareKey=0100d12295d954814fb4acbb300eb1c1"/>.
  </div>

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