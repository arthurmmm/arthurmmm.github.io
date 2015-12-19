---
layout: post
title:  'Python命令行利器logging和argparse模块'
date:   2015-12-05 20:22 +0800
comments: true
category: os
tags: [Python]
---

argparse模块和logging模块的简单介绍

## logging 模块的基本用法

```python
import logging

'''
level设定显示log的级别
datefmt是时间的显示方式
FORMAT是一个自定义的字符串，用于定义日志的格式，非常重要。
'''
logging.basicConfig(level=logging.DEBUG,
    format = FORMAT,
    datefmt = '%Y-%m-%d %H:%M:%S')


'''
基本格式，显示时间、事件级别和日志信息
'''
FORMAT = '%(asctime)s %(levelname)s: %(message)s'

'''
额外显示了线程号，模块名和代码行号，调试的时候非常有用
'''
FORMAT = '%(thread)d %(asctime)s %(levelname)s %(module)s/%(lineno)d: %(message)s'

'''
还可以自定义一些额外的属性
'''
FORMAT='%(thread)d %(asctime)s %(levelname)s %(module)s/%(lineno)d [%(username)s]: %(message)s'
d = {'username':'arthur'}
logger.warning('hihi',extra=d)

'''
或者将日志写入文件
'''
FORMAT = '%(asctime)s %(levelname)s: %(message)s'
logging.basicConfig(format=FORMAT,
        filename='test.log'
)
```

## logging模块的一些参考表格

Format|Description
---|---|---
filename|Specifies that a FileHandler be created, using the specified filename, rather than a StreamHandler.
filemode|Specifies the mode to open the file, if filename is specified (if filemode is unspecified, it defaults to ‘a’).
format|Use the specified format string for the handler.
datefmt|Use the specified date/time format.
level|Set the root logger level to the specified level.
stream|Use the specified stream to initialize the StreamHandler. Note that this argument is incompatible with ‘filename’ - if both are present, ‘stream’ is ignored.