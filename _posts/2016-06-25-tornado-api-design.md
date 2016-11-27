---
layout: post
title:  '使用Tornado实现RESTFUL Style API(1) -- 搭建和设计API服务'
date:   2016-06-25 20:07 +0800
comments: true
category: python
tags: [Python, Tornado]
---

## API需求

项目开发上经常需要从其他组的数据源那里去读取数据，而公司内部开发组太多，每个组使用的技术各不相同，还有各种安全认证，往往使数据访问、转换的过程变得非常痛苦。在这种情况下一个跨平台，标准化的API中间件能够很好的将数据处理逻辑与业务逻辑分隔开来。   

## 技术选型

现在主流的Python Web框架包括Django、Flask和Tornado。Django太过笨重首先被排除了，Tornado和Flask比较起来并发处理更强，更适合用作API Server。  

## 代码架构

项目结构如下：

```
|- start.py
|- base.py
|- setting.py
|- api_gui.html
|- API
  |- __init__.py
  |- index.py
  |- ...
|- CONFIG
  |- __main__.yml
  |- base.yml
  |- API.index.yml
  |- ...
|- static
  |- favicon.icon
  |- ...
```

主目录下start.py用于启动服务器，base.py定义了一些继承类和模板，setting.py用于管理配置文件，api_gui是API图形界面的HTML模板。   
API下存放了各个API的业务逻辑，理论上可以做到在添加新的API时只需要添加一个模块到API包即可。   
CONFIG下存放了各种配置文件，各种常量都会定义在配置文件中。这里用了YAML作为配置文件的格式 -- 个人比较喜欢YAML，因为它和JSON一样简介，又有XML差不多的表达能力，还可以添加注释..   
python下操作YAML文件可以使用PyYAML: http://pyyaml.org/    

## start.py

我们将URL和Handler的映射关系拆分到各个API Module，这样在每个API模块中可以很直观的看到它对应的URL，在新添加API时也无需更改start.py   
实现方法是在API包中添加router变量，然后在start.py中通过getattr来获取。   

另外我们将server address, port等启动函数写到了配置文件中，再通过**kwargs的方式传递给Application类，其中default_handler_class由于是一个类，需要使用eval进行转化   

```python

def make_app():
    import base, API
    url_router = []
    for a in dir(API):
        if not a.startswith('__'):
            api_module = getattr(API, a)
            api_url_router = getattr(api_module, 'router')
            url_router.extend(api_url_router)
    url_router.append(
        URLSpec(r'/(favicon.icon)', StaticFileHandler, {'path': config.STATIC_PATH}, name='favicon')
    )
    application_args = config.APPLICATION
    if 'default_handler_class' in application_args:
        application_args['default_handler_class'] = eval(application_args['default_handler_class'])
    return Application(url_router, **application_args)
    
```