---
title: 'Crash on dyld: Library not loaded: @rpath/App.framework/App'
date: 2022-01-25 11:24:29
categories: "Flutter"
tags:
  - Flutter
---


# 记录一次找不到 App.framework/App 的 crash

## Crash Info

最近做 flutter 混编的项目中。我和个别同事的启动 app 也崩溃，发现 xcode 13 下异常错误如下
```
dyld: Library not loaded: @rpath/App.framework/App

Referenced from: /private/var/containers/Bundle/Application/C673170A-6497-4467-985D-6FE61DB09EA7/house591.app/house591

Reason: image not found

dyld: launch, loading dependent libraries

DYLD_LIBRARY_PATH=/usr/lib/system/introspection

DYLD_PRINT_STATISTICS=1

DYLD_INSERT_LIBRARIES=/Developer/usr/lib/libBacktraceRecording.dylib:/Developer/usr/lib/libMainThreadChecker.dylib:/usr/lib/libMTLCapture.dylib:/Developer/Library/PrivateFrameworks/DTDDISupport.framework/libViewDebuggerSupport.dylib

DYLD_PRINT_TO_STDERR=YES

Message from debugger: Terminated due to signal 6
```

## 我自己 xcode Info
![](/images/2022/flutter-crash/1.png)

## 解决

- 通过调试发现主要原因是 App.framework 这个库 pod install 后路径找不到，手动补充了路径 ` install_framework "${PODS_ROOT}/../../../tw591flutter/.ios/Flutter/App.framework" ` 到`Pods-xxx-frameworks ` 下即可正常编译运行。

![](/images/2022/flutter-crash/2.png)

- 当然这样不是最好的解决方案，毕竟每次 install 都会还原，后来搜索了 [flutter issue](https://github.com/flutter/flutter/issues/92896#issuecomment-994476030) 发现有这个问题。解决方法是更新 ruby-macho，执行命令 `sudo gem update ruby-macho` 。
- 补充细节，同事执行了 `sudo gem update ruby-macho` 后升级到 3.0.0 版本发现 Cocopods 报错，说当前版本不支持使用 3.0.0 版本，后来调整为 2.5.0 版本即可。


