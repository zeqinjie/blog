---
title: iOS怎么推送统计到达率
date: 2018-07-04 19:03:02
categories: "iOS"
tags:
  - iOS
---

[掘金地址](https://juejin.im/post/5b3ca2d05188251afe7b66be)

## ios摘要
* [iOS10里的通知与推送](http://www.cocoachina.com/ios/20170126/18618.html)
* [国内 90%以上的 iOS 开发者，对 APNs 的认识都是错的](https://www.jianshu.com/p/ace1b422bad4)
## 体系图
![](https://zeqinjie.github.io/images/2018/iOS怎么推送统计到达率/1.png)


## 直接进入正题如何统计到达

###  在iOS10中新增两个拓展

1. 其中的一个拓展UNNotificationServiceExtension 通知服务扩展
* UNNotificationServiceExtension 是修改远程推送携带的内容。
* UNNotificationServiceExtension 类 可以让开发者自定义推送展示的内容。你可以用 extension 修改推送内容和下载推送相关的资源。你可以在extension 中解密和加密的数据或下载推送相关的图片

2.因此在这里可以提交到达的请求注意是这里处理时间只有30秒。为了不影响达到统计接口。一般会先发统计请求。同时通过缓存减少图片的请求。及图片的大小处理。

```objc
@interface NotificationService ()<NSURLSessionDelegate>

@property (nonatomic, strong) void (^contentHandler)(UNNotificationContent *contentToDeliver);
@property (nonatomic, strong) UNMutableNotificationContent *bestAttemptContent;

@end

@implementation NotificationService

- (void)didReceiveNotificationRequest:(UNNotificationRequest *)request withContentHandler:(void (^)(UNNotificationContent * _Nonnull))contentHandler {
    
    self.contentHandler = contentHandler;
    self.bestAttemptContent = [request.content mutableCopy];
    NSDictionary *userInfo = request.content.userInfo;
    
    NSString *iconUrl = userInfo[[self fcmPushStrFromStr:@"small_image"]];//小图
    NSString *imageUrl = userInfo[[self fcmPushStrFromStr:@"image"]];//大图
    
    //重点这里发送给后端到达统计
    [ProfessionTool sendFcmPushStatUserInfo:userInfo];//统计到达数
    
    //这里是处理富文本图片 ，媒体视频等
    UNNotificationAttachment *iconAtt = [self attachmentWithUrl:iconUrl fileName:@"small_image"];//待优化缓存方式
    UNNotificationAttachment *imageAtt = [self attachmentWithUrl:imageUrl fileName:@"image"];
    NSMutableArray *attArr = [NSMutableArray array];
    if (iconAtt) [attArr addObject:iconAtt];
    if (imageAtt) [attArr addObject:imageAtt];
    self.bestAttemptContent.attachments = attArr;
    self.contentHandler(self.bestAttemptContent);

}

- (UNNotificationAttachment *)attachmentWithUrl:(NSString *)url fileName:(NSString *)fileName{
    if (url) {
        UIImage *iconImg = [self getImageFromURL:url];
        NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
        NSString *documentsDirectoryPath = [paths firstObject];
        NSString *iconPath = [self saveImage:iconImg withFileName:fileName ofType:@"png" inDirectory:documentsDirectoryPath];
        if (iconPath && ![iconPath isEqualToString:@""]) {
            UNNotificationAttachment * attachment = [UNNotificationAttachment attachmentWithIdentifier:fileName URL:[NSURL URLWithString:[@"file://" stringByAppendingString:iconPath]] options:nil error:nil];
            if (attachment) {
                return attachment;
            }
        }
    }
    return nil;
}

- (void)serviceExtensionTimeWillExpire {
    // Called just before the extension will be terminated by the system.
    // Use this as an opportunity to deliver your "best attempt" at modified content, otherwise the original push payload will be used.
    self.contentHandler(self.bestAttemptContent);
}

#pragma mark - private Method
- (UIImage *)getImageFromURL:(NSString *)fileURL {
    DLog(@"执行图片下载函数");
    UIImage * result;
    //dataWithContentsOfURL方法需要https连接
    NSData * data = [NSData dataWithContentsOfURL:[NSURL URLWithString:fileURL]];
    result = [UIImage imageWithData:data];
    
    return result;
}

//将所下载的图片保存到本地
- (NSString *)saveImage:(UIImage *)image withFileName:(NSString *)imageName ofType:(NSString *)extension inDirectory:(NSString *)directoryPath {
    NSString *urlStr = @"";
    if ([[extension lowercaseString] isEqualToString:@"png"]){
        urlStr = [directoryPath stringByAppendingPathComponent:[NSString stringWithFormat:@"%@.%@", imageName, @"png"]];
        [UIImagePNGRepresentation(image) writeToFile:urlStr options:NSAtomicWrite error:nil];
        
    } else if ([[extension lowercaseString] isEqualToString:@"jpg"] ||
               [[extension lowercaseString] isEqualToString:@"jpeg"]){
        urlStr = [directoryPath stringByAppendingPathComponent:[NSString stringWithFormat:@"%@.%@", imageName, @"jpg"]];
        [UIImageJPEGRepresentation(image, 1.0) writeToFile:urlStr options:NSAtomicWrite error:nil];
        
    } else{
        DLog(@"extension error");
    }
    return urlStr;
}

//因项目中使用google 的FCM推送收到的推送字段有点不一样
- (NSString *)fcmPushStrFromStr:(NSString *)str{
    NSString *resultStr = str;
    NSString *fcmPre = @"gcm.notification.";
    if (![resultStr containsString:fcmPre]) {
        resultStr = [NSString stringWithFormat:@"%@%@",fcmPre,resultStr];
    }
    return resultStr;
}
```
3.服务端配置内容需要添加"mutable-content":1 （富文本推送）
很多人会搞混"content-available":1（静默推送）

4.当然了google FCM的富文本推送是"mutable_content":"true"
国内用的比较多的是激光JPush.百度推送等。后续会写篇google FCM 推送的配置方式

---


```
拿到的回调内容
//iOS10 之前
{
"aps" : {
    "alert" : "title",
    "badge" : 1,
    "sound":"default"
        },
}

//iOS10 新增的文案多样性
{
"aps" : {
    "alert" : { 
         "title" : "title", 
         "subtitle" : "subtitle",         
         "body" : "body"
                },
    "badge" : 1,
    "sound":"default"
        },
}

```

```objc
//接收到推送回调函数
//iOS1O之前的 

//NS_AVAILABLE_IOS(3_0)
- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo{
    //最开始的接收推送回调
}

//NS_AVAILABLE_IOS(7_0)
- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo fetchCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler{
    //这里亦可接收静默推送回调   
}

//iOS10之后统一 
//  iOS10特性。App在前台获取通知

- (void)userNotificationCenter:(UNUserNotificationCenter *)center willPresentNotification:(UNNotification*)notification withCompletionHandler:(void (^)(UNNotificationPresentationOptions options))completionHandler {

   completionHandler(UNNotificationPresentationOptionAlert);

}

//点击通知进入App
- (void)userNotificationCenter:(UNUserNotificationCenter *)center didReceiveNotificationResponse:(UNNotificationResponse *)response withCompletionHandler:(void (^)())completionHandler {
    
}

```

5.与之前不同是ios10前台运行也能收到推送通知栏同样在上面userNotificationCenter接收处理点击事件

6.当然本人曾在低于iOS10版本通用过pod 'JDStatusBarNotification'库模仿一个推送通知栏

7.测试发送iOS10的富文本推送低于iOS10版本的的用户只能收到title. subtitle内容是获取不到的。


8.Target的调试 将项目运行起来，然后发送一条推送之后，激活Service Extension，在XCode -DEBUG下Attach to Process 选择对于的target
