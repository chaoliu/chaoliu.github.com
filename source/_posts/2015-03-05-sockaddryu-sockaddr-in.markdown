---
layout: post
title: "sockaddr与sockaddr_in"
date: 2015-03-05 15:02:29 +0800
comments: true
categories: linux
---

``` c
struct sockaddr {
    unsigned short sa_family;     /* address family, AF_xxx */
    char sa_data[14];             /* 14 bytes of protocol address */
};

```

sa_family是地址家族，一般都是“AF_xxx”的形式。好像通常大多用的是都是AF_INET。
sa_data是14字节协议地址。
此数据结构用做bind、connect、recvfrom、sendto等函数的参数，指明地址信息。
但一般编程中并不直接针对此数据结构操作，而是使用另一个与sockaddr等价的数据结构sockaddr_in。
<!-- more -->
``` c netinet/in.h

struct sockaddr_in {
    short int sin_family;              /* Address family */
    unsigned short int sin_port;       /* Port number */
    struct in_addr sin_addr;           /* Internet address */
    unsigned char sin_zero[8];         /* Same size as struct sockaddr */
};

struct in_addr {
    unsigned long s_addr;
};

typedef struct in_addr {
    union {
        struct{
            unsigned char s_b1,
            s_b2,
            s_b3,
            s_b4;
        } S_un_b;
        
        struct {
            unsigned short s_w1,
            s_w2;
        } S_un_w;
            
        unsigned long S_addr;
    } S_un;
} IN_ADDR;

```
sin_family指代协议族，在socket编程中只能是AF_INET
sin_port存储端口号（使用网络字节顺序）
sin_addr存储IP地址，使用in_addr这个数据结构
sin_zero是为了让sockaddr与sockaddr_in两个数据结构保持大小相同而保留的空字节。
s_addr按照网络字节顺序存储IP地址
sockaddr_in和sockaddr是并列的结构，指向sockaddr_in的结构体的指针也可以指向
sockadd的结构体，并代替它。也就是说，你可以使用sockaddr_in建立你所需要的信息,
在最后用进行类型转换就可以了

``` c

struct sockaddr_in server_addr; 
bzero((char*)&server_addr,sizeof(server_addr));//初始化
server_addr.sa_family = AF_INET;
server_addr.sin_addr.s_addr = inet_addr("192.168.0.1");
……
等到要做转换的时候用：
（struct sockaddr*） server_addr

```

###sockaddr_un结构体
>
sockaddr_un
Unix socket address record.
Declaration
Source position: socketsh.inc line 152
type sockaddr_un = packed record
   sun_family: sa_family_t;//sa_family_t的类型是WORD,即unsiged short
Address family
   sun_path: array [0..107] of Char;
File name
end;
Description
sockaddr_un is used to store a UNIX socket addres for the Bind, Recv and Send calls.


### 其他
一般关心的是如何把网络中的地址转化成字符串的形式。用到的函数可以是
char * inet_ntoa(struct in_addr in); 以将in_addr类型的IP地址转换为字符串形式IP地址。
基本步骤是获得网络接口地址pcap_addr中的sockaddr *addr，然后强制类型转化成sockaddr_in。再获得其中的struct  in_addr sin_addr，这个网络字节存储的IP地址，最后调用函数inet_ntoa，完成转换。

常用字节转化函数：

``` c
u_long htonl(u_long hostlong)：将主机字节序转换为TCP/IP网络字节序.
u_short htons(u_short hostshort)：将主机字节序转换为TCP/IP网络字节序.
unsigned long in inet_addr(const char *cp)：将一个点分制的IP地址(如192.168.0.1)转换为网络二进制数字
int inet_aton(cont char* cp, struct in_addr *inp)：将网络地址转为网络二进制数字，与inet_addr的区别是，结果不是作为返回值，而是保存形参inp所指的in_addr结构体中
char *inet_ntoa(struct in_addr in)：将网络二进制数字转为网络地址

```