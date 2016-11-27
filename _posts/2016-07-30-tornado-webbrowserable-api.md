---
layout: post
title:  '使用Tornado实现RESTFUL Style API(3) -- 可视化API'
date:   2016-07-30 21:53 +0800
comments: true
category: python
tags: [Python, Tornado]
---

给API系统提供一个图形界面，可以在网页上完成基本的操作需求和测试需求。

## 辨别客户端请求

我们通过判断HTTP请求HEADER中的'ACCEPT'域，来决定返回的输出格式：

- ACCEPT中包含text/html： 返回GUI界面
- OTHER：返回JSON数据

严格应该要求客户端设置ACCEPT: Application/JSON的HEADER才会返回JSON数据，不过目前没有其他格式的需求，这里就简单的将其他类型的请求都视作JSON请求了。   

另外我们还引入了一个特殊参数__format__用来显式指定输出格式，用于处理无法设置HEADER的情况以及方便测试。

## 渲染HTML Template

通常Tornado渲染HTML的方式是用request.render函数来实现。但是在我们的API中，统一使用了request.finish方法来返回结果，因此需要先使用request.render_string函数将HTML模板转化为字符串，再返回给用户。

这个处理过程放在了__format__的Transform代码中

```python

def trainsform__format__(request, result, format):
   if format == 'json':
      return result
   elif format == 'html':
      html_dump = request.render_string('api_gui.html', result=result, json_dumps=json.dumps)
      return html_dump
   else:
      return result

```

注意代码中 ```json_dumps=json.dumps``` ，这句话表明我们将函数json.dumps作为参数传递给HTML Template了，之后我们可以在HTML中使用json_dumps(...)来调用

## 图形界面设计

- 整体上使用了twitter-bootstrap框架设计UI
- 为了提供更好的可读性以及方便用户输入数据，在图形界面上我们使用了YAML格式来替换JSON格式，使用了JS包：jsyaml
- 将输出结果中的URL转换成超链接，使用了JS包：Autolinker.js
- 高亮YAML输出结果，使用了JS包：highlight.min.js
- 根据API meta中的内容显示对应的GET/POST/PUT/DELETE按钮
- 用户点击按钮后，触发AJAX事件，调用对应API并将结果重载到输出框中

```javascript

function loadData(raw_data) {
    text_data = jsyaml.dump(raw_data);
    ...
}

$('#submit button').on('click', function(){
    ...
    $.ajax({
        type: method,
        url: url,
        data: data,
        contentType: 'application/json',
        dataType: 'json',
        complete: function(xhr) {
            loadData(xhr.responseJSON);
            ...
        }
    })
})

```