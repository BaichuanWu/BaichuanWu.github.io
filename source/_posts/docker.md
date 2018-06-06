---
title: docker
date: 2016-12-09 09:28:51
categories: programming
tags: tools
mathjax: true
---

## Docker

##### 基本概念

1.镜像：	一个只读模板（ex:一个包含完整debian操作系统环境，里面安装了nginx），镜像用来安装容器。

2.容器：镜像创建的运行实例（可以被启动，开始，停止，删除），容器在启动时创建一层可写作层作为最上层。

3.仓库：集中放置镜像的地方。



常用命令

```sh
docker run -it salivawbc/debian:tag /bin/bash  # 没有冒号默认latest 启动docker

docker commit -m "improve" -a "alian" 72f1a8a0e394 salivawbc/debian:tag  #将容器转化为镜像

docker (attach| rm |start | stop | restart) (container_name|container_id) 

docker login 

docker run -it -v /home/alian/Downloads:/home/outer_doc salivawbc/debian /bin/bash #挂载外部目录

docker exec -it (container_name|container_id) /bin/bash # 多终端进入
docker run --security-opt seccomp=unconfined  # 保证能访问地址空间 不开启无法gdb调试
```


​				
​					         


​			
​		
​	
