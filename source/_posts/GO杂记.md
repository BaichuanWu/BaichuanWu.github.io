---
title: GO杂
date: 2018-04-05 00:05:32
tags:
---

### goroutine

> - Go的并发同步模型CSP
> - Go的并发调度器是在操作系统之上，将操作系统的线程和语言运行时的逻辑处理器绑定，在逻辑处理器运行goroutine

> 1. 将goroutine 放在逻辑处理器的执行队列
> 2. goroutine 执行了阻塞的系统调用时，调度器将线程与处理器分离，线程一边阻塞等待系统调用返回，同时创建新的线程绑定到逻辑处理器，分配新的goroutine。
> 3. goroutine执行网络IO调用时，会将goroutine放到网络轮询器，当轮询器表明网络IO操作已经就绪，重新分配goroutine到逻辑处理器

<!-- more -->