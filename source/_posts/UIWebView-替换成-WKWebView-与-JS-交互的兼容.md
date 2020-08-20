---
title: UIWebView 替换成 WKWebView 与 JS 交互的兼容
date: 2018-06-20 09:52:15
tags:
  - iOS
---

## JS代码
```
let postMessageToApp = (json) => {
    if (!json) {
        throw new Error('參數不正確')
    }
    try {
        AppModel.jumpToApp(json) // UIWebview 的调用方式，老的
        //分享 AppModel.share(json)
    } catch (error) {
        console.log(error)
    }

    try {
        window.webkit.messageHandlers.AppModel.postMessage(json) // WKWebview 的调用方式，新的
        //分享 window.webkit.messageHandlers.AppShareModel.postMessage(json)
    } catch (error) {
        console.log(error)
    }
}

```

## 交互代码
```

#import <JavaScriptCore/JavaScriptCore.h>
#import <WebKit/WebKit.h>
typedef void(^JumpToAppBlock)(NSDictionary *json);

@protocol JavaScriptObjectiveCDelegate <JSExport>
//分享
- (void)share:(NSString *)json;
//跳转
- (void)jumpToApp:(NSString *)json;

@end


@interface TWWebAppJumpBaseModel:NSObject
@property (nonatomic, weak) UIViewController *vc;
@property (nonatomic, copy) JumpToAppBlock jumpToAppBlock;

- (void)jumpToApp:(NSString *)json;
- (void)share:(NSString *)json;

@end

@interface TWWebAppJumpModel : TWWebAppJumpBaseModel  <JavaScriptObjectiveCDelegate>
@property (nonatomic, weak) JSContext *jsContext;
@property (nonatomic, weak) UIWebView *webView;
@end

@interface TWWKWebJSBridgeModel:TWWebAppJumpBaseModel
@property (nonatomic, weak) WKWebView *webView;
- (void)setJSUserScriptNames:(NSArray *)userScriptNames;
- (void)removeAllUserScripts;
@end

```

```

#import "TWWebAppJumpModel.h"

@implementation TWWebAppJumpBaseModel

/**
 跳轉
 */
- (void)jumpToApp:(NSString *)json{
    //处理跳转
    dispatch_async(dispatch_get_main_queue(), ^{
        
    });
    
}

/**
 分享
 */
- (void)share:(NSString *)json{
    //处理分享
    dispatch_async(dispatch_get_main_queue(), ^{
        
    });
}

@end

@implementation TWWebAppJumpModel


@end


@interface TWWKWebJSBridgeModel ()<WKScriptMessageHandler>

@property (nonatomic, strong) NSArray *userScriptNames;

@end

@implementation TWWKWebJSBridgeModel

/// 注入JS MessageHandler和Name
- (void)setJSUserScriptNames:(NSArray *)userScriptNames{
    if(_userScriptNames.count)[self removeAllUserScripts];
    _userScriptNames = userScriptNames;
    [userScriptNames enumerateObjectsUsingBlock:^(NSString * _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        [self.userContentController addScriptMessageHandler:self name:obj];
    }];
}


/// 移除JS MessageHandler
- (void)removeAllUserScripts{
    [self.userScriptNames enumerateObjectsUsingBlock:^(NSString * _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        [self.userContentController removeScriptMessageHandlerForName:obj];
    }];
    self.userScriptNames = nil;
}

/// 接收JS调iOS的事件消息
- (void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message{
    DLog(@"WK JS调iOS  name : %@    body : %@",message.name,message.body);
    NSString *body = message.body;
    if ([message.name isEqualToString:@"AppModel"]) {
        if (IsStrClass(body)) {
            [self jumpToApp:body];
        }
    }else if ([message.name isEqualToString:@"AppShareModel"]) {//分享
        if (IsStrClass(body)) {
            [self share:body];
        }
    }
}

@end


```

#### UIWebView

```
- (TWWebAppJumpModel *)appModel{
    if (!_appModel) {
        _appModel = [[TWWebAppJumpModel alloc]init];
        WS(weakSelf);
        _appModel.jumpToAppBlock = ^(NSDictionary *json) {
            
        };
    }
    return _appModel;
}

//设置jsContext
- (void)webViewContext{
    self.jsContext = [self.webView valueForKeyPath:@"documentView.webView.mainFrame.javaScriptContext"];
    // 通过模型调用方法。
    self.jsContext[@"AppModel"] = self.appModel;
    self.appModel.jsContext = self.jsContext;
    self.appModel.webView = self.webView;
    self.appModel.vc = self;
    self.jsContext.exceptionHandler = ^(JSContext *context, JSValue *exceptionValue) {
        context.exception = exceptionValue;
        DLog(@"异常信息：%@", exceptionValue);
    };
}

#pragma mark - UIWebViewDelegate
- (void)webViewDidFinishLoad:(UIWebView *)webView{
    [self webViewContext];
}

```

#### WKWebView

```

- (TWWKWebJSBridgeModel *)jsBridgeModel{
    if (!_jsBridgeModel) {
        _jsBridgeModel = [[TWWKWebJSBridgeModel alloc]init];
        WS(weakSelf);
        _jsBridgeModel.jumpToAppBlock = ^(NSDictionary *json) {
            
        };
    }
    return _jsBridgeModel;
}

- (WKWebView *)createWKWebView{
    //设置网页的配置文件
    WKWebViewConfiguration * configuration = [[WKWebViewConfiguration alloc]init];
    //允许视频播放
    if (IOS_Versions >= 9.0) {
        configuration.allowsAirPlayForMediaPlayback = YES;
        if ([self isUseWKCacheStore]) {
            configuration.websiteDataStore = [WKWebsiteDataStore defaultDataStore];
        }
    }
    configuration.preferences.javaScriptEnabled = YES;
    
    configuration.preferences.javaScriptCanOpenWindowsAutomatically = NO;
    // 允许在线播放
    configuration.allowsInlineMediaPlayback = YES;
    // 允许可以与网页交互，选择视图
    configuration.selectionGranularity = YES;
    // web内容处理池
    configuration.processPool = [[WKProcessPool alloc] init];
    
    configuration.suppressesIncrementalRendering = NO;
    
    // 通过JS与webview内容交互
    configuration.userContentController = [[WKUserContentController alloc] init];
    self.jsBridgeModel.userContentController = configuration.userContentController;
    self.jsBridgeModel.vc = self;
    [self.jsBridgeModel setJSUserScriptNames:@[@"AppModel",@"AppShareModel"]];

    
    // 允许用户更改网页的设置
    WKWebView *prewkwebview = [[WKWebView alloc] initWithFrame:CGRectZero configuration:configuration];
    if (IOS_Versions >= 9.0) {
        NSString *agent = [USER_DEFAULTS objectForKey:@"UserAgent"];
        if(![HouseValidateTool isEmpty:agent]){
            prewkwebview.customUserAgent = agent;
        }
    }
    prewkwebview.navigationDelegate = self;
    prewkwebview.scrollView.delegate = self;
    prewkwebview.UIDelegate = self;
    self.jsBridgeModel.webView = prewkwebview;
    return prewkwebview;
}

```

```
- (void)dealloc{
    [self.jsBridgeModel removeAllUserScripts];
}
```

```

//在JS端调用alert函数时，会触发此代理方法。
- (void)webView:(WKWebView *)webView runJavaScriptAlertPanelWithMessage:(NSString *)message initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(void))completionHandler;
 
//JS端调用confirm函数时，会触发此方法
- (void)webView:(WKWebView *)webView runJavaScriptConfirmPanelWithMessage:(NSString *)message initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(BOOL result))completionHandler;
 
//JS端调用prompt函数时，会触发此方法
- (void)webView:(WKWebView *)webView runJavaScriptTextInputPanelWithPrompt:(NSString *)prompt defaultText:(nullable NSString *)defaultText initiatedByFrame:(WKFrameInfo *)frame completionHandler:(void (^)(NSString * _Nullable result))completionHandler;



```

#### UIWebView 及 WKWebView执行js函数

```

//执行js 函数
- (void)executejsContext:(NSString *)context{
    WS(weakSelf);
    if ([self isShowWKWebView]) {
        dispatch_async(dispatch_get_main_queue(), ^{
            [self.wkwebview evaluateJavaScript:context completionHandler:^(id _Nullable response, NSError * _Nullable error) {
                DLog(@"response: %@ error: %@", response, error);
            }];
        }）;
    }else{
         dispatch_async(dispatch_get_main_queue(), ^{
            if (weakSelf.jsContext && context) {
                [weakSelf.jsContext evaluateScript:context];
            }
        });

    }
}

```

- WKWebview的js与客户端交互回调函数didReceiveScriptMessage 是在主线程中
- UIWebview的js与客户端的回调函数是实现JSExport协议的方法是在子线程中所以不是线程安全