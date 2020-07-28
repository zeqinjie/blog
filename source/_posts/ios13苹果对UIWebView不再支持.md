---
title: ios13苹果对UIWebView不再支持
date: 2020-05-13 19:03:04
categories: "iOS"
tags:
  - iOS
---

## UIWebView 将被禁止提交审核
### 在 iOS 13 推出后，如果开发者将包含 UIWebViewapi 的应用更新上传到 App Store 审核后，其将会收到包含 ITMS-90809信息的回复邮件，提示你在下一次提交时将应用中 UIWebView的 api 移除。
> Dear Developer,
We identified one or more issues with a recent delivery for your app, "xxxxxxx. Your delivery was successful, but you may wish to correct the following issues in your next delivery:
ITMS-90809: Deprecated API Usage - Apple will no longer accept submissions of new apps that use UIWebView as of April 30, 2020 and app updates that use UIWebView as of December 2020. Instead, use WKWebView for improved security and reliability. Learn more (https://developer.apple.com/documentation/uikit/uiwebview).
After you’ve corrected the issues, you can upload a new binary to App Store Connect.
Best regards,
The App Store Team

### 项目用了一些第三库，比如WebViewJavascriptBridge 和 AFNetworking
- 目前AFNetworking 已出4.0版本已移除UIWebView相关文件
- 假设我们不想手动删除文件或者更新最新库，可以通过Podfile配置移除涉及UIWebView相关的文件
> 参考如下
```
post_install do |installer|
    dir_web = File.join(installer.sandbox.pod_dir('WebViewJavascriptBridge'), 'WebViewJavascriptBridge')
    Dir.foreach(dir_web) {|x|
      real_path = File.join(dir_web, x)
      if (!File.directory?(real_path) && File.exists?(real_path))
        if(x == 'WebViewJavascriptBridge.h' || x == 'WebViewJavascriptBridge.m')
          File.delete(real_path)
        end
      end
    }
    
    dir_web = File.join(installer.sandbox.pod_dir('AFNetworking'), 'UIKit+AFNetworking')
    Dir.foreach(dir_web) {|x|
      real_path = File.join(dir_web, x)
      if (!File.directory?(real_path) && File.exists?(real_path))
        if(x == 'UIWebView+AFNetworking.h' || x == 'UIWebView+AFNetworking.m' || x == 'UIKit+AFNetworking.h')
          File.delete(real_path)
        end
      end
    }
    
end

```
