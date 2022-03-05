---
title: 一）Python 实战：解决 Flutter 项目 bitcode = true 编译失败问题
date: 2022-03-02 22:15:57
categories: "Python"
tags:
	- Python
	- Flutter
---

## 前言
我们在写 flutter 项目时每次执行 `pub clean` 后， `.ios` 项目会重新创建，项目的 `bitcode` 会重置为 `true`，项目就是编译失败，相信大家都遇到过，每次都需要手动设置 `bitcode = false`，考虑这种手动操作繁琐让我团队成员不胜其扰。同时我近期有学习 `Python` ，就拿这练练手，通过 `py` 脚本一键处理这个问题，大家也就能愉快开发了~😆

![](/images/2022/一）Python-实战：解决-Flutter-项目-bitcode-true-编译失败问题/1.png)

### bitcode 问题
![](/images/2022/一）Python-实战：解决-Flutter-项目-bitcode-true-编译失败问题/2.png)

### 思考
之前有想过用 `shell` 、`ruby` 等去实现这个，搜索下发现有人提了思路 [传送门](https://stackoverflow.com/questions/27178452/set-xcode-build-setting-from-terminal)，我尝试过用 `stackoverflow`上 25 个赞的建议后，尝试发现能执行，但编译失败就放弃。后来采用 [pbxproj](https://github.com/kronenthaler/mod-pbxproj) 刚好是 `Python` 写的，是个契机，学以致用。
![](/images/2022/一）Python-实战：解决-Flutter-项目-bitcode-true-编译失败问题/3.png)

```shell
#pod install
xcodebuild -target house591 -configuration Debug ENABLE_BITCODE=YES
xcodebuild -target house591 -configuration Test ENABLE_BITCODE=YES
xcodebuild -target house591 -configuration Release ENABLE_BITCODE=YES

```

## 环境
首先需要的是安装 `Python3` 环境，这块我就不细说了。如果想用 `py` 虚拟环境，可以使用 `pyenv` 参考这篇 [传送门](https://juejin.cn/post/7056800493753860103) （by 洵锋同学写的）,我用的是 `Python 3.8.10`。


## 插件

设置 `Xcode` `build Setting` 使用的是第三方插件 [传送门](https://github.com/kronenthaler/mod-pbxproj)

```python
# 安装如下
pip3 install pbxproj
```

使用设置 Flutter 项目 Bitcode 核心代码

```python
# 设置 bitcode
def set_bit_code_path(path, is_close_bitcode=True):
    project = XcodeProject.load(path)
    bitcode_str = "NO"
    if not is_close_bitcode:
        bitcode_str = "YES"
    print(f"set_bit_code_path 当前项目路径：{path} bitcode_str: {bitcode_str}")
    set_bit_code_flag(project, bitcode_str, configuration_name="Debug")
    set_bit_code_flag(project, bitcode_str, configuration_name="Profile")
    set_bit_code_flag(project, bitcode_str, configuration_name="Release")
    project.save()


# 设置 bitcode 参数
def set_bit_code_flag(project, bitcode_str, target_name="Runner", configuration_name='Debug'):
    project.set_flags('ENABLE_BITCODE', bitcode_str, target_name=target_name, configuration_name=configuration_name)
```

## 使用
### 方式一

- 直接终端执行，我们自己是使用 `ssh 工具`  [shuttle](https://github.com/fitztrev/shuttle)  一键执行-目前我们的项目往 `Flutter` 技术栈的方向转，所以建了三个 `Flutter module` 业务组件，因此我这边只建三个 Flutter 项目的脚本路径，后续视情况补充脚本命令。
![](/images/2022/一）Python-实战：解决-Flutter-项目-bitcode-true-编译失败问题/4.png)
- 当然我们也可以直接终端执行，如下

```shell
cd /Users/zhengzeqin/Desktop/GitLab/TWHouseScript; python disable_bitcode.py -p '/Users/zhengzeqin/Desktop/GitLab/tw591_salehouse' -s close
```

- 其实 `shuttle` 的命令是用 `json` 来设置，可以结合 [jsonnet](https://github.com/google/jsonnet) 去动态生成，这样就可以复用给别人

### 方式二

如果不想像上述那样，多个项目配合 `shuttle`  命令。路径我们也可以写在脚本里，每个 `Flutter` 项目配置自己的 Python 脚本命令，记得定义好自己的项目路径就好，具体如下


```python
# 定义项目 module 文件名
_module_names = ['tw591_salehouse']
# 项目的父目录
_file_path = "/Users/zhengzeqin/Desktop/GitLab/"
# flutter project.pbxpro
_flutter_pro = '/.ios/Runner.xcodeproj/project.pbxproj'

# 设置 bitcode
def set_bit_code(file_path, module_name):
    path = file_path + module_name + _flutter_pro
    # print("当前项目路径：", path)
    set_bit_code_path(path)
    
# 本地文件配置路径处理 bitcode
def handle_projects_bit_code():
    for module_name in _module_names:
        set_bit_code(_file_path, module_name)
        
if __name__ == "__main__":
    # 本地文件配置路径处理 bitcode
    handle_projects_bit_code()
    # 如果是 flutter 同级下配置可以这样读取 project.pbxproj 路径
    current_path = os.path.abspath(os.curdir) + _flutter_pro
    print(current_path)
```

## 完整脚本代码

```python
# -*- coding: utf-8 -*-
# -*- author: zhengzeqin -*-
# -*- date: 2022-03-01 -*-

# 需要安装依赖库
# pip3 install pbxproj

import os
import getopt
from pbxproj import XcodeProject
import sys

# 定义 flutter 项目 module 文件名
_module_names = ['tw591_salehouse']
# 项目的父目录
_file_path = "/Users/zhengzeqin/Desktop/GitLab/"
# flutter project.pbxpro
_flutter_pro = '/.ios/Runner.xcodeproj/project.pbxproj'


# 本地文件配置路径处理 bitcode
def handle_projects_bit_code():
    for module_name in _module_names:
        set_bit_code(_file_path, module_name)


# 设置 bitcode
def set_bit_code(file_path, module_name):
    path = file_path + module_name + _flutter_pro
    # print("当前项目路径：", path)
    set_bit_code_path(path)


# 设置 bitcode
def set_bit_code_path(path, is_close_bitcode=True):
    project = XcodeProject.load(path)
    bitcode_str = "NO"
    if not is_close_bitcode:
        bitcode_str = "YES"
    print(f"set_bit_code_path 当前项目路径：{path} bitcode_str: {bitcode_str}")
    set_bit_code_flag(project, bitcode_str, configuration_name="Debug")
    set_bit_code_flag(project, bitcode_str, configuration_name="Profile")
    set_bit_code_flag(project, bitcode_str, configuration_name="Release")
    project.save()


# 设置 bitcode 参数
def set_bit_code_flag(project, bitcode_str, target_name="Runner", configuration_name='Debug'):
    project.set_flags('ENABLE_BITCODE', bitcode_str, target_name=target_name, configuration_name=configuration_name)

if __name__ == "__main__":
    # 本地文件配置路径处理 bitcode
    # handle_projects_bit_code()
    # 如果是 flutter 同级下配置可以这样读取 project.pbxproj 路径
    # current_path = os.path.abspath(os.curdir) + _flutter_pro
    # print(current_path)

    argv = sys.argv[1:]
    # 项目路径
    project_path = ""
    # 默认关闭
    is_close_bitcode = True
    try:
        opts, args = getopt.getopt(argv, "p:s:", ["path=", "switch="])
    except getopt.GetoptError:
        print('disable_bitcode.py -p "项目路径"')
        sys.exit(2)

    print("opts ===>", opts)

    for opt, arg in opts:
        if opt in ["-p", "--path"]:
            project_path = arg
            if len(project_path) == 0:
                print('请输入项目的地址')
                sys.exit('请输入项目的地址')
        if opt in ["-s", "--switch"]:
            if arg == "open":
                is_close_bitcode = False
            if arg == "close":
                is_close_bitcode = True

    # 获取需要修复项目的路径
    path = project_path + _flutter_pro
    # 设置 bitcode = false
    set_bit_code_path(path, is_close_bitcode)
```

## 后序思考 🤔

上述解决的是 `iOS` ` bitcode = true  `编译失败问题，其实 `Android` 也有个默认版本低引起编译失败问题，是不是我们也可以通过脚本去解决呢？
