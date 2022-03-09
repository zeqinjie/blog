---
title: 二）Python 实战：解决 Flutter 项目 Android minSdkVersion 及 kotlin 版本低的问题
date: 2022-03-08 15:25:57
categories: "Python"
tags:
	- Python
	- Flutter
---

## 前言

我们在写 `flutter` 项目时每次执行  `pub clean` 后 `.android` 项目同样会重新创建，在我们项目中遇到`minSdkVersion` 默认`(17)`及 `kotlin` 默认 `(1.3.50)` 版本低，相信大家都遇到过，每次都需要手动设置，考虑这种手动操作繁琐让我团队成员不胜其扰。通过  `py` 脚本一键处理这个问题，大家也就能愉快开发了~😆

### minSdkVersion 版本问题

![](/images/2022/二）Python-实战：解决-Flutter-项目-Android-minSdkVersion-及-kotlin-版本低的问题/1.png)

### kotlin 版本问题

![](/images/2022/二）Python-实战：解决-Flutter-项目-Android-minSdkVersion-及-kotlin-版本低的问题/2.png)

### 思考
之前我们手动修改 flutter 自动生成的 build.gradle 的模板文件，具体可以看这里 [传送门](https://blog.csdn.net/pf_1308108803/article/details/117221491)。但每次都需要自己去手动去修改模板文，文件路径打开比较麻烦，并且我们更新 flutter 版本后这里也会重置掉。既然如此我想通过 `Python` 脚本自动去生成对应的模板文件覆盖即可。

## 使用
### 方式一

- 配置好 `ssh 工具`  [shuttle](https://github.com/fitztrev/shuttle)  一键执行，其实 `shuttle` 的命令是用 `json` 来设置，可以结合 `[jsonnet](https://github.com/google/jsonnet)` 去动态生成，这样就可以复用给别人

![](/images/2022/二）Python-实战：解决-Flutter-项目-Android-minSdkVersion-及-kotlin-版本低的问题/3.png)

- 当然我们也可以直接终端执行,不过需要自己写好对应的路径相对麻烦些如下, 所以我更推荐使用 `ssh 工具`  [shuttle](https://github.com/fitztrev/shuttle)  一键执行。

```shell
cd /Users/zhengzeqin/Desktop/GitLab/TWHouseScript; python flutter_android_build_gradle_set.py -p '/Users/addcn/flutter/packages/flutter_tools/templates/module/android/gradle/build.gradle.tmpl' -a /Users/addcn/flutter/packages/flutter_tools/templates/module/android/host_app_common/app.tmpl/build.gradle.tmpl
```

### 方式二

路径我们也可以写在脚本里，定义好需要修改的 flutter sdk 的目标模板路径即可，具体如下

```python
# 定义 flutter 目标 android project build_gradle 配置的路径
_project_build_gradle_set_file = "/Users/zhengzeqin/flutter/packages/flutter_tools/templates/module/android/gradle/build.gradle.tmpl"
# 定义 flutter 目标 android app build_gradle 配置的路径
_app_build_gradle_set_file = "/Users/zhengzeqin/flutter/packages/flutter_tools/templates/module/android/host_app_common/app.tmpl/build.gradle.tmpl"

# 定义需要写入的 android project 根目录的 build_gradle 配置
_build_gradle_tmpl_for_project = "/build_gradle_tmpl_for_project.txt"

# 定义需要写入的 android app 目录的 build_gradle 配置
_build_gradle_tmpl_for_app = "/build_gradle_tmpl_for_app.txt"


# ...


if __name__ == "__main__":
    # 执行脚本路径修改 Android build gradle 的目标模板
    set_android_build_gradle_params_files()
    pass
```

## 完整脚本代码

### 自定义 Android Project 的 build gradle 模板

```
def flutterPluginVersion = 'managed'

apply plugin: 'com.android.application'

android {
    compileSdkVersion 31

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }

    defaultConfig {
        applicationId "{{androidIdentifier}}.host"
        minSdkVersion 21
        targetSdkVersion 31
        versionCode 1
        versionName "1.0"
    }

    buildTypes {
        profile {
            initWith debug
        }
        release {
            // TODO: Add your own signing config for the release build.
            // Signing with the debug keys for now, so `flutter run --release` works.
            signingConfig signingConfigs.debug
        }
    }

}
buildDir = new File(rootProject.projectDir, "../build/host")
dependencies {
    implementation project(':flutter')
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation 'androidx.appcompat:appcompat:1.0.2'
    implementation 'androidx.constraintlayout:constraintlayout:1.1.3'
}


```

### 自定义 Android App 的 build gradle 模板

```
// Generated file. Do not edit.

buildscript {
	ext.kotlin_version = '1.6.10'
    repositories {
        google()
        mavenCentral()
    }

    dependencies {
        classpath 'com.android.tools.build:gradle:{{agpVersion}}'
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
        classpath "org.jetbrains.kotlin:kotlin-android-extensions:$kotlin_version"
    }
}

allprojects {
    repositories {
        google()
        mavenCentral()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}

```

### 需要修改的 Flutter 目标模板

![](/images/2022/二）Python-实战：解决-Flutter-项目-Android-minSdkVersion-及-kotlin-版本低的问题/4.png)

### 脚本代码

```python
# -*- coding: utf-8 -*-
# -*- author: zhengzeqin -*-
# -*- date: 2022-03-07 -*-

import os
import sys
import getopt

# 定义 flutter 目标 android project build_gradle 配置的路径
_project_build_gradle_set_file = "/Users/zhengzeqin/flutter/packages/flutter_tools/templates/module/android/gradle/build.gradle.tmpl"
# 定义 flutter 目标 android app build_gradle 配置的路径
_app_build_gradle_set_file = "/Users/zhengzeqin/flutter/packages/flutter_tools/templates/module/android/host_app_common/app.tmpl/build.gradle.tmpl"

# 定义需要写入的 android project 根目录的 build_gradle 配置
_build_gradle_tmpl_for_project = "/build_gradle_tmpl_for_project.txt"

# 定义需要写入的 android app 目录的 build_gradle 配置
_build_gradle_tmpl_for_app = "/build_gradle_tmpl_for_app.txt"

# 读取文件内容
def readfile(file_name):
    file2 = open(file_name)
    str = file2.read()
    file2.close()
    return str


# 设置 build_gradle 的内容
def android_gradle_set():
    # 获取当前文件路径
    concurrent = os.path.abspath(os.path.dirname(__file__))
    read_file_path = concurrent + '/build_gradle_tmpl_for_project.txt'
    write_file_path = concurrent + '/build.gradle.tmpl'
    read_file = open(read_file_path, 'r')
    write_file = open(write_file_path, "w")

    s = read_file.read()
    w = write_file.write(s)

    read_file.close()
    write_file.close()

# 从文件路径中写入到目标文件
def set_build_gradle_file(read_file_path, write_file):
    read_file = open(read_file_path, 'r')
    write_file = open(write_file, "w")
    s = read_file.read()
    write_file.write(s)
    read_file.close()
    write_file.close()

# 获取当前文件夹的路径
def get_current_path():
    return os.path.abspath(os.path.dirname(__file__))

# 设置 app 的 file
def set_app_build_gradle_file():
    read_file_path = get_build_gradle_file(_build_gradle_tmpl_for_app)
    set_build_gradle_file(read_file_path, _app_build_gradle_set_file)

# 设置 project 根目录的 file
def set_pro_build_gradle_file():
    read_file_path = get_build_gradle_file(_build_gradle_tmpl_for_project)
    set_build_gradle_file(read_file_path, _project_build_gradle_set_file)

# 读取本地配置文件
def get_build_gradle_file(build_gradle_tmpl):
    concurrent = os.path.abspath(os.path.dirname(__file__))
    read_file_path = concurrent + build_gradle_tmpl
    return  read_file_path

# 通过本地文件配置路径修改 Android 项目的 build_gradle 配置
def set_android_build_gradle_params_files():
    set_pro_build_gradle_file()
    set_app_build_gradle_file()

# 通过参数配置路径修改 Android 项目的 build_gradle 配置
def set_android_build_gradle_argv_files():
    argv = sys.argv[1:]
    # android project build_gradle 配置的路径
    project_path = ""
    # android app build_gradle 配置的路径
    app_path = ""
    # 默认关闭
    is_close_bitcode = True
    try:
        opts, args = getopt.getopt(argv, "p:a:", ["project=", "app="])
    except getopt.GetoptError:
        print('flutter_android_gradle_set.py -p "项目路径"')
        sys.exit(2)

    print("opts ===>", opts)

    for opt, arg in opts:
        if opt in ["-p", "--project"]:
            project_path = arg
        if opt in ["-a", "--app"]:
            app_path = arg
    # 设置 project build gradle
    read_pro_file_path = get_build_gradle_file(_build_gradle_tmpl_for_project)
    set_build_gradle_file(read_pro_file_path, project_path)
    # 设置 app build gradle
    read_app_file_path = get_build_gradle_file(_build_gradle_tmpl_for_app)
    set_build_gradle_file(read_app_file_path, app_path)

if __name__ == "__main__":
    # set_android_build_gradle_params_files()
    set_android_build_gradle_argv_files()
    pass

```

## 脚本地址

- [python_demo](https://github.com/zeqinjie/python_demo)