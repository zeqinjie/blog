---
title: Flutter 仿 Hero 的动画
date: 2023-08-08 14:58:12
categories: "Flutter"
tags:
  - Flutter
---

Flutter 模仿 Hero 动画的效果，实现逻辑比较简单，就是用 Stack 结合 AnimatedBuilder 组件实现类似 Hero 的转场的动画效果。

## 效果
![](/images/2023/Flutter仿Hero的动画/1.gif)

## 代码
### DEMO
```dart
class TWAnimationHeroApp extends StatelessWidget {
  final controller = TWAnimationHeroController();
  TWAnimationHeroApp({super.key});

  @override
  Widget build(BuildContext context) {
    Widget heroChild = GestureDetector(
      onTap: () => controller.executeAnimation(),
      child: Image.asset(
        Assets.beauty.path,
        fit: BoxFit.fitHeight,
      ),
    );

    return MaterialApp(
      theme: ThemeData(primarySwatch: Colors.grey),
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(),
        body: TWAnimationHero(
          controller: controller,
          heroChild: heroChild,
          child: Stack(
            children: [
              ListView(
                children: [
                  Container(
                    height: 100,
                    alignment: Alignment.center,
                    color: Colors.orange,
                    child: GestureDetector(
                      onTap: () => controller.reverseAnimation(),
                      child: SizedBox(
                        width: 50,
                        height: 50,
                        key: controller.targetKey,
                        child: Image.asset(
                          Assets.beauty.path,
                        ),
                      ),
                    ),
                  ),
                  Container(
                    height: 100,
                    color: Colors.black,
                  ),
                  Container(
                    height: 100,
                    color: Colors.green,
                  ),
                  Container(
                    height: 100,
                    color: Colors.red,
                  ),
                  Container(
                    height: 100,
                    color: Colors.lime,
                  ),
                  Container(
                    height: 100,
                    color: Colors.green,
                  ),
                  Container(
                    height: 100,
                    color: Colors.yellow,
                  ),
                  Container(
                    height: 100,
                    color: Colors.blueAccent,
                  ),
                ],
              ),
            ],
          ),
        ),
      ),
    );
  }
}
```
### TWAnimationHeroController
```dart
class TWAnimationHeroController extends ChangeNotifier {
  GlobalKey targetKey = GlobalKey();
  GlobalKey heroKey = GlobalKey();

  /// 是否可见
  bool get isHeroVisible => _isHeroVisible;

  bool _isHeroVisible = true;

  set heroVisible(bool value) {
    _isHeroVisible = value;
    notifyListeners();
  }

  /// 是否方向状态
  bool isReverse = false;
  AnimationController? controller;
  Animation? animation;

  double offTop = 0;
  double offBottom = 0;
  double offLeft = 0;
  double offRight = 0;
  TWAnimationHeroController();

  /// 执行正向动画
  executeAnimation() {
    if (isReverse) return;
    isReverse = true;
    final child1Rect = fetchChildRect(targetKey);
    final child2Rect = fetchChildRect(heroKey);
    if (child1Rect == null || child2Rect == null) return;
    offTop = child1Rect.top - child2Rect.top;
    offBottom = child2Rect.bottom - child1Rect.bottom;
    offLeft = child1Rect.left - child2Rect.left;
    offRight = child2Rect.right - child1Rect.right;
    controller?.forward();
  }

  /// 执行反向动画
  reverseAnimation() {
    if (!isReverse) return;
    heroVisible = true;
    isReverse = false;
    controller?.reverse();
  }

  Rect? fetchChildRect(GlobalKey key) {
    RenderBox? renderBox = key.currentContext?.findRenderObject() as RenderBox?;
    if (renderBox == null) return null;
    final size = renderBox.size;
    final offset = renderBox.localToGlobal(Offset.zero);
    final childRect = offset & size;
    return childRect;
  }
}
```

### TWAnimationHero 组件
```dart
class TWAnimationHero extends StatefulWidget {
  final Widget child;
  final Widget? heroChild;

  final TWAnimationHeroController controller;
  const TWAnimationHero({
    super.key,
    required this.controller,
    required this.child,
    this.heroChild,
  });

  @override
  State<TWAnimationHero> createState() => _TWAnimationHeroState();
}

class _TWAnimationHeroState extends State<TWAnimationHero>
    with TickerProviderStateMixin {
  @override
  void initState() {
    super.initState();
    createController();
  }

  /// 创建控制器
  createController() {
    final controller = AnimationController(
      vsync: this,
      duration: const Duration(seconds: 1),
    );

    //应用curve
    widget.controller.animation = CurvedAnimation(
      parent: controller,
      curve: Curves.easeIn,
    );

    controller.addListener(() {
      // 注意正向动画才会监听到 isCompleted
      if (controller.isCompleted) {
        widget.controller.heroVisible = false;
      }
    });

    widget.controller.controller = controller;
  }

  @override
  void didUpdateWidget(covariant TWAnimationHero oldWidget) {
    super.didUpdateWidget(oldWidget);
    if (widget.controller.controller == null) {
      widget.controller.controller?.dispose();
      createController();
    }
  }

  @override
  void dispose() {
    widget.controller.controller?.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Stack(
      children: [
        widget.child,
        if (widget.heroChild != null &&
            widget.controller.controller != null &&
            widget.controller.animation != null)
          AnimatedBuilder(
            animation: widget.controller.controller!,
            builder: (BuildContext context, Widget? child) {
              return Positioned(
                top: widget.controller.animation!.value *
                    widget.controller.offTop,
                bottom: widget.controller.animation!.value *
                    widget.controller.offBottom,
                left: widget.controller.animation!.value *
                    widget.controller.offLeft,
                right: widget.controller.animation!.value *
                    widget.controller.offRight,
                child: child!,
              );
            },
            child: AnimatedBuilder(
              animation: widget.controller,
              builder: (BuildContext context, Widget? child) {
                return Visibility(
                  visible: widget.controller.isHeroVisible,
                  child: Container(
                    color: Colors.transparent,
                    key: widget.controller.heroKey,
                    child: widget.heroChild,
                  ),
                );
              },
            ),
          ),
      ],
    );
  }
}

```

