---
layout: post
title:  '使用Splunk生成Linux系统内存用量图'
date:   2015-12-19 15:20 +0800
comments: true
category: splunk
tags: [Splunk]
---

做项目遇到一个问题，Splunk上的内存信息数据源分别记录了free、cached、buffer，用户需要整体的用量，那么怎么用Splunk来动态计算实际内存用量并显示成图表呢？

## 以下是解决方法

Linux的可用内存算法如下：  
`available = free + cached + buffer`   
现在系统日志中分别记录了free, cached, buffer的数值和内存总量，日志格式大致如下：   

```bash
<timestamp> hostname=<host> itemKey=memory.free value=<value>
<timestamp> hostname=<host> itemKey=memory.cached value=<value>
<timestamp> hostname=<host> itemKey=memory.buffers value=<value>
<timestamp> hostname=<host> itemKey=memory.total value=<value>
<timestamp> hostname=<host> itemKey=memory.free value=<value>
...

```

现在需要使用Splunk统计若干个host的内存用量并生成报表  
由于需要做计算的信息分布在不同行上，涉及到了Splunk的分组计算，最后的搜索语句如下：   

```bash
index=xxx
{
    hostname=host1 OR
    hostname=host2 OR
    ...
}
itemKey=memory.* |

# 第一步使用BUCKET进行强制分片，计算每分钟的平均值，避免timestamp不同步的问题
# 比如下面这个数据：
#     11:30 itemKey=memory.cached value = 1
#     11:30 itemKey=memory.free value = 1
#     11:31 itemKey=memory.buffers value = 1
#     11:31 itemKey=memory.total value = 5
# 如果不进行显式分片的话默认按照时间戳分片，就会把同一组的值分到两组中去，导致结果不正确。

BUCKET _time span=1m | 
STATS AVG(value) AS value BY _time hostname itemKey | 

# 由于stats不能做除法，这里需要把memory.total和其他值分开放在不同的field中，做法如下：
# - 使用IF语句区分value的含义并存到指定的field中，如果不是指定的类型就设为0
# - 使用STATS语句将相同的field求和，这样就能将不同行的数据处理到同一行中
# - 最后使用EVAL进行同行数据计算

EVAL memfree = IF(
    itemKey == "memory.cached" OR
    itemKey == "memory.free" OR
    itemKey == "memory.buffers"
    , value, 0
) | 
EVAL memtotal = IF(
    itemKey == "memory.total"
    , value, 0
) | 
STATS
    SUM(memfree) AS memfree
    SUM(memtotal) AS memtotal
BY hostname, _time | 
EVAL memuseage = (memtotal - memfree) / memtotal * 100 | 
TIMECHART minspan=1m AVG(memuseage) BY hostname
```
