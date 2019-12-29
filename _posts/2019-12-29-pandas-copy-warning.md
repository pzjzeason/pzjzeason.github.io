---
layout: post
title: Pandas SettingWithCopyWarning
date: 2019-11-14
catalog: true
header-img: img/programming.jpg
tags:
    - python
    - pandas
---

利用pandas进行数据处理时，有时会碰到SettingWithCopyWarning这个警告，虽然不是错误但仍需非常小心，因为这样可能导致你没有预料到的结果。
例如意外修改了原始数据，或者赋值没起作用。警告的内容主体是：
```python
xxx/pandas/core/frame.py:3391: SettingWithCopyWarning: 
A value is trying to be set on a copy of a slice from a DataFrame.
Try using .loc[row_indexer,col_indexer] = value instead

See the caveats in the documentation: http://pandas.pydata.org/pandas-docs/stable/indexing.html#indexing-view-versus-copy
  self[k1] = value[k2]
```
意思是当前的赋值操作正在dataframe的一个切片上进行，去这个链接看看关于dataframe视图和拷贝的注意事项。

首先熟悉两个概念：视图vs拷贝，chain assignment
### 1 视图vs拷贝

视图是原始dataframe的一部分，对视图的修改会反映到原始数据上。拷贝是从原始dataframe复制出来的一部分，对拷贝的修改不会影响原始dataframe。
![-20191229-09-16-57.png](https://blog-data.oss-cn-beijing.aliyuncs.com/img/-20191229-09-16-57.png){:width="300px"}
![-20191228-21-11-39.png](https://blog-data.oss-cn-beijing.aliyuncs.com/img/-20191228-21-11-39.png){:width="300px"}

### 2 chain indexing
chain是紧挨着的indexing操作，例如`data[1:5][1:3]`，除了方括号，`.col`, `.loc`, `.iloc`等从dataframe获得部分数据的操作都算indexing。当chain indexing碰到赋值操作时，就会引发这个警告。主要分为两种情况：显式的chain assignment和hidden chain。下面是以[ebay xbox拍卖数据](http://www.modelingonlineauctions.com/datasets/Xbox%203-day%20auctions.csv?attredirects=0&d=1)来复现和消除这个警告的例子。

### 3 chain assignment
读入数据。
```python
import pandas as pd
data = pd.read_csv('xbox-3-day-auctions.csv')
data.head()
```
![-20191229-09-00-09.png](https://blog-data.oss-cn-beijing.aliyuncs.com/img/-20191229-09-00-09.png){:width="300px"}

如果我们想要将bidder是parakeet2004的记录的bidderrate修改为100，下面的代码就会产生这个警告。
![-20191229-09-03-14.png](https://blog-data.oss-cn-beijing.aliyuncs.com/img/-20191229-09-03-14.png){:width="600px"}

修改并没有生效，因为赋值发生在copy上。解决办法是使用只使用一次loc来定位行列索引再赋值。
![-20191229-09-03-53.png](https://blog-data.oss-cn-beijing.aliyuncs.com/img/-20191229-09-03-53.png){:width="400px"}

### 4 hidden chain
上面这种情况是很容易发现并避免的，而hidden chain是比较隐蔽的，因为第一次indexing取subset操作与后续的indexing 赋值操作中间可能隔了很多行代码。

我们经常会从原始数据中取出一些子列或者满足条件的行，例如下面：
![-20191229-09-07-02.png](https://blog-data.oss-cn-beijing.aliyuncs.com/img/-20191229-09-07-02.png){:width="400px"}

然后我们会对这个新的dataframe做很多操作，再之后可能会有修改的操作：
![-20191229-09-13-20.png](https://blog-data.oss-cn-beijing.aliyuncs.com/img/-20191229-09-13-20.png){:width="600px"}

修改成功，但是还是有警告。因为winner是通过indexing在data上取得的，上述代码赋值时又使用了一次indexing，触发了pandas的警告。winners的is_copy属性会记录这个dataframe是否来自某个dataframe，不过这个属性将要被废弃，修改这个属性从而避免警告是不推荐的。解决方法是使用深拷贝。
![-20191229-09-13-58.png](https://blog-data.oss-cn-beijing.aliyuncs.com/img/-20191229-09-13-58.png){:width="300px"}


参考文章：[settingwithcopywarning](https://www.dataquest.io/blog/settingwithcopywarning/)。里面有更多的细节，例如在dataframe的dtypes都是一样时，indexing得到的一定是copy。不一样时，就不一定了。