---
layout: post
title:  '使用pdb来debug python程序'
date:   2016-03-05 22:29 +0800
comments: true
category: python
tags: [Python]
---

Python的命令行debug程序pdb笔记

## 使用PDB打开python脚本

```python
'''
方法1：使用python -m pdb <your script>
相当于在第一行程序之前设置一个断点
'''
$ python -m pdb test.py
> /root/test.py(3)<module>()
-> import sys,os

'''
方法2：在程序代码中添加pdb断点
比方法1方便，但是会更改源程序，调试完得全部删除
'''
import pdb

print('hello')
pdb.set_trace()
print('world')
```

## pdb常用命令

```
h => 显示帮助，h后面可以跟上具体的指令来显示该指令的文档
l => 浏览代码，默认是从第一行开始的11行，连续使用可以翻页。可以添加参数l(ist) [first [,last]]，比如l 0能回到行首。
b => 设置断点，后面跟行号，会回显断点号
cl => 清除断点，后面跟断点号可以清除指定的断点
disable => 禁用断点，用法如cl，用enable解禁
n => 下一行。不进入函数体
s => 下一行，进入函数体
c => 执行程序直到断点
p => 后跟变量，打印变量值

TIPS：在pdb中直接回车一般会执行上一条指令，可以更方便的用n/s跟踪程序或者用l翻页查看
```