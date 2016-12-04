---
layout: post
title:  '把jekyll blog搬运到我的服务器上'
date:   2016-09-26 20:18 +0800
comments: true
category: OS
tags: [Python, Docker]
---

> 更新:   
> 嗨呀好气啊，因为没有备案还是被墙了，只能老老实实用回gitpage版本了。。。   
> 另外因为博客是浅色系的，用深色系的代码高亮比较难看，就把CSS文件又换回来了。

最近gitpage访问有点不稳定，打算在自己docker+nginx环境上也搭一个博客镜像，记录下过程。

## jekyll 2.4 => jekyll 3.0 (gitpage update)

之前github有做过一次更新，将jekyll升级到3.0版本并强制使用rouge, 更新完并按照要求更改config文件后，gitpage是没啥问题但本地的jekyll服务器出现了代码高亮异常和分页的问题。。  

```yaml
# 按照github要求修改highlighter
highlighter: rouge
markdown: kramdown
```
 
经过研究后解决方法如下：

### 升级jekyll版本

本地的jekyll还停留在2.4版本需要升级：

```
gem update jekyll
```

### 解决rouge高亮问题

先安装rouge，然后生成CSS文件，并修改博客中的style标签至新的CSS文件

```
gem install rouge
rougify style monokai.sublime > rouge.css
```

header.html:

```html
<link href="{{ site.baseurl }}/css/rouge.css" rel="stylesheet">
```

修改完后发现rouge的css是深色系的，需要将代码块的背景色黑一些

```css
pre{
    font-size: 0.9em;
    background: #0c2029;
}
```

### 解决分页问题

需要安装jekyll-paginate，并修改配置文件

```
gem install jekyll-paginate
```

_config.yml

```
# 添加一行
gems: [jekyll-paginate]
```

这样本地的页面就和gitpage一致了

## 制作jekyll的dockerfile

将安装步骤和上文的修复步骤结合起来写了如下的dockerfile：

```bash
# Python 3 Development With Jeklly

FROM centos

MAINTAINER arthur_mao@live.com

# YUM install apps
RUN yum install -y epel-release
RUN yum install -y wget
RUN yum install -y git
RUN yum install -y vim
RUN yum install -y which
RUN yum groupinstall -y "Development Tools"
RUN yum install -y sqlite-devel openssl openssl-devel

# Source Code install Python3 and PIP
ADD Python-3.4.3.tgz /tmp/
WORKDIR /tmp/Python-3.4.3/
RUN ./configure --with-ensurepip=install
RUN make
RUN make install
RUN ln -s /opt/Python/bin/python3 /usr/bin/python3

# Source Code install Ruby 2.3
ADD ruby-2.3.1.tar.gz /tmp/
WORKDIR /tmp/ruby-2.3.1/
RUN ./configure
RUN make
RUN make install
RUN gem sources --add https://gems.ruby-china.org/ --debug -v
RUN gem sources --remove  https://rubygems.org/
RUN gem install bundler
ADD Gemfile /
RUN bundle install
RUN gem install rouge
RUN gem install jekyll-paginate
RUN gem update jekyll
RUN yum install -y nodejs

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
```

## 制作docker-compose.yml

我们需要nginx来代理这个jekyll server，使用docker-compose来定义这两个容器

```yaml
jekyll.compose.arthurmao.me:
    build: ./docker_jekyll
    ports:
        - "8000:8000"
    volumes:
        - /home/arthur/git/arthurmmm.github.io/:/root/git/arthurmmm.github.io/
    command: "jekyll serve --source /root/git/arthurmmm.github.io -H 0.0.0.0 -P 8000"
    restart: always

nginx.compose.arthurmao.me:
    build: ./docker_nginx/nginx.arthurmao.me
    ports:
        - "80:80"
        - "443:443"
    volumes:
        - /home/arthur/git/docker/docker_nginx/nginx.arthurmao.me/nginx.conf:/etc/nginx/nginx.conf
    links:
        - jekyll.compose.arthurmao.me
    restart: always
```

## 配置nginx

```
# 添加如下配置到nginx.conf => http
    upstream jekyll {
        server jekyll.compose.arthurmao.me:8000;
    }
    
    server {
        listen 80;
        server_name www.arthurmao.me;
        location / {
            proxy_pass http://jekyll;
        }
    }
```

这样一来访问www.arthurmao.me就可以访问云服务器上的博客，访问blog.arthurmao.me还是可以访问gitpage上的博客。