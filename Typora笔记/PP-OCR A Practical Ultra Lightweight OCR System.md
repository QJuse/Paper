### PP-OCR: A Practical Ultra Lightweight OCR System

#### 摘要

OCR应用：办公自动化OA，工业自动化，在线教育，地图产品

OCR挑战：多样的文本形式，计算性能

本文成果：实用的轻量级OCR系统

* 开放中英文识别预训练模型，3.5M模型识别6622个中文字符；2.8M模型识别63个字符；文本检测器97K图片，方向分类器600K图片，识别器17.9M图片
* 提供策略包增强模型能力同时减少模型大小，能迁移到其他的语言任务，如法语，日语和韩语
* [开源代码](https://github.com/PaddlePaddle/PaddleOCR)

#### 介绍

##### 文本多样性

自然场景文本STR，透视，尺度，弯曲，遮挡，字体，多语言，模糊，光照等都是动态变量；文档文本实践中应用多，主要是高密集和长文本的问题需要解决，同时**还原成结构化信息**

![1614911101132](C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1614911101132.png)

##### 计算效率

实践应用中往往对计算敏感，CPU比GPU更加经济，还有很多的场景需要运行在嵌入式设备上，例如手机等，平衡模型大小和性能有很大的价值，PP-OCR包括文本检测，框分类校正和文本识别三个模块

![1614911903477](C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1614911903477.png)

##### 文本检测

定位图片中的文本位置。PP-OCR采用分割网络DB（2020）。采用轻量化backbone, Head， 裁减SE单元，FPGM裁减4个策略减小模型，采用余弦学习率衰减，学习率Warm-up加速收敛

##### 边框分类校正

将边框转化成水平的矩形边框，用一个分类器识别文本方向。采用轻量化backbone，PACT裁减策略减小模型，采用数据增强，增大图片分辨率提高准确度。

##### 文本识别

PP-OCR采用CRNN框架（2016）。采用轻量化backbone，Head和PACT量化减少模型尺寸， 采用余弦学习率衰减，学习率Warm-up加速收敛， 采用数据增强，预训练模型，特征图分辨率，参数正则化提高识别准确率。

##### 数据集构建--数据限制

实践OCR系统构建了一个大的中英文数据集，文本检测使用97K图片，方向分类使用600K图片，文本识别使用17.9M图片；另外挑选了一个小的数据集用于切除实验，利于快速的选择策略。



#### 具体模型和策略

##### 文本检测

<img src="C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1614914000536.png" alt="1614914000536" style="zoom: 67%;" />

* 轻量级backbone：
  * MobileNetV3推理时间一样精度最好，采用MobileNetV3_large_x0.5(PaddleClas提高大量网络结构)
* 轻量Head:
  * 采用FPN结构提高对小文本区域的检测，将通道数从256减小到96（从7M将到4.1M，精度基本不变）
* 去除SE模块：通道自注意机制
  * MobileNetV3有大量SE单元，但当输入分辨率增大到640x640时精度提升受限，但时间却很高。去除SE单元，模型大小从4.1M降到2.5M，精度基本不变。

<img src="C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1614915291671.png" alt="1614915291671" style="zoom:67%;" />

* 余弦学习率衰减：
  * 相比阶段学习率规划，整个过程中保持相对大的学习率，收敛更慢，但有利于提高精度

<img src="C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1614915734239.png" alt="1614915734239" style="zoom:67%;" />

* 学习率Warm-up:
  * 有助于提高分类精度（2019），在开始阶段使用一个较小的lr，保证数值稳定，然后再使用初始化学习率。对于检测任务，实验效果很好。
* **FPMG裁减：**
  * 找到源模型中不重要的子网络结构（2019）， 使用几何平均作判别标准，卷积层中的每个滤波器作为欧氏空间中的一个点，计算每个点的几何平均，去除值相近的点。每一层的压缩比很重要，会导致精度下降，PP-OCR参考（2016）方法，计算每一层的敏感度，评估冗余，然后进行裁减

<img src="C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1614922334780.png" alt="1614922334780" style="zoom:67%;" />

##### 方向分类器

* 轻量级backbone：
  * MobileNetV3_small_x0.35，方向分类任务相对简单，使用大backbone精度没有提升
* 数据增强：
  * 基础增强BDA（2020）， 旋转，透视扭曲，运动模糊，高斯噪声，随机增加到训练数据中。
  * **RandA-2020**增强。其他数据增强工作：AutoA-2019， CutOut-2017， **RandErasing-2020**， Hide&Seek-2017, GridMask-2020, Mixup-2017, Cutmix-2019，大部分并无work
* 扩大输入分辨率：
  * 提升分辨率会增加准确率，由于backbone是轻量的并不会显著增加计算量；
  * 输入分辨率从[32，100]提升到[48， 192]
* PACT量化：使用PaddleSlim工具
  * 量化减小推理时间，离线量化不需要重训练，参考定点量化方法使用KL散度和移动平均确定量化参数；在线量化在训练过程中确定量化参数，精度损失会小
  * PACT-2018首先去除激活值得异常点，然后学习量化尺度；修改PACT量化函数同时适应swish和ReLu。

<img src="C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1614924282788.png" alt="1614924282788" style="zoom: 67%;" />

<img src="C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1614924458000.png" alt="1614924458000" style="zoom:67%;" />

##### 文本识别

* 轻量级backbone：

  * MobileNetV3_small_x0.5， x1.0效果也好，大小仅增加2M

* 轻量级head：

  * 序列特征的维度对文本识别器的大小有重大影响，并不是更高的维度就有更好的序列特征表示，PP-OCR按经营采用48

* **数据增强：**BDA增强，TIA-2020增强，根据控制点进行扭曲

  <img src="C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1614924736912.png" alt="1614924736912" style="zoom:67%;" />

* 余弦学习率规划和许习率Warm-up：参见文本检测
* **特征图分辨率：**
  
  * 适应中文识别将高宽设置为[32, 320]，修改原MobileNetV3下采样的stride适应文本识别任务，下采样的stride通过影响特征图分辨率进而影响准确度。
* **参数正则化：**
  
  * 使用L2正则化，网络的权重倾向于小的值，整个网络的参数趋向于0，能提高网络的泛化性能
* **预训练模型：**
  * 数据量小时，迁移分类和检测任务预训练的模型能加快收敛并提高精度
  * 在大多数场景中如果有10M以上的样本（尽管大部分是合成的）也能显著提高精度
* PACT量化：参见方向分类，目前跳过了LSTM（量化比较复杂）





#### 实验部分

##### 数据集：

构建了大规模的中英文数据集。真实图片68K来源于公开数据集和百度图片搜索（主要是文档文本）

* 文本检测：LSVT-19, RCTW-17，MTWI-18, CASIA-10K-18, SROIE -19, MLT-19, BDI-11, MSRA-TD500 -12，CCPD-19
* 方向识别：LSVT-19, RCTW-17, MTWI-18；500K合成图片用垂直字体合成，测试图片均来自真实场景
* 文本识别：LSVT-19, RCTW-17, MTWI-18，CCPD-19，16M合成图片聚焦不同的背景，变换，旋转，透视，线干扰，噪声和垂直文本等，（语料来自真实场景图片）

![1614925875450](C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1614925875450.png)

构建快速的开发数据集：4k图片用于模型检测，300K用于文本识别；另外收集了300张不同场景的图片测试整个系统。



#### 总结

* 模型轻量化方向：剪枝量化，轻量化backbone和head
* 技术细节OCR
  * 训练技巧：warm-up，和学习率规划
  * 准确率：**DB模型**，L2正则化， 扩大分辨率，特征图分辨率
    * Seq框架，特征Attention，二值化，超分辨率
  * **模型技术工具**
* 数据集收集
  * 快速策略验正，业务图片
  * 图片模板，字符集
* **文档结构化模型**





### PP-OCR技术索引

* MobileNet_v3架构：[SE-block](Squeeze-and-excitation
  networks-2018)
* ResNet架构
* 检测头FPN架构
* DB文本检测算法
* CRNN文本识别算法
* [Learning Rate Warm-up](Bag of tricks for image classification with convolutional
  neural networks-2019) 
* [FPGM Pruner](Filter
  pruning via geometric median for deep convolutional neural
  networks acceleration-2019)  [敏感度](Pruning filters for efficient convnets)

Data Augmentation



技术点过：

