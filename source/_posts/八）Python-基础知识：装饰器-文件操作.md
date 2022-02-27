---
title: 八）Python 基础知识：装饰器 & 文件操作
date: 2022-02-27 15:43:15
categories: "Python"
tags:
	- Python
---


## 装饰器
### 不带参数

```python
def timmer(func):
    def wrapper():
        start_time = time.time()
        func()
        stop_time = time.time()
        print("运行时间是 %s 秒 " % (stop_time - start_time))
    return wrapper

@timmer
def i_can_sleep():
    time.sleep(3)
```

### 带参数

```python
def tips(func):
    def nei(a, b):
        print('start')
        func(a, b)
        print('stop')
        return 'I am here!!!!'
    return nei

@tips
def add(a, b):
    print(a+b)
```

### 给装饰器带参数

```python
def new_tips(argv):
    def tips(func):
        def nei(a, b):
            start_time = time.time()
#这里加个变量接收返回值
            result = func(a, b)
            end_time = time.time()
            print('%s 运行时间: %s ' %(argv, end_time-start_time))
#这里再返回
            return result
        return nei
    return tips

@new_tips('sum_model')
def sum(a, b):
   return a+b

@new_tips('sub_model')
def sub(a, b):
   return a-b
```

## 文件操作

### open 

语法 `file object = open(file_name [, access_mode][, buffering])`

- file_name：file_name变量是一个包含了你要访问的文件名称的字符串值。
- access_mode：access_mode决定了打开文件的模式：只读，写入，追加等。所有可取值见如下的完全列表。这个参数是非强制的，默认文件访问模式为只读(r)。
- buffering:如果buffering的值被设为0，就不会有寄存。如果buffering的值取1，访问文件时会寄存行。如果将buffering的值设为大于1的整数，表明了这就是的寄存区的缓冲大小。如果取负值，寄存区的缓冲大小则为系统默认。
```python
# 读取文件内容
def readfile(file_name = 'name.txt'):
    file2 = open(file_name)
    str = file2.read()
    file2.close()
    return str
```

### write

- write()方法可将任何字符串写入一个打开的文件。需要重点注意的是，Python字符串可以是二进制数据，而不是仅仅是文字。
- write()方法不会在字符串的结尾添加换行符('\n')：
- 参数 `w` : 写入  `a` : 追加

```python
# 写入文件内容
def writefile(file_name = 'name.txt', txt = ''):
    file1 = open(file_name, 'w')
    file1.write(txt)
    file1.close()

# 追加文件内容
def appendfile(file_name = 'name.txt', txt = ''):
    file1 = open(file_name, 'a')
    file1.write(txt)
    file1.close()
```

### read
语法：`fileObject.read([count])`

- 方法从一个打开的文件中读取一个字符串。需要重点注意的是，Python字符串可以是二进制数据，而不是仅仅是文字。
```python
# 读取文件内容
def readfile(file_name = 'name.txt'):
    file2 = open(file_name)
    str = file2.read()
    file2.close()
    return str
```

### readline & readlines

- readline 读取一行
- readlines 读取每行

```python
# 读取一行
def readfileline(file_name = 'name.txt'):
    file4 = open(file_name)
    str = file4.readline()
    file4.close()
    return str

# 读取每行
def readfilelines(file_name = 'name.txt'):
    file5 = open(file_name)
    print("====读取每行====")
    for line in file5.readlines():
        print(line)
        print("===============")
    file5.close()
```

### seek

- `tell() `方法告诉你文件内的当前位置, 换句话说，下一次的读写会发生在文件开头这么多字节之后。
- `seek(offset [,from])`方法改变当前文件的位置。Offset 变量表示要移动的字节数。From变量指定开始移动字节的参考位置。
- 如果from被设为0，这意味着将文件的开头作为移动字节的参考位置。如果设为1，则使用当前的位置作为参考位置。如果它被设为2，那么该文件的末尾将作为参考位置。

```python
# 打印当前位置，和读取个数
def readfilecurrent(file, num = 1):
    print('当前文件指针的位置 %s' % file.tell())
    print('当前读取到了一个字符，字符的内容是 %s' % file.read(num))
    print('当前文件指针的位置 %s' % file.tell())
```
```python

def file_func():
    # 写入文件
    writefile(txt = 'a.诸葛亮')

    # 读取文件
    print(f"读取文件内容: {readfile()}")

    # 增加内容
    appendfile(txt = '\nb.刘备')

    # 读取 1 行
    print("读取 1 行", readfileline())

    # 读取每行
    readfilelines()

    file6 = open('name.txt')
    readfilecurrent(file6, num=1)
    # 第一个参数代表偏移位置，第二个参数  0 表示从文件开头偏移  1表示从当前位置偏移，   2 从文件结尾
    file6.seek(5, 0)
    readfilecurrent(file6, num=1)

    file6.close()

def main():
    file_func()

if __name__ == '__main__':
    main()
```


## 参考

- [Python 函数装饰器](https://www.runoob.com/w3cnote/python-func-decorators.html)
- [Python 文件操作](https://www.runoob.com/python/python-files-io.html)
- [demo](https://github.com/zeqinjie/python_demo)
