---
title: 01初识flutter和环境搭建
date: 2021-07-21 19:03:02
tags:
  - Flutter
---

## 跨平台解决方案
![image.png](https://cdn.nlark.com/yuque/0/2021/png/2127799/1627794173602-e5e50dc9-71b3-4450-b89e-d327833cc6f0.png#clientId=ufd473d58-39ca-4&from=paste&height=231&id=aVukp&margin=%5Bobject%20Object%5D&name=image.png&originHeight=461&originWidth=658&originalType=binary&ratio=1&size=188413&status=done&style=none&taskId=uaeecbea0-80b1-4bdb-b74a-284186b5fe4&width=329)
### RN

- RN的本质是通过JavaScript VM调用远程接口，通信相对比较低效，而且框架本身不负责渲染，而是是间接通过原生进行渲染的
### Flutter
![image.png](https://cdn.nlark.com/yuque/0/2021/png/2127799/1627802284865-cb7d9609-03c0-4d84-b470-fb17412ee729.png#clientId=ufd473d58-39ca-4&from=paste&height=222&id=u1a56bc36&margin=%5Bobject%20Object%5D&name=image.png&originHeight=444&originWidth=1154&originalType=binary&ratio=1&size=240948&status=done&style=none&taskId=ue6845513-214c-46da-8c45-041f250ad03&width=577)

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
![image.png](https://cdn.nlark.com/yuque/0/2021/png/2127799/1627799885961-00a9b2ba-ce20-444e-a312-f1bf498768a0.png#clientId=ufd473d58-39ca-4&from=paste&height=209&id=ud138c897&margin=%5Bobject%20Object%5D&name=image.png&originHeight=417&originWidth=702&originalType=binary&ratio=1&size=107268&status=done&style=none&taskId=uc6fdded5-dd9d-4349-80ff-a11043300f9&width=351)
> 在计算机系统中，CPU、GPU和显示器以一种特定的方式协作：
> - CPU将计算好的显示内容提交给 GPU；
> - GPU渲染后放入帧缓冲区；
> - 视频控制器按照 VSync信号从帧缓冲区取帧数据传递给显示器显示；
> - 双重缓存，三重缓存机制 

![image.png](https://cdn.nlark.com/yuque/0/2021/png/2127799/1627798650439-abb61f81-8165-4e70-849f-300eff6c5034.png#clientId=ufd473d58-39ca-4&from=paste&height=260&id=LMEwC&margin=%5Bobject%20Object%5D&name=image.png&originHeight=519&originWidth=692&originalType=binary&ratio=1&size=86122&status=done&style=none&taskId=u2bd36dbd-37c0-4a54-960a-effd871852e&width=346)

- GPU将信号同步到 UI 线程
- UI 线程用Dart来构建图层树
- 图层树在GPU 线程进行合成
- 合成后的视图数据提供给Skia 引擎
- Skia 引擎通过OpenGL 或者 Vulkan将显示内容提供给GPU



### 渲染引擎skia

- **Skia**全名**Skia Graphics Library**（SGL）是一个由C++编写的开源图形库，能在低端设备如手机上呈现高质量的2D图形，最初由Skia公司开发，后被Google收购，应用于Android、Google Chrome、Chrome OS等等当中。
- 对于 iOS 平台来说，由于 Skia 是跨平台的，因此它作为 Flutter iOS 渲染引擎被嵌入到 Flutter 的 iOS SDK 中，替代了 iOS 闭源的 Core Graphics/Core Animation/Core Text，这也正是 Flutter iOS SDK 打包的 App 包体积比 Android 要大一些的原因。

​

![image.png](https://cdn.nlark.com/yuque/0/2021/png/2127799/1627800173552-e8d54688-da5e-4d98-aec7-a67932f57ffc.png#clientId=ufd473d58-39ca-4&from=paste&height=173&id=uec62ba77&margin=%5Bobject%20Object%5D&name=image.png&originHeight=345&originWidth=655&originalType=binary&ratio=1&size=186036&status=done&style=none&taskId=ubafc4be8-83a7-45cd-827d-7fa9120a28f&width=327.5)
> Flutter 架构图

### Flutter 安装过程
![截屏2021-08-01 15.13.31.png](https://cdn.nlark.com/yuque/0/2021/png/2127799/1627802049903-3a39d1ae-0718-469a-9641-ffd713145e1f.png#clientId=ufd473d58-39ca-4&from=paste&height=625&id=u6dcf9b1c&margin=%5Bobject%20Object%5D&name=%E6%88%AA%E5%B1%8F2021-08-01%2015.13.31.png&originHeight=1250&originWidth=2368&originalType=binary&ratio=1&size=3947350&status=done&style=none&taskId=uba3f6687-6a58-4afd-ad86-c64a89f5780&width=1184)

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

[

](http://www.cocoachina.com/articles/34756)
截图 ： Shift + Command + 4
