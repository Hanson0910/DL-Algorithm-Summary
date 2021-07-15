- ### 常用数据集
    #### PASCAL VOC
- ##### 简介
        PASCAL VOC挑战赛 （The PASCAL Visual Object Classes ）是一个世界级的计算机视觉挑战赛, PASCAL全称：Pattern Analysis, Statical Modeling and Computational Learning，是一个由欧盟资助的网络组织。PASCAL VOC从2005年开始举办挑战赛，每年的内容都有所不同，从最开始的分类，到后面逐渐增加检测，分割，人体布局，动作识别（Object Classification 、Object Detection、Object Segmentation、Human Layout、Action Classification）等内容，数据集的容量以及种类也在不断的增加和改善。最后一届挑战赛是2012年，**现在论文用的比较多的数据集是VOC2007和VOC2012这两个年份的数据。**
    
    - ##### 类别
        PASCAL VOC 数据集一共包含20个类别，及其层级结构：
        ![image](https://arleyzhang.github.io/articles/1dc20586/1522997065951.png)
        从2007年开始，PASCAL VOC每年的数据集都是这个层级结构，总共四个大类：vehicle,household,animal,person，总共20个小类，预测的时候是只输出图中黑色粗体的类别
    
    - ##### 使用形似
        - 只用VOC2007的trainval 训练，使用VOC2007的test测试
        - 只用VOC2012的trainval 训练，使用VOC2012的test测试，这种用法很少使用，因为大家都会结合VOC2007使用
        - 使用 VOC2007 的 train+val 和 VOC2012的 train+val 训练，然后使用 VOC2007的test测试，这个用法是论文中经常看到的 07+12 ，研究者可以自己测试在VOC2007上的结果，因为VOC2007的test是公开的。
        - 使用 VOC2007 的 train+val+test 和 VOC2012的 train+val训练，然后使用  VOC2012的test测试，这个用法是论文中经常看到的 07++12 ，这种方法需提交到VOC官方服务器上评估结果，因为VOC2012 test没有公布。   
        - 先在 MS COCO 的 trainval 上预训练，再使用 VOC2007 的 train+val、 VOC2012的 train+val 微调训练，然后使用 VOC2007的test测试，这个用法是论文中经常看到的 07+12+COCO 。
        - 先在 MS COCO 的 trainval 上预训练，再使用 VOC2007 的 train+val+test 、 VOC2012的 train+val 微调训练，然后使用 VOC2012的test测试 ，这个用法是论文中经常看到的 07++12+COCO，这种方法需提交到VOC官方服务器上评估结果，因为VOC2012 test没有公布。
    
    - ##### 数据集数量
        ![image](https://arleyzhang.github.io/articles/1dc20586/1523033381386.png)
        黑色字体所示数字是官方给定的，由于VOC2012数据集中 test 部分没有公布，因此红色字体所示数字为估计数据，按照PASCAL 通常的划分方法，即 trainval 与test 各占总数据量的一半。
    
    - ##### 提交详情
        每一类都有一个txt文件，里面每一行都是测试集中的一张图片，每行的格式按照如下方式组织
        
            <image identifier> <confidence> <left> <top> <right> <bottom>