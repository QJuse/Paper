### 知识蒸馏（KD）摘要

#### Paper方法

知识蒸馏是模型压缩的方法之一，以下是主要的模型压缩方法：

| 模型压缩算法     | 模型加速                     | 优化工具和库         |
| ---------------- | ---------------------------- | -------------------- |
| 参数量化-PQ      | Op级快速算法                 | TensorRT（nvidia）   |
| 结构剪枝-PP      | Layer级快速算法              | TVM（张量虚拟机）    |
| 网络架构搜索-NAS | 硬件计算单元优化             | Tensor Comprehension |
| 知识蒸馏-KD      | （CPU/GPU/NPU）（FPGA/ASIC） | Distiller（Intel）   |
| 矩阵低秩分解     |                              |                      |

KD在06年由Bucilua提出，而在14年Hilton正式定义了KD，并提出相应的训练方法。模型在有监督的数据上训练，框架如下：

<img src="C:\Users\viruser.v-desktop\Pictures\v2-fa3adaddf20b65912a6a40575c092233_b.jpg" alt="v2-fa3adaddf20b65912a6a40575c092233_b" style="zoom:80%;" />

##### [KD主要方法](https://bbs.cvmart.net/topics/4100)

* Logits(Response)-based：让S网络学习T网络的预测输出，基于soft softmax的loss是核心
  * Distilling the Knowledge in a Neural Network Hilton NIPS 2014
  * Deep mutual learning CVPR 2018
  * On the efficacy of knowledge distillation, ICCV 2019
  * Self-training with noisy student improves imagenet classification 2019
  * Training deep neural networks in generations: A more tolerant teacher educates better students AAAI 2019
  * Distillation-based training for multi-exit architectures ICCV 2019
* Feature-based：让S网络学习T网络的中间特征图，尽可能相近
  * Fitnets: Hints for thin deep nets. ICLR 2015
  * Paying more attention to attention: Improving the performance of convolutional neural networks via attention transfer. ICLR 2017
* Relation-based：是学习输入-隐层-输出之间的关系，相当于一个黑盒
  * A gift from knowledge distillation: Fast optimization, network minimization and transfer learning CVPR 2017
  * Similarity-preserving knowledge distillation ICCV 2019



#### KD[和迁移学习主要工具](https://cloud.tencent.com/developer/article/1559625)

* PaddleSlim：支持若干知识蒸馏算法，可以在teacher网络和student网络任意层添加组合loss
* **Distiller**：是Intel基于Pytorch开源的模型优化工具，支持Hinton等人提出的Knowledge distillation算法
* Knowledge-Distillation-Zoo[项目](https://github.com/AberHu/Knowledge-Distillation-Zoo)
* deep-transfer-[learning](https://github.com/easezyc/deep-transfer-learning)
* [TextBrewer](https://github.com/airaria/TextBrewer/blob/master/README_ZH.md#%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B)



#### 总结

我们采用TextBrewer框架，基于Feature-based和Logist-based两种方法对OCR模型进行进行蒸馏；

