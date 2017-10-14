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
如图所示，我们通过顶点法线可以在属性插值时期计算出某个像素所对应的平面上的顶点的法线。当然，这种法线模型（如图）是不真实的，只是因为这样可以使得某些面数较少的物体也具有光滑的效果，即其面与面之间的过渡较为自然。这种方法被称为**Phong差值着色**。

## 顶点法线的计算

&emsp;&emsp;那么我们应如何计算顶点法线呢？我们由定义可知：顶点法线是在平面上的顶点上的，所以我们就可以由要计算的顶点所在的三角形的两条边的叉乘，求出垂直于该三角形所在平面的向量，再将之单位化，即为我们所求的顶点法线，**当然，如果这个顶点同时在多个三角形上，要对各个三角形的法向量计算结果求均值**。  
放一段代码：  
```c++
	void CalculationNormalVector()
	{
		for(int i=0;i<m_Vertices.size();i+=3)
		{
			Vector3D normal=CrossProduct((m_Vertices[i+1].m_Position-m_Vertices[i].m_Position),(m_Vertices[i+2].m_Position-m_Vertices[i+1].m_Position));
			m_Vertices[i].m_NormalVector=normal;
			m_Vertices[i+1].m_NormalVector=normal;
			m_Vertices[i+2].m_NormalVector=normal;
		}
		bool* flag=new bool[m_Vertices.size()];
		memset(flag,false,sizeof(bool)*m_Vertices.size());
		for(int i=0;i<m_Vertices.size();i++)
		{
			if(!flag[i])
			{
				vector<Vertex*> pointers;
				int n=0;
				Vector3D sum={0,0,0};
				for(int j=i;j<m_Vertices.size();j++)
				{
					if(m_Vertices[i].m_Position==m_Vertices[j].m_Position)
					{
						n+=1;
						sum=sum+m_Vertices[j].m_NormalVector;
						flag[j]=true;
						pointers.push_back(&m_Vertices[j]);
					}
				}
				Vector3D ans=sum/n;
				for(auto p:pointers)
				{
					p->m_NormalVector=ans;
				}
			}
		}
		delete[] flag;
	}
```