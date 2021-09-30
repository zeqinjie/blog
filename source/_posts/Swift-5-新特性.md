---
title: Swift 5 新特性
date: 2021-03-30 10:41:34
categories: "iOS"
tags:
  - iOS
---

## 标准 Result 型
> [SE-0235 ](https://github.com/apple/swift-evolution/blob/master/proposals/0235-add-result.md)`Result` 在标准库中引入了一种类型，从而为我们提供了一种更简单，更清晰的方式来处理诸如异步API之类的复杂代码中的错误。

```swift
enum NetworkError: Error {
    case errorURL
}

func fetchUnreadCount(from urlString: String, completionHandler: @escaping (Result<Int, NetworkError>) -> Void)  {
    guard let url = URL(string: urlString) else {
        print("errorUrl...")
        completionHandler(.failure(.errorURL))
        return
    }

    // 此处省略复杂的网络请求
    print("Fetching \(url.absoluteString)...")
    completionHandler(.success(5))
}

fetchUnreadCount(from: "") { result in
    switch result {
    case .success(let count):
        print("\(count) 个未读信息。")
    case .failure(let error):
        print(error.localizedDescription)
    }
}
```
### 原始字符串
> [SE-0200](https://github.com/apple/swift-evolution/blob/master/proposals/0200-raw-string-escaping.md) `原始字符串(raw string)` 让我们能够编写出更自然的字符串，尤其是在使用 `反斜杠`  和 `引号`时。正如将在下面看到的那样，在某些情况下效果是很明显的，如正则表达式。
> - `\` 不再作为转义字符
> -  `\n` 字面意思是反斜杠跟着 `“n”` 而不是换行符，
> -  `\(variable)` 不再表示字符串插值，而是实实在在的字符串

```swift
/// 以下等效
let regularString = "\" Hello \" World"
let rawString = #"Hello" World"#

/// 以下等效
let regex1 = "\\\\[A-Z]+[A-Za-z]+\\.[a-z]+"
let regex2 = #"\\[A-Z]+[A-Za-z]+\.[a-z]+"#

let name = "Taylor"
let greeting = #"Hello, \#(name)!"#

/// 换行
let message = #"""
This is rendered as text: (example).
This uses string interpolation: #(example).
"""#
```
### 自定义字符串插值
> [SE-0228](https://github.com/apple/swift-evolution/blob/master/proposals/0228-fix-expressiblebystringinterpolation.md) 改进了Swift的字符串插值系统，使其更高效，更灵活，并且创建了以前无法实现的全新功能。

#### **通过 extension** String.StringInterpolation
```swift
struct User {
    var name: String
    var age: Int
}

extension String.StringInterpolation {
    mutating func appendInterpolation(_ value: User) {
        appendInterpolation("My name is \(value.name) and I'm \(value.age)")
    }
    
    mutating func appendInterpolation(repeat str: String, _ count: Int) {
        for _ in 0 ..< count {
            appendLiteral(str)
        }
    }
    
    mutating func appendInterpolation(
        _ values: [String], empty defaultValue: @autoclosure () -> String) {
        if values.count == 0 {
            appendLiteral(defaultValue())
        } else {
            appendLiteral(values.joined(separator: ", "))
        }
    }
}

let user = User(name: "Guybrush Threepwood", age: 33)
print("User details: \(user)")

print("Baby shark \(repeat: "doo ", 6)")

let names = ["Harry", "Ron", "Hermione"]
print("List of students: \(names, empty: "No one").")
```
#### 通过实现 `ExpressibleByStringLiteral，ExpressibleByStringInterpolation，CustomStringConvertible`
> 结合使用 `ExpressibleByStringLiteral`  和   `ExpressibleByStringInterpolation` 协议议，现在可以使用字符串插值创建整个类型，如果添加，`CustomStringConvertible`我们甚至可以根据需要将这些类型打印为字符串。

```swift
struct HTMLComponent:
    ExpressibleByStringLiteral,
    ExpressibleByStringInterpolation,
    CustomStringConvertible {
    struct StringInterpolation: StringInterpolationProtocol {
        // 以空字符串开始
        var output = ""

        // 分配足够的空间来容纳双倍文字的文本
        init(literalCapacity: Int, interpolationCount: Int) {
            output.reserveCapacity(literalCapacity * 2)
        }

        // 一段硬编码的文本，只需添加它就可以
        mutating func appendLiteral(_ literal: String) {
            print("追加 ‘\(literal)’")
            output.append(literal)
        }

        // Twitter 用户名，将其添加为链接
        mutating func appendInterpolation(twitter: String) {
            print("追加 ‘\(twitter)’")
            output.append("<a href=\"https://twitter/\(twitter)\">@\(twitter)</a>")
        }

        // 电子邮件地址，使用 mailto 添加
        mutating func appendInterpolation(email: String) {
            print("追加 ‘\(email)’")
            output.append("<a href=\"mailto:\(email)\">\(email)</a>")
        }
    }

    // 整个组件的完整文本
    let description: String

    // 从文字字符串创建实例
    init(stringLiteral value: String) {
        description = value
    }

    // 从插值字符串创建实例
    init(stringInterpolation: StringInterpolation) {
        description = stringInterpolation.output
    }
}

let text: HTMLComponent = "你应该在 Twitter 上关注我 \(twitter: "twostraws")，或者你可以发送电子邮件给我 \(email: "paul@hackingwithswift.com")。"
print(text)
```
### 动态可调用（dynamicCallable）类型
> [SE-0216](https://github.com/apple/swift-evolution/blob/master/proposals/0216-dynamic-callable.md) `@dynamicCallable`在Swift中添加了一个新属性，该属性可以将类型标记为可直接调用。它是语法糖，而不是任何形式的编译器魔术

```swift
func dynamicCall(withArguments args: [Int]) -> Double
func dynamicCall(withKeywordArguments args: KeyValuePairs <String，String>) -> String
```
```swift
@dynamicCallable
struct RandomNumberGenerator {
    func dynamicallyCall(withKeywordArguments args: KeyValuePairs<String, Int>) -> Double {
        let numberOfZeroes = Double(args.first?.value ?? 0)
        let maximum = pow(10, numberOfZeroes)
        return Double.random(in: 0...maximum)
    }
}

let random = RandomNumberGenerator()
let result = random(numberOfZeroes: 0)
```

- 您可以将其应用于结构、枚举、类和协议。
- 如果使用**withKeywordArguments: **并且不使用**withArguments:**您的类型仍然可以在没有参数标签的情况下调用 - 您只会获得键的空字符串。
- 如果 withKeywordArguments: 或与withArguments: 被标记为**throwing**,调用类型也将**throwing**。
- **不能@dynamicCallable添加到扩展**,只能添加类型的主要定义。
- 您仍然可以向类型添加其他方法和属性,并正常使用它们。
#### 补充：在 4.2 中新增的 @dynamicMemberLookup 注解 动态成员查找
> @dynamicMemberLookup 标记了对象后（对象、结构体、枚举、protocol），实现了subscript(dynamicMember member: String)方法后我们就可以访问到对象不存在的属性

```swift
@dynamicMemberLookup
struct DynamicMemberLookup {
    subscript (dynamicMember name: String) -> String {
        return "zhengzeqin"
    }
    subscript (dynamicMember age: String) -> Int {
        return 30
    }
    subscript (dynamicMember2 member: String) -> Int {
        return 172
    }
}
let dynamicMember = DynamicMemberLookup()
let dynamicMemberName: String = dynamicMember.name
let dynamicMemberAge: Int = dynamicMember.age
let dynamicHeight: Int = dynamicMember.height
print(dynamicMemberName, dynamicMemberAge, dynamicHeight)
```
### 枚举 case @unknown
> [SE-0192](https://github.com/apple/swift-evolution/blob/master/proposals/0192-non-exhaustive-enums.md) 增加了在固定的枚举和可能将被改变的枚举间的区分度
> @unknown  编译器警告⚠️ 

```swift
enum PasswordError: Error {
    case short
    case obvious
    case simple
    case old
}

func showOld(error: PasswordError) {
    switch error {
    case .short:
        print("Your password was too short.")
    case .obvious:
        print("Your password was too obvious.")
    @unknown default:
        print("Your password was too simple.")
    }
}
```
### try? 嵌套可选的展平
> [SE-0230](https://github.com/apple/swift-evolution/blob/master/proposals/0230-flatten-optional-try.md) 修改 `try?` 的工作方式，以便嵌套的可选项被展平成为一个常规的选择。这使得它的工作方式与可选链和条件类型转换（if let）的工作方式相同，这两种方法都在早期的 Swift 版本中展平了可选项

```swift
struct TryUser {
    var id: Int
    init?(id: Int) {
        if id < 1 {
            return nil
        }
        self.id = id
    }

    func getMessages() throws -> String {
        // 复杂的一段代码
        return "No messages"
    }
}

let tryUser = TryUser(id: 1)
let messages = try? tryUser?.getMessages()
```
### 整倍数自省(Checking for integer multiples)
> [SE-0225](https://github.com/apple/swift-evolution/blob/master/proposals/0225-binaryinteger-iseven-isodd-ismultiple.md) 为整数添加 `isMultiple(of:)` 方法来允许我们以比使用取余数运算 `%` 更清晰的方式检查一个数是否是另一个数的倍数。

```swift
let rowNumber = 4

if rowNumber.isMultiple(of: 2) {
    print("Even")
} else {
    print("Odd")
}
```
### compactMapValues() 转换和解包字典值
> [SE-0218](https://github.com/apple/swift-evolution/blob/master/proposals/0218-introduce-compact-map-values.md) 为字典添加了一个新的 `compactMapValues()` 方法，它能够将数组中的 `compactMap()` 功能转换我需要的值，解包结果，然后丢弃任何 nil，与字典中的 `mapValues()` 方法一起使用能保持键的完整并只转换值

```swift
let times = [
    "Hudson": "38",
    "Clarke": "42",
    "Robinson": "35",
    "Hartis": "DNF"
]

let finishers = times.compactMapValues { Int($0) }
print(finishers)
```


### 被移除的特性：计算序列中的匹配项
> [SE-0220](https://github.com/apple/swift-evolution/blob/master/proposals/0220-count-where.md) 引入了一个新的 `count(where:)` 方法，该方法执行 `filter()` 的等价方法并在一次传递中计数。这样可以节省立即丢弃的新阵列的创建，并为常见问题提供清晰简洁的解决方案。
> 注意：**这个 Swift 5.0 功能在 beta 版中被撤销，因为它导致了类型检查器的性能问题。希望它能够在 Swift 5.1 回归 **

```swift
let scores = [100, 80, 85]
let passCount = scores.count { $0 >= 85 }
```


## 提问环节
### Question one 

- 以下写法编译能正常通过吗？

```swift
/// 写法一
class IOSDevice {
    struct iPhone {}
    struct iPad {}
    struct Mac {}
    
    func setUp(product: iPhone) {
        print("iPhone is bought")
    }
    
    func setUp(product: iPad) {
        print("iPad is bought")
    }
    
    func setUp(product: Mac) {
        print("Mac is bought")
    }
}

/// 写法二
class Book {
    struct IOS { }
    func makeIOS() {
        func add(item: IOS) {
            print("Adding IOS…")
        }
        add(item: IOS())
        print("Come and get some book!")
    }
}

/// 写法二
class Cookies {
    struct Butter { }
    struct Flour { }
    struct Sugar { }
    func makeCookies() {
        func add(item: Butter) {
            print("Adding butter…")
        }

        func add(item: Flour) {
            print("Adding flour…")
        }

        func add(item: Sugar) {
            print("Adding sugar…")
        }
        add(item: Butter())
        add(item: Flour())
        add(item: Sugar())
        print("Come and get some cookies!")
    }
}
```
### Question two

- 以下写法是回调哪个正确？ a. 写法二正确  b. 写法一正确 c. 以下都存在错误 d. 写法一和二都正确

```swift
/// 写法一
func appendString(string: String, otherString: String, moreStrings: String...) -> String {
    var resurltString = string + " " + otherString
    resurltString += moreStrings.reduce("", { (x, y) -> String in
        x + " " + y
    });
    return resurltString
}
let res = appendString(string: "Hello", otherString: "I", moreStrings: "am", "a", "good", "man")

/// 写法二
func insertString(string: String, otherString: String..., moreStrings: String...) -> String {
    var resurltString = string + " "
    resurltString += otherString.reduce("", { (x, y) -> String in
        x + " " + y
    });
    
    resurltString += moreStrings.reduce("", { (x, y) -> String in
        x + " " + y
    });
    return resurltString
}

let res2 = insertString(string: "Hello", otherString: "I", "am", moreStrings: "a", "good", "man")
```
### Question three

- 以下写法是否正确？如错误请纠正，如果正确请回答输出的结果

```swift
import Foundation
func OuterFunc(_ a:Int) -> (Int)->Int
{
    var start = 0
    func InnerFunc(b:Int)->Int
    {
        start += a*b
        return start
    }
    return InnerFunc
}
 
var funtest = OuterFunc(2)
 
print(funtest(5))
print(funtest(5))  
```

> Swift 5.4 新特性这里就不做分享~ 感兴趣的参考链接可以查阅

## 参考

- [whatsnewinswift](https://www.whatsnewinswift.com/)
- [[译] Swift 5.0 新特性](https://juejin.cn/post/6844903812184932366)
- [Swift 5.4 的新特性](https://juejin.cn/post/6939869067821973540)
- [demo](https://github.com/zeqinjie/SwiftNewsDemo)
