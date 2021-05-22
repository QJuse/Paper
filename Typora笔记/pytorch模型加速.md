### pytorch模型加速

#### 训练加速

* **换学习率schedule**加速收敛：     [TODO]

  * 1Cycle周期学习率：较小-较大-较小 

    ```
    torch.optim.lr_scheduler.CyclicLR；和torch.optim.lr_scheduler.OneCycleLR
    ```

* **DataLoader 设置worker和页锁定内存 ** :   [加速不明显]

  * worker 数量的经验法则是将其设置为可用 GPU 数量的四倍；pin_memory=True
  * worker 不能设置太大，不然会线程阻塞或撑爆内存，一般设置为 cpu 核心数或gpu数量

* **调大batch：**      [资源受限]

  * batch 大小加倍时，学习率也要加倍
  * 梯度累积，变相加大batch（学习率是否要加倍？）

  

* **使用另一种优化器**：  [TODO]

  * torch.optim.AdamW 实现；LARS 和 LAMB

* 使用梯度裁剪加速收敛：

  * torch.nn.utils.clip_grad_norm

* **使用. as_tensor 而不是. tensor**

  - torch.tensor 总是会复制数据。使用 torch.as_tensor 或 torch.from_numpy 来避免复制数据。
  - 避免频繁在CPU和GPU之间传数据

  - 新创建的张量用device赋给GPU

  

##### 从GPU和cuda入手 [加速方法](https://www.zhihu.com/question/356829360/answer/903015862)

- 单机多GPU分配Tensor
  - `PyTorch` 中所有 GPU 的运算默认都是异步操作。但在 CPU 和 GPU 或者两个 GPU 之间的数据复制是需要同步的`PyTorch` 中所有 GPU 的运算默认都是异步操作。但在 CPU 和 GPU 或者两个 GPU 之间的数据复制是需要同步的。
- 数据并行多GPU？？
  - torch.nn.DataParallel
- 使用并发：**torch.multiprocessing**是对 Python 的 `multiprocessing` 模块的一个封装
- 其他建议
  - 使用`apex.DistributedDataParallel`替代`torch.DataParallel`，使用`apex`进行加速；



#### 预处理提速--PASS

- 减少每次预处理的操作，将一些固定的操作如resize事先处理好，保存起来。

- Linux上用GPU来加速预处理，借助[DALI库 ](https://docs.nvidia.com/deeplearning/dali/user-guide/docs/examples/getting%20started.html)  TODO

  - > tasks such as: load data from disk, decode, crop, random resize, color and spatial augmentations and format conversions, are mainly carried out on the CPUs, limiting the performance and scalability of training and inference

#### IO提速

- 使用[mmcv](https://github.com/open-mmlab/mmcv)库对数据读取和处理比较高效，使用[FileClient类](https://zhuanlan.zhihu.com/p/339190576)解耦数据读取和解码
- 使用更快的图片处理：opencv > PIL , 存成bmp图
- 大规模小图转成单独二进制文件：TFRecord，**lmdb**等

#### 预读取数据 

- **prefetch_generator 加速 Pytorch 数据读取 ** --Work

> Pytorch默认的DataLoader会创建一些worker线程来预读取新的数据，但是除非这些线程的数据全部都被清空，这些线程才会读下一批数据。使用prefetch_generator，我们可以保证线程不会等待，每个线程都总有至少一个数据在加载

```python
from torch.utils.data import DataLoader
from prefetch_generator import BackgroundGenerator
class DataLoaderX(DataLoader):
    def __iter__(self):
        return BackgroundGenerator(super().__iter__())
    # 用DataLoaderX替换原本的DataLoader
```

- **使用cuda.Steam加速数据copy到GPU ** -- fail

  - https://zhuanlan.zhihu.com/p/97190313 
  - [系统方法](http://www.go2live.cn/nocate/%E7%BB%99%E8%AE%AD%E7%BB%83%E8%B8%A9%E8%B8%A9%E6%B2%B9%E9%97%A8-pytorch%E5%8A%A0%E9%80%9F%E6%95%B0%E6%8D%AE%E8%AF%BB%E5%8F%96.html)

  > 将下一个batch数据的预读取（涉及cpu->gpu）与当前batch的前向传播并行处理，就必须：
  >
  > （1） cpu上的数据batch必须pinned;
  >
  > （2）预读取操作必须在另一个stream上进行



#### 

> ```
> torch.backends.cudnn.benchmark = True
> ```
>
> Do numpy-like operations on the GPU wherever you can
>
> Free up memory using `del` 
>
> Avoid unnecessary transfer of data from the GPU
>
> Use pinned memory, and use `non_blocking=True` to parallelize data transfer and GPU number crunching
>
> - 文档：[https://pytorch.org/docs/stable/nn.html#torch.nn.Module.to](https://link.zhihu.com/?target=https%3A//pytorch.org/docs/stable/nn.html%23torch.nn.Module.to)
> - 关于 `non_blocking=True` 的设定的一些介绍：Pytorch有什么节省显存的小技巧？ - 陈瀚可的回答 - 知乎 https://www.zhihu.com/question/274635237/answer/756144739
>
> 网络设计很重要，外加不要初始化任何用不到的变量，因为 PyTorch 的初始化和 `forward` 是分开的，他不会因为你不去使用，而不去初始化
>
> 合适的`num_worker` ： Pytorch 提速指南 - 云梦的文章 - 知乎 https://zhuanlan.zhihu.com/p/39752167（这里也包含了一些其他细节上的讨论）

#### 模型设计--PASS

> 来自 ShuffleNetV2 的结论：（内存访问消耗时间，`memory access cost` 缩写为 `MAC`）
>
> - 卷积层输入输出通道一致：卷积层的输入和输出特征通道数相等时 MAC 最小，此时模型速度最快
> - 减少卷积分组：过多的 group 操作会增大 `MAC` ，从而使模型速度变慢
> - 减少模型分支：模型中的分支数量越少，模型速度越快
> - 减少 `element-wise` 操作：`element-wise` 操作所带来的时间消耗远比在 FLOPs 上的体现的数值要多，因此要尽可能减少 `element-wise` 操作（`depthwise convolution`也具有低 FLOPs 、高 MAC 的特点）
>
> 其他：
>
> - 降低复杂度：例如模型裁剪和剪枝，减少模型层数和参数规模
> - 改模型结构：例如模型蒸馏，通过知识蒸馏方法来获取小模型







#### 推理加速

> 在推理中使用低精度（`FP16` 甚至 `INT8` 、二值网络、三值网络）表示取代原有精度（`FP32`）表示：
>
> - `TensorRT`是 NVIDIA 提出的神经网络推理(Inference)引擎，支持训练后 8BIT 量化，它使用基于交叉熵的模型量化算法，通过最小化两个分布的差异程度来实现
> - Pytorch1.3 开始已经支持量化功能，基于 QNNPACK 实现，支持训练后量化，动态量化和量化感知训练等技术
> - 另外 `Distiller` 是 Intel 基于 Pytorch 开源的模型优化工具，自然也支持 Pytorch 中的量化技术
> - 微软的 `NNI` 集成了多种量化感知的训练算法，并支持 `PyTorch/TensorFlow/MXNet/Caffe2` 等多个开源框架
>
> 【参考】：
>
> - 有三AI：【杂谈】[当前模型量化有哪些可用的开源工具？]([https://mp.weixin.qq.com/s?__biz=MzA3NDIyMjM1NA==&mid=2649037243&idx=1&sn=db2dc420c4d086fc99c7d8aada767484&chksm=8712a7c6b0652ed020872a97ea426aca1b06adf7571af3da6dac8ce991fd61001245e9bf6e9b&mpshare=1&scene=1&srcid=&sharer_sharetime=1576667804820&sharer_shareid=1d0dbdb37c6b95413d1d4fe7d61ed8f1&exportkey=A6g%2Fj50pMJYVXsedNyDVh9k%3D&pass_ticket=winxjBrzw0kHErbSri5yXS88yBx1a%2BAL9KKTG6Zt1MMS%2FeI2hpx%2BmeaLsrahnlOS#rd](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/s%3F__biz%3DMzA3NDIyMjM1NA%3D%3D%26mid%3D2649037243%26idx%3D1%26sn%3Ddb2dc420c4d086fc99c7d8aada767484%26chksm%3D8712a7c6b0652ed020872a97ea426aca1b06adf7571af3da6dac8ce991fd61001245e9bf6e9b%26mpshare%3D1%26scene%3D1%26srcid%3D%26sharer_sharetime%3D1576667804820%26sharer_shareid%3D1d0dbdb37c6b95413d1d4fe7d61ed8f1%26exportkey%3DA6g%2Fj50pMJYVXsedNyDVh9k%3D%26pass_ticket%3DwinxjBrzw0kHErbSri5yXS88yBx1a%2BAL9KKTG6Zt1MMS%2FeI2hpx%2BmeaLsrahnlOS%23rd))





**网络 inference 阶段 Conv 层和 BN 层融合**

【参考】

- https://zhuanlan.zhihu.com/p/110552861
- PyTorch本身提供了类似的功能，但是我没有使用过，希望有朋友可以提供一些使用体会：[https://pytorch.org/docs/1.3.0/quantization.html#torch.quantization.fuse_modules](https://link.zhihu.com/?target=https%3A//pytorch.org/docs/1.3.0/quantization.html%23torch.quantization.fuse_modules)
- 网络inference阶段conv层和BN层的融合 - autocyz的文章 - 知乎 [https://zhuanlan.zhihu.com/p/48](https://zhuanlan.zhihu.com/p/48005099)

***多分支结构融合成单分支***

- ACNet、RepVGG这种设计策略也很有意思
- - RepVGG|让你的ConVNet一卷到底，plain网络首次超过80%top1精度：[https://mp.weixin.qq.com/s/M4Ks](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/s/M4Kspm6hO3W8fXT_JqoEhA)

## **项目推荐**

- 基于 Pytorch 实现模型压缩（[https://github.com/666DZY666/model-compression](https://link.zhihu.com/?target=https%3A//github.com/666DZY666/model-compression)）：
- 量化：8/4/2 bits(dorefa)、三值/二值(twn/bnn/xnor-net)
- 剪枝：正常、规整、针对分组卷积结构的通道剪枝
- 分组卷积结构
- 针对特征二值量化的BN融合

