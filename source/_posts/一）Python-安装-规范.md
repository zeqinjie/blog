---
title: 一）Python 安装 & 规范
date: 2022-02-20 16:37:12
categories: "Python"
tags:
	- Python
---


## 安装
### 官方地址下载
![](/images/2022/一）Python-安装-规范/1.png)

> 终端进入 python 环境，可以直接编辑运行exit() 退出

![](/images/2022/一）Python-安装-规范/2.png)
### Pyenv 设置不同版本 python 版本环境


- 现在 mac 系统默认是安装 Python 2.7 及 Python 3.8.9 两个版本  （`/usr/bin/python2.7`）
-  pyenv 可在不同 python 版本之间轻松切换，实现 python 环境隔离，且支持自动激活和退出虚拟环境 [Mac上pyenv的安装与使用](https://juejin.cn/post/7056800493753860103)


### Python Charm 配置 python 版本环境
![](/images/2022/一）Python-安装-规范/3.png)

## 代码规范  [传送门](https://www.readwithu.com/Article/codeSpecification/codeSpecification_first.html)
### 基本

- 编码：使用  UTF-8 编码 如 u"你好世界"
- 缩进：统一使用 4 个空格进行缩进 
   - Python 的函数没有  `{}`，所以是以缩进划分代码块
- 每行代码字符数： < 120


### 引号

- 自然语言 使用双引号 `"..."` 
- 机器标识 使用单引号  `'...'`  例如 dict 里的 key
- 正则表达式 使用原生的双引号 `r"..."`
- 文档字符串 (docstring) 使用三个双引号 `"""......"""`


### 空行

- 模块级函数和类定义之间空两行；
- 类成员函数之间空一行；


### import

- import 语句应该分行书写
- 导入其他模块的类定义时，可以使用相对导入 
```python
from myclass import MyClass
```

- 如果发生命名冲突，则可使用命名空间 
```python
import bar
import foo.bar

bar.Bar()
foo.bar.Bar()
```

### 空格

- 在二元运算符两边各空一格 `[=,-,+=,==,>,in,is not, and]:`
```python
# 正确的写法
i = i + 1

# 不推荐的写法
i=i+1
```

- 函数的参数列表中`，,`之后要有空格
```python
# 正确的写法
def complex(real, imag):
    pass

# 不推荐的写法
def complex(real,imag):
    pass
```


- 函数的参数列表中`，`默认值等号两边不要添加空格
```python
# 正确的写法
def complex(real, imag=0.0):
    pass

# 不推荐的写法
def complex(real, imag = 0.0):
    pass
```

- 左括号之后`，`右括号之前不要加多余的空格
- 字典对象的`左括号`之前不要多余的空格
- 不要为`对齐赋值语句`而使用的额外空格
```python
# 正确的写法
x = 1
y = 2
long_variable = 3

# 不推荐的写法
x             = 1
y             = 2
long_variable = 3
```


### 换行

- 括号内的换行
   - 第二行缩进到括号的起始处
```python
foo = long_function_name(var_one, var_two,
                         var_three, var_four)
```

   - 第二行缩进 4 个空格，适用于起始括号就换行的情形
```python
def long_function_name(
        var_one, var_two, var_three,
        var_four):
    print(var_one)
```


### 注释

- “#”号后空一格，段落件用空行分开
```python
# 块注释
# 块注释
#
# 块注释
# 块注释
```

- 文档注释 
```python
""" 文档描述
这是文档注释
"""
```


### 命名规范

- 模块使用小写命名，下划线 `_` 分割 `import html_parser`
- 类名使用驼峰命名风格，首字母大写，私有类可用一个下划线 `_` 开头
```python
class AnimalFarm(Farm):
    pass

class _PrivateFarm(Farm):
    pass
```

- 函数名一律小写，如有多个单词，用下划线`_`隔开
```python
def run_with_env():
    pass
```

- 私有函数在函数前加一个下划线`_`
```python
def _private_func():
        pass
```

- 变量名小写, 用下划线`_`隔开 `school_name = ''`
- 常量采用全大写，用下划线`_`隔开 `MAX_CLIENT = 100`	



## 官方网站				

- [Python官方文档 ](https://www.python.org/doc/ )
- [iPython](https://www.python.org/doc/ ) 
- [jupyter notebook ](http://jupyter-notebook.readthedocs.io/en/latest/ )
- [sublime text ](https://www.sublimetext.com )
- [PyCharm](https://www.jetbrains.com/pycharm/) 
- [pip](https://pip.pypa.io/en/stable/installing/ ) 



## Python 参考学习资料

- [Python 系列教程（入门系列已写完）](https://juejin.cn/post/6844903567715729415)
- [极客时间 - 零基础学习 Python ](https://time.geekbang.org/course/detail/98-8336)
   - [demo资料 ](https://gitee.com/geektime-geekbang/geekbangpython)
- [草根学 Python](https://www.readwithu.com/Article/PythonBasis/python1/Installation.html)
- [学习 demo](https://github.com/zeqinjie/python_demo)




 				
 			
 		
 	 
