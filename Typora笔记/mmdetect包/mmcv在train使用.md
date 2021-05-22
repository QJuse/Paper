### mmcv在train使用

* train接口，核心参数model，dataset, cfg

  * 模型包内定义data_build
  * 模型分布式设置，mmcv.parallel中的MMDataParallel, MMDistributedDataParallel

  

  * 构建训练runner： build_optimizer，EpochBasedRunner（model，optim）
  * 构建runner的hooks：学习率，优化器设置等register_training_hooks（）
    * 评估过程：数据重新导入，model不变，注册eval_hook
    * 自定义的hook注册
  * runner重载入模型
  * runner运行run（data_loaders, **cfg.workflow**, cfg.total_epochs）



* test接口不使用runner过程，get_dist_info()，load_checkpoint()



* mmcv.utils提供Registry, build_from_cfg



#### mmcv注册机制

注册机制可以理解为将类映射为字符串，通过字符串可实例化对应的类。[被注册的类通常包含相似的APIs，但实现的算法或支持的数据集不同]。（典型示例：通过配置文件产生相应的类）

* 通过Registry管理模块的步骤

  * 创建注册器，创建build方法，用注册器管理相应模块

    ```python
    converter_cls = CONVERTERS.get(converter_type) # 字符串转成类，在用cfg参数初始化
    # 从配置参数cfg中获得pipe的字符串
    ```

    ```python
    from .builder import CONVERTERS
    # use the registry to manage the module
    @CONVERTERS.register_module() # 用实例方法作装饰器，将一个类注册到“类名”中
    class 
    ```

    

