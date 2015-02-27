---
layout: post
title: "Octopress个人博客搭建"
date: 2015-02-27 14:42:47 +0800
comments: true
categories: 效率提升
---

> 文章内容主要记录Octopress个人博客搭建过程，部分内容参考自互联网

<!-- more -->

####1.首先你可以登录Octopress主页一下主要功能

http://octopress.org

并且确保mac安装了以下工具：

git（应该是mac自带的）

ruby 1.9.3以上版本（可以在终端中用ruby --version查看版本是否满足）

缺少的请单独下载安装，这里就不具体讲了

 

####2.开始安装，mac上基本自带安装了git，所以直接打开终端，输入：

git clone git://github.com/imathis/octopress.git octopress

之后git将会从github克隆下Octopress项目文件到本地的octopress目录，本地目录可以根据需要更改

cd octopress

进入Octopress项目根目录

 

####3.安装相关工具


>sudo gem install bundler

这过程可能会比较长,众所周知的原因，在国内已经无法使用gem的默认源了, 为gem更换淘宝的源

``` sh
gem sources -l
gem sources --remove https://rubygems.org/
gem sources -a http://ruby.taobao.org/
gem sources -l

```

好的，安装完成进入下一步 

>bundle install

开始安装具体的工具，这里没有碰到什么问题

再接使用rake工具安装默认的主题和配置

>rake install

####4.接下来开始部署博客

官方推荐了3种部署方式：

1-github，部署允许自定义域名，免费，好处是多人开发更方面，坏处是文件随时可以被任何人拉下来。

2-heroku，部署允许自定义域名，免费，并且是私有的，看样子这个比较适合我，后面的过程就用这个方法。

3-rsync，建议用来部署有自己服务器的个人博客。



本文主要介绍第一种方式：

首先设置github

>rake setup_github_pages\[repo\]

repo填写github 上的 clone URL, 例如：https://github.com/chaoliu/chaoliu.github.com.git

接着可以修改_config.yml, 进行一些配置

```
# ----------------------- #
#      Main Configs       #
# ----------------------- #

url: http://chaoliu.github.io
title: A true master is an eternal student.
subtitle: 
author: ChaoLiu
simple_search: https://www.google.com/search
description:

...

```

现在你可以通过
>new_post[title]           -- begin a new post in source/_posts

写文章了， 如果你觉得默认主题不是很好看，可以通过修改sass目录下的样式表进行样式调整
