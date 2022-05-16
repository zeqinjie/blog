---
title: Flutter搭建私有Pub仓库Docker部署
date: 2022-05-16 13:58:42
categories: "Flutter"
tags:
	- Flutter
---

## 前言
在 Flutter 开发中，考虑到我们不同业务组件下依赖不同版本的基础组件，如果采用分支依赖管理特别不方便，因此搭建私有 pub 包服务很有搭建必要。在技术调研后发现 pub 官方开源的的 [pub server](https://github.com/dart-archive/pub_server) 已有两年多没有更新，且现在已调整为只读。因此使用基于字节开源的 [unpub](https://github.com/bytedance/unpub) 开源搭建私有仓库平台。

## unpub 私有平台搭建

### 安装 `MongoDB`

#### 官方安装方式  [传送门](https://www.mongodb.com/try#community)
 选择 `On-premises MongoDB locally` 下载 
 
![images](/images/2022/Flutter搭建私有Pub仓库Docker部署/1.png)

在 `.zshrc`  添加环境变量

```shell
# 写入环境变量
export PATH=/Users/zhengzeqin/flutter/bin:$PATH
# 写入环境变量
export PATH="/usr/local/opt/mongodb-community@4.4/bin:$PATH
```
#### `homebrew` 安装
```shell
brew tap mongodb/brew
# 4.4 版本号
brew install mongodb-community@4.4
# 启动服务
brew services start mongodb-community@4.4
# 查看已启动服务
brew services list
```

![images](/images/2022/Flutter搭建私有Pub仓库Docker部署/2.png)

#### Docker 安装 [ 传送门](https://www.runoob.com/docker/docker-install-mongodb.html)
### unpub  [传送门](https://github.com/bytedance/unpub)
#### 在 `.zshrc`  添加环境变量
```shell
export PATH="$PATH":"$HOME/flutter/.pub-cache/bin"
```
#### 安装 `unpub`
```shell
flutter pub global activate unpub
```
### 去掉 unpub 的 google 验证
![images](/images/2022/Flutter搭建私有Pub仓库Docker部署/3.png)

- 查找 `app.dart` ， 修改使用 `_getUploaderEmail` 的地方

- 第一处

![images](/images/2022/Flutter搭建私有Pub仓库Docker部署/4.png)

- 第二处

![images](/images/2022/Flutter搭建私有Pub仓库Docker部署/5.png)

- 第三处

![images](/images/2022/Flutter搭建私有Pub仓库Docker部署/6.png)

#### Flutter 启动命令

```shell
flutter pub global run 'unpub:unpub' --database 'mongodb://localhost:27017/dart_pub'
# 失效
flutter pub global deactivate unpub
```

#### Dart 启动命令

```shell
dart pub global activate unpub

dart pub --trace pub global run 'unpub:unpub' --database 'mongodb://localhost:27017/dart_pub'

# 失效
dart pub global deactivate unpub
```

#### Get 点

- 项目中存在两个 `.pub-cache` 注意别修改错地方
   - /Users/zhengzeqin/flutter/.pub-cache
   - /Users/zhengzeqin/.pub-cache
- 项目 `.pub-cache` 下的 `pub.flutter-io.cn` 和 `pub.dartlang.org` 文件夹， 插件源码安装与执行上述 `flutter pub global activate` 和 `dart pub global activate` 有关系
- 修复 flutter 缓存插件包问题

```shell
flutter channel stable 
flutter upgrade
flutter pub cache repair //To perform a clean reinstall of the packages in your system cache, use pub cache repair
```

### mogodb 启动后执行下面命令启动

```shell
flutter pub global run 'unpub:unpub' --database 'mongodb://localhost:27017/dart_pub'
```

#### 成功启动

![images](/images/2022/Flutter搭建私有Pub仓库Docker部署/7.png)

#### 私有平台

![images](/images/2022/Flutter搭建私有Pub仓库Docker部署/8.png)

### 开机启动 shell 脚本问题

- [开机自动启动 shell 脚本](https://www.xiaocaicai.com/2021/11/mac-os-%e5%a2%9e%e5%8a%a0%e5%bc%80%e6%9c%ba%e8%87%aa%e5%90%af%e5%8a%a8%e8%84%9a%e6%9c%ac/)
- [Mac上如何开机与关机时自动运行Shell脚本](https://www.jianshu.com/p/4945a63b60a4)

#### 通过 shell 脚本自启动服务

通过脚本校验服务是否连接成功

```shell
#!/bin/bash

function network()
{
    #超时时间
    local timeout=120

    #目标网站
    local target=http://0.0.0.0:4000/

	echo "check ${target}"

    #获取响应状态码
    local ret_code=`curl -I -s --connect-timeout ${timeout} ${target} -w %{http_code} | tail -n1`

    if [ "x$ret_code" == "x200" ]; then
        #网络畅通
        return 1
    else
        #网络不畅通
        return 0
    fi

    return 0
}

while [[ true ]]; do
	echo 'connecting...'
	if network == 0
	then
		echo "unpub service start fail..."
	    flutter pub global run 'unpub:unpub' --database 'mongodb://localhost:27017/dart_pub'
	else
	    echo "unpub service start success..."
	    exit 0
	fi	
	sleep 15
done
```

### 安装遇到的问题

####  ['String?' is nullable and 'Object' isn't.](https://github.com/bytedance/unpub/issues/67)

修改源码

![images](/images/2022/Flutter搭建私有Pub仓库Docker部署/9.png)

#### mongodb 启动失败

- [centos mongodb启动失败](https://blog.csdn.net/youif/article/details/105850300)
- [系统重启后Mongo服务启动失败](https://www.jianshu.com/p/06b1be11d00b)

#### 端口被占用问题

```shell
# 排查占用端口
sudo lsof -iTCP -sTCP:LISTEN -n -P
# 删掉端口
sudo kill 449
```

![images](/images/2022/Flutter搭建私有Pub仓库Docker部署/10.png)

#### 鉴权问题

![images](/images/2022/Flutter搭建私有Pub仓库Docker部署/11.png)

重新安装 mongoDB ，去掉管理员的账户密码即可

## 发布私有 package 到 unpub 平台
### 跳过谷歌验证

- 下载项目：[https://github.com/ameryzhu/pub](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fameryzhu%2Fpub)

```shell
# 根目录执行
flutter pub get
# 继续执行
dart --snapshot=pub.dart.snapshot bin/pub.dart 
```

- 生成 `pub.dart.snapshot` 文件

![images](/images/2022/Flutter搭建私有Pub仓库Docker部署/12.png)

- 复制之后放入 `flutter/bin/cache/dart-sdk/bin/snapshots/` 目录下
- 如果是 flutter 直接安装则放入 `flutter/bin/cache` 目录下

### 发布 Pub 私有包

📢 注意 `http://192.168.x.xx:4000/` 是自己的 pub 私有包服务 ip 地址，否则会上传失败

#### 创建 dev package 包

```shell
# 创建一个 package
flutter create --template=package zq_log
# 创建 example
flutter create example
```

#### 代码推 GitLab 私仓库

![images](/images/2022/Flutter搭建私有Pub仓库Docker部署/13.png)

#### 配置 yaml 文件信息

```shell
name: zq_log
description: A flutter log tool for developing.
version: 0.0.2
homepage: https://code.addcn.com/tw591fluttermodules/zq_log
publish_to: http://192.168.x.xx:4000/
```

#### 检测命令

```shell
flutter packages pub publish --dry-run
```

#### 发布包到私有包管理平台

```shell
flutter packages pub publish --server=http://192.168.x.xx:4000/
```
#### 发布成功

![images](/images/2022/Flutter搭建私有Pub仓库Docker部署/14.png)

![images](/images/2022/Flutter搭建私有Pub仓库Docker部署/15.png)

#### 更新 yaml 信息执行 `pub get`
```shell
  zq_log:
    version: ^0.0.1
    hosted:
      name: zq_log
      url: http://192.168.x.xx:4000/
```

## `通过 Docker 方便大家部署`
为了方便大家移植部署，这边将 `unpub` 打包成 `docker` 镜像环境 

### 安装镜像

首先拉取 [GitHub 地址](https://github.com/zeqinjie/unpub-2.0.0-docker)  代码，安装 [docker](https://www.docker.com/get-started) 环境,  然后执行下面命令即可

```shell
# 先安装 docker 环境启动后， 在当前 docker-compose.yml 文件下执行下面命令即可
docker compose up -d 
```

### 安装运行成功如下

安装成功

![images](/images/2022/Flutter搭建私有Pub仓库Docker部署/16.png)

通过 `docker ps -a` 命令查看运行中容器

![images](/images/2022/Flutter搭建私有Pub仓库Docker部署/17.png)

## 參考

### 私有库搭建

- [使用 unpub 搭建私有pub服务器](https://www.jianshu.com/p/8d75ee4d06f7)
- [Flutter 发布package到公有或私有pub](https://www.jianshu.com/p/6ef0159af4c7)
- [私有仓库 web 平台](https://github.com/bytedance/unpub)
- [Flutter Pub私有仓库搭建及使用](https://blog.csdn.net/blog_jihq/article/details/115380948)
- [Flutter发布Package(Pub.dev或私有Pub仓库)](https://www.jianshu.com/p/5c3721efc6f9)
- [Dart私有仓库-私服搭建](https://zhuanlan.zhihu.com/p/433169569)
- [Dart依赖和搭建Flutter-or-Dart简易私人仓库](https://kahnsen.github.io/kahnblog/2018/12/28/Dart%E4%BE%9D%E8%B5%96%E5%92%8C%E6%90%AD%E5%BB%BAFlutter-or-Dart%E7%AE%80%E6%98%93%E7%A7%81%E4%BA%BA%E4%BB%93%E5%BA%93/)
- [https://medium.com/dartlang/hosting-a-private-dart-package-repository-774c3c51dff9](https://medium.com/dartlang/hosting-a-private-dart-package-repository-774c3c51dff9)
- [https://dart.dev/tools/pub/custom-package-repositories](https://dart.dev/tools/pub/custom-package-repositories)

### 遇到问题

- [brew update 更新时 shallow clone](https://zhuanlan.zhihu.com/p/351199589)
- [使用brew services管理服务](https://www.bbsmax.com/A/A7zgQjKK54/)
- [MongoDB 常见问题 - 解决 brew services list 查看 MongoDB 服务 status 显示 error 的问题](https://blog.csdn.net/qq_33801641/article/details/117408923)
- ['String?' is nullable and 'Object' isn't.](https://github.com/bytedance/unpub/issues/67)
- [No active package dartdoc Flutter](https://stackoverflow.com/questions/70318598/no-active-package-dartdoc-flutter)
- [Null check operator used on a null value](https://stackoverflow.com/questions/64278595/null-check-operator-used-on-a-null-value)

### Docker 

- [Docker-开机自启&&容器自启动](https://blog.csdn.net/u014565127/article/details/120718897)
- [Docker-从入门到实践](https://yeasy.gitbook.io/docker_practice/basic_concept)


