---
layout: post
title: C++inline&selectany
categories: [c++]
tags: c++
---

# C++`inline`与selectany的使用

## `inline`
&emsp;&emsp;`inline`是C++中比较基础的一个修饰符，它的功能就是内联。但是我不准备介绍`inline`发挥内联功能的使用方法，而是准备介绍一下它作为一种能使函数和对象在头文件定义的修饰符的用法。

&emsp;&emsp;众所周知，将函数或是全局变量的定义直接写在头文件中，是会报出`fatal error LNK1169: 找到一个或多个多重定义的符号`这样的错误的。然而，这个错误并不是由编译器报出的，而是由链接器报出的。因为我们在一个头文件中定义了某个函数或是变量后，一旦这个头文件被多个源文件包含，那么每个包含了这个头文件的源文件在被编译后所产生的`.obj`文件中都会有你在头文件中定义的函数或是变量的二进制符号。当链接器在链接时，就会发现有多个重复定义的符号，于是就会报错。而`C++`的`inline`关键字就可以解决这个问题。

&emsp;&emsp;目前，C++的标准是这样定义的:
* 对于不是外部连接的`inline`变量和函数，其可在不同的翻译单元中拥有不重复且等同的定义，而当某个翻译单元的代码访问某个`inline`变量或函数时，其所访问到(也只能访问到)的是其所在翻译单元的实现。
* 对于是外部连接的`inline`变量和函数(非`static`的)，其在每个翻译单元中均拥有相同地址。
* 在内联函数中，
    1. 所有函数定义中的函数局部静态对象在所有翻译单元间共享(它们都指代相同的定义于某一个翻译单元中的对象)。
    2. 所有函数定义中所定义的类型亦在所有翻译单元中相同。
* 命名空间作用域的内联`const`变量默认具有外部连接，这点不同于非`inline`非`volatile`的有`const`限定的变量。

&emsp;&emsp; 测试代码如下:
```c++
//test2.h
#include <iostream>

#define PRINT_ADDR_VALUE(x)\
	std::cout<<&x<<'\t'<<x<<std::endl;

inline int test()
{
	static int cot=0;
	return cot++;
}

inline int i1=0;
extern inline int i2=0;
inline const int ci1=test();
extern inline const int ci2=test();
inline static int si1=0;

void print2();

//test2.cpp
#include "test2.h"

void print2()
{
	PRINT_ADDR_VALUE(i1);
	PRINT_ADDR_VALUE(i2);
	PRINT_ADDR_VALUE(ci1);
	PRINT_ADDR_VALUE(ci2);
	PRINT_ADDR_VALUE(si1);
}

//test.cpp
#include <iostream>
#include "test2.h"
using namespace std;

void print()
{
	PRINT_ADDR_VALUE(i1);
	PRINT_ADDR_VALUE(i2);
	PRINT_ADDR_VALUE(ci1);
	PRINT_ADDR_VALUE(ci2);
	PRINT_ADDR_VALUE(si1);
}
int main()
{
	print();
	std::cout<<"-----------------------"<<std::endl;
	print2();
	return 0;
}

/*
最后在macos下的输出结果为:
0x1098f90dc	0
0x1098f90d8	0
0x1098f90d4	0
0x1098f90d0	1
0x1098f90e8	0
-----------------------
0x1098f90dc	0
0x1098f90d8	0
0x1098f90d4	0
0x1098f90d0	1
0x1098f90ec	0

这里值得注意的是:虽然这里的const变量不在命名空间内，但编译器依旧将其当做了外部连接的变量来处理,当然这也有可能是一种优化的结果。
*/
```
### `inline`的用法

&emsp;&emsp;在头文件中，你的全局变量、类中的静态对象(注意：不可将声明与定义分离放在头文件中)或是函数（非类中方法）的定义前直接加上`inline`即可。但是对于全局的`const`常量或是类中的方法来说，`inline`可以看做是默认带有的，不用加(msvc)。

## `selectany`

&emsp;&emsp;前几天，我遇到这样一个问题：一个定义在头文件中的全局常量，竟然在不同的源文件中调用，值是不同的（被多次地初始化了，且不同的源文件链接向了不同值的同名全局常量）。后来才发现使用`__declspec(selectany)`去修饰这个全局常量，就可以解决这一问题（仅限msvc编译器)。同样是在链接时，一旦某个符号是由`__declspec(selectany)`修饰的，链接器就会从多个相同的该符号的定义中选取一个链接，并删去其他重复的定义。于是就避免了这样的问题的产生。

### `selectany`的用法

&emsp;&emsp;使用也很简单，就是在头文件的**全局作用域**中的全局变量的定义前加上`__declspec(selectany) extern`修饰符。但是，类中的静态变量要把声明与定义分离，并在定义前加上`__declspec(selectany)`(不加`extern`)。且函数不能用`__declspec(selectany)`修饰(也没有必要)。

## `inline`与`selectany`的比较

&emsp;&emsp;`selectany`与`inline`相比，功能上更强大，但是却是微软的私货，没法通用。在效果上来说，`inline`只是保证链接不报错，你自己还要保证每份同名的符号定义的内容是一样的，不然会出蜜汁BUG。但是`selectany`就从根本上解决了这个问题，因为它只会保留一份符号定义，并将其他同名的符号定义在链接时删去。
