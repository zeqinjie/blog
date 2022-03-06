---
title: App Clip 尝鲜
date: 2020-12-29 16:40:34
categories: "iOS"
tags:
	- iOS
---

# iOS App Clip 尝鲜

![](/images/2020/App-Clip-尝鲜/1.png)


> 苹果在 WWDC2020 上发布了 App Clip，有媒体叫做“苹果小程序”。虽然 Clip 在产品理念上和小程序有相似之处，但是在技术实现层面却是截然不同的东西。



# 什么是 App Clip?
> App Clip 是 App 应用程序的轻量级版本，可在用户需要的位置和时间提供某些功能。



## 简单理解如图


![](/images/2020/App-Clip-尝鲜/2.png)

## 触发方式


- 在物理位置扫描 NFC 标签或二维码
- 选择 Siri 提供的基于位置的建议，例如定位
- 在 Map 应用中点击链接
- 在网站上点击一个智能的 App Banner
- 点击某人在 iMessage 应用中共享的链接

![](/images/2020/App-Clip-尝鲜/3.png)


# App Clip 限制


- 只能用户主动唤起，不存在用户不小心点击某个 APP 唤起
- App Clip 的大小被限制在了 10MB 以下
- App Clip 的生命周期由操作系统全权接管
- App Clip 无法把数据存储到 keychain 来共享给完整版App，数据共用只能通过一个共享的 app container 或者 user defaults 
- App Clip 无法获取到设备名（[UIDevice name]）和广告标识符（identifierForVendor），其对应的 API 会返回空串
- App Clip 不能执行后台任务：例如，在App Clip不使用时，用 NSURLSession 进行后台联网或保持蓝牙连接

## App Clip 在框架限制

![](/images/2020/App-Clip-尝鲜/9.jpeg)

## 用户隐私保护

### 有条件的定位权限

- App Clip 的通知和定位权限是免申请，用户可以主动关闭：通知在8小时内有效，位置只能获取一次，第二天凌晨4点会自动重置。如需再度使用，可通过弹窗申请。
### 不支持以下用户数据访问

- 运动和健身数据
- 苹果音乐与媒体
- 来自通讯录，文件，消息，提醒和照片等应用程序的数据

# 网易严选 App Clip

![](/images/2020/App-Clip-尝鲜/4.png)

## 体验
[![RPReplay_Final1608798232 (1).MP4 (11.82MB)](https://gw.alipayobjects.com/mdn/prod_resou/afts/img/A*NNs6TKOR3isAAAAAAAAAAABkARQnAQ)]()

# 三种轻应用的对比



| 分类 | 免安装 | 需要审核 | 大小限制 | 系统要求 | 来源 |
| :---: | :---: | :---: | :---: | :---: | :---: |
| App Clip - iOS | ☑️  | ☑️  | <= 10 M | iOS 14 | 二维码，safari 浏览器， nfc， 短信 |
| Install Apps - android | ☑️  | ☑️  | <= 4 M | Android 6.0(API 23) | 浏览器, 短信 |
| 微信小程序 | ☑️  | ☑️  | < 12M | - | 微信 app |



# 项目接入 App Clip

## 创建 App Clip Target
> App Clip 是依附于原生 APP，建 App Clip Target 
> - Xcode 菜单项File-> New-> Target…

![](/images/2020/App-Clip-尝鲜/5.png)

## 创建App Clips ID

![](/images/2020/App-Clip-尝鲜/66.png)
![](/images/2020/App-Clip-尝鲜/6.png)
![](/images/2020/App-Clip-尝鲜/88.png)


## 配置apple-app-site-association.json

> 假设你的开发者账号的Team Id是A123，Clips的 bundle id是 A123.com.xxxx.xxx.clips，主工程的bundle id 是 A123.com.xxxx.xxx，则配置如下

```json
{
    "appclips":{
        "apps":["A123.com.xxxx.xxx.clips"]
    },
    "applinks": {
    "apps": [],
    "details": [{
      	"appID":"A123.com.xxxx.xxx",
        "paths":[
        "xxxxx",
        ]
    	}
    ]
  }
}

```


## 前端页面添加meta标签


如果要在H5页面显示Clips入口，加上一段meta标签即可, 如果你的应用市场App id是 `123456`, Clips的 bundle id 是 `com.abc.clips` 则应该配置
```
<meta name="apple-itunes-app" content="app-id=123456,app-clip-bundle-id=com.abc.clips">
```


# 遇到问题
## 保持一致版本号
![](/images/2020/App-Clip-尝鲜/7.png)
![](/images/2020/App-Clip-尝鲜/14.png)

> Version 版本号需要主项目一致

## Bundle ID 命名规则
![](/images/2020/App-Clip-尝鲜/10.png)

> App clip 所包含的功能必须是 main app 的子集。App clip 的 bundle ID 必须是 main app 的 bundle ID 后缀加上 `.Clip` 



# 开发调试

- 可以选择Clip Target直接使用Xcode编译运行
- 可以使用真机扫描二维码，调起Clip卡片，但是前提是 Clip 要先在真机跑一遍。
   1. 手机点击`设置-开发者-Local Experiences-Register Local Experience`
   1. 输入域名、Clip的bundle id、标题、子标题，选择按钮标题、选择Clip弹出的卡片上的图片，然后点击存储即可。
   1. 将刚才输入的域名，去草料二维码等二维码生成网站生成一个二维码，然后手机相机扫描即可弹出卡片样式。
   1. 具体内容可以参考[官方文档](https://developer.apple.com/documentation/app_clips/testing_your_app_clip_s_launch_experience?language=objc)

![](/images/2020/App-Clip-尝鲜/13.png)






# 参考

- [苹果 App Clip 技术详解](https://juejin.cn/post/6844904199210139661)
- [学习 iOS14 新特性，教你如何创建一个优秀的 App Clip](https://juejin.cn/post/6904122939483160584)
- [iOS 14 APP Clips开发](https://juejin.cn/post/6899342253815234574)
- [一些关于 App Clips 的笔记](https://onevcat.com/2020/06/first-look-app-clips/)
- [官方文档](https://developer.apple.com/documentation/app_clips)
- [微信小程官方文档](https://developers.weixin.qq.com/miniprogram/dev/framework/subpackages.html)
