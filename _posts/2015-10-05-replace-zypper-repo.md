---
layout: post
title:  'OpenSUSE配置国内源'
date:   2015-10-05 23:31 +0800
comments: true
category: Linux/Unix
---

OpenSUSE的zypper官方源速度奇慢，替换国内镜像源是装机必不可少的过程。
本文介绍如何利用sed/awk快速将zypper源替换成国内镜像。

## 查看zypper帮助文档：

```
$ zypper -h | less
```

## 列出所有安装源：

```
$ zypper -lr
 | Alias                     | Name                               | Enabled | Refresh | Type
--+---------------------------+------------------------------------+---------+---------+------
1 | openSUSE-13.2-0           | openSUSE-13.2-0                    | Yes     | No      | yast2
2 | repo-debug                | openSUSE-13.2-Debug                | No      | Yes     | NONE
3 | repo-debug-update         | openSUSE-13.2-Update-Debug         | No      | Yes     | NONE
4 | repo-debug-update-non-oss | openSUSE-13.2-Update-Debug-Non-Oss | No      | Yes     | NONE
5 | repo-non-oss              | openSUSE-13.2-Non-Oss              | Yes     | Yes     | yast2
6 | repo-oss                  | openSUSE-13.2-Oss                  | Yes     | Yes     | yast2
7 | repo-source               | openSUSE-13.2-Source               | No      | Yes     | NONE
8 | repo-update               | openSUSE-13.2-Update               | Yes     | Yes     | NONE
9 | repo-update-non-oss       | openSUSE-13.2-Update-Non-Oss       | Yes     | Yes     | NONE
```

## 禁用并停止更新自带源：

```
$ zypper mr -d -R --all
```


## 配置浙大源：


1] 进入`/etc/zypp/repos.d`文件夹  

```
$ ll
total 36
-rw-r--r-- 1 root root 191 Oct  5 23:22 openSUSE-13.2-0.repo
-rw-r--r-- 1 root root 188 Oct  5 23:25 repo-debug-update-non-oss.repo
-rw-r--r-- 1 root root 164 Oct  5 23:25 repo-debug-update.repo
-rw-r--r-- 1 root root 165 Oct  5 23:25 repo-debug.repo
-rw-r--r-- 1 root root 168 Oct  5 23:25 repo-non-oss.repo
-rw-r--r-- 1 root root 156 Oct  5 23:25 repo-oss.repo
-rw-r--r-- 1 root root 168 Oct  5 23:25 repo-source.repo
-rw-r--r-- 1 root root 170 Oct  5 23:25 repo-update-non-oss.repo
-rw-r--r-- 1 root root 146 Oct  5 23:25 repo-update.repo
```  


2] 执行以下指令批量创建ZJU源(http://mirrors.zju.edu.cn/opensuse)  

```
$ find . -name "repo*" | sed 's/.\///g' | xargs -i sh -c "cat {} | sed 's/download.opensuse.org/mirrors.zju.edu.cn\/opensuse/g' | sed 's/repo-/zju\/repo-/g' | sed 's/openSUSE-13.2/zju\/openSUSE-13.2/g' > zju-{}"

$ zypper lr
#  | Alias                         | Name                                   | Enabled | Refresh
---+-------------------------------+----------------------------------------+---------+--------
 1 | openSUSE-13.2-0               | openSUSE-13.2-0                        | No      | No
 2 | repo-debug                    | openSUSE-13.2-Debug                    | No      | No
 3 | repo-debug-update             | openSUSE-13.2-Update-Debug             | No      | No
 4 | repo-debug-update-non-oss     | openSUSE-13.2-Update-Debug-Non-Oss     | No      | No
 5 | repo-non-oss                  | openSUSE-13.2-Non-Oss                  | No      | No
 6 | repo-oss                      | openSUSE-13.2-Oss                      | No      | No
 7 | repo-source                   | openSUSE-13.2-Source                   | No      | No
 8 | repo-update                   | openSUSE-13.2-Update                   | No      | No
 9 | repo-update-non-oss           | openSUSE-13.2-Update-Non-Oss           | No      | No
10 | zju/repo-debug                | zju/openSUSE-13.2-Debug                | No      | No
11 | zju/repo-debug-update         | zju/openSUSE-13.2-Update-Debug         | No      | No
12 | zju/repo-debug-update-non-oss | zju/openSUSE-13.2-Update-Debug-Non-Oss | No      | No
13 | zju/repo-non-oss              | zju/openSUSE-13.2-Non-Oss              | No      | No
14 | zju/repo-oss                  | zju/openSUSE-13.2-Oss                  | No      | No
15 | zju/repo-source               | zju/openSUSE-13.2-Source               | No      | No
16 | zju/repo-update               | zju/openSUSE-13.2-Update               | No      | No
17 | zju/repo-update-non-oss       | zju/openSUSE-13.2-Update-Non-Oss       | No      | No
```  

3] 执行以下指令启用zju源并打开自动更新   

```
$ zypper lr | grep zju | awk '{print $3}' | xargs -i zypper mr -e -r {}
Repository 'zju/repo-debug' has been successfully enabled.
Autorefresh has been enabled for repository 'zju/repo-debug'.
Repository 'zju/repo-debug-update' has been successfully enabled.
Autorefresh has been enabled for repository 'zju/repo-debug-update'.
Repository 'zju/repo-debug-update-non-oss' has been successfully enabled.
Autorefresh has been enabled for repository 'zju/repo-debug-update-non-oss'.
Repository 'zju/repo-non-oss' has been successfully enabled.
Autorefresh has been enabled for repository 'zju/repo-non-oss'.
Repository 'zju/repo-oss' has been successfully enabled.
Autorefresh has been enabled for repository 'zju/repo-oss'.
Repository 'zju/repo-source' has been successfully enabled.
Autorefresh has been enabled for repository 'zju/repo-source'.
Repository 'zju/repo-update' has been successfully enabled.
Autorefresh has been enabled for repository 'zju/repo-update'.
Repository 'zju/repo-update-non-oss' has been successfully enabled.
Autorefresh has been enabled for repository 'zju/repo-update-non-oss'.

$ zypper lr
#  | Alias                         | Name                                   | Enabled | Refresh
---+-------------------------------+----------------------------------------+---------+--------
 1 | openSUSE-13.2-0               | openSUSE-13.2-0                        | No      | No
 2 | repo-debug                    | openSUSE-13.2-Debug                    | No      | No
 3 | repo-debug-update             | openSUSE-13.2-Update-Debug             | No      | No
 4 | repo-debug-update-non-oss     | openSUSE-13.2-Update-Debug-Non-Oss     | No      | No
 5 | repo-non-oss                  | openSUSE-13.2-Non-Oss                  | No      | No
 6 | repo-oss                      | openSUSE-13.2-Oss                      | No      | No
 7 | repo-source                   | openSUSE-13.2-Source                   | No      | No
 8 | repo-update                   | openSUSE-13.2-Update                   | No      | No
 9 | repo-update-non-oss           | openSUSE-13.2-Update-Non-Oss           | No      | No
10 | zju/repo-debug                | zju/openSUSE-13.2-Debug                | Yes     | Yes
11 | zju/repo-debug-update         | zju/openSUSE-13.2-Update-Debug         | Yes     | Yes
12 | zju/repo-debug-update-non-oss | zju/openSUSE-13.2-Update-Debug-Non-Oss | Yes     | Yes
13 | zju/repo-non-oss              | zju/openSUSE-13.2-Non-Oss              | Yes     | Yes
14 | zju/repo-oss                  | zju/openSUSE-13.2-Oss                  | Yes     | Yes
15 | zju/repo-source               | zju/openSUSE-13.2-Source               | Yes     | Yes
16 | zju/repo-update               | zju/openSUSE-13.2-Update               | Yes     | Yes
17 | zju/repo-update-non-oss       | zju/openSUSE-13.2-Update-Non-Oss       | Yes     | Yes
```
 
4] 更新源  

```
$ zypper refresh
Retrieving repository 'zju/openSUSE-13.2-Debug' metadata ............................................................................................[done]
Building repository 'zju/openSUSE-13.2-Debug' cache .................................................................................................[done]
Retrieving repository 'zju/openSUSE-13.2-Update-Debug' metadata .....................................................................................[done]
Building repository 'zju/openSUSE-13.2-Update-Debug' cache ..........................................................................................[done]
Retrieving repository 'zju/openSUSE-13.2-Update-Debug-Non-Oss' metadata .............................................................................[done]
Building repository 'zju/openSUSE-13.2-Update-Debug-Non-Oss' cache ..................................................................................[done]
Retrieving repository 'zju/openSUSE-13.2-Non-Oss' metadata ..........................................................................................[done]
Building repository 'zju/openSUSE-13.2-Non-Oss' cache ...............................................................................................[done]
Retrieving repository 'zju/openSUSE-13.2-Oss' metadata ..............................................................................................[done]
Building repository 'zju/openSUSE-13.2-Oss' cache ...................................................................................................[done]
Retrieving repository 'zju/openSUSE-13.2-Source' metadata ...........................................................................................[done]
Building repository 'zju/openSUSE-13.2-Source' cache ................................................................................................[done]
Retrieving repository 'zju/openSUSE-13.2-Update' metadata ...........................................................................................[done]
Building repository 'zju/openSUSE-13.2-Update' cache ................................................................................................[done]
Retrieving repository 'zju/openSUSE-13.2-Update-Non-Oss' metadata ...................................................................................[done]
Building repository 'zju/openSUSE-13.2-Update-Non-Oss' cache ........................................................................................[done]
```

<EOF>
