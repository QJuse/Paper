### What Is Wrong With STR Model Comparisons? Dataset and Model Analysis

[Paper]()：2019，京都大学

[源码地址](https://github.com/clovaai/deep-text-recognition-benchmark)

#### 摘要

训练和测评数据的不一致，导致对算法的评估缺少整体、公平的比较。本文同时采用多个数据集进行训练和测试，解决数据的不一致；提出一个四阶段的STR框架；分析了各模块对精度，速度和内存的影响；归纳了误识别类型；

#### 相关工作

2016-CRNN：奠定CNN+LSTM+CTC的框架

2016-Recursive recurrent nets with attention modeling for ocr in the wild

2016-Attn：Robust scene text recognition with automatic rectification

2017-Attn：Focusing attention: Towards accurate text recognition in natural images

#### 模型设计

![1613809876025](C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1613809876025.png)

* 图片校正：TPS方法（STN网络的变体）
* CNN特征提取：VGG，RCNN， ResNet
* 序列建模阶段：两层BiLSTM（平衡复杂度和内存时会去掉）
* 预测阶段：CTC， Attnetion

根据代码分析模型实现细节：

* CNN将高度32的图片编码成1D向量F，再用BiLSTM编码成文本向量Q
* **作Attention时利用的是LSTM编码后的向量，而非特征向量**
* 文本向量Q的作用是作为“源”特征，而不是LSTM的初始态[**尝试使用GRU**]

<img src="C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1613812689482.png" alt="1613812689482" style="zoom:67%;" />

<img src="C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1613812753538.png" alt="1613812753538" style="zoom:67%;" />



#### 实验结果

实现细节：

* MJ数据8.9M+ ST数据5.5M，batch=192，训练300k步 
* AdaDelta优化器，梯度阶段=5，权重初值用He方法

>联合数据集训练提高4个点精度，样本多样性比样本数量重要，复杂数据集比单一数据集好

分析了各模块组合模型，精度，参数量，速度的情况：

* Attention的精度比CTC高1个多点，但推理速度慢一倍
* 用ResNet比RCNN和VGG精度高，参数量大很多

![1613811141392](C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1613811141392.png)

误识别情况分类：

* 归纳了6类误识别情况：艺术字体，竖排文字，**特定字符，遮挡，低分辨率，标注噪声**

![1613811526452](C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1613811526452.png)