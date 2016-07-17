---
layout: post
title:  'Django下将命令行程序异步图形化'
date:   2016-06-10 19:22 +0800
comments: true
imagedir: /images/2016-06-10/
category: web-build
tags: [Python, Django]
---

项目上需要将一个命令行程序图形化成Django网页应用。   
起初尝试按照一般设计思路，设计一个表单提交参数，然后等待服务器回应，结果发现在程序执行时间过长时会导致网页超时，因此需要重新设计，将整个流程异步化。   

## 设计思路

### Stage 1: 用户输入

首先当用户访问APP时，会被url.py定位到index view，并提供一个表单给用户输入参数。   
用户提交表单后，数据会被提交给submit view   

[![]({{site.baseurl}}{{page.imagedir}}stage_1.png)]({{site.baseurl}}{{page.imagedir}}stage_1.png)

### Stage 2：执行过程

由于程序执行时间过长，我们并不会在submit view中直接处理业务逻辑，而是创建一个线程Batch Thread。   
然后对于每一个线程，我们都会指定一个唯一的ID用于标识，submit view在拿到Batch ID后会根据Batch ID渲染template Track.html，并立即返回给用户。   
Track.html是一个用户追踪任务进程的page，用于显示任务进度给用户。在该页面上会有JS代码通过Ajax Call的方式，根据Batch ID查询进度。   

[![]({{site.baseurl}}{{page.imagedir}}stage_2.png)]({{site.baseurl}}{{page.imagedir}}stage_2.png)

### Stage 3: 结果显示

在所有任务完毕后，track page会重定向到Result View，然后从Batch Thread中获取程序执行的结果，渲染成Result.html并返回给用户，整个程序就算执行完毕了。   

[![]({{site.baseurl}}{{page.imagedir}}stage_3.png)]({{site.baseurl}}{{page.imagedir}}stage_3.png)

## 技术细节

### 表单实现

采用了Django的表单设计模式和Twitter Bootstrap网页框架。   
另外还使用了bootstrap-select插件实现了一个可多选的下拉框。

[![]({{site.baseurl}}{{page.imagedir}}form.png)]({{site.baseurl}}{{page.imagedir}}form.png)

### 任务线程实现

在线程模型中利用getattr方式实现了一个简单的工厂模型：   

- 一次执行被分隔成多个Task
- 建立两个映射用于控制不同Task之间的依赖关系，以及不同Task对应的数据内容
- 对于没有依赖关系的多个Task，用户可以自行选择是否执行。
- 用户的选择之间通过getattr(BatchThread, taskname)映射到对于函数

以下这个函数可以给用户表单提供所有可用Task的列表，供用户选择：   

```python

def allBatches(cls):
    res = []
    all_attr = dir(cls)
    for attr in all_attr:
        regex = re.match('^batch_(.+)', attr)  ## All task method would be named as batch_xxxx
        if callable(getattr(cls, attr)) and regex:
            res.append(regex.group(1))
    return res
    
```

### 网页模拟控制台实时输出

在track.html页面中我们使用了js setInterval的方式来给用户实时显示程序进度：每秒执行一个Ajax请求来获得后台进度，并更新页面。

```javascript

function checkStatus(mon) {
    $.ajax({
        ...
        complete: function(xhr) {
            ....
            window.localtion.replace(...); // Using 'replace' rather than 'href' to redirect page.
            clearInterval(mon); // Able to stop interval inside ajax call.
        }
    });
}

var mon = setInterval(function(){
    checkStatus(mon); // Ajax call function.
    $("html, body").animate({
        scrollTop: $("#end").offset().top
    }, 1000) //Scroll to the bottom of page. #end is defined at the end.
}, 1000); //Call inline function every second

```