---
layout: post
title:  '给自己的爬虫做一个简单的动态代理池'
date:   2017-02-14 22:28 +0800
comments: true
category: python
tags: [Python, Redis, Scrapy]
---

使用代理服务器一直是爬虫防BAN最有效的手段，但网上的免费代理往往质量很低，大部分代理完全不能使用，剩下能用的代理很多也只有几分钟的寿命，没法直接用到爬虫项目中。   
下面简单记录一下我用scrapy+redis实现动态代理池的过程。

# 我对“动态代理池” 的需求

我的爬虫项目需要7*24小时监控若干个页面，考虑了一下希望代理池能满足下面几个要求：   

*  始终保持一个相对稳定的代理数量
*  始终保持池内代理的高可靠率（希望90%的代理都能用）
*  尽可能减少对爬虫项目代码的更改

# 参考过的项目

找过一些现成的轮子，但发现或多或少都不太符合我的需求   

[kohn](https://github.com/kohn)/**[HttpProxyMiddleware](https://github.com/kohn/HttpProxyMiddleware)**   
一个“被动”选择代理的方式。在scrapy middleware中加入大量代码，让爬虫在代理失效后再去代理网站找代理，会有下面几个问题：   

*  大多数情况下始终使用一个代理去访问，造成流量集中在一个IP上，可能导致代理IP被BAN。
*  切换代理的过程比较耗时，导致爬虫性能下降比较厉害
*  所有逻辑都嵌套在了爬虫项目里面，有兼容性和可移植性方面的问题（有多个爬虫的话每个都要改一遍..）
*  基于python2写的，我的项目在python3上..

[jhao104](https://github.com/jhao104)/**[proxy_pool](https://github.com/jhao104/proxy_pool)**   
这个几乎和我想要的一样，但还是因为下面几点原因没有使用：   

*  基于SSDB，这边没有现成的SSDB，因为已经在用Redis了不想专门去装一个
*  只管定时收集代理，在收集过程中做一次验证。但事实上收集到的代理即便通过了验证，也可能活不过10分钟，时间一长代理池内就会有很多过期的代理，需要在爬虫代码中处理这些过期代理（爬虫代码调用delete删除）。

# 设计思路和代码

看了那么多方案，我就在想：能不能让代理池具有自我检查自我修复的功能？这样我们的爬虫就只需随机拿一个代理用就可以了，就算恰巧拿到一个失效代理，只要做一次retry，代理池的修复机制可以保证retry的时候失效的代理已经被移走了，不会再次取到。   

简单规划一下：   

*  我们需要一个自检程序，它每10S跑一次，保证代理池内所有代理至少在10S内有效
*  我们需要一个代理获取的程序，当代理池内代理数量过低时（阈值定为<5），访问免费代理网站补充新代理
*  我们需要一个调度程序，用来监控代理池数量并调用上面两个程序，维持代理池平衡。
*  作为私有代理池就维护在redis了(SET)，让爬虫直接从redis里取代理。

对应的代码就有这么三个部分：   

*  一个scrapy爬虫去爬代理网站，获取免费代理，验证后入库   (proxy_fetch)
*  一个scrapy爬虫把代理池内的代理全部验证一遍，若验证失败就从代理池内删除   (proxy_check)
*  一个调度程序用于管理上面两个爬虫   (start.py)

强行画个流程图：

![hq-proxies.png](http://upload-images.jianshu.io/upload_images/4610828-edbea71e6ff36157.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

爬虫部分全部用scrapy写了，其实没必要，用requests就够了..但做的时候刚学scrapy，就顺便练练手了..   
start.py的调度方式比较粗暴，直接起两个线程，在线程内用os.system调用scrapy爬虫（毕竟轻量级代理池嘛..好吧其实就是我懒图个方便..）      

另外还有几个调度策略需要说一下：    

*  每次调用proxy_fetch后会自动设定一个“保护时间”10分钟，在保护期内除非代理池只剩一个代理了，否则不会触发proxy_fetch，避免频繁调用。
*  加入了一个“刷新时间”24小时，保证每24小时内至少执行proxy_fetch一次
*  验证程序设定timeout为5秒，5秒内没访问到测试页面就认为验证失败

其他细节可以看源码：[arthurmmm](https://github.com/arthurmmm)/**[hq-proxies](https://github.com/arthurmmm/hq-proxies)**

# 部署和使用
需要先改一下配置文件hq-proxies.yml，把Redis的地址密码之类的填上，改完后放到/etc/hq-proxies.yml下。   
在配置文件中也可以调整相应的阈值和免费代理源和测试页面。   
测试页面需要频繁访问，为了节省流量我在某云存储上丢了个helloworld的文本当测试页面了，云存储有流量限制建议大家换掉。。验证方式很粗暴，比较一下网页开头字符串。。   

另外写了个Dockerfile可以直接部署到Docker上（python3用的是Daocloud的镜像），跑容器的时候记得把hq-proxies.yml映射到容器/etc/hq-proxies.yml下。   
手工部署的话跑`pip install -r requirements.txt`安装依赖包   

在scrapy中使用代理池的只需要添加一个middleware，每次爬取时从redis SET里用srandmember随机获取一个代理就行了，代理失效和一般的请求超时一样retry就行了，代理池的自检特性保证了我们retry时候再次拿到失效代理的概率很低。

最后放一张萌萌哒日志菌触发proxy_fetch时候的截图：   

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/4610828-29e8d33a438a606f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
