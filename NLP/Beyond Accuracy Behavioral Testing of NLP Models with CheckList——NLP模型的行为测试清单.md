### Beyond Accuracy: Behavioral Testing of NLP Models with CheckList——NLP模型的行为测试清单

[paper](https://arxiv.org/pdf/2005.04118.pdf)， [github](https://github1s.com/marcotcr/checklist)， [zhihu](https://zhuanlan.zhihu.com/p/159035275)， ACL 2020 最佳论文-Microsoft Research



#### 背景

基于数据驱动的模型，如果希望部署后有良好的泛化性能，训练和测评数据都应该充分的覆盖场景数据。但场景数据是不可控的，这导致了无法准确测评模型性能的问题。

<img src="C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1619680004332.png" alt="1619680004332" style="zoom:67%;" />

传统的测评方法是在额外公认的benchmark数据集跑统计指标。但这些测试集通常不能充分覆盖场景数据，而且经常和训练数据有一样的分布，这就会高估模型的泛化能力；同时用简单统计指标，难以发现模型失效的具体情况并进行改进。而其他一些评估方法，如1）评价模型对噪声的鲁棒性和对抗攻击的能力；2）提供诊断数据集并交互式的分析错误样本等，大多只关注特定任务，没有全面而综合的提出如何评价模型性能。

本文提出的改进和主要贡献：

* 提出Checklist模型评估理论和附带工具测试NLP模型
* 引入测试类型，将潜在的失效情况分解为特定的类别
* 制作了大量的抽象来帮助用户快速的产生大量测试样例



#### Checklist测试

Checklist测试方法类似软件工程中的黑盒测试，通过检查输入和输出测试模型的能力。形式上是一张表单，每行代表一种待测试的模型能力，每列是对应的测试类型。应用中首先checklist要提供一份模型的“语言能力”的清单，然后将失效情况归类为不同的测试类型

<img src="C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1619596992649.png" alt="1619596992649" style="zoom:67%;" />

1.模型能力：

借鉴软件工程测试系统组件的方法，通过创建测试样例每次测试模型的一种“语言能力”，进而评估模型的性能。

不同的NLP模型都需要处理一些通用的语言问题，比如否定词，名称实体，同称指代，语义角色等，这些语言问题基本从属于每个NLP任务，恰好提供了衡量模型能力的清单。本文归类了NLP模型需要具备的基本语言能力：

* Vocabulary+POS：是否掌握任务相关的词汇及词性
* Taxonomy：是否理解同义词、反义词
* Robustness ：是否能够应对拼写错误和不相关的变化
* Temporal ：理解事件的顺序
* Negation：是否理解否定词
* Coreference：是否理解指代关系
* Semantic Role Labeling：是否理解各种角色
* Logic ：是否能够处理对称性、前后一致性以及连接词

2.三种测试类型：

* MFT 最小功能测试：测试模型具备简单基本的能力
* INV 不变性测试：对输入做一定扰动，测试模型保持输出不变的能力
* DIR 方向期望测试：按特定的方向扰动输入，检查模型是否按期望的方式输出

3.产生大规模的测试样例：

1）从头创建测试数据；2）扰动现有的数据；3）使用模板进行批量生成；比如

* 测试样例：“I didn’t love the food.”

* 模板化："I {NEGATION} {POS_VERB} the {THING}."

另外通过一个msak语言模型，自动生成预选词来扩展模板



#### OCR如何借鉴

Step1：定义出模型的“能力”清单，要结合实际场景，有通用的价值

Setp2：整理出模型失效的情况，看是否能进行归类

Step3：如何快速生成大量测试样例

> 哪些图像问题从属于每个OCR任务？ 1）图像质量：模糊，低分辨率；2）图像干扰：遮挡，噪声扰动，对抗攻击；3）图像尺度；4）文本内容：字体，字形，形近字等



#### 附录1——Checklist样例生成

MFT测试

```python
import checklist
from checklist.editor import Editor
from checklist.perturb import Perturb
from checklist.test_types import MFT, INV, DIR
editor = Editor()

t = editor.template('This is {a:adj} {mask}.',  
                      adj=['good', 'great', 'excellent', 'awesome']) # MLM和模板结合
test1 = MFT(t.data, labels=1, name='Simple positives',
           capability='Vocabulary', description='') # 注意要添加标签
```

INV测试

```python
dataset = ['This was a very nice movie directed by John Smith.',
           'Mary Keen was brilliant.',
          'I hated everything about this.',
          'This movie was very bad.',
          'I really liked this movie.',
          'just bad.',
          'amazing.',
          ]
t = Perturb.perturb(dataset, Perturb.add_typos)
test2 = INV(**t) # 关键字参数，将任意个参数以字典形式传入
```

DIR测试

```python
from checklist.expect import Expect
def add_negative(x):
    phrases = ['Anyway, I thought it was bad.', 'Having said this, I hated it', 'The director should be fired.']
    return ['%s %s' % (x, p) for p in phrases]

t = Perturb.perturb(dataset, add_negative) # 为dataset添加后缀
monotonic_decreasing = Expect.monotonic(label=1, increasing=False, tolerance=0.1) # 这里还是要给原始数据设置标签，希望模型单调下降（应该是标签类对应的概率？）
test3 = DIR(**t, expect=monotonic_decreasing)
```



#### 附录2——Checklist方法测试开放的模型

>![1619686761286](C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1619686761286.png)

情感分析任务的checklist测试

![1619686852732](C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1619686852732.png)

OOP任务的checklist测试

![1619686644432](C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1619686644432.png)

阅读理解任务checklist测试

![1619686947919](C:\Users\viruser.v-desktop\AppData\Roaming\Typora\typora-user-images\1619686947919.png)





