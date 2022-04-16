---
title: 四）Flutter 布局详解
date: 2022-04-16 11:32:43
categories: "Flutter"
tags:
  - Flutter
---

## 前言
对于约束布局深入探索，从布局原理 -> 布局约束 -> 打破布局探索

## 布局原理
> 感兴趣可以看这篇 [传送门](https://juejin.cn/post/6897012238318895117#heading-7)

StatelessWidget 和StatefulWidget 其实只是 **组合类 **的控件，实际上他并不负责绘制，所有我们在屏幕上看到的UI 最终几乎都会通过 **RenderObjectWidget**实现。而 **RenderObjectWidget **中有个 createRenderObject() 方法生成RenderObject对象，RenderObject 实际负责实际 的 layout() 和 paint()

Flutter的渲染流程关键在于 **drawFrame() **方法中
整个过程和原生分为三个阶段 **build()**, **layout()**, **paint()**

```dart
void drawFrame() {
  //在这之前已经完成了build()
  pipelineOwner.flushLayout();
  pipelineOwner.flushCompositingBits();
  pipelineOwner.flushPaint();
  renderView.compositeFrame(); // this sends the bits to the GPU
  pipelineOwner.flushSemantics(); // this also sends the semantics to the OS.
}
```
## 布局约束
### 什么是紧约束？
tight（紧约束）：当 max 和 min 值相等时，这时传递给子类的是一个确定的宽高值

```dart
BoxConstraints.tight(Size size)
    : minWidth = size.width,
      maxWidth = size.width,
      minHeight = size.height,
      maxHeight = size.height;

const BoxConstraints.expand({
    double width,
    double height,
  }) : minWidth = width ?? double.infinity,
       maxWidth = width ?? double.infinity,
       minHeight = height ?? double.infinity,
       maxHeight = height ?? double.infinity;

```
### 什么是松约束？
_loose_（松约束）：当 minWidth 和 minHeight 为 0，这时传递给子类的是一个不确定的宽高值

```dart
  BoxConstraints.loose(Size size)
    : minWidth = 0.0,
      maxWidth = size.width,
      minHeight = 0.0,
      maxHeight = size.height;
```

- 官方介绍：[传送门](https://api.flutter-io.cn/flutter/rendering/BoxConstraints-class.html)
- 当然如果最大值和最小值都为0， 那它即是紧约束也是松约束
- bounded（有界）：最大约束不是 double.infinity
   - [hasBoundedWidth](https://api.flutter-io.cn/flutter/rendering/BoxConstraints/hasBoundedWidth.html) 有限的宽度
   - [hasBoundedHeight](https://api.flutter-io.cn/flutter/rendering/BoxConstraints/hasBoundedHeight.html) 有限的高度
- unbounded（无界）： 最小约束是 double.infinity
   - [hasInfiniteWidth](https://api.flutter-io.cn/flutter/rendering/BoxConstraints/hasInfiniteWidth.html)  无限的宽度
   - [hasInfiniteHeight](https://api.flutter-io.cn/flutter/rendering/BoxConstraints/hasInfiniteHeight.html)  无限的高度
### 布局的过程
> **向下传递约束 & 向上传递尺寸**

#### 流程

- Widget 会通过它的 **父级** 获得自身的约束。约束实际上就是 4 个浮点类型的集合：最大/最小宽度，以及最大/最小高度。

```dart
  BoxConstraints({
    this.minWidth,
    this.maxWidth,
    this.minHeight,
    this.maxHeight,
  });
```

- 然后，这个 widget 将会逐个遍历它的 **children** 列表。向子级传递 **约束**（子级之间的约束可能会有所不同），然后询问它的每一个子级需要用于布局的大小。
- 然后，这个 widget 就会对它子级的 **children** 逐个进行布局。（水平方向是 x 轴，竖直是 y 轴）
- 最后，widget 将会把它的大小信息向上传递至父 widget（包括其原始约束条件）。

![](/images/2022/四）Flutter-布局详解/1.png)

#### 概括

- 首先，上层 widget 向下层 widget 传递约束条件；
- 然后，下层 widget 向上层 widget 传递大小信息。
- 最后，上层 widget 决定下层 widget 的位置。

#### 限制
正如上述所介绍的布局规则中所说的那样， Flutter 的布局引擎有一些重要限制：

- 一个 widget 仅在其父级给其约束的情况下才能决定自身的大小。这意味着 widget 通常情况下 **不能任意获得其想要的大小**。
- 一个 widget **无法知道，也不需要决定其在屏幕中的位置**。因为它的位置是由其父级决定的。
- 当轮到父级决定其大小和位置的时候，同样的也取决于它自身的父级。所以，在不考虑整棵树的情况下，几乎不可能精确定义任何 widget 的大小和位置。
- 如果子级想要拥有和父级不同的大小，然而父级没有足够的空间对其进行布局的话，子级的设置的大小可能会不生效。 **这时请明确指定它的对齐方式。**



### 组件
#### Column

- 📢 细节： 当 crossAxisAlignment 为 CrossAxisAlignment.stretch 时候组件会在纵轴调整为紧约束，宽度占满父 widget
- 源码注释说明了如下

![](/images/2022/四）Flutter-布局详解/2.png)

![](/images/2022/四）Flutter-布局详解/3.png)

- 使用 Expanded 或者 Flexible 包裹的是有弹性的组件
- 当 Column 包裹 ListView 会出现异常 `Vertical viewport was given unbounded height.` 原因 Column 和 ListView 都是 unbounded height。无限高度组件嵌入无限高度组件就报异常了解决如下。对 ListView 用 Expanded 或者 Flexible 包裹，转为弹性组件，也即是占位 Column 剩余的空间即可

#### Stack

- 用了 Positioned 包裹的是有位置组件, 📢 细节：如果 Positioned 的参数一个都不传，那就还是无位置组件
- 布局大小计算
   - 当包含有位置和无位置的组件时候：先布局没有位置，大小等于无位置的组件的尺寸
   - 如果都是无位置的组件，那就越小越好，大小等于无位置的组件的尺寸
   - 如果都是有位置的组件，那就越大越好，占满父组件大小
   - 如果存在 Align、Center 组件，那就越大越好，占满父组件大小
- fit 属性：
   - 默认 loose: 宽松允许组件的大小在 0 ~ 上限之间  即是 BoxConstraints.loose
   - expand: 让 Stack 的子组件占满其父组件大小
   - passthrough: 将 Stack 父组件的约束传递给子组件

```dart
Widget _buildStack() {
  return Container(
    color: Colors.blue,
    constraints: BoxConstraints(
      minWidth: 20,
      maxWidth: 300,
      minHeight: 20,
      maxHeight: 500,
    ),
    child: Stack(
      fit: StackFit.passthrough,
      children: [
        Container(
          color: Colors.yellow,
          child: Text(
            "zhengzeqin...",
            style: TextStyle(fontSize: 20),
          ),
        ),
        Positioned(
          top: 0,
          right: 0,
          child: Text(
            "0",
            style: TextStyle(
              fontSize: 20,
            ),
          ),
        ),
        Positioned(
          top: 0,
          right: 0,
          child: FlutterLogo(
            size: 100,
          ),
        ),
      ],
    ),
  );
}
```

- 📢 细节：如果使用 Transform.translate 子组件偏移，即使 clipBehavior: Clip.hardEdge 也不会裁剪

#### Container

- 有 child 的会匹配 child 大小，除非当 child 要对齐时 (child 有 Align 组件) 则占满父组件大小 。

```dart
/// 加了 Align 的组件，那他就占满父组件的大小，否则就是 child 组件的大小
Widget _buildContainer1() {
  return Container(
    color: Colors.orange,
    alignment: Alignment.center,
    child: const FlutterLogo(
      size: 100,
    ),
  );
}
```

- 没有 child 越大越好，除非约束无边界，(比如 Column 无边界，包裹的 Container 那它的高度为 0)

```dart
  /// 没 child 越大越好，除非无边界
  /// 当 Column 无边界（unbounded）包裹，那么就高度为 0
  Widget _buildContainer2() {
    return Column(
      children: [
        Container(
          color: Colors.orange,
        ),
      ],
    );
  }
```

- LimitedBox 父级给我高度是无界，则最多使用它的限制值，如果父级给的是有一定高度值，则依照父级。

```dart
/// LimitedBox 在无边界组件（Row、Column）才生效
/// 当在 Column（高度无边界），那么最大是 LimitedBox 的限制高度
Widget _buildContainer4() {
  return Column(
    children: [
      LimitedBox(
        maxHeight: 10,
        child: Container(
          color: Colors.orange,
          alignment: Alignment.center,
          child: const FlutterLogo(size: 200,),
        ),
      ),
    ],
  );
}
```
📢 细节：Container 其实是一些组件的组合，源码如下

```dart
Widget build(BuildContext context) {
    Widget? current = child;

    if (child == null && (constraints == null || !constraints!.isTight)) {
      current = LimitedBox(
        maxWidth: 0.0,
        maxHeight: 0.0,
        child: ConstrainedBox(constraints: const BoxConstraints.expand()),
      );
    }

    if (alignment != null)
      current = Align(alignment: alignment!, child: current);

    final EdgeInsetsGeometry? effectivePadding = _paddingIncludingDecoration;
    if (effectivePadding != null)
      current = Padding(padding: effectivePadding, child: current);

    if (color != null)
      current = ColoredBox(color: color!, child: current);

    if (clipBehavior != Clip.none) {
      assert(decoration != null);
      current = ClipPath(
        clipper: _DecorationClipper(
          textDirection: Directionality.maybeOf(context),
          decoration: decoration!,
        ),
        clipBehavior: clipBehavior,
        child: current,
      );
    }

    if (decoration != null)
      current = DecoratedBox(decoration: decoration!, child: current);

    if (foregroundDecoration != null) {
      current = DecoratedBox(
        decoration: foregroundDecoration!,
        position: DecorationPosition.foreground,
        child: current,
      );
    }

    if (constraints != null)
      current = ConstrainedBox(constraints: constraints!, child: current);

    if (margin != null)
      current = Padding(padding: margin!, child: current);

    if (transform != null)
      current = Transform(transform: transform!, alignment: transformAlignment, child: current);

    return current!;
  }
```
#### CustomMultiChildLayout

- 自定义 ZQUnderLineTextDelegate 继承 `MultiChildLayoutDelegate` 协议
   - 重写 `void performLayout(Size size)`方法
      - 向下传递约束调用 `layoutChild(Object childId, BoxConstraints constraints)` 方法
      - 向上传递尺寸 `size = layoutChild(...) `
   - 调用 `positionChild` 方法设置位置
   - 重写 `shouldRelayout` 方法，是否需要重绘
- 📢 id 唯一且一样

![](/images/2022/四）Flutter-布局详解/4.png)

```dart
/// 自定义 下划线文本 Layout
Widget _buildCustomMultiChildLayout2() {
  return CustomMultiChildLayout(
    delegate: ZQUnderLineTextDelegate(),
    children: [
      LayoutId(
        id: 'underline',
        child: Container(
          color: Colors.red,
        ),
      ),
      LayoutId(
        id: 'text',
        child: const Text(
          'hello world',
          style: TextStyle(
            fontSize: 20,
            color: Colors.black,
          ),
        ),
      ),
    ],
  );
}


class ZQUnderLineTextDelegate extends MultiChildLayoutDelegate {
  final double thickness;

  ZQUnderLineTextDelegate({this.thickness = 2.0});

  @override
  void performLayout(Size size) {
    if (hasChild('text') && hasChild('underline')) {
      // 向下传递约束，向上传递尺寸
      Size textSize = layoutChild(
        'text', // 组件的 ID 唯一
        BoxConstraints.loose(size),
      );
      layoutChild(
        'underline',
        BoxConstraints.tight(Size(textSize.width, thickness)),
      );
      final left = (size.width - textSize.width) / 2;
      final top = (size.height - textSize.height) / 2;
      // 设置位置
      positionChild('text', Offset(left, top));
      positionChild('underline', Offset(left, top + textSize.height));
    }
  }

  @override
  bool shouldRelayout(covariant MultiChildLayoutDelegate oldDelegate) => true;
}
```

#### RenderObject

- 自定义 ZQRenderBox 继承 `SingleChildRenderObjectWidget`
   - 多子布局 `SingleChildRenderObjectWidget` & `MultiChildRenderObjectWidget` 都是继承 RenderObjectWidget
   - 重写 `createRenderObject` 方法返回一个 RenderObject 对象
   - 📢 如果需要热更新，实现 updateRenderObject
- 自定义 RenderZQRenderBox 继承 `RenderBox`
   - RenderBox 继承 `RenderObject`
   - 实现 `performLayout` 布局方法
      - 向下传递约束 `child?.layout(constraints, parentUsesSize: true);`
         - 📢 如果想父 widget 和当前 widget 尺寸大小一样需 `parentUsesSize` = true， 如果 parentUsesSize = false 对性能有帮助，保证了父 widget 大小不会因当前 child 变化而变化
      - 向上传递尺寸 size = (child as RenderBox).size;
   - 可以灵活实现绘制 `void paint(PaintingContext context, Offset offset)`
   - 📢 如果要有 overflow 提示，需要混入 `DebugOverflowIndicatorMixin` 调用 ` `paintOverflowIndicator` 方法

![](/images/2022/四）Flutter-布局详解/5.png)

```dart
/// 自定义 RenderObject
Widget _buildRenderBox() {
  return Container(
    color: Colors.blue,
    child:  ZQRenderBox(
      const FlutterLogo(
        size: 1000,
      ),
      distance: 10,
      parentSize: const Size(300, 300),
    ),
  );
}

/// 自定义 ZQRenderBox 继承 SingleChildRenderObjectWidget
/// 多子布局 SingleChildRenderObjectWidget & MultiChildRenderObjectWidget 都是继承 RenderObjectWidget
class ZQRenderBox extends SingleChildRenderObjectWidget {
  /// 确定父组件大小
  Size? parentSize;
  double? distance;
  ZQRenderBox(Widget child, {Key? key, this.parentSize, this.distance = 0}) : super(key: key, child: child);

  @override
  RenderObject createRenderObject(BuildContext context) {
    // TODO: implement createRenderObject
    // 返回一个 RenderObject 对象
    return RenderZQRenderBox(parentSize: parentSize, distance: distance);
  }

  /// 实现热更新 RenderObject 参数
  @override
  void updateRenderObject(BuildContext context, covariant RenderZQRenderBox renderObject) {
    // TODO: implement updateRenderObject
    // super.updateRenderObject(context, renderObject);
    renderObject.distance = distance;
    renderObject.parentSize = parentSize;
  }
}

/// 渲染对象
/// RenderBox 继承 RenderObject
class RenderZQRenderBox extends RenderBox with RenderObjectWithChildMixin, DebugOverflowIndicatorMixin {  // RenderProxyBox
  Size? parentSize;
  /// 偏移量
  double? distance;
  RenderZQRenderBox({this.parentSize, this.distance = 0});

  @override
  void performLayout() {
    // TODO: implement performLayout
    // super.performLayout(); 注意这个就不调用了
    print('constraints: $constraints');
    // 父 widget 固定了大小
    if (parentSize != null) {
      // 对子组件布局松约束 Size(300, 300)
      // 📢 子组件的布局约束不能大于当前 constraints 值
      child?.layout(BoxConstraints.loose(parentSize!));
      size = parentSize!;//parentSize!;
    } else {
      // 默认是 flutter: constraints: BoxConstraints(0.0<=w<=375.0, 0.0<=h<=706.0)
      child?.layout(constraints, parentUsesSize: true);
      // 如果想父 widget 大小和当前 widget 一样可以
      // 注意 layout 方法需要设置 parentUsesSize = true， 当为 false 对性能有帮助，保证了父 widget 大小不会因当前 child 变化而变化
      print('RenderBox Size: ${(child as RenderBox).size}');
      size = (child as RenderBox).size;
    }
  }

  @override
  void paint(PaintingContext context, Offset offset) {
    // TODO: implement paint
    super.paint(context, offset);
    if (child != null) {
      // 绘制
      context.paintChild(child!, offset);

      if (distance != null) {
        context.pushOpacity(offset, 100, (context, offset) {
          context.paintChild(child!, offset + Offset(distance!, distance!));
        });
      }
      // offset & size = offset(offset.x)
      // print("offset & size: ${offset & size}");
      // print("offset & size: ${offset.dx + size.width}");
      // print("offset & size: ${offset.dy + size.height}");
      paintOverflowIndicator(context, offset, offset & size, offset & (child as RenderBox).size);
    }
  }
}
```
 
## 打破紧约束

- 通过 Scaffold 、Center、Align、Flex、Stack 等组件 `放松约束`，向下传递约束变为 `松约束`。
   - 📢 Scaffold 本身是组合组件，打破紧约束的原因是组合了 `CustomMultiChildLayout`

![](/images/2022/四）Flutter-布局详解/6.png)

- 通过 UnconstrainedBox `解除约束`，让自身约束变为 `无约束`。
- 通过 CustomSingleChildLayout、CustomMultiChildLayout 等自定义布局组件施加 `新约束`。
### 案例：Center

![](/images/2022/四）Flutter-布局详解/7.png)

- 结合上面流程的案例，此时 FlutterLogo 无论设置多大，都被 Container 父 widget 传递的紧约束控制死 BoxConstraints(w=200.0, h=200.0)

![](/images/2022/四）Flutter-布局详解/8.png)

- 如果想打破这紧约束，可以给 FlutterLogo 添加 `Center` 父 widget
- 上述的约束变为了 BoxConstraints(0.0<=w<=200.0, 0.0<=h<=200.0)

📢 细节：从布局原理知道，决定布局大小的是 `RenderObject` 因此我们查阅盒子约束是 `RenderConstrainedBox`
## LayoutBuilder

- 通过  `LayoutBuilder `打印信息，也能知道当前约束变为松约束 BoxConstraints(0.0<=w<=200.0, 0.0<=h<=200.0)

```dart
/// LayoutBuilder
/// 打印信息： flutter: BoxConstraints(0.0<=w<=200.0, 0.0<=h<=200.0)
Widget _buildLogo3(){
  return Container(
    color: Colors.red,
    width: 0.04,
    height: 0.0008,
    child: Center(
      child: Container(
        width: 200,
        height: 200,
        color: Colors.white,
        child: Center(
          child: LayoutBuilder(
            builder: (context, constraints) {
              print(constraints);
              return const FlutterLogo(
                size: 11,
              );
            },
          ),
        ),
      ),
    ),
  );
}


```

## 参考

- [Demo](https://github.com/zeqinjie/flutter_demo)
- [官方 widgets layout](https://docs.flutter.dev/development/ui/widgets/layout)
- [深入理解 Flutter 布局约束](https://flutter.cn/docs/development/ui/layout/constraints)
- [【译】Flutter | 深入理解布局约束](https://juejin.cn/post/6846687593745088526)
- [总结了30个例子之后，我悟到了Flutter的布局原理](https://juejin.cn/post/6914155427651387399)
- [Flutter 布局（王叔）](https://www.bilibili.com/video/BV1254y1s7Zo?spm_id_from=333.999.0.0)

[
](https://stackoverflow.com/questions/56607908/how-to-make-scrolling-counter-in-flutter)

