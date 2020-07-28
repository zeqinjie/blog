---
title: Firebase的使用集合（二）
date: 2018-12-06 19:03:02
categories: "iOS"
tags:
  - iOS
---

## [远程配置](https://firebase.google.com/docs/remote-config/)

### 简介
> Firebase 远程配置是一项云端服务，可让您更改应用的行为和外观，而无需用户下载应用更新。使用远程配置时，您可以创建应用内默认值，用于控制应用的行为和外观。之后，您便可以使用 Firebase 控制台或 Remote Config REST API，使得应用的所有用户或细分用户群获得不同于默认值的行为和外观。您的应用可控制何时安装更新，并能经常检查有无更新并安装更新，且对性能的影响微乎其微。[掘金地址](https://juejin.im/post/5c08df4c6fb9a04a0c2e3e07)

### 主要功能

- 将更改快速发布至应用的用户群
  - 您可以通过更改服务端参数值来更改应用的默认行为和外观。
  - 例如，您可以更改应用的布局或颜色主题背景以配合季节性促销，而无需发布应用更新。
- 针对细分用户群量身打造应用
  - 您可以使用远程配置为不同的细分用户群（按应用版本、按 Google Analytics for Firebase 受众群体、按语言及更多因素划分）提供多样化的应用用户体验。
- 运行 A/B 测试以改进您的应用
  - 您可以结合使用远程配置随机百分位定位和 Google Analytics for Firebase，在不同的细分用户群中针对应用的改进之处进行 A/B 测试，以便能够先验证这些改进之处，然后再将其推向整个用户群。


### 过程
```
//通过Cocopod导入
pod 'Firebase/RemoteConfig'，
```
> 与performance一样，默认会在[FIRApp configure]; 初始化 <br>
<!--> -FIRAnalyticsDebugDisabled-->
<!---FIRAnalyticsDebugEnabled-->

![](https://zeqinjie.github.io/images/2018/Firebase的使用集合（二）/1.png)

> Firebase   DebugView调试部分 <br>
> [√] FIRAnalyticsDebugDisabled : 调试关闭<br>
> [√] FIRAnalyticsDebugEnabled : 调试打开<br>

#### 创建参数及条件
![](https://zeqinjie.github.io/images/2018/Firebase的使用集合（二）/2.png)

Parameters ：参数列表
- 设定参数的指定条件
- 限制最多2000个参数

Conditions ：条件列表
- Remote Config 提供多种条件选项，比如操作系统,语言，国家地区，目标对象...
- 限制最多 500个条件

Parameters和Conditions的限制
- 参数键最多可包含 256 个字符，且必须以下划线或英文字母（A-Z、a-z）开头，还可以包含数字。一个项目中所有参数值字符串的总长度不能超过 80 万个字符

> [我们配置app_color这个参数，支持json字符串](https://firebase.google.com/docs/remote-config/parameters)
<br> [[FIRRemoteConfig remoteConfig]configValueForKey:@"app_color"]


#### app获取远程配置好的数据
```
- (void)firebaseRemoteConfigure{
    //过期时间。默认设置为60分钟
    NSTimeInterval duration = 3600;
#if DEBUG
    //配置设置，是否打开调试模式
    FIRRemoteConfigSettings *setting = [[FIRRemoteConfigSettings alloc]initWithDeveloperModeEnabled:YES];
    [FIRRemoteConfig remoteConfig].configSettings = setting;
    //这边测试调试，所以设置为0分钟
    duration = 0;
#endif
    // 设置60分钟触发更新  3600
    [[FIRRemoteConfig remoteConfig] fetchWithExpirationDuration:duration completionHandler:^(FIRRemoteConfigFetchStatus status, NSError * _Nullable error) {
        if(!error){
            DLog(@"FIRRemoteConfigstatus = %d",status);
            BOOL activateFetched = [[FIRRemoteConfig remoteConfig]activateFetched];
            if (activateFetched) {
            //获取服务端的值
                FIRRemoteConfigValue *value = [[FIRRemoteConfig remoteConfig] configValueForKey:@"app_color"];
                DLog(@"FIRRemoteConfigvalue = %@, %@",value.dataValue,value.stringValue);
            }
        }
    }];
}
```
#### 注意：
- fetchWithExpirationDuration:completionHandler: 使用 (默认情况下，缓存在 12 小时后失效) 
<br>需限制是在 60 分钟的时间段内最多可以提取 5次。否则如果您的应用多次使用 fetchWithExpirationDuration:completionHandler: 请求刷新值，请求会遭到阻止，并向您的应用提供缓存的值。 [参考](https://firebase.google.com/docs/remote-config/ios?authuser=0#caching_and_throttling)
- 使用远程配置模板时，请注意以下要求：这些模板有不同的版本，每个版本的有效期均为 90 天（从创建之日起到将其替换为更新版本为止），而存储的版本总数不超过 300 个


## [A/B Test](https://firebase.google.com/docs/ab-testing/)

### 简介
> Firebase A/B 测试可让您轻松地运行、分析和扩展产品和营销实验，从而帮助您改进应用。它使您能够测试应用界面、功能或互动广告系列的更改，以确认这些更改是否确实使关键指标（如收入和用户留存率）较更改前有所改观。

### 支持两种测试方式
- [创建远程配置实验](https://firebase.google.com/docs/ab-testing/abtest-config?authuser=0)
- [创建消息传递实验](https://firebase.google.com/docs/ab-testing/abtest-with-console?authuser=0)


### 主要功能

- 运行测试并提升您的产品使用体验
  - 通过远程配置创建实验，在实验的变体中更改应用的行为和外观，并测试哪种产品使用体验能最有效地带来您最关注的效果。
- 使用通知编辑器寻找再次吸引用户的方法
  - 使用 A/B 测试帮助您找出最有效的措辞和消息设置来吸引用户使用您的应用。
- 安全推出新功能
  - 要推出新功能，首先必须在一小部分用户身上进行测试，确保其符合您的目标。等到对 A/B 测试结果有了信心后，再面向全部用户推出功能。
- 定位“预测”的用户群
  - 借助 Firebase 预测功能，您可以针对预计会执行特定操作的用户运行 A/B 测试，这类操作包括花钱消费（或不花钱）、停止使用您的应用，以及执行您通过 Analytics 定义的任何其他转化事件等。
 
### 过程
```
//通过Cocopod导入
pod 'FirebaseABTesting'
```
> 默认会在[FIRApp configure]; 初始化 <br>
> 通过远程配置方式获取参数数据代码参考RemoteConfig部分

#### 创建A/B TEST实验
> 创建remote_a/b_test实例

![](https://zeqinjie.github.io/images/2018/Firebase的使用集合（二）/3.png)

> 创建测试条件及参数
- 控制组，Variant A  两组数据

![](https://zeqinjie.github.io/images/2018/Firebase的使用集合（二）/4.png)

> 测试配置执行的ID凭证
- ID凭证即是fcmToken

![](https://zeqinjie.github.io/images/2018/Firebase的使用集合（二）/5.png)

> 通过远程配置的方式获取到参数条件
- 参考Remote Configure 远程配置方式
