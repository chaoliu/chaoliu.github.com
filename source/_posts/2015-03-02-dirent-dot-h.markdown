---
layout: post
title: "dirent.h"
date: 2015-03-02 12:12:36 +0800
comments: true
categories: linux c
---

###alphasort

alphasort（依字母顺序排序目录结构）
相关函数 <code>scandir，qsort</code>
表头文件 <code>#include<dirent.h></code>
定义函数 
``` C
int alphasort(const struct dirent **a,const struct dirent **b);
```
函数说明 alphasort()为scandir()最后调用qsort()函数时传给qsort()作w为判断的函数，详细说明请参考scandir()及qsort()。
返回值 参考qsort()。

<!-- more -->

示例

``` C
#include <stdio.h>
#include <stdlib.h>
#include <dirent.h>
int main()
{
    struct dirent **namelist;
    int i,total;

    total = scandir("/", &namelist , 0, alphasort);
    if(total <0){
        perror("scandir");
    } else {
        for(i=0;i<total;i++)
            printf("%s\n", namelist[i]->d_name);
        printf("total = %d\n",total);
    }
}
```
###dirent 结构体

```
struct dirent
{
    long d_ino; /* inode number 索引节点号 */
    off_t d_off; /* offset to this dirent 在目录文件中的偏移 */
    unsigned short d_reclen; /* length of this d_name 文件名长 */
    unsigned char d_type; /* the type of d_name 文件类型 */
    char d_name [NAME_MAX+1]; /* file name (null-terminated) 文件名，最长256字符 */
}

```
###php

在php中为了保持兼容性，当<code>scandir</code>和<code>alphasort</code>不存在时，将会使用自定义的版本，具体可以参考php-src/main/php_scandir.c