---
title: 三）Flutter 动画详解（二）
date: 2022-04-05 13:19:15
categories: "Flutter"
tags:
  - Flutter
---

## 前言

接上一篇文章 [Flutter 动画详解（一）](https://juejin.cn/post/7080158569378611214)，继续对视频教程做笔记。

## 动画背后的机制和原理
### 隐式动画原理

- AnimatedContainer 继承 ImplicitlyAnimatedWidget 本质就是 ImplicitlyAnimatedWidgetState
- ImplicitlyAnimatedWidgetState 使用的是 AnimationController
- 查看源码发现无论是 AnimatedContainer、AnimatedPadding 还是其他的隐式动画组件它们的 state 都是 AnimatedWidgetBaseState
- AnimatedWidgetBaseState 做的一件事就是对 controller 添加回调监听方法调用的是 `setState(() {  });`

```dart
Widget _buildAnimatedContainer() {
  return Center(
    child: AnimatedContainer(
      duration: const Duration(seconds: 1),
      color: Colors.blue,
      height: _height,
      width: _height,
    ),
  );
}

class AnimatedContainer extends ImplicitlyAnimatedWidget {
  //...
  
  AnimatedWidgetBaseState<AnimatedContainer> createState() => _AnimatedContainerState();
}

abstract class AnimatedWidgetBaseState<T extends ImplicitlyAnimatedWidget> extends ImplicitlyAnimatedWidgetState<T> {
  @override
  void initState() {
    super.initState();
    controller.addListener(_handleAnimationChanged);
  }

  void _handleAnimationChanged() {
    setState(() { /* The animation ticked. Rebuild with new animation value */ });
  }
}

/// 隐式动画源码
abstract class ImplicitlyAnimatedWidgetState<T extends ImplicitlyAnimatedWidget> extends State<T> with SingleTickerProviderStateMixin<T> {
  /// The animation controller driving this widget's implicit animations.
  @protected
  AnimationController get controller => _controller;
  late final AnimationController _controller = AnimationController(
    duration: widget.duration,
    debugLabel: kDebugMode ? widget.toStringShort() : null,
    vsync: this,
  );

  /// The animation driving this widget's implicit animations.
  Animation<double> get animation => _animation;
  late Animation<double> _animation = _createCurve();

  @override
  void initState() {
    super.initState();
    _controller.addStatusListener((AnimationStatus status) {
      switch (status) {
        case AnimationStatus.completed:
          widget.onEnd?.call();
          break;
        case AnimationStatus.dismissed:
        case AnimationStatus.forward:
        case AnimationStatus.reverse:
      }
    });
    _constructTweens();
    didUpdateTweens();
  }

  @override
  void didUpdateWidget(T oldWidget) {
    super.didUpdateWidget(oldWidget);
    if (widget.curve != oldWidget.curve) {
      (_animation as CurvedAnimation).dispose();
      _animation = _createCurve();
    }
    _controller.duration = widget.duration;
    if (_constructTweens()) {
      forEachTween((Tween<dynamic>? tween, dynamic targetValue, TweenConstructor<dynamic> constructor) {
        _updateTween(tween, targetValue);
        return tween;
      });
      _controller
        ..value = 0.0
        ..forward();
      didUpdateTweens();
    }
  }

  CurvedAnimation _createCurve() {
    return CurvedAnimation(parent: _controller, curve: widget.curve);
  }

  @override
  void dispose() {
    (_animation as CurvedAnimation).dispose();
    _controller.dispose();
    super.dispose();
  }

  bool _shouldAnimateTween(Tween<dynamic> tween, dynamic targetValue) {
    return targetValue != (tween.end ?? tween.begin);
  }

  void _updateTween(Tween<dynamic>? tween, dynamic targetValue) {
    if (tween == null)
      return;
    tween
      ..begin = tween.evaluate(_animation)
      ..end = targetValue;
  }

  bool _constructTweens() {
    bool shouldStartAnimation = false;
    forEachTween((Tween<dynamic>? tween, dynamic targetValue, TweenConstructor<dynamic> constructor) {
      if (targetValue != null) {
        tween ??= constructor(targetValue);
        if (_shouldAnimateTween(tween, targetValue))
          shouldStartAnimation = true;
      } else {
        tween = null;
      }
      return tween;
    });
    return shouldStartAnimation;
  }

  
  @protected
  void forEachTween(TweenVisitor<dynamic> visitor);

  @protected
  void didUpdateTweens() { }
}


```
### 显式动画的原理

- 显式动画原理, AnimatedBuilder 继承 AnimatedWidget 本质就是 _AnimatedState
- _AnimatedState 添加的监听回调方法调用也是 `setState(() {  });`

```dart
abstract class AnimatedWidget extends StatefulWidget {
  // ...
}

class _AnimatedState extends State<AnimatedWidget> {
  @override
  void initState() {
    super.initState();
    widget.listenable.addListener(_handleChange);
  }

  @override
  void didUpdateWidget(AnimatedWidget oldWidget) {
    super.didUpdateWidget(oldWidget);
    if (widget.listenable != oldWidget.listenable) {
      oldWidget.listenable.removeListener(_handleChange);
      widget.listenable.addListener(_handleChange);
    }
  }

  @override
  void dispose() {
    widget.listenable.removeListener(_handleChange);
    super.dispose();
  }

  void _handleChange() {
    setState(() {
      // The listenable's state is our build state, and it changed already.
    });
  }

  @override
  Widget build(BuildContext context) => widget.build(context);
}
```

### 通过源码得出结论
无论是隐式动画还是显式动画本质上都是调用 setState 去更新 UI

### Ticker
Ticker 的作用是目的其实是类似定时器一样，去刷新 UI 

#### 使用

```dart
void initState() {
  super.initState();
  // ticker 使用
  Ticker ticker = Ticker(
    (_) => setState(
      () {
        _height += 1;
        if (_height > 500) {
          _height = 100;
        }
      },
    ),
  );
  ticker.start();
}

Widget _buildAnimatedContainer() {
  return Center(
    child: Container(
      color: Colors.blue,
      height: _height,
      width: _height,
    ),
  );
}
```

####  源码

- _onTick 是我们刷新 UI 的回调函数，当 ticker 调用 start 方法，调用的是 scheduleTick 方法
- scheduleTick 方法调用的 SchedulerBinding._instance_!.scheduleFrameCallback(_tick, rescheduling: rescheduling);
- _tick 方法会发现它调用的是 _onTick(timeStamp - _startTime!); 因此在 ticker 的作用：根据条件调用了 scheduleTick 从而造成循环调用 _onTick 方法
- 比如：手机 60hz 刷新率。那它就会 1秒钟触发 60 次 _onTick 方法。一般_onTick 方法会调用 setState() 从而到达更新 UI 


```dart
class Ticker {
  
  Ticker(this._onTick, { this.debugLabel }) {
    assert(() {
      _debugCreationStack = StackTrace.current;
      return true;
    }());
  }

  TickerFuture? _future;

  bool get muted => _muted;
  bool _muted = false;

  set muted(bool value) {
    if (value == muted)
      return;
    _muted = value;
    if (value) {
      unscheduleTick();
    } else if (shouldScheduleTick) {
      scheduleTick();
    }
  }

  bool get isTicking {
    if (_future == null)
      return false;
    if (muted)
      return false;
    if (SchedulerBinding.instance!.framesEnabled)
      return true;
    if (SchedulerBinding.instance!.schedulerPhase != SchedulerPhase.idle)
      return true; // for example, we might be in a warm-up frame or forced frame
    return false;
  }

  bool get isActive => _future != null;

  Duration? _startTime;

  TickerFuture start() {
    assert(() {
      if (isActive) {
        throw FlutterError.fromParts(<DiagnosticsNode>[
          ErrorSummary('A ticker was started twice.'),
          ErrorDescription('A ticker that is already active cannot be started again without first stopping it.'),
          describeForError('The affected ticker was'),
        ]);
      }
      return true;
    }());
    assert(_startTime == null);
    _future = TickerFuture._();
    if (shouldScheduleTick) {
      scheduleTick();
    }
    if (SchedulerBinding.instance!.schedulerPhase.index > SchedulerPhase.idle.index &&
        SchedulerBinding.instance!.schedulerPhase.index < SchedulerPhase.postFrameCallbacks.index)
      _startTime = SchedulerBinding.instance!.currentFrameTimeStamp;
    return _future!;
  }

  DiagnosticsNode describeForError(String name) {
    // TODO(jacobr): make this more structured.
    return DiagnosticsProperty<Ticker>(name, this, description: toString(debugIncludeStack: true));
  }

  void stop({ bool canceled = false }) {
    if (!isActive)
      return;

    final TickerFuture localFuture = _future!;
    _future = null;
    _startTime = null;
    assert(!isActive);

    unscheduleTick();
    if (canceled) {
      localFuture._cancel(this);
    } else {
      localFuture._complete();
    }
  }


  final TickerCallback _onTick;

  int? _animationId;
  
  @protected
  bool get scheduled => _animationId != null;

  @protected
  bool get shouldScheduleTick => !muted && isActive && !scheduled;

  void _tick(Duration timeStamp) {
    assert(isTicking);
    assert(scheduled);
    _animationId = null;

    _startTime ??= timeStamp;
    _onTick(timeStamp - _startTime!);

    if (shouldScheduleTick)
      scheduleTick(rescheduling: true);
  }

  @protected
  void scheduleTick({ bool rescheduling = false }) {
    assert(!scheduled);
    assert(shouldScheduleTick);
    _animationId = SchedulerBinding.instance!.scheduleFrameCallback(_tick, rescheduling: rescheduling);
  }


  @protected
  void unscheduleTick() {
    if (scheduled) {
      SchedulerBinding.instance!.cancelFrameCallbackWithId(_animationId!);
      _animationId = null;
    }
    assert(!shouldScheduleTick);
  }

  void absorbTicker(Ticker originalTicker) {
    assert(!isActive);
    assert(_future == null);
    assert(_startTime == null);
    assert(_animationId == null);
    assert((originalTicker._future == null) == (originalTicker._startTime == null), 'Cannot absorb Ticker after it has been disposed.');
    if (originalTicker._future != null) {
      _future = originalTicker._future;
      _startTime = originalTicker._startTime;
      if (shouldScheduleTick)
        scheduleTick();
      originalTicker._future = null; // so that it doesn't get disposed when we dispose of originalTicker
      originalTicker.unscheduleTick();
    }
    originalTicker.dispose();
  }
  
  @mustCallSuper
  void dispose() {
    if (_future != null) {
      final TickerFuture localFuture = _future!;
      _future = null;
      assert(!isActive);
      unscheduleTick();
      localFuture._cancel(this);
    }
    assert(() {
      // We intentionally don't null out _startTime. This means that if start()
      // was ever called, the object is now in a bogus state. This weakly helps
      // catch cases of use-after-dispose.
      _startTime = Duration.zero;
      return true;
    }());
  }
  
  /// ...
}
```

### 动画流程图 [传送门](http://gityuan.com/2019/07/13/flutter_animator/)
![](/images/2022/三）Flutter-动画详解（二）/1.png)

## Hero 动画

![iShot2022-04-04 15.18.23.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2e6af6863c024586a04c74545c13f149~tplv-k3u1fbpfcp-zoom-1.image)

- 对应的用需要动效用 Hero 包裹，注意 tag 保持一样的值
- `timeDilation` 是单利对象 `SchedulerBinding` 的属性，可以使动效变慢

```
/// hero 动画
Widget _buildHeroAnimated() {
  const path = "assets/beauty.jpeg";
  return Center(
    child: GridView.count(
      crossAxisCount: 5,
      mainAxisSpacing: 5,
      crossAxisSpacing: 5,
      children: List.generate(100, (index) {
        return GestureDetector(
          onTap: () {
            Navigator.push(
              context,
              MaterialPageRoute(builder: (_) {
                return TWHeroAnimation(
                  tag: index.toString(),
                  assetImageName: path,
                );
              }),
            );
          },
          child: Hero(
            tag: index.toString(),
            child: Image.asset(path),
          ),
        );
      }),
    ),
  );
}

/// hero 动画
class TWHeroAnimation extends StatelessWidget {
  final String tag;
  final String assetImageName;

  const TWHeroAnimation({
    Key? key,
    required this.tag,
    required this.assetImageName,
  }) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Hero 基础动画详情'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Hero(
              tag: tag,
              child: SizedBox(
                width: 200,
                height: 200,
                child: Image.asset(
                  assetImageName,
                  height: 100,
                  width: 100,
                ),
              ),
            ),
            const Padding(
              padding: EdgeInsets.all(8.0),
              child: Material(
                child: Text(
                  "介绍了几篇 Hero 动画，我们来一个 Hero 动画应用案例。在一些应用中，列表的元素和详情的内容是一致的，这个时候利用 Hero 动画切换到详情会感觉无缝过渡，用户体验会更好",
                  style: TextStyle(fontSize: 20),
                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
}

```
## 操作 CustomPainter 动画

![iShot2022-04-04 15.09.02.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4672e9fc000a4874ac04a5f58e9ca451~tplv-k3u1fbpfcp-zoom-1.image)

- 本质上也是每一帧调用 setState() 从而更新 UI

```dart
Widget _buildSnowmanContent(BuildContext context) {
  return Center(
    child: Container(
      color: Colors.black,
      width: double.infinity,
      height: double.infinity,
      child: AnimatedBuilder(
        animation: _controller,
        builder: (context, child) {
          final snowFlakes = _snowFlakes ?? [];
          snowFlakes.forEach((element) {
            element.fall();
          });
          return CustomPaint(
            painter: TWSnowmanPainter(context: context, snowFlakes: snowFlakes),
          );
        },
      ),
    ),
  );
}

class TWSnowmanPainter extends CustomPainter {
  List<TWShowFlake> snowFlakes;
  BuildContext context;

  TWSnowmanPainter({required this.context, required this.snowFlakes});

  @override
  void paint(Canvas canvas, Size size) {
    final width = MediaQuery.of(context).size.width;
    final height = MediaQuery.of(context).size.height;
    final p = Paint()..color = Colors.white;
    // TODO: implement paint
    canvas.drawCircle(
      size.center(const Offset(0, 100)),
      60,
      p,
    );
    canvas.drawOval(
      Rect.fromCenter(
        center: Offset(width / 2.0, height - 180),
        width: 200,
        height: 250,
      ),
      p,
    );
    snowFlakes.forEach((element) {
      canvas.drawCircle(Offset(element.x, element.y), element.radius, p);
    });
    //
  }

  @override
  bool shouldRepaint(covariant CustomPainter oldDelegate) {
    // TODO: implement shouldRepaint
    return true;
  }
}

class TWShowFlake {
  BuildContext context;
  late double x;
  late double y;
  late double radius;
  late double velocity;

  TWShowFlake(this.context) {
    _initData();
  }

  _initData() {
    final width = MediaQuery.of(context).size.width;
    final height = MediaQuery.of(context).size.height;
    x = Random().nextDouble() * width;
    y = Random().nextDouble() * height;
    radius = Random().nextDouble() * 2 + 2;
    velocity = Random().nextDouble() * 4 + 2;
  }

  fall() {
    y = y + velocity;
    if (y > 800) {
      _initData();
    }
  }
}
```

## 嵌入式 Rive 插件动画

### 效果

![iShot2022-04-04 18.48.35.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6ac826d6f1aa4c2aac576c5de3057df3~tplv-k3u1fbpfcp-watermark.image?)

- 翻看了下 Flare 已改名为 [Rive](https://rive.app/) , 原依赖库 [Flare-Flutter ](https://github.com/2d-inc/Flare-Flutter) 也调整为 [rive-flutter](https://github.com/rive-app/rive-flutter)
- 注意 SimpleAnimation('idle') 的 `animationName` 是指定的动画名称，控制器初始化时不能随意写。需要查看下图这块

![](/images/2022/三）Flutter-动画详解（二）/2.png)

```dart

/// 初始化动画
void _initRivAnimation() {
  _riveAnimationController = SimpleAnimation('idle');
  rootBundle.load('assets/knight.riv').then(
        (data) async {
      final file = RiveFile.import(data);
      final artboard = file.mainArtboard;
      _dayNightAnimation = SimpleAnimation('day_night');
      _nightDayAnimation = SimpleAnimation('night_day');
      artboard.addController(SimpleAnimation('idle'));
      setState(() => _riveArtboard = artboard);
    },
  );
}

/// 切换到白天
void _playDayNightAnimation() {
  _riveArtboard?.removeController(_nightDayAnimation);
  _riveArtboard?.addController(_dayNightAnimation);
}

/// 切换到黑夜
void _playNightDayAnimation() {
  _riveArtboard?.removeController(_dayNightAnimation);
  _riveArtboard?.addController(_nightDayAnimation);
}

/// 动画视图
Widget _buildFlareView() {
  return Column(
    children: [
      _riveArtboard == null
          ? const SizedBox()
          : SizedBox(
              width: double.infinity,
              height: 300,
              child: Rive(
                artboard: _riveArtboard!,
              ),
            ),
      SizedBox(
        width: double.infinity,
        height: 300,
        child: RiveAnimation.network(
          'https://cdn.rive.app/animations/vehicles.riv',
          controllers: [_riveAnimationController],
          // Update the play state when the widget's initialized
          onInit: (_) => setState(() {}),
        ),
      ),
    ],
  );
}
```

### RiveAnimation 构造函数

```dart
/// asset 文件
const RiveAnimation.asset(
  this.name, {
  this.artboard,
  this.animations = const [],
  this.stateMachines = const [],
  this.fit,
  this.alignment,
  this.placeHolder,
  this.antialiasing = true,
  this.controllers = const [],
  this.onInit,
}) : src = _Source.asset;

/// 网络图
const RiveAnimation.network(
  this.name, {
  this.artboard,
  this.animations = const [],
  this.stateMachines = const [],
  this.fit,
  this.alignment,
  this.placeHolder,
  this.antialiasing = true,
  this.controllers = const [],
  this.onInit,
}) : src = _Source.network;

/// 文件
const RiveAnimation.file(
  this.name, {
  this.artboard,
  this.animations = const [],
  this.stateMachines = const [],
  this.fit,
  this.alignment,
  this.placeHolder,
  this.antialiasing = true,
  this.controllers = const [],
  this.onInit,
}) : src = _Source.file;
```

## 参考

- [Demo](https://github.com/zeqinjie/flutter_demo)
- [王叔不秃-动画教程](https://www.bilibili.com/video/BV1dz411v76Z?spm_id_from=333.999.0.0)
- [深入理解Flutter动画原理](http://gityuan.com/2019/07/13/flutter_animator/?spm=a2c4e.10696291.0.0.453819a4ns4joC)
- [Flutter 动画详解](https://juejin.cn/post/6844903687966425101)
- [Flutter 隐式动画 Implicit Animation 初解](https://zhuanlan.zhihu.com/p/52572411)
- [Flutter中的千变万化——Flare动画](https://juejin.cn/post/6861865147557183495)
- [CustomPainter画音乐跳动动画](https://juejin.cn/post/6944230617236111391)
- [Flutter 理解 Ticker、Animate 原理](https://juejin.cn/post/6896046208121470989)
- [rive](https://rive.app/)


