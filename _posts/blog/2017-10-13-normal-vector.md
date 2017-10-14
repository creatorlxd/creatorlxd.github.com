---
layout: post
title: 顶点法线的概念及其相关计算
categories: [DX11,Graphics]
description: 图形学顶点法线的概念及其相关计算
keywords: Light,Material,Graphics,Normal Vector
---

# 顶点法线的概念及其相关计算

## 顶点法线的概念

&emsp;&emsp;平面法线是一个平面用来描述其所朝的方向的单位向量，而顶点法线则是一个顶点在某一平面上时的朝向。

&emsp;&emsp;这里简单描述一下顶点法线的作用：  
![图片1](/images/normal-vector1.png)  
如图所示，我们通过顶点法线可以在属性插值时期计算出某个像素所对应的平面上的顶点的法线。当然，这种法线模型（如图）是不真实的，只是因为这样可以使得某些面数较少的物体也具有光滑的效果，即其面与面之间的过渡较为自然。这种方法被称为**Phong差值着色**

