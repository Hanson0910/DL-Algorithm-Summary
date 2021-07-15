- ##### 简介
    MS COCO的全称是Microsoft Common Objects in Context，起源于微软于2014年出资标注的Microsoft COCO数据集，与ImageNet竞赛一样，被视为是计算机视觉领域最受关注和最权威的比赛之一。COCO数据集是一个大型的、丰富的物体检测，分割和字幕数据集。这个数据集以scene understanding为目标，主要从复杂的日常场景中截取，图像中的目标通过精确的segmentation进行位置的标定。图像包括91类目标，328,000影像和2,500,000个label。目前为止有语义分割的最大数据集，提供的类别有80 类，有超过33 万张图片，其中20 万张有标注，整个数据集中个体的数目超过150 万个。

- ##### 数据集任务
    - object detection: 目标检测；

    - keypoint detection: 关键点检测；

    - stuff segmentation: stuff没有固定形状的物体，例如天空、草地等，分割任务；

    - panoptic segmentation: 全景分割，图片中things，stuff等全被分割；

    - image captioning: “看图说话”，一个图片中的场景描述；
    
- ##### 数据集类别
    91个类别，包括things + stuff，实际目标检测只有80个类别，就是things。

- ##### 数据集划分

    数据集有2014年(2014 Train images, 2014 Val images, 2014 Test images), 2015年(2015 Test images) 和 2017年(2017 Train images, 2017 Val images, 2017 Test images). 其中,训练集和验证集有标签(2014 Train/Val annotations, 2017 Train/Val annotations),而测试集没有标签.

    在2014年数据集中,训练集82783张,验证集40504张,测试集40775张. 另外,验证集分为两部分,miniVal有5000张,剩下的35504张图像和训练集称为Trainval35k (Trainval35k==train2014+val2014-minival2014.). 通常在论文中使用Trainval35k当作训练集.

    在2015年测试集中有81434张图像.

    在2017年数据集中,训练集118287张,验证5000张,测试集40670张. 其中,训练集就是Trainval35k(82783+40504-5000=118287), 验证集就是miniVal. 即train2017==trainval35k==train2014+val2014-minival2014==train2014+val2014-val2017.