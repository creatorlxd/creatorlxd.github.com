---
layout: post
title: string 与 wstring 的相互转换
categories: [Utility,string]
tags: 计算机科学,字符串
---

# string 与 wstring 的相互转换

&emsp;&emsp;直接在开发`Space`游戏引擎时，就被这个问题困扰了很长时间，一直没有解决。知道最近无聊上google上翻，翻了好久竟然在C++文档（关于C的标准库）里找到了解决办法。

&emsp;&emsp;有这样两个函数:`mbstowcs_s`、`wcstombs_s`可以用来在字符串做**普通字符(char)**和**宽字符(wchar_t)**之间的转换。  
函数声明如下：
```c++
errno_t mbstowcs_s(  
   size_t *pReturnValue,  
   wchar_t *wcstr,  
   size_t sizeInWords,  
   const char *mbstr,  
   size_t count   
); 

errno_t wcstombs_s(
   size_t *pReturnValue,
   char *mbstr,
   size_t sizeInBytes,
   const wchar_t *wcstr,
   size_t count 
);
```
> 这里介绍的是**安全版本**，也可使用**不安全版本**

&emsp;&emsp;所以，就可以用这两个库函数来做转换了。  
代码如下（出自我的**Space**游戏引擎）：
```c++
std::wstring SpaceGameEngine::StringToWString(const std::string& str)
{
	std::wstring wstr;
	size_t size;
	wstr.resize(str.length());
	mbstowcs_s(&size, &wstr[0], wstr.size() + 1, str.c_str(), str.size());
	return wstr;
}

std::string SpaceGameEngine::WStringToString(const std::wstring & wstr)
{
	std::string str;
	size_t size;
	str.resize(wstr.length());
	wcstombs_s(&size, &str[0], str.size() + 1, wstr.c_str(), wstr.size());
	return str;
}
```

&emsp;&emsp;自我感觉比网上其他用系统API的方法要好一些吧。