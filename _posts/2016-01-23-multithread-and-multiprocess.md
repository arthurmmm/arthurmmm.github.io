---
layout: post
title:  'Python的多线程与多进程'
date:   2016-01-23 17:06 +0800
comments: true
category: python
tags: [Python]
---

Python的多线程与多进程笔记

## 多线程与多进程的区别和适用场景

一般来讲多线程比多进程更轻量，消耗资源更少，因为线程之间是共享内存的而进程都拥有自己独立的内存。   
但是Python(CPython)在多线程的实现方面受限与GIL，在执行CPU密集型程序时并不能体现出优势，所以对于这类程序通常使用多进程实现。
另外python的threading模块并没有实现kill/terminate之类的强制退出方法，自己实现的话比较ugly，所以对于那些无法通过设置标志位退出的程序还是用multiprocess比较方便。  

## python的多线程模块threading

官方文档：https://docs.python.org/2.7/library/threading.html   
一个简单的示例程序：

```python
import sys, os
import time
from datetime import datetime, timedelta
import threading
'''
Queue.Queue是一个线程安全队列，非常适合用于线程间数据传递以及生产者消费者模型
'''
from Queue import Queue

def threadTask(qin, qout):
    '''
    threading.current_thread => 获取当前线程的实例
    '''
    cur_thd = threading.current_thread()
    while qin.qsize() > 0:
        inputs = qin.get()
        print('Thread %s is running...' % cur_thd.ident)
        print('Accept input: %s' % inputs)
        print('Sleeping %s seconds' % inputs)
        time.sleep(inputs)
        qout.put(inputs+1)
    print('Thread %s is exiting...' % cur_thd.ident)

def main():
    inputs = [2,3,4,60,70]
    qin = Queue()
    for i in inputs:
        qin.put(i)
    qout = Queue()
    thd_list = []
    for i in range(3):
        thd = threading.Thread(target=threadTask, kwargs = {
            'qin': qin,
            'qout': qout,
        })
        '''
        设置为守护线程：当主线程结束时自动终止这些线程
        如果没有设置daemon=True，当timeout发生后由于子线程仍未退出，主线程会继续等待
        '''
        thd.daemon = True
        thd.start()
        thd_list.append(thd)

    for thd in thd_list:
        '''
        join => 主线程阻塞在此处等待子线程结束，可设置timeout。
        timeout发生后主线程取消阻塞，但子线程依然在后台执行
        '''
        thd.join(5)
        '''
        threading.is_alive可以用来判断子线程是超时退出还是超时退出
        '''
        if (thd.is_alive()):
            print('Thread %s is timeout!!' % thd.ident)
    
    '''
    从qout中获取输出
    '''
    while qout.qsize():
        out = qout.get()
        print('=> %s' % out)

if __name__ == '__main__':
    main()
```

另一种实现方法--继承threading.Thread类并重写run函数

```python
class MyThread(threading.Thread):
    qin = None
    qout = None
    def __init__(self, qin, qout):
        self.qin = qin
        self.qout = qout
        threading.Thread.__init__(self)

    def run(self):
        while self.qin.qsize() > 0:
            inputs = self.qin.get()
            print('Thread %s is running...' % self.ident)
            print('Accept input: %s' % inputs)
            print('Sleeping %s seconds' % inputs)
            time.sleep(inputs)
            self.qout.put(inputs+1)
        print('Thread %s is exiting...' % self.ident)

def main():
    inputs = [3,4,50,60,70]
    qin = Queue()
    for i in inputs:
        qin.put(i)
    qout = Queue()
    thd_list = []
    for i in range(3):
        thd = MyThread(qin, qout)
        thd.daemon = True
        thd.start()
        thd_list.append(thd)

    for thd in thd_list:
        thd.join(5)
        if (thd.isAlive()):
            print('Thread %s is timeout!!' % thd.ident)

    while qout.qsize():
        out = qout.get()
        print('=> %s' % out)

if __name__ == '__main__':
    main()
```

## python的多进程模块multiprocessing

Python的multiprocessing模块可以用类似多线程的方法实现多进程
官方文档：https://docs.python.org/2.7/library/multiprocessing.html

```python
import sys, os
import time
from datetime import datetime, timedelta
import threading
import multiprocessing
'''
用multiprocessing.Queue替换Queue.Queue
'''
# from Queue import Queue
from multiprocessing import Queue

def processTask(qin, qout):
    while qin.qsize() > 0:
        '''
        os.getpid()直接可以获取当前进程的进程号
        '''
        print('Process %s is running...' % os.getpid())
        inputs = qin.get()
        print('Accept input: %s' % inputs)
        print('Sleeping %s seconds' % inputs)
        time.sleep(inputs)
        qout.put(inputs+1)
    print('Process %s is exiting...' % os.getpid())

def main():
    inputs = [2,3,4,60,70]
    qin = Queue()
    for i in inputs:
        qin.put(i)
    qout = Queue()
    proc_list = []
    for i in range(3):
        proc = multiprocessing.Process(target=processTask, kwargs={
            'qin': qin,
            'qout': qout,
        })
        proc.deamon = True
        proc.start()
        '''
        用proc.pid获取子进程进程号
        '''
        print('Start process %s' % proc.pid)
        proc_list.append(proc)

    for proc in proc_list:
        proc.join(5)
        if (proc.is_alive()):
            print('Process %s is timeout!!' % proc.pid)
            '''
            子进程是可以强制终止的，这点比线程好
            '''
            proc.terminate()

    while qout.qsize():
        out = qout.get()
        print('=> %s' % out)
        
if __name__ == '__main__':
    main()
```

## Lock机制

threading和multiprocessing还提供了各种的Lock机制，但在实际编程中大部分同步问题用Queue就能解决了，这里就简单记录一下。

```python
'''
基本锁
'''
threading.Lock/multiprocess.Lock 

'''
RLock，和Lock的区别在于允许同一线程内多次require而不会死锁
多次require RLock也需要多次release，直到所有的锁被释放
'''
threading.RLock/multiprocess.RLock

'''
Condition，比较高级的锁，支持notify，wait等较复杂的同步
'''
threading.Condition/multiprocess.Condition
```