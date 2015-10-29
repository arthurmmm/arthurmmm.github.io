---
layout: post
title:  '配置安装CentOS'
date:   2015-10-24 23:00 +0800
comments: true
category: os
tags: [centOS, Linux]
---

配置和安装CentOS的笔记，包括：配置国内源，配置jekyll，打开防火墙端口

## 安装过程

* 配置IP信息
* 选择webserver安装并勾选python/perl/java等常用开发包

## 配置国内源

* 进入[浙大源](http://mirrors.zju.edu.cn/)
* 在配置生成器中选择CentOS及对应版本，按提示设置`CentOS-Base.repo`

```
[root@ArthurCentOS yum.repos.d]# vim CentOS-Base.repo
[root@ArthurCentOS yum.repos.d]# ll
total 28
-rw-r--r--. 1 root root  526 Oct 26 23:04 CentOS-Base.repo
-rw-r--r--. 1 root root 1664 Apr  1  2015 CentOS-Base.repo.save
-rw-r--r--. 1 root root 1309 Apr  1  2015 CentOS-CR.repo
-rw-r--r--. 1 root root  649 Apr  1  2015 CentOS-Debuginfo.repo
-rw-r--r--. 1 root root  290 Apr  1  2015 CentOS-fasttrack.repo
-rw-r--r--. 1 root root 1331 Apr  1  2015 CentOS-Sources.repo
-rw-r--r--. 1 root root 1002 Apr  1  2015 CentOS-Vault.repo
[root@ArthurCentOS yum.repos.d]# yum makecache
Loaded plugins: fastestmirror, langpacks
base                                                                                                                                | 3.6 kB  00:00:00
extras                                                                                                                              | 3.4 kB  00:00:00
updates                                                                                                                             | 3.4 kB  00:00:00
updates/7/x86_64/other_db                                                                                                           |  25 MB  00:00:01
Determining fastest mirrors

Metadata Cache Created
```

* 使用`yum update`更新系统包

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