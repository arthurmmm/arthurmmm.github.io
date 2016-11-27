---
layout: post
title:  '使用dockerfile制作一个简单镜像'
date:   2016-09-17 21:43 +0800
comments: true
category: OS
tags: [Python, Docker]
---

制作dokcer镜像的方法有两种：创建dockerfile或者commit一个容器，commit方法的优点是比较简单，在一个已经定义好的容器上执行一系列的安装配置后执行commit就能把它作为镜像保存下来了，对应的指令为：`docker commit $container_id$ image_name`。但commit方法的缺点是不够直观，你无法知道一个commit过后的镜像究竟是通过哪些指令生成的。

相对而言dockerfile虽然比较麻烦，但可以很清楚的知道镜像的定义和内容，以后再做修改扩展也比较方便。

## Dockerfile里的常见

```
# 声明基镜像
FROM 

# 运行指令，该指令只在执行dockerfile的时候会被运行，用于安装软件           
RUN 

# 切换工作目录
WORKDIR

# 声明创建者，用于项目管理，对容器内容没什么实际影响
MAINTAINER $author$         

# 容器启动后执行的命令，有多个CMD时只有最后一个生效，可以被命令行参数覆盖
CMD                         

# 容器入口，和CMD比较容易混淆，区别是ENTRYPOINT无法被命令行参数覆写
ENTRYPOINT                  

# 定义执行容器的userid
USER                        

# 容器内部开放的端口，实际启动容器时需要通过命令行和主机物理端口做映射
EXPOSE                      

# 设置环境变量
ENV

# 将外部文件拷入容器内部，注意这里用的是相对路径，如果是gz之类的压缩文件会自动解压
ADD

# 挂载本地文件夹，这个文件夹可以在不同容器之间共享
# 注意如果要在本地管理文件需要在docker run的时候使用-v参数
VOLUME
```

## 一个简单的python环境镜像

Docker File: 

```
# Python 3 Development

FROM centos

MAINTAINER arthur_mao@live.com

# YUM install apps
RUN yum install -y epel-release
RUN yum install -y wget
RUN yum install -y git
RUN yum install -y vim
RUN yum install -y which

# Source Code install Python3 and PIP
RUN yum install -y gcc sqlite-devel openssl openssl-devel
ADD Python-3.4.3.tgz /tmp/
WORKDIR /tmp/Python-3.4.3/
RUN ./configure --with-ensurepip=install
RUN make
RUN make install
RUN ln -s /opt/Python/bin/python3 /usr/bin/python3

WORKDIR /root

# PIP install packages
ADD .pip .pip
RUN python3 -m pip install --upgrade pip -i https://pypi.tuna.tsinghua.edu.cn/simple
ADD pip-install /tmp/pip-install
RUN python3 -m pip install -r /tmp/pip-install -i https://pypi.tuna.tsinghua.edu.cn/simple

# Preference setting
ADD .vim .vim
ADD .vimrc .vimrc
ADD .bashrc .bashrc

CMD /bin/bash
```

这里python用的是tar包安装，先用ADD指令复制到容器内并解压，再切换到相应目录进行编译安装。

另外dockerfile运行的时候没有shell环境，在往用户目录安装路径的时候记得不要用`~/xxx`这种路径或者使用其他shell命令

最后使用`docker build -t python_centos .`来创建镜像

## 一个简单的nginx镜像

Docker官方有提供一个nginx的基镜像，可以直接使用:

```
docker pull nginx
```

为了方便配置，我们希望nginx容器的配置文件和日志可以在本地被浏览访问：

```
# 创建一个临时容器
docker run -d --name tmp.nginx nginx

# 将原生的配置文件考到本地，方便修改
docker cp tmp.nginx:/etc/conf/nginx.conf ./nginx.conf

# 运行容器，记得将本地的conf文件挂载到容器内，并做好端口映射
docker run -d -v /home/arthur/my.nginx/nginx.conf:/etc/nginx/nginx.conf:ro -v /home/arthur/my.nginx/log:/var/log/nginx/:rw -p 80:80 -p 443:443 --name my.nginx nginx
```

有没有觉得这个run指令太长了？docker-compose工具可以用yaml文件来统一管理这些容器，下一篇讲一下docker-compose。