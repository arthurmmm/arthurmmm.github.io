---
layout: post
title:  'Python logging模块介绍'
date:   2015-12-05 20:22 +0800
comments: true
category: os
tags: [Python]
---

Python logging模块介绍

## logging 模块的基本用法

```python
import logging

'''
level设定显示log的级别
datefmt是时间的显示方式
FORMAT是一个自定义的字符串，用于定义日志的格式。
'''
logging.basicConfig(level=logging.DEBUG,
    format = FORMAT,
    datefmt = '%Y-%m-%d %H:%M:%S')

'''
默认情况下日志输出到屏幕，
需要写文件时得加上filename属性
'''
logging.basicConfig(format=FORMAT,
        filename='test.log'
)
    
'''
需要打印信息的时候这样做
'''
logging.debug('a debug message')
logging.info('a debug message')
logging.warning('a debug message')
logging.error('a debug message')
logging.critical('a debug message')

'''
也可以打印异常堆栈
'''
logging.exception('a debug message')

'''
推荐的基本格式，显示时间、事件级别和日志信息
'''
FORMAT = '%(asctime)s %(levelname)s: %(message)s'

'''
额外显示了线程号，模块名和代码行号，调试的时候非常有用
'''
FORMAT = '%(thread)d %(asctime)s %(levelname)s %(module)s/%(lineno)d: %(message)s'

'''
还可以自定义一些额外的字段，例如下面的username，在打印时需要用extra指定一个dict用于解释自定义字段
'''
FORMAT='%(thread)d %(asctime)s %(levelname)s %(module)s/%(lineno)d [%(username)s]: %(message)s'
d = {'username':'arthur'}
logger.warning('hihi',extra=d)

```

## logging模块的一些参考表格

### basicConfig的属性

Format|Description
---|---|---
filename|Specifies that a FileHandler be created, using the specified filename, rather than a StreamHandler.
filemode|Specifies the mode to open the file, if filename is specified (if filemode is unspecified, it defaults to ‘a’).
format|Use the specified format string for the handler.
datefmt|Use the specified date/time format.
level|Set the root logger level to the specified level.
stream|Use the specified stream to initialize the StreamHandler. Note that this argument is incompatible with ‘filename’ - if both are present, ‘stream’ is ignored.

### FORMAT中可以使用的参数

Attribute|Format|Description
---|---|---
args|You shouldn’t need to format this yourself.|The tuple of arguments merged into msg to produce message, or a dict whose values are used for the merge (when there is only one argument, and it is a dictionary).
asctime|%(asctime)s|Human-readable time when the LogRecord was created. By default this is of the form ‘2003-07-08 16:49:45,896’ (the numbers after the comma are millisecond portion of the time).
created|%(created)f|Time when the LogRecord was created (as returned by time.time()).
exc_info|You shouldn’t need to format this yourself.|Exception tuple (à la sys.exc_info) or, if no exception has occurred, None.
filename|%(filename)s|Filename portion of pathname.
funcName|%(funcName)s|Name of function containing the logging call.
levelname|%(levelname)s|Text logging level for the message ('DEBUG', 'INFO', 'WARNING', 'ERROR', 'CRITICAL').
levelno|%(levelno)s|Numeric logging level for the message (DEBUG, INFO, WARNING, ERROR, CRITICAL).
lineno|%(lineno)d|Source line number where the logging call was issued (if available).
module|%(module)s|Module (name portion of filename).
msecs|%(msecs)d|Millisecond portion of the time when the LogRecord was created.
message|%(message)s|The logged message, computed as msg % args. This is set when Formatter.format() is invoked.
msg|You shouldn’t need to format this yourself.|The format string passed in the original logging call. Merged with args to produce message, or an arbitrary object (see Using arbitrary objects as messages).
name|%(name)s|Name of the logger used to log the call.
pathname|%(pathname)s|Full pathname of the source file where the logging call was issued (if available).
process|%(process)d|Process ID (if available).
processName|%(processName)s|Process name (if available).
relativeCreated|%(relativeCreated)d|Time in milliseconds when the LogRecord was created, relative to the time the logging module was loaded.
thread|%(thread)d|Thread ID (if available).
threadName|%(threadName)s|Thread name (if available).