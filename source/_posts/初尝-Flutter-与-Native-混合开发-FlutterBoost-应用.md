---
title: 初尝 Flutter 与 Native 混合开发 - FlutterBoost 应用
date: 2020-08-14 17:38:13
categories: "iOS"
tags:
  - iOS
  - Flutter
---


## 前言
> 在跨平台开发中 `Flutter` 优势很明显，简单总结 : <br>
> 1. 接近原生的性能  
> 2. 热重载  
> 3. 丰富的组件<br>
> 考虑项目有大量原生业务，我们也不可能基于 Flutter 重构所有的业务。<br>因此只能在原有的基础上混合使用 Flutter 来开发新业务或重构旧业务。<br>参考闲鱼，哈罗等，他们也提供相应的解决混合开发方案 [flutter_boost](https://github.com/alibaba/flutter_boost) 和 [flutter_thrio](https://github.com/hellobike/flutter_thrio)。 <br>目前我们采用 flutter_boost 作为我们 tw591 项目的解决方案。

## Flutter 混编现有项目
> 正式接入：

1. 创建 Flutter 项目`flutter create -t module flutter_module`, 建议创建完可以执行用 AndoirdStudio 编译执行，看是否正常,不过记得删除生成的ios 文件，否则会遇第4条说的flutter_export_environment 问题

2. 将创建的 flutter_module 项目到自己目标仓库中

3. 在 Podfile 文件添加如下脚本

```
// 注意：![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/913f7748bf1841e3827501c40a6ff75f~tplv-k3u1fbpfcp-zoom-1.image)这个指定路径即是你的 flutter_module 项目的目标路径
flutter_application_path = '../../TWFlutter591/flutter_module'
load File.join(flutter_application_path, '.ios', 'Flutter', 'podhelper.rb')

target 'twhouse' do
install_all_flutter_pods(flutter_application_path)
end

```

4. 注意项目报 flutter_export_environment.sh 文件路径错误
> 处理方式是删除flutter_module/ios 文件
用AndoirdStudio 或者 VSCode 运行一遍，产生新的 flutter_export_environment.sh 文件

![](/images/2020/初尝-Flutter-与-Native-混合开发-FlutterBoost-应用/1.png)

5. 集成闲鱼 flutter_boost 混合开发方案
- 注意：flutter_boost 版本需要和我们 flutter 版本对应，例如：flutter_boost:v1.17.1-hotfixes 对应的flutter sdk：1.17.1 
- 使用 Flutter --version 命令查看当前版本
- 打开 flutter_module 文件夹 下pubspec.yaml 文件，添加依赖
```
  flutter_boost:
    git:
        url: 'https://github.com/alibaba/flutter_boost.git'
        ref: 'v1.17.1-hotfixes'
```
- 执行命令 `flutter packages get`
- 最后在项目中执行 `pod install --repo-update`

![](/images/2020/初尝-Flutter-与-Native-混合开发-FlutterBoost-应用/2.png)

- 执行 install 后原生项目 Flutter 目录，会将我们 Flutter 项目打包成 framework

![](/images/2020/初尝-Flutter-与-Native-混合开发-FlutterBoost-应用/3.png)

## Native 部分

![](/images/2020/初尝-Flutter-与-Native-混合开发-FlutterBoost-应用/4.png)

> - TWFlutterUtil 单例类用来注册 flutter 引擎，同时封装一层调用，避免 FlutterBoostPlugin 直接调用
> - TWFlutterPlatformRouter 基于 Flutter_boost DEMO 提供的基础上，根据我们项目业务做了调整，主要是 flutter 与 Native 平台交互的 Router
> - TWFlutterJumpUtil 主要是处理 Flutter 与 Native 相互路由跳转间的业务逻辑
> - TWFlutterNativePageName 定义页面路由名称
> - TWFlutterNativeEventUtil 主要是处理 Flutter 与 Native 事件的业务逻辑

### 示例 TWFlutterUtil 部分
> `TWFlutterUtil.swift`	

```swift
import UIKit
import flutter_boost
@objcMembers class TWFlutterUtil: NSObject {
    
    static let shareInstance = TWFlutterUtil()
    var router: TWFlutterPlatformRouter?
    var engine: FlutterEngine?
    
    /// 注册 flutter 引擎
    func registerFlutter() {
        router = TWFlutterPlatformRouter()
        guard let router = router else {return}
        FlutterBoostPlugin.sharedInstance().startFlutter(with: router) {[weak self](engine) in
            self?.engine = engine
            HouseTool.dispatch(afterTime: 2.0) {
                TWFlutterNativeEventUtil.sendConfigureInfo()
            }
        }
    }
    
    // MARK: - sendEvent
    
    /**
     * Native层往Dart层发送事件，事件名称通过eventName指定
     *
     * @param eventName 事件名称
     * @param arguments 参数
     */
    open class func sendEvent(_ eventName: String, arguments: [AnyHashable: Any]) {
        FlutterBoostPlugin.sharedInstance().sendEvent(eventName, arguments: arguments)
    }
    
    /**
    * 添加监听Dart层调用Native层的事件
    *
    * @param name 事件名称
    * @param listner 事件监听器
    */
    open class func addEventListener(_ listener: @escaping FLBEventListener, name: String) -> FLBVoidCallback {
        return FlutterBoostPlugin.sharedInstance().addEventListener(listener, forName: name)
    }
    
    // MARK: - open/close Page
    
    /**
     * 关闭页面，混合栈推荐使用的用于操作页面的接口
     *
     * @param uniqueId 关闭的页面唯一ID符
     * @param resultData 页面要返回的结果（给上一个页面），会作为页面返回函数的回调参数
     * @param exts 额外参数
     * @param completion 注意必传 关闭页面的即时回调，页面一旦关闭即回调
     */
    open class func close(_ uniqueId: String, result resultData: [AnyHashable: Any], exts: [AnyHashable: Any], completion: @escaping (Bool) -> Void) {
        FlutterBoostPlugin.close(uniqueId, result: resultData, exts: exts, completion: completion)
    }
    
    /**
     * 打开新页面（默认以push方式），混合栈推荐使用的用于操作页面的接口；通过urlParams可以设置为以present方式打开页面：urlParams:@{@"present":@(YES)}
     *
     * @param url 打开的页面资源定位符
     * @param urlParams 传入页面的参数; 若有特殊逻辑，可以通过这个参数设置回调的id
     * @param exts 额外参数
     * @param resultCallback 当页面结束返回时执行的回调，通过这个回调可以取得页面的返回数据，如close函数传入的resultData
     * @param completion 注意必传 打开页面的即时回调，页面一旦打开即回调
     */
    open class func open(_ url: String, urlParams: [AnyHashable: Any], exts: [AnyHashable: Any], onPageFinished resultCallback: @escaping ([AnyHashable: Any]) -> Void, completion: @escaping (Bool) -> Void) {
        FlutterBoostPlugin.open(url, urlParams: urlParams, exts: exts, onPageFinished: resultCallback, completion: completion)
    }
    
    /**
     * Present方式打开新页面，混合栈推荐使用的用于操作页面的接口
     *
     * @param url 打开的页面资源定位符
     * @param urlParams 传入页面的参数; 若有特殊逻辑，可以通过这个参数设置回调的id
     * @param exts 额外参数
     * @param resultCallback 当页面结束返回时执行的回调，通过这个回调可以取得页面的返回数据，如close函数传入的resultData
     * @param completion 注意必传 打开页面的即时回调，页面一旦打开即回调
     */
    open class func present(_ url: String, urlParams: [AnyHashable: Any], exts: [AnyHashable: Any], onPageFinished resultCallback: @escaping ([AnyHashable: Any]) -> Void, completion: @escaping (Bool) -> Void) {
        FlutterBoostPlugin.present(url, urlParams: urlParams, exts: exts, onPageFinished: resultCallback, completion: completion)
    }
    
}
```

### Native 打开 Flutter 页面

> 页面采用文件 `page` `页面类名`，作为注册 id

```swift
//在 iOS 中 TWFlutterUtil 打开更多页面
TWFlutterUtil.open("TWMorePage",
                           urlParams: ["isDebug":API_DEBUG == 1 ? "1" : "0"],
                           exts: ["isHideNavigationBar":"1"],
                           onPageFinished: { (result) in
        }) { (finish) in
        }
    
```



## Flutter 部分

![](/images/2020/初尝-Flutter-与-Native-混合开发-FlutterBoost-应用/5.png)

> - TWFlutterBoostPage 主要是 flutter 项目的入口页面
> - TWRouterBoost 用于注册原生调用 flutter 的路由

### 示例 TWFlutterBoostPage 部分
> `TWFlutterBoostPage`

```dart
import 'package:flutter/material.dart';
import 'package:flutter_boost/flutter_boost.dart';
import 'package:flutter_module/common/tw_app_color.dart';
import 'package:flutter_module/router/tw_router_boost.dart';

class TWFlutterBoostPage extends StatefulWidget {
  @override
  _TWFlutterBoostAppState createState() => _TWFlutterBoostAppState();
}

class _TWFlutterBoostAppState extends State<TWFlutterBoostApp> {
  @override
  void initState() {
    super.initState();
    //初始化注册...
    TWRouterBoost routerBoost  = TWRouterBoost();
    routerBoost.registerRouter();
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
        title: 'Flutter Boost example',
        //（2）初始化路由
        debugShowCheckedModeBanner: true,
        theme: ThemeData(
          primaryColor: TWAppColor.tw_ff7f00,
          dividerColor: TWAppColor.tw_eeeeee,
        ),
        builder: FlutterBoost.init(postPush: _onRoutePushed),
        home: Container(
            color:Colors.white
        ));
  }

  void _onRoutePushed(
      String pageName,
      String uniqueId,
      Map<String, dynamic> params,
      Route<dynamic> route,
      Future<dynamic> _,
      ) {

  }
}

```

> `TWRouterBoost`

```dart
import 'package:flutter/material.dart';
import 'package:flutter_boost/flutter_boost.dart';
import 'package:flutter_module/features/tw_more_page.dart';
import 'package:flutter_module/util/tw_log.dart';

class TWRouterBoost extends NavigatorObserver{
  /// 初始化注册路由...
  void registerRouter() {
    FlutterBoost.singleton.registerPageBuilders(<String, PageBuilder>{
      'TWMorePage': (String pageName, Map<String, dynamic> params, String _) {
        String isDebug = params["isDebug"];
        String isDark = params["isDark"];
        String isLogin = params["isLogin"];
        print("flutter_more_page params123:$params , $isDebug");
        return TWMorePage(isLogin: isLogin == "1" ? true : false, isDebug: isDebug == "1" ? true : false,isDark: isDark == "1" ? true : false,);
      },
    });
    FlutterBoost.singleton.addBoostNavigatorObserver(this);
  }

  ///************** NavigatorObserver Method *******************/
  void didPush(Route<dynamic> route, Route<dynamic> previousRoute) {
    TWLog("flutterboost#didPush");
  }

  void didPop(Route<dynamic> route, Route<dynamic> previousRoute) {
    TWLog("flutterboost#didPop");
  }

  void didRemove(Route<dynamic> route, Route<dynamic> previousRoute) {
    TWLog("flutterboost#didRemove");
  }

  void didReplace({Route<dynamic> newRoute, Route<dynamic> oldRoute}) {
    TWLog("flutterboost#didReplace");
  }
}

```

### Flutter 打开 Native 页面
> 采用项目路由方式打开
> 注意：`TWFlutterOpenNative` 标识 ID ，即是标记从 Flutter 页面打开 Native 页面 ，打开 Native 页面路由 exts 字段
> 路由部分，建议是原来项目有的，统一Android 和 iOS 的路由规则标准，我们 iOS 采用 MGJRouter , Android 采用 ARouter。定义一套项目的路由 URL 标准

```
  ///************** Private Method *******************/
  void clickAction(int index) {
    TWLog("點擊 index = $index");
    /// 打开评价，
    String open_page_url = "app:///xxxx/more_page?entrance=more_evalue&pushAnimation=1";
	/// 闲鱼库打开 原生api
    FlutterBoost.singleton
        .open('TWFlutterOpenNative',exts: {"open_page_url":xxxx_url})
        .then((value) => print('call me when page is finished. did recieve native route result $value'));
 }
```

> 用 flutter 重构更多页面, 体验感觉和原生差不多

![](/images/2020/初尝-Flutter-与-Native-混合开发-FlutterBoost-应用/6.jpg)



## 优秀的第三方库

> 为了减少与原生桥接，尽量让 Android 和 iOS 能公用一套，那么接下来要做的是一些基础设施建设了。包括 网络库，状态管理，图片缓存，数据缓存等...

| 分类          | 地址           | star           |
| -------------- | -------------- | -------------- |
| 网络库 |  [Dio](https://github.com/flutterchina/dio) | 8.2k |
| 状态管理 | [bloc](https://github.com/felangel/bloc) | 5.3k |
| 状态管理 | [provider](https://github.com/rrousselGit/provider) | 2.8k |
| 数据库 | [sqflite](https://github.com/tekartik/sqflite) | 1.8k |
| 图片缓存库 | [flutter_cached_network_image](https://github.com/Baseflow/flutter_cached_network_image) | 1.4k |
| 刷新控件 | [flutter_easyrefresh](https://github.com/xuelongqy/flutter_easyrefresh) | 2.2k |
| 刷新控件 | [flutter_pulltorefresh](https://github.com/peng8350/flutter_pulltorefresh) | 1.6k |
| 轮播 | [flutter_swiper](https://github.com/best-flutter/flutter_swiper) | 2.6k |
| 本地通知 | [flutter_local_notifications](https://github.com/MaikuB/flutter_local_notifications) | 1.1k |
| 菜单 | [flutter_slidable](https://github.com/letsar/flutter_slidable) | 1.4k |
| 地图 | [flutter_amap](https://github.com/best-flutter/flutter_amap) | 137 |
| 地图 | [flutter_amap_location](https://github.com/best-flutter/flutter_amap_location) | 249 |
| 调试工具 | [flutter_slidable](https://github.com/learn-flutter-dev/flutter_flipperkit) | 308 |

## 目前项目结构
```
flutter_module
├──images #图片资源
├──asset  #本地资源
├──lib   #项目代码
	├── features    # 业务模块
	|   ├── home    # 首页	
	|   ├── search  # 搜寻	
	|   ├── news 	# 新闻	
	|   └── mine 	# 我的
	|       └──more # 更多 (例：更多页面)
	|   		├── model 	# 模型
	|   		├── view 	# 视图
	|   		├── tool 	# 工具	
	|   		└── page 	# 页面  	
	|   	
	├── page        # 入口主页面
	├── common      # 通用组件,头文件,定义常量 (例：颜色常量)
	├── util        # 工具类,http工具,公共方法   
	├── widgets     # 各种封装的可复用组件  
	├── store       # 数据相关 缓存库等
	├── blocs       # 状态管理相关	
	├── config      # 配置中心 (例：主题颜色，debug 线上切换方法等)
	└── router      # 路由
	     └──  页面映射配置、observe 方法导出
```



