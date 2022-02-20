---
title: 自定义 iOS14 PageControl 控件新功能
date: 2021-09-28 18:38:37
tags:
 - iOS
---


# 前言

> 产品需求有提一个 PageControl 功能交互效果，发现是 iOS14 新特性。即是在`iOS 14`之后`UIPageControl` 的圆点个数将不再受限制，它会自适应去显示圆点。比如设置 100 个圆点，当前一页无法展示出全部圆点时，它会分页展示，同时两边的圆点会渐渐变小。具体可以了解 [iOS14 UIPageControl变化](https://www.jianshu.com/p/e21985a33ceb) 。但问题是 iOS14 以下怎么办！没办法，只能自己封装一个。

Github：[ ZQEndlessPageControl](https://github.com/zeqinjie/ZQEndlessPageControl)

## 安装
```
pod 'ZQEndlessPageControl'
```

## 功能
- 为了适配 iOS14 的 pageControl 自定义的控件 ☑️
- 设置大小、间隙、背景颜色 ☑️
- 设置边框大小，颜色 ☑️
- 设置大、中、小，三种状态的缩放系数 ☑️
- 设置背景图 ☑️


## 效果

<img src="https://github.com/zeqinjie/blog/blob/master/source/images/2021/%E8%87%AA%E5%AE%9A%E4%B9%89%20iOS14%20PageControl%20%E6%8E%A7%E4%BB%B6%E6%96%B0%E5%8A%9F%E8%83%BD/1.gif" alt="show" />


## 使用

### 配置对象

```swift
public struct ZQEndlessPageControlConfiguration {
    /// 总共点个数
    var numberOfDots: Int
    /// 支持最大展示多少个点
    var maxNumberOfDots: ZQEndlessPageControlMaxNumberOfDots
    /// 选择点的颜色
    var selectedDotColor: UIColor
    /// 未选择点的颜色
    var unselectedDotColor: UIColor
    /// 点大小
    var dotSize: CGFloat
    /// 点间隙
    var spacing: CGFloat
    /// 未选中的缩放系数
    var unselectedScale: CGFloat
    /// 选中的缩放系数
    var selectedScale: CGFloat
    /// 最小的点的缩放系数
    var smallScale: CGFloat
    /// 点边框色
    var dotBorderColor: UIColor?
}
```

### DEMO

```swift
import ZQEndlessPageControl

fileprivate let indicatorPageControl1: ZQEndlessPageControlIndicator = ZQEndlessPageControlIndicator()
self.view.addSubview(indicatorPageControl1)
indicatorPageControl1.snp.makeConstraints { (make) in
    make.center.equalToSuperview()
}
    
let indicatorConfigure = ZQEndlessPageControlConfiguration(
    numberOfDots: Metric.indicatorPageDotNum,
    dotSize: 8,
    dotBorderColor: UIColor.black.withAlphaComponent(0.19)
)
indicatorPageControl1.setup(configuration: indicatorConfigure)
```

