---
layout: post
title:  'OpenSUSE搭建apache环境'
date:   2015-10-11 11:30 +0800
comments: true
category: OS
tags: [Linux, OpenSUSE, apache]
---

OpenSUSE上搭建apache服务器的步骤

##安装apache

1] 下载apache包  

```
# zypper install apache2
```

2] 打开apache服务  

```
# systemctl enable apache2.service
# systemctl start apache2.service
```  

或者  

```
# rcapache2 start
```

3] 在浏览器中打开`http://localhost/`测试，出现403 ERROR就说明apache已经成功启动  


##更改站点目录  

默认的站点目录在LINUX文件系统中，为了方便使用WINDOWS编辑文件，这里需要将它移到SAMBA共享目录上。  

### 默认目录：  
- 站点位置`/srv/www/htdocs/`  
- 配置文件：`/etc/apache2/`  

1] 将原先的站点文件复制到新位置  

```
# cp -a /srv/www /mnt/NETSTORAGE/scripts/ApacheSite/
```

2] 创建`changedir.sh`脚本用于批量修改  

```
# vim changedir.sh
```

3] 脚本如下，复制到`changedir.sh`中，保存并更改为可执行权限`chmod +x changedir.sh`  

```bash
#!/bin/bash

usage(){
    echo "Usage: changedir [source] [target]"
}

SOURCE=$1
TARGET=$2

SOURCE=${SOURCE//\//\\\/}
TARGET=${TARGET//\//\\\/}

echo $SOURCE
echo $TARGET

if [ $# -le 1 ]; then
    usage
    exit 1
fi

grep -r /srv * | awk -F ':' '{print $1}' | uniq | egrep -v "changedir|.new|.bak" | xargs -i cp {} {}.bak
grep -r /srv * | awk -F ':' '{print $1}' | uniq | egrep -v "changedir|.new|.bak" |  xargs -i sh -c "sed 's/$SOURCE/$TARGET/g' {} > {}.new"
find . -name "*.new" | sed "s/.new//g" | xargs -i mv {}.new {}
```  

4] 执行脚本  

```
# ./changedir.sh /srv/www /mnt/NETSTORAGE/scripts/ApacheSite
```