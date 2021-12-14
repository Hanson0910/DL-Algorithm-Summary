

# Bilateral-Branch Network with Cumulative Learning-1

## 1.概述

**这篇文章针对分类问题中的长尾效应设计了一个双分支网络，两份分支共享权重，但是输入的数据的分布相反。**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB64c693b22b305bb286f270e1614f146a?method=download&shareKey=dadcb14ac28d74a35943b71b7abf7a19"/>.
</div>


## 2.具体实现

**如上图所示两个分支的结构是一样的权重共享，输入不一样，上面一个分支的输入是原始数据分布，下面一个分支的输入是原始分布的reverse版。其中通过一个变量a来控制两个分支的权重。**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBd06692bf7f9143dcc408e6db6d438494?method=download&shareKey=ae98fb22d0f13cff4cf63ac389873792"/>.
</div>

**a是根据训练阶段来动态改变的，具体公式如下：**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEB8b467c38afa0c4f2c6cda89aab1cc8fa?method=download&shareKey=169bcc0921c6f86c4e050fc312395a96"/>.
</div>

