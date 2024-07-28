---
title: 三）Python 实战：解决 Flutter 项目的简体字问题
date: 2024-07-27 14:57:15
categories: "Python"
tags:
	- Python
	- Flutter
---

## 前言

作为面向大陆外市场的应用，我们经常编写代码的时候往往忘记切换繁体字导致上线后出现简体字。因为研究下业内相关插件，看看怎么好解决这个问题。
[OpenCC](https://github.com/BYVoid/OpenCC) 支持语言比较多，所以基于此尝试了用 Python 去实现。

### 遇到问题

1、不支持 m1 的芯片[issue](https://github.com/BYVoid/OpenCC/issues/759)
。 最后采用的是他人修改后的包 [ds-opencc](https://pypi.org/project/ds-opencc/)

2、不过 ds-opencc 要求 python 版本最低需要 3.11.x [support macos arm64 记录](https://github.com/HFAiLab/OpenCC/commit/e368f68801684e11cc30aef91086be33be21ea76)

### 结合 git hooks

结合 git hooks 我们可以很好在每次提交代码去执行一次脚本



1.创建 Git 钩子
在你的 Git 仓库中，进入 .git/hooks 目录。创建一个名为 pre-commit 的文件，Git 会在执行 git commit 之前调用这个钩子
```
#!/bin/bash

# 进入你的项目目录
cd "$(dirname "$0")/../.."

# 执行 Python 脚本
python3 path/to/your/chinese_convert.py -p "$(pwd)"

# 检查脚本执行是否成功
if [ $? -ne 0 ]; then
    echo "转换失败，提交被取消！"
    exit 1
fi

```

当然如果是使用了 pyenv 管理 python 版本时，可能我们需要激活对应的版本脚本可以改成如下
```
#!/bin/bash

# 进入你的项目目录
cd "$(dirname "$0")/../.."

path="$(pwd)"

cd xxx/TWHouseScript

# 确保 pyenv 已经初始化
if command -v pyenv >/dev/null; then
  eval "$(pyenv init --path)"
  eval "$(pyenv init -)"
else
  echo "pyenv 未安装或未正确初始化"
  exit 1
fi

# 激活虚拟环境
if pyenv versions | grep -q 'env3124'; then
  pyenv activate env3124
else
  echo "指定的 pyenv 虚拟环境不存在"
  exit 1
fi

python3 chinese_convert.py -p "$path"

# 检查脚本执行是否成功
if [ $? -ne 0 ]; then
    echo "转换失败，提交被取消！"
    exit 1
fi

```

2.赋予执行权限
> chmod +x .git/hooks/pre-commit


### python 代码
```Python
import os
import sys
import getopt
import ds_opencc

# 创建 OpenCC 实例
cc = ds_opencc.OpenCC('s2tw.json')


def is_comment(line):
    # 判断是否是 Dart 文件中的注释
    return line.strip().startswith('//') or line.strip().startswith('/*') or line.strip().endswith('*/') or line.strip().startswith('*')


def convert_file(file_path):
    with open(file_path, 'r', encoding='utf-8') as file:
        lines = file.readlines()

    converted_lines = []
    in_block_comment = False

    for line in lines:
        if '/*' in line and '*/' not in line:
            in_block_comment = True
        elif '*/' in line:
            in_block_comment = False

        if in_block_comment or is_comment(line):
            converted_lines.append(line)
        else:
            converted_lines.append(cc.convert(line))

    with open(file_path, 'w', encoding='utf-8') as file:
        file.writelines(converted_lines)


def convert_dart_files_in_directory(directory):
    print(f'Converting Dart files in {directory}...')
    for root, _, files in os.walk(directory):
        for file in files:
            if file.endswith('.dart'):
                convert_file(os.path.join(root, file))

# python chinese_convert.py -p '/Users/zhengzeqin/Desktop/GitLab/tw591_xxx'
if __name__ == '__main__':
    argv = sys.argv[1:]
    # 项目路径
    project_path = ""
    try:
        opts, args = getopt.getopt(argv, "p:", ["path="])
    except getopt.GetoptError:
        print('convert.py -p "项目路径"')
        sys.exit(2)

    print("opts ===>", opts)

    for opt, arg in opts:
        if opt in ["-p", "--path"]:
            project_path = arg
            if len(project_path) == 0:
                print('请输入项目的地址')
                sys.exit('请输入项目的地址')

    # 获取需要修复项目的路径
    if len(project_path) == 0:
        current_directory = os.path.dirname(os.path.abspath(__file__))
    else:
        current_directory = project_path
    print(f'current_directory: {current_directory}')
    convert_dart_files_in_directory(current_directory)

```
