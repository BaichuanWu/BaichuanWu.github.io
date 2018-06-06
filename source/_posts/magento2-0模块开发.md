---
title: magento2.0模块开发
date: 2016-07-26 10:37:27
categories: programming
tags: php 
---

#### 前言

<!-- more -->

magento2.0之后是一个MVVM 框架系统，（[MVC 等系统区别](http://www.ruanyifeng.com/blog/2015/02/mvcmvp_mvvm.html)）。

如果你要创建一个新的url：创建一个module 告诉magento （1）哪一个controller用来处理这个url （2）哪一个block需要被加载到系统中。

创建插件时，未达到预期，可能是cache 的问题，清空下cache会有帮助（如下）：

```
rm -rf /path/to/magento/var/generation/*
rm -rf /path/to/magento/var/cache/* 
```



#### 模块开发

主要参考了[这篇文章](http://alanstorm.com/magento_2_mvvm_mvc)，或者说蹩脚翻译吧😂。



