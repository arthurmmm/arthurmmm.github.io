---
layout: post
title:  'NO SQL in your code! SQLAlchemy学习'
date:   2015-10-16 21:54 +0800
comments: true
category: python
tags: [SQLAlchemy, SQLite, Python]
---

SQLAlchemy是一种python ORM工具，简单记录一下它的安装使用方法。  

**Links：**  
[SQLAchemy官网](http://www.sqlalchemy.org/)  
[SQLAchemy1.0帮助文档](http://docs.sqlalchemy.org/en/rel_1_0/index.html)

## 安装SQLAlchemy

推荐使用pip安装：   

```
确认pip版本
# pip -V
pip 1.5.6 from /usr/lib/python3.4/site-packages (python 3.4)
这里发现系统自带的pip是for python3的，我们需要的是2.7的版本

安装pip for python 2.x
# zypper install python-pip
Loading repository data...
Reading installed packages...
Resolving package dependencies...

The following NEW package is going to be installed:
  python-pip

1 new package to install.
Overall download size: 1.0 MiB. Already cached: 0 B  After the operation, additional 4.7 MiB will be used.
Continue? [y/n/? shows all options] (y):
Retrieving package python-pip-1.5-2.1.4.noarch                                                                                          (1/1),   1.0 MiB (  4.7 MiB unpacked)
Retrieving: python-pip-1.5-2.1.4.noarch.rpm .............................................................................................................[done (266.5 KiB/s)]
Checking for file conflicts: ..........................................................................................................................................[done]
(1/1) Installing: python-pip-1.5-2.1.4 ................................................................................................................................[done]

使用python2.x调用pip，确认版本
# python -m pip -V
pip 1.5 from /usr/lib/python2.7/site-packages (python 2.7)

安装SQLAlchemy
# python -m pip install SQLAlchemy
```

## 连接数据库

```python
from sqlalchemy import *
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import *

DB = '/mnt/NETSTORAGE/scripts/Test/VSC.db'
ENGINE = create_engine('sqlite:///%s' % DB, echo = True)
BASE = declarative_base()
SESSION = sessionmaker(bind=ENGINE)()
```

DB定义了数据库的位置，ENGINE用来连接数据库，这里`echo = True`用来显示调试信息  
BASE是一个ORM基本类，之后的ORM表类需要继承这个类  
SESSION是用来进行查询的指针，类似Sqlite3包中的cursor  

## 建立表映射 

对于一个并不复杂的数据库，它所包含的表的映射关系完全可以通过类的结构表达清楚。  
以下是一个简单的例子：

```python
# ORM CLASSES
class Person(BASE):
    __tablename__ = 'Person'

    pid = Column('id', Integer, primary_key = True)
    name = Column(String)

    login = relationship('Login', viewonly = True)

class Account(BASE):
    __tablename__ = 'Account'

    aid = Column('id', Integer, primary_key = True)
    account = Column(String)
    atype = Column('type', String)

    login = relationship('Login', viewonly = True)

class PersonAccount(BASE):
    __tablename__ = 'PersonAccount'

    pid = Column(Integer, primary_key = True)
    aid = Column(Integer, primary_key = True)

    ForeignKeyConstraint([pid], [Person.pid])
    ForeignKeyConstraint([aid], [Account.aid])

    account = relationship('Account')
    person = relationship('Person')
    login = relationship('Login')


class Login(BASE):
    __tablename__ = 'Login'

    pid = Column('persionId', Integer, primary_key = True)
    aid = Column('accountId', Integer, primary_key = True)
    date = Column(Date, primary_key = True)

    ForeignKeyConstraint([pid], [Person.pid])
    ForeignKeyConstraint([aid], [Account.aid])
    ForeignKeyConstraint([pid, aid], [PersonAccount.pid, PersonAccount.aid])

    pa_map = relationship('PersonAccount')

```  

- 属性`__tablename__`映射数据库中的表名。
- 凡是映射到`Column(Integer, primary_key = True)`的类成员，都对应数据库中的一列。
- `ForeignKeyConstraint`定义了外键关系，这是之后建立表之间`relationship`的关键。
- `relationship`定义了表之间的关系，很多人会用`backref`的参数提供反向查找，但我更倾向于在两个类上都使用`relationship`，这样表逻辑更清楚。

如果不做插入的话，类的属性不用覆盖所有表的列，`primary_key`之类的设置也不用和数据库一致。  
对于多对多映射，创建relationship太麻烦，建议还是用到的时候用`join`查询吧。  

## 查询

一些查询实例：

```python
print "Print Arthur's name"
arthur = SESSION.query(Person).filter(Person.name == 'Arthur').one()
print arthur.name

print "Print Arthur's login history"
for l in arthur.login:
    print l.date
    print "> 2015-10-14:%s" % ( l.date > datetime.date(2015,10,14))

print "Arthur's account"
record = SESSION.query(Account.account)\
        .join(PersonAccount).join(Person)\
        .filter(Person.name == 'Arthur')\
        .all()
print record

print "> 2015-10-14 Login History for arthur"
record = SESSION.query(Login.date, Person.name, Account.account)\
        .join(Person).join(Account)\
        .filter(Login.date > datetime.date(2015,10,14))\
        .filter(Person.name == 'Arthur')\
        .all()

for d,n,a in record:
    print "%s use %s login when %s" % (n,a,d)
```