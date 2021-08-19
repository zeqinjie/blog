---
title: 01初识flutter和环境搭建
date: 2021-07-21 19:03:02
tags:
  - Flutter
---

## 跨平台解决方案
![](/images/2021/01初识flutter和环境搭建/1.png)
### RN

- RN的本质是通过JavaScript VM调用远程接口，通信相对比较低效，而且框架本身不负责渲染，而是是间接通过原生进行渲染的
### Flutter
![](/images/2021/01初识flutter和环境搭建/2.png)

- Flutter利用Skia绘图引擎，直接通过CPU、GPU进行绘制，不需要依赖任何原生的控件（后面有原理讲解）
- Android操作系统中，我们编写的原生控件实际上也是依赖于Skia进行绘制，所以flutter在某些Android操作系统上甚至还要高于原生（因为原生Android中的Skia必须随着操作系统进行更新，而Flutter SDK中总是保持最新的）
- 而类似于RN的框架，必须通过某些桥接的方式先转成原生进行调用，之后再进行渲染。
#### 
### RN 与 Flutter 区别
> - React Native 之类的框架，只是通过 JavaScript 虚拟机扩展调用系统组件，由 Android 和 iOS 系统进行组件的渲染；
> - Flutter 是自己完成了组件渲染的闭环。

### 
## Flutter 介绍


### Flutter 渲染机制
![](/images/2021/01初识flutter和环境搭建/3.png)

> 在计算机系统中，CPU、GPU和显示器以一种特定的方式协作：
> - CPU将计算好的显示内容提交给 GPU；
> - GPU渲染后放入帧缓冲区；
> - 视频控制器按照 VSync信号从帧缓冲区取帧数据传递给显示器显示；
> - 双重缓存，三重缓存机制 

![](/images/2021/01初识flutter和环境搭建/4.png)

- GPU将信号同步到 UI 线程
- UI 线程用Dart来构建图层树
- 图层树在GPU 线程进行合成
- 合成后的视图数据提供给Skia 引擎
- Skia 引擎通过OpenGL 或者 Vulkan将显示内容提供给GPU



### 渲染引擎skia

- **Skia**全名**Skia Graphics Library**（SGL）是一个由C++编写的开源图形库，能在低端设备如手机上呈现高质量的2D图形，最初由Skia公司开发，后被Google收购，应用于Android、Google Chrome、Chrome OS等等当中。
- 对于 iOS 平台来说，由于 Skia 是跨平台的，因此它作为 Flutter iOS 渲染引擎被嵌入到 Flutter 的 iOS SDK 中，替代了 iOS 闭源的 Core Graphics/Core Animation/Core Text，这也正是 Flutter iOS SDK 打包的 App 包体积比 Android 要大一些的原因。

​

![](/images/2021/01初识flutter和环境搭建/5.png)
> Flutter 架构图

### Flutter 安装过程
![](/images/2021/01初识flutter和环境搭建/6.png)

- flutter sdk 安装
   - 网站地址：[flutter.dev/docs/develo…](https://flutter.dev/docs/development/tools/sdk/releases?tab=macos)
- 配置环境变量
   - open ~/.bash_profile
   - open ~/.zshrc
   - **注意:** 如果你使用的是zsh，终端启动时 ~/.bash_profile 将不会被加载，解决办法就是修改 ~/.zshrc ，在其中添加：source ~/.bash_profile



​

## 参考

- [学习Flutter从这开始(邂逅Flutter)](http://www.cocoachina.com/articles/34756)
- [Flutter(二)之环境搭建](https://juejin.cn/post/6844903935132581902)

