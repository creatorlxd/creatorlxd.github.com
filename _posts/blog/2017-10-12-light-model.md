---
layout: post
title: 光照模型--光源与材质的相互作用
categories: [DX11,Graphics]
description: 图形学光照模型的介绍
keywords: Light,Material,Graphics
---

# 光照模型--光源与材质的相互作用

&emsp;&emsp;近日在看*《Introduction to 3D game programming with the Direct X 11》*的光照部分，于是便决定写一个总结我的所学习的光照方面的文章。

&emsp;&emsp;在为物体加上光照效果时，我们不是直接地指明物体顶点的具体颜色，而是使用光源和材质这两种概念来描述物体受光照后的颜色状态。

&emsp;&emsp;材质是一种用于描述物体的表面与怎样光源交互的一种属性。