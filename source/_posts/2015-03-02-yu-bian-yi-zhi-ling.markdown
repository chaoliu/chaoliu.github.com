---
layout: post
title: "C中#if 0 和#if 1"
date: 2015-03-02 13:02:52 +0800
comments: true
categories: 

---

当注释掉大块代码时，使用<code>#if 0</code>比使用
<code>/\*\*/</code>
要好，因为用
<code>/\*\*/</code>
做大段的注释要防止被注释掉的代码中有嵌套的
<code>/\*\*/</code>,这会导致注释掉的代码区域不是你想要的范围， 当被注释掉的代码很大时容易出现这种情况，特别是过一段时间后又修改该处代码时更是如此。

<!-- more -->
```
#if 1
###执行
#endif

#if 0
###不执行
#endif 

```
编译阶段的注释语句

###php

``` C php-src/zend/zend_string.h
static zend_always_inline zend_string *zend_string_alloc(size_t len, int persistent)
{
    zend_string *ret = (zend_string *)pemalloc(ZEND_MM_ALIGNED_SIZE(_STR_HEADER_SIZE + len + 1), persistent);

    GC_REFCOUNT(ret) = 1;
#if 1
    /* optimized single assignment */
    GC_TYPE_INFO(ret) = IS_STRING | ((persistent ? IS_STR_PERSISTENT : 0) << 8);
#else
    GC_TYPE(ret) = IS_STRING;
    GC_FLAGS(ret) = (persistent ? IS_STR_PERSISTENT : 0);
    GC_INFO(ret) = 0;
#endif
    ret->h = 0;
    ret->len = len;
    return ret;
}
```
