---
layout: post
title: "结构体初始化"
date: 2015-03-11 15:07:57 +0800
comments: true
categories: 
---

在看php fpm 源码时注意到有这样一段代码:

``` c php-src/sapi/fpm/fpm.c

struct fpm_globals_s {
    pid_t parent_pid;
    int argc;
    char **argv;
    char *config;
    char *prefix;
    char *pid;
    int running_children;
    int error_log_fd;
    int log_level;
    int listening_socket; /* for this child */
    int max_requests; /* for this child */
    int is_child;
    int test_successful;
    int heartbeat;
    int run_as_root;
    int force_stderr;
    int send_config_pipe[2];
};

extern struct fpm_globals_s fpm_globals;


struct fpm_globals_s fpm_globals = {
    .parent_pid = 0,
    .argc = 0,
    .argv = NULL,
    .config = NULL,
    .prefix = NULL,
    .pid = NULL,
    .running_children = 0,
    .error_log_fd = 0,
    .log_level = 0,
    .listening_socket = 0,
    .max_requests = 0,
    .is_child = 0,
    .test_successful = 0,
    .heartbeat = 0,
    .run_as_root = 0,
    .force_stderr = 0,
    .send_config_pipe = {0, 0},
};

```


结构体有几种初始化方式:

struct a {
    int b;
    int c;
}

有几种初始化方式：

struct a a1 = {
    .b = 1,
    .c = 2
};

或者
struct a a1 = {
    b:1,
    c:2
}

或者
struct a a1 = { 1, 2};

内核喜欢用第一种，这种方式称为指定初始化（designated initializer）。使用第一种和第二种时，成员初始化顺序可变。


它源自ISO C99标准。以下我摘录了C Primer Plus第五版中相关章节的内容，从而就可以很好的理解2.6版内核采用这种方式的优势就在于由此初始化不必严格按照定义时的顺序。这带来了极大的灵活性，其更大的益处还有待大家在开发中结合自身的应用慢慢体会。

已知一个结构，定义如下
struct book {
    char title[MAXTITL];
    char author[MAXAUTL];
    float value;
};

C99支持结构的指定初始化项目，其语法与数组的指定初始化项目近似。只是，结构的指定初始化项目使用点运算符和成员名（而不是方括号和索引值）来标识具体的元素。例如，只初始化book结构的成员value，可以这样做：
    struct book surprise = { .value = 10.99 };

可以按照任意的顺序使用指定初始化项目：
    struct book gift = { .value = 25.99,
                                    .author = "James Broadfool",
                                    .title = "Rue for the Toad"};
正像数组一样，跟在一个指定初始化项目之后的常规初始化项目为跟在指定成员后的成员提供了初始值。另外，对特定成员的最后一次赋值是它实际获得的值。例如，考虑下列声明：
    struct book gift = { .value = 18.90,
                                    .author = "Philionna pestle",
                                    0.25};
    这将把值0.25赋给成员value，因为它在结构声明中紧跟在author成员之后。新的值0.25代替了早先的赋值18.90。
    有关designated initializer的进一步信息可以参考c99标准的6.7.8节Ininialization。




5.22 特定的初始化
标准C89需要初始化语句的元素以固定的顺序出现，和被初始化的数组或结构体中的元素顺序一样。

在ISO C99中，你可以按任何顺序给出这些元素，指明它们对应的数组的下标或结构体的成员名，并且GNU C也把这作为C89模式下的一个扩展。这个扩展没有在GNU C++中实现。

为了指定一个数组下标，在元素值的前面写上“[index] =”。比如：

    
int a[6] = { [4] = 29, [2] = 15 };
            
相当于：

    
int a[6] = { 0, 0, 15, 0, 29, 0 };
            
下标值必须是常量表达式，即使被初始化的数组是自动的。

一个可替代这的语法是在元素值前面写上“.[index]”，没有“=”，但从GCC 2.5开始就不再被使用，但GCC仍然接受。为了把一系列的元素初始化为相同的值，写为“[first ... last] = value”。这是一个GNU扩展。比如：

    
int widths[] = { [0 ... 9] = 1, [10 ... 99] = 2, [100] = 3 };
            
如果其中的值有副作用，这个副作用将只发生一次，而不是范围内的每次初始化一次。

注意，数组的长度是指定的最大值加一。

在结构体的初始化语句中，在元素值的前面用“.fieldname = ”指定要初始化的成员名。例如，给定下面的结构体，

    
struct point { int x, y; };
            
和下面的初始化，

    
struct point p = { .y = yvalue, .x = xvalue };
            
等价于：

    
struct point p = { xvalue, yvalue };
            
另一有相同含义的语法是“.fieldname:”，不过从GCC 2.5开始废除了，就像这里所示：

    
struct point p = { y: yvalue, x: xvalue };
            
“[index]”或“.fieldname”就是指示符。在初始化共同体时，你也可以使用一个指示符（或不再使用的冒号语法），来指定共同体的哪个元素应该使用。比如：

    
union foo { int i; double d; };
            union foo f = { .d = 4 };
            
将会使用第二个元素把4转换成一个double类型来在共同体存放。相反，把4转换成union foo类型将会把它作为整数i存入共同体，既然它是一个整数。（参考5.24节向共同体类型转换。）

你可以把这种命名元素的技术和连续元素的普通C初始化结合起来。每个没有指示符的初始化元素应用于数组或结构体中的下一个连续的元素。比如，

    
int a[6] = { [1] = v1, v2, [4] = v4 };
            
等价于

    
int a[6] = { 0, v1, v2, 0, v4, 0 };
            
当下标是字符或者属于enum类型时，标识数组初始化语句的元素特别有用。例如：

    
int whitespace[256]
            = { [' '] = 1, ['\t'] = 1, ['\h'] = 1,
            ['\f'] = 1, ['\n'] = 1, ['\r'] = 1 };
            
你也可以在“=”前面写上一系列的“.fieldname”和“[index]”指示符来指定一个要初始化的嵌套的子对象；这个列表是相对于和最近的花括号对一致的子对象。比如，用上面的struct point声明：

    
struct point ptarray[10] = { [2].y = yv2, [2].x = xv2, [0].x = xv0 };
            
如果同一个成员被初始化多次，它将从最后一次初始化中取值。如果任何这样的覆盖初始化有副作用，副作用发生与否是非指定的。目前，gcc会舍弃它们并产生一个警告。

摘自:  http://www.kerneltravel.net/newbie/gcc_man.html#5.22

 

10、 结构体的初试化

gcc开始采用ANSI C的struct结构体的初始化形式：

static struct some_structure = {

.field1 = value,

.field2 = value,

..

};

老版本：非标准的初试化形式

static struct some_structure = {

field1: value,

field2: value,

..

};


>

参考:
1 http://blog.csdn.net/adaptiver/article/details/7494081
2 http://blog.csdn.net/zhsxcn/archive/2008/02/27/2125211.aspx
3 http://hi.baidu.com/hufeifeihu/blog/item/4991b6fb0ea16e136c22eb39.html