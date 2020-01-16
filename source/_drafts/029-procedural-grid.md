---
title: Post Template(0)-SubTitle
date: 2018-07-17 16:40:48
tags: [unity, 教程, 基础, csharp]
categories: Catlike学习笔记
description: 随便写点东西试试有没有用
---


<!--more-->

## PART 1 概述

- Create a grid of points. 生成网格顶点
- Use a coroutine to analyze their placement. 
- Define a surface with triangles. 使用三角形生成表面
- Automatically generate normals. 自动生成法线
- Add texture coordinates and tangents. 添加贴图坐标和切线

## PART 2 生成网格顶点

想要生成 mesh 的话首先我们需要一组顶点，创建`Grid.cs`并添加以下代码：

```csharp
using UnityEngine;
using System.Collections;

[RequireComponent(typeof(MeshFilter), typeof(MeshRenderer))]
public class Grid : MonoBehaviour
{
	public int XSize;
	public int YSize;

	private Vector3[] _vertices;

	private void Awake()
	{
		Generate();
	}

	private void Generate()
	{
		_vertices = new Vector3[(XSize + 1) * (YSize + 1)];
		for (int i = 0, y = 0; y <= YSize; y++) {
			for (int x = 0; x <= XSize; x++, i++) {
				_vertices[i] = new Vector3(x, y);
			}
		}
	}

	private void OnDrawGizmos()
	{
		if (_vertices == null) {
			return;
		}
		
		Gizmos.color = Color.black;
		for (int i = 0; i < _vertices.Length; i++)
		{
			Gizmos.DrawSphere(_vertices[i], 0.1f);
		}
	}
}
```

首先通过`XSize`和`YSize`来规定要画出的网格的大小，然后可以看到我们在`Generate()`中创建了大小为 $(x+1)*(y+1)$的`Vector3`类型的数组用于保存所有的顶点。同时为了我们可以方便的看到顶点，使用`OnDrawGizmos()`来显示每个顶点的位置，最终效果如下：

![](https://blog-1301118239.cos.eu-frankfurt.myqcloud.com/Images/2018102202.png)

但是有一个小小的缺点就是我们没有办法分辨这些顶点的生成顺序，有两种方式可以解决这个问题，第一种方法是使用颜色进行区分，第二种就是我们接下来要采用的，使用`Coroutine`让点以肉眼可见的速度逐个生成。修改如下代码：

```csharp
private void Awake()
{
    StartCoroutine(Generate());
}

private IEnumerator Generate()
{
    var wait = new WaitForSeconds(0.05f);
    _vertices = new Vector3[(XSize + 1) * (YSize + 1)];
    for (int i = 0, y = 0; y <= YSize; y++) {
        for (int x = 0; x <= XSize; x++, i++) {
            _vertices[i] = new Vector3(x, y);
            yield return wait;
        }
    }
}
```

我们将`Generate()`函数修改成协程并在每次创建单个点后等待 0.05 秒，然后在`Awake()`中通过`StartCoroutine()`来调用。效果如下

![](https://blog-1301118239.cos.eu-frankfurt.myqcloud.com/Images/2018102204.gif)

## PART 3 创建Mesh

那么完成所有顶点以后就该创建三角面了，在创建之前需要注意的是，三角面是有朝向的，只有从正面看才能被渲染出来，那么如何定义正面与背面呢，我们先来创建一个三角面试一下。

```csharp
public class Grid : MonoBehaviour
{
	private Vector3[] _vertices;
	private Mesh _mesh;
	...
	
	private IEnumerator Generate()
	{	
		GetComponent<MeshFilter>().mesh = _mesh = new Mesh();
		_mesh.name = "Procedural Grid";
		
		var wait = new WaitForSeconds(0.05f);
		_vertices = new Vector3[(XSize + 1) * (YSize + 1)];
		for (int i = 0, y = 0; y <= YSize; y++) {
			...
		}
		
		_mesh.vertices = _vertices;
		
		int[] triangles = new int[3];
		triangles[0] = 0;
		triangles[1] = XSize + 1;
		triangles[2] = 1;
		_mesh.triangles = triangles;
	}
	...
}
```

首先我们将之前创建好的`_vertices`数组赋值给了`_mesh.vertices`，又创建了名为`triangles`的`int`数组并在里面添加了顶点的索引，分别为 `0, XSize+1, 1`。运行一下会发现确实出现了正确的三角面，如图所示：

![](https://blog-1301118239.cos.eu-frankfurt.myqcloud.com/Images/2018102205.png)

那么如果顶点的索引是`0, 1, XSize+1`会发生什么事情呢？答案是三角面消失，需要我们转到背面才可以看到。因此我们需要注意的是，只有顶点的顺序为顺时针时该三角面才可见。接下来我们在一个循环中将所有的三角面都添加进来：

```csharp
private IEnumerator Generate()
{	
    GetComponent<MeshFilter>().mesh = _mesh = new Mesh();
    _mesh.name = "Procedural Grid";

	...
    _mesh.vertices = _vertices;

    int[] triangles = new int[XSize * YSize * 6];
    for (int ti = 0, vi = 0, y = 0; y < YSize; y++, vi++)
    {
        for (int x = 0; x < XSize; x++, ti += 6, vi++)
        {
            triangles[ti] = vi;
            triangles[ti + 3] = triangles[ti + 2] = vi + 1;
            triangles[ti + 4] = triangles[ti + 1] = vi + XSize + 1;
            triangles[ti + 5] = vi + XSize + 2;
            _mesh.triangles = triangles;
            yield return wait;
        }
    }
}
```

需要注意的大概就是此处的两层`for`循环以及`triangles`中点的顺序了，一定要是顺时针排布。两层循环的算法原理就不多讲了大家稍微自己写一下就能瞬间明白～最后为了防止继续出现材质丢失的刺眼粉色我们先随便添加一个材质～最终运行效果如下：

![](https://blog-1301118239.cos.eu-frankfurt.myqcloud.com/Images/2018102206.gif)

## PART 4 添加其他数据



## PART 5 总结

![](https://blog-1301118239.cos.eu-frankfurt.myqcloud.com/Images/.png)

---

原文链接：https://snatix.com/2018/07/17/023-frames-per-second/

本文由 [sNatic](https://github.com/sNaticY) 发布于『[大喵的新窝](https://snatix.com)』 转载请保留本申明