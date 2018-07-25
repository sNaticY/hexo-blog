---
title: Catlike学习笔记(1.2)-使用Unity画函数图像
date: 2018-06-09 20:55:42
tags: [unity, 教程, 基础, csharp]
categories: Catlike学习笔记
mathjax: true
description: Catlike Coding, Unity使用Cube绘制函数图像, Unity基础教程
---

『[Catlike系列教程](https://catlikecoding.com/unity/tutorials/)』第二篇来了~今天周六，~~早上~~（上午11点）醒来去超市买了一周的零食回来以后就玩了一整天游戏非常有负罪感。现在晚上九点天还亮着感觉像下午7点左右的样子好像还不是很晚。。。所以就写一点东西吧。这一篇是「[Building a Graph](https://catlikecoding.com/unity/tutorials/basics/building-a-graph/)」挑战一下试试吧。

<!--more-->

## PART 1 概述

那么大概文章看下来我们预计要做以下事情。

* 使用一定数量的小方块表达函数图像
* 做一个 Shader 给图像上色使其更好看
* 给图像传入时间参数使其动起来

## PART 2 画函数图像

首先我们确认一下要支持的功能细节，根据默认摄像机的位置和视野我们就暂定需要画出函数[-1. 1]之间的图像，同时可以通过一个条拖动来修改函数的解析度，假设解析度可以是[10,100]，那么我们需要在x=[-1, 1]之间生成[10,100]个方块。同时动态的调整方块的大小使其完美衔接。代码如下：

```csharp
public class GraphController : MonoBehaviour
{
	[Range(10, 100), SerializeField] private int _resolution;
	[SerializeField] private GameObject _cube;
	
	// Use this for initialization
	private void Start ()
	{
		_cube.SetActive(false);
		
		var step = 2f / _resolution;
		var startPosX = -1f;
		var scale = Vector3.one * step;
		for (int i = 0; i < _resolution; i++)
		{
			var pos = new Vector3(startPosX + i * step, Calc(startPosX + i * step), 0);
			var point = Instantiate(_cube, transform);
			point.transform.localPosition = pos;
			point.transform.localScale = scale;
			point.SetActive(true);
		}
	}

	private float Calc(float x)
	{
		return Mathf.Pow(x, 2);
	}
}
```

根据`Calc(float)`可以看出我们画出来曲线是如下函数的图像。 $y=x^2$ $f(x)=ax+b$

![Graph](http://ojgpkbakj.bkt.clouddn.com/2018061001.png)

## PART 3 给曲线上色

首先我们创建一个 Surface Shader，通过`Assets / Create / Standard Surface Shader`创建一个新的 Shader。修改代码如下：

```c
Shader "Custom/ColoredPoint" {
	Properties {
		_Glossiness ("Smoothness", Range(0,1)) = 0.5
		_Metallic ("Metallic", Range(0,1)) = 0.0
	}
	SubShader {
		Tags { "RenderType"="Opaque" }
		LOD 200

		CGPROGRAM
		// Physically based Standard lighting model, and enable shadows on all light types
		#pragma surface surf Standard fullforwardshadows

		// Use shader model 3.0 target, to get nicer looking lighting
		#pragma target 3.0
        
        // 此处将世界坐标传入IN
		struct Input {
			float3 worldPos;
		};

		half _Glossiness;
		half _Metallic;

		// Add instancing support for this shader. You need to check 'Enable Instancing' on materials that use the shader.
		// See https://docs.unity3d.com/Manual/GPUInstancing.html for more information about instancing.
		// #pragma instancing_options assumeuniformscaling
		UNITY_INSTANCING_BUFFER_START(Props)
			// put more per-instance properties here
		UNITY_INSTANCING_BUFFER_END(Props)

		void surf (Input IN, inout SurfaceOutputStandard o) {
		    // 在这里根据世界坐标将方块染上不同的颜色
		    o.Albedo.rgb = IN.worldPos.xyz * 0.5 + 0.5;
			// Metallic and smoothness come from slider variables
			o.Metallic = _Metallic;
			o.Smoothness = _Glossiness;
			o.Alpha = 1;
		}
		ENDCG
	}
	FallBack "Diffuse"
}

```

运行效果如下~

![Graph](http://ojgpkbakj.bkt.clouddn.com/2018061002.png)

## PART 4 让曲线动起来

首先曲线之所以会动当然是因为我们将时间作为参数同时传进去导致的~所以我们首先稍微改一下`Calc(float)`这个函数让时间参数也生效～这里用到`Mathf.PI`是为了图像从[-1, 1]之间完整的显示一个周期，大家可以随便改改试试。

```csharp
private float Calc(float x)
{
	return Mathf.Sin(Mathf.PI * (x + Time.time));
}
```

然后直接运行肯定是不行的，我们需要把计算`y`和设置每个小方块的位置的代码移到`Update()`中去。最终代码如下：

```csharp
public class GraphController : MonoBehaviour
{
	[Range(10, 100), SerializeField] private int _resolution;
	[SerializeField] private GameObject _cube;
	
	private List<Transform> _points;
	private float _step;
	private float _startX;
	
	// Use this for initialization
	private void Start ()
	{
		_cube.SetActive(false);
		_points = new List<Transform>();
		_step = 2f / _resolution;
		_startX = -1f;
		
		var scale = Vector3.one * _step;
		
		for (int i = 0; i < _resolution; i++)
		{
			var point = Instantiate(_cube, transform);
			_points.Add(point.transform);
			point.transform.localScale = scale;
			point.SetActive(true);
		}
		
	}

	private void Update()
	{
		for (int i = 0; i < _resolution; i++)
		{
			var x = _startX + i * _step;
			var pos = new Vector3(x, Calc(x), 0);
			var point = _points[i];
			point.transform.localPosition = pos;
		}
	}

	private float Calc(float x)
	{
		return Mathf.Sin(Mathf.PI * (x + Time.time));
	}
}

```

运行效果如图：

![Animation](http://ojgpkbakj.bkt.clouddn.com/2018061003.gif)

## PART 5 总结

这一篇也很简单啊一边做一边写再随便划划水很快就完成了～最后大家可以下载「[Github Repo](https://github.com/sNaticY/CatlikePractice)」查看运行结果和全部代码~或者到「[原文地址](https://catlikecoding.com/unity/tutorials/basics/building-a-graph/)」查看更加详细的过程和思路。

------

原文链接：https://snatix.com/2018/06/09/020-building-a-grap/

本文由 [sNatic](https://github.com/sNaticY) 发布于『[大喵的新窝](https://snatix.com)』 转载请保留本申明