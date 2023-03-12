---
title: Flutter 引擎编译探索
date: 2023-03-13 00:14:25
categories: "Flutter"
tags:
  - Flutter
---

## 前言

基础 Flutter 开发我相信大家都没有什么问题了，但有时候遇到问题想一探引擎载入的问题，但确因为 iOS 项目是将 Flutter 代码打包成 Flutter Framework 报。📢 在拉引擎仓库代码是注意开“通畅的网络” 😁 。

## 环境

### 第一步下载 [depot_tools](http://commondatastorage.googleapis.com/chrome-infra-docs/flat/depot_tools/docs/html/depot_tools_tutorial.html#_setting_up) 

- 下载并配置环境变量

```json
$ mkdir engine
$ cd engine
$ git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
 #(换成自己的目录或在当前目录执行 export PATH=$PATH:`pwd`/depot_tools)
$ export PATH=/path/to/depot_tools:$PATH
```

### 第二步配置 .gclient 文件

- 创建 .gclient 配置文件

```json
solutions = [
  {
    "managed": False,
    "name": "src/flutter",
    "url": "git@github.com:zeqinjie/engine.git",
    "custom_deps": {},
    "deps_file": "DEPS",
    "safesync_url": "",
  },
]
```

- 执行 `gclient sync` 同步仓库代码

![image.png](/images/2023/Flutter引擎编译探索/1.png)

### 第三步切换本地 Flutter 引擎版本

注意：使用对应的Flutter 版本引擎
![image.png](/images/2023/Flutter引擎编译探索/2.png)

- 因为我们项目用的是 3.3.10 ，查看当前 Flutter 版本对应的引擎版本

```shell
cat /Users/zhengzeqin/fvm/versions/3.3.10/bin/internal/engine.version
# Flutter 3.3.10 版本 3316dd8728419ad3534e3f6112aa6291f587078a
```

- 方式一：修改 `**.gclient**` 文件 "url": "git@github.com:flutter/engine.git@3316dd8728419ad3534e3f6112aa6291f587078a",
- or 方式二：切换版本 git reset --hard 3316dd8728419ad3534e3f6112aa6291f587078a

```json
# 切换到engine所在目录
cd engine/src/flutter
# 最好同步远端仓库的 Flutter Engine 代码
git remote add upstream git@github.com:flutter/engine.git
git pull upstream master 
# 制定git版本和本地flutter sdk使用版本一致 3.3.10 版本，
# 遇到切换失败可以先切别版本再切目标版本看看
git reset --hard 3316dd8728419ad3534e3f6112aa6291f587078a
# engine版本切换后，DEPS可能也会改变，后面两个参数表示将tag、refspecs等仓库信息一起同步
# 再次同步下
gclient sync -D --with_branch_heads --with_tags
```

- 如此本地的 Flutter 的引擎版本就和自己调试的一致

![image.png](/images/2023/Flutter引擎编译探索/3.png)

- 同步执行后  gclient sync -D --with_branch_heads --with_tags

![image.png](/images/2023/Flutter引擎编译探索/4.png)

### FlutterEngine 目录结构

```shell
├── assets  #资源读取
├── common  #公共逻辑
├── flow  #渲染管道相关逻辑
├── flutter_frontend_server  #Dart构建相关逻辑
├── fml   #消息循环相关逻辑
├── lib   #Dart Runtime及渲染和Web相关逻辑
├── runtime  #Dart Runtime相关逻辑
├── shell
    ├──platform
        ├──android  #Android Embedder相关逻辑
        ├──common #Embedder公共逻辑
├── sky
├── testing  #测试相关
├── third_party
```

## 安装 ant 和 ninja

```json
brew install ant
brew install ninja
```

## 生成 FlutterEngine 项目

在 flutter/tools 下并没 xcode  的产物，需要通过 GN 来构建元文件，并用 Ninja 编译
![image.png](/images/2023/Flutter引擎编译探索/5.png)

- 使用 `gn` 生成引擎项目

```shell
# 引擎 tools 目录
cd /Users/zhengzeqin/Desktop/engine/src/flutter/tools
# 生成ios设备用的未经编译的工程
./gn --ios --unoptimized  
# 生成ios设备用的工程，不带符号表
./gn --ios
# 生成release工程
./gn --ios --runtime-mode=release
# 生成支持模拟器版本工程
./gn --ios --simulator --unoptimized
# *生成仅支持真机的工程
./gn --ios --unoptimized 
# *生成主机端可执行文件所需的构建文件
./gn --unoptimized
# 可以指定cpu
./gn --runtime-mode=release --ios --ios-cpu=arm64
```

- 执行完毕后，src/out 文件目录下生成 iOS 产物

![image.png](/images/2023/Flutter引擎编译探索/6.png)

## 编译 FlutterEngine 项目

- 使用 `ninja` 编译项目

```shell
# 引擎 src/out 目录
cd /Users/zhengzeqin/Desktop/engine/src/out
# 编译release：
ninja -C ios_release
# 编译真机使用不带符号的debug模式：
ninja -C ios_debug && ninja -C host_debug
# *编译真机使用带符号的debug模式
ninja -C ios_debug_unopt && ninja -C host_debug_unopt
# 编译模式器使用的debug模式
ninja -C ios_debug_sim_unopt && ninja -C host_debug_unopt
```

### 遇到问题

![image.png](/images/2023/Flutter引擎编译探索/7.png)

- - 参考 [#34870](https://github.com/flutter/engine/pull/34870/files#diff-ba1ed52d40eaf296f5ef2032ac802f734204ca428b8ef669d97c913004eb423a) 修改后即可

![image.png](/images/2023/Flutter引擎编译探索/8.png)

## 调试项目

- vscode 中 launch.json 设置

```shell
{
  "name": "flutter_demo",
  "request": "launch",
  "type": "dart",
  "args": [
    "--local-engine-src-path",
    "/Users/zhengzeqin/Desktop/engine/src",
    "--local-engine",
    "ios_debug_unopt"
  ]
},
```

![image.png](/images/2023/Flutter引擎编译探索/9.png)

- 调试引擎代码，请打开你的 Runner.xcworkspace，然后将 flutter_engine 拖入项目

![image.png](/images/2023/Flutter引擎编译探索/10.png)

## 其他

### gclient 简介 [传送门](https://www.cnblogs.com/xl2432/p/11596695.html)

gclient 是谷歌开发的一套跨平台git仓库管理工具，用来将多个git仓库组成一个solution进行管理。总体上，其核心功能是根据一个Solution的DEPS文件所定义的规则将多个git仓库拉取到指定目录。例如，chromium就是由80多个独立仓库组成。

## 参考

- iOS 引擎调试
  - [Flutter - 引擎调试（iOS篇）](https://juejin.cn/post/7209542304862945340) 
  - [Flutter Engine 编译 —— 我是这样读源码的](https://juejin.cn/post/6844904071342768141)
  - [探究Flutter Engine调试](https://juejin.cn/post/6844904048949198862#comment)
- Android 引擎调试
  - [2.0 Flutter源码获取与编译](https://juejin.cn/post/7081438492588245022)
- 其他
  - [https://github.com/flutter/flutter/wiki/Compiling-the-engine](https://github.com/flutter/flutter/wiki/Compiling-the-engine)
  - https://chromium.googlesource.com/chromium/tools/depot_tools.git/+/HEAD/README.gclient.md