---
layout: post
title:  '使用Tornado实现RESTFUL Style API(2) -- 使用threadpool并发处理非异步IO'
date:   2016-07-09 17:03 +0800
comments: true
category: python
tags: [Python]
---

Tornado的异步功能很强大，相对的它要求对应的操作本身需要支持异步，但其实并不是所有的IO操作都有对应的异步实现，比如坑爹的Sybase...   
为了不让这些非异步IO阻塞进程，我在参考了网上一些讨论后，设计了如下的方法。

## 使用python ThreadPoolExecutor处理并发 

```python
from concurrent.futures import ThreadPoolExecutor

EXECUTOR = ThreadPoolExecutor(max_workers=10)

def threaded(f):    
    @asynchronous
    @wraps(f)
    def wrapper(*args, **kwargs):
        self = args[0]
        
        def callback(future):
            try:
                result = future.result()
                self.finish(result)
            except Exception as e:
                self.set_status(500)
                self.finish({
                    'error': 'Unexcepted error: %s' % e
                })
            return
        
        EXECUTOR.submit(f, *args, **kwargs).add_done_callback(
            lambda future: IOLoop.instance().add_callback(callback, future)
        )
        
        return wrapper
```

当调用被装饰的函数f时，首先会将它submit到线程池EXECUTOR中执行，EXECUTOR的回调函数直接将future传递给Tornado的IOLoop，之后Tornado就会在下一次IO Loop时调用真正的callback函数。

## 如何在真正的Handler中控制页面操作？

在用上述threaded方法装饰API Handler后，这个Handler就会运行在线程池中，也失去了直接操作Tornado IOLoop的能力，所以我们只能让Handler返回一个字典，再通过回调函数去执行self.finish来将结果输出到Tornado IOLoop中。   

那么如果我们希望执行一些稍微复杂点的页面操作，比如设置一下status code呢？   
显然我们不可能为每个Handler都创建一个对应的回调函数，并且操作的内容也得在Handler中控制，下面是一个可行解决方法。

```python
from concurrent.futures import ThreadPoolExecutor

EXECUTOR = ThreadPoolExecutor(max_workers=10)

class ResultTransformer(object):
    ORDER = [
        '__status_code__',
        '__finish__',
    ]
    
    def trainsform__status_code__(request, result, status_code):
        request.set_status(status_code)
        return result
    
    def transform__finish__(request, result, value):
        request.finish(result)
        return result

def threaded(f):    
    @asynchronous
    @wraps(f)
    def wrapper(*args, **kwargs):
        self = args[0]
        
        def callback(future):
            try:
                result = future.result()
                
                private_items = OrderedDict(sorted(
                    [(k,v) for k,v in result.items() if re.match('^__.*__$', k)],
                    key = lambda t: ResultTransformer.ORDER.index(t[0])
                ))
                result = {k: v for k,v in result.items() if k not in private_items}
                for k,v in private_items.items():
                    fn = 'transform%s' % k
                    if not hasattr(ResultTransformer, fn):
                        raise HTTPError(400, reason='Invalid special param: %s' % fn)
                    f = getattr(ResultTransformer, fn)
                    result = f(self, result, v)

            except Exception as e:
                self.set_status(500)
                self.finish({
                    'error': 'Unexcepted error: %s' % e
                })
        
        EXECUTOR.submit(f, *args, **kwargs).add_done_callback(
            lambda future: IOLoop.instance().add_callback(callback, future)
        )
        
        return wrapper
```

为了处理这些页面操作，我定义了一类API的“特殊参数”和一个工厂类ResultTransformer，然后为每一个可能出现的特殊参数注册一个同名的工厂函数。在工厂类中又定义了一个数组ORDER用来管理工厂函数执行的顺序，所有工厂函数的格式均为result=f(result, value)，即通过value的值来处理result，并返回一个新的result。      

回调函数在处理result结果集时，首先寻找形如__.*__的特殊参数，把它们pop到有序字典private_items中，再遍历一遍private_items中的每个成员，依次执行它们对应的工厂函数并得到一个新的result值，通过这样类似流水线式的作业，我们就能够比较灵活的管理一些特殊的页面操作了。   

在上面的例子中我们定义了两个最基础的Transformer: __status_code__和__finish__，在API的Handler中，我们可以通过设定__status_code__的值来控制页面的返回值，而__finish__函数其实并不需要任何参数，在Handler中设定__finish__参数表示我们希望结果集直接通过request.finish来返回给用户。

