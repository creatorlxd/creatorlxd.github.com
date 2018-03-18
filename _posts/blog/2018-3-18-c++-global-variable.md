---
layout: post
title: C++全局变量
categories: [c++]
tags: c++
---

# C++全局变量

## 概念
&emsp;&emsp;直接使用c++的全局变量，是有一定的风险的。因为你并不能清楚地容易地控制全局变量的构造和析构的顺序，所以就会很容易出问题。比如说，你有一个全局的内存分配器，又有另一个全局变量需要使用这个分配器。我们在这种情况下，通常会使用`local static`方法，即使用函数内的`static`对象来代替裸的全局变量，来解决构造顺序的问题。当我们需要这个全局变量时，就会调用相对应的函数，然后我们所需要的全局变量就会被构造，从而避免了构造顺序上的问题。然而`local static`不能解决析构顺序的问题，如果内存分配器在其他全局变量之前析构了，那么程序就会崩掉。  
&emsp;&emsp;所以为了优雅地解决这样的一系列的问题。我采用了一种`GlobalVariableManager`来解决这类全局变量构造和析构顺序的问题。我们先使用一个`GlobalVariableTagClass`基类来标记出哪些是全局变量，并且有一个`Release`虚方法。然后有一个`GlobalVariableManager`来用一个栈来去管理所有的全局变量，并负责在最后程序结束时依照栈的顺序来析构所有的全局变量（调用它们的`Release`方法）。而实际的全局变量的创建则是使用一个继承自`GlobalVariableTagClass`的类模板`GlobalVariable`，并结合`local static`来实现的。这样，当`GlobalVariable`构造时，它会把自己的指针插入到Manager的栈中，从而使整个程序的正确的构造顺序能被记录下来，而在析构时，Manager就能按照与构造顺序相反的顺序来析构了。`GlobalVariableManager`保证只析构一次，并且会在第一个`GlobalVariable<T>`析构前调用所有`GlobalVariable`的`Release`方法。

## 代码
&emsp;&emsp;这里放一下SpaceGameEngine中的代码：

```c++
//GlobalVariable.h
/*
Copyright 2018 creatorlxd

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
*/
#pragma once
#include "Common/MemoryManager/Include/AllocatorForSTL.hpp"

namespace SpaceGameEngine
{
	class GlobalVariableTagClass
	{
	public:
		virtual ~GlobalVariableTagClass();
	
		friend class GlobalVariableManager;
	private:
		virtual void Release();
	};

	class GlobalVariableManager
	{
	public:
		friend GlobalVariableManager& GetGlobalVariableManager();
		~GlobalVariableManager();

		template<typename T,typename AllocatorInterface>
		friend class GlobalVariable;
	private:
		GlobalVariableManager();
		void Insert(GlobalVariableTagClass* ptr);
		void Release();
	private:
		std::stack<GlobalVariableTagClass*> m_Content;
		bool m_IfReleased;
	};

	GlobalVariableManager& GetGlobalVariableManager();

	template<typename T, typename AllocatorInterface = MemoryManagerAllocatorInterface>
	class GlobalVariable :public GlobalVariableTagClass
	{
	public:
		template<typename... Arg>
		GlobalVariable(Arg&&... arg)
		{
			m_pContent = AllocatorInterface::New<T>(std::forward<Arg>(arg)...);
			GetGlobalVariableManager().Insert(this);
		}

		T& Get()
		{
			return *m_pContent;
		}

		~GlobalVariable()
		{
			GetGlobalVariableManager().Release();
		}
	private:
		void Release() override
		{
			AllocatorInterface::Delete(m_pContent);
		}
	private:
		T * m_pContent;
	};
}
```

```c++
//GlobalVariable.cpp
/*
Copyright 2018 creatorlxd

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
*/
#include "stdafx.h"
#include "..\Include\GlobalVariable.h"

SpaceGameEngine::GlobalVariableTagClass::~GlobalVariableTagClass()
{

}

void SpaceGameEngine::GlobalVariableTagClass::Release()
{

}

SpaceGameEngine::GlobalVariableManager::~GlobalVariableManager()
{
	Release();
}

SpaceGameEngine::GlobalVariableManager::GlobalVariableManager()
{
	m_IfReleased = false;
}

void SpaceGameEngine::GlobalVariableManager::Insert(GlobalVariableTagClass * ptr)
{
	m_Content.push(ptr);
}

void SpaceGameEngine::GlobalVariableManager::Release()
{
	if (!m_IfReleased)
	{
		while (!m_Content.empty())
		{
			m_Content.top()->Release();
			m_Content.pop();
		}
		m_IfReleased = true;
	}
}

SpaceGameEngine::GlobalVariableManager & SpaceGameEngine::GetGlobalVariableManager()
{
	static GlobalVariableManager g_GlobalVariableManager;
	return g_GlobalVariableManager;
}
```

&emsp;&emsp;使用示例：
```c++
struct Test
{
	int i;
};

Test& GetTest()	//local static
{
	static GlobalVariable<Test> g_Test;
	return g_Test.Get();
}
```