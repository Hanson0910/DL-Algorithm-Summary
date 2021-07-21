# YoloV5
### 1. 网络结构
与yolo4不同，yolo5首先采用了focus层，设计了两种CSP结构，以Yolov5s网络为例，CSP1_X结构应用于Backbone主干网络，另一种CSP2_X结构则应用于Neck中。yolo3、yolo4、yolo5的网络结构如下图所示：
![image](https://img-blog.csdnimg.cn/2020091205304470.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zODg0MjgyMQ==,size_16,color_FFFFFF,t_70#pic_center)
![image](https://img-blog.csdnimg.cn/20200912053056600.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zODg0MjgyMQ==,size_16,color_FFFFFF,t_70#pic_center)
![image](https://img-blog.csdnimg.cn/20200912052742459.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zODg0MjgyMQ==,size_16,color_FFFFFF,t_70#pic_center)
#### 1.1 网络深度
![image](https://img-blog.csdnimg.cn/20200914141100919.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zODg0MjgyMQ==,size_16,color_FFFFFF,t_70#pic_center)
#### 1.2 网络宽度
![image](https://img-blog.csdnimg.cn/20200914141033152.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zODg0MjgyMQ==,size_16,color_FFFFFF,t_70#pic_center)
#### 1.3 Focus结构
![image](https://img-blog.csdnimg.cn/20200914095419472.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zODg0MjgyMQ==,size_16,color_FFFFFF,t_70#pic_center)
#### 1.4 CSP结构
Yolov4借鉴了CSPNet的设计思路，在主干网络中设计了CSP结构，但只有主干网络使用了CSP结构。
![image](https://img-blog.csdnimg.cn/20200914102451360.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zODg0MjgyMQ==,size_16,color_FFFFFF,t_70#pic_center)

Yolov5中设计了两种CSP结构，以Yolov5s网络为例，CSP1_X结构应用于Backbone主干网络，另一种CSP2_X结构则应用于Neck中。
https://img-blog.csdnimg.cn/20200914102443272.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zODg0MjgyMQ==,size_16,color_FFFFFF,t_70#pic_center
CSPNet(Cross Stage PartialNetwork):跨阶段局部网络，以缓解以前需要大量推理计算的问题。
- 增强了CNN的学习能力，能够在轻量化的同时保持准确性。
- 降低计算瓶颈。
- 降低内存成本。
- CSPNet通过将梯度的变化从头到尾地集成到特征图中，在减少了计算量的同时可以保证准确率。
CSPNet和PRN都是一个思想，将featuremap拆成两个部分，一部分进行卷积操作，另一部分和上一部分卷积操作的结果进行concate。
#### 1.5 neck
Yolov4的Neck结构中，采用的都是普通的卷积操作。而Yolov5的Neck结构中，采用借鉴CSPnet设计的CSP2结构，加强网络特征融合的能力。
![image](https://img-blog.csdnimg.cn/20200914113727493.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl8zODg0MjgyMQ==,size_16,color_FFFFFF,t_70#pic_center)
