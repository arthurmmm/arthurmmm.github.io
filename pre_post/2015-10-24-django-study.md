---
layout: post
title:  'Django MVC框架学习'
date:   2015-10-24 23:00 +0800
comments: true
category: web-build
tags: [Django, Python]
---

现在的动态站点开发，除了传统的WEB->CGI->DATABASE模式就是MVC框架了。   
本文介绍的Django即是Python最流行的MVC框架。  
Django的官方网站戳[这里](https://www.djangoproject.com/) 。

## 安装Django

建议使用pip安装。pip的简单实用方法在["NO SQL in your code! SQLAlchemy学习"](http://www.arthurmao.me/2015/10/sqlalchemy-intro/) 一文中有所提及可供参考。

```
Python 3.x

# pip install django
Downloading/unpacking django
  Downloading Django-1.8.5-py2.py3-none-any.whl (6.2MB): 6.2MB downloaded
Installing collected packages: django
Successfully installed django
Cleaning up...

Python 2.x

# python -m pip install django
Downloading/unpacking django
  Downloading Django-1.8.5-py2.py3-none-any.whl (6.2MB): 6.2MB downloaded
Installing collected packages: django
Successfully installed django
Cleaning up...
```
