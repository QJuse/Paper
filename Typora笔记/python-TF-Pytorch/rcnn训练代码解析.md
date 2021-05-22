### rcnn训练代码解析

#### 代码逻辑

训练环

```python
1.parse_arg()函数
	训练入口，解析命令行参数。将配置文件的内容放入config字典变量中
    config = edict(yaml.load(f))
2.utils.creat_log_folder(config)
	创建输出文件夹，并将相关信息放入config字典中
3.cudnn配置和“记录器”配置

## 模型配置
4.lib/crnn.get_crnn(config）从配置中创建模型，并将模型部署到设备     ### 
5.配置CTC损失和优化器utils.get_optimizer()   ##
6.加载模型参数，从头训练，ckpt加载
7.设置学习率规划
                    
# 数据集加载
8.lib/get_dataset(condig) #集成Dataset类             ###
9.数据集加载loader， 设置batch,shuffle, 线程等

10.utils.strLabelConverter()转换器                   ###
11.训练环：调用lib/function()进行训练和valid            ###
       torch保存模型，参数信息,epoch信息，最佳精度等
       关闭记录器（同TB)
                           
```



模型代码理解

```python
# lib/models/crnn.py
1.get_crnn（）根据config参数中的模型type来选择模型，加载模型权重。
2.CRNN类继承nn.Module类，只需要实现init和forward两个类即可。
	init函数中nc表示初始的图片纬度，nh表示LSTM隐含的神经元大小。
	保证imgH是16的倍数，16是CNN缩放的倍率。
    
    
```



torch学习

```
1.nn.Sequentials是一个有序容器，依次添加神经网络模块。同keras类似是一种构建网络的方式。
	X.add_module(name, ops)
2.nn.Conv2d(in_channels, out_channels, kernel_size, stride=1, padding=0, dilation=1, groups=1, bias=True)) # 输入，输出，kernel大小，步长，padding,空洞
3.nn.batchnorm2d(num_features, eps=1e-5, affine=True): # 输入特征，affine放射偏移。
4.nn.ReLU(inplace=True) # 设置为True后,输出数据会覆盖输入数据,导致无法求取relu的梯度,一般在模型测试的时候,设置为True是没有问题的,但是在训练过程中,这样做就会导致梯度传递出现问题,所以会导致loss飙升
5.nn.MaxPool2d(kernel_size, stride=1, padding=0, dilation=1, return_indices=False)

# 输入：[B, W, H, C]的特征图，H要等于1，转换为[Seq, B, C]的序列，网络输入的神经元个数则相当于通道数C。

6.nn.LSTM（*args, **kwargs）  
	input_size：x的特征维度（和输入的序列长度无关）
	hidden_size：隐藏层的特征维度（同x的特征纬度，构成单步时间下的网络）
	num_layers：lstm隐层的层数，默认为1
		bias：False则bih=0和bhh=0. 默认为True
		batch_first：True则输入输出的数据格式为 (batch, seq, feature)
    	dropout：除最后一层，每一层的输出都进行dropout，默认为: 0
    bidirectional：True则为双向lstm默认为False
    输入：input(seq_len, batch, input_size), (h0, c0)
    输出：output(seq_len, batch, hidden_size * num_directions), (hn,cn)
7.F.log_softmax(x, dim=2) 选择对某个纬度作为softmax,再做log运算
```

crnn结构

![1591581433130](C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1591581433130.png)



#### 问题

初始权重从哪里加载，选择的是哪个？



模型训练完之后保存的方式和地址， 

```python
# 查看train.py， config.py文件
输出的root目录：cfg.OUTPUT_DIR
     dataset = cfg.DATASET.SATASET  model  = cfg.MODEL.NAME
模型checkpoints输出和tb的log文件，root/dataset/model/time_str
```



部署时怎么重用参数，用的那个模型参数？

* 部署时用的模型版本，训练配置
* 图片的预处理流程（按比例resize到固定大小32，w*W/OW）(减均值，和方差)





### 改进点

* CNN结构，BN的使用
* 图片resize和感受宽度调整
* 图片质量分析, 统计图片尺度分布，字符-质量情况