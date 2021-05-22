### python库

#### [pydantic](https://pydantic-docs.helpmanual.io/)

pydantic库是一种常用的用于数据接口schema定义与检查的库。通过pydantic库，我们可以更为规范地定义和使用数据接口，这对于大型项目的开发将会更为友好。相比valideer库、marshmallow库、trafaret库以及cerberus库，pydantic库的执行效率会更加优秀一些。仅针对pydantic库来介绍一下如何规范定义标准schema并使用

* [typing库](https://docs.python.org/zh-cn/3/library/typing.html)中的Optional方法

  类型提示支持

* enum库Enum型数据类型

#### [pathlib](https://docs.python.org/zh-cn/3/library/pathlib.html)

该模块提供表示文件系统路径的类，其语义适用于不同的操作系统。

* Path

#### [logging](https://docs.python.org/zh-cn/3/library/logging.html)

这个模块为应用与库实现了灵活的事件日志系统的函数与类

#### [urllib](https://www.liaoxuefeng.com/wiki/1016959663602400/1019223241745024)

urllib提供了一系列用于操作URL的功能。

#### [collections](https://www.liaoxuefeng.com/wiki/897692888725344/973805065315456)

collections是Python内建的一个集合模块，提供了许多有用的集合类

#### [re](https://docs.python.org/zh-cn/3/library/re.html)

这个模块提供了与 Perl 语言类似的正则表达式匹配操作

#### [prettytable](https://blog.csdn.net/u010359398/article/details/82766474)

PrettyTable 是python中的一个第三方库，可用来生成美观的ASCII格式的表格

#### [terminaltables](Easily draw tables in terminal/console applications from a list of lists of strings. Supports multi-line rows)

Easily draw tables in terminal/console applications from a list of lists of strings. Supports multi-line rows



#### [codecs](https://www.cnblogs.com/666666pingzi/p/11462722.html)

codecs专门用作编码转换，当我们要做编码转换的时候可以借助codecs很简单的进行编码转换

#### [warnings](https://blog.csdn.net/chensilly8888/article/details/80195239)

和exception异常要求用户立刻进行处理不同，warning通常用于提示用户一些错误或者过时的用法

#### [shutil](https://www.jb51.net/article/145522.htm)

shutil模块提供了许多关于文件和文件集合的高级操作，特别提供了支持文件复制和删除的功能

#### [dill](https://www.cnpython.com/pypi/dill)

`dill`扩展python的`pickle`模块以进行序列化和反序列化 [python对象](https://www.cnpython.com/qa/50125)的大多数内置python类型。串行化 是将对象转换为字节流的过程，反之亦然 其中之一是将字节流转换回python对象层次结构上

#### [itertools](https://www.liaoxuefeng.com/wiki/1016959663602400/1017783145987360)

Python的内建模块`itertools`提供了非常有用的用于操作迭代对象的函数

#### unicodedata

此模块提供对Unicode字符数据库的访问，该字符数据库为所有Unicode字符定义字符属性

#### [glob](https://www.cnblogs.com/dxnui119/p/13804697.html)

glob模块是一个文件操作相关模块，用它可以查找符合要求的文件，支持通配符操作

#### [bisect](https://blog.csdn.net/qq_34914551/article/details/100062973)

bisect是python内置模块，用于有序序列的插入和查找

#### [pkgutil](https://cloud.tencent.com/developer/section/1368027)

该模块为导入系统提供实用程序，特别是软件包支持

#### [msgpack](https://blog.csdn.net/angus_monroe/article/details/79986759)

msgpack用起来像json，但是却比json快，并且序列化以后的数据长度更小，言外之意，使用msgpack不仅序列化和反序列化的速度快，数据传输量也比json格式小，msgpack同样支持多种语言

#### [traceback](https://docs.python.org/zh-cn/3.7/library/traceback.html?highlight=traceback#module-traceback)

该模块提供了一个标准接口来提取、格式化和打印 Python 程序的堆栈跟踪结果 。打印或检索堆栈回溯

#### [dataclasses](https://blog.csdn.net/sc_lilei/article/details/85264831)

这个dataclasses是当做装饰器来用，作用是在我们定义数据class对象时减少我们的代码量

#### [six](https://www.jb51.net/article/175911.htm)

Six 就是来解决这个烦恼的，这是一个专门用来兼容 Python 2 和 Python 3 的模块

#### [fastapi](https://www.oschina.net/p/fastapi?hmsr=aladdin1e1)

FastAPI 是一个高性能 Web 框架，用于构建 API

#### [multiprocessing](https://docs.python.org/zh-cn/3.6/library/multiprocessing.html)

[`multiprocessing`](https://docs.python.org/zh-cn/3.6/library/multiprocessing.html#module-multiprocessing) 是一个用与 [`threading`](https://docs.python.org/zh-cn/3.6/library/threading.html#module-threading) 模块相似API的支持产生进程的包。 [`multiprocessing`](https://docs.python.org/zh-cn/3.6/library/multiprocessing.html#module-multiprocessing) 包同时提供本地和远程并发，使用子进程代替线程，有效避免 [Global Interpreter Lock](https://docs.python.org/zh-cn/3.6/glossary.html#term-global-interpreter-lock) 带来的影响。因此， [`multiprocessing`](https://docs.python.org/zh-cn/3.6/library/multiprocessing.html#module-multiprocessing) 模块允许程序员充分利用机器上的多个核心

#### [queue](https://docs.python.org/zh-cn/3.7/library/queue.html)

[`queue`](https://docs.python.org/zh-cn/3.7/library/queue.html#module-queue) 模块实现多生产者，多消费者队列。当信息必须安全的在多线程之间交换时，它在线程编程中是特别有用的。

#### [jinja2](https://www.cnblogs.com/dachenzi/p/8242713.html)

jinja2是Flask作者开发的一个模板系统，起初是仿django模板的一个模板引擎，为Flask提供模板支持，由于其灵活，快速和安全等优点被广泛使用。要了解jinja2，那么需要先理解模板的概念。模板在Python的web开发中广泛使用，它能够有效的将业务逻辑和页面逻辑分开，使代码可读性增强、并且更加容易理解和维护

#### [click](https://blog.csdn.net/u010339879/article/details/80542099)

click[模块](https://click.palletsprojects.com/en/6.x/) 可以 轻松将一个函数变成一个命令行工具

#### [setuptools](https://www.jianshu.com/p/ea9973091fdf)

setuptools是Python distutils增强版的集合，它可以帮助我们更简单的创建和分发Python包，尤其是拥有依赖关系的

#### munch

一个点可访问的字典

#### anyconfig





### 常用库

* os
* sys
* time
* yaml
* json
* random
* math
* numpy
* cv2
* PIL
* sklearn
* torch
* tensorflow
* tensorboardX
* copy
* jieba


