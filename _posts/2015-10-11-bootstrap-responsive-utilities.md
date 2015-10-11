---
layout: post
title:  'Bootstrap学习笔记'
date:   2015-10-11 22:04 +0800
comments: true
category: front-end 
tags: [Bootstrap, CSS]
---

前端接触的不多，在做这个博客的时候基本都是在别人模板的基础上做一些调整。  
无奈现成模板的布局不是很对我的胃口，所以又花了些时间研究Bootstrap的布局，简单的做一些记录  
详细的帮助文档[在这里](http://getbootstrap.com/css/)

## xs < sm < md < lg

这个是bootstrap对屏幕大小的划分，直观的讲：  

- `xs` -> 手机  
- `sm` -> ipad竖屏/ipad-mini
- `md` -> ipad横屏
- `lg` -> 电脑

## col-(xs/sm/md/lg)-(number) 

bootstrap的宽度控制。  
bootstrap把屏幕划分成12格，后面的数字`<number>`表示在对应的屏幕上占用几格。  
如果设置了`col-sm`，那么该设置会对所有大于等于`sm`的屏幕(即`sm、md、lg`)生效。  

```html
Example:

<div class="col-sm-9"></div>
```

## hidden-(xs/sm/md/lg)   

仅在对应的屏幕尺寸隐藏该组件。  
如果需要在`xs`和`sm`同时隐藏，需要设置两次。  

```html
Example:

<div class="col-md-9 hidden-xs hidden-sm"></div>
```

## 响应式导航栏 

响应式导航栏会在`xs`屏幕上会折叠成一个按钮  

```html
Example:

<div class="navbar-header">
  <button type="button" class="navbar-toggle" data-toggle="collapse" data-target="#bs-example-navbar-collapse-1">
    <span class="sr-only">Toggle navigation</span>
    <span class="icon-bar"></span>
    <span class="icon-bar"></span>
    <span class="icon-bar"></span>
  </button>
  <a class="navbar-brand" href="{{ site.baseurl }}/">{{ site.name }}</a>
</div>
<div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
  <ul class="nav navbar-nav">
	  <li class="active"><a href="{{ site.baseurl }}/">Home</a></li>
	  <li class="active visible-xs-block visible-sm-block"><a href="{{ site.baseurl }}/categories.html">Categories</a></li>
	  <li class="active visible-xs-block visible-sm-block"><a href="{{ site.baseurl }}/tags.html">Tags</a></li>
	  <li class="active"><a href="{{ site.baseurl }}/archive.html">Archive</a></li>
	  <li class="active"><a href="{{ site.baseurl }}/feed.xml">RSS</a></li>
	  <li class="active"><a href="{{ site.baseurl }}/about.html">About Me</a></li>
  </ul>
</div>
```  

分两部分：  

### navbar-header 

折叠时的设置，一般包含一个按钮和一个LOGO  

按钮：  

- 设定`class`为`navbar-toggle`
- `data-toggle`设置为`collapse`
- `data-target`指向`narbar-collapse`的`id`
- 内容一般为三组`<span class="icon-bar">`，即按钮显示成三条横线

LOGO：

- 设定`class`为`navbar-brand`的超链接  

### collapse navbar-collapse

展开时的设置，一组无序列表。  

- 设置的`id`需要与`data-target`对应  
- 列表`ul`的`class`设置为`nav navbar-nav`  
