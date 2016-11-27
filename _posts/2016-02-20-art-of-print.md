---
layout: post
title:  'Linux下实现彩色/动态的命令行输出'
date:   2016-02-20 17:53 +0800
comments: true
category: shell
tags: [Python, shell]
---

做项目有实时、动态显示结果的需求，并且要用红色高亮错误，记录一下实现方法

## 彩色输出

SHELL本身可以通过ANSI控制码来控制输出字体的格式，具体格式如下：    

```bash
\033[<字体格式>;<前景色>;<背景色>m<输出>\033[0m
```

在shell脚本中直接echo或者在python中按照上述格式print，就能显示对应的效果。   
基于这个原理我写了一个简单的python函数，在需要彩色输出的时候给出对应的参数就可以了：   

```python
def color_print(msg, fcolor='white', bcolor='black', mode='normal', end=None):
    fcolor_map = {
        'black': 30,
        'red': 31,
        'green': 32,
        'yellow': 33,
        'blue': 34,
        'purple': 35,
        'cyan': 36,
        'white': 37,
    }
    bcolor_map = {
        'black': 40,
        'red': 41,
        'green': 42,
        'yellow': 43,
        'blue': 44,
        'purple': 45,
        'cyan': 46,
        'white': 47,
    }
    mode_map = {
        'normal': 0,
        'bold': 1,
        'underline': 4,
        'blink': 5,
        'invert': 7,
        'hide': 8,
    }
    msg = '\033[%(mode)s;%(fcolor)s;%(bcolor)sm%(msg)s\033[0m' % {
        'mode': mode,
        'fcolor': fcolor,
        'bcolor': bcolor,
        'msg': msg,
    }
    if (not end):
        print(msg)
    else:
        print(msg,end=end)
```

## 动态输出

在我上面的代码中用到了print函数中的end选项，这个是python3.x中print函数非常实用的一个选项，简单来讲就是替换掉print的结束符--在默认情况下实用print都会在结尾自动跟一个换行符。   

所以如果你不希望print换行，就可以`print(msg,end='')`，而为了实现动态输出，我们需要在print完之后将光标移到行首，对应的转移字符就是`\r`，因此使用`print(msg,end='\r')`就可以让光标在每次print完后回到行首了。    
另外在print完后记得使用`sys.stdout.flush()`来及时显示信息。      

一些TIPS：   

- 在python2中可以通过`from __future__ import print_function`来使用python3的print功能。
- 在动态刷新当前行的时候，如果前一行的字符串比较长，多出来的部分不会被清掉，记得在结尾补上空白符来清除。