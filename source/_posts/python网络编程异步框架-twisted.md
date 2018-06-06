---
title: python网络编程异步框架1-twisted
date: 2016-09-21 16:11:51
categories: programming
tags: article
---

## 1 前言

##### 1.1 异步的编程模型和多线程模型

> - 多线程中每个任务都是单独在线程中完成，其调度有操作系统（或是更底层）管理，涉及互斥元，涉及线程中的调度的相关日后补充。
> - 异步中一个任务运行必须显示的放弃当前任务，由程序员调度。往往需要将一个任务拆分来交替小步完成；任务之间的切换要不是此任务完成，要不就是它被阻塞。

##### 1.2 异步模型适用场景

> - 有大量的任务，因此在一个时刻至少有一个任务要运行
> - 任务执行大量的I/O操作，这样同步模型就会在因为任务阻塞而浪费大量的时间
> - 任务之间相互独立，以至于任务内部的交互很少。

## 1.3 select实现的一个异步的socket客户端

> - reactor模式: 利用循环体来等待事件发生，然后处理发生的事件的模型1

**recator的功能：**

> - .监视一系列与你I/O操作相关的文件描述符（description)
> - .不停地向你汇报那些准备好I/O操作的文件描述符

<!-- more -->

```python
import datetime, errno, optparse, select, socket

def parse_args():
    usage = """usage: %prog [options] [hostname]:port ...
This is the Get Poetry Now! client, asynchronous edition.
Run it like this:
  python get-poetry.py port1 port2 port3 ...
If you are in the base directory of the twisted-intro package,
you could run it like this:
  python async-client/get-poetry.py 10001 10002 10003
to grab poetry from servers on ports 10001, 10002, and 10003.
Of course, there need to be servers listening on those ports
for that to work.
"""
    parser = optparse.OptionParser(usage)
    _, addresses = parser.parse_args()

    if not addresses:
        print parser.format_help()
        parser.exit()

    def parse_address(addr):
        if ':' not in addr:
            host = '127.0.0.1'
            port = addr
        else:
            host, port = addr.split(':', 1)

        if not port.isdigit():
            parser.error('Ports must be integers.')
        return host, int(port)
    return map(parse_address, addresses)


def get_poetry(sockets):
    """Download poety from all the given sockets."""
    poems = dict.fromkeys(sockets, '') # socket -> accumulated poem
    # socket -> task numbers
    sock2task = dict([(s, i + 1) for i, s in enumerate(sockets)])
    sockets = list(sockets) # make a copy
    # we go around this loop until we've gotten all the poetry
    # from all the sockets. This is the 'reactor loop'.
    while sockets:
        # this select call blocks until one or more of the
        # sockets is ready for read I/O
        rlist, _, _ = select.select(sockets, [], [])
        # 当有IO准备好的时才返回，rlist,_,_分别是可读，可写，可执行的IO  select.select(rlist, wlist, xlist, timeout=None) select模块中的select方法是用来识别是其监视的socket是否有完成数据接收的，如果没有它就处于阻塞状态
        for sock in rlist:
            data = ''
            while True:
                try:
                    new_data = sock.recv(1024)
                except socket.error, e:
                    if e.args[0] == errno.EWOULDBLOCK:
                        # this error code means we would have
                        # blocked if the socket was blocking.
                        # instead we skip to the next socket
                        break
                    raise
                else:
                    if not new_data:
                        break
                    else:
                        data += new_data
            task_num = sock2task[sock]
            if not data:
                sockets.remove(sock)
                sock.close()
            poems[sock] += data
    return poems


def connect(address):
    """Connect to the given server and return a non-blocking socket."""
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.connect(address)
    sock.setblocking(0) # 设置了非阻塞模式
    return sock


def format_address(address):
    host, port = address
    return '%s:%s' % (host or '127.0.0.1', port)


def main():
    addresses = parse_args()
    start = datetime.datetime.now()
    sockets = map(connect, addresses)
    poems = get_poetry(sockets)
    elapsed = datetime.datetime.now() - start
    for i, sock in enumerate(sockets):
        print 'Task %d: %d bytes of poetry' % (i + 1, len(poems[sock]))
    print 'Got %d poems in %s' % (len(addresses), elapsed)

if __name__ == '__main__':
    main()
    
# 将处理业务混杂到循环，非标准reactor
```



## 2 twisted

##### 2.1 twisted基本操作

```python
from twited.internet import pollreactor
pollreactor.install()  # 决定了reactor 系统调用使用poll 默认select
from twisted.internet import reactor
def hello():
    print 'Hello from the reactor loop!'
    print 'Lately I feel like I\'m stuck in a rut.'
reactor.callWhenRunning(hello) # 不要调用阻塞的函数影响性能，reactor不会因为回调函数的错误而中断运行
reactor.run()
```

> - 我们的代码与Twisted代码运行在同一个进程中
> - 当我们的代码运行时，Twisted代码是处于暂停状态的。
> - 同样，当Twisted代码处于运行状态时，我们的代码处于暂停状态。
> - reactor事件循环会在我们的回调函数返回后恢复运行。

##### 2.2 twisted的基本运用

> **reactor是twisted的核心**
>
> - Twisted的reactor只有通过调用reactor.run()才启动。
> - reactor循环是在其开始的线程中运行，也就是运行在主线程中。
> - 一旦启动，reactor就会在程序的控制下（或者具体在一个启动它的线程的控制下）一直运行下去。
> - reactor空转时并不会消耗任何CPU的资源。
> - 并不需要显式的创建reactor，只需要引入就OK了。
>
>



> - Transports 抽象是通过Twisted中interfaces模块中ITransport接口定义的。一个Twisted的Transport代表一个可以收发字节的单条连接。twisted会根据传递的host和port 生成一个twisted.internet.tcp.Client为transport并绑定到之后的protocol
>
>   ```python
>   class BaseProtocol:
>       """
>       This is the abstract superclass of all protocols.
>
>       Some methods have helpful default implementations here so that they can
>       easily be shared, but otherwise the direct subclasses of this class are more
>       interesting, L{Protocol} and L{ProcessProtocol}.
>       """
>       connected = 0
>       transport = None
>
>       def makeConnection(self, transport):
>           """Make a connection to a transport and a server.
>
>           This sets the 'transport' attribute of this Protocol, and calls the
>           connectionMade() callback.
>           """
>           self.connected = 1
>           self.transport = transport
>           self.connectionMade()
>   ```
>
> - Protocols 抽象由interfaces模块中的IProtocol定义，Protocol对象实现协议内容；严格意义上讲，每一个Twisted的Protocols类实例都为一个具体的连接提供协议解析。因此我们的程序每建立一条连接（对于服务方就是每接受一条连接），都需要一个协议实例。这就意味着，Protocol实例是存储协议状态与间断性（由于我们是通过异步I/O方式以任意大小来接收数据的）接收并累积数据的地方。
>
>   因此，Protocol实例如何得知它为哪条连接服务呢？如果你阅读IProtocol定义会发现一个makeConnection函数。这是一个回调函数，Twisted会在调用它时传递给其一个也是仅有的一个参数，即Transport实例。这个Transport实例就代表Protocol将要使用的连接。
>
>   ```python
>       def buildProtocol(self, addr):
>           """
>           Create an instance of a subclass of Protocol.
>
>           The returned instance will handle input on an incoming server
>           connection, and an attribute "factory" pointing to the creating
>           factory.
>
>           Alternatively, C{None} may be returned to immediately close the
>           new connection.
>
>           Override this method to alter how Protocol instances get created.
>
>           @param addr: an object implementing L{twisted.internet.interfaces.IAddress}
>           """
>           p = self.protocol()   # protocal 一般为 Protocol或其子类的引用
>           p.factory = self
>           return p
>   ```
>
> - Protocol Factories 抽象由IProtocolFactory来定义 ；buildProtocol方法在每次被调用时返回一个新Protocol实例，它就是Twisted用来为新连接创建新Protocol实例的方法。

##### 2.3 deferred

> deferred是twisted处理回调的抽象：一个Deferred有一对回调链，一个是为针对正确结果，另一个针对错误结果。
>
> - 每个deferred链每一节都会返回一个defer对象
>
> deferred 的流程
>
> 1. 一个deferred有一个callback/errback对链，它们以添加到deferred中的顺序依次排列
>
> 2. stage 0，即第一对callback/errback，会在deferred激活时调用，具体调用那个看激活deferred的方式，若是通过.errback激活，则调用errback；若是通过.callback激活则调用callback。
>
> 3. 如果stage N执行出现异常，则stage N+1的errback被调用，并且其参数即为stage N出现的异常
>
> 4. 同样，如果stage N成功，即没有抛出异常，则N+1的callback被调用，其第一个参数为stage N的返回值。(类似js里的promise，都是抽象了层层嵌套的回调函数？jquery也有deferred，**deferred的中间值存在哪里？**)![twisted流程](https://github.com/BaichuanWu/prictures/raw/master/python_info/network_frame/twisted_1.png)
>
>    tips:
>
>    addCallback向链中添加一个显式的callback函数与一个隐式的"pass-through"函数。一个pass-through函数只是虚设的函数，只将其第一个参数返回。由于errback回调函数的第一个参数是Failure，因此一个"path-through"的errback总是执行"失败"，即将异常传给下个errback回调。

