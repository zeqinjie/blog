---
title: Flutter Widget 组件的截图
date: 2023-07-23 21:56:32
categories: "Flutter"
tags:
  - Flutter
---

## 前言

最近需要对 Widget 组件做截图~

## 一般组件

### 普通组件

如果是 Flutter 提供的组件，而不是 platform 一般可以使用 `RepaintBoundary` 进行截图。当然也可以使用第三方提供插件 [screenshot](https://pub-web.flutter-io.cn/packages/screenshot)

-   代码示例如下

```
import 'dart:ui' as ui;
import 'dart:async';

import 'package:flutter/material.dart';
import 'package:flutter/rendering.dart';

typedef TWCaptureImage = ui.Image;

class TWCaptureImageView extends StatelessWidget {
  final TWCaptureImage? image;
  final BoxFit? fit;

  const TWCaptureImageView({
    super.key,
    this.fit,
    this.image,
  });

  @override
  Widget build(BuildContext context) {
    return Center(
      child: RawImage(
        fit: fit,
        image: image,
      ),
    );
  }
}

class TWCaptureView extends StatelessWidget {
  final GlobalKey globalKey;
  final Widget child;

  const TWCaptureView({
    super.key,
    required this.child,
    required this.globalKey,
  });

  @override
  Widget build(BuildContext context) {
    return RepaintBoundary(
      key: globalKey,
      child: child,
    );
  }
}

class TWCaptureImageTool {
  static Future<TWCaptureImage?> captureImage(GlobalKey globalKey) async {
    try {
      RenderRepaintBoundary boundary =
          globalKey.currentContext!.findRenderObject() as RenderRepaintBoundary;
      ui.Image image = await boundary.toImage();
      return image;
    } catch (e) {
      print("Error TWCaptureTool captureImage: $e");
      return null;
    }
  }
}
```

### 视频组件

如果是视频播放器，只能使用 [video_thumbnail](https://pub-web.flutter-io.cn/packages/video_thumbnail) 插件，因为涉及到 platform view 原因：[issue](https://github.com/flutter/flutter/issues/25306)

```
import 'dart:typed_data';
import 'package:video_thumbnail/video_thumbnail.dart';
import 'dart:async';


class TWCaptureVideoTool {
  static Future<Uint8List?> captureImage({
    required String path,
    ImageFormat imageFormat = ImageFormat.PNG,
    int maxHeight = 0,
    int maxWidth = 0,
    int timeMs = 0,
    int quality = 25,
  }) async {
    try {
      final uint8list = await VideoThumbnail.thumbnailData(
        video: path,
        imageFormat: imageFormat,
        maxHeight: maxHeight,
        maxWidth: maxWidth,
        timeMs: timeMs,
        quality: quality,
      );
      return uint8list;
    } catch (e) {
      print("Error TWCaptureVideoTool captureImage: $e");
      return null;
    }
  }
}
```

### Google Map

则使用插件 GoogleMapController 的 takeSnapshot 方法即可

```
  Future<Uint8List?> takeSnapshot() {
    return GoogleMapsFlutterPlatform.instance.takeSnapshot(mapId: mapId);
  }
```

## 参考

-   <https://github.com/flutter/flutter/issues/25306>
-   <https://pub-web.flutter-io.cn/packages/video_thumbnail>
-   <https://pub-web.flutter-io.cn/packages/screenshot>
