---
layout: post
title:  '在OpenSUSE上搭建jekyll服务器'
imagedir:  /images/2015-10-06-00/
date:   2015-10-06 20:10 +0800
comments: true
category: Linux
---

这个博客站点是使用gitpage+jekyll搭建而成的，为了在部署到github前在本地做测试，需要搭建jekyll+gitpage的测试服务器。  
本文记录了在OpenSUSE环境下搭建jekyll+gitpage服务器的过程  

## 安装Ruby

OpenSUSE 13.2自带了ruby2.1，无需额外安装  

```
# ruby --version
ruby 2.1.3p242 (2014-09-19 revision 47630) [x86_64-linux-gnu]
```

## 安装bundler

1] 将gem源替换成国内的taobao镜像以提升速度

```
# gem source -r https://ruby.taobao.org
https://ruby.taobao.org removed from sources
# gem sources -a https://ruby.taobao.org
https://ruby.taobao.org added to sources
# gem sources
*** CURRENT SOURCES ***

https://ruby.taobao.org
```

2] 安装bundle  

```
# gem install bundler
Fetching: bundler-1.10.6.gem (100%)
Successfully installed bundler-1.10.6
Parsing documentation for bundler-1.10.6
Installing ri documentation for bundler-1.10.6
Done installing documentation for bundler after 6 seconds
1 gem installed
```

## 使用bundler安装github-pages  

1] 在根目录下创建`Gemfile`文件，并写入以下内容  

```
source 'https://ruby.taobao.org'
gem 'github-pages'
```

2] 对于全新的OpenSUSE 13.2需要额外安装以下依赖包  

```
# zypper install ruby-devel make zlib-devel patch gcc nodejs
```

3] 启动bundler安装github-pages，如果报错请检查一下必要的依赖包是否都装了  

```
# bundle.ruby2.1 install
```

4] 切换到gitpage根目录即可  

后续如何在jekyll+gitpage上创建站点这里暂不做详细介绍，想要了解的话推荐这篇博文：[一步步在GitHub上创建博客主页](http://www.pchou.info/web-build/2014/07/04/build-github-blog-page-08.html) 