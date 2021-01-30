---
layout: post
title: 解析c++ range实现（二），meta
categories: [c++,range]
tags: c++,range
keywords: c++,range
---
# Meta
&emsp;&emsp;`meta`里的东西很多，没法一一地去解释，所以我先选取一些使用得比较多的内容来加以分析。

## namespace
&emsp;&emsp;`meta`主要有三个部分的名空间`meta`,`meta::detail`,`meta::lazy`，其中暴露出去的主要是`meta`名空间，而实现则放在`detail`中，`lazy`名空间放的都是特殊的、适用于lazy情况下的功能实现。我将主要分析`meta`中的内容。`lazy`中的功能多半是`meta`中的功能的`defer`修饰后的版本。

### meta部分

* `nil_` : 空类型，主要用于后续的sfinae
* `_t` : 用于获取trait类型的`::type`
* `_v` : 用于获取`std::integral_constant`这样的编译期常量的值
* `bool_ int_ ...` : 用于创建对应类型`std::integral_constant`的模板
* `inc dec plus minus ...` : 处理`std::integral_constant`的编译期类型函数，实际是编译期计算的工具。其返回的也是`std::integral_constant`。
* `template <META_TYPE_CONSTRAINT(invocable) Fn, typename... Args> invoke` : 调用`Fn::template invoke<Args...>`，应该是为了调用编译期类型函数。
* `template <typename T> struct id` : 一种特殊的trait，其总是返回其参数`T`，其同时也是可调用的一个编译期类型函数(其内含`::invoke`)，调用的结果也是其参数`T`
* `id_t` : 其参数`T`的别称，用在非推导的环境下
* `is_trait` : 利用sfinae实现的，通过判断类型具不具有`::type`来判断其是不是trait，返回`std::integral_constant`
* `is_callable` : 利用sfinae实现的，通过判断类型具不具有`::invoke`来判断其是不是编译期类型函数，返回`std::integral_constant`
* `make_integer_sequence`和`make_index_sequence`的实现部分，是为了兼容c++14以下的c++版本。
	* `detail::indices_strategy_` : 表示索引策略的枚举类型，有`done`,`repeat`,`recurse`三种取值
	* `constexpr detail::indices_strategy_ detail::strategy_(std::size_t cur, std::size_t end)` : 根据给定的参数，返回索引策略`indices_stragtegy_`，其划分大致为下：
		* `done` : `cur >= end`
		* `repeat` : `cur*2 <= end`
		* `recurse` : `other`
	* `constexpr std::size_t detail::range_distance_(T begin, T end)` : 在`begin<=end`的前提下，直接将`end-begin`转化为`size_t`
	* `template <std::size_t End, typename State, indices_strategy_ Status_> struct detail::make_indices_` : 用于创建一个已有`State`，目标为`End`的`index_sequence`即(`[0,1,2,...,N-1]`的序列，这里`N=End`)，而`Status_`则是用来判断如何构造这个序列，其值可用前文的`strategy_`来取得。`repeat`代表将当前`State`整个叠加到自身，而`recurse`则是加上距离目标序列缺失的部分内容。这样，整个构造过程的复杂度就是`O(log(N))`的。其结果放在其内`::type`中。
	* `template <typename T, T, typename> struct detail::coerce_indices_` : 其原始意义不是很明确，但是其的一个偏特化实现`template <typename T, T Offset, std::size_t... Values> struct coerce_indices_<T, Offset, index_sequence<Values...>>` 实现了将`index_sequence`的所有值都加上一个`Offset`的功能。其结果也放在其内的`::type`中。
	* `integer_sequence`: 类型为`T`的编译期的整数序列
	* `index_sequence`: 类型为`std::size_t`的编译期序列，实际上是`integer_sequence`的特殊版本
	* `template <typename, typename> struct detail::concat_indices_` : 其特化的版本表明，其主要功能是将两个`index_sequence`合并，实际上就是第二部分的`index_sequence`每个元素都加上前一个的`size`。
	* `make_integer_sequence` : 生成一个元素类型为T的`[0,1,2,...,N-1]`的`integer_sequence`。
	* `make_index_sequence` : 生成一个`size_t`的`[0,1,2,...,N-1]`的`index_sequence`
* `defer` : 给定一个模板`C`，返回`id<C<Ts...>>`类型
* `defer_i` : 给定一个接受若干相同类型模板参数的模板`D`，返回`id<D<Is...>>`
* `defer_trait` : 给定一个trait和其模板参数，使用`defer`、`detail::defer_`和`detail::_t_t`来延迟获取其结果`::type`
* `defer_trait` : 给定一个接受`T`类型模板参数的trait，使用`defer`、`detail::defer_i_`和`detail::_t_t`来延迟获取其结果`::type`
* `compose` : 将多个编译器类型函数复合成`F1(F2(...(Fn(Ts...)))`的形式
* `quote` : 将一个模板包装成一个带有`::invoke`的编译期类型函数
* `quote_i` : 将一个接受`T`类型模板参数的模板包装成一个带有`::invoke`的编译期类型函数
* `quote_trait` : 将一个trait模板类(带有`::type`)包装成一个带有`::invoke`的编译期类型函数
* `quote_trait_i` : 将一个接受`T`类型模板参数的trait模板类(带有`::type`)包装成一个带有`::invoke`的编译期类型函数
* `bind_front` : 绑定一个编译期类型函数的前半部分参数
* `bind_back` : 绑定一个编译期类型函数的后半部分参数
* `apply` : 给定一个带有`::invoke`的编译期类型函数`Fn`，和一个类型`L`，其会根据几种情况去利用`L`中的类型信息去调用`Fn`(`invoke`)
	* `L=Ret(Args...)` : `lazy::invoke<Fn,Ret,Args...>`
	* `L=T<Ts...>` : `lazy::invoke<Fn,Ts...>`
	* `L=std::integer_sequence<T, Is...>` : `lazy::invoke<Fn,std::integral_constant<T, Is>...>`
* `curry` : 给定一个带有`::invoke`的编译期类型函数`Fn`和一个返回值为列表类型(形如`list<Args...>`)的带有`::invoke`的编译期类型函数`Q`，返回一个以`Q`调用后返回的列表类型为参数去调用`Fn`的带有`::invoke`的编译期类型函数
* `uncurry` : 给定一个带有`::invoke`的编译期类型函数`Fn`，返回一个接受列表类型(形如`list<Args...>`)的参数去解包调用`Fn`的带有`::invoke`的编译期类型函数
* `flip` : 将模板参数列表的第一项和第二项交换位置，然后去调用`Fn`(`Fn`是带有`::invoke`的编译期类型函数)
* `template<typename Fn,typename Gs...> on` : 生成一个带有`::invoke`的编译期类型函数，其通过传递其自身的模板参数表给`compose<Gs...>`，将最终得到的模板参数表给`Fn`调用。
* `template<bool If,typename Then,typename Eles> conditional_t` : 通过编译期bool型常量，来确定是哪个类型
* `if_` : 接受`list<Args...>`作为参数的`conditional_t`
* `if_c` : 接受`list<bool_<If>,Args...>`作为参数的`conditional_t`(指定了if的结果)
* `fold` : 接受一个`list`、`State`和`Fn`，构造出一个包装在`defer`里的形如`Fn(Fn(...Fn(State,T0)...,TN-2),TN-1)`的类型
* `accumulate` : `fold`的别名
* `reverse_fold` : 接受一个`list`、`State`和`Fn`，构造出一个包装在`defer`里的形如`Fn(Fn(...Fn(State,TN-1)...,T1),T0)`的类型
* `npos` : 一个表示找不到结果的值(实际是`std::integral_constant`类型)
* `list` : 一个可以包裹任意多类型的模板类，其内含有`::type`(值为其自身)，和`::size()`(返回其含的模板参数的个数)
* `size` : 获取一个`list`的`size`，返回的是一个`std::integral_constant`
* `concat` : 将多个`list`合为一个`list`
* `join` : 将一个由多个`list`构成的`list`合并为一个`list`
* `transform` : 将一个`list<Args...>`和`Fn`构成的`list`作为参数，返回一个`list`，其中每个元素都将`list<Args...>`中对应元素传入`Fn`后得到的结果。亦可以传入两个`list<Args...>`，那么新的`list`的元素就是以两个`list<Args...>`中的对应元素同时传入`Fn`后得到的结果
* `repeat_n_c` : 返回一个含有`N`个`T`类型的`list`
* `repeat_n` : 功能与`repeat_n_c`相同，不过接受的`N`是`std::integral_constant`
* `at_c` : 返回一个`list`中下标为`N`的类型
* `at` ： 功能与`at_c`相同，不过接受的`N`是`std::integral_constant`
* `drop_c` : 返回一个新的`list`，其不含有原`list`的前`N`个元素
* `drop` : 功能与`drop_c`相同，不过接受的`N`是`std::integral_constant`
* `front` : 返回`list`的首元素
* `back` : 返回`list`的尾元素
* `push_front` : 将一些元素加到`list`的首部，返回这个新的`list`
* `pop_front` : 返回一个`list`，其是原`list`移去首元素得到的。
* `push_back` : 将一些元素加到`list`的末尾，返回这个新的`list`
* `empty` : 判断一个`list`是不是空的，返回一个`std::integral_constant`
* `pair` : 只含两个元素的`list`
* `first` : 返回`pair`的第一个元素
* `second` : 返回`pair`的第二个元素
* `find_index` : 返回`list`中第一个与`T`相同的元素的下标，其类型为`std::integral_constant`
* `reverse_find_index` : 返回`list`中最后一个与`T`相同的元素的下标，其类型为`std::integral_constant`
* `find` : 返回一个`list`，其是将原`list`中第一个`T`元素前的元素全部删去后的结果，如果原`list`没有`T`元素，则返回空`list`。
* `reverse_find` : 返回一个`list`，其是将原`list`中最后一个`T`元素前的元素全部删去后的结果，如果原`list`没有`T`元素，则返回空`list`。
* `find_if` : 返回一个`list`，其是将原`list`中将第一个传入`Fn`将返回`true`的元素前的元素全部删去后的结果，如果原`list`没有满足条件的元素，则返回空`list`。
* `reverse_find_if` : 返回一个`list`，其是将原`list`中将最后一个传入`Fn`将返回`true`的元素前的元素全部删去后的结果，如果原`list`没有满足条件的元素，则返回空`list`。
* `replace` : 返回一个`list`，其是原`list`中所有`T`元素都被替换成`U`后的结果
* `replace_if` : 返回一个`list`，其是原`list`中所有被传入`Fn`将返回`true`的元素都被替换成`U`后的结果
* `count` : 返回一个`list`中有多少个`T`元素，是`std::integral_constant`
* `count_if` : 返回一个`list`中有多少个元素被传入`Fn`将返回`true`，结果是`std::integral_constant`
* `filter` : 返回一个`list`，其是所有原`list`中所有被传入`Fn`将返回`true`的元素所构成的
* `for_each` : 将一个`list`的每个元素都传入`Fn`做调用(这个`Fn`不是编译期类型函数)
* `transpose` : 返回一个二维的`list`转置后的二维`list`
* `zip_with` : 给定一个二维`list`和一个编译期类型函数`Fn`，将这个二维`list`中的每一个`list`都当作其对应位置模板参数的取值，将这些取值按照一维`list`的长度，分次传给`Fn`，将这些结果放到要返回的新的`list`中。
* `zip` : `transpose`的别名
* `as_list` : 利用`apply`的规则，将一个类型`T`变成一个`list`
* `reverse` : 返回一个翻转后的`list`
* `not_fn` : 返回一个新的编译期类型函数，其将原函数的返回值取反
* `all_of` : 返回是不是`list`的所有值都满足调用`Fn`返回`true`，结果是`std::integral_constant`
* `any_of` : 返回是不是存在`list`的任意一个值满足调用`Fn`返回`true`，结果是`std::integral_constant`
* `none_of` : 返回是不是`list`的所有值都不满足调用`Fn`返回`true`，结果是`std::integral_constant`
* `in` : 返回`list`中有没有`T`元素，结果是`std::integral_constant`
* `inherit` : 返回一个继承了`list`中所有类型的类型
* `unique` : 根据原`list`，返回一个去除重复元素的`list`
* `partition` : 根据`list`中的元素被传入`Fn`后返回的是不是`true`，返回一个新的`pair<list,list>`，其中第一个`list`里全是能返回`true`的元素，第二个`list`中全是返回`false`的元素。
* `sort` : 利用`Fn`作为比较函数，对`list`做快排，返回排序后的`list`
* `lambda` : 编译期类型函数层面的lambda表达式
* `var` : 用于在`let`中定义编译期类型变量
* `let` : 可以在其中定义编译期类型变量的块表达式
* `cartesian_product` : 给定一个`list`的`list`，求这些`list`中的`list`的笛卡尔积
* `add_const_if_c` : 给定一个`bool`值，如果其为`true`，返回一个能将`T`类型变成`const T`的编译期类型函数
* `add_const_if` : 功能与`add_const_if_c`相同，不过接受的`bool If`是`std::integral_constant`
* `const_if_c` : 接受一个`bool If`和`T`类型，返回`add_const_if_c<If>::invoke<T>`
* `const_if` : 功能与`const_if_c`相同，不过接受的`bool If`是`std::integral_constant`
* `detail::atoi_` : 将字符转换成数字，返回`std::integral_constant`
* `detail::operator ""_z` : 利用`detail::atoi_`将形如`123_z`的表达式转化为`size_t`的`std::integral_constant`
* `std_vector,std_map,std_...` : 对STL模板容器的支持，为这些容器添加了编译期类型函数的形式