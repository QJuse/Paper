## 无监督对比学习

### Unsupervised Feature Learning via Non-Parametric Instance Discrimination

#### 论文理论

##### 研究问题和思路

* 如何捕捉图片之间的相似度来学习一个好的特征表示，从而代替标签仅通过特征表示来区分图片
* 采用**度量学习**建模该问题，提取每个图片实例的特征，存储在”**记忆单元**“中，通过非参数的方法计算实例特征的相似度，从而判断实例之间的距离；推理时采用线形分类器或KNN评估无监督模型的效果

##### 模型方法

![1611749588465](C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1611749588465.png)

在无监督的情况下，学到一个特征表示函数v=f(x)，将所有样本映射到特征空间，在特征空间中定义一个度量标准，进而计算样本实例x和y之间的距离。

**非参数的Softmax分类器（NPSC）**

P(i|x )表示特征v属于某个实例i的概率，左图是有参数的典型Softmax分类，右图是修改后的NPSC，通过L2正则将特征v的范数限定为1，T是控制分布密度的超参数。

<center class="half">
<img src="C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1611749933481.png" width="300"/>
<img src="C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1611749946374.png" width="300"/>
</center>

学习目标：最大化所有样本的联合分布（最小化负对数似然函数）

![1611750572406](C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1611750572406.png)

**特征向量的存储和更新**

通过一个Memory bank存储所有特征（随机向量初始化），每个迭代步，更新优化输入图片的特征和网络参数，再用优化后的图片特征去刷新Memory bank(mini-batch训练时有特征滞后的问题，后续讨论)

**Noise Contrastive Estimation**（NCE） 

大样本情况下，降低相似度计算的方法有分层Softmax，NCE和负采样，本文用NCE近似计算NPS，将多分类的问题转为数据样本和噪声样本的两分类问题。用均匀分布建模噪声分布，假定噪声样本出现频率是数据样本的m倍。

* 计算loss时需要用到全样本采用NCE解决 [9]
* Noise-contrastive estimation: A new estimation principle for unnormalized statistical models
* 计算单个样本相似度似要用到全样本特征，采用蒙特卡罗方法 [25] 
* Learning word embeddings efficiently with noise-contrastive estimation

![1611752444635](C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1611752444635.png)

![1611752497397](C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1611752497397.png)

##### Proximal Regularization近端正则化

和分类有多样本不同，训练时每个实例只有自身一个样本，学习过程在随机采样中波动很大。引入PR方法[29]在动态训练中平滑波动。损失： Proximal algorithms

![1611753495131](C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1611753495131.png)

最终的损失：

![1611753678917](C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1611753678917.png)

**KNN分类测试**

计算图片x的特征，用余弦相似度计算同Memery bank中特征的距离，选择前200个，使用权重投票的方式确定类别，怎么算的具体？

![1611753968902](C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1611753968902.png)

#### 实验结果  http://github.com/zhirongw/lemniscate.pytorch.

测试NPS的效果和KNN评价，以及NCE中m的影响

![1611754109506](C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1611754109506.png)

评价特征泛化能力

![1611754524953](C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1611754524953.png)

embedding特征大小

![1611754635308](C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1611754635308.png)

训练数据大小

![1611754688373](C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1611754688373.png)

##### 半监督学习

学习的特征泛化到其他任务，是否能提供好的迁移知识。在大的无监督上训练，在用小的的标注数据fine-tune。

![1611754997200](C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1611754997200.png)

![1611755012465](C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1611755012465.png)