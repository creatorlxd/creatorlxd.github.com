---
layout: post
title: 解析c++ range实现（一），初步介绍
categories: [c++,range]
tags: c++,range
keywords: c++,range
---
# Range的初步介绍
&emsp;&emsp;`Range`是`C++20`的重要组成部分之一，其提供了一系列有关容器（主要是序列）的抽象的便利的操作。鉴于我想要在我的游戏引擎中也实现一个这样的功能，我决定开始研究一下`Range`的源代码，目前我所阅读的源代码来自[Range-v3](https://github.com/ericniebler/range-v3)。

## Range的主要功能

### View
&emsp;&emsp;`View`是一种轻量级的表示序列的包装器，它可以很快地被复制或是创建，且其自身没有其所含元素的所有权。

&emsp;&emsp;`Range`库利用`View`来提供了一系列的可随意组合的，纯函数式的，lazy的对序列的操作。

### Action
&emsp;&emsp;`Action`是`View`的弱化版本，你可以利用一系列的`Action`相互组合，直接地在一个序列实体上进行操作。

## Range源码目录分析

&emsp;&emsp;项目的源码主要放在`include`文件夹下。其内含：

* concecpts (提供对于非cpp20的concept支持，一些要用的宏定义以及range所需的基础concept定义和traits)
* meta (一套模板元编程的库)
* range (主要功能的实现)
* std (和标准库相关的，起着一些兼容作用的工具) 

&emsp;&emsp;在这些目录中，std和concepts没有什么东西好说的，如果用到其中的内容再看就行，而对于这个`Range`库来说，元编程的部分是整个库的基石，所以我准备先从meta文件夹内的内容开始，逐步深入地分析解读整个`Range`库。