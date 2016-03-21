---
layout: post
title:  '配置安装CentOS'
date:   2015-10-24 23:00 +0800
comments: true
category: OS
tags: [centOS, Linux]
---

配置和安装CentOS的笔记，包括：配置国内源，配置jekyll，打开防火墙端口

## 安装过程

* 配置IP信息
* 选择webserver安装并勾选python/perl/java等常用开发包

## 配置国内源

推荐使用中科大源：https://lug.ustc.edu.cn/wiki/mirrors/help

* 配置完成后可以`yum makecache`和`yum update`更新一下系统包

## 安装jekyll

基本与OpenSUSE类似，参考：[OpenSUSE搭建jekyll服务器](http://www.arthurmao.me/2015/10/establish-jekyll-on-suse/)   
要注意的是yum没有自带nodejs，需要安装EPEL库后再安装nodejs：

```
# yum install epel-release
# yum install nodejs
```

## 打开防火墙端口

```
//暂时打开
# firewall-cmd --add-port=4000/tcp

//永久打开
# firewall-cmd --permanent --add-port=4000/tcp

//测试端口状态
# firewall-cmd --permanent --query-port=4000/tcp
yes

//关闭端口
# firewall-cmd --permanent --remove-port=4000/tcp
success
```