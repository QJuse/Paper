### pytorch代码CPU和内存占比过大

* 与batch大小直接相关，但batch不能降低
* 与大dataloader的线程数有一定关系，不是核心问题

代码情况

* dataloader里有对batch图片重新处理的pipeline代码
* 数据集dataset个数有多个，都比较占内存--json|csv|txt标注



### dataloader功能分析

* dataset|sampler|collect_fn
  * 自定义的dataset中一个index对应一个样本
  * sampler用于定义采样方式，是一系列index组成的可迭代对象
* dataloader的数据管道机制--上千万数据加载慢-CPU开销大
  * 加载一个batch数据用时多久？CPU开销多少？能否用GPU处理的吗
  * 策略 https://www.yuque.com/lart/ugkv9f/ugysgn
    * IO提速：使用**LMDB**数据库，避免大量小图片的IO开销
    * 预处理提速：减小预处理操作，把resize等预处理事先保存下来；

### 策略

* 重新做离线数据清理，引入LMDB数据库；减小resize， 加噪声，pad预处理操作
* 定义sampler
* 定义transfors



### 收集模型中间层特征和梯度

* 固定某些模型某些梯度
* 

