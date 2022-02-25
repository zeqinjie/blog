---
title: 六）Python 基础知识：迭代器 & 生成器
date: 2022-02-25 10:03:15
categories: "Python"
tags:
	- Python
---


## 迭代器

什么叫做迭代? 通过 for 循环来遍历这个 list 或 tuple ，这种遍历就是迭代

```python
# for 循环输出每个值
def forFunc():
    class_names = ['PHP', 'Swift', 'Java', 'Python', 'Objective-C']
    for name in class_names:
	    print("class name:", name)
```


-   迭代器，迭代器是一个可以记住遍历的位置的对象
-   迭代器对象从集合的第一个元素开始访问，直到所有的元素被访问完结束。迭代器只能往前不会后退
-   迭代器有两个基本的方法：` iter()  `和 `next()`
-   注意：如果迭代器已执行完，就不会再输出任何值


```python
# 迭代器
def iterFunc():
    class_names1 = ['PHP', 'Swift', 'Java']
    iter1 = iter(class_names1)
    print(next(iter1))
    print(next(iter1))
    print(next(iter1))
    try:
        print(next(iter1))
    except Exception as e:
        print("捕获异常", e)
    # 也可以通过该 for 直接遍历，不过注意如果已执行完 next() 那下面就不会输出任何值了
    for x in iter1:
        print(x, end=" ")

    class_names2 = ['Python', 'Shell', 'Ruby']
    iter2 = iter(class_names2)
    while True:
        try:
            print(next(iter2))
        except StopIteration:
            print("异常抛出：exit")
            sys.exit()
```


## 生成器

-   在 Python 中，这种一边循环一边计算的机制，称为生成器：generator
-   使用了 `yield` 的函数被称为生成器（generator）
-   在调用生成器运行的过程中，每次遇到 yield 时函数会暂停并保存当前所有的运行信息，返回 yield 的值。并在下一次执行 next()方法时从当前位置继续运行
-   `yield`的主要用途是需要一个数据时，才产生一个，而不是把数据线一次性存入内存；相对于把数据提前定义成列表来使用，要极为节省系统资源

```python
# 生成器
def yieldFunc():
    print("------实现支持步长值为浮点的 range：------")
    for value in frange(10, 20, 0.5):
        print(value, end=',')

    print("\n------杨辉三角形生成器：------")
    n = 0
    for t in triangles(10):
        print(t)
        n = n + 1
        if n == 10:
            break

    print("\n------斐波那契生成器：------")
    # f 是一个迭代器，由生成器返回生成
    f = fibonacci(10)
    while True:
        try:
            print(next(f), end=",")
        except StopIteration:
            sys.exit()


# 实现支持步长值为浮点的 range
def frange(start,stop,step):
    x = start
    while x < stop:
        yield x
        x +=step

# 杨辉三角形
def triangles(n):
    L = [1]
    while True:
        yield L
        L.append(0)
        L = [L[i - 1] + L[i] for i in range (len(L))]

# 生成器函数 - 斐波那契
def fibonacci(n):
    a, b, counter = 0, 1, 0
    while True:
        if (counter > n):
            return
        yield a
        a, b = b, a + b
        counter += 1
```


## 参考

-   [80%人都没搞懂python迭代器和生成器的区别，本文详解](https://www.cnblogs.com/chengxuyuanaa/p/13065794.html?ivk_sa=1024320u)
-   [Python3 迭代器与生成器](https://www.runoob.com/python3/python3-iterator-generator.html)
