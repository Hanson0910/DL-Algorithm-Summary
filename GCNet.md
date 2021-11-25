## ****GCNet: Non-local Networks Meet Squeeze-Excitation Networks and Beyond****

## 1.概述：

**GCNet通过分析No-Local(Non-local Neural Networks)网络，实现发现No-Local网络用了大量的计算量来建立的特征长程关系，但是却发现通过No-local网络来建模的全局语义信息在不同的点上几乎是相同的。对于这一现象，作者进行了改进。**




## 2. 具体实现：

### 2.1 No-Local Net

**No-Local信息能够更好的理解图像，不管是RNN还是CNN，都通过重复堆叠的形式来获取No-Local信息，但是这会存在几个问题：**

- **为了获取No-Local信息，通常需要堆叠很深的深度**
- **大量的参数导致优化并不容易**
- **这种方式提取的特征也并不是那么No-Local，比如通过卷积，最终32倍下采样，那么特征图的右下角和左上角肯定还是没什么交集的，所以并不是那么No-Local**

**为此No-Local网络的提出使得网络能够以更浅的层实现完全No-Local的目的，主要分为3个步骤：**

1. **一个上下文建模单元通过聚集所有点的特征形成一个全局上下文特征**
2. **一个特征转换单元获取channel-wise的互相依赖关系**
3. **一个融合单元融合所有点全局上下文特征**

**只看这三个步骤可能有些疑惑，看公式和框架图会更加清楚一点：**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBd1be9ec2105c6a6017ad83182641b529?method=download&shareKey=993ef12b756ff8a42b715ae9607879ff"/>
</div>


<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB7f026814cf57f27dd61682a5efa09213?method=download&shareKey=6a5f72bb67d68d882e6f230eaeb6cc19"/>
</div>

​                                                                                                                                          图1

### 2.2 No-Local Net缺点

为了更加直观的了解No-Local block，作者可视化了不同位置的attention map，如下图所示：

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBa09430bbb44e001246ec8ef8ef5bedc3?method=download&shareKey=1aafbf23684a5b21166123ccfa24d3fe"/>
</div>

**图中红色的点是query position，可以看到不同的query position的attention map竟然几乎相同，如果说图像比较直观，那么数据就更加客观，作者分析特征之间的距离，主要包括：1. no-local block input之间的距离，2. no-local block output before fusio之间的距离，3. query position attention map之间的距离，结果如下图所示：**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB749a1a6ec3022c6857706c9d3cc6a582?method=download&shareKey=2b5952e3baafc9bc29049281cda00ea9"/>
</div>



**从上图中可以看出不同input的余弦距离还是蛮大的，但是output的距离却很小，attention map的距离也很小。说明为每一个query position通过一个No-Local block获取global context，global context随着训练的进行会变得相似。**

### 2.2 Simplify-No-Local Net

**有了以上发现，这座机智的设计了一个简化版本的 No-Local Net，既然global context与query position关系不大，那么就没必要为每一个query position获取其global context，这样太耗时。于是作者就为每一个query position共享一个global attention map，反正都是一样的，那么共享一个能够省掉很多计算量，岂不美哉。**
<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB3f39cd8b22868b9c36af941f6afb8373?method=download&shareKey=b0a882273fba776d555f3cff3aa6a829"/>
</div>

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBc91b46baab7e8dcecfc549765e32ae02?method=download&shareKey=d8c47d10680a0dc61f1c22977fe2ffab"/>
</div>

### 2.3 GC Net

**GCNet是作者分析SENet得出的，其实就是通过1*1的卷积达到降维的目的：**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBa69c4984e01cfdf9d7f6d301e08d35b1?method=download&shareKey=04371a298978302145c6691772a732fb"/>
</div>



## 3. 总结：

****

**GCNet我觉得十一篇非常棒的文章，我个人非常喜欢这种类型的文章，通过试验分析然后再解决问题，不像很多文章基本都是发现一个提升点就在那里狂讲故事。作者通过删掉每个query position的no-local bkock，建立一个共享global attention为每一个query position享用，这一步能省去很多计算量，最后再通过1*1卷积降维的作用进一步减小计算量，使之能够嵌到网络的每一个residual block中可以提升精度。no-local net的层数一般不会太多，计算负担不起，而嵌入GCNet的网络类似resnet，层数一般很深又加持了no-local特征精度当然蹭蹭涨。**