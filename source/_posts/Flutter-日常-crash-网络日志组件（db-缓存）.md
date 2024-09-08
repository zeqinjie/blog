---
title: Flutter tw_logger 一款支持缓存 db 的日志组件（network、crash、regular）
date: 2024-09-08 16:15:01
categories: "Flutter"
tags:
  - Flutter
---

## 前言
目前我们项目使用的日志比较简单仅支持日志输出，考虑业内有很多开源的日志组件，因此基于 [logger](https://pub.dev/packages/logger) 进行一定的封装缓存相关db 日志，方便查看历史数据，以满足我们的调试目的。

## 特性
- 支持一般的 logger 的输出功能，把日志内容写入 db
- 支持网络请求拦截输出功能，把请求与响应结果写入 db
- 支持异常捕获功能，把错误写入 db
- 实现 UI 界面增删查 & 标签查询记录

## 效果

![](/images/2024/tw_logger/1.gif)

## Use

```dart
import 'dart:async';
import 'dart:math';

import 'package:flutter/material.dart';
import 'package:tw_logger/tw_logger.dart';
import 'package:dio/dio.dart';

void main(List<String> args) async {
  FlutterError.onError = (FlutterErrorDetails details) {
    TWCrashHelper.handleCrashCache(details);
  };
  runZonedGuarded(
    () {
      runApp(const MyApp());
    },
    (error, stacktrace) {
      TWCrashHelper.handleCrashCache(FlutterErrorDetails(
        stack: stacktrace,
        exception: error,
      ));
    },
  );
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'tw_logger',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(
          seedColor: Colors.deepPurple,
        ),
        useMaterial3: true,
      ),
      home: const MyHomePage(title: 'tw_logger'),
    );
  }
}

class MyHomePage extends StatefulWidget {
  const MyHomePage({super.key, required this.title});

  final String title;

  @override
  State<MyHomePage> createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> {
  final dio = Dio();
  late OverlayEntry overlayEntry;
  @override
  void initState() {
    super.initState();
    setUp();
    setLoggerFilter();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        backgroundColor: Theme.of(context).colorScheme.inversePrimary,
        title: Text(widget.title),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Switch(
              value: TWLoggerConfigure().open,
              onChanged: switchOpenChanged,
            ),
            FilledButton(
              onPressed: () {
                log();
              },
              child: const Text('log'),
            ),
            FilledButton(
              onPressed: () {
                log2();
              },
              child: const Text('log2'),
            ),
            FilledButton(
              onPressed: () {
                log3();
              },
              child: const Text('log3'),
            ),
            FilledButton(
              onPressed: () {
                throw Exception('This is a crash test');
              },
              child: const Text('throw error'),
            ),
            FilledButton(
              onPressed: () {
                get1();
              },
              child: const Text('get1'),
            ),
            FilledButton(
              onPressed: () {
                get2();
              },
              child: const Text('get2'),
            ),
            FilledButton(
              onPressed: () {
                delete();
              },
              child: const Text('delete'),
            ),
            FilledButton(
              onPressed: () {
                post();
              },
              child: const Text('post'),
            ),
          ],
        ),
      ),
    );
  }

  void setLoggerFilter() {
    TWLogger().filter = (
      level,
      message,
      error,
      stackTrace,
      time,
    ) {
      return message != 'filter log';
    };
  }

  void setUp() {
    if (TWLoggerConfigure().open) {
      overlayEntry = TWLoggerOverlay.attachTo(context);
    } else {
      overlayEntry.remove();
    }
    TWNetworkSetting.updateInterceptor(dio);
  }

  void switchOpenChanged(bool value) {
    setState(() {
      TWLoggerConfigure().isEnabled = value;
      setUp();
    });
  }

  void log() {
    String randomString = generateRandomString(120);
    TWLogger.log(randomString);
  }

  void log2() {
    String randomString = generateRandomString(120);
    TWLogger.log(
      randomString,
      level: TWLogLevel.error,
    );
  }

  void log3() {
    TWLogger.log('filter log');
  }

  String generateRandomString(int length) {
    const characters =
        'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
    Random random = Random();
    return List.generate(
            length, (index) => characters[random.nextInt(characters.length)])
        .join();
  }

  void get1() {
    dio.get('https://flutter.dev/');
  }

  void get2() {
    dio.get('https://jsonplaceholder.typicode.com/todos');
  }

  void delete() {
    dio.delete(
        'http://ajax.googleapis.com/ajax/services/feed/load?q=FeedName&v=1.0');
  }

  void post() {
    dio.post(
      'https://run.mocky.io/v3/9d059bf9-4660-45f2-925d-ce80ad6c4d15',
      data: <String, dynamic>{
        "id": 1,
        "title": "Hello World",
        "content": "This is a post"
      },
    );
  }
}
```
## 最后
调整后已上传 [pub](https://pub.dev/packages/tw_logger)，使用如下

```yaml
dependencies:
  tw_logger: latest_version
```

tw_logger 地址 [传送门](https://github.com/zeqinjie/tw_logger)， 如果你觉得对你有帮助，请不吝给个 Star 。如果有问题，也请大佬们指点 ~ 感谢！

  
