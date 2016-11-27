---
layout: post
title:  'Python 反射机制'
date:   2016-05-02 12:18 +0800
comments: true
category: python
tags: [Python]
---

Python 反射机制简单介绍和适用场景。

## Python的反射机制

在获取一个类或模块的某个属性时，通常需要在代码中直接指定属性名，如：   

```python
class MyClass(object):
    int a
    float b
    string c

c = MyClass()

print c.a

```

但如果我们不知道属性名呢？比如属性名是在代码中动态生成并存在某个变量中。在这种情况下就需要借助python的反射机制了。

## hasattr, getattr, setattr

python代码可以通过三个内置函数来实现反射：hasattr, getattr, setattr

```python

class MyClass(object):
    a = 0
    b = 1
    
c = MyClass()

myattr = 'a'
new_value = 123

if hasattr(c, myattr):
    val = getattr(c, myattr)
    print 'Old attribute: %s' % val

    setattr(c, myattr, new_value)
    val = getattr(c, myattr)
    print 'New attribute: %s' % val

```

## 使用eval实现getattr

内置函数eval可以将一段字符串当做python表达式处理，灵活使用eval函数也可以部分实现反射的功能：

```python

class MyClass(object):
    a = 0
    b = 1
    
c = MyClass()

myattr = 'a'
new_value = 123

val = eval('c.%s' % myattr)
print val

```

## 使用callable, dir获得额外信息

内置函数callable可以帮助判断一个属性是否是个函数

```python

class MyClass(object):
    a = 0
    b = 1
    
    def myfunc(self):
        pass
    
c = MyClass()

myattr = 'myfunc'

if hasattr(c, myattr):
    myfunc = getattr(c, myattr)
    if callable(myfunc):
        myfunc()

```

内置函数dir可以帮助列出一个类的所有属性

```python

class MyClass(object):
    a = 0
    b = 1
    
    def myfunc(self):
        pass
        
>>> c = MyClass()
>>> dir(c)
['__class__', '__delattr__', '__dict__', '__doc__', '__format__', '__getattribute__', '__hash__', '__init__', '__module__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'a', 'b']

```
