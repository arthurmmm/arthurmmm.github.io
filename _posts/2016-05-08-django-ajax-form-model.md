---
layout: post
title:  'Django框架下表单的开发模型'
date:   2016-05-08 16:27 +0800
comments: true
category: web-build
tags: [Python, Django, Bootstrap]
---

介绍使用Django Form + Bootstrap实现表单管理的思路和方式

## 创建Django的ModelForm模块

Django的Form类可以在后台生成和管理表单，而其中的ModelForm类更是可以方便的将数据库模型映射成一个表单，实现快速开发。   
文档：https://docs.djangoproject.com/en/1.9/topics/forms/modelforms/   

简单来讲流程如下   

```python
## 1. Define a ModelForm class
class MyForm(ModelForm):
    ## 2. Add a subclass "Meta"
    class Meta:
        model = MyModel ## 3. Define the Model you use
        fields = '__all__' ## 4. The fields you want to show

```

### 让Django生成Bootstrap风格的表单

Django自动生成的表单并不能很好的契合前端Bootstrap的风格，因为Bootstrap的表单需要将每个域囊括在'form-control'的DIV中。   
解决方法如下：   

```python
class MyForm(ModelForm):
    def __init__(self, *args, **kwargs):
        super(MyForm, self).__init__(*args, **kwargs)
        for field in iter(self.field):
            self.fields[field].widget.attrs.update({
                'class': 'form-control',
            })
```

### 重定义/初始化表单Widget

默认情况下Django会根据Model的属性自动生成Widget，不过有时候我们也会需要自定义部分Widget，比如添加提示或者设置成只读。   
我们可以在Meta中定义labels/widgets来实现这个功能：   

```python
class MyForm(ModelForm):
    def __init__(self, *args, **kwargs):
        super(MyForm, self).__init__(*args, **kwargs)
        for field in iter(self.field):
            self.fields[field].widget.attrs.update({
                'class': 'form-control',
            })
            
    class Meta:
        model = MyModel
        fields = '__all__'
        labels = {
            'user_id': _('User'), ## Rename user_id to User
        }
        widgets = {
            'user_id': TextInput(attrs={'readonly': True}), ## Set user_id as readonly
        }
```

### 表单检查

Django的ModelForm默认会根据Model本身的限制进行检查（例如主键唯一，类型检查等）   
我们也可以提供一些自定义检查规则：   

```python
class MyForm(ModelForm):
    def __init__(self, *args, **kwargs):
        super(MyForm, self).__init__(*args, **kwargs)
        for field in iter(self.field):
            self.fields[field].widget.attrs.update({
                'class': 'form-control',
            })
    
        ## Use Bootstrap alert to show error message
        self.error_css_class = 'alert alert-danger'
        self.required_css_class = 'alert alert-danger'
    
    def clean_name(self):
        v = self.cleaned_data['name']
        RegexValidattor(
            regex = '^[a-z0-9_]+$', ## Only allow lower case chars, ints and underline
            message = _('Only allow lower case chars, ints and underline'),
        )(v)
        return v
        
    class Meta:
        model = MyModel
        fields = '__all__'
        labels = {
            'user_id': _('User'), ## Rename user_id to User
        }
        widgets = {
            'user_id': TextInput(attrs={'readonly': True}),
        }
```

## 创建前端Template

后台类编写完毕后开始为前端页面创建模板，这里使用了Bootstrap的'form-horizontal'模板

```html
<form class="form-horizontal">
    <!-- Load general error -->
    { % if form.errors % }
        { % for error in form.non_field_errors % }
            <div class="alert alert-danger"> { { error|escape } }</div>
        { % endfor % }
    { % endif % }

    { % for field in form % }
        { % if field.errors % }
            <!-- Load fields error -->
            <div class="form-group has-error">
                <label class="col-xs-3 control-label" for="{ { field.name } }">{ { field.label } }</label>
                <div class="col-xs-9">{ { field } }</div>
                <span class="help-block col-xs-12">{ { field.errors.as_text } }</span>
            </div>
        { % else % }
            <!-- Load fields only -->
            <div class="form-group">
                <label class="col-xs-3 control-label" for="{ { field.name } }">{ { field.label } }</label>
                <div class="col-xs-9">{ { field } }</div>
            </div>
        { % endif % }
    { % endfor % }
    <div class="form-group">
        <div class="col-xs-offset-9 col-xs-3">
            <input type="submit" class="btn btn-primary col-xs-12" value="Submit">
        </div>
    </div>
</form>
```

## 为表单创建views

有了ModelForm类和HTML模板，下面的步骤就是创建一个接口view用于将数据加载至模板中。   
在这里我们可以创建一个通用view class用于加载所有的ModelForm   

```python
class FormView(View):
    
    ## Exclude csrf check for POST
    @method_decorator(csrf_exempt)
    def dispatch(self, request, *args, **kwargs):
        return super(FormView, self).dispatch(request, *args, **kwargs)
    
    ## Get a form for new object, or get a form to edit existing object
    ## Using 'is_new' param to identify if we want a new form.
    def get(self, request, form_class, template):
        template_file  = 'myapp/form/%s.html' % template ## a template file for form
        request_dict = request.GET.dict()
        is_new = request_dict.pop('is_new')
        if is_new == 'true':
            ## Generating a new form
            form = eval(form_class)(initial=request.GET.dict())
            id = ''
        else:
            ## Getting a model object, then load it to form for editing
            model = eval(form_class)._meta.model
            inst = model.object.get(**request_dict)
            form = eval(form_class)(instance=inst)
            id = inst.id
        context = {
            'form': form,
            'id': id,
            'form_class': form_class,
            'template': template,
        }
        return render(request, template_file, context)
    
    ## Edit existing form, steps:
    ## Get info submitted by form => get model by info => load model into form => save form
    def put(self, request, form_class, template):
        template_file = 'myapp/form/%s.html' % template
        request_querydict = QueryDict(request.body, mutable=True)
        id = request_querydict.pop('id')[0]
        model = eval(form_class)._meta.model
        inst = model.objects.get(id=id)
        form = eval(form_class)(request_querydict, instance=inst)
        if form.is_valid():
            form.save()
            return HttpResponse(status=200)
        else:
            context = {
                'form': form,
                'id': id,
                'form_class': form_class,
                'template': template,
            }
            return render(request, template_file, context, status=400)
    
    ## Create a new model instance by submitted form 
    def post(self, request, form_class, template):
        template_file = 'myapp/form/%s.html' % template
        form = eval(form_class)(request.POST)
        if form.is_valid():
            form.save()
            return HttpResponse(status=200)
        else:
            context = {
                'form': form,
                'id': '',
                'form_class': form_class,
                'template': template,
            }
            return render(request, template_file, context, status=400)
```

配置urls.py：

```python
urlpatterns = [
    url(r'_form/(?P<form_class>.+)/(?P<template>.+)/$'),
]
```

有了这个view我们的思路就很清楚了：   
1. 创建一个model：使用GET获取一个空表单(is_new='true') => 使用POST提交表单   
2. 编辑一个model：使用GET获取一个加载了现有数据的表单(is_new='false') => 使用PUT提交表单   

