# **CARAFE: Content-Aware ReAssembly of FEatures**

## 概述：

**CARAFE提出的目的是为了提高特征上采样的精度与速度，分析了传统插值和deconvolution的缺点，认为传统插值精度不好，deconvolution比较费时，为此提出了一种新的上采样策略。**

## 方案

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB5c08088468bde907da27aace1040ec64?method=download&shareKey=7dba8853af8fb9c1b10d04dd150d28d1"/>
</div>

**文章扯了一大堆有的没的，其实全文的思路在这个图中就可以看出来，整个过程分为两个部分：**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB43d06c2f52f0f93748b1b64c2317f541?method=download&shareKey=f202549911e4478493ac16b16c85b795"/>
</div>





# 总结

**CARAFE同篇文章说实话我是没多大兴趣的，感觉就是为了讲故事而写的一篇文文章，所以很多文章细节没有写出来，只是写了一个大概思想。文章虽然在baseline上有一定的提升，但是这个提升确定不是引入了更多参数？？？？文章说比deconv更加light，说实话我觉得并没有，一同操作下来复杂度还是挺高的。比deconv和最近邻提升0.5个点。**