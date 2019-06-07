---
title: docker_example
date: 2019-05-12 15:10:34
tags: docker_container
---


### 主题　　
　　这两天学了一点容器和docker的东西，主要是正在看一本书《容器与容器云》，做了书上的第二章的例子，记录一下踩到的坑。
　　
　　例子主要是搭建一个简单的http服务器，用到了redis、django、haproxy等几个镜像。  
　　书上的过程说得不是很清楚，直接照着做有些问题，需要根据前面的内容和理解做些调整。主要的步骤如下：  
　　<!-- more -->
```
sudo docker run -it --name redis-master -v ~/Documents/tmp/project/redis-master:/data redis /bin/bash
sudo docker run -it --name redis-slave1 --link redis-master:master  -v ~/Documents/tmp/project/redis-slave1:/data redis /bin/bash
sudo docker run -it --name redis-slave2 --link redis-master:master  -v ~/Documents/tmp/project/redis-slave2:/data redis /bin/bash
sudo docker run -it --name APP1 --link redis-master:db -v ~/Documents/tmp/project/Django/App1
sudo docker run -it --name APP2 --link redis-master:db -v ~/Documents/tmp/project/Django/App2
sudo docker run -it --name HAProxy --link APP1:APP1 --link APP2:APP2 -p 6301:6301 -v ~/Documents/tmp/project/HAProxy:/tmp haproxy /bin/bash
```
　　前面这几步主要是基于容器镜像建立相关的容器，同时把主机上的一些路径挂载映射到容器中。查看挂载路径时，用sudo docker inspect redis-master | grep -A 10 'Mounts' 可以查看。

　　
　　同时需要自己在网上下载redis的配置文件：
```
wget http://download.redis.io/releases/redis-4.0.8.tar.gz
tar -xzvf redis-4.0.8.tar.gz
```
　　把配置文件拷到主机上比如~/Documents/tmp/project/redis-master，相应的在redis-master容器中的/data目录下就可以看到了，此处需要注意的坑是要修改配置文件，书上没有提到bind 127.0.0.1处，不修改的话，主从redis无法连接：
　　[参考](https://blog.csdn.net/chwshuang/article/details/54929277)
```
redis-master:

bind 0.0.0.0		//不是127.0.0.1
daemonsize yes
pidfile /var/run/redis.log

redis-slave
daemonsize yes
pidfile /var/run/redis.log
slaveof redis-master 6379   //不是master
```
　　书上django需要的文件比较多，有些可能比较旧了，需要稍微改一些：
```
view.py

from django.shortcuts import render
from django.http import HttpResponse
# Create your views here.

import redis

def hello(request):
    str=redis.__file__
    str+="<br>"
    r = redis.Redis(host='db', port=6379, db=0)
    info = r.info()
    str+=("Set Hi <br>")
    r.set('Hi', 'HelloWorkd-App1')
    str+=("Get Hi: %s <br>" % r.get('Hi'))
    str+=("Redis Info: <br>")
    str+=("Key: Info Value")
    for key in info:
        str+=("%s : %s <br>" % (key, info[key]))
    return HttpResponse(str)
    
urls.py

from django.conf.urls import include, url
from django.contrib import admin
from helloworld.views import hello
urlpatterns = [
        url(r'^admin/', include(admin.site.urls)),
        url(r'^helloworld$', hello),
        ]
        
settings.py

ALLOWED_HOSTS = ['*']  //允许不同ip主机访问，不加'*'，一会无法访问网址

```
　　最后的效果如图：
　　![img](docker_container/1.png)

