---
title: 五）Flutter 布局详解(二) 之探索约束传递与尺寸确定
date: 2022-07-03 22:16:29
categories: "Flutter"
tags:
  - Flutter
---

## 前言

接着上一篇 [Flutter 布局详解（一）](https://juejin.cn/post/7087057528374165512) 我们继续深入学习，探索约束传递与尺寸确定，以加深巩固。

### 源码分析

```dart
void main() {
  runApp(const ColoredBox(color: Colors.blue));
}

class ColoredBox extends SingleChildRenderObjectWidget {
  ...
  @override
  RenderObject createRenderObject(BuildContext context) {
    return _RenderColoredBox(color: color);
  }
}

```

- 控制本质 `RenderObjectWidget` 负责布局，通过重写
`createRenderObject` ，在 `ColoredBox` 中是 `_RenderColoredBox` 负责渲染组件

```dart
class _RenderColoredBox extends RenderProxyBoxWithHitTestBehavior {
  ...
  @override
  void paint(PaintingContext context, Offset offset) {
    if (size > Size.zero) {
      context.canvas.drawRect(offset & size, Paint()..color = color);
    }
    if (child != null) {
      context.paintChild(child!, offset);
    }
  }
}

abstract class RenderProxyBoxWithHitTestBehavior extends RenderProxyBox {
  ...
}

class RenderProxyBox extends RenderBox with RenderObjectWithChildMixin<RenderBox>, RenderProxyBoxMixin<RenderBox> {
  ...
}

```

- 发现 canvas 绘制当前组件的 `size` 决定组件的尺寸大小，定义在  RenderBox
- `_RenderColoredBox` 继承  `RenderProxyBoxWithHitTestBehavior` 继承 `RenderProxyBox` 继承  `RenderBox` 混入 `RenderObjectWithChildMixin`

```dart
abstract class RenderBox extends RenderObject {
  ...
  @protected
  set size(Size value) {
    ...
    _size = value;
    assert(() {
      debugAssertDoesMeetConstraints();
      return true;
    }());
  }
}
```

- 断点此处  `_size = value;`  发现是 `mix` 类 `RenderProxyBoxMixin`  执行 `performLayout()` 发现 size 的赋值 `size = computeSizeForNoChild(constraints);`

 ![](/images/2022/五）Flutter-布局详解-探索约束传递与尺寸确定/1.png)

```dart
mixin RenderProxyBoxMixin<T extends RenderBox> on RenderBox, RenderObjectWithChildMixin<T> {    
  @override
  void performLayout() {
    if (child != null) {
      child!.layout(constraints, parentUsesSize: true);
      size = child!.size;
    } else {
      size = computeSizeForNoChild(constraints);
    }
  }
  
  Size computeSizeForNoChild(BoxConstraints constraints) {
    return constraints.smallest;
  }
}


abstract class RenderBox extends RenderObject {
  ...
  @override
  BoxConstraints get constraints => super.constraints as BoxConstraints;
}

abstract class RenderObject extends AbstractNode with DiagnosticableTreeMixin implements HitTestTarget {
  ...
  @protected
  Constraints get constraints {
    if (_constraints == null)
      throw StateError('A RenderObject does not have any constraints before it has been laid out.');
    return _constraints!;
  }
  Constraints? _constraints;
  
  void layout(Constraints constraints, { bool parentUsesSize = false }) {
    ...
    _constraints = constraints;
  }
}


```

- 从源码看了解到 混入 `RenderProxyBoxMixin` 继承  `RenderBox` ，`constraints` 是定义在  `RenderObject` 
- `constraints` 赋值是在 `RenderObject layout` 方法中， 断点此处发现是 `RenderView.performLayout`

 ![](/images/2022/五）Flutter-布局详解-探索约束传递与尺寸确定/2.png)

```dart
class RenderView extends RenderObject with RenderObjectWithChildMixin<RenderBox> {
  @override
  void performLayout() {
    assert(_rootTransform != null);
    _size = configuration.size;
    assert(_size.isFinite);

    if (child != null)
      child!.layout(BoxConstraints.tight(_size));
  }
}
```

- 设置断点参数 `child` 发现他即是 `RenderObject (_RenderColoredBox)`
- 决定它大小的源头在 `RenderView.performLayout` 的  `_size = configuration.size;`

### 继承关系

![](/images/2022/五）Flutter-布局详解-探索约束传递与尺寸确定/3.png)

 ![](/images/2022/五）Flutter-布局详解-探索约束传递与尺寸确定/4.png)

### 布局流程

 ![](/images/2022/五）Flutter-布局详解-探索约束传递与尺寸确定/5.png)

### size 的赋值调用堆栈

 ![](/images/2022/五）Flutter-布局详解-探索约束传递与尺寸确定/6.png)

### RenderProxyBox 

- [尺寸法则 1]. 在渲染对象没有子级的情况下，其 [尺寸] 是受到盒约束的 [最小尺寸]。
-  [尺寸法则 2]. 在渲染对象有子级的情况下，其 [尺寸] 会参考 [子级尺寸]。

```dart
mixin RenderProxyBoxMixin<T extends RenderBox> on RenderBox, RenderObjectWithChildMixin<T> {  
  ...
  void performLayout() {
    if (child != null) {
      child!.layout(constraints, parentUsesSize: true);
      size = child!.size;
    } else {
      size = computeSizeForNoChild(constraints);
    }
  }
}
```
### 总结

- 像 ColoredBox 、DecoratedBox 、Opacity 、Offstage 、Transform 这些单子组件，本身只是为了实现某个功能，并没有什么布局特性。但任何渲染对象都需要布局的处理逻辑，所以这些组件对应的渲染对象会继承自 RenderProxyBox ，通过 RenderProxyBoxMixin 来提供统一的默认布局逻辑。
- 像 SizedBox 、ConstrainedBox 、AspectRatio、LimitedBox 、FittrdBox 等组件有布局特性，其对应的渲染对象也继承自 RenderProxyBox ，其源码中对 performLayout 进行了重写。

 ![](/images/2022/五）Flutter-布局详解-探索约束传递与尺寸确定/7.png)

## 参考

- [Flutter 布局探索 - 薪火相传](https://juejin.cn/book/7075958265250578469/section/7077532465027350535)

