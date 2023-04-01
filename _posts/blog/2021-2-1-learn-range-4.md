---
layout: post
title: 解析c++ range实现（四），iterator
categories: [c++,range]
tags: c++,range
keywords: c++,range
---
# Iterator
&emsp;&emsp;在上次我们初步得看完整个`range`的concept后，我们不难发现，基本上大部分的`range`定义都依赖于对其迭代器的定义，所以这次我们就来看一下`iterator/concepts.hpp`中的各种迭代器的concept定义。
&emsp;&emsp;但是在直接看concept之前，我们得先看一下`iterator/traits.hpp`中的各种traits。

## Traits
* 各种`iterator_tag` :
	* `input_iterator_tag`
	* `forward_iterator_tag : input_iterator_tag`
	* `bidirectional_iterator_tag : forward_iterator_tag`
	* `random_access_iterator_tag : bidirectional_iterator_tag`
	* `contiguous_iterator_tag : random_access_iterator_tag`
* `iter_rvalue_reference_t` : 获取迭代器解引用后的类型的右值形式
* `iter_common_reference_t` : 获取一种迭代器解引用后的类型与迭代器的`value_type`的`common_type`(两者都能被转化到的一个类型)的引用形式
* `iter_difference_t` : 获取迭代器的`difference_type`，即用来表示迭代器距离的类型
* `iter_value_t` : 获取迭代器的`value_type`
* `iter_reference_t` : 获取迭代器解引用后的类型的引用形式

## Concept

* `detail::iter_concept_t` : 获取迭代器的类别信息，其具体按照以下顺序排布的过程来获取这一信息:
	* 如果迭代器是指针，返回`contiguous_iterator_tag`
	* 如果有`iter_traits_t<I>::iterator_concept`，则返回这个`iterator_concept`
	* 如果有`iter_traits_t<I>::iterator_category`，则返回这个`iterator_category`
	* 如果没有特化`std::iterator_traits<I>`，返回`random_access_iterator_tag`
* `indirectly_readable`	: 间接可读的，要求迭代器的`const`形式的解引用的结果的类型与`iter_reference_t<I>`相同；`const`形式的迭代器解引用的结果的右值形式与`iter_rvalue_reference_t<I>`相同。简单地来说，就是要求迭代器能解引用且其的解引用的结果的类型不会因为迭代器是不是`const`的而产生变化
* `indirectly_writable` : 间接可写的，要求迭代器的左值和右值引用形式都可以支持接受类型`T`的赋值操作(这里我又在源码里学到一个新知识点：`const`同`&&`一样都不会对左值引用类型产生影响)
* `detail::integer_like_` : 要求类型是一种数值类型
* `detail::signed_integer_like_` : 要求类型是一种有符号数值类型
* `weakly_incrementable` : 弱可增长的，要求迭代器是一个`semiregular`(可默认构造，可拷贝)，并且其要支持`++i`、`i++`这两种操作，其中`++i`的返回类型得是`I&`，同时这个迭代器类型`I`得有`difference_type`，且其`difference_type`得符合`detail::signed_integer_like_`
* `incrementable` : 可增长的，首先要求迭代器是一个`regular`(`semiregular`加上可做等同性比较)，并且其得符合`weakly_incrementable`，且要求`i++`的返回类型是`I`(个人认为`const I`在语义上更为严谨，可能是历史遗留问题吧)
* `input_or_output_iterator` : 可以输入或输出的迭代器，要求迭代器可以解引用(支持`operator*()`)，同时其也得满足`weakly_incrementable`
* `sentinel_for` : 接受一个待检验的守卫类型`S`和一个迭代器类型`I`，其要求守卫类型`S`是一个`semiregular`，`I`是一个`input_or_output_iterator`，同时`S`和`I`是满足弱相等的
* `disable_sized_sentinel<S,I>` : 类似`range`中的`disable_sized_range`，表示这个守卫类型是不是不能求大小的，其默认值为`false`
* `sized_sentinel_for` : 接受一个待检验的守卫类型`S`和一个迭代器类型`I`，其要求这两者满足`sentinel_for<S,I>`，同时`disable_sized_sentinel<S,I>`的值要为`false`，支持`s - i`和`i - s`这两种操作，且这两种操作的返回值的类型要与`iter_difference_t<I>`(即`I::difference_type`)相同
* `output_iterator` : 要求迭代器是`input_or_output_iterator`的，同时满足`indirectly_writable<Out, T>`，且支持`*o++ = (T &&) t`操作
* `input_iterator` : 要求迭代器是`input_or_output_iterator`的，同时满足`indirectly_readable<Out, T>`，且对其使用`detail::iter_concept_t`求得的tag是`input_iterator_tag`
* `forward_iterator` :