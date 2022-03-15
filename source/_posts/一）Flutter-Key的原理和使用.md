---
title: 一）Flutter Key的原理和使用
date: 2022-03-15 22:42:01
categories: "Flutter"
tags:
  - Flutter
---

## 没有 Key 
如果没有使用 `key` 情况下，当 widget 的顺序变化，状态是不会跟着变化

### 变化前
![](/images/2022/一）Flutter-Key的原理和使用/1.png)

### 变化后
![](/images/2022/一）Flutter-Key的原理和使用/2.png)

```dart
import 'package:flutter/material.dart';

/// create by:  zhengzeqin
/// create time:  2021-12-19 14:29
/// des:  https://www.bilibili.com/video/BV15k4y1B74z?spm_id_from=333.999.0.0

class TWFlutterKey extends StatelessWidget {
  
  const TWFlutterKey({Key? key}) : super(key: key);
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'key Demo',
      debugShowCheckedModeBanner: false,
      home: Scaffold(
        appBar: AppBar(
          title: const Text("key demo"),
        ),
        body: Center(
          child: _buildRowBox(),
        ),
      ),
    );
  }
  
  Widget _buildColumnBox() {
    return Column(
      mainAxisAlignment: MainAxisAlignment.center,
      children: const [
        _Box(
          // key: ValueKey("grey"),
          color: Colors.grey,
        ),
        _Box(
          // key: ValueKey("red"),
          color: Colors.red,
        ),
        _Box(
          // key: ValueKey("blue"),
          color: Colors.blue,
        ),
      ],
    );
  }
  
  Widget _buildRowBox() {
    return Row(
      mainAxisAlignment: MainAxisAlignment.center,
      children: const [
        _Box(
          // key: ValueKey("grey"),
          color: Colors.grey,
        ),
        _Box(
          // key: ValueKey("red"),
          color: Colors.red,
        ),
        _Box(
          // key: ValueKey("blue"),
          color: Colors.blue,
        ),
      ],
    );
  }
}

class _Box extends StatefulWidget {
  final Color color;
  final width = 100.0;
  final height = 100.0;
  
  const _Box({Key? key, this.color = Colors.yellow}) : super(key: key);
  
  @override
  _CountState createState() => _CountState();
}

class _CountState extends State<_Box> {
  int _counter = 0;
  
  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: () {
        _incrementCounter();
      },
      child: Container(
        alignment: Alignment.center,
        width: widget.width,
        height: widget.height,
        color: widget.color,
        child: Text("count:$_counter"),
      ),
    );
  }
  
  void _incrementCounter() {
    setState(() {
      _counter++;
    });
  }
}

```

## key 的作用
Widget 的 state 是根据 key 来记录的，当没有使用 key 下变化位置，颜色是调整了。但数字没有改变。

![](/images/2022/一）Flutter-Key的原理和使用/3.png)
![](/images/2022/一）Flutter-Key的原理和使用/4.png)
Widget 的状态是根据以下做对比的

- 类型，比如 Column 的 Element tree 是 ColumnElement 。如果 Colunmn 换成 Row 那就会引起状态变化。所以判断两个对象是否相等, 会先去比较它们的类型
- key，其次判断 key 是否一致, 来定位对应的 widget 

## 三种 Key 的

- LocalKey 局部键,在同一级中要唯一, 可以理解为同级唯一性
   - 同一级指的是同个 tree 下，比如下面 Center 和 Column 不在同一级下所以同样的 `ValueKey(1)`就没有影响
   
![](/images/2022/一）Flutter-Key的原理和使用/5.png)

   - 三种 LocalKey
      - ValueKey
      - ObjectKey
      - UniqueKey
- GlobalKey 全局键 , 在整个App中必须是唯一的.

### ValueKey

```dart
class ValueKey<T> extends LocalKey {
  /// Creates a key that delegates its [operator==] to the given value.
  const ValueKey(this.value);

  /// The value to which this key delegates its [operator==]
  final T value;

  @override
  bool operator ==(Object other) {
    if (other.runtimeType != runtimeType)
      return false;
    return other is ValueKey<T>
        && other.value == value;
  }

  @override
  int get hashCode => hashValues(runtimeType, value);

  @override
  String toString() {
    final String valueString = T == String ? "<'$value'>" : '<$value>';
    // The crazy on the next line is a workaround for
    // https://github.com/dart-lang/sdk/issues/33297
    if (runtimeType == _TypeLiteral<ValueKey<T>>().type)
      return '[$valueString]';
    return '[$T $valueString]';
  }
}
```

### ObjectKey 

```dart
class ObjectKey extends LocalKey {
  /// Creates a key that uses [identical] on [value] for its [operator==].
  const ObjectKey(this.value);
  
  /// The object whose identity is used by this key's [operator==].
  final Object? value;
  
  @override
  bool operator ==(Object other) {
    if (other.runtimeType != runtimeType)
      return false;
    return other is ObjectKey
      && identical(other.value, value);
  }
  
  @override
  int get hashCode => hashValues(runtimeType, identityHashCode(value));
  
  @override
  String toString() {
    if (runtimeType == ObjectKey)
      return '[${describeIdentity(value)}]';
    return '[${objectRuntimeType(this, 'ObjectKey')} ${describeIdentity(value)}]';
  }
}
```
### UniqueKey

```dart
class UniqueKey extends LocalKey {
  /// Creates a key that is equal only to itself.
  ///
  /// The key cannot be created with a const constructor because that implies
  /// that all instantiated keys would be the same instance and therefore not
  /// be unique.
  // ignore: prefer_const_constructors_in_immutables , never use const for this class
  UniqueKey();

  @override
  String toString() => '[#${shortHash(this)}]';
}
```

## 参考
- [Flutter 键（Key）的原理和使用（王叔视频）](https://www.bilibili.com/video/BV1b54y1z7iD?spm_id_from=333.999.0.0)
- [Flutter Key的原理和使用 (一) 没有Key会发生什么](https://juejin.cn/post/7001085222070517796/)
- [Flutter 中的key、LocalKey、GlobalKey等等](https://blog.csdn.net/qq_32760901/article/details/91798507)
- [Flutter: 当使用了PageStorageKey后发生了什么?](https://blog.csdn.net/u013066292/article/details/112943145)
- [flutter_demo](https://github.com/zeqinjie/flutter_demo)
