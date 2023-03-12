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

![image.png](https://cdn.nlark.com/yuque/0/2023/png/2127799/1678603676926-62bf2db4-b578-4cf4-829d-2b7c4ccc00c0.png#averageHue=%23090909&clientId=u3a707ba6-39e5-4&from=paste&height=346&id=u4cb92ac0&name=image.png&originHeight=692&originWidth=1266&originalType=binary&ratio=2&rotation=0&showTitle=false&size=216104&status=done&style=none&taskId=ub58f47bd-de9c-4902-9343-e431f942f94&title=&width=633)

### 第三步切换本地 Flutter 引擎版本

注意：使用对应的Flutter 版本引擎
![image.png](https://cdn.nlark.com/yuque/0/2023/png/2127799/1678602342880-3c7fa327-585b-451b-b791-f1eda70ebaca.png#averageHue=%23060606&clientId=u3a707ba6-39e5-4&from=paste&height=423&id=u970719e4&name=image.png&originHeight=846&originWidth=1412&originalType=binary&ratio=2&rotation=0&showTitle=false&size=200628&status=done&style=none&taskId=u1598848d-cfb6-49af-a974-2c2b7842b8b&title=&width=706)

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

![image.png](https://cdn.nlark.com/yuque/0/2023/png/2127799/1678605759245-7908bf19-7377-4893-8b2a-b7422f7b8a81.png#averageHue=%23090808&clientId=u3a707ba6-39e5-4&from=paste&height=118&id=uffc0899f&name=image.png&originHeight=236&originWidth=1150&originalType=binary&ratio=2&rotation=0&showTitle=false&size=62885&status=done&style=none&taskId=ub050e67f-2d69-409d-9fff-ad90eeb7f12&title=&width=575)

- 同步执行后  gclient sync -D --with_branch_heads --with_tags

![image.png](https://cdn.nlark.com/yuque/0/2023/png/2127799/1678605386304-26ea086c-6588-4760-899a-7fa78009ea00.png#averageHue=%23070707&clientId=u3a707ba6-39e5-4&from=paste&height=428&id=udf516860&name=image.png&originHeight=856&originWidth=1534&originalType=binary&ratio=2&rotation=0&showTitle=false&size=239887&status=done&style=none&taskId=u722dedbe-409c-4e56-b711-f223b52304c&title=&width=767)

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
![image.png](https://cdn.nlark.com/yuque/0/2023/png/2127799/1678606969228-0f6f1e12-5625-400a-b015-a7893de2ffa1.png#averageHue=%23040303&clientId=u3a707ba6-39e5-4&from=paste&height=346&id=ud94c1a88&name=image.png&originHeight=692&originWidth=1216&originalType=binary&ratio=2&rotation=0&showTitle=false&size=179558&status=done&style=none&taskId=ub401436d-e988-44d0-bcd0-386e2fe7adf&title=&width=608)

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

![image.png](https://cdn.nlark.com/yuque/0/2023/png/2127799/1678616718216-233a8371-54d9-4b03-91cb-151752f75d38.png#averageHue=%23a2a2a0&clientId=u3a707ba6-39e5-4&from=paste&height=229&id=u812950c3&name=image.png&originHeight=458&originWidth=1194&originalType=binary&ratio=2&rotation=0&showTitle=false&size=105103&status=done&style=none&taskId=u036d910e-24c7-43fa-b95b-6bc9c918e43&title=&width=597)

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

![image.png](https://cdn.nlark.com/yuque/0/2023/png/2127799/1678632852125-abbedc03-0f39-47be-a1fe-6bbc63ce0f53.png#averageHue=%230d0d0d&clientId=u3a707ba6-39e5-4&from=paste&height=255&id=u248a2aaa&name=image.png&originHeight=510&originWidth=1222&originalType=binary&ratio=2&rotation=0&showTitle=false&size=126194&status=done&style=none&taskId=u5bf9e751-6b52-4a4e-88fd-de963866d4a&title=&width=611)

- - 参考 [#34870](https://github.com/flutter/engine/pull/34870/files#diff-ba1ed52d40eaf296f5ef2032ac802f734204ca428b8ef669d97c913004eb423a) 修改后即可

![image.png](https://cdn.nlark.com/yuque/0/2023/png/2127799/1678635185487-6c96171a-7a06-4216-9097-ddf3b5f2ec4d.png#averageHue=%23080707&clientId=u3a707ba6-39e5-4&from=paste&height=175&id=u546514cc&name=image.png&originHeight=350&originWidth=1156&originalType=binary&ratio=2&rotation=0&showTitle=false&size=68624&status=done&style=none&taskId=uc7eb3518-10aa-420e-ae4f-58fe09a64d3&title=&width=578)

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

![image.png](https://cdn.nlark.com/yuque/0/2023/png/2127799/1678636481886-c6e8a706-bad7-44ab-a2b3-965ea52c2113.png#averageHue=%23f3e0a1&clientId=ue382833a-a1a1-4&from=paste&height=382&id=u04c9a61b&name=image.png&originHeight=764&originWidth=1664&originalType=binary&ratio=2&rotation=0&showTitle=false&size=252699&status=done&style=none&taskId=u1e142deb-ddb7-47c7-89a8-1c6bc80697c&title=&width=832)

- 调试引擎代码，请打开你的 Runner.xcworkspace，然后将 flutter_engine 拖入项目

## ![image.png](https://cdn.nlark.com/yuque/0/2023/png/2127799/1678636849666-78a81b14-5463-4e9a-b3f2-b1685612ef7b.png#averageHue=%23ebdfb2&clientId=ue382833a-a1a1-4&from=paste&height=412&id=uff45b1f1&name=image.png&originHeight=824&originWidth=2252&originalType=binary&ratio=2&rotation=0&showTitle=false&size=521970&status=done&style=none&taskId=u59872e2c-6588-4d75-a20d-cadd50cdc21&title=&width=1126)

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