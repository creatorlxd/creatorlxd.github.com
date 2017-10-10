---
layout: post
title: priority_queue 优先队列
categories: STL,c++
description: 有关priotity_queue 的介绍
keywords: STL,c++,priority_queue
---

# priority_queue 优先队列

本文主要是介绍`priority_queue`的使用方法。

`priority_queue`优先队列，一种有异于传统的队列的FIFO结构

看一下它的模板：
```c++
template<
    class T,
    class Container = std::vector<T>,
    class Compare = std::less<typename Container::value_type> > class priority_queue;
```

&emsp;&emsp;第一个模板参数自然就是你所要储存的类型了

&emsp;&emsp;第二个模板参数是指你所要使用的容器。  
我们知道：`queue`、`stack`都不是类似于`vector`的实际的容器，而是使用其他容器进行操作的模板。  
&emsp;&emsp;这里的容器可以用`vector`，也可以用`forward_list`（没试过）。
&emsp;&emsp;第三个参数尤为重要，直接关系到元素出栈的顺序。  
&emsp;&emsp;**选择 `less<T>`则最大元素出队列  
&emsp;&emsp;选择`great<T>`则最小元素出队列**

&emsp;&emsp;剩下的跟一般的`queue`操作差别不大，一样的`push`、`front`、`pop`......

&emsp;&emsp;然而值得注意的是可以自定义过一个**仿函数**用于比较操作。

### 例子(使用了C++17)：

```c++
#include <functional>
#include <queue>
#include <vector>
#include <iostream>
 
template<typename T> void print_queue(T& q) {
    while(!q.empty()) {
        std::cout << q.top() << " ";
        q.pop();
    }
    std::cout << '\n';
}
 
int main() {
    std::priority_queue<int> q;
 
    for(int n : {1,8,5,6,3,4,0,9,7,2})
        q.push(n);
 
    print_queue(q);
 
    std::priority_queue<int, std::vector<int>, std::greater<int> > q2;
 
    for(int n : {1,8,5,6,3,4,0,9,7,2})
        q2.push(n);
 
    print_queue(q2);
 
    // 用 lambda 比较元素。
    auto cmp = [](int left, int right) { return (left ^ 1) < (right ^ 1);};
    std::priority_queue<int, std::vector<int>, decltype(cmp)> q3(cmp);
 
    for(int n : {1,8,5,6,3,4,0,9,7,2})
        q3.push(n);
 
    print_queue(q3);
 
}
```

### 输出：

```
9 8 7 6 5 4 3 2 1 0   
0 1 2 3 4 5 6 7 8 9   
8 9 6 7 4 5 2 3 0 1  
```