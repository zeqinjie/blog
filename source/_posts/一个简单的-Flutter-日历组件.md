---
title: 一个简单的 Flutter 日历组件
date: 2022-08-04 19:30:49
categories: "Flutter"
tags:
  - Flutter
---

## 前言
近期有个业务需求，涉及用户付费相关的计算，需要一个日历组件，组件功能如下：
- 仅支持从明天开始选择预定日期
- 仅支持可选范围内的日期
- 日期的选择是连续的
- 有个推荐日期，需要联动更新
- todo:
    - 支持不连续的日期选择


Github：[tw_calendar](https://github.com/zeqinjie/tw_calendar)

## 安装
```yaml
dependencies:
  tw_calendar: latest_version
```

## 效果

### demo 演示
![](/images/2022/一个简单的-Flutter-日历组件/1.gif)

### 业务使用 headerView
![](/images/2022/一个简单的-Flutter-日历组件/2.gif)

## 使用

### 配置属性

```dart
 /// 开始的年月份
  final DateTime firstDate;

  /// 结束的年月份
  final DateTime lastDate;

  /// 选择开始日期
  final DateTime? selectedStartDate;

  /// 选择结束日期
  final DateTime? selectedEndDate;

  /// 点击方法回调
  final Function? onSelectFinish;

  /// 头部组件
  final Widget? headerView;

  /// 选择模式
  final TWCalendarListSeletedMode? seletedMode;

  /// 月视图高度，为空则占满剩余空间
  final double? monthBodyHeight;

  /// 周视图高度， 默认 48
  final double? weekDayHeight;

  /// 水平间隙
  final double? horizontalSpace;

  /// 确认周视图高度， 默认 66
  final double? ensureViewHeight;

  /// 确认按钮的间隙
  final EdgeInsetsGeometry? ensureViewPadding;

  /// 确认按钮选中颜色
  final Color? ensureViewSelectedColor;

  /// 确认未按钮选中颜色
  final Color? ensureViewUnSelectedColor;

  /// 今天的日期的背景颜色
  final Color? dayNumberTodayColor;

  /// 选中日期背景颜色
  final Color? dayNumberSelectedColor;

  /// 确认按钮字体大小
  final double? ensureTitleFontSize;

  /// 点击回调
  final void Function(DateTime seletedDate, int seletedDays)? onSelectDayRang;

  /// 选择确认按钮 title 回调
  final String Function(
          DateTime? selectStartTime, DateTime? selectEndTime, int seletedDays)?
      onSelectDayTitle;
  
```

### DEMO

```dart
TWCalendarList(
      firstDate: TWCalendarTool.tomorrow,
      lastDate: DateTime(2022, 11, 21),
      selectedStartDate: DateTime(2022, 9, 2),
      selectedEndDate: DateTime(2022, 9, 10),
      monthBodyHeight: 300.w,
      seletedMode: TWCalendarListSeletedMode.singleSerial,
      headerView: Container(
        alignment: Alignment.center,
        height: 55.w,
        child: Text(
          '日历组件',
          style: TextStyle(
            color: TWColors.tw333333,
            fontSize: 18.w,
          ),
        ),
      ),
      onSelectDayRang: ((seletedDate, seletedDays) {
        print('seletedDate : $seletedDate, seletedDays : $seletedDays');
      }),
      onSelectFinish: (selectStartTime, selectEndTime) {
        print(
            'selectStartTime : $selectStartTime, selectEndTime : $selectEndTime');
        Navigator.pop(context);
      },
    )
```

## 感谢

参考及修改至 demo: [flutter_calendar_list](https://github.com/heruijun/flutter_calendar_list)