### ASTER: An Attentional Scene Text Recognizer with Flexible Rectification

xiang bai 2018 华中科技

#### 摘要

提出ASTER模型解决STR中文本扭曲和不规则排布的问题。主要创新：1）使用专门的校正机制解决不规则文本识别（不需要额外标注）；2）使用Seq2Seq+Attention方法识别文本并扩展了双向解码器；3）提出利用ASTER在文本校正和识别方面的能力来增强文本检测器的方法

#### 介绍

<img src="C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1615016367568.png" alt="1615016367568" style="zoom: 80%;" />

ASTER模型由校正网络和识别网络两部分组成：

* 校正网络通过TPS方法参数化变换器，将不规则文本修正成水平文本，推理时需要先从图片中预测TPS参数。因为基于STN框架，校正模块可以通过识别网络的误差反传来训练，无需人工标注。
* 识别网络采用Seq2Seq+Attention框架，将字符检测，识别和语言模型压缩成一个模型。并提出**双向的解码器**。实验证明在规则和不规则文本上效果都不错

#### 相关工作

##### 文本识别

