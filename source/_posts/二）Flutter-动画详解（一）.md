---
title: 二）Flutter 动画详解（一）
date: 2022-03-28 22:25:39
categories: "Flutter"
tags:
  - Flutter
---


## 前言
最近翻看王叔的动画视频教程 [传送门](https://www.bilibili.com/video/BV1JZ4y1p7NG/?spm_id_from=pageDriver)，实话说他的讲解细节很到位。通过看他的 Flutter 动画详解篇章，做了下笔记。

## 如何选择合适的动画控件

![image.png](https://cdn.nlark.com/yuque/0/2022/png/2127799/1647183239924-5f5bcfcb-5ac1-45f0-baee-d1efb0653c8d.png#clientId=u28daff04-a71e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=582&id=u5f995e8a&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1164&originWidth=2422&originalType=binary&ratio=1&rotation=0&showTitle=false&size=2853612&status=done&style=none&taskId=ud4addf3b-f058-44cf-839f-2b694ee04be&title=&width=1211)

## 隐式动画

### Animated+`WidgetName`

![iShot2022-03-27 21.56.19.gif](https://cdn.nlark.com/yuque/0/2022/gif/2127799/1648389416073-06dd80ec-70fb-4d39-bae2-235993066d32.gif#clientId=uea5ead90-a0aa-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=330&id=u0b4731a3&margin=%5Bobject%20Object%5D&name=iShot2022-03-27%2021.56.19.gif&originHeight=898&originWidth=694&originalType=binary&ratio=1&rotation=0&showTitle=false&size=2864325&status=done&style=none&taskId=udb08686c-756f-461b-94a4-bd72e845457&title=&width=255)

```dart
Widget _buildAnimatedContainer() {
  return Center(
    child: AnimatedContainer(
      alignment: Alignment.center,
      duration: const Duration(milliseconds: 1000),
      height: _height,
      width: 200,
      decoration: BoxDecoration(
        boxShadow: const [BoxShadow(spreadRadius: 25, blurRadius: 25)],
        borderRadius: BorderRadius.circular(150),
        gradient: const LinearGradient(
          begin: Alignment.bottomCenter,
          end: Alignment.topCenter,
          colors: [Colors.blue, Colors.white],
          stops: [0.2, 0.3],
        ),
      ),
      child: AnimatedSwitcher(
        duration: const Duration(milliseconds: 1000),
        transitionBuilder: (child, animation) {
          return ScaleTransition(
            scale: animation,
            child: FadeTransition(
              opacity: animation,
              child: child,
            ),
          );
        },
        child: _height >= 500
            ? null
            : Center(
                key: ValueKey(_height),
                child: Text(
                  "zhengzeqin $_height",
                  style: TextStyle(
                    fontSize: 18,
                    color: Colors.black,
                  ),
                ),
              ),
      ),
    ),
  );
}
```

#### AnimatedContainer

- 只有 AnimatedContainer 下的属性参与到动画变化中， child 的 widget 的属性变化则无效

#### AnimatedSwitcher

- AnimatedSwitcher 组件用来执行动画组件的切换功能，如 A 的缩小 B 的放大，当 child 组件的 key 或者组件类型改变会引起动画（参考 key 使用篇章）
- AnimatedSwitcher child 如果等于 null 会隐私消失， 默认 FadeTransition 渐变动画
- transitionBuilder 的是支持组合多个动画组件如下 ScaleTransition 嵌套  FadeTransition (类似AnimatedCrossFade 渐入渐出)

#### 相关组件有

- AnimatedSwitcher
- AnimationPadding
- AnimatedOpacity
- AnimatedAlign
- ...
```dart
AnimatedSwitcher(
  duration: const Duration(milliseconds: 1000),
  transitionBuilder: (child, animation) {
    return ScaleTransition(
      scale: animation,
      child: FadeTransition(
        opacity: animation,
        child: child,
      ),
    );
  },
  child: _height >= 500 ? null : Center(
    key: ValueKey(_height),
    child: Text(
      "zhengzeqin $_height",
      style: TextStyle(
        fontSize: 18,
        color: Colors.black,
      ),
    ),
  ),
)
```
 
### 隐式动画都有个 curve 曲线

![iShot2022-03-19 21.15.44.gif](https://cdn.nlark.com/yuque/0/2022/gif/2127799/1647695761187-6171a759-89e0-4b02-8d8c-439cc8debbe9.gif#clientId=u361a582e-1c1e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=352&id=u0518b8da&margin=%5Bobject%20Object%5D&name=iShot2022-03-19%2021.15.44.gif&originHeight=986&originWidth=714&originalType=binary&ratio=1&rotation=0&showTitle=false&size=341959&status=done&style=none&taskId=u5078a836-6911-4fa1-a33b-8dcc65f8ee9&title=&width=255)

```dart
/// 更多动画及曲线
/// 隐式动画都有个 curve 曲线 Curves.bounceOut, // 弹性结束
/// 官方文档：https://api.flutter-io.cn/flutter/animation/Curves-class.html   
Widget _buildAnimated() {
  return Center(
    child: AnimatedPadding(
      curve: Curves.bounceOut, // 弹性结束
      duration: Duration(milliseconds: 1000),
      padding: EdgeInsets.only(top: _height),
      child: AnimatedOpacity (
        curve: Curves.bounceInOut, // 弹性开始和结束
        duration: Duration(milliseconds: 2000),
        opacity: _opacity,
        child: Container(
          height: 300,
          width: 300,
          color: Colors.blue,
        ),
      ),
    ),
  );
}
```

### 关键帧动画 Tween

- TweenAnimationBuilder 
- tween: Tween<double>(begin: 0.0, end: 1.0)
   - 注意：关键帧动画是从当前值到目标值变化，比如设置 0 -> 100。 如果当前是 10，则是从 10 -> 100
- 平移：Transform.translate
   - offset: Offset(0,0) -> Offset(-x, -x)。 是中心点到右下
- 缩放：Transform.scale
   - scale: 0.0 -> 1.0
- 旋转：Transform.rotate
   - angle: 0.0 -> 6.28 。 旋转一圈 

![iShot2022-03-19 21.13.33.gif](https://cdn.nlark.com/yuque/0/2022/gif/2127799/1647695651077-dd54cf93-dcb7-466b-854c-39f0a2ab730f.gif#clientId=u361a582e-1c1e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=390&id=u3142c246&margin=%5Bobject%20Object%5D&name=iShot2022-03-19%2021.13.33.gif&originHeight=1138&originWidth=744&originalType=binary&ratio=1&rotation=0&showTitle=false&size=151210&status=done&style=none&taskId=u99ac7ae4-017e-4dd6-a40e-954f0892235&title=&width=255)

```dart
/// between 之间 ，关键帧动画
Widget _buildTweenAnimated() {
  return Center(
    child: TweenAnimationBuilder(
      duration: Duration(seconds: 1),
      builder: (BuildContext context, double value, Widget? child) {
        return Opacity(
          opacity: value,
          child: Container(
            alignment: Alignment.center,
            height: 300,
            width: 300,
            color: Colors.blue,
            child: Transform.rotate(
              angle: value * 6.28, // 6.28 一圈 p 是 3.14 半圈
              child: Text(
                "zhengzeqin",
                style: TextStyle(fontSize: 10 + 10 * value),
              ),
            ),
          ),
        );
      },
      tween: Tween<double>(begin: 0.0, end: 1.0),
    ),
  );
}
```

### 案例：翻转的计数器

- 通过 TweenAnimationBuilder 关键帧动画结合 Positioned 位置移动实现翻转

![iShot2022-03-20 17.41.41.gif](https://cdn.nlark.com/yuque/0/2022/gif/2127799/1647769323235-d0879bb0-a0a9-4349-8507-9fee2906c3e0.gif#clientId=uf85cb564-dc8c-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=264&id=u3e0c0131&margin=%5Bobject%20Object%5D&name=iShot2022-03-20%2017.41.41.gif&originHeight=712&originWidth=688&originalType=binary&ratio=1&rotation=0&showTitle=false&size=212394&status=done&style=none&taskId=u7e3da806-f338-4bfb-b459-bc23c639dc4&title=&width=255)

```dart

/// 封装计数器
class TWAnimatedCounter extends StatelessWidget {
  final Duration? duration;
  final double fontSize;
  final double count;

  const TWAnimatedCounter({
    Key? key,
    this.duration,
    this.fontSize = 100,
    this.count = 0,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return TweenAnimationBuilder(
      duration: duration ?? const Duration(seconds: 1),
      tween: Tween(end: count),
      builder: (BuildContext context, double value, Widget? child) {
        final whole = value ~/ 1; // 取整数
        final decimal = value - whole; // 取小数
        return Stack(
          alignment: Alignment.center,
          children: [
            Positioned(
              top: -fontSize * decimal, // 0 -> -100
              child: Opacity(
                opacity: 1.0 - decimal, // 1.0 -> 0.0
                child: Text(
                  "$whole",
                  style: TextStyle(
                    fontSize: fontSize,
                  ),
                ),
              ),
            ),
            Positioned(
              top: fontSize - decimal * fontSize, // 100 -> 0
              child: Opacity(
                opacity: decimal, // 0 -> 1.0
                child: Text(
                  "${whole + 1}",
                  style: TextStyle(
                    fontSize: fontSize,
                  ),
                ),
              ),
            ),
          ],
        );
      },
    );
  }
}
```

## 显示动画
### `WidgetName` + Transition
#### SingleTickerProviderStateMixin

- Ticker 是每次刷新时候。 60hz 每秒触发 60 次
#### AnimationController

- reset: 重置恢复   
- stop: 暂停
- forward: 执行一次 
- repeat: 重复执行
- reverse: 倒序
- lowerBound -> upperBound 默认是 0 -> 1
#### 相关组件有

- RotationTransition
- FadeTransition
- ScaleTransition
- ...

```dart
/// 继承 Animation<double>
class AnimationController extends Animation<double> {
  ```
}

_controller = AnimationController(
  // upperBound: 10,
  // lowerBound: 1,
  vsync: this, // 垂直同步
  duration: const Duration(milliseconds: 1000),
);
_controller.addListener(() {
    print("${_controller.value}");
});


/// 调用 setState
setState(() {
  if (_isLoading) {
    _controller.stop(); // reset: 重置恢复   stop: 暂停
  } else {
    _controller.repeat(reverse: true); // forward: 执行一次  repeat: 重复执行
  }
  _isLoading = !_isLoading;
});
```

![iShot2022-03-20 22.33.34.gif](https://cdn.nlark.com/yuque/0/2022/gif/2127799/1647786835195-42e8b090-56c9-460c-a5b2-2e2781e4ab17.gif#clientId=uae25af0d-109b-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=287&id=u9d43bd03&margin=%5Bobject%20Object%5D&name=iShot2022-03-20%2022.33.34.gif&originHeight=574&originWidth=702&originalType=binary&ratio=1&rotation=0&showTitle=false&size=310738&status=done&style=none&taskId=u7a6e7a11-7294-4c42-a4b1-5c28d8a64ac&title=&width=351)

```dart
/// 旋转动画
Widget _buildRotationTransition() {
  return Center(
    child: RotationTransition(
      turns: _controller,
      child: const Icon(
        Icons.refresh,
        size: 100,
      ),
    ),
  );
}

/// 透明度渐变动画
Widget _buildFadeTransition() {
  return Center(
    child: FadeTransition(
      opacity: _controller, // 透明度
      child: Container(
        height: 100,
        width: 100,
        color: Colors.blue,
      ),
    ),
  );
}

/// 缩放动画
Widget _buildScaleTransition() {
  return Center(
    child: ScaleTransition(
      scale: _controller, // 缩放倍数
      child: Container(
        height: 100,
        width: 100,
        color: Colors.yellow,
      ),
    ),
  );
}
```

### 关键帧动画 Tween

- 通过 _controller.drive(Tween(...)) 添加 Tween 关键帧开始与结束值。
- 等价的写法 Tween(...).animate(_controller)
- chain 叠加一个 CurveTween(curve: Interval(0.8, 1.0))  即表示 80% 之前的时间动画不执行，剩余 20% 时间完成动画 
- chain 叠加一个 CurveTween(curve: Curves._bounceInOut_)  即表示加了个弹性入出的动效
- 注意动画的叠加类似计算是函数式的： g(f(x))
- 在 Slide 平移动画中，offset 是以 （0，0）原点。偏移 0.5 即是 0.5 倍数

```dart
Widget _buildTweenAnimation() {
  return SlideTransition(
    child: Container(
      height: 100,
      width: 100,
      color: Colors.red,
    ),
    position: Tween(
      begin: const Offset(0, 0),
      end: const Offset(0.5, 1),
    )
        .chain(
          CurveTween(
            curve: const Interval(0.8, 1.0) , // Curves.bounceOut
          ),
        )
        .animate(_controller),
    // position: _controller.drive(
    //   Tween(
    //     begin: const Offset(0, 0),
    //     end: const Offset(0.5, 1),
    //   ),
    // ),
  );
}
```

### 案例：交错动画
![iShot2022-03-23 23.12.23.gif](https://cdn.nlark.com/yuque/0/2022/gif/2127799/1648048356035-6f44e8c5-8488-47cf-b3d2-26e08d396096.gif#clientId=uc5def3e2-247c-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=550&id=ub8ec92e8&margin=%5Bobject%20Object%5D&name=iShot2022-03-23%2023.12.23.gif&originHeight=1100&originWidth=612&originalType=binary&ratio=1&rotation=0&showTitle=false&size=178138&status=done&style=none&taskId=uc24df9a1-22e0-42ae-888a-454adc80973&title=&width=306)

```dart
/// 交错动画
class TWSlidingBox extends StatelessWidget {
  final AnimationController controller;
  final Color color;
  final Interval interval;

  const TWSlidingBox({
    Key? key,
    required this.controller,
    required this.color,
    required this.interval,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return SlideTransition(
      child: Container(
        width: 300,
        height: 100,
        color: color,
      ),
      position: Tween(
        begin: Offset.zero,
        end: const Offset(0.1, 0),
      )
          .chain(
            CurveTween(curve: interval),
          )
          .chain(
            CurveTween(curve: Curves.bounceInOut),   // g(f(x))
          )
          .animate(controller),
    );
  }
}
```

### 自定义动画

```dart
/// 自定义动画
  Widget _buildCustomAnimation() {
    final Animation opacityAnimation = Tween(begin: 0.5, end: 0.8).animate(_controller);
    final Animation heightAnimation = Tween(begin: 100.0, end: 200.0).animate(_controller);
    return AnimatedBuilder(
      animation: _controller,
      child: const Text("Custom Animation",textAlign: TextAlign.center,), // 这个 child 及传给 builder 的，从性能考虑
      builder: (BuildContext context, Widget? child) {
        return Opacity(
          opacity: opacityAnimation.value,
          child: Container(
            alignment: Alignment.center,
            color: Colors.blue,
            width: 100,
            height: heightAnimation.value,
            child: child,
          ),
        );
      },
    );
  }
```

### 案例：478 呼吸法的动画 

- 注意如果使用多个 AnimationController 则需要混入 TickerProviderStateMixin，单个下是SingleTickerProviderStateMixin
- 通过 await Future.delayed 来让动画处于等待效果

![iShot2022-03-27 22.58.22.gif](https://cdn.nlark.com/yuque/0/2022/gif/2127799/1648393142810-030e5885-9af4-4a25-87a2-841293da7ebf.gif#clientId=uea5ead90-a0aa-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=306&id=uead61ff0&margin=%5Bobject%20Object%5D&name=iShot2022-03-27%2022.58.22.gif&originHeight=612&originWidth=696&originalType=binary&ratio=1&rotation=0&showTitle=false&size=2018192&status=done&style=none&taskId=ucd7c77dc-8d5f-47da-a482-4e0acdb0061&title=&width=348)

```dart
void handleBreatedAnimation() async {
  _expansionController.duration = const Duration(seconds: 4);
  _expansionController.forward();
  await Future.delayed(const Duration(seconds: 4));

  _opacityController.duration = const Duration(milliseconds: 1750);
  _opacityController.repeat(reverse: true);
  await Future.delayed(const Duration(seconds: 7));
  _opacityController.reset();

  _expansionController.duration = const Duration(seconds: 8);
  _expansionController.reverse();
}

Widget _buildBreatedAnimation() {
  // _controller.duration = const Duration(seconds: 20);
  // Animation animation1 = Tween(begin: 0.0, end: 1.0)
  //     .chain(CurveTween(curve: const Interval(0.1, 0.2)))
  //     .animate(_controller);
  // Animation animation2 = Tween(begin: 1.0, end: 0.0)
  //     .chain(CurveTween(curve: const Interval(0.4, 0.95)))
  //     .animate(_controller);
  return FadeTransition(
    opacity: Tween(begin: 0.5, end: 1.0).animate(_opacityController),
    child: AnimatedBuilder(
      // animation: _controller,
      animation: _expansionController,
      builder: (BuildContext context, Widget? child) {
        return Container(
          width: 300,
          height: 300,
          decoration: BoxDecoration(
            shape: BoxShape.circle,
            color: Colors.blue,
            gradient: RadialGradient(
              colors: [
                Colors.blue[600]!,
                Colors.blue[100]!,
              ],
              stops: [
                _expansionController.value,
                _expansionController.value + 0.1,
              ],
            ),
          ),
        );
      },
    ),
  );
}
```

## 总结

![](/images/2022/二）Flutter-动画详解（一）/动画总结.png)

[动画.xmind](https://www.yuque.com/attachments/yuque/0/2022/xmind/2127799/1648477075100-30137c3d-599a-49ac-944a-adb6f4669541.xmind)


## 参考

- [demo](https://github.com/zeqinjie/flutter_demo)
- [王叔不秃-动画教程](https://www.bilibili.com/video/BV1JZ4y1p7NG/?spm_id_from=pageDriver)
- [官方 Curves 曲率](https://api.flutter-io.cn/flutter/animation/Curves-class.html)
- [How to make scrolling counter in flutter](https://stackoverflow.com/questions/56607908/how-to-make-scrolling-counter-in-flutter)

