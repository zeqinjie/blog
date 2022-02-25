---
title: 五）Python 基础知识：函数 & 推导式
date: 2022-02-23 23:14:46
categories: "Python"
tags:
	- Python
---

## 函数

- 函数代码块以 def 关键词开头，后接函数标识符名称和圆括号()
- 任何传入参数和自变量必须放在圆括号中间。圆括号之间可以用于定义参数
- 函数的第一行语句可以选择性地使用文档字符串（用于存放函数说明）
- 函数内容以冒号起始，并且缩进
- return [表达式] 结束函数，选择性地返回一个值给调用方。不带表达式的 return 相当于返回 None

### 定义函数

- 无返回值，默认返回 None ， `return None`
- 函数如果没有函数体，可以用 `pass` 或者 `return` 占位

```python
def add(num1, num2):
    "求两数相加值"
    return num1 + num2

def reduce(num1, num2):
    "求两数相减值"
    return num1 - num2

def multiply(num1, num2):
    "求两数相乘值"
    return num1 * num2

def testFunc():
    a = 100
    b = 20
    print("a+b = ", add(a, b))
    print("a-b = ", reduce(a, b))
    print("axb = ", multiply(a, b))
    print("a/b = ", divide(a, b))
```


### 默认参数值的函数

-   第一个默认值可以省略形参名，其他位置不可以

```
# 补充默认值
def divide(num1 = 4, num2 = 2):
    "求两数相除值"
    return num1 / num2

def testFunc():
    print("b 取默认值 = ", divide(num1=20))
    print("a 取默认值 = ", divide(num2=100))
```


### 返回多个值的函数

- 注意一次接受多个返回值的数据类型就是元组

``` python
# 返回多个值
def getValues(num1, num2):
    a = num1 % num2
    b = (num1 - a) / num2
    return b, a

def testFunc():
    num1, num2 = getValues(10, 2)
    tuple1 = getValues(10, 2)
    print("求商与余数", num1, num2)
    print("求商与余数的元组", tuple1)
```

### 不定长参数函数

- 通过跟参数加 `*` 添加不定长的参数
- 其实本质上加了 `*` 的参数就是元组

``` python
def printInfo(arg1, *vartuple):
    print("输出: ", arg1)
    for var in vartuple:
        print("输出: ", var)
    return None

def testFunc():
    printInfo("我叫", "zhenzeqin", "age = 20", "height = 100kg")
```


### 只接受关键字参数

- 通过给函数加 `*` 形参，后面的参数则强制要使用参数名，否则报错
``` python
# 打印用户信息
def print_user_info(name, *, age, height= 200):
    print('昵称：{}'.format(name), end=' ')
    print('年龄：{}'.format(age), end=' ')
    print('体重：{}'.format(height))
    
def testFunc():
    # print_user_info("zhengzeqin", 20, 100) # 报错 参数是 * 号后的不能省略参数名
    print_user_info("zhengzeqin", age= 20, height = 100)
```

### 匿名函数

- lambda 只是一个表达式，函数体比 def 简单很多。
- lambda 的主体是一个表达式，而不是一个代码块。仅仅能在 lambda 表达式中封装有限的逻辑进去。
- lambda 函数拥有自己的命名空间，且不能访问自有参数列表之外或全局命名空间里的参数。

基本语法: `lambda [arg1 [,arg2,.....argn]]:expression`

```python
# lambda 匿名函数
def lambdaFunc():
    sum = lambda num1, num2: num1 + num2;
    print(sum(1, 2))
```

### 函数传值

- 不可变的对象：字符串，整形，浮点型，所以函数内部不会改变外传入的参数
- 可以变的对象： list ， dict 等，与上相反

```python
# 不可以变
def unchange(num):
    num = num + 1
    print("unchange 里 num 的值", num)

# 可以变
def change(list):
    list.append(123)
    print("change 里 list 的值", list)

def testFunc():
    # 不可变
    num11 = 100
    unchange(num11)
    print("unchange 外 num 的值", num11)

    # 可以变
    list11 = [1, 2, 3]
    change(list11)
    print("change 外 list 的值", list11)
```


## 推导式

```python
# 推导式
def forFunc():
    # 从1 到 10  所有偶数的平方
    alist = []
    for i in range(1, 11):
        if i % 2 == 0:
            alist.append( i*i )
    print("从1 到 10  所有偶数的平方：", alist)
    blist = [i*i for i in range(1, 11) if(i % 2) == 0]
    print("blist 价于上面 alist：", blist)
    
    # 数组 class_namse 转字典
    class_names = ['PHP', 'Swift', 'Java', 'Python', 'Objective-C']
    adict = {}
    for i in class_names:
        adict[i] = 0
    print("生成字典：", adict)
    bdict = {i: 0 for i in class_names}
    print("bdict 等价于上面 adict：", bdict)
```


## 参考

-  [草根学 Python](https://www.readwithu.com/Article/PythonBasis/python6/4.html)
-  [推导式](https://www.cnblogs.com/songdanlee/p/11104725.html)
-  [demo](https://github.com/zeqinjie/python_demo)
