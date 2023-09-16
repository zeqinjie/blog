---
title: 一个简单的 Flutter 步进器计算组件
date: 2022-08-01 14:51:11
categories: "Flutter"
tags:
  - Flutter
---


## 前言
近期有个业务需求，涉及用户付费相关的计算，需要一个步进器计算组件，组件功能如下：
- 仅限支持整数，限制值范围
- 每次步进加累加 5
- 支持文本框手动输入，失去焦点后结果会自动校正为 5 的倍数
- 每次结果变化会有个放大动画的效果

![](/images/2022/一个简单的-Flutter-步进器计算组件/1.gif)

Github：[tw_step_counter](https://github.com/zeqinjie/tw_step_counter)

## 安装
```yaml
dependencies:
  tw_step_counter: ^0.1.5
```

## 效果

![](/images/2022/一个简单的-Flutter-步进器计算组件/2.gif)

## 使用

### 配置属性

```dart
 /// 每次按钮点击递增递减值
  final double differValue;

  /// 每次输入倍数，失去焦点时触发
  final double? inputMultipleValue;

  /// 支持最小值
  final double mixValue;

  /// 支持最大值
  final double maxValue;

  /// 按钮宽度
  final double? btnWidth;

  /// 高度
  final double? height;

  /// 值颜色
  final Color? valueColor;

  /// 值字体大小
  final double? valueFontSize;

  /// 单位
  final String? unit;

  /// 单位颜色
  final Color? unitColor;

  /// 单位字体大小
  final double? unitFontSize;

  /// 当前值
  final double? currentValue;

  /// 点击回调
  final void Function(double value)? onTap;

  /// 输入的回调
  final void Function(double value)? inputTap;

  /// 默认颜色
  final Color? iconColor;

  /// 禁止点击颜色
  final Color? forbiddenIconColor;

  /// 保留多少位小数位
  final int decimalsCount;

  /// 高亮颜色
  final Color? highlightColor;

  /// 默认背景色
  final Color? defaultColor;

  /// 边线颜色
  final Color? borderLineColor;

  /// 间隙
  final EdgeInsetsGeometry? padding;

  /// 值的组件间隙
  final EdgeInsetsGeometry? valuePadding;

  /// 控制器
  final TWStepCounterController? controller;

  /// 是否支持小数点
  final bool? decimal;

  /// 是否自动限制值范围，默认会
  final bool isUpdateInLimitValue;

  /// 是否支持动画，默认会
  final bool isSupportAnimation;

    /// 限制输入的长度
  final int? limitLength;
  
```

### DEMO

```dart
TWStepCounter(
      unit: '元/天',
      currentValue: 130,
      mixValue: 75,
      maxValue: 250,
      onTap: (value) {
        print('点击回调===>$value');
      },
      inputTap: (value) {
        print('输入回调===>$value');
      },
      controller: controller,
      defaultColor: TWColors.tw999999,
      highlightColor: Colors.yellow,
      borderLineColor: Colors.red,
      inputMultipleValue: 5,
      // height: 70,
      // isUpdateInLimitValue: false,
      // isSupportAnimation: false,
      // decimal: true,
      // decimalsCount: 2,
      valuePadding: const EdgeInsets.only(
        left: 10,
        right: 10,
      ),
    )
```