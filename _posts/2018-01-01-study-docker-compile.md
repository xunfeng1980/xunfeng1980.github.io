---
layout: post
title:  "Docker Compose 的简单使用"
date:   2018-01-01 23:44:18 +0800
categories: docker
---

## 介绍

​	Docker Compose 官方介绍：

​	*Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application’s services. Then, with a single command, you create and start all the services from your configuration.*

​	docker-compose是docker官方开源的容器编排工具，优点是简单易用，而且功能强大。虽然docker公司在商业化后主推它的生产级容器集群平台Swarm，但是Swarm好像没有那么幸运(前几天看到博客园就因为上了Swarm，结果基本全站挂了…),最近被k8s打得不行。如果容器数量较少，或是为了学习docker，可以先学习使用docker-compose。



## 安装

​	在https://www.docker.com/community-edition下载社区版本Docker CE，安装之后，docker和docker-compose都有了。

​	确认是否安装成功：

``` shell
docker-compose -v
docker-compose version 1.17.1, build 6d101fb
```



## 使用

​	下面我们按照官方的demo来一遍：

​	1.创建项目目录

```shell
$ mkdir composetest
$ cd composetest
```

​	

​	2.创建一个app.py,直接上vim，粘贴以下代码

```python
import time

import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)


@app.route('/')
def index():
    count = cache.get('hits')
    return 'You are get: {} times.\n'.format(count)

@app.route('/incr')
def incr():
    count = cache.incr('hits')
    return 'You are incr: {} times.\n'.format(count)

@app.route('/decr')
def decr():
    count = cache.decr('hits')
    return 'You are decr: {} times.\n'.format(count)


if __name__ == "__main__":
    app.run(host="0.0.0.0", debug=True)

```

​	

​	3.创建构建镜像使用的Dockerfile

```dockerfile
FROM python:3.4-alpine
ADD . /code
WORKDIR /code
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```



​	4.由2可知我们这个项目需要依赖Flask和Redis，需要把依赖放到requirements.txt，然后pip去安装

```shell
flask
redis
```



​	5.最后最重要的一步，docker-compose.yml文件描述了整个项目服务的定义

```dockerfile
version: '3'
services:
  web:
    build: .
    ports:
     - "5000:5000"
    volumes:
     - .:/code
  redis:
    image: "redis:alpine"
```

​	6.在项目目录执行docker-compose up -d项目就可以跑起来，需要下载基础镜像和构建，会花费一些时间。

如果遇到ERROR: Couldn't connect to Docker daemon. You might need to start Docker for Mac.请检查Docker是否已经启动。

​	启动成功输出：

```shell
docker-compose up -d
Starting composetest_web_1 ...
Starting composetest_web_1
Starting composetest_redis_1 ...
Starting composetest_web_1 ... done
```

```shell
docker-compose ps
       Name                      Command               State           Ports
-------------------------------------------------------------------------------------
composetest_redis_1   docker-entrypoint.sh redis ...   Up      6379/tcp
composetest_web_1     python app.py                    Up      0.0.0.0:5000->5000/tcp
```

​	访问http://127.0.0.1:5000/将会看到目前一个统计数，http://127.0.0.1:5000/incr 将会增加计数，反之decr将会减少计数。



## 后续计划

1.k8s和istio搭建以及简单使用

2.spring boot/cloud侵入式微服务与Service Mesh非侵入式微服务的对比

3.Docker与Unikernel的对比



## 参考文档

1. [Docker Compose Doc](https://docs.docker.com/compose/overview/)
2. [博客园Docker Swarm 集群宕机事件](http://www.cnblogs.com/cmt/p/8143854.html)
3. [后Kubernetes时代](http://mp.weixin.qq.com/s?__biz=MjM5MDE0Mjc4MA==&mid=2651000439&idx=1&sn=df791ea111ecf6237ca1fabad9142d45&chksm=bdbef6248ac97f326cf1efd16056538e8c43d94805ee0040bfee9033ce38b508bcc035ca766d&mpshare=1&scene=23&srcid=0109DQ37i2cMeL8mKtd7LcTW#rd)

