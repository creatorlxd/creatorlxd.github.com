---
layout: post
title: 解析c++ range实现（三），range
categories: [c++,range]
tags: c++,range
keywords: c++,range
---
# Range
&emsp;&emsp;整个Range-v3的实现文件夹(`range/v3`)里的东西特别多，要想较好地去理解整个实现，就必须找一个好的切入点。之前，我已阅读完了整个项目所使用的模板元编程的库的部分，那是整个项目所使用的基础工具。现在，我决定从整个库的最核心的概念`range`开始看，如果中途遇到其他的辅助的功能实现再加以解释，而不是专门地去看那些实现，这样主次分明，能大大地减少我们阅读整个项目所需的时间。

&emsp;&emsp;`range`是整个库的基本概念，库的大部分功能都是围绕这个概念而展开的。在文件`range/concepts.hpp`中，定义了对于不同种类`range`的各种特征。下面，我将围绕着这些不同种类的`range`展开介绍。
* `range` : 只要求能适用于`ranges::begin`和`ranges::end`(即有成员函数`::begin()`和`::end()`或是一个普通的数组)，不要求返回类型相同或是可比较
* `borrowed_range` : 一种特殊的`range`，它所储存的内容的所有权不是由其自身来管理的，例如`std::string_view`就可以说是一个`borrowed_range`。是否是`borrowed_range`是由`enable_borrowed_range`这个模板变量决定的，当我们实现`range`时，我们需要设定这个模板变量，其默认值是`false`
* `safe_range` : `borrowed_range`的别名，因为不用自己管理资源，所以安全
* `output_range` : 一种特殊的`range`，对其使用`ranges::begin`返回的`iterator`是一个`output_iterator`，具体什么是`output_iterator`，我们下次再去深入地看`iterator`相关的内容
* `input_range` : 一种特殊的`range`，对其使用`ranges::begin`返回的`iterator`是一个`input_iterator`
* `forward_range` : 一种特殊的`input_range`，对其使用`ranges::begin`返回的`iterator`是一个`forward_iterator`
* `bidirectional_range` : 一种特殊的`forward_range`，对其使用`ranges::begin`返回的`iterator`是一个`bidirectional_iterator`
* `random_access_range` : 一种特殊的`bidirectional_range`，对其使用`ranges::begin`返回的`iterator`是一个`random_access_iterator`
* `contiguous_range_` : 内存连续的`random_access_range`，对其使用`ranges::data`(如果是数组，返回地址，否则调用其`::data()`)所求得的结果与其迭代器解引用后得到的类型的指针形式相同，且其迭代器是一个`contiguous_iterator`
* `common_range` : 一种特殊的`range`，其只要求其的`iterator`类型同`sentinel`类型是相同的(即`begin()`和`end()`的返回值的类型是相同的)
* `bounded_range` : 是`common_range`的别名，既然不需要特殊的`sentinel`，那么必然是有限的、可确定的、有界的`range`
* `sized_range` : 一种特殊的`range`，其要求`disable_sized_range`这个模板变量的值为`false`(同`enable_borrowed_range`一样，其默认值也为`false`，是根据不同的`range`实际类型去特化修改其值的)，且其能被`range::size`函数所调用(这个函数对于`range`是数组的情况，返回数组的大小，否则，调用其`::size()`函数)，得到的结果得是整数形式的。
* `viewable_range` : 一种特殊的`range`，其要求要么其`enable_borrowed_range`的值为`true`；要么其得是`semiregular`的(即可拷贝、可默认构造的)，同时其`enable_view`这个模板变量的值得是`true`(这个值同`disable_sized_range`一样，其默认值为判断类型`T`是否是`view_base`的子类，也可以自己特化来设定其值，比如`std::string_view`的`enable_view`就被设定为`true`)
* 各种`range_tag` : 用来判断`range`类型的用作标志用途的一些类型，有：
	* `range_tag`
	* `input_range_tag : range_tag`
	* `forward_range_tag : input_range_tag`
	* `bidirectional_range_tag : forward_range_tag`
	* `random_access_range_tag : bidirectional_range_tag`
	* `contiguous_range_tag : random_access_range_tag`
* `range_tag_of` : 根据各种`range`的concept，来判断所给类型是哪种`range`，返回不同的`range_tag`的类型
* `common_range_tag : range_tag` : 不归`range_tag_of`管的一种`range_tag`，用`common_range_tag_of`来判断
* `sized_range_tag : range_tag` : 不归`range_tag_of`管的一种`range_tag`，用`sized_range_tag_of`来判断

&emsp;&emsp;各种`range`的concept基本就这些，我们不难发现，各种`range`都离不开各种`iterator`的concept，所以下次我就将阅览一下各种`iterator`的concept。