---
title: Docker 基础
date: 2022-05-15 22:41:59
categories: "Docker"
tags:
	- Docker
---

## Docker 
### 什么是 docker ？
个划时代的开源项目，它彻底释放了计算虚拟化的威力，极大提高了应用的维护效率，降低了云计算应用开发的成本！使用 Docker，可以让应用的部署、测试和分发都变得前所未有的高效和轻松！
### Docker 与 虚拟机

![image](/images/2022/Docker/1.png)

## Docker 安装
### 方式一: Homebrew
```shell
brew cask install docker 
```
### 方式二: 桌面程序
访问官方地址[ 传送门](https://www.docker.com/get-started)，在 Docker Desktop 下选择你当前系统对应的软件进行下载安装

![image](/images/2022/Docker/2.png)

### 安装后 Docker 命令查看

![image](/images/2022/Docker/3.png)

```shell
# docker 帮助信息
docker help 

# 查看版本
docker -v 
# 查看 Docker 信息
docker info 
```
## Docker 基本概念
### 关系图

![image](/images/2022/Docker/4.png)

```shell
注册服务器（Registry）
├─Repository（共有仓库）
│  ├─镜像1
│  │  ├─容器1
│  │  ├─容器2
│  ├─镜像2
│  │  ├─容器1
│  │  ├─容器...
│  ├─镜像...
├─Repository（私有仓库）
│  ├─镜像3
│  │  ├─容器...
│  ├─镜像4
│  │  ├─容器...
│  ├─镜像...
```
### 概述

- 注册服务器（Registry）
   - 管理镜像仓库的具体服务器，每个服务器上可以有多个仓库，而每个仓库下面有多个镜像
- 镜像仓库（Repository）
   - 集中存放镜像的地方，仓库分为[公有仓库](https://hub.docker.com/)与私有仓库
- 镜像（Image）
   - 一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）
   - 包含了各种环境或者服务(tomcat)一个模板
- 容器（Container）
   - 容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等
   - 镜像和容器的关系，就像是面向对象程序设计中的类和实例一样，镜像是静态的定义
   - 容器的实质是进程

## 参考

- [深入理解docker的镜像和容器](https://www.jianshu.com/p/c7cc19ef8cb2)
- [https://docs.docker.com/go/guides](https://docs.docker.com/go/guides)
- [Docker — 从入门到实践](https://vuepress.mirror.docker-practice.com/)
- [Docker - 安装、加速和基本使用](https://juejin.cn/post/6921330672955031560)
- [https://docs.docker.com/compose/](https://docs.docker.com/compose/)
