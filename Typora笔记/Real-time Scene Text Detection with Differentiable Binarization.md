### Real-time Scene Text Detection with Differentiable Binarization

2019-华中科技-白翔团队

#### 简况和摘要：

DB是采用实例分割的方法检测场景文本。二值化后处理，将CNN产生的概率图，转换成文本边框和区域。DB方法在分割网络中自动处理二值化过程（自适应设置阈值，避免人工设置阈值）

* 简化后处理，增强检测性能
* 对小网络效果也不错，训练速度快，样本需求少

**不同算法性能的大致比较**---（快速的尝试过）

<img src="C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1617696705389.png" alt="1617696705389" style="zoom:80%;" />

选择td500作为测试集比较不同算法性能。使用F-评价标准（召回和准确结合）

* PixelLink-DB算法
* corner--CRAFT
* PSENet: 强调后处理提高精度

**基于分割的方法的pipe：**

* 得到分割的特征图-->对图进行二值化-->像素聚集成文本边框

<img src="C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1617697081549.png" alt="1617697081549" style="zoom:80%;" />

**采用回归的常见方法**，字符层文本框聚合的方法，无法处理不规则形状的文本。TextBoxes（++）， SegLink，RRD（基于SSD方法）， EAST基于PVANet提高检测速度



**DB方法：**

Unet架构生成1/4特征图F-->预测成概率图P和阈值图T-->处理成DB图

![1617698045470](C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1617698045470.png)

**标准二值化：**大于某个阈值是1，其他是0；不可微分，不能在网络中通过梯度优化

**微分二值化：**

![1617698384507](C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1617698384507.png)

B是近似二值化图，T是网络预测的自适应阈值图，k是功率因子，经验的取50，P是预测的概率图

