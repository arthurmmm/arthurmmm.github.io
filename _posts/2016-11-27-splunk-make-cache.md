---
layout: post
title:  '使用lookup优化Splunk Dashboard'
date:   2016-11-27 16:53 +0800
comments: true
category: splunk
tags: [Splunk]
---

在制作Splunk Dashboard时经常会使用多种视图来表现同一组数据，在这种情况下如果不做任何优化直接使用搜索语句的话，会导致同一段数据被反复搜索多次，性能下降的同时也会大量浪费系统计算资源。   

这里介绍一下如何通过inputlookup/outputlookup实现一个自带缓存机制的Dashboard.

## inputlookup / outputlookup

这两条指令是我们实现缓存的关键。

简单来讲，我们可以通过`$search_query$ | outputlookup myoutput.csv`来讲结果保存到lookup table

之后，我们就可以使用`| inputlookup myoutput.csv`来使用这个结果

另外Splunk中还有其他几种实现缓存的方法：
   
* scheduled report结合savedsearch
* 将结果保存到summary_index，再从summary_index中搜索

官方文档： http://docs.splunk.com/Documentation/Splunk/6.5.1/SearchReference/Inputlookup

## 实现Dashboard中的缓存Loading界面

先上splunk simple xml的代码，option部分都是默认值被我省略掉了：

```xml
<row rejects="$caching$">
    <panel>
        <single>
            <search>
                <query>$datasource$ | outputlookup myoutput.csv | eval msg="Loading..." | table msg </query>
                <earliest>-30d@h</earliest>
                <latest>now</latest>
                <finalized>
                    <condition>
                        <set token="caching">| inputlookup myoutput.csv</set>
                    </condition>
                </finalized>
            </search>
        </single>
        .... 
    </panel>
</row>
```

这段代码的用处是：   

* 首先，实现一个panel将需要的数据全部收集到myoutput.csv中
* 我们不需要显示搜索结果，只需要一个`Loading...`标签来提示用户，所以在outputlookup后接了一个eval和table来覆盖结果
* 在搜索完成后，设定一个名为caching的token，其结果为`| inputlookup myoutput.csv`，这条语句如果被执行的话就会导出lookup table中的数据
* 注意到我们把这个panel设置为`rejects="$caching$"`，所以一旦caching token不为空时，这个panel就会被隐藏

这样，我们就实现了一个执行时显示loading，执行完毕后隐藏的caching panel

## 在其他table中使用caching的值

很简单，直接使用caching token即可，大致如下：

```xml
<row depends="$caching$">
    <panel>
        <table>
            <search>
                <query>$caching$ | search XXX=XXX | wher XXX = XXX | timechart XXXX ....
            </search>
            ....
        </table>
    </panel>
</row>
```

搜索$caching$可以直接从lookup table获得数据源，避免了同一份数据重复搜索   

设置`depends="$caching$"`可以让这些panel只在缓存完成后显示，避免误导

相关文档：   

http://docs.splunk.com/Documentation/Splunk/6.5.1/Viz/PanelreferenceforSimplifiedXML      

http://docs.splunk.com/Documentation/Splunk/6.5.1/Viz/tokens

## 局限性

在完成缓存后所有的操作都是在lookup table的数据上执行的，因此如果源数据发生改变的话，需要reload整个dashboard，重新更新缓存。   

所以这个方案适合数据源更新不频繁，又需要对同一份数据进行多次操作的dashboard。