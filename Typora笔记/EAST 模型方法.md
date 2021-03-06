### EAST 模型方法

#### 模型结构和loss怎么设计？解决了什么关键问题

#### 技术方法

* 神经网络模型，预测整张图片中的文本及其形状

* 模型使用FCN，输出逐个像素的预测；预测几何框时有阈值过滤和NMS（排除了候选框，文本区域格式化，字分割等中间过程）

  

* 借鉴DenseBox：image -- FCN -- 多通道像素级文本得分图和形状

  * 预测输出A：得分图，值在0-1之间：代表几何形状预测正确的置信度
  * RBOX旋转框：QUAD矩形框：不同的几何框设计不同的loss
  * 通过阈值来过滤，再通过NMS输出最终结果



#### 网络设计--特征图合并

![1602643128201](C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1602643128201.png)

* 大字需要用到网络后面层的信息，而准确监测小字需要网络早期层的信息。--必须使用不同层的特征。[HyperHet满足条件特征要求，但是合并大量通道到大的特征图，显著增加了后面阶段的计算量] 使用U-shape来合并特征图,使用较小的上采样分支
* 主干分支：PVANet：4层依次缩小1/2、提取4层特征
* 合并分支：U-shape:上采样上层特征图，和当前层连接，使用Conv1x1压缩计算量，在使用Conv3x3融合信息输出。最后一层再用Conv3x3产生最后输出。
* 输出Fs，Fg两张特征图，Fg代表有两类（矩形输出8通道，RBOX输出两张图5通道）[输出坐标形式！！]