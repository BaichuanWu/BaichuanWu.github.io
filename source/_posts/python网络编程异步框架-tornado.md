---
title: python网络编程-tornado
date: 2016-10-18 16:30:33
categories: programming
tags: article
---

### Tornado

#### 1.相关概念

##### 1.1 tornado的组成

> - web应用框架（包括RequestHandler）
> - http服务端和客户端实现（HTTPServer 和 AsyncHTTPClient）
> - 一个异步网络库，用作HTTP组件的构建块，并且还可以用于实现其他协议（IOLoop,IOStream）
> - corountine库，提供除了链式回调函数别的异步的接口（tornado.gen）
>
> 可以组合使用wsgi接口服务器和web应用服务器
>
> - 使用WSGIAdapter 可以将tornado web应用和别的wsgi接口服务器结合
> - 使用WSGIContainer可以讲tornado 的wsgi 接口服务器和 其他web应用结合

##### **1.2异步接口形式**

<!-- more -->

> - Callback argument
>
>   ```python
>   from tornado.httpclient import AsyncHTTPClient
>
>   def asynchronous_fetch(url, callback):
>       http_client = AsyncHTTPClient()
>       def handle_response(response):
>           callback(response.body)
>       http_client.fetch(url, callback=handle_response)
>   ```
>
> - Return a placeholder ([`Future`](http://www.tornadoweb.org/en/stable/concurrent.html#tornado.concurrent.Future), `Promise`, `Deferred`)
>
>   ```python
>   from tornado.concurrent import Future
>   from tornado import gen
>
>   def async_fetch_future(url):
>     http_client = AsyncHTTPClient()
>     my_future = Future()
>     fetch_future = http_client.fetch(url)
>     fetch_future.add_done_callback(
>         lambda f: my_future.set_result(f.result()))
>     return my_future
>
>   @gen.coroutine
>   def fetch_coroutine(url):
>     http_client = AsyncHTTPClient()
>     response = yield http_client.fetch(url)
>     raise gen.Return(response.body)   # python2 不支持返回生成器，在python3中直接 return response.body
>   ```
>
>
> - Deliver to a queue
>
> - Callback registry (e.g. POSIX signals)



##### 