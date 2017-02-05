---
layout: post
title:  'python+redis让微信公众号根据上下文回复用户消息'
date:   2017-02-05 23:17 +0800
comments: true
category: python
tags: [Python, Redis]
---

最近打算用微信公众号做一些玩票小项目，研究了半天后发现很多功能对个人订阅号都不开放（需要认证），比如自定义菜单啦，管理图文消息啦，获取用户信息等等很基本的功能..所以只能用被动回复消息，通过一问一答的方式实现用户交互。

然而这就需要记录会话状态，对于相同的信息要根据状态的不同给出不同的回复，比如在刚关注公众号的时候，如果用户发送了一个“苹果”过来，我要返回给他一段帮助消息：

> 用户：苹果
> 公众号：你发苹果给我干嘛呀？这里是一段帮助信息...

但如果前一条消息是一个问题，用户发送“苹果”那我就需要返回对应的回复给用户，比如下面这样：

> 用户：苹果
> 公众号：你发苹果给我干嘛呀？这里是一段帮助信息...
> 用户：那你问我一个问题吧
> 公众号：请发送一个水果名字
> 用户：苹果
> 公众号：给你一个苹果=￣ω￣=

但是呢微信给出的官方例子并没有相关会话处理的部分，也没有找到类似的解决方案。
我觉得我需要一个轮子来解决这个问题，没有现成轮子的话就自己造一个吧

于是就有了下面这个小轮子，用到了python generator和redis。
代码开源在GITHUB上：[arthurmmm/wechat-dialog](https://github.com/arthurmmm/wechat-dialog)

# 部署demo

代码中附带了一个Flask写的小demo，可以直接部署测试。另外代码是在python3下写的。

安装步骤如下：
* 根据requirement.txt安装依赖包（其实就redis和flask..）
* 安装一个Redis，知道它的IP地址，端口号，密码等信息
* 在demo_dialog.py的开头更改相应的REDIS配置信息
* 启动demo_server.py: python ./demo_server.py
* 去微信公众平台绑定公众号服务器

# 使用方法

demo_dialog.py是示例用的会话逻辑程序，需要根据业务要求配置一个类似的文件，具体用法参考源码中的注释，简单来讲就是用yield在一个函数内处理一段对话的所有问答信息，比如示例中的accumulate：

```python
def accumulator(to_user):
    yield None
    msg_content = yield None
    
    num_count = yield ('TextMsg', '您需要累加几个数字？')
    try:
        num_count = int(num_count)
    except Exception:
        return ('TextMsg', '输入不合法！我们需要一个整数，请输入"开始"重新开启累加器')
    res = 0
    for i in range(num_count):
        num = yield ('TextMsg', '请输入第%s个数字, 目前累加和:%s' % (i+1, res))
        try:
            num = int(num)
        except Exception:
            return ('TextMsg', '输入不合法！我们需要一个整数，请输入"开始"重新开启累加器')
        res += num
        
    # 注意：最后一个消息一定要用return不要用yield！return用于标记会话结束。
    return ('TextMsg', '累加结束，累加和: %s' % res)
```

上面这段代码运行起来是这个效果：

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/4610828-e4d47cdc45d03c89.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


配置完dialog逻辑后，将类似下面这段代码加入服务器，下面是flask上的配置方法，也可以用到其他python web框架下：

```python
import wechat.bot
import demo_dialog

@app.route('/', methods=['POST'])
def wechat_post():
    data = request.get_data()
    return wechat.bot.answer(data, demo_dialog).format()
```

answer方法接收data和dialog模块作为参数，用dialog中定义的逻辑处理收到的用户信息data，最后返回一个replyMsg，再调用format格式化后作为回复。（关于公众号的receiveMsg和replyMsg可以参考微信官方文档，我基本是照搬的..）

# 设计思路

主要的轮子代码在wechat/bot.py中。wechat/reply.py和receive.py是根据微信官方文档做的消息类。

在调用answer函数后，bot会先根据用户的open_id检查对应的redis key，如果redis key中没有值或者出现意外状况，那么就认为这是一段新的对话，通过ROUTER中配置的静态映射关系，进入对应的对话处理函数并返回。

如果key中有值，那么bot就认为用户的这条信息是针对这段会话的一个回复消息，会从redis中取出之前的历时消息记录，不断触发yield重现会话上下文，到达正确的断点后返回。

redis key默认设置了60秒的过期时间，用户60秒内不回复就丢弃这段会话。
key中的数据结构是一段json序列化的数组，第一个元素存储了dialog_handler的名字作为入口，后面是一组历时消息用于重现上下文：

```python
[<dialog_handler_name>, <msg1>, <msg2>, <msg3> ....]
```

如果发现生成器return了(即抛出了StopIteration异常)就结束这段对话，回复用户return的消息并清空redis key。

代码量不多，更多的还是看源码吧，欢迎吐槽。
轮子是顺手造的，代码写的比较随意还请见谅。。

------
PS: redis的expire真的是神器..我要成redis脑残粉了