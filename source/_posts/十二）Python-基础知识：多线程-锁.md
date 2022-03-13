---
title: 十二）Python 基础知识：多线程 & 锁
date: 2022-03-13 15:12:00
categories: "Python"
tags:
	- Python
---

## 多线程

```python
import threading
import time
from threading import current_thread


# 定义线程方法
def my_thread(arg1, arg2):
    print(threading.currentThread().getName(), 'start')
    print('%s %s' % (arg1, arg2))
    time.sleep(1)
    print(threading.currentThread().getName(), 'end')


# 测试
def test_func():
    for i in range(1, 6):
        t1 = threading.Thread(target=my_thread, args=(i, i + 1))
        t1.start()
    print(current_thread().getName(), 'end')


class MyThread(threading.Thread):
    def run(self):
        print(current_thread().getName(), 'start')
        print('run')
        print(current_thread().getName(), 'stop')


def test_func2():
    t1 = MyThread()
    t1.start()
    t1.join()
    print(current_thread().getName(), 'end')


# test_func()
test_func2()

```

### Thread 方法

- **run():** 用以表示线程活动的方法。
- **start():**启动线程活动。
- **join([time]):** 等待至线程中止。这阻塞调用线程直至线程的join() 方法被调用中止-正常退出或者抛出未处理的异常-或者是可选的超时发生。
- **isAlive():** 返回线程是否活动的。
- **getName():** 返回线程名。
- **setName():** 设置线程名。


### 经典的消费者和生产者

```python
from threading import Thread, current_thread
import time
import random
from queue import Queue

queue = Queue(5)
# 经典的消费者和生产者问题

class ProducerThread(Thread):
    def run(self):
        name = current_thread().getName()
        nums = range(100)
        global queue
        while True:
            num = random.choice(nums)
            queue.put(num)
            print('生产者 %s 生产了数据 %s' % (name, num))
            t = random.randint(1, 3)
            time.sleep(t)
            print('生产者 %s 睡眠了 %s 秒' % (name, t))


class ConsumerTheard(Thread):
    def run(self):
        name = current_thread().getName()
        global queue
        while True:
            num = queue.get()
            queue.task_done()
            print('消费者 %s 消耗了数据 %s' % (name, num))
            t = random.randint(1, 5)
            time.sleep(t)
            print('消费者 %s 睡眠了 %s 秒' % (name, t))

def test_func():
    p1 = ProducerThread(name='p1')
    p1.start()
    p2 = ProducerThread(name='p2')
    p2.start()
    p3 = ProducerThread(name='p3')
    p3.start()
    c1 = ConsumerTheard(name='c1')
    c1.start()
    c2 = ConsumerTheard(name='c2')
    c2.start()

test_func()

```

### 线程同步（锁）

- 创建锁：lock = threading.Lock()
- 加锁：lock.acquire()
- 解锁：lock.release()

```python
from queue import Queue
import threading
import time

exitFlag = 0

class myThread(threading.Thread):
    def __init__(self, threadID, name, q):
        threading.Thread.__init__(self)
        self.threadID = threadID
        self.name = name
        self.q = q

    def run(self):
        print("Starting " + self.name)
        process_data(self.name, self.q)
        print
        "Exiting " + self.name


def process_data(threadName, q):
    while not exitFlag:
        queueLock.acquire()
        if not workQueue.empty():
            data = q.get()
            queueLock.release()
            print("%s processing %s" % (threadName, data))
        else:
            queueLock.release()
        time.sleep(1)


threadList = ["Thread-1", "Thread-2", "Thread-3"]
nameList = ["One", "Two", "Three", "Four", "Five"]
queueLock = threading.Lock()
workQueue = Queue(10)
threads = []
threadID = 1

# 创建新线程
for tName in threadList:
    thread = myThread(threadID, tName, workQueue)
    thread.start()
    threads.append(thread)
    threadID += 1

# 填充队列
queueLock.acquire()
for word in nameList:
    workQueue.put(word)
queueLock.release()

# 等待队列清空
while not workQueue.empty():
    pass

# 通知线程是时候退出
exitFlag = 1

# 使用 json 方法等待所有线程完成
for t in threads:
    t.join()
print("Exiting Main Thread")
```

### 线程合并（join方法）
需要主线程要等待子线程运行完后，再退出可以使用 join 方法

```python
# 创建的 thread 调用 join 确保子线程结束
def test_func2():
    t1 = MyThread()
    t1.start()
    t1.join()
    print(current_thread().getName(), 'end')
```

### 线程间通信
从一个线程向另一个线程发送数据最安全的方式可能就是使用 queue 库中的队列了。创建一个被多个线程共享的 Queue 对象，这些线程通过使用 put() 和 get() 操作来向队列中添加或者删除元素。
#### 
#### Queue 方法

- Queue.qsize() 返回队列的大小
- Queue.empty() 如果队列为空，返回True,反之False
- Queue.full() 如果队列满了，返回True,反之False
- Queue.full 与 maxsize 大小对应
- Queue.get([block[, timeout]])获取队列，timeout等待时间
- Queue.get_nowait() 相当Queue.get(False)
- Queue.put(item, block=True, timeout=None) 写入队列，timeout等待时间
- Queue.put_nowait(item) 相当 Queue.put(item, False)
- Queue.task_done() 在完成一项工作之后，Queue.task_done()函数向任务已经完成的队列发送一个信号
- Queue.join() 实际上意味着等到队列为空，再执行别的操作


## 参考

- [菜鸟学习](https://www.runoob.com/python/python-multithreading.html)
- [草根学Python - 多线程编程](https://www.readwithu.com/Article/PythonBasis/python13/2.html)
- [demo](https://github.com/zeqinjie/python_demo)
- [Python中的global关键字](https://www.cnblogs.com/ronyjay/p/12792786.html)

