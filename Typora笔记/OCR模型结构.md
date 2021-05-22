### OCR模型结构

#### 2016--Learning to Read Irregular Text with Attention Mechanisms

问题域：提高对变形扭曲文字的泛化能力；

技术点：1）增加"密集字符检测"的辅助任务，更好的学习文本特定的视觉模式；2）通过分配的loss指导attention模型的训练

模型结构：[端到端训练，可适用不同大小的输入图片和字符标签]

![1612420549487](C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1612420549487.png)

* CNN模块提取图片特征 f(x)
* FCN模块输入f(x)，输出同原图大小一致的像素预测值y（0-1之间）；推理事去掉FCN
* 循环网络GRU，联合位置感知输入M和f(x)，解码出字符序列

核心观点：用空间位置信息引导注意力聚焦；构建2D坐标图，大小和特征f(x)一致，归一化到（-0.5, 0.5)之间；[文本识别建模成序列依赖，学习ROI和字符对应关系的问题]

定义了一个**字符检测任务辅助Attention**，通过loss惩罚字符定位不准确的情况

![1612702989198](C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1612702989198.png)

Latt怎么构建？需要什么额外的标注数据？

