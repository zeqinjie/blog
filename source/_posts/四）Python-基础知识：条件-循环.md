---
title: 四）Python 基础知识：条件 & 循环
date: 2022-02-23 10:10:37
categories: "Python"
tags:
    - Python
---

## 条件

-   or （或）表示两个条件有一个成立时判断条件成功
-   and （与）表示只有两个条件同时成立的情况下，判断条件才成功。

```
def conditionFunc():
    english = 89
    chinese = 90
    if english > 90 and chinese > 90:
        print('优秀')
    elif english > 80 and chinese > 80:
        print('良好')
    elif english < 60 or chinese < 60:
        print ('有一门不及格')
    else:
        print ('及格')
```

### if 嵌套

```
def ifFunc():
    english = 89
    chinese = 90
    if english > 90:
        if chinese > 90:
            print('优秀') 
        else:
            print ('english 优秀，chinese 及格')
    else:
        print ('及格')
```

## 循环

```
# 字典 & 集合遍历
def dictFunc():
    dict1 = {'1': 'Java', '2': 'PHP', '3': 'Swift'}
    for key in dict1.keys():
            print("key is:", key, "value is:", dict1[key])
    set1 = set(['Shell', 'Python', 'Ruby', 'Java']) 
    for value in set1:
        print("value 的值:", value)
```

### range()

```
def rangeFunc():
    # 取从 0 到 3 之间  结果： 0,1,2
    for i in range(3):
        print(i)
    # 取从 3 到 6 之间  结果： 3,4,5
    for i in range(3, 6):
        print(i)
    # 取从 0 到 10 之间，递增2  结果： 0,2,4,6,8
    for i in range(0, 10, 2):
        print(i)
    
```

### while

```
def whileFunc():
    i = 0
    while i < 10:
        print(i)
        i = i + 1

    count = 1
    sum = 0
    
    while (count <= 100):
        if ( count % 2 == 0):  # 双数时跳过输出
            count = count + 1
            continue
        sum = sum + count
        count = count + 1
    print(sum)
    
    # 打印九九乘法表
    for i in range(1, 10):
            for j in range(1, i+1):
                # 打印语句中，大括号及其里面的字符 (称作格式化字段) 将会被 .format() 中的参数替换,注意有个点的
                print('{}x{}={}\t'.format(i, j, i*j), end='')  
            print()
```

| **循环控制语句** | **描述**                         |
| ---------- | ------------------------------ |
| break      | 在语句块执行过程中终止循环，并且跳出整个循环         |
| continue   | 在语句块执行过程中终止当前循环，跳出该次循环，执行下一次循环 |
| pass       | pass 是空语句，是为了保持程序结构的完整性        |

## 参考

-   [草根学 Python](https://www.readwithu.com/Article/PythonBasis/python5/Cycle.html)
-   [demo](https://github.com/zeqinjie/python_demo)
