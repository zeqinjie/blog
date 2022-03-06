---
title: Jenkins-fastlane自动构建
date: 2020-04-28 19:03:02
categories: "iOS"
tags:
  - CI/CD
---

# Jenkins自动构建 去年基于Jenkins平台搭建的自动构建流程图

![](/images/2020/Jenkins+fastlane自动构建/1.png)

![](/images/2020/Jenkins+fastlane自动构建/2.png)
## 目录

- Jenkins 作用及特点
- 怎么搭建Jenkins平台
- Jenkins+fastlane的持续集成
- 怎么使用Jenkins触发自动打包构建
- Jenkins构建遇到问题整理及解决
- 目前功能与未来的拓展

## 作用：

Jenkins是开源CI&CD软件领导者， 提供超过1000个插件来支持构建、部署、自动化， 满足任何项目的需要。
## 特点：
- 持续集成和持续交付
- 简易安装，配置简单
- 支持插件
- 扩展，分布式
## 怎么搭建Jenkins平台
- 安装Java环境
    - 安装Java环境，目前jenkins只支持jdk8 
- 安装 Jenkins
    - 先确保已安装Homebrew
    - 命令安装 brew install jenkins
    - 设置开机自启动
- 启动 Jenkins http://localhost:8080
- 安装插件
    - Keychains and Provisioning Profiles - Management（管理本地的keychain和iOS证书的插件）
    - Xcode integration （用于xcode构建）


## Jenkins+fastlane持续集成
- Fastlane是一整套的客户端CICD工具集合。Fastlane可以非常快速简单的搭建一个自动化发布服务，并且支持Android，iOS，MacOS。
- Fastlane命令执行的底层并不是自己实现的，而是调用其他的插件或者工具执行的。比如说打包，Fastlane中的gym工具只是xcodebuild工具的一个封装，调用的其实还是xcodebuild中的打包命令。
- Fastlane本身没有一套特殊语法，使用的Ruby语言。
- Fastlane的插件工具叫做action，每一个action都对应一个具体的功能

## 目前功能与未来的拓展
- 目前集成功能：
    - 测试包部署
    - 线上包部署
    - 静态图片压缩
    - OCLint语法检测
- 未来优化点
    - 推送代码整合一套构建流程功能
    - 简繁体转换集成
    - SwiftLint语法检测集成
    - ...

![](/images/2020/Jenkins+fastlane自动构建/3.png)

### 具体配置待补充 , 未完待续..
