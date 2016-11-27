---
layout: post
title:  'Python subprocess模块'
date:   2016-01-16 21:02 +0800
comments: true
category: python
tags: [Python]
---

Python subprocess通常用来调用并获得shell指令的输出，这里整理一下用法.
官方文档：https://docs.python.org/2.7/library/subprocess.html

## subprocess 模块的基本用法

```python
import subprocess

cmd = ''' echo "Your shell command" '''
p = subprocess.Popen(cmd, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
res, err = p.communicate()

'''
设置shell=True会存在shell injection的安全隐患，出于安全性考虑可以这么用
'''
cmd = ['echo', '"Your shell command"']
p = subprocess.Popen(cmd, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
res, err = p.communicate()
```

## 使用subprocess动态获取输出

有时候我们需要实时跟踪一个指令的输出，比如监控一个长ping的结果：

```python
import subprocess

cmd = ''' ping localhost -c 5 '''
p = subprocess.Popen(cmd, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
while True:
    out = p.stdout.readline()
    if (out == ''):
        break
    print 'Am in subprocess => %s' % out
```

## Python2 VS Python3

Python3对subprocess模块有了一些很不错的改进，比如：  

```python
'''
1. Python3 支持context manager：通过dir指令可以看出Python3的subprocess模块存在__enter__和__exit__函数
'''
# python2
Python 2.7.9 (default, Nov 14 2015, 17:52:58)
[GCC 4.8.3 20140911 (Red Hat 4.8.3-9)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import subprocess
>>> dir(subprocess.Popen)
['__class__', '__del__', '__delattr__', '__dict__', '__doc__', '__format__', '__getattribute__', '__hash__', '__init__', '__module__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', '_child_created', '_close_fds', '_communicate', '_communicate_with_poll', '_communicate_with_select', '_execute_child', '_get_handles', '_handle_exitstatus', '_internal_poll', '_set_cloexec_flag', '_translate_newlines', 'communicate', 'kill', 'pipe_cloexec', 'poll', 'send_signal', 'terminate', 'wait']

# python3
Python 3.4.3 (default, Nov 14 2015, 18:34:36)
[GCC 4.8.3 20140911 (Red Hat 4.8.3-9)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import subprocess
>>> dir(subprocess.Popen)
['__class__', '__del__', '__delattr__', '__dict__', '__dir__', '__doc__', '__enter__', '__eq__', '__exit__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', '_check_timeout', '_child_created', '_close_fds', '_communicate', '_execute_child', '_get_devnull', '_get_handles', '_handle_exitstatus', '_internal_poll', '_remaining_time', '_save_input', '_translate_newlines', '_try_wait', 'communicate', 'kill', 'poll', 'send_signal', 'terminate', 'wait']


# 因此在Python3中，正确打开一个Popen的姿势应该是这样的
with subprocess.Popen(cmd) as p:
    res,err = p.communicate()
    
'''
2. Python3 中的subprocess支持timeout选项，在执行可能会hung的指令时非常有用！
'''
cmd = ''' sleep 10 '''
with subprocess.Popen(cmd, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE) as p:
    try:
         out,res = p.communicate(timeout=5)
     except subprocess.TimeoutExpired:
         p.kill()
         print('Command timeout!!')

'''
3. Python3中subprocess的返回值是byte，如果要用作string的话需要做个转换
'''
cmd = ''' echo "hello" '''
with subprocess.Popen(cmd, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE) as p:
    try:
         out,res = p.communicate(timeout=5)
         out = out.decode()
         print(out)
     except subprocess.TimeoutExpired:
         p.kill()
         print('Command timeout!!')
         
```
