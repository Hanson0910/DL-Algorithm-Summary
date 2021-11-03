# **Hybrid Task Cascade for Instance Segmentation**

## 1.概述

**Hybrid Task Cascade for Instance Segmentation在cascade mask rcnn基础上进行改进，充分利用detection与segmentation的联系，使得detection与segmentation由之前的并行学习改为串行学习，让segmentation能够学习到detection信息，同时也添加语义分割模块辅助训练进一步提升网路性能。**

**paper link:https://arxiv.org/pdf/1901.07518.pdf**

**code link: https://github.com/open-mmlab/mmdetection**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBf0fa55b4f6833706348d420507cc5a11?method=download&shareKey=731ef7854c9a09ac9b5ac02e2f5e402f"/>.
</div>

​                                                                                                                                   **图1**

## 2.具体实现

### 2.1 提出背景

**cascade rcnn证明通过级联的方式确实能够提升网络的性能，参看[cascade rcnn](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Base/two-stage/Cascade-RCNN.md)，所以提出了cascade mask rcnn，但是cascade mask-rcnn存在如下问题：**

- **图1(a)所示，cascade Mask R-CNN的每一级的mask分支与bbox分支是并行训练的，也就是说两个分支没有进行相互作用，mask分支没有从bbox学到任何信息。**
- **各级mask分支之间没有进行相互作用，互相独立，这不利于网络性能提升。**

**Cascade Mask R-CNN公式如下式所示：**
$$
\large \begin{aligned}
\mathbf{x}_{t}^{b o x} &=\mathcal{P}\left(\mathbf{x}, \mathbf{r}_{t-1}\right), & \mathbf{r}_{t}=B_{t}\left(\mathbf{x}_{t}^{b o x}\right) \\
\mathbf{x}_{t}^{\text {mask }} &=\mathcal{P}\left(\mathbf{x}, \mathbf{r}_{t-1}\right), & \mathbf{m}_{t}=M_{t}\left(\mathbf{x}_{t}^{\text {mask }}\right)
\end{aligned} \\
\large x为backbone输出的特征，x_t^{box},x_t^{mask}分别表示roi经过x后通过roi pooling得到的box和mask特征。\\
\large r_t和m_t分别代表预测的box和mask，B_t,M_t代表box和mask的head。
$$

### **2.2解决方案**

- **针对2.1的第一个问题，提出了图1(b)的结构，每一级的mask特征都由这一级的box提供，而不是上一级。**
  $$
  \large \begin{array}{ll}
  \mathbf{x}_{t}^{b o x}=\mathcal{P}\left(\mathbf{x}, \mathbf{r}_{t-1}\right), & \mathbf{r}_{t}=B_{t}\left(\mathbf{x}_{t}^{b o x}\right) \\
  \mathbf{x}_{t}^{\text {mask }}=\mathcal{P}\left(\mathbf{x}, \mathbf{r}_{t}\right), & \mathbf{m}_{t}=M_{t}\left(\mathbf{x}_{t}^{m a s k}\right)
  \end{array}
  $$

- **针对2.1中问题2，提出了图1(c)的结构，不同级的mask特征进行融合**
  $$
  \large \begin{array}{ll}
  \mathbf{x}_{t}^{b o x}=\mathcal{P}\left(\mathbf{x}, \mathbf{r}_{t-1}\right), & \mathbf{r}_{t}=B_{t}\left(\mathbf{x}_{t}^{b o x}\right) \\
  \mathbf{x}_{t}^{\text {mask }}=\mathcal{P}\left(\mathbf{x}, \mathbf{r}_{t}\right), & \mathbf{m}_{t}=M_{t}\left(\mathcal{F}\left(\mathbf{x}_{t}^{\text {mask }}, \mathbf{m}_{t-1}^{-}\right)\right)
  \end{array}\\
  \large \mathcal{F}\left(\mathbf{x}_{t}^{\text {mask }}, \mathbf{m}_{t-1}\right)=\mathbf{x}_{t}^{m a s k}+\mathcal{G}_{t}\left(\mathbf{m}_{t-1}^{-}\right)\\
  \large \mathcal{G}_{t}是1*1的卷积层
  $$
  

  **其中**：
  $$
  \large \begin{aligned}
  \mathbf{m}_{1}^{-} &=M_{1}^{-}\left(\mathbf{x}_{t}^{\text {mask }}\right) \\
  \mathbf{m}_{2}^{-} &=M_{2}^{-}\left(\mathcal{F}\left(\mathbf{x}_{t}^{\text {mask }}, \mathbf{m}_{1}^{-}\right)\right) \\
  & \vdots \\
  \mathbf{m}_{t-1}^{-} &=M_{t}^{-}\left(\mathcal{F}\left(\mathbf{x}_{t}^{\text {mask }}, \mathbf{m}_{t-2}^{-}\right)\right)
  \end{aligned}\\
  \large M_t^{-}代表mask-head,M_{t}的特征转换单元，由4个连续的3*3卷积组成，转换特征m_{t-1}^{-}\\
  \large 再接一个1*1的卷积为了维度对齐。
  $$

  <div align=center>
  <img src="https://note.youdao.com/yws/api/personal/file/WEB07737c7e3662e53c74a3a11aa5b94724?method=download&shareKey=edb298c8a44f951730c4b129c57b6388"/>.
  </div>

​                                                                                                                                            图2

- **在上面基础上添加一个语义分割部分辅助训练，语义分割部分能够提供一个全局的空间上下文信息，帮助区分正负样本，如图1(d)所示。**

$$
\large \begin{aligned}
\mathbf{x}_{t}^{b o x} &=\mathcal{P}\left(\mathbf{x}, \mathbf{r}_{t-1}\right)+\mathcal{P}\left(S(\mathbf{x}), \mathbf{r}_{t-1}\right) \\
\mathbf{r}_{t} &=B_{t}\left(\mathbf{x}_{t}^{b o x}\right) \\
\mathbf{x}_{t}^{\text {mask }} &=\mathcal{P}\left(\mathbf{x}, \mathbf{r}_{t}\right)+\mathcal{P}\left(S(\mathbf{x}), \mathbf{r}_{t}\right) \\
\mathbf{m}_{t} &=M_{t}\left(\mathcal{F}\left(\mathbf{x}_{t}^{\text {mask }}, \mathbf{m}_{t-1}^{-}\right)\right)
\end{aligned}
$$

**语义分割S由特征由FPN组成，如下图所示：**

<div align=center>
<img src="https://note.youdao.com/yws/api/personal/file/WEBb5f50692d2f6aef6c7e690ed35cc5a43?method=download&shareKey=0597d139dbc538e34939224f227cd55b"/>.
</div>

​                                                                                                                                         图3

## 3.总结

**Hybrid Task Cascade for Instance Segmentation最指的学习的地方应该是其对网路设计的思路，通过分析baseline(cascade mask-rcnn)再结合其它论文(cascade rcnn)的有点来改进网路，再在其基础上进行改进，这种网络设计的思路可以指导我们以后自己设计网络。Segmentation和detection本来就是一个相辅相成的东西，让它们相互作用是一个非常棒的idea，可以互相学习。**

