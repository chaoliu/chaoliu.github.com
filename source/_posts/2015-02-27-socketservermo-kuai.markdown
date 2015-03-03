---
layout: post
title: "SocketServer模块"
date: 2015-02-27 18:10:42 +0800
comments: true
categories: python
---
SocketServer是标准库中一个高级别的模块，用于简化网络客户与服务器的实现。模块中，已经实现了一些可供使用的类。

<!-- more -->

在Python3中，本模块为socketserver模块。在Python 2中，本模块为SocketServer模块。所以在用import导入时，要分情况导入，否则会报错。导入的代码如下：

``` python
try:
    import socketserver      #Python 3
except ImportError:
    import SocketServer      #Python 2

```

SocketSerror模块包括许多可以简化TCP、UDP、UNIX域套接字  服务器实现的类。

一、处理程序
-----------
要使用本模块，必须定义一个继承于基类BaseRequestHandler的处理程序类。BaseRequestHandler类的实例h可以实现以下方法：

1、h.handle()  调用该方法执行实际的请求操作。调用该函数可以不带任何参数，但是几个实例变量包含有用的值。h.request包含请求，h.client_address包含客户端地址，h.server包含调用处理程序的实例。对于TCP之类的数据流服务，h.request属性是套接字对象。对于数据报服务，它是包含收到数据的字节字符串。

2、h.setup()   该方法在handle()之前调用。默认情况下，它不执行任何操作。如果希望服务器实现更多连接设置（如建立SSL连接），可以在这里实现。

3、h.finish()   调用本方法可以在执行完handle()之后执行清除操作。默认情况下，它不执行任何操作。如果setup()和handle()方法都不生成异常，则无需调用该方法。

如果知道应用程序只能操纵面向数据流的连接（如TCP），那么应从StreamRequestHandler继承，而不是BaseRequestHandler。StreamRequestHandler类设置了两个属性，h.wfile是将数据写入客户端的类文件对象，h.rfile是从客户端读取数据的类文件对象。

如果要编写针对数据包操作的处理程序并将响应持续返回发送方，那么它应当从DatagramRequestHandler继承。它提供的类接口与StramRequestHandler相同。

二、服务器
----------

要使用处理程序，必须将其插入到服务器对象。定义了四个基本的服务器类。

1、 TCPServer(address,handler)   支持使用IPv4的TCP协议的服务器，address是一个（host,port）元组。Handler是BaseRequestHandler或StreamRequestHandler类的子类的实例。

2、 UDPServer(address,handler)   支持使用IPv4的UDP协议的服务器，address和handler与TCPServer中类似。

3、UnixStreamServer(address,handler)   使用UNIX域套接字实现面向数据流协议的服务器，继承自TCPServer。

4、 UnixDatagramServer(address,handler)  使用UNIX域套接字实现数据报协议的服务器，继承自UDPServer。

所有四个服务器类的实例都有以下方法和变量：

1、s.socket   用于传入请求的套接字对象。

2、s.sever_address  监听服务器的地址。如元组（"127.0.0.1",80）

3、s.RequestHandlerClass   传递给服务器构造函数并由用户提供的请求处理程序类。

4、s.serve_forever()  处理无限的请求

5、s.shutdown()   停止serve_forever()循环

6、s.fileno()   返回服务器套接字的整数文件描述符。该方法可以有效地通过轮询操作（如select()函数）使用服务器实例。

三、定义自定义服务器
--------------
服务器往往需要特殊的配置来处理不同的网络地址族、超时期、并发和其他功能，可以通过继承上面四个基本服务器类来自行定义。

可以通过混合类获得更多服务器功能，这也是通过进程或线程分支添加并发行的方法。为了实现并发性，定义了以下类：

（1）ForkingMixIn         将UNIX进程分支添加到服务器的混合方法，使用该方法可以让服务器服务多个客户。

（2）ThreadingMixIn    修改服务器的混合类，可以使用线程服务多个客户端。

要向服务器添加这些功能，可以使用多重继承，其中首先列出混了类。

由于并发服务器很常用，为了定义它，SocketServer预定义了以下服务器类：

（1）ForkingUDPServer(address,handler)   

（2）ForkingTCPServer(address,handler)

（3）ThreadingUDPServer(address,handler)

（4）ThreadingTCPServer(address,handler)

 

上面有点乱，现总结以下：

SocketServer模块中的类主要有以下几个：

1、BaseServer    包含服务器的核心功能与混合类（mix-in）的钩子功能。这个类主要用于派生，不要直接生成这个类的类对象，可以考虑使用TCPServer和UDPServer类。

2、TCPServer    基本的网络同步TCP服务器

3、UDPServer    基本的网络同步UDP服务器

4、ForkingMixIn   实现了核心的进程化功能，用于与服务器类进行混合(mix-in)，以提供一些异步特性。不要直接生成这个类的对象。

5、ThreadingMixIn   实现了核心的线程化功能，用于与服务器类进行混合(mix-in)，以提供一些异步特性。不要直接生成这个类的对象。

6、ForkingTCPServer     ForkingMixIn与TCPServer的组合

7、ForkingUDPServer    ForkingMixIn与UDPServer的组合

8、BaseRequestHandler

9、StreamRequestHandler    TCP请求处理类的一个实现

10、DataStreamRequestHandler   UDP请求处理类的一个实现

 

现在繁杂的事务都已经封装到类中了，直接使用类即可。

使用SocketServer模块编写的TCP服务器端代码：

``` python
#! /usr/bin/env python
#coding=utf-8
"""使用SocketServer来实现简单的TCP服务器"""
from SocketServer import (TCPServer,StreamRequestHandler  as SRH)
from time import ctime
 
class MyRequestHandler(SRH):
    def handle(self):
        print "connected from ",self.client_address
        self.wfile.write("[%s] %s"  %(ctime(),self.rfile.readline()))
 
tcpSer=TCPServer(("",10001),MyRequestHandler)
print "waiting for connection"
tcpSer.serve_forever()

```

相应的TCP客户端代码：


``` python
#! /usr/bin/env python
#coding=utf-8
from socket import *
BUFSIZE=1024
#每次都要创建新的连接
while True:
    tcpClient=socket(AF_INET,SOCK_STREAM)
    tcpClient.connect(("localhost",10001))
    data=raw_input(">")
    if not data:
        break
    tcpClient.send("%s\r\n" %data)
    data1=tcpClient.recv(BUFSIZE)
    if not data1:
        break
    print data1.strip()
    tcpClient.close()

```

> 本文转载自: http://my.oschina.net/duhaizhang/blog/68639