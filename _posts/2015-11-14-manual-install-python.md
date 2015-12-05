---
layout: post
title:  '手工安装和配置Python'
date:   2015-11-14 23:00 +0800
comments: true
category: python
tags: [Python, Linux]
---

Python2/Python3 多版本安装和配置

## 下载python包

网站：https://www.python.org/ftp/python/  

```
$ wget https://www.python.org/ftp/python/3.4.3/Python-3.4.3.tgz   
$ wget https://www.python.org/ftp/python/2.7.9/Python-2.7.9.tgz  
```

## 配置和安装

```
$ yum install sqlite-devel openssl openssl-devel

$ tar xvzf Python-2.7.9.tgz    
$ ./configure --with-ensurepip=install
$ make  
$ make install

$ tar xvzf Python-3.4.3.tgz    
$ ./configure --with-ensurepip=install
$ make  
$ make altinstall
```

注意对于多个版本的python，主要版本使用`make install`安装，其他版本使用`make altinstall`安装！  
在python 2.7或者3.4之后，使用--with-ensurepip=install可以自带pip，非常方便。