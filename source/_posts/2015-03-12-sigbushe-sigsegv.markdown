---
layout: post
title: "SIGBUS和SIGSEGV"
date: 2014-03-12 17:50:55 +0800
comments: true
categories: linux
---

SIGBUS和SIGSEGV也许是我们在平时遇到的次数最多的两个内存错误信号。内存问题一直是最令我们头疼的事情，弄清楚两个信号的发生缘由对我们很好的理解程序的运行是大有裨益的。

我们来看两段程序：

``` c
//testsigsegv.c
int main() {
        char *pc = (char*)0x00001111;
        *pc = 17;
}

//testsigbus.c
int main() {
        int *pi = (int*)0x00001111;
        *pi = 17;
}

```

上面的代码那么的相似，我们也同样用gcc编译(加上-g选项，便于gdb调试；平台Solaris Sparc)，执行结果也都是dump core。但通过GDB对core进行观察，你会发现细微的不同。

第一个例子出的core原因是：Program terminated with signal 11, Segmentation fault. 

第二个例子的core则提示：Program terminated with signal 10, Bus error. 

两者有什么不同呢？这两段代码的共同点都是将一个非法地址赋值给指针变量，然后试图写数据到这个地址。

如果要说清楚这个问题，我们就要结合汇编码和一些计算机的体系结构的知识来共同分析了。

先来看testsigsegv.c的汇编码：
```
main:
        !#PROLOGUE# 0
        save    %sp, -120, %sp
        !#PROLOGUE# 1
        sethi   %hi(4096), %i0
        or      %i0, 273, %i0
        st      %i0, [%fp-20]
        ld      [%fp-20], %i1
        mov     17, %i0
        stb     %i0, [%i1]
        nop
        ret
        restore
```

我们关注的是这句：stb     %i0, [%i1]
从计算机底层的执行角度来说，过程是如何的呢？%i0寄存器里存储的是立即数17，我们要将之存储到寄存器%i1的值指向的内存地址。这一过程对于CPU来说其指挥执行的正常过程是：将寄存器%i0中的值送上数据总线，将寄存器%i1的值送到地址总线，然后使能控制总线上的写信号完成这一向内存写1 byte数据的过程。

我们再看testsigbus.c的汇编码：

```
main:
        !#PROLOGUE# 0
        save    %sp, -120, %sp
        !#PROLOGUE# 1
        sethi   %hi(4096), %i0
        or      %i0, 273, %i0
        st      %i0, [%fp-20]
        ld      [%fp-20], %i1
        mov     17, %i0
        st      %i0, [%i1]
        nop
        ret
        restore
```

同样最后一句：st      %i0, [%i1]，CPU执行的过程与testsigsegv.c中的一致(只是要存储数据长度是4字节)，那为什么产生错误的原因不同呢？一个是SIGSEGV，而另一个是SIGBUS。这里涉及到的就是对内存地址的校验的问题了，包括对内存地址是否对齐的校验以及该内存地址是否合法的校验。

我们假设如果首先进行的内存地址是否合法的校验(是否归属于用户进程的地址空间)，那么我们回顾一下，这两个程序中的地址0x00001111显然都不合法，按照这种流程，两个程序都应该是SIGSEGV导致的core才对，但是事实并非如此。那难道是先校验内存地址的对齐？我们再看这种思路是否合理？

testsigsegv.c中，0x00001111这个地址值被赋给了char *pc；也就是告诉CPU通过这个地址我们要存取一个字节的值，对于一个字节长度的数据，无所谓对齐，所以该地址通过对齐校验；并被放到地址总线上了。而在testsigbus.c里，0x00001111这个地址值被赋给了int *pi；也就是告诉CPU通过这个地址我们要存取一个起码4个字节的值，那么对于长度4个字节的对象，其存放地址起码要被4整除才可以，而0x00001111这个值显然不能满足要求，也就不能通过内存对齐的校验。也就是说SIGBUS这个信号在地址被放到地址总线之后被检查出来的不符合对齐的错误；而SIGSEGV则是在地址已经放到地址总线上后，由后续流程中的某个设施检查出来的内存违法访问错误。

一般我们平时遇到SIGBUS时总是因为地址未对齐导致的，而SIGSEGV则是由于内存地址不合法造成的。
1) SIGBUS(Bus error)意味着指针所对应的地址是有效地址，但总线不能正常使用该指针。通常是未对齐的数据访问所致。
2) SIGSEGV(Segment fault)意味着指针所对应的地址是无效地址，没有物理内存对应该地址。
 Linux的mmap(2)手册页
使用映射可能涉及到如下信号
SIGSEGV    试图对只读映射区域进行写操作
SIGBUS     试图访问一块无文件内容对应的内存区域，比如超过文件尾的内存区域，或者以前有文件内容对应，现在为另一进程截断过的内存区域。

调试方法：
gcc -g 编译 
ulimit -c 20000 
之后运行程序，等core dump 
最后gdb -c core <exec file> 

>
转载: http://bigwhite.blogbus.com/logs/12296535.html
