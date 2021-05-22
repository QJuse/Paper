m = Mask(net)

m.init_length() ?

* model_length= {} 每一层参数的总数

* model_size = {} 每一层参数的shape长度

m.model = net



m.**init_mask**(rate_norm_per_layer, dist)   ：确定要压缩的层和压缩率

首先设置所有层的压缩率为1  [不同模型结构压缩情况不一样！！]

* rate_norm

* rate_dist

* compress_rate = {}  # 通过参数指定卷积压缩率

* distance_rate = {}     # 通过计算 1- v / out_c

* mask_index = []  压缩层idx

  init_rate:

  * vgg:逐层压缩
  * resnet:  strip不压缩，只压缩卷积

  get_filter_codebook(参数，压缩率，参数长度)

  ​	确定过滤的filter数量

  ​    计算每个filter[in，k，k]的L2正则化

    排序正则值，得到小值得index

  ​    code_book 中对应的kernel值置0     [length]

  * mat = {} : 对应层的， 张量形式

  get_filter_similar(参数，压缩率，dist，模型长度)

  * similar_mask = {} 

**do_mask**

**do_similar_mask**



net = m.model

net 传给eval测评精度

再训练 每个epoch再剪枝，测评在保存最佳



if_zero()







1.FPGM初步调通

2.确定每一层的裁减敏感度

* 特定结构ResNet的裁减方法
* 每一层裁减的精度损失和重训练恢复弹性

3.重新训练的方式

* 一次裁减多层，每一epoch训练
* 逐层裁减，再训练，在固定参数再裁减-再训练



能做成一个框架进行跑

* 输入：1)测评数据；2）裁减规划；3）重训练数据；4）不同的裁减模式；