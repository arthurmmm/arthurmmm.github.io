---
layout: post
title:  'CentOS安装Redis数据库'
date:   2016-04-23 15:24 +0800
comments: true
category: OS
tags: [OS]
---

Redis安装笔记

## 安装依赖包

```bash
yum install gcc
yum install tcl
```

## 下载安装包

```bash
wget http://download.redis.io/releases/redis-stable.tar.gz
cd redis-stable
make MALLOC=libc
make test
```

## 复制命令行和配置文件

```bash
ln -s ~/redis-stable/src/redis-server /usr/local/bin/redis-server
ln -s ~/redis-stable/src/redis-cli /usr/local/bin/redis-cli
mkdir /etc/redis
cp ~/redis-stable/redis.conf /etc/redis/redis.conf
```

## 操作

```bash
## 启动
redis-server &
## 连接
redis-cli
## 关闭
redis-cli
shutdown
```

## python 包

```bash
## 安装
python -m pip install redis
```

```python

import redis

r = redis.Redis(host='192.168.111.211', port=6379, db=1)
print(r.info())

## string
r['name'] = 'Arthur'
print(r['name'])

## list
r.lpush('listtest', 'aaa')
r.lpush('listtest', 'bbb')
r.lrange('listtest', start=0, end=-1)
r.lindex('listtest', 0)

## set
r.sadd('settest', 'aa')
r.sadd('settest', 'aa')
r.sadd('settest', 'bb')
r.sadd('settest1', 'bb')
r.smembers('settest')
r.sinter('settest','settest1') ## 交集
r.sunion('s1','s2') ## 并集
r.sdiff('settest','settest1') ## 不同

## hash
r.hset('hashtest', 'name', 'Arthur')
r.hset('hashtest', 'age', '25')
r.hgetall('hashtest')
r.hget('hashtest', 'name')
```