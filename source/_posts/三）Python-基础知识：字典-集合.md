---
title: 三）Python 基础知识：字典 & 集合
date: 2022-02-22 10:07:51
categories: "Python"
tags:
	- Python
---


## 字典（dict）

字典是一系列_无序_的键值对的组合

```python
def dictFunc():
    dict1 = {'1': 'Java', '2': 'PHP', '3': 'Swift'}
    print(dict1)
    # 新增一个键值对
    dict1['4'] = 'Objctive-c'
    print("新增一个键值对:", dict1)
    # 修改键值对
    dict1['2'] = 'Dart'
    print("修改键值对:", dict1)
    # 通过 key 值，删除对应的元素
    del dict1['2']
    print("通过 key 值，删除对应的元素:", dict1)
    for key in dict1.keys():
        print("key is:", key, "value is:", dict1[key])
    # 删除字典中的所有元素
    dict1.clear()
    print("删除字典中的所有元素:", dict1)
```
### 字典函数
| **方法和函数** | **描述** |
| --- | --- |
| len(dict) | 计算字典元素个数 |
| str(dict) | 输出字典可打印的字符串表示 |
| type(variable) | 返回输入的变量类型，如果变量是字典就返回字典类型 |
| dict.clear() | 删除字典内所有元素 |
| dict.copy() | 返回一个字典的浅复制 |
| dict.values() | 以列表返回字典中的所有值 |
| popitem() | 随机返回并删除字典中的一对键和值 |
| dict.items() | 以列表返回可遍历的(键, 值) 元组数组 |


## 集合（set）

- 与字典的实现非常类似，唯一的区别在于集合里的元素不是键值对，是单一的一个元素
- 字典和集合中的元素的类型都是混合型的，不一定必须要是同一类型元素

```shell
def setFunc():
    set1 = set(['Java', 'OC', 'Swift', 'Swift'])
    print(set1)
    set1.add('Dart')
    print("添加 Dart:", set1)
    set1.remove('Swift')
    print("移除 Swift:", set1)
    set2 = set(['Shell', 'Python', 'Ruby', 'Java'])
    # 交集 & : set1&set2，返回一个新的集合，包括同时在集合 set1 和 set2 中的共同元素。
    set3 = set1 & set2
    print("交集 求两个 set 集合中相同的元素:", set3)
    # 并集 | : set1|set2，返回一个新的集合，包括集合 set1 和 set2 中所有元素。
    set4 = set1 | set2
    print("并集 包括集合 set1 和 set2 中所有元素:", set4)
    # 差集 - : set1-set2，返回一个新的集合,包括在集合 set1 中但不在集合 set2 中的元素
    set5 = set1 - set2
    print("差集 包括在集合 set1 中但不在集合 set2 中的元素:", set5)
    # 补集 ^ : set1^set2，返回一个新的集合，包括集合 set1 和 set2 的非共同元素。
    set6 = set1 - set2
    print("差集 集合 set1 和 set2 的非共同元素:", set6)
    for value in set6:
        print("value 的值:", value)
```

## 参考

- [菜鸟学习 - 字典](https://www.runoob.com/python/python-dictionary.html)
- [菜鸟学习 - set](https://www.runoob.com/python/python-func-set.html)
- [Python基础之dict和set](https://zhuanlan.zhihu.com/p/66804819)
- [demo](https://github.com/zeqinjie/python_demo)
