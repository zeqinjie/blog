---
title: 十）Python-基础知识：模块定义


date: 2022-03-09 22:38:58
categories: "Python"
tags:
	- Python
---

## 模块
定义：在 Python 中，一个 .py 文件就称之为一个模块（Module）

### import
一般的导入模块的方式: `import module1[, module2[,... moduleN]`

### import ... as
导入模块使用别名的方式:  `import name1 as name2`

### from … import 
导入模块的部分方法: `from modname import name1[, name2[, ... nameN]]`

### from … import *  
导入模块的所有内容: `from modname import *`

```python
# 一般的导入模块的方式
import set
# 导入模块使用别名 as
import iter_and_yield as my_yield
# 导入模块的部分方法
from filter_and_map import filter_func, map_func
# 导入模块的所有内容
from dict import *

if __name__ == "__main__":
    # 一般使用情况
    set.set_func()

    # 使用别名 as
    my_yield.yield_func()

    # 使用 from 方式
    filter_func()
    map_func()
```

## __name__

一个模块被另一个程序第一次引入时，其主程序将运行。如果我们想在模块被引入时，模块中的某一程序块不执行，我们可以用__name__属性来使该程序块仅在该模块自身运行时执行


```python
if __name__ == '__main__':
   print('程序自身在运行')
else:
   print('我来自另一模块')
```

## 包使用
包的出现，是解决不同目录下的模块引用:  import 包.模块 、 from 包 import 模块

![](/images/2022/十）Python 基础知识：模块定义/1.png)

```python
import com.Learn.module.nameattributes.lname
```

## 问题

自定义模块与Python自带的模块重名
> 模块的加载顺序，如果自定义和内置模块重名，当import 一个模块时，会依次的在以上路径顺序中查找，找到了就不再往后找了，找不到就导入异常，只搜索指定目录，不递归搜索


## 参考

- [菜鸟学习](https://www.runoob.com/python3/python3-module.html)
- [官方文档 - packages](https://docs.python.org/3/tutorial/modules.html#packages)
- [demo](https://github.com/zeqinjie/python_demo)

