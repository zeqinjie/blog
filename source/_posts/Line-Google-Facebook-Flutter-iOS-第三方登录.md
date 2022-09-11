---
title: Line, Google, Facebook (Flutter & iOS) 第三方登录
date: 2022-09-11 15:09:09
categories: "Flutter"
tags:
  - Flutter
  - iOS
---

## Line

### 注册 line 账号

- 创建 Providers & channel
- 配置 bundle ID

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b0f5356b2b76452b9aee62c33253d526~tplv-k3u1fbpfcp-zoom-1.image)

### 注意设置 URL Type

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/399b9f1ff9a841fc95e14ea1deeff470~tplv-k3u1fbpfcp-watermark.image?)

- 配置如下 [传送门](https://github.com/line/line-sdk-ios-swift/issues/173)

![image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f50c4ab0dc80401dac86b039cb9bc583~tplv-k3u1fbpfcp-zoom-1.image)

### 代码

#### iOS

```
/// 注册渠道 ID
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    // Override point for customization after application launch.
    LoginManager.shared.setup(channelID: "xxxxxxx", universalLinkURL: nil)
    return true
}

/// 登录 line
func lineAction(_ sender: UIButton) {
    LoginManager.shared.login(permissions: [.profile], in: self) { result in
        switch result {
        case .success(let loginResult):
            if let profile = loginResult.userProfile {
               print("User ID: (profile.userID)")
               print("User Display Name: (profile.displayName)")
               print("User Icon: (String(describing: profile.pictureURL))")
           }
        case .failure(let error):
            print(error)
        }
    }
}

/// 登出 line
func lineLoginOutAction()  {
    LoginManager.shared.logout { (result) in
        switch result {
        case .success(let success):
            print(success)
        case .failure(let error):
            print(error)
        }
    }
}
```

#### Flutter

```
/// 注册渠道 ID
void main() {
  WidgetsFlutterBinding.ensureInitialized();
  LineSDK.instance.setup("xxxxxx").then((_) {});
  runApp(const MyApp());
}

/// 登录 line
void lineLoginHandler() async {
  try {
    final result = await LineSDK.instance.login();
    final userId = result.userProfile?.userId;
    final name = result.userProfile?.displayName;
    final avatar = result.userProfile?.pictureUrl;
    print('userId: $userId name: $name avatar: $avatar');
  } on PlatformException catch (e) {
    print(e.toString());
  }
}

/// 登出 line
void lineLoginOutHandler() async {
  try {
    await LineSDK.instance.logout();
  } on PlatformException catch (e) {
    print(e.message);
  }
}
```

## Google

### 创建 OAuth 客户端 ID

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/71bb3e7fe8c6412bb1ddd35ff9f74088~tplv-k3u1fbpfcp-zoom-1.image)

### 配置 OAuth 同意画面

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7a212d34e84545eb878617af17d124ee~tplv-k3u1fbpfcp-zoom-1.image)

### 注意

#### 设置 URL Type

- 注意设置 URL Schemes 是凭证 iOS 网址通讯协定

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2b88c1d76c154c3fbc20dd6c56143422~tplv-k3u1fbpfcp-watermark.image?)

#### 设置 OAuth

- 如果没有设置配置 OAuth 同意画面如下问题

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0cca69ce19d2451b8c484326ee81c202~tplv-k3u1fbpfcp-zoom-1.image)

- 需要对自己站台配置 DNS 设定，[传送门](https://search.google.com/search-console/about)

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0a901ed71a3a481b8d2dd637e6e0a6be~tplv-k3u1fbpfcp-zoom-1.image)

### 代码

#### iOS

```
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    GIDSignIn.sharedInstance.restorePreviousSignIn { user, error in
        if error != nil || user == nil {
          // Show the app's signed-out state.
        } else {
          // Show the app's signed-in state.
        }
    }
    return true
}

/// 登入 google
func googleAction(_ sender: UIButton) {
    // 注意客户端 id 是凭着里的用戶端编号
    let signInConfig = GIDConfiguration(clientID: "xxxxxxx-xxxxxxxxxxxxxx.apps.googleusercontent.com")
    GIDSignIn.sharedInstance.signIn(with: signInConfig, presenting: self) { user, error in
        guard error == nil else { return }
        // If sign in succeeded, display the app's main content View.
    }
}

/// 登出 google
func googleLoginOutAction()  {
    GIDSignIn.sharedInstance.signOut();
}
```

#### Flutter

```
final GoogleSignIn _googleSignIn = GoogleSignIn(
  clientId:
      'xxxxxxx-xxxxxxxxxx.apps.googleusercontent.com',
  scopes: [
    'email',
    'https://www.googleapis.com/auth/contacts.readonly',
  ],
);

/// google 登录
void googleLoginHandler() async {
  try {
    final result = await _googleSignIn.signIn();
    final userId = result?.id;
    final name = result?.displayName;
    final email = result?.email;
    final avatar = result?.photoUrl;
    print('userId: $userId name: $name email: $email avatar: $avatar');
  } catch (error) {
    print(error);
  }
}

/// google 登出
void googleLoginOutHandler() async {
  try {
    await _googleSignIn.signOut();
  } catch (error) {
    print(error);
  }
}
```

## Facebook

### 创建应用，并授权

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a32aa57c5a8046f788620afbf1705920~tplv-k3u1fbpfcp-zoom-1.image)

### 注意设置 URL Type 等信息
- 应用编号 (FacebookAppID)
- 客户端口令 (FacebookClientToken)
- 显示名称（FacebookDisplayName）

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/acde4530f6934a63a87ef8ae1998634b~tplv-k3u1fbpfcp-zoom-1.image)

### 代码

#### iOS

```
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {        
    ApplicationDelegate.shared.application(
                application,
                didFinishLaunchingWithOptions: launchOptions
            )
    return true
}

// AppDelegate.swift
func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {
    var handled: Bool
    handled = GIDSignIn.sharedInstance.handle(url)
    if handled {
        return true
    }
    ApplicationDelegate.shared.application(
                app,
                open: url,
                sourceApplication: options[UIApplication.OpenURLOptionsKey.sourceApplication] as? String,
                annotation: options[UIApplication.OpenURLOptionsKey.annotation]
            )
    return LoginManager.shared.application(app, open: url)
}


/// facebook 登录
@IBAction func facebookAction(_ sender: UIButton) {
    let loginManager = LoginManager()
    loginManager.logIn(permissions: ["public_profile"], from: self) { result, error in
        if let error = error {
            print("Encountered Erorr: (error)")
        } else if let result = result, result.isCancelled {
            print("Cancelled")
        } else {
            print("Logged In")
            if let userID = result?.token?.userID  {
                print("userID: ===> (userID)")
            }
            if let tokenString = result?.token?.tokenString  {
                print("tokenString: ===> (tokenString)")
            }
        }
    }
}

/// 登出 facebook
func facebookLoginOutAction()  {
    let loginManager = LoginManager()
    loginManager.logOut()
}
```

#### Flutter

```
/// facebook 登录
void facebookLoginHandler() async {
  final fb = FacebookLogin();

  final res = await fb.logIn(permissions: [
    FacebookPermission.publicProfile,
    FacebookPermission.email,
  ]);

  switch (res.status) {
    case FacebookLoginStatus.success:
      // Logged in

      // Send access token to server for validation and auth
      final FacebookAccessToken? accessToken = res.accessToken;
      print('Access token: ${accessToken?.token}');

      // Get profile data
      final profile = await fb.getUserProfile();
      print('Hello, ${profile?.name}! You ID: ${profile?.userId}');

      // Get user profile image url
      final imageUrl = await fb.getProfileImageUrl(width: 100);
      print('Your profile image: $imageUrl');

      // Get email (since we request email permission)
      final email = await fb.getUserEmail();
      // But user can decline permission
      if (email != null) print('And your email is $email');

      break;
    case FacebookLoginStatus.cancel:
      // User cancel log in
      break;
    case FacebookLoginStatus.error:
      // Log in failed
      print('Error while log in: ${res.error}');
      break;
  }
}

/// facebook 登出
void facebookLoginOutHandler() async {
   try {
    final fb = FacebookLogin();
    await fb.logOut();
  } catch (error) {
    print(error);
  }
}
```

## Demo

- [iOS](https://github.com/zeqinjie/thrid_login_demo/tree/main/thrid_login_native)
- [Flutter](https://github.com/zeqinjie/thrid_login_demo/tree/main/thrid_login_flutter)

## 参考

### 官方文档

- [Developers Line](https://developers.line.biz/en/)
- [LINE SDK for iOS Swift overview](https://developers.line.biz/en/docs/ios-sdk/swift/overview/)
    - [LINE SDK for iOS Objective-C overview](https://developers.line.biz/en/docs/ios-sdk/objective-c/overview/)
    - [LINE SDK for Android overview](https://developers.line.biz/en/docs/android-sdk/overview/)
    - [LINE SDK for Flutter](https://developers.line.biz/en/docs/flutter-sdk/)
- [Developers Facebook](https://developers.facebook.com/docs/facebook-login/ios/advanced#custom-login-button)
- [Developers Google](https://developers.google.com/identity/sign-in/ios/start-integrating)

### Flutter SDK

- [flutter_line_sdk](https://github.com/line/flutter_line_sdk)
- [flutter_login_facebook](https://github.com/Innim/flutter_login_facebook)
- [google_sign_in](https://github.com/flutter/plugins/tree/main/packages/google_sign_in/google_sign_in)
