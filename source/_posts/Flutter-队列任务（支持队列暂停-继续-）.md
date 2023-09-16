---
title: Flutter 队列任务（支持队列暂停 & 继续...）
date: 2023-09-16 14:59:41
categories: "Flutter"
tags:
  - Flutter
---

最近 Flutter 重构原生首页，大部分的 App 随着产品迭代都会在首页加各自弹窗，比如广告弹窗，推送弹窗，版本更新提示等等。在这种场景下不得不用队列去管理弹窗，让弹窗按顺序去一一弹出。效果如下~

![](/images/2023/Flutter-队列任务/1.gif)

## Use

```
showDialog(
    BuildContext context,
    String title,
  ) {
    queue.add(
      () => twShowAnimationDialog(
        context: context,
        transitionType: TwTransitionType.inFromBottom,
        builder: (ctx) {
          return Material(
            color: Colors.transparent,
            child: Center(
              child: Container(
                height: 200,
                width: 200,
                color: TWTool.randomColor(),
                child: Column(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: [
                    Text(
                      title,
                      style: const TextStyle(
                        fontSize: 12,
                      ),
                    ),
                    ElevatedButton(
                      child: const Text(
                        'open',
                      ),
                      onPressed: () {
                        showDialog(context, TWTool.randomColor().toString());
                      },
                    ),
                    ElevatedButton(
                      child: const Text('Close'),
                      onPressed: () {
                        Navigator.of(context).pop();
                      },
                    ),
                  ],
                ),
              ),
            ),
          );
        },
      ),
    );
  }
```

## Question?

遇到另外一个场景，有些只能在首页弹窗的组件，进入子页面也会弹出，这不是我们产品想要的。

遇到这个问题我也提了个问题，延迟的弹窗怎么限制不再第二页面弹出？ [传送门](https://stackoverflow.com/questions/77032707/flutter-delays-the-display-of-a-dialog-box)，不过并不是我想要答案。

🤔 如果队列组件支持暂停和继续，那结合页面路由管理，只要在进入第二个页面暂停队列，回到首页继续队列，那是不是就可以？

## Just do it!

一开始采用 `dart_queue` 这个组件管理。不过遇到 `cancel` 后它抛出异常后，就不能给继续给队列添加任务了，我也给原作者提了 [pr](https://github.com/rknell/dart_queue/pull/17) ，但作者好像不怎么维护了，其他大佬提 `pr` 没有得到回复。因此决定结合自己场景并稍微修改它。

### 1、Support set task priority

```
final queue = TWQueue();
final t1 = 'testQueue4-1';
final t2 = 'testQueue4-2';
final t3 = 'testQueue4-3';
final t4 = 'testQueue4-4';
final results = <String?>[];
//Queue up a future and await its result
queue.add(
  tag: t1,
  () async {
    await Future.delayed(const Duration(seconds: 1));
    results.add(t1);
    print('res1 = $t1');
  },
);
queue.add(
  tag: t2,
  () async {
    await Future.delayed(const Duration(seconds: 1));
    results.add(t2);
    print('res2 = $t2');
  },
);

queue.add(
  tag: t3,
  () async {
    await Future.delayed(const Duration(seconds: 1));
    results.add(t3);
    print('res3 = $t3');
  },
  priority: TWPriority.low,
);

queue.add(
  tag: t4,
  () async {
    await Future.delayed(const Duration(seconds: 1));
    results.add(t4);
    print('res4 = $t4');
  },
  priority: TWPriority.high,
);
await queue.onComplete;
print('results = $results');
```

### 2、Support remove task

📢 正在执行的队列任务是无法删除~

```
final queue = TWQueue();
final t1 = 'testQueue4-1';
final t2 = 'testQueue4-2';
final t3 = 'testQueue4-3';
final t4 = 'testQueue4-4';
final results = <String?>[];
unawaited(
  queue.add(
    () async {
      await Future.delayed(const Duration(seconds: 1));
      results.add(t1);
    },
    tag: t1,
  ),
);
unawaited(
  queue.add(
    () async {
      await Future.delayed(const Duration(seconds: 1));
      results.add(t2);
    },
    tag: t2,
  ),
);

unawaited(
  queue.add(
    () async {
      await Future.delayed(const Duration(seconds: 1));
      results.add(t3);
    },
    tag: t3,
  ),
);

unawaited(
  queue.add(
    () async {
      await Future.delayed(const Duration(seconds: 1));
      results.add(t4);
    },
    tag: t4,
  ),
);
// remove t2 and t4
queue.remove(t2);
queue.remove(t4);
await queue.onComplete;
// log: results [testQueue4-1, testQueue4-3]
print('results $results');
```

### 3、Support pause and resume task

📢 正在执行的队列任务是无法暂停~

```
final queue = TWQueue();
final results = <String?>[];
final t1 = 'testQueue6-1';
final t2 = 'testQueue6-2';
final t3 = 'testQueue6-3';
final t4 = 'testQueue6-4';

await queue.add(
  () async {
    await Future.delayed(const Duration(seconds: 1));
    results.add(t1);
  },
);
await queue.add(
  () async {
    await Future.delayed(const Duration(seconds: 1));
    results.add(t2);
  },
);
queue.pause();
unawaited(queue.add(
  () async {
    await Future.delayed(const Duration(seconds: 1));
    results.add(t3);
  },
));
unawaited(queue.add(
  () async {
    await Future.delayed(const Duration(seconds: 1));
    results.add(t4);
  },
));
Future.delayed(const Duration(seconds: 1), () {
  // delayed results [testQueue4-1, testQueue4-2]
  print('delayed results $results');
  queue.resume();
});
await queue.onComplete;
// onComplete results [testQueue4-1, testQueue4-2, testQueue4-3, testQueue4-4]
print('onComplete results $results');
```

项目中效果

![](/images/2023/Flutter-队列任务/2.gif)

## Last

调整后已上传 [pub](https://pub.dev/packages/tw_queue)，使用如下

```
dependencies:
  tw_queue: latest_version
```

tw_queue 地址 [传送门](https://github.com/zeqinjie/tw_queue)， 如果你觉得对你有帮助，请不吝给个 Star 。如果有问题，也请大佬们指点 ~ 感谢！

## Thx

-  fork from [dart_queue](https://github.com/rknell/dart_queue)
