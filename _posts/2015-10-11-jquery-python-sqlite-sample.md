---
layout: post
title:  '创建JQuery PythonCGI SQLite示例'
date:   2015-10-11 17:40 +0800
comments: true
category: web-build
tags: [JQuery, Python, SQLite, CGI]
---

学习过程中制作的一个简单的JQuery+PythonCGI+SQLite站点

##SQLite

创建数据库test.db和表employee  

```
# sqlite3 test.db
SQLite version 3.8.6 2014-08-15 11:46:33
Enter ".help" for usage hints.

sqlite> .database
seq  name             file
---  ---------------  ----------------------------------------------------------
0    main             /mnt/NETSTORAGE/scripts/sqlite_db/test.db

sqlite> CREATE TABLE employee(id INTEGER PRIMARY KEY, name TEXT, age INTEGER);

sqlite> .table
employee

sqlite> INSERT INTO employee(id, name, age) VALUES(1, "arthur", 24);
sqlite> INSERT INTO employee(id, name, age) VALUES(2, "vincent", 35);
sqlite> INSERT INTO employee(id,name,age) VALUES(3,"arthur",22);

sqlite> .mode column
sqlite> .header on
sqlite> SELECT * FROM employee;
id          name        age
----------  ----------  ----------
1           arthur      24
2           vincent     35
3           arthur      22
```

##Index.html

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">

<html lang="zh-cn">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<meta http-equiv="Page-Enter" content="blendTrans(Duration=0.5)">
<meta http-equiv="Page-Exit" content="blendTrans(Duration=0.5)">
<title>404 NOT FOUND!</title>

<script src="http://code.jquery.com/jquery-1.4.1.min.js"></script>

<script>
    $(function()
    {
		console.log('调试信息');
		$("#btn").click(function(){
			$.get("/cgi-bin/test.py",{name: $("#emp_name").val()},function(data){
				if (data.length == 0) {
					alert("查无此人！");
				}
				for (var i = 0; i < data.length; i++) {
					alert("员工ID：" + data[i].id + "\n员工姓名：" + data[i].name + "\n员工年龄：" + data[i].age);
				}			
			}, "json");
		});
    });
</script>

</head>
<body>
<div>
<p>一个简单的查询程序：</p>
<p>输入员工姓名，查询数据库中该员工的详细信息（ID、姓名、年龄）</p>
<p>数据库中共有三名员工：</p>
<code>
	<p>id|name|age</p>
	<p>1|arthur|24</p>
	<p>2|vincent|35</p>
	<p>3|arthur|22</p>
</code>
</div>
<div>
	请输入员工姓名：
	<input id="emp_name" /><button id="btn">OK</button>
</div>
</body></html>
```

##test.py

创建python CGI，放入`cgi-bin`目录下并且设置权限`chmod +x test.py`  

```python
#!/usr/bin/python

import os, sys, cgi, json, sqlite3

# Header
print ""

# Get values from HTTP request
form = cgi.FieldStorage()
name = form.getvalue('name')

# Connect SQLite
db = sqlite3.connect('/mnt/NETSTORAGE/scripts/sqlite_db/test.db')
dbcu = db.cursor()

# Query thru name
query = 'SELECT * FROM employee WHERE name="%s"' % name
# return 
employee = []
results = dbcu.execute(query).fetchall()
for res in results:
	employee.append({\
		"id": res[0],\
		"name": res[1],\
		"age": res[2]})
print json.dumps(employee)
```