---
title: Firebase的使用集合
date: 2018-11-21 19:03:03
categories: "iOS"
tags:
  - iOS
---

# Firebase
> Firebase是一家实时后端数据库创业公司，它能帮助开发者很快的写出Web端和移动端的应用。自2014年10月Google收购Firebase以来，用户可以在更方便地使用Firebase的同时，结合[Google的云服务](https://firebase.google.cn/)

## 前言
因为我们公司是用户群是在台湾，所以在第三方库的选型上选择了Firebase。因为它拥有较丰富的第三方库集。


## 基础接入

1. 创建Firebase开发者账户[注册地址](https://www.firebase.com/login/)
2. 安装Firebase.framework
通过Cocopod 安装
```
pod 'Firebase/Core'
```
3.添加Firebase配置文件 [将 Firebase 添加到您的 iOS 项目](https://firebase.google.com/docs/ios/setup?authuser=0)
- 在控制台中创建项目后，下载配置文件GoogleService-Info.plist，移动加入项目中
- 在didFinishLaunchingWithOptions 初始化firebase

```
[FIRApp configure]; 
```


#### 目前支持Firebase服务有

pod | 服务
---|---
pod 'Firebase/Core' | 必备库和 Analytics
pod 'Firebase/AdMob' | AdMob
pod 'Firebase/Messaging' | 必备库和 Analytics
pod 'Firebase/AdMob' | AdMob
pod 'Firebase/Database'	| 实时数据库
pod 'Firebase/Invites'	| 邀请
pod 'Firebase/DynamicLinks'| 	动态链接
pod 'Fabric'，pod 'Crashlytics' |	Crashlytics
pod 'Firebase/RemoteConfig'	|远程配置
pod 'Firebase/Auth' |	身份验证
pod 'Firebase/Storage' |	存储
pod 'Firebase/Performance' |	性能监控
pod 'Firebase/Firestore' |	Cloud Firestore
pod 'Firebase/Functions' |	Cloud Functions for Firebase 客户端 SDK
pod 'Firebase/MLVision' |	ML Kit Vision API
pod 'Firebase/MLVisionLabelModel'	| 机器学习套件（基于设备的标签检测）
pod 'Firebase/MLVisionBarcodeModel'	| 机器学习套件（基于设备的条形码扫描）
pod 'Firebase/MLVisionTextModel'	| 机器学习套件（基于设备的文字识别）
pod 'Firebase/MLVisionFaceModel'	| 机器学习套件（基于设备的面部检测）


## 基于Firebase下的GA统计
1.配合GTM SDK 将firebase采集到的数据发送到Google Analytics中

> 添加GTM SDK 与 Google Analytics SDK
```
pod 'GoogleIDFASupport'  // Google Analytics 
pod 'GoogleTagManager'   // GTM SDK
```
- 登录GTM的平台 [地址](https://tagmanager.google.com/?hl=zh_CN#/org/xM83TFWlQfiVaBKyzHY6Zw)

![](https://zeqinjie.github.io/images/2018/Firebase的使用集合/1.png)

- 创建触发器

![](https://zeqinjie.github.io/images/2018/Firebase的使用集合/2.png)

- 创建代码  GA-ID变量即是 我们的Google Analytics平台的ID

![](https://zeqinjie.github.io/images/2018/Firebase的使用集合/3.png)

> 简述：我们先在 GTM container 网页上设定想追踪的 tag（标籤），每个 tag 会包含一个 trigger（触发条件），当 Firebase 发送事件后 GTM 会去比对这个事件是不是符合某个 tag 的触发条件，在这次的例子中就会将数据发送到 GA 报表上。触发条件比较灵活，可以自定义。包括维度的划分。这里就不再详细介绍。



> firebase的发送函数是

```
//触发器事件名称
#define Event_Trigger    @"event_trigger_str"
// 统计类别
#define Event_Category    @"event_category_str"
// 统计动作
#define Event_Action    @"event_action_str"
// 统计标签
#define Event_Label    @"event_label_str"
              
// 即可将统计事件发送到GA中              
+ (void)fireBaseEvent:(NSString *)category
               action:(NSString *)action
                label:(NSString *)label{
    [FIRAnalytics logEventWithName:Event_Param_Trigger parameters:@{Event_Param_Category:category,Event_Param_Action:action,Event_Param_Label:label}];
}           
```

## FCM推送
1. 通过cocopod库设置Fcm sdk, fcm 中支持[通知消息和数据消息](https://firebase.google.com/docs/cloud-messaging/concept-options?authuser=0)
2. 在"专案设置-设置-Cloud Messaging"中上传开发者凭证到fcm平台

![](https://zeqinjie.github.io/images/2018/Firebase的使用集合/4.png)

3. 配置代码如下

```
//通过Cocopod导入
pod 'Firebase/Messaging'
```


```
// 第一步设置代理
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions{
    //配置FCM 代理
    [FIRMessaging messaging].delegate = self;
}


- (void)applicationDidEnterBackground:(UIApplication *)application {
    [self disconnectToFCM];
}

- (void)applicationDidBecomeActive:(UIApplication *)application{
    [self connectToFCM];
}

// 注册TOKEN
- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken{
   #if DEBUG
    [[FIRMessaging messaging] setAPNSToken:deviceToken type:FIRMessagingAPNSTokenTypeSandbox];
    #else
    [[FIRMessaging messaging] setAPNSToken:deviceToken type:FIRMessagingAPNSTokenTypeProd];
    #endif
}


#pragma mark - RemoteMessage Connect
// 链接fcm
- (void)connectToFCM{
    NSString *refreshedToken = [[FIRInstanceID instanceID] token];
    DLog(@"InstanceID token: %@", refreshedToken);
    [FIRMessaging messaging].shouldEstablishDirectChannel = YES;
}

// 断开fcm
- (void)disconnectToFCM{
    [FIRMessaging messaging].shouldEstablishDirectChannel = NO;
}


#pragma mark - FIRMessagingRemoteMessage Delegate
//获取fcmtoken回调函数
- (void)messaging:(nonnull FIRMessaging *)messaging didReceiveRegistrationToken:(nonnull NSString *)fcmToken{
    DLog(@"didReceiveRegistrationToken fcmToken = %@",fcmToken);
    [self connectToFCM];
    // 注意 将fcm 发送到自己的后端采集token及用户的设备id,用于发送推送
    // .......
}

//注意：fcm data 数据消息的接收回调，如果是通知消息回调，走的是Apple 的api此处不详述

//ios10 之前didReceiveRegistrationToken
- (void)applicationReceivedRemoteMessage:(nonnull FIRMessagingRemoteMessage *)remoteMessage{
    DLog(@"RemoteMessageDeleagte remoteMessage = %@",remoteMessage);
}
//iOS10 及之后
- (void)messaging:(nonnull FIRMessaging *)messaging didReceiveMessage:(nonnull FIRMessagingRemoteMessage *)remoteMessage{
    DLog(@"RemoteMessageDeleagte didReceiveMessage = %@",remoteMessage);//fcm data 数据没有带前缀gcm.notification.
}
```

## Performace监控

![](https://zeqinjie.github.io/images/2018/Firebase的使用集合/5.png)

firebase 的performance能采集到数据有[链接](https://support.google.com/firebase/answer/6318039?hl=zh-Hans)
- 启动速度
- 网络呼叫成功率
- 網路回應 MIME 類型
- 網路回應延遲時間
- 不同畫面的顯示速度緩慢資料 （可以自定义采集内容）
```
//通过Cocopod导入
pod 'Firebase/Performance'
```
 > performance 只要解决导入库,即可。默认会在[FIRApp configure]; 已初始化
 
#### 关于自定义采集 page_speed

![](https://zeqinjie.github.io/images/2018/Firebase的使用集合/6.png)

```
// 自定义采集是通过setValue:forAttribute 方式设置采集属性及值
+ (void)performaceLoadTimeWithName:(NSString *)name comeDate:(NSDate *)comeDate{
        FIRTrace *trace = [FIRPerformance startTraceWithName:name];
        NSTimeInterval time = fabs([comeDate timeIntervalSinceNow]);
        [trace setValue:[NSString stringWithFormat:@"%.2f秒",time] forAttribute:@"page_speed"];
        DLog(@"PageName = %@,时间差(秒) = %.2f",name,time);
        [trace stop];
}
```





