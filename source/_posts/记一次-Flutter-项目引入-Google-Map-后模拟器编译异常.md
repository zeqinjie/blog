---
title: 记一次 Flutter 项目引入 Google Map 后模拟器编译异常
date: 2022-04-29 23:41:07
categories: "Flutter"
tags:
  - Flutter
---

## 记录一次 Flutter 项目模拟器编译失败的问题
最近 Flutter 项目引入的 google 地图，发现在模拟器下不能正常编译，但真机是正常编译运行。
错误信息 `xxx/Frameworks/GoogleMaps.framework/GoogleMaps' for architecture arm64`

## 截图如下

![](/images/2022/flutter-crash/3.png)

- 问题的初步是发生在 M1 电脑模拟器编译运行会报错误信息，如上截图。

- Google 一番发现有提供参考解决方案 [传送门](https://github.com/flutter/flutter/issues/94914#issuecomment-992898782), 所以我尝试在 Podfile 文件补充如下脚本。install 后发现问题依然报错。

```ruby
 post_install do |installer|
   installer.pods_project.targets.each do |target|
     flutter_additional_ios_build_settings(target)
+    target.build_configurations.each do |config|
+      config.build_settings["EXCLUDED_ARCHS[sdk=iphonesimulator*]"] = "arm64 i386"
+   end
   end
 end
```

- 后来回想了下每个 Flutter iOS 项目配置可以在 `Generated.xcconfig` 调整配置, 因此补充 `arm64` 后重新 install 在 M1 模拟器下能正常编译运行。
```
EXCLUDED_ARCHS[sdk=iphonesimulator*]=i386 arm64 
```

## 截图如下

![](/images/2022/flutter-crash/4.png)

