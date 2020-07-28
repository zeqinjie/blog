---
title: OC和Swift混编经验
date: 2018-09-10 19:03:02
categories: "iOS"
tags:
  - iOS
---


## Swift简介 
> Swift，苹果于2014年WWDC（苹果开发者大会）发布的新开发语言，可与Objective-C*共同运行于Mac OS和iOS平台，用于搭建基于苹果平台的应用程序。
2015年12月4日，苹果公司宣布其Swift编程语言现在开放[源代码](https://github.com/apple/swift)。
[掘金地址](https://juejin.im/post/5b9650c35188255c8a05dd94)

## Swift 优势
### 简洁代码
> 1. Swift所有的变量定义使用var ,常量定义使用let。
> 2. 数组或字符串可以使用"+"符号直接相加。
> 3. 可以使用高阶函数map，flatMap，reduce，filter简化代码量。

### 易维护
> 1. Swift抛弃了OC头文件（.h）和实现文件（.m）组合成一个代码文件（.swift）。
> 2. 因为Swift不是基于C构建的，而OC是在C和Smalltalk下构建。所以在设计上比较新颖，同时我们能看到现代语言（JavaScript，Java，Python，C#，以及 C++ ）的身影特性：泛型，可选类型，类型推断，高阶函数。基于泛型和高阶函数的使用导致更清晰，更可重用的代码，更易于维护。

### 安全
> 1. 在OC中不同类型可以直接相加，默认会隐式转换。但是swift不同类型的是不能直接相加，必须先进行类型转换。
> 2. 在其他的编程语言当中，你并不能知道哪个变量可以为空(null)哪个不能，这就强制开发者去考虑可能为空的情况。
> 3. 在OC中if判断存在非0即是true的方式，但是在swift中if判断必须为true或者false。
### 快
> 1. OC是一门动态的语言，很多实际执行需要在运行时才可以确定，Swift不一样，Swift将很多在运行时才可以确定的信息，在编译期就决定了。这就让Swift更加快速。[Swift性能探索和优化分析-王巍](https://onevcat.com/2016/02/swift-performance/)
> 1. 在纯Swift中的函数调用就不是OC的那套runtime了，而是类似C++的vtable，在编译期就决定调用哪个函数。那如果想使用OC的特性需要继承自NSObject。

## Swift 与 OC 的混编
> 1. 当我们在OC项目中添加Swift文件的时候系统会默认帮我创建一个桥接文件。
> “项目名称-Bridging-Header.h”的文件。
> 2. 在Swift中调用OC的类时，只需要在上面桥接头文件import即可
> 3. 在OC中调用Swift类，则需在OC文件中import "项目名称-Swift.h" 头文件即可

## 混合开发的问题
> 最近有个新闻列表改版需求，那在此下使用Swift去重做这块需求，需求中遇到一些坑和大家分享下。
- 数模转换的坑
>  在OC项目中常用的MJExtension第三方库。那在swift中我们怎么做呢？
> 当然可以使用目前第三方库SwiftJson或者HandyJSON或者实现Codable协议。不过这里我使用的是系统KVC方式,遇到问题如下
>   1. 因为swift的构造函数与OC不一样，当子类中有自己的构造函数时，那么将不继承父类自定义的构造函数。所以子类需重写父类init(dict : [String : Any])这个构造函数否则将无法使用。
>   2. 因为使用的是NSObject的KVC特性所以需加@objc定义子类属性。注意在swift4.0之前默认只要继承NSObject就会系统默认添加，但这之后需要手动添加了。

```
class BaseModel: NSObject {
    override init() {
    }
    
    init(dict : [String : Any]) {
        super.init()
        setValuesForKeys(dict)
    }
    //记得重写该方法避免崩溃
    override func setValue(_ value: Any?, forUndefinedKey key: String) {}
}

class NewsNavModel: BaseModel {
    //type = 998 最新,999 收藏
    @objc var type:Int = 0
    @objc var type_name:String = ""
    
    init(_ type : Int , type_name : String) {
        super.init()
        self.type = type
        self.type_name = type_name
    }
    //需重写父类方法
    override init!(dic: [String : Any]!) {
        super.init(dic: dic)
    }
}

```
- 关于@objc坑
> 1. 在swift3使用#selector指定的方法，只有当方法权限为private时需要加@objc修饰符，现在Swift4.0全都要加@objc修饰符
> 2. 定义的方法，枚举，协议或者是属性如果需要被OC调用则需添加@objc
> 3. 自定义的protocol协议中,有optional修饰的非必须实现的方法,需要用@objc修饰

- 被废弃的方法:
> initialize/dispatch_once方法已经被Swift4.0废弃所以后续不能再使用

- NSClassFromString方法的坑
> 在OC的时候直接使用类名即可转换为对于Class，但是Swift中有命名空间存在所以使用这个方法需要添加项目的名称

```
//OC 中
Class ocClass = NSClassFromString(@"MyFavModel");

//Swift中
let swiftClass = NSClassFromString("项目名称.MyFavModel") 
```

- fatalError
> 在Swift 中继承了遵守NSCoding protocol的类时，并自定义构造函数时候则需加入required init(coder aDecoder: NSCoder)。这个是在OC中则不存在的。
其作用是表明子类不能通过被fatalError定义的函数做初始化
```
override init(nibName nibNameOrNil: String?, bundle nibBundleOrNil: Bundle?) {
    super.init(nibName: nibNameOrNil, bundle: nibBundleOrNil)
    self.hidesBottomBarWhenPushed = true
}

required init(coder aDecoder: NSCoder) {
    fatalError("init(coder:) has not been implemented")
}
```
- OC的类不能继承至Swift的类，但是Swift类可以继承至OC的类。
> 补充：即使Swift的父类是OC也不能让OC类继承该Swift类
- Swift中没有宏，只能使用全局常量或者全局函数替代。
```
/// 当前app信息
let AppInfo = Bundle.main.infoDictionary

/// 当前app版本号
let AppCurrentVersion = Bundle.main.object(forInfoDictionaryKey: "CFBundleShortVersionString") as! String

/// 日志打印
func DLog(str: String) {
    #if DEBUG
    print("file: \(#file), line:\(#line),\(str)")
    #endif
}
```

- Swift不像OC那样能直接调用C++,需通过OC或者C去调用C++。




## 其他
### Swift版本升级 
> 例如:Swift3.0 -> Swift4.0
> 1. 选中要转换的 target
> 2. Edit -> Convert -> To Current Swift Syntax 
> 3. 勾选需要转换的 target （pod 引用不用勾选），Next 
> 4. 选择转换选项，Next 

![](https://zeqinjie.github.io/images/2018/OC和Swift混编经验/1.png)



### 快速的将OC语言转换成Swift语言
> 这里推荐一款swiftify插件。支持在线转换，或者添加插件到Xcode中。
> 1. [插件安装AppStore下载地址](https://itunes.apple.com/cn/app/swiftify-objective-c-converter-for-xcode/id1183412116?mt=12)
> 2. [在线转换地址](https://objectivec2swift.com/#/home)

![](https://zeqinjie.github.io/images/2018/OC和Swift混编经验/2.png)


### 快速将json转化为Swift的模型对象
> 基于Codable协议快速的数模序列化
> 1. 它不仅支持Swift,还支持其他语言比如Kotlin,Java,C#,Ruby,Object-c,Python
> 1. [在线链接](https://app.quicktype.io/)

![](https://zeqinjie.github.io/images/2018/OC和Swift混编经验/3.png)

### 通过R.swift快速生成资源代码
根据项目内容来自动化生 ImageName 和 SegueName代码 可以使用Switf的自动化工具了，[R.swift](https://github.com/mac-cain13/R.swift) 和 [SwiftGen](https://github.com/SwiftGen/SwiftGen)
```
let image = UIImage(imageName: .imgIcon)
```

### Swift 测试速度
- [Swift 与 Object-C的速度测试对比](https://www.jessesquires.com/blog/apples-to-apples-part-two/)
- [Swift 与 C的速度测试对比](https://www.jessesquires.com/blog/apples-to-apples-part-three/)

## 推荐
> Swift学习文档：
> 1. [The Swift Programming Language (Swift 4)](https://link.jianshu.com/?t=https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/TheBasics.html#//apple_ref/doc/uid/TP40014097-CH5-ID309)
> 2. [Swift的180个介绍](https://github.com/apple/swift-evolution/tree/master/proposals)

> Swift 书籍 
> 1. [Swifter - Swift 必备 tips](http://swifter.tips/buy)
> 2. [Swift 进阶](https://www.objccn.io/products/advanced-swift/)

> 参考地址
> 1. [Swift4.0新特性](https://www.cnblogs.com/baitongtong/p/7250940.html)
