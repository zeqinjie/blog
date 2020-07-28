---
title: MVVM架构优势及应用场景
date: 2019-08-23 19:03:02
categories: "iOS"
tags:
  - iOS
---
# RxSwift应用

## 响应式编程 && 函数式编程
### 什么是响应式编程？
> 响应式编程，响应式编程是一种面向数据流和变化传播的编程方式式，简单理解就是异步的数据流的开发。


### 什么是函数式编程？
> 特点是将函数作为一等公民，当作参数和返回值使用。典型的如OC和Swift 中的 map函数、filter函数、reduce函数等。每个函数的处理结果给到下一个函数，最后的结果由自身函数调出。

---
- [从入门到放弃](https://www.v2ex.com/amp/t/367308)
![](https://zeqinjie.github.io/images/2019/MVVM架构优势及应用场景/1.png)
## 为什么要使用 RxSwift ？

### 我们先看一下 RxSwift 能够帮助我们做些什么：

####  Target Action

传统实现方法：

```swift
button.addTarget(self, action: #selector(buttonTapped), for: .touchUpInside)

func buttonTapped() {
    print("button Tapped")
}
```

通过 Rx 来实现：

```swift
button.rx.tap
    .subscribe(onNext: {
        print("button Tapped")
    })
    .disposed(by: disposeBag)
```

你不需要使用 Target Action，这样使得代码逻辑清晰可见。

---


### 代理

传统实现方法：

```swift
class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        scrollView.delegate = self
    }
}

extension ViewController: UIScrollViewDelegate {
    func scrollViewDidScroll(_ scrollView: UIScrollView) {
        print("contentOffset: \(scrollView.contentOffset)")
    }
}
```

通过 Rx 来实现：

```swift
class ViewController: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()

        scrollView.rx.contentOffset
            .subscribe(onNext: { contentOffset in
                print("contentOffset: \(contentOffset)")
            })
            .disposed(by: disposeBag)
    }
}
```

你不需要书写代理的配置代码，就能获得想要的结果。

---

### 闭包回调

传统实现方法：

```swift
URLSession.shared.dataTask(with: URLRequest(url: url)) {
    (data, response, error) in
    guard error == nil else {
        print("Data Task Error: \(error!)")
        return
    }

    guard let data = data else {
        print("Data Task Error: unknown")
        return
    }

    print("Data Task Success with count: \(data.count)")
}.resume()
```

通过 Rx 来实现：

```swift
URLSession.shared.rx.data(request: URLRequest(url: url))
    .subscribe(onNext: { data in
        print("Data Task Success with count: \(data.count)")
    }, onError: { error in
        print("Data Task Error: \(error)")
    })
    .disposed(by: disposeBag)
```

回调也变得十分简单

---

### 通知

传统实现方法：

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    NotificationCenter.default.addObserver(self, selector: #selector(updateNotificationStatus), 
                                                     name: UIApplication.willEnterForegroundNotification,
                                                   object: nil)
}

deinit {
    NotificationCenter.default.removeObserver(self)
}

// MARK: - Notification
@objc func updateNotificationStatus(){
    print("Application Will Enter Foreground")
}
```

通过 Rx 来实现：

```swift
override func viewDidLoad() {
    super.viewDidLoad()

    NotificationCenter.default.rx.notification(UIApplication.willEnterForegroundNotification)
    .subscribe(onNext: { (notification) in
        print("Application Will Enter Foreground")
    }).disposed(by: disposeBag)
}
```

你不需要去管理观察者的生命周期，这样你就有更多精力去关注业务逻辑。

---

####  KVO

传统实现方法：

```swift
private var observerContext = 0

override func viewDidLoad() {
    super.viewDidLoad()
    user.addObserver(self, forKeyPath: #keyPath(User.name), options: [.new, .initial], context: &observerContext)
}

override func observeValue(forKeyPath keyPath: String?, of object: Any?, change: [NSKeyValueChangeKey : Any]?, context: UnsafeMutableRawPointer?) {
    if context == &observerContext {
        let newValue = change?[.newKey] as? String
        print("do something with newValue")
    } else {
        super.observeValue(forKeyPath: keyPath, of: object, change: change, context: context)
    }
}

deinit {
    user.removeObserver(self, forKeyPath: #keyPath(User.name))
}
```

通过 Rx 来实现：

```swift
override func viewDidLoad() {
    super.viewDidLoad()

    user.rx.observe(String.self, #keyPath(User.name))
        .subscribe(onNext: { newValue in
            print("do something with newValue")
        })
        .disposed(by: disposeBag)
}
```

这样实现 KVO 的代码更清晰，更简洁并且更准确。

---

####  多个任务之间有依赖关系

例如，先通过用户名密码取得 Token 然后通过 Token 取得用户信息，

传统实现方法：

```swift
/// 用回调的方式封装接口
enum API {

    /// 通过用户名密码取得一个 token
    static func token(username: String, password: String,
        success: (String) -> Void,
        failure: (Error) -> Void) { ... }

    /// 通过 token 取得用户信息
    static func userinfo(token: String,
        success: (UserInfo) -> Void,
        failure: (Error) -> Void) { ... }
}
```

```swift
/// 通过用户名和密码获取用户信息
API.token(username: "beeth0ven", password: "987654321",
    success: { token in
        API.userInfo(token: token,
            success: { userInfo in
                print("获取用户信息成功: \(userInfo)")
            },
            failure: { error in
                print("获取用户信息失败: \(error)")
        })
    },
    failure: { error in
        print("获取用户信息失败: \(error)")
})
```

通过 Rx 来实现：

```swift
/// 用 Rx 封装接口
enum API {

    /// 通过用户名密码取得一个 token
    static func token(username: String, password: String) -> Observable<String> { ... }

    /// 通过 token 取得用户信息
    static func userInfo(token: String) -> Observable<UserInfo> { ... }
}
```

```swift
/// 通过用户名和密码获取用户信息
API.token(username: "beeth0ven", password: "987654321")
    .flatMapLatest(API.userInfo)
    .subscribe(onNext: { userInfo in
        print("获取用户信息成功: \(userInfo)")
    }, onError: { error in
        print("获取用户信息失败: \(error)")
    })
    .disposed(by: disposeBag)
```

这样你无需嵌套太多层，从而使得代码易读，易维护。

---

####  等待多个并发任务完成后处理结果

例如，需要将两个网络请求合并成一个，

通过 Rx 来实现：

```swift
/// 用 Rx 封装接口
enum API {

    /// 取得老师的详细信息
    static func teacher(teacherId: Int) -> Observable<Teacher> { ... }

    /// 取得老师的评论
    static func teacherComments(teacherId: Int) -> Observable<[Comment]> { ... }
}
```

```swift
/// 同时取得老师信息和老师评论
Observable.zip(
      API.teacher(teacherId: teacherId),
      API.teacherComments(teacherId: teacherId)
    ).subscribe(onNext: { (teacher, comments) in
        print("获取老师信息成功: \(teacher)")
        print("获取老师评论成功: \(comments.count) 条")
    }, onError: { error in
        print("获取老师信息或评论失败: \(error)")
    })
    .disposed(by: disposeBag)
```

这样你可用寥寥几行代码来完成相当复杂的异步操作。

---

### RxSwift 的单向数据流
RxSwift 可以在 UniDirectional Data Flow 的各个阶段都发挥作用，从而让 Data 的处理和流动更加简洁和清晰。
![](https://zeqinjie.github.io/images/2019/MVVM架构优势及应用场景/2.png)

- 通过对 RxCocoa 的各种回调进行统一处理，方便了"交互"「Interact」的处理。
- 通过对 Observable 的 transform 和 composite，方便了 Action 的生成（比如使用 throttle 来压缩 Action）。
- 通过对网络请求以及其他异步数据的获取进行 Observable 封装，方便了异步数据的处理。
- 通过 RxCocoa 的 binding，方便了数据的渲染。

### RxSwift优势
* 组合 - Rx对不同的信号进行组合
* 复用 - 因为它是可组合的
* 清晰 - 因为声明都是不可变的，改变的只有数据
* 易用 - 因为它抽象的了异步编程，使我们统一了代码风格
* 内存回收 - 简单内存管理
* 稳定 - 因为 Rx 是完全通过单元测试的

## RxSwift 与 MVVM的邂逅
> MVVM 是 Model-View-ViewModel 的缩写。
- MVVM 增加了 ViewModel 层。我们可以将原来 Controller 中的业务逻辑抽取出来放到 ViewModel 中，从而大大减轻了 ViewController 的负担。
- 同时在 MVVM 中，ViewController 只担任 View 的角色（ViewController 与 View 现在共同作为 View 层），负责 View 的显示和更新，其他业务逻辑不再需要 ViewController 来管了。

> 同样使用 MVVM 架构时，Model 与 View|ViewControllter 之间是不允许直接通信的，而是由 ViewModel 层进行协调

![](https://zeqinjie.github.io/images/2019/MVVM架构优势及应用场景/3.png)
#### 基于RxSwift对网络工具封装 TWSwiftHttpTool
```
// 声明一个枚举，包含成功和失败的情况
enum TWSwiftHttpResult {
    case success(Any)  //成功
    case failure(String) //失败
    case noNet() //无网络
}
    
    
// 使用RxSwift进行扩展
extension Reactive where Base: TWSwiftHttpTool {
    
    /// 基于YYCache 缓存的RXSwift 请求方式
    static func request(type: TWRequestType,
                        url: String,
                        parameters: [AnyHashable: Any]?,
                        flag: Bool = true,
                        isCache: Bool = false,
                        cacheKey: String? = nil,
                        cacheBlock: (CacheBlock)? = nil) -> Observable<Any> {
        return Observable.create { observer in
            let task = TWSwiftHttpTool.request(type: type, url: url, flag: flag, parameters: parameters, isCache: isCache, cacheKey: cacheKey, cacheBlock: cacheBlock, complete: { (result) in
                dealComplete(result: result, observer: observer)
            })
            return Disposables.create(with: task.cancel)
        }
    }
    
    
    
    // MARK: - Private Common Method
    private static func dealComplete(result:TWSwiftHttpResult,observer:AnyObserver<Any>) {
        switch result {
        case .success(let response):
            observer.onNext(response)
            observer.onCompleted()
        case .failure(let reason):
//            observer.onNext([TWSwiftErrorMsg:reason])
            observer.onError(RxNetworkError.general(reason))
        case .noNet:
//            observer.onNext([TWSwiftErrorMsg:TWSwiftNoNetMsg])
            observer.onError(RxNetworkError.noNet)
        }
    }
}

```
#### viewModel 定义
```

//订阅输入输出协议
protocol TWSwiftViewModelProtocol {
    associatedtype TWSwiftInput
    associatedtype TWSwiftOutput
    
    func transform(input: TWSwiftInput) -> TWSwiftOutput
}

//viewModel实现TWSwiftViewModelProtocol的协议
extension NewHouseWeekViewModel:TWSwiftViewModelProtocol{
    
    typealias TWSwiftInput = Input
    
    typealias TWSwiftOutput = Output
    
    // MARK: - Override Method
    struct Input {
        //即是订阅又是被订阅
        let requestId = PublishSubject<String>()
    }
    
    struct Output {
        //输出数据源
        let sections: Driver<[NewHouseWeekSectionModel]>
        //成功输出
        let successSubject = PublishSubject<String>()
        //错误输出
        let errorSubject = PublishSubject<String>()

        init(sections: Driver<[NewHouseWeekSectionModel]>) {
            self.sections = sections
        }
    }
    
    func transform(input: NewHouseWeekViewModel.Input) -> NewHouseWeekViewModel.Output {
        ```
    }
}
```

#### viewController通过viewModel实现model与view绑定
```
/// viewModel绑定
    fileprivate func bindViewModel() {
        let vmInput = NewHouseWeekViewModel.Input() ///输入
        let vmOutput = viewModel.transform(input: vmInput) ///输出
        ///数据源绑定到tableview中
        vmOutput.sections.asDriver().drive(tableView.rx.items(dataSource: dataSource)).disposed(by: TWSwiftDisposeBag)
        ///错误订阅
        vmOutput.errorSubject.subscribe(onNext:{[weak self] (errorMsg) in
            self?.hideHud(in: self?.tableView, hint: errorMsg)
            self?.setEmptyDic()
        }).disposed(by: TWSwiftDisposeBag)
        ///成功订阅
        vmOutput.successSubject.subscribe(onNext: nil, onError: nil, onCompleted: {[weak self] in
            self?.hideHUD()
            self?.setEmptyDic()
        }, onDisposed: nil).disposed(by: TWSwiftDisposeBag)
        let regionId = TWSwiftGuardNullString(GlobalObject.share()?.regionId)
        ///发起数据请求
        vmInput.requestId.onNext(regionId)
        self.showActivityIndicatorSuperView(tableView)
    }
```


### RxSwift的看法
> 理解响应链的编程思路
> RxSwift给我们带来最大影响的Reactive思想，OOP告诉我们，在编写应用程序的时候，要考虑的是对象有什么，对象做什么，对象与对象之间的联系，而Reactive思想将对象所做的都看成是数据流，我们关注的是事件本身的影响。

## 参考链接
- [RxSwift 使用详解系列](https://www.jianshu.com/p/f61a5a988590)
- [RxSwift中文文档](https://beeth0ven.github.io/RxSwift-Chinese-Documentation/)
- [是时候学习 RxSwift 了](https://limboy.me/tech/2016/12/11/time-to-learn-rxswift.html)
- [RxSwift的整理](http://note.youdao.com/noteshare?id=ad1212ce231507cb934fd08dcc673d30)
- [RxSwift github](https://github.com/ReactiveX/RxSwift)
- [项目框架](http://bookmobile.debug.591.com.tw/)
