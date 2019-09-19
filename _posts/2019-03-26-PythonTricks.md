---
layout: post
title: Python Tricks
date: 2019-03-26
catalog: true
header-img: img/programming.jpg
tags:
    - python
---
## 模块、包、导入
#### module
一个`xx.py`文件就是一个模块，里面封装好了一系列方法和变量，可以在其他的模块中通过import语句来导入，这大大的提高了代码的复用性，而不是在不同的模块中copy and paste。

#### 包
模块的集合，外在形式表现为文件夹，包可以包含集合和子包。需要`__init__.py`这个初始化模块来标识这个文件夹是一个包。

#### import
import是用来导入模块和包的关键字。
有三种方式来导入
```python
import foo
from foo import bar
from foo import *
```
第二种是推荐的用法。第三种应该尽量避免，这会让代码很难阅读，这样无法辨识代码中突然冒出来的方法和变量是从哪来的。
- import 一个模块的时候，会执行这个模块的顶层代码，然后该模块就会被编译成对象，可以访问它的方法和变量。如果是导入一个包，或者从包中包入一个模块，例如`import foo.bar`, 先会执行foo包的`__init__.py`, 再执行`bar.py`。
- 当然也可以导入方法和变量。
- 在`__init__.py`中又个特殊变量`__all__`, 指向一个字符串列表，用来控制`from foo import *`能导入什么。

#### relative import
参考stack overflow这个问题的高票[回答](https://stackoverflow.com/questions/14132789/relative-imports-for-the-billionth-time?rq=1)，太精彩了。

```
package/
    __init__.py
    subpackage1/
        __init__.py
        moduleX.py
        moduleY.py
    subpackage2/
        __init__.py
        moduleZ.py
    moduleA.py
```
在moduleX中导入moduleZ，`from ..subpackage2 import moduleZ` or `from .. import subpackage2.moduleZ` 等同于 `from package.subpackage2 import moduleZ`。当然，`from .. import subpackage2.moduleZ`意味着使用moduleZ是的命名空间是`subpackage2.moduleZ`.
这里的`.`用来表示包的层级， 有几个`.`意味着要向上几层。跟path中的`../..`中的点是有区别的。`from ..subpackage2 import moduleZ`能正常运行，取决于解释器是否知道moduleX所在的包层级结构，两个`.`意味着要向上找两层，即解释器是否知道`package.subpackage1.moduleX`这一事实。解释器通过模块的`__package__ and __name__`来解析导入路径，当`__package__`为None是，仅由`__name__`决定。


1. moduleX不是主程序入口，在系统的其他路径下的脚本中，`from package.subpackage1 import moduleX`。这时候`moduleX.__name__`是`package.subpackage1。moduleX`,这样解释器就知道了moduleX是package子包subpackage1的模块。
2. moduleX是主程序入口，通过`python -m package.subpackage1.moduleX`的方式运行moduleX，在命令行显示告诉解释器moduleX所在的包层级结构。
这两种方式的前提是解释器知道package在哪，也就是package所在的目录在sys.path中。

## 实用的 `*` & `**` 
在官方文档中，这两个符号的作用是展开参数列表( Unpacking Argument Lists).
#### 函数形参与实参
形参是指定义函数时设置的参数，实参时调用函数时实际传递的参数
大部分自定义函数都是根据实际需求设置的，定义了几个，就传几个。在某些场景下，可以允许用户传递任意数量的参数。
两种用例：接受任意数量实参`*args`， 接受任意数量带关键字实参`**args`。
```python
def func(arg1, arg2, *args):
    # args是一个实参列表
    pass
def func(arg1, arg2, **args):
    # args是一个字典
    for key, value in args.items():
        pass
```

#### 解构
除了在传参时的应用，在操作数据时，可以使用这两个符号来解构（展开）变量。
```python
>>> a = [1, 2]
>>> b = [3, *a]
>>> b
[3, 1, 2]
>>> c = {'foo': 1}
>>> d = {'bar': 2, **c}
>>> d
{'bar': 2, 'foo': 1}
```

## 标准库 logging 笔记
#### basic points
1. 使用日志有两个目的：
    - 将程序运行情况，尤其是error输出到文件中或者标准输出（stdout，stderr），让我们能够更好的记录、监控程序运行状况。
    - 商业分析用途，例如计算产品的UV、PV数据，就需要记录用户的访问、交互日志。
2. 除了要在命令行中打印帮助文档，其他情况下都应选择logging输出内容
3. 在logging模块中，常接触的是这三个对象:
    - Logger, 通过logger完成日志命令，logger.error("massage")
    - Handler, 设置日志的输出形式，是保存到文件中（很多种选择），还是标准输出流
    - Formatter， 格式化设置日志的内容
4. 日志有5个等级 
    ![levels.jpg](http://blog-data.oss-cn-beijing.aliyuncs.com/img/levels.jpg)
    在给logger设置了处理日志的等级后，例如WARNING，那么低于该等级（比如info）的日志，都不会处理。
    其作用在层级式的loggers中更为明显，parent logger可以设置level过滤掉child logger打过来的日志，不知道这是不是官方推荐的用法。
5. logger有层级，就像包，子包，模块，package.foo.bar. 再定义logger名字的时候，就是通过`.`来表示层级。而已root关键字命名的logger表明是所有其他logger的祖先logger。层级意味着向上传递，默认设置下，每个logger都会向祖先logger传递日志，如果祖先logger存在的话。可以通过设置propagate为False来更改这一默认行为。
    
#### simple config 硬编码到代码中
最简单的用例如下，适合于小型项目、简单的日志输出任务中。
```python
import logging

logging.basicConfig(
     filename='app.log',
     level=logging.WARNING,
     format='%(levelname)s:%(asctime)s:%(message)s')
logging.error("it's a error message")
logging.info()

```

#### High level config
当项目有很多包和模块时，有必要采取复杂一些的配置来处理不同需求、级别的日志打印需求。这时候会把配置写到文件中，这样到代码中去修改了。
两种方式可以处理文件配置：
1. logging.config.fileConfig('logconfig.ini'), logconfig.ini文件内容示例。

    ```python
    # 可分为两个部分，先定义名字，在添加模块来设置这个名字的配置
    [loggers]
    keys=root

    [handlers]
    keys=defaultHandler

    [formatters]
    keys=defaultFormatter

    [logger_root]
    level=INFO
    handlers=defaultHandler
    qualname=root

    [handler_defaultHandler]
    class=FileHandler
    formatter=defaultFormatter
    args=('app.log', 'a')

    [formatter_defaultFormatter]
    format=%(levelname)s:%(name)s:%(message)s
    ```
    这种方法不推荐，logging模块最新的一些配置项不支持再这种格式的文件中配置。

2. 最新的官方tutorial推荐使用logging.config.dictConfig(), 接受一个字典。所以可以把配置写到文件中，读取转换成字典类型，再传递给这个方法。配置可以存储成json，yaml，甚至是pickle，只要能转换成字典就行。

    下面是一个yaml格式的配置文件，推荐这种格式，很方便阅读、编辑。
    ```yaml
    version: 1
    formatters:
    simple:
        format: '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
    handlers:
    console:
        class: logging.StreamHandler
        level: DEBUG
        formatter: simple
        stream: ext://sys.stdout
    file:
        class: logging.handlers.RotatingFileHandler
        level: INFO
        formatter: simple
        filename: info.log
        maxBytes: 10485760
        backupCount: 20
        encoding: utf8
    loggers:
    simpleExample:
        level: DEBUG
        handlers: [console]
        propagate: no
    root:
    level: DEBUG
    handlers: [console]
    ```

$a^2 + b^2 = c^2$

$h_\theta(x) = \Large\frac{1}{1 + \mathcal{e}^{(-\theta^\top x)}}$
