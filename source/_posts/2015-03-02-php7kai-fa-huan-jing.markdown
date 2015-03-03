---
layout: post
title: " php7开发环境"
date: 2015-03-02 15:53:27 +0800
comments: true
categories: 
---


###required： 10.10.2  xcode installed

> $ ./buildconf

```
buildconf: checking installation...
buildconf: autoconf not found.
           You need autoconf version 2.59 or newer installed
           to build PHP from Git.
make: *** [buildmk.stamp] Error 1

```
<!-- more -->
> $ brew install autoconf

```
==> Downloading https://homebrew.bintray.com/bottles/autoconf-2.69.yosemite.bottle.1.tar.gz
######################################################################## 100.0%
==> Pouring autoconf-2.69.yosemite.bottle.1.tar.gz
/usr/local/Cellar/autoconf/2.69: 70 files, 3.1M
```
> $ ./buildconf

> $ ./configure --enable-debug --disable-all


```
checking for bison... bison -y
checking for bison version... invalid
configure: WARNING: This bison version is not supported for regeneration of the Zend/PHP parsers (found: 2.3, min: 204, excluded: ).
checking for re2c... re2c
checking for re2c version... 0.13.7.5 (ok)
configure: error: bison is required to build PHP/Zend when building a GIT checkout!
```
>$ wget http://mirror.hust.edu.cn/gnu/bison/bison-3.0.4.tar.gz
>
>$ ./configure
>
>$ make && make install

```
before
$ bison -V                                                                             
bison (GNU Bison) 2.3
Written by Robert Corbett and Richard Stallman.

Copyright (C) 2006 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

after
$ bison -V                                                                                      *[master] 
bison (GNU Bison) 3.0.4
Written by Robert Corbett and Richard Stallman.

Copyright (C) 2015 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

```

>$ ./configure --enable-debug --disable-all

```
Thank you for using PHP.

config.status: creating php7.spec
config.status: creating main/build-defs.h
config.status: creating scripts/phpize
config.status: creating scripts/man1/phpize.1
config.status: creating scripts/php-config
config.status: creating scripts/man1/php-config.1
config.status: creating sapi/cli/php.1
config.status: creating sapi/cgi/php-cgi.1
config.status: creating main/php_config.h
config.status: executing default commands

```

> $make

```
Build complete.
Don't forget to run 'make test'.

```

