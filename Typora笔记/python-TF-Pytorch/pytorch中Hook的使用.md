#### pytorch中Hook的使用

三个hook主要可用来控制中间梯度，模型的中间层参数和特征图之类 [参考](https://blog.csdn.net/qq_41375609/article/details/102891831)

##### hook函数

不改变主体（前向、后向传播等）情况下，实现额外的功能，如在backward之后，仍然可以得到特征图和非叶子节点的梯度，即便它们被释放。

![2050515-20200717185127518-1664060414.png](https://img2020.cnblogs.com/blog/2050515/202007/2050515-20200717185127518-1664060414.png)

##### 使用hook可视化所有层的特征图

* 调用上面的register_forward_hook获取网络层的输出







#### 可视化CNN特征图



#### https://my.oschina.net/u/4356413/blog/4520431



https://github.com/TingsongYu/PyTorch_Tutorial 代码使用不熟悉（pytorch）（cv2）

