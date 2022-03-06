---
title: 七）Python 基础知识：内建函数 & 闭包
date: 2022-02-26 21:44:07
categories: "Python"
tags:
	- Python
---

## 内建函数

### filter

> filter(function,  iterable) 函数用于过滤序列，过滤掉不符合条件的元素，返回由符合条件元素组成的新列表。

- function 判断函数
- iterable 可迭代对象

```python
def filter_func():
    oldlist =  [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    newlist1 = list(filter(isEvenNum, oldlist))
    print("过滤出奇数：", newlist1)
    newlist2 = list(filter(lambda x: x % 2 == 0, oldlist))
    print("过滤出偶数：", newlist2)
    
# 获取基数
def isEvenNum(n):
    return n % 2 == 1
```


### map

> map(function, iterable, ...) 会根据提供的函数对指定序列做映射

- function 函数
- iterable  一个或多个序列

```python
def map_func():
    oldlist = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    newlist1 = list(map(lambda x: x * 2, oldlist))
    print("旧数组里的数 * 2 = ", newlist1)
    newlist2 = list(map(lambda x, y: x + 2, oldlist, newlist1))
    print("oldlist 与 newlist2 相加等于", newlist2)
```


### reduce
> reduce(function, iterable[, initializer])  函数会对参数序列中元素进行累积。
> 注意要引入：from  functools import  reduce

- function 函数，有两个参数
- iterable 可迭代对象
- initializer 可选，初始参数

```python
def reduce_func():
    res = reduce(lambda x, y: x + y, [2, 3, 4], 1)
    print("累加等于：", res)
```


### zip
> zip([iterable, ...]) 函数用于将可迭代的对象作为参数，将对象中对应的元素打包成一个个元组，然后返回由这些元组组成的列表

- iterabl  一个或多个迭代器

```python
# zip 合并函数
def zip_func():
    for i in zip((1, 2, 3), (4, 5, 6), (7, 8, 9)):
        print(i)
    dict1 = {"a": "aa", "b": "bb", "c": "cc"}
    dict2 = dict(zip(dict1.values(), dict1.keys()))
    print("key 与 value 对调：", dict2)
```


## 闭包
> 在函数中可以（嵌套）定义另一个函数时，如果内部的函数引用了外部的函数的变量，则可称为闭包

```python
def add_func1(num1, num2):
    return num1 + num2

# 闭包函数
def add_func2(num1):
    def add(num2):
        return num1 + num2
    return add

# 计算器, 这里用列表是
def counter_func(num1 = 0):
    cnt = [num1]
    def add_one():
        cnt[0] += 1
        return cnt[0]
    return add_one


def closure_func():
    re1 = add_func1(1, 2)
    print("re1 类型", type(re1), re1)
    re2 = add_func2(1)
    print("re2 类型", type(re2), re2(2))
    num1 = counter_func(5)
    print(num1())
    print(num1())
    print(num1())
```

## 参考

- [Python 函数的参数传递*args和**kwargs](https://blog.csdn.net/xushibi4580/article/details/90173803)




