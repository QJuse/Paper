### Decoupled Attention Network for Text Recognition

来源：20-AAAI、华南理工

资料：[论文](https://arxiv.org/abs/1912.10205)、[代码][https://github.com/Wang-Tianwei/Decoupled-attention-network]

#### 背景和思路

目前端到端的文本识别，主要基于LSTM+CTC和seq2seq+Attention两种框架。CTC的缺点是依赖于独立输出的假设，而Attn方法能更好的利用序列的相关信息，主流研究的较多。

Attn方法主要通过**学习特征权重**来解决字符对齐的问题，但传统的Attn方法需要两个输入：1）encoder的图片特征；2）历史的解码信息。实践中发现这种Attn方法会有**字符特征对齐不够准确**的问题。

![](imgs\1593604338168.png)

论文认为：传统Attn方法利用历史的预测结果来做对齐操作，这样一旦历史预测结果有错误，误差就很容易在之后的对齐中有**积累效应**，导致越来越对不准。

论文提出的新思路是：将字符对齐和解码预测两个过程解耦，直接拿掉解码阶段的反馈。字符对齐本质上是在做特征匹配，论文从图片特征融合的角度来构建新的注意力方法。



#### 算法关键点

DAN算法主要包括三个模块：**CNN特征提取**模块，**CAM对齐模块**和**RNN解码模块**。CNN计算图片的特征图，CAM也计算一张特征图，两者融合后，送入解码模块。

![](imgs\1593593990397.png)

* **对齐模块CAM**

借鉴FPN的思路，先对ResNet提取的各层特征图做下采样，再融合在一起得到特征图M；再借鉴FCN的思路， 在特征图M上做编解码得到一张注意力特征图 [T, H/r, W/r]。

![](imgs\1593594060005.png)

* **RNN解码模块**

  接受融合后的特征向量C，将C送入GRU组成的RNN网络作预测，最后接一层线性层做预测输出。

<img src="imgs\1592635542642.png" style="zoom:50%;" />

文本向量C的计算（即怎么融合两张特征图，构成一个RNN序列输入）：

![](imgs\1592633144770.png)

```python
C = feature.view(nB, 1, nC, nH, nW) * A.view(nB, nT, 1, nH, nW) # 扩展操作
C = C.view(nB,nT,nC,-1).sum(3).transpose(1,0)  # 求和操作
```

求解GRU的隐含状态；t时刻的预测器输出以及 loss函数：

![](imgs\1592633956322.png)

![](imgs\1592633892253.png)

![](imgs\1592634071962.png)

#### 算法效果

![](imgs\1593602466540.png)

H/r=1就是1D识别器，适合长的规则文本的识别；H/r > 1就是2D识别器，适合不规则文本的识别。在IC13和IC03等数据集上论文的实验效果还是比较先进的。

值得学习的地方：

* 特征融合的思路，提取更准确更丰富的特征
* 在特征输入和输出层加dropout层。







### DAN代码解析——使用自己的数据集训练

#### 训练loops流程

```python
main.py 
-- model = Load_network()  # 从配置字典中，构建fe,cam, dtd 三个模型
	cfgs_hw.py  # 配置文件，导入DAN.py中的模型类，dataset_hw.py中的数据类
-- train_loader = load_dataset()  # 同理，先产生dataset,再用DataLoader
-- encdec = cha_encdec() # 标签编码器   utils.py
-- 训练loops：
   1.标签处理
   2.损失和acc计算
   3.梯度阶段
 
```



#### 数据馈入[独立拆出去]

源代码使用lmdb数据库（读取高效，多线程）

```python
# LineGenerate类
	从Imdb数据路径中加载，image和label列表收集图片和标签信息。
    1.图片大小调整，保持比例，放到固定图的中间。相当于有boader
    2.图片增强：Augment.Distort/Stretch/Perspective 对大小处理后的图片增强 # 用imgaug重构
    3.归一化处理减去0.5/0.5
   # 为什么分word_gen 和line_gen  
   # 策略：吸收图片预处理的方法，Data的代码则全部替换掉
```

