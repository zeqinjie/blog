---
title: 公司夺旗比赛-iOS题
date: 2018-11-21 19:02:59
categories: "iOS"
tags:
  - iOS
  - addcn
---

# 前言 
> 公司每年的6月份会举办一场夺旗大赛，作为读书会的小组成员。我就负责出了一份比较简单的iOS题
[掘金地址](https://juejin.im/post/5bf4b76af265da61602c8ddd)

##  CTF移动端题目

### 1.灰原同学提出可以查看监控，但监控室的门是用密码锁上的，门上只有了一串英文字母MjM4MzIxNDE0Mg==,你可以帮助少年侦探团打开门吗？

##### 解题思路
```
base64解码得到2383214142 
通过手机键盘-九宫格得到
答案 CVAGH
```

### 2.在暗格门里发现作案者留下的[文件包](https://pan.baidu.com/s/1nxddHhFBr8BjTJkoZn5a-w)，但是却需要账户和密码才能打开，通过下面的代码和文件你能帮助柯南找出答案吗？


```
@interface MyAccount : NSObject 

@property (nonatomic, copy) NSString *flag;

@end

@implementation MyAccount

@end
```
##### 解题思路

```
1、.m文件实现NSCoding协议，补充代码如下

#import "MyAccount.h"
@interface MyAccount()<NSCoding>
@end
@implementation MyAccount

- (void)encodeWithCoder:(NSCoder *)aCoder{
    [aCoder encodeObject:self.flag forKey:@"flag"];
}

- (id)initWithCoder:(NSCoder *)decoder {
    if (self = [super init]) {
        _flag = [decoder decodeObjectForKey:@"flag"];
    }
    return self;
}
@end


2、导入data包，NSKeyedUnarchiver解档对象，得到flag: c2h1aXJ1aWtlamljdGY=

NSString *dataFile = [[NSBundle mainBundle]pathForResource:@"data" ofType:@"asd"];
MyAccount *model = [NSKeyedUnarchiver unarchiveObjectWithFile:dataFile];
NSLog(@"dataFile = %@",model.flag);

3、使用base64解码得到
答案： shuiruikejictf
```

### 3.侦探少年团解答出账户和密码后并打开文件包后，发现盗贼留下了一份挑战书“你想要的密码就在我的ipa包中”，你能从[MySercretKey.ipa](https://pan.baidu.com/s/1YiNuM3bLXj4-C_2jUsU9Gw)包中读取到答案吗？


##### 解题思路
```
使用Hopper Disassembler工具反编译
如下图读取到
flag = U2FsdGVkX18wjamzHeMlywW3nE/EPSImPYlN25ihcf0=
decode = shurui

使用AES 解密（密钥shurui）
答案:flag = asdxczcsa
