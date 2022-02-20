---
title: 二）Python 基础知识：数据类型 & 序列
date: 2022-02-20 23:05:59
categories: "Python"
tags:
	- Python
---

##  数据类型
- 整数、浮点数、字符串、布尔值
- 类型判断: type(var)
- 类型转换： int()、float()、str()、bool()


### 整数
![](/images/2022/二）Python-基础知识：数据类型-序列/1.png)

```python
# 整数
def num():
    int1 = 1 + 1
    int2 = 1 * 1
    print("整数：", int1, "和", int2)
```


### 浮点数
```python
# 浮点
def float():
    float1 = 0.1 * 3
    # 取整除的返回整数的部分
    float2 = 11 // 3
    print("整数：", float1, "和", float2)
```


### 布尔值

- True 、 False
- 布尔值可以用 and、or 和 not  运算



```python
# 布尔值
def bool():
    isRun = True
    isFly = False
    if isRun:
        print("它会跑")
    if isRun and isFly:
        print("它既会跑也会飞")
    if not isFly:
        print("它不会飞")
```
### 
### 空值
```python
def null():
    nullValue = None
```


### 类型转换
> 类型转换  int()、float()、str()、bool()

| **方法** | **说明** |
| --- | --- |
| int(x [,base ]) | 将x转换为一个整数 |
| float(x ) | 将x转换到一个浮点数 |
| complex(real [,imag ]) | 创建一个复数 |
| str(x ) | 将对象 x 转换为字符串 |
| repr(x ) | 将对象 x 转换为表达式字符串 |
| eval(str ) | 用来计算在字符串中的有效 Python 表达式,并返回一个对象 |
| tuple(s ) | 将序列 s 转换为一个元组 |
| list(s ) | 将序列 s 转换为一个列表 |
| chr(x ) | 将一个整数转换为一个字符 |
| unichr(x ) | 将一个整数转换为 Unicode 字符 |
| ord(x ) | 将一个字符转换为它的整数值 |
| hex(x ) | 将一个整数转换为一个十六进制字符串 |
| oct(x ) | 将一个整数转换为一个八进制字符串 |

```python
# 案例2
def baseVaule():
    num = 2
    moneyString = "100"
    money = int(moneyString)
    txt = "hello word"
    isError = True
    print("num:", num, "money:", money, "txt:", txt)
    print("isError type:", type(isError))
    print("money type:", type(money))
```


### 案例
```python
def caculator():
    # 定义美元
    dollar = 100
    # 定义汇率
    exchange = 6.4696
    # 输出结果
    print('{dol}美元兑换的人民币数量为{yuan}'.format(dol=dollar, yuan=dollar * exchange))
```


## 序列
### 特点

- 都可以通过索引得到每一个元素
- 默认索引值总是从零开始
- 可以通过切片的方法得到一个范围内的元素的集合
- 有很多共同的操作符(重复操作符、拼接操作符、成员关系操作符)
- Python针对序列有非常多的内置函数：list(), tuple(), str(), len(), max(), min(), sum(), sorted(), reversed(),enumerate(), zip()等等



### 字符串
#### 操作符
| **操作符** | **描述** | **实例** |
| --- | --- | --- |
| + | 字符串连接 | >>>a + b'HelloPython' |
| * | 重复输出字符串 | >>>a * 2'HelloHello' |
| [] | 通过索引获取字符串中字符 | >>>a[1]'e' |
| [ : ] | 截取字符串中的一部分 | >>>a[1:4]'ell' |
| in | 成员运算符 - 如果字符串中包含给定的字符返回 True | >>>"H"inaTrue |
| not in | 成员运算符 - 如果字符串中不包含给定的字符返回 True | >>>"M"notinaTrue |
| r/R | 原始字符串 - 原始字符串：所有的字符串都是直接按照字面的意思来使用，没有转义特殊或不能打印的字符。 原始字符串除在字符串的第一个引号前加上字母"r"（可以大小写）以外，与普通字符串有着几乎完全相同的语法。 | >>>printr'\\n' \\n >>> printR'\\n' \\n |
| % | 格式字符串 | ​
 |

```python
def string():
    chinese_zodiac1 = "鼠牛虎兔龙蛇"
    chinese_zodiac2 = "马羊猴鸡狗猪"
    chinese_zodiac = chinese_zodiac1 + chinese_zodiac2
    print("chinese_zodiac 输出结果：", chinese_zodiac)
    print("chinese_zodiac * 2 输出结果：", chinese_zodiac * 2)
    print("chinese_zodiac[1] 输出结果：", chinese_zodiac[1])
    print("chinese_zodiac[1:4] 输出结果：", chinese_zodiac[1:4])
    if ("牛" in chinese_zodiac):
        print("牛 在变量 chinese_zodiac 中")
    else:
        print("牛 不在变量 chinese_zodiac 中")

    if ("猫" not in chinese_zodiac):
        print("猫 不在变量 chinese_zodiac 中")
    else:
        print("M 在变量 chinese_zodiac 中")
    print(r'\n')
    print(R'\n')
```
#### 格式化
| **    符   号** | **描述** |
| --- | --- |
|       %c |  格式化字符及其ASCII码 |
|       %s |  格式化字符串 |
|       %d |  格式化整数 |
|       %u |  格式化无符号整型 |
|       %o |  格式化无符号八进制数 |
|       %x |  格式化无符号十六进制数 |
|       %X |  格式化无符号十六进制数（大写） |
|       %f |  格式化浮点数字，可指定小数点后的精度 |
|       %e |  用科学计数法格式化浮点数 |
|       %E |  作用同%e，用科学计数法格式化浮点数 |
|       %g |  %f和%e的简写 |
|       %G |  %F 和 %E 的简写 |
|       %p |  用十六进制数格式化变量的地址 |

```python
def stringFormat():
    chinese_zodiac = "鼠牛虎兔龙蛇马羊猴鸡狗猪"
    # Python 字符串格式化:
    chinese_zodiac_str = chinese_zodiac[2:5]
    print("截取下标 2 到下标 4: %s 共: %d 个数" % (chinese_zodiac_str, len(chinese_zodiac_str)))
    print("chinese_zodiac 输出结果：", chinese_zodiac)
```
#### 字符串函数
> 更多查看 ：[Python 菜鸟学习](https://www.runoob.com/python/python-strings.html)

```python
# 字符串函数
def stringFunc():
    name = "zhengzeqin"
    print("首字母大写:", name.capitalize())
    # 长度
    print("原字符串居中,并使用 * 填充至长度 12 的新字符串:", name.center(12, "*"))
```


### 列表
#### 操作符
| **Python 表达式** | **结果** | **描述** |
| --- | --- | --- |
| len([1, 2, 3]) | 3 | 长度 |
| [1, 2, 3] + [4, 5, 6] | [1, 2, 3, 4, 5, 6] | 组合 |
| ['Hi!'] * 4 | ['Hi!', 'Hi!', 'Hi!', 'Hi!'] | 重复 |
| 3 in [1, 2, 3] | True | 元素是否存在于列表中 |
| for x in [1, 2, 3]: print x, | 1 2 3 | 迭代 |

| L[2] | 'Taobao' | 读取列表中第三个元素 |
| --- | --- | --- |
| L[-2] | 'Runoob' | 读取列表中倒数第二个元素 |
| L[1:] | ['Runoob', 'Taobao'] | 从第二个元素开始截取列表 |

```python
def list():
    class_names = ['PHP', 'Swift', 'Java', 'Python', 'Objective-C']
    print("class_names 个数:", len(class_names))
    print("下标 1 到 3", class_names[1:4])
    if 'PHP' in class_names:
        print("php 在 class_names 中")
    else:
        print("php 不在 class_names 中")
    print("class_names 倒数第 2 个: ", class_names[-2])
    print("下标 3: ", class_names[3])
    print("从 0 读取到下标 2: ", class_names[:3])
    print("从下标 1 开始读取: ", class_names[1:])
```
#### 列表函数
> 更多查看 ：[Python 菜鸟学习](https://www.runoob.com/python/python-lists.html)

| **函数&方法** | **描述** |
| --- | --- |
| len(list) | 列表元素个数 |
| max(list) | 返回列表元素最大值 |
| min(list) | 返回列表元素最小值 |
| list(seq) | 将元组转换为列表 |
| list.append(obj) | 在列表末尾添加新的对象 |
| list.count(obj) | 统计某个元素在列表中出现的次数 |
| list.extend(seq) | 在列表末尾一次性追加另一个序列中的多个值（用新列表扩展原来的列表） |
| list.index(obj) | 从列表中找出某个值第一个匹配项的索引位置 |
| list.insert(index, obj) | 将对象插入列表 |
| list.pop(obj=list[-1]) | 移除列表中的一个元素（默认最后一个元素），并且返回该元素的值 |
| list.remove(obj) | 移除列表中的一个元素（参数是列表中元素），并且不返回任何值 |
| list.reverse() | 反向列表中元素 |
| list.sort([func]) | 对原列表进行排序 |

```python
def listFun():
    class_names = ['PHP', 'Swift', 'Java', 'Python', 'Objective-C']
    class_names.append("Dart")
    print(class_names)
    print("Python 的下标：", class_names.index("Python"))
```


### 元组
#### 操作符
| **Python 表达式** | **结果** | **描述** |
| --- | --- | --- |
| len((1, 2, 3)) | 3 | 计算元素个数 |
| (1, 2, 3) + (4, 5, 6) | (1, 2, 3, 4, 5, 6) | 连接 |
| ('Hi!',) * 4 | ('Hi!', 'Hi!', 'Hi!', 'Hi!') | 复制 |
| 3 in (1, 2, 3) | True | 元素是否存在 |
| for x in (1, 2, 3): print x, | 1 2 3 | 迭代 |

```python
def tuple():
    tuple1 = ('PHP', 'Swift', 'Java',  1, 2, 3)
    tuple2 = 'Python', 'Objective-C', 4, 5
    print("tuple1 取下标 0:", tuple1[1])
    print("tuple1 + tuple2 :", tuple1 + tuple2)
    # 元组内 list 的值被修改了，元组结果也变化
    list = [1, 2, 3]
    tuple3 = (tuple2, list)
    print(tuple3)
    list[0] = 4
    print(tuple3)
    # 删除元组
    del tuple2
```
#### 元组函数
| **方法及描述** |
| --- |
| [cmp(tuple1, tuple2)](https://www.runoob.com/python/att-tuple-cmp.html)  比较两个元组元素。 |
| [len(tuple)](https://www.runoob.com/python/att-tuple-len.html) 计算元组元素个数。 |
| [max(tuple)](https://www.runoob.com/python/att-tuple-max.html) 返回元组中元素最大值。 |
| [min(tuple)](https://www.runoob.com/python/att-tuple-min.html)  返回元组中元素最小值。 |
| [tuple(seq)](https://www.runoob.com/python/att-tuple-tuple.html) 将列表转换为元组。 |



## 参考

- [Python 菜鸟学习](https://www.runoob.com/python/python-strings.html)
- [草根学 Python](https://www.readwithu.com/Article/PythonBasis/python3/List.html)
- [demo](https://github.com/zeqinjie/python_demo) 
