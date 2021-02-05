---
layout: wiki
title: const对左值引用类型的影响
categories: [c++]
description: const对左值引用类型的影响
keywords: c++
---

# const对左值引用类型的影响
`const`同`&&`一样，对左值引用类型不会产生影响。
例：
```c++
T const => T //T 是左值引用类型
T && => T //T 是左值引用类型
```