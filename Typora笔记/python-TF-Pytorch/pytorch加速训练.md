### pytorch加速训练

#### 几点建议

* **换学习率schedule**
  * 1Cycle周期学习率：较小-较大-较小： https://pytorch.org/docs/stable/optim.html
  * torch.optim.lr_scheduler.CyclicLR；和torch.optim.lr_scheduler.OneCycleLR
* **DataLoader 设置worker和页锁定内存**--work
  * worker 数量的经验法则是将其设置为可用 GPU 数量的四倍；pin_memory=True
  * num_worker 不能设置太大，不然会线程阻塞或撑爆内存，一般设置为 cpu 核心数或gpu数量
* **调大batch**
  * **batch 大小加倍时，学习率也要加倍**



* **使用自动混合精度（AMP）**
  
  * 与单精度 (FP32) 相比，某些运算在半精度 (FP16) 下运行更快，而不会损失准确率
  
* 使用另一种优化器
  
  *  torch.optim.AdamW 实现；LARS 和 LAMB
  
* 使用梯度累积
  
  * 变相增大batch
  
* 

  - torch.nn.utils.clip_grad_norm

* **使用. as_tensor 而不是. tensor**
  
  * torch.tensor 总是会复制数据。使用 torch.as_tensor 或 torch.from_numpy 来避免复制数据。
  * 避免频繁在CPU和GPU之间传数据
  
  - 新创建的张量用device赋给GPU

* * 

    



[加速方法](https://www.zhihu.com/question/356829360/answer/903015862)

- >





#### 训练策略

* 使用低精度数据
  * 使用 `Apex` 的混合精度或者是PyTorch1.6开始提供的`torch.cuda.amp`模块来训练. 可以节约一定的显存并提速, 但是要小心一些不安全的操作如mean和sum

#### 代码层面 -- PASS

>```
>torch.backends.cudnn.benchmark = True
>```
>
>Do numpy-like operations on the GPU wherever you can
>
>Free up memory using `del` 
>
>Avoid unnecessary transfer of data from the GPU
>
>Use pinned memory, and use `non_blocking=True` to parallelize data transfer and GPU number crunching
>
>- 文档：[https://pytorch.org/docs/stable/nn.html#torch.nn.Module.to](https://link.zhihu.com/?target=https%3A//pytorch.org/docs/stable/nn.html%23torch.nn.Module.to)
>- 关于 `non_blocking=True` 的设定的一些介绍：Pytorch有什么节省显存的小技巧？ - 陈瀚可的回答 - 知乎 https://www.zhihu.com/question/274635237/answer/756144739
>
>网络设计很重要，外加不要初始化任何用不到的变量，因为 PyTorch 的初始化和 `forward` 是分开的，他不会因为你不去使用，而不去初始化
>
>合适的`num_worker` ： Pytorch 提速指南 - 云梦的文章 - 知乎 https://zhuanlan.zhihu.com/p/39752167（这里也包含了一些其他细节上的讨论）

#### 模型设计--PASS

>来自 ShuffleNetV2 的结论：（内存访问消耗时间，`memory access cost` 缩写为 `MAC`）
>
>- 卷积层输入输出通道一致：卷积层的输入和输出特征通道数相等时 MAC 最小，此时模型速度最快
>- 减少卷积分组：过多的 group 操作会增大 `MAC` ，从而使模型速度变慢
>- 减少模型分支：模型中的分支数量越少，模型速度越快
>- 减少 `element-wise` 操作：`element-wise` 操作所带来的时间消耗远比在 FLOPs 上的体现的数值要多，因此要尽可能减少 `element-wise` 操作（`depthwise convolution`也具有低 FLOPs 、高 MAC 的特点）
>
> 其他：
>
>- 降低复杂度：例如模型裁剪和剪枝，减少模型层数和参数规模
>- 改模型结构：例如模型蒸馏，通过知识蒸馏方法来获取小模型



### 推理加速



>
>
