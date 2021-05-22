### A Simple and Strong Baseline for Irregular Text Recognition

[源码地址](https://github.com/wangpengnorman/SAR-Strong-Baseline-for-Text-Recognition)：lua语言 

[Paper]()：2019, 机构

#### 摘要

本文提出一种简单有效的baseline模型，通过字符级的监督标注，识别自然场景中不规则的文本。模型由31层的ResNet，LSTM编解码框架和一个2D-Attention模块构成。在不规则和规则的benchmarks上都达到STA水平。

#### 追溯相关工作

pass

#### 模型细节设计

![1613712210202](C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1613712210202.png)CNN层提取特征：

* 采用31层ResNet，所有kerner大小是3x3。做残差块时，输入输出维度不同，shutcut中加1x1卷积；

* 采用2x2和2x1池化。在水平方向保留更多信息，有利于对狭窄字符的识别；

* 输入整张图片，保持比例压缩到固定高度，宽度可变；

* 输出2D特征图，用于1）提取图片整体特征；2）作为Attention模块的上下文环境；

* **CNN细节结构**：卷积层stride和padding都是1，池化层没有padding；

  <img src="C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1613713183977.png" alt="1613713183977" style="zoom:67%;" />

LSTM编解码：

* 编码器：两层LSTM含512个隐含单元，每个时间步接受一列（最大池化后）特征数据，遍历W步后，第2层LSTM输出的隐含态hw（长度固定）作为整体的特征表示，用于初始化解码器

  <img src="C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1613713376902.png" alt="1613713376902" style="zoom:50%;" />

* 解码器：另一个两层LSTM（512单元），接受hw作为初始隐含态，每个时间步，接受上一步的输出（预测值和隐含态）作为输入，输入以one-hot向量+线性层的方式构建。训练时用gt序列代替预测序列作为输入。每一步的输出，是将当前LSTM的隐含态hi和Attention模块的输出gi拼接起来，再加一个线性层（把特征表示成类别空间，中文OCR类别空间通常很大）做softmax。

<img src="C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1613714018694.png" alt="1613714018694" style="zoom: 67%;" />

2D-Attention模块：

代替文本校正，自动适应形状，方向和排布不规则的文本。

* 考虑像素位置的相关性，构建Attention权重时加入8-领域的位置信息；

  <img src="C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1613714399292.png" alt="1613714399292" style="zoom:67%;" />

* 通过3x3的卷积实现领域操作，结合膨胀后的隐向量（[H，W，d]维）得到eij。再用1x1卷积+softmax得到2D的权重图；

<img src="C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1613716480786.png" alt="1613716480786" style="zoom:67%;" />

#### 其他细节

实验数据集：Syn90k，SVTP，CUTE80， COCO-Text

训练参数：

* 交叉熵Loss，ADAM优化器
* batch=32，Ir_init=0.001, 每1w步衰减0.9， 直到0.00001 

#### 实验结果

在公共benchmarks下的精度

* 规则文本+有字典；shi et al.2018-Aster表现最好（本文也不差）
* 不规则文本；本文表现很好

![1613717170458](C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1613717170458.png)

消融实验：

* 加入真实图片提高近10个点；
* 2D-Attention比1D提高2~3个点；
* 减少CNN和LSTM隐含层数量会降低性能;(CNN更明显)
* 改变下采样率（改变特征图大小）会导致性能小幅下降

![1613718029145](C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1613718029145.png)