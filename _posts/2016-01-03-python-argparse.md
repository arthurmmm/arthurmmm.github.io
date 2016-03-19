---
layout: post
title:  'Python argparse模块'
date:   2016-01-03 11:09 +0800
comments: true
category: python
tags: [Python]
---

Python有很多命令行选项相关的模块，比如getopt, optparse等，各种都用过一下后感觉还是argparse最方便最轻量。    
官方详细文档：  https://docs.python.org/2.7/library/argparse.html

## argparse 模块的基本用法

```python
import logging

'''
第一步：创建parser实例
'''
parser = argparse.ArgumentParser(description='Write overall help message here')

'''
第二步：创建参数和选项
'''
parser.add_argument('myargs', nargs='+', 
    help='My arguments')
parser.add_argument('-o', '--myopts', default='hello'
    help='My options')
parser.add_argument('-f', '--myflag', action='store_true'
    help='My flag')

'''
第三步：获取参数
'''
args = parser.parse_args()
myargs = args.myargs
myopts = args.myopts
myflag = args.myflag
```

## argparse 实用选项

以下是一些常见的参数，通常已经能满足一般命令行工具的需求了。

```

default：参数默认值。

nargs: 参数个数，若有多个参数则用空格分隔
注意如果有配置nargs（哪怕nargs=1），那么拿到的参数是一个list，否则就是一个值

action：执行操作，通常设置为store_true用于无需接受参数的选项比如--debug这种。

type：输入值的类型，其实就是帮你做了一个显示类型转换，不指定的话默认是string

choice：限制输入值的种类，例如choice=['choiceA', 'choiceB']

help：简单说明

```