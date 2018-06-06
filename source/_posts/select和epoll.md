---
title: select和epoll
date: 2017-08-12 23:25:00
tags: summary
---

### SELECT 和EPOLL

<!-- more -->

- 一个流可以是文件，socket，pipe 等等可以进行I/O操作的内核对象。不管是文件，还是套接字，还是管道，我们都可以把他们看作流

- 

- **阻塞I/O模式下**，一个线程只能处理一个流的I/O事件。如果想要同时处理多个流，要么多进程(fork)，要么多线程(pthread_create)，很不幸这两种方法效率都不高。

- **非阻塞忙轮询的I/O方式**,我们只要不停的把所有流从头到尾问一遍，又从头开始。这样就可以处理多个流了，但这样的做法显然不好，因为如果所有的流都没有数据，那么只会白白浪费CPU。这里要补充一点，阻塞模式下，内核对于I/O事件的处理是阻塞或者唤醒，而非阻塞模式下则把I/O事件交给其他对象（后文介绍的 select 以及 epoll）处理甚至直接忽略。

- 为了避免CPU空转，可以引进了一个**代理**(select,epoll)，代理可以同时观察许多流的I/O事件，在空闲的时候，**会把当前线程阻塞掉**，当有一个或多个流有I/O事件时，就从阻塞态中醒来

  ​

#### select:

- 如果没有I/O事件产生，我们的程序就会阻塞在select处。但是依然有个问题，我们从select那里仅仅知道了，有I/O事件发生了，但却并不知道是那几个流（可能有一个，多个，甚至全部），我们只能**无差别轮询**所有流，找出能读出数据，或者写入数据的流，对他们进行操作
- 我们有O(n)的**无差别**轮询复杂度，同时处理的流越多，每一次无差别轮询时间就越长。



#### epoll:

- ​

```python
# 忙轮询
while true {
	for i in stream[]; {
		if i has data
			read until unavailable
	}
} 

#无差别轮训select
while true {
	select(streams[])  #都没有IO时阻塞
	for i in streams[] {
		if i has data
			read until unavailable
	}
}

# epoll
while true {
	active_stream[] = epoll_wait(epollfd)
	for i in active_stream[] {
		read or write till unavailable
	}
}
```