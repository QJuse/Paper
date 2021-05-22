#### Pytorch学习率调整策略

通过torch.optim.lr_scheduler接口实现，策略分为三大类：

> a.有序调整: 等间隔调整(Step)，按需调整学习率(MultiStep)，指数衰减调整(Exponential)和 余弦退火CosineAnnealing
>
>b.自适应调整：自适应调整学习率 ReduceLROnPlateau
>
>c.自定义调整：自定义调整学习率 LambdaLR



##### StepLR 等间隔

```
torch.optim.lr_scheduler.StepLR(optimizer, step_size, gamma=0.1, last_epoch=-1)
```

调整间隔为 step_size，指的是epoch。调整倍数为 gamma 倍。last_epoch，取-1时学习率为初始值，中间意外中断可设置该值，重新从上一个epoch的学习率开始（而非初始lr）

##### MultiStepLR 按需调整

```
torch.optim.lr_scheduler.MultiStepLR(optimizer, milestones, gamma=0.1, last_epoch=-1)
```

milestones代表各epoch的一个list，元素递增[30， 80， 120]。学习率调整倍数gamma。

>按设定的间隔调整学习率。这个方法适合**后期调试使用**，观察 loss 曲线，为每个实验定制学习率调整时机。

##### ExponentialLR 指数衰减

```
torch.optim.lr_scheduler.ExponentialLR(optimizer, gamma, last_epoch=-1)
```

按指数衰减公式调整学习率，lr = lr*gamma^epoch。gamma是学习率调整倍数的底，指数是epoch。

##### CosineAnnealingLR 余弦退火

```
torch.optim.lr_scheduler.CosineAnnealingLR(optimizer, T_max, eta_min=0, last_epoch=-1)
```

以余弦函数为周期，在每个周期最大值时重置学习率， 一个周期内先下降后上升。初始学习率为最大学习率。T_max是学习率迭代周期，T_max个epoch后重新设置。eta_min最小学习率，一个周期内最小的学习率。



##### **ReduceLROnPlateau 自适应调整学习率** 

> 当某指标不再变化（下降或升高），调整学习率，这是非常实用的学习率调整策略。
>
> 例如，当验证集的 loss 不再下降，或者监测验证集的 accuracy，当accuracy 不再上升时，则调整学习率。

```
torch.optim.lr_scheduler.ReduceLROnPlateau(optimizer, mode='min', factor=0.1, patience=10, verbose=False, threshold=0.0001, threshold_mode='rel', cooldown=0, min_lr=0, eps=1e-08)
```

参数解释：

```
mode(str)- 模式选择。min 表示当指标不再降低(如监测loss)， max 表示当指标不再升高(如监测 accuracy)
factor(float)- 学习率调整倍数(等同于其它方法的 gamma)，即学习率更新为 lr = lr * factor
patience(int)- 忍受该指标多少个 step 不变化。（此处是epoch吗？）
verbose(bool)- 是否打印学习率信息
cooldown(int)- “冷却时间“，当调整学习率之后，先让模型再训练一段时间，再重启监测模式
min_lr(float or list)- 学习率下限，可为 float，或者 list，当有多个参数组时，可用 list 进行设置。
eps(float)- 学习率衰减的最小值，当学习率变化小于 eps 时，则不调整学习率。
```



##### LambdaLR 自定义调整学习率

```
torch.optim.lr_scheduler.LambdaLR(optimizer, lr_lambda, last_epoch=-1)
```

lr_lambda(function or list)- 一个计算学习率调整倍数的函数，输入通常为 step，当有多个参数组时，设为 list。

lr=base_lr∗lmbda(self.last_epoch)。fine-tune 中十分有用，我们不仅可为不同的层设定不同的学习率，还可以为其设定不同的学习率调整策略。





### 实践测试

* 和代码的结合

* 怎么选择学习率策略？

  [提供测试代码](https://www.jianshu.com/p/a20d5a7ed6f3)

  [blog2](https://www.cnblogs.com/jiangkejie/p/10127325.html)

```
import torch
import torch.optim as optim
from torch.optim import lr_scheduler
from torchvision.models import AlexNet
import matplotlib.pyplot as plt


model = AlexNet(num_classes=2)
optimizer = optim.SGD(params=model.parameters(), lr=0.05)

# lr_scheduler.StepLR()
# Assuming optimizer uses lr = 0.05 for all groups
# lr = 0.05     if epoch < 30
# lr = 0.005    if 30 <= epoch < 60
# lr = 0.0005   if 60 <= epoch < 90

scheduler = lr_scheduler.StepLR(optimizer, step_size=30, gamma=0.1)
plt.figure()
x = list(range(100))
y = []
for epoch in range(100):
    scheduler.step()
    lr = scheduler.get_lr()
    print(epoch, scheduler.get_lr()[0])
    y.append(scheduler.get_lr()[0])

plt.plot(x, y)
```





#### [优化器和学习率调整](https://blog.csdn.net/qyhaill/article/details/103043637)

https://blog.csdn.net/qq_41131535/article/details/108463225

[源码分析] (https://zhuanlan.zhihu.com/p/87209990?from_voters_page=true)

* 增加学习率规划
* 设置梯度截断
* 正则化衰减







#### cuDNN模式

设置torch.backends.cudnn.benchmark = True

**Benchmark模式会提升计算速度，但是由于计算中有随机性，每次网络前馈结果略有差异。**

设置torch.backends.cudnn.deterministic = True  来避免这种结果波动



#### torch.manual_seed() 和 torch.cuda.manual_seed() 功能及实例

- 设置固定生成随机数的种子，使得每次运行该 .py 文件时生成的随机数相同

  ```python
  import torch
  
  if torch.cuda.is_available():
      print("gpu cuda is available!")
      torch.cuda.manual_seed(1000)
  else:
      print("cuda is not available! cpu is available!")
      torch.manual_seed(1000)
  
  print(torch.rand(1, 2))   # 生成的值一样
  ```



#### pytorch 多GPU

* optimizer（dataloader，model-ops） 

