---
layout: post
title:  '使用docker-compose管理容器'
date:   2016-09-18 17:43 +0800
comments: true
category: OS
tags: [Python, Docker]
---

制作一个原生的docker容器需要以下几个步骤：创建镜像(dockerfile)，创建容器(docker create/run)，运行容器（docker start）   

docker-compose工具可以通过yaml配置文件来统一管理这整个过程，非常实用

## 安装docker-compose

可以直接使用pip安装

```
sudo python3 -m pip install docker-compose
```

# docker-compose.yml

Docker-compose的所有配置都是基于YAML文件的，文件名必须是docker-compose.yml   

上一篇文章中我们用了docker run指令来创建容器：`docker run -d -v /home/arthur/my.nginx/nginx.conf:/etc/nginx/nginx.conf:ro -v /home/arthur/my.nginx/log:/var/log/nginx/:rw -p 80:80 -p 443:443 --name my.nginx nginx`   

这条指令转化为compose配置即   

```
my.nginx:
    image: nginx
    volumes:
        - /home/arthur/my.nginx/nginx.conf:/etc/nginx/nginx.conf:ro
        - /home/arthur/my.nginx/log:/var/log/nginx/:rw
    ports:
        - "80:80"
        - "443:443"
```

一个docker-compose.yml可以包含多个容器的定义，可以很方便的管理多容器的应用。

# docker-compose CLI

有了配置文件后可以通过docker-compose指令启动容器：

```
$ docker-compose -h
Define and run multi-container applications with Docker.

Usage:
  docker-compose [-f <arg>...] [options] [COMMAND] [ARGS...]
  docker-compose -h|--help

Options:
  -f, --file FILE             Specify an alternate compose file (default: docker-compose.yml)
  -p, --project-name NAME     Specify an alternate project name (default: directory name)
  --verbose                   Show more output
  -v, --version               Print version and exit
  -H, --host HOST             Daemon socket to connect to

  --tls                       Use TLS; implied by --tlsverify
  --tlscacert CA_PATH         Trust certs signed only by this CA
  --tlscert CLIENT_CERT_PATH  Path to TLS certificate file
  --tlskey TLS_KEY_PATH       Path to TLS key file
  --tlsverify                 Use TLS and verify the remote
  --skip-hostname-check       Don't check the daemon's hostname against the name specified
                              in the client certificate (for example if your docker host
                              is an IP address)

Commands:
  build              Build or rebuild services
  bundle             Generate a Docker bundle from the Compose file
  config             Validate and view the compose file
  create             Create services
  down               Stop and remove containers, networks, images, and volumes
  events             Receive real time events from containers
  exec               Execute a command in a running container
  help               Get help on a command
  kill               Kill containers
  logs               View output from containers
  pause              Pause services
  port               Print the public port for a port binding
  ps                 List containers
  pull               Pulls service images
  push               Push service images
  restart            Restart services
  rm                 Remove stopped containers
  run                Run a one-off command
  scale              Set number of containers for a service
  start              Start services
  stop               Stop services
  unpause            Unpause services
  up                 Create and start containers
  version            Show the Docker-Compose version information
```

大部分指令和docker比较像就不一一介绍了，通常情况下我们可以简单的cd到compose file对应的目录，然后使用docker-compose up 来启动容器，up=build+run+start，如果镜像已经创建就会跳过build的过程，如果容器已经存在会重新创建容器，这点很实用，不像docker run如果容器已经存在的话会报错退出。   
官方文档： https://docs.docker.com/compose/