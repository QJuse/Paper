### An End-to-End Trainable Neural Network for Image-based Sequence Recognition and Its Application to Scene Text Recognition
xiang Bai，2015，华中科技

#### 摘要

提出新的集成特征提取，序列建模和字符转录的一体化OCR识别框架，优点：1)端到端训练，2）能处理任意长度的序列，3）和之前的字典方法不冲突，无字典和基于字典的任务都可以用，4）模型高效且更小。提供在标准数据集IIIT-5K，SVT和IC上的测试结果，达到SOT水平。

#### 网络结构

从下到上包括一个CNN卷积层， RNN循环层和一个转录层，完成用同一个loss端到端训练

![1614936300070](C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1614936300070.png)

##### 特征序列提取

<img src="C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1615009042756.png" alt="1615009042756" style="zoom:67%;" />

* 整体参考VGG网络，由卷积层+最大池化层构成。

* 为适应英文识别，在第3和第4池化层采用1x2的kernel，结果会增加特征图的宽度，即特征序列的长度；产生矩形感受野，对识别狭长字符有帮助。固定图片高度为32，会产生25帧的特征。

* 将CNN输出的特征图，按列取为特征向量（列宽1pixel）, 每列特征对应于原图的一个矩形区域(感受野)，可以视为该区域的一个表示器；

  * 感受野：卷积用到的像素点范围，还是对应的缩放区域？（按本文是后者，但直观理解不应该是特征点对原图像素的敏感度吗？？）

  <img src="C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1615000955235.png" alt="1615000955235" style="zoom:67%;" />

* CNN输出的特征图是变长的，使用RNN将其表示成序列特征

  

##### 序列标签

* 使用BiLSTM构建RNN层，作用是从特征序列的每一帧中预测一个标签分布，大致有三点作用：
  * 有效的捕捉序列的上下文信息。例如宽字符需要多帧来表示，基于上下文更好的识别混淆字符
  * RNN能将误差以差分的方式传递给他的输入，可以组成统一的网络
  * 能从头到尾操作任意长度的序列
* 传统RNN和LSTM的差别
  * 传统RNN有梯度消失的问题（因为同一个导数连乘），限制上下文感知范围
  * LSTM由一个记忆单元，三个门控单元构成，记忆单元存储过去的文本，输入和输出gate允许存储长期信息，遗忘gate可以清除记忆单元的信息。LSTM可以捕捉长范围的依赖。
  * LSTM是定向的，而图片序列需要的文本两个方向都有用，因此联合两个LSTM构成双向的BiLSTM；同时在深度方向堆叠了两层BiLSTM（多层LSTM在语音识别中提升显著）
  * 使用BPTT反向传播误差。实践中构建了一个Map-to-Sequence的网络层连接CNN和RNN

<img src="C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1615003176209.png" alt="1615003176209" style="zoom:67%;" />

##### 转录解码模块

* 将RNN每一帧的输出转为字符序列，在无字典方式中，以得到字符序列概率最大的方式进行预测
* 采用CTC层进行转录，输入是一个序列，每个时间步包含字符的序列分布。[参考CTC细节]
* CTC没有模型结构，只是在做路径搜索计算



##### 训练设置

* 网络用SGD训练，CTC层用前后向算法传递误差，RNN层用BPTT。
* 优化时用ADADELTA自动计算每维的学习率，比动量方法收敛更快





#### 实验结果

* CRNN模型参数大小8.3M，存储大小33MB（4B单精度），便于嵌入式划

![1615010617164](C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1615010617164.png)