---
layout: post
title: "unix domain socket"
date: 2015-03-05 14:44:25 +0800
comments: true
categories: linux
---

socket API原本是为网络通讯设计的，但后来在socket的框架上发展出一种IPC机制，就是UNIXDomain Socket。虽然网络socket也可用于同一台主机的进程间通讯（通过loopback地址127.0.0.1），但是UNIX Domain Socket用于IPC更有效率：不需要经过网络协议栈，不需要打包拆包、计算校验和、维护序号和应答等，只是将应用层数据从一个进程拷贝到另一个进程。这是因为，IPC机制本质上是可靠的通讯，而网络协议是为不可靠的通讯设计的。UNIX Domain Socket也提供面向流和面向数据包两种API接口，类似于TCP和UDP，但是面向消息的UNIX Domain Socket也是可靠的，消息既不会丢失也不会顺序错乱。
<!-- more -->
UNIX Domain Socket是全双工的，API接口语义丰富，相比其它IPC机制有明显的优越性，目前已成为使用最广泛的IPC机制，比如X Window服务器和GUI程序之间就是通过UNIX Domain Socket通讯的。

使用UNIX Domain Socket的过程和网络socket十分相似，也要先调用socket()创建一个socket文件描述符，address family指定为AF_UNIX，type可以选择SOCK_DGRAM或SOCK_STREAM，protocol参数仍然指定为0即可。

UNIX Domain Socket与网络socket编程最明显的不同在于地址格式不同，用结构体sockaddr_un**表示，网络编程的socket地址是IP地址加端口号，而UNIX Domain Socket的地址是一个socket类型的文件在文件系统中的路径，这个socket文件由bind()调用创建，如果调用bind()时该文件已存在，则bind()错误返回

服务器端的步骤如下：
>
1. socket：      建立一个socket
2. bind：          将这个socket绑定在某个文件上（AF_UNIX）或某个端口上（AF_INET），我们会分别介绍这两种。
3. listen：        开始监听
4. accept：      如果监听到客户端连接，则调用accept接收这个连接并同时新建一个socket来和客户进行通信
5. read/write：读取或发送数据到客户端
6. close：        通信完成后关闭socket


客户端的步骤如下：
>
1. socket：      建立一个socket
2. connect：   主动连接服务器端的某个文件（AF_UNIX）或某个端口（AF_INET）
3. read/write：如果服务器同意连接（accept），则读取或发送数据到服务器端
4. close：        通信完成后关闭socket


``` C Makefile
all: tcp_client.c tcp_server.c  
    gcc -g -Wall -o tcp_client tcp_client.c  
    gcc -g -Wall -o tcp_server tcp_server.c  
  
clean:  
    rm -rf *.o tcp_client tcp_server

```

``` C tcp_server.c

#include <sys/types.h>  
#include <sys/socket.h>  
#include <sys/un.h>  
#include <unistd.h>  
#include <stdlib.h>  
#include <stdio.h>  
  
int main()  
{  
  /* delete the socket file */  
  unlink("server_socket");  
    
  /* create a socket */  
  int server_sockfd = socket(AF_UNIX, SOCK_STREAM, 0);  
    
  struct sockaddr_un server_addr;  
  server_addr.sun_family = AF_UNIX;  
  strcpy(server_addr.sun_path, "server_socket");  
    
  /* bind with the local file */  
  bind(server_sockfd, (struct sockaddr *)&server_addr, sizeof(server_addr));  
    
  /* listen */  
  listen(server_sockfd, 5);  
    
  char ch;  
  int client_sockfd;  
  struct sockaddr_un client_addr;  
  socklen_t len = sizeof(client_addr);  
  while(1)  
  {  
    printf("server waiting:\n");  
      
    /* accept a connection */  
    client_sockfd = accept(server_sockfd, (struct sockaddr *)&client_addr, &len);  
      
    /* exchange data */  
    read(client_sockfd, &ch, 1);  
    printf("get char from client: %c\n", ch);  
    ++ch;  
    write(client_sockfd, &ch, 1);  
      
    /* close the socket */  
    close(client_sockfd);  
  }  
    
  return 0;  
}  

```

``` C tcp_client.c
#include <sys/types.h>  
#include <sys/socket.h>  
#include <sys/un.h>  
#include <unistd.h>  
#include <stdlib.h>  
#include <stdio.h>  
  
int main()  
{  
  /* create a socket */  
  int sockfd = socket(AF_UNIX, SOCK_STREAM, 0);  
    
  struct sockaddr_un address;  
  address.sun_family = AF_UNIX;  
  strcpy(address.sun_path, "server_socket");  
    
  /* connect to the server */  
  int result = connect(sockfd, (struct sockaddr *)&address, sizeof(address));  
  if(result == -1)  
  {  
    perror("connect failed: ");  
    exit(1);  
  }  
    
  /* exchange data */  
  char ch = 'A';  
  write(sockfd, &ch, 1);  
  read(sockfd, &ch, 1);  
  printf("get char from server: %c\n", ch);  
    
  /* close the socket */  
  close(sockfd);  
    
  return 0;  
}

```