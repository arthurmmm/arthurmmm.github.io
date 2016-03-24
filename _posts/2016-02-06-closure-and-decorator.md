---
layout: post
title:  'Python的闭包和装饰器'
date:   2016-02-06 21:33 +0800
comments: true
category: python
tags: [Python]
---

Python的闭包和装饰器整理

## 什么是闭包/装饰器？

用我自己的理解来讲：   

- 闭包：一个函数，它的作用是通过输入参数构造另一个函数
- 装饰器：输入参数是一个函数的闭包，可以用语法糖@xxx来使用   
 

举点栗子

```python
'''
闭包例子。实现了一个简单的html标签函数
'''
def func_maker(tag):
    def _func(msg):
        return '<%(tag)s>msg</%(tag)s>' % {'tag': tag}
    return _func

html_maker = func_maker('html')
div_maker = func_maker('div')

print html_maker(div_maker('Div Content'))

'''
装饰器例子。实现了一个简单的函数运行时间统计
'''
import sys, os
import time
from datetime import datetime, timedelta
def runtime(fmt):
    def _runtime(func):
        def _func(*args, **kwargs):
            start_ts = datetime.now()
            func(*args, **kwargs)
            end_ts = datetime.now()
            delta = end_ts - start_ts
            print 'start time: %s' % datetime.strftime(start_ts, fmt)
            print 'runtime: %s' % delta
        return _func
    return _runtime

@runtime('%Y-%m-%d %H:%M:%S')
def testfunc1(sec):
    time.sleep(sec)

@runtime('%H:%M:%S')
def testfunc2(sec):
    time.sleep(sec)

testfunc1(3)
testfunc2(5)
```