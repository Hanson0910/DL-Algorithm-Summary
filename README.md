# DL-Algorithm-Summary
**该仓库记录本人平时阅读一些论文和方法的总结，也为了督促自己多学习多分享，有些内容也会同步分享到[知乎](https://www.zhihu.com/people/feng-yi-54-69/posts),分享不易记得点赞哟。具体内容可点击readme下的超链接查看，链接中有些图片外链可能需要较长加载时间。**

# 目标检测篇
## Anchor-Base

### two-stage

1. [Faster R-CNN: Towards Real-Time Object Detection with Region Proposal Networks，在Fast-RCNN基础上设计RPN层替换选来的selective search方式生成ROI，达到真正的end-to-end的训练方式](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Base/two-stage/Faster-RCNN.md)

  ### ont-stage
1. [SSD:Single Shot MultiBox Detector,，通过特征金字塔能够检测不同大小的目标，没有RPN步骤，采用全卷积的方式能够极大提升检测速度。使用hard_negative_mining可以有效解决正负样本不均很的问题，同时加速收敛速度](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/Anchor-Base/one-stage/SSD.md)


## Anchor-Free

### Key-Point

### Anchor-Point



## 数据集
1. [MS-COCO](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/%E6%95%B0%E6%8D%AE%E9%9B%86/ms-coco.md)
2. [PASCAL-VOC](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/%E6%95%B0%E6%8D%AE%E9%9B%86/pascal-voc.md)



### 评测方式
1. [mAP](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E7%9B%AE%E6%A0%87%E6%A3%80%E6%B5%8B%E7%AF%87/%E8%AF%84%E6%B5%8B%E6%8C%87%E6%A0%87/mAP.md)



# 优化篇

1. [正则项,加入正则项能有效避免模型过拟合同时使模型更加稀疏](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E4%BC%98%E5%8C%96%E7%AF%87/%E6%AD%A3%E5%88%99%E9%A1%B9.md)
2. [优化器，优化器就像灯塔指引网络往正确的方向收敛，一个好的优化器能够帮助网络快速收敛，跳出局部最优，训练过程更加稳定](https://github.com/Hanson0910/DL-Algorithm-Summary/blob/main/%E4%BC%98%E5%8C%96%E7%AF%87/%E6%AD%A3%E5%88%99%E9%A1%B9.md)