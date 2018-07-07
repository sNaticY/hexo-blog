---
title: Catlike学习笔记(1.3)-使用Unity画更复杂的3D函数图像
date: 2018-06-20 16:46:27
tags: [unity, tutorial, basic, csharp]
categories: Catlike学习笔记
mathjax: true
---

第三篇来了～今天去参加了 Unite 2018 Berlin，感觉就是。。。。非常困。。。回来以后稍微睡了下清醒了觉得是时候认真学习下了，不过讲的很多东西都是还没有发布或者只有 Preview 的版本，按照 Unity 的习惯肯定 Bug 多到令人发指，最近不太想折腾所以就先继续写文章把。。按照惯例奉上『[原文链接](https://catlikecoding.com/unity/tutorials/basics/mathematical-surfaces/)』

<!--more-->

## PART 1 概述

首先大概介绍一下什么是『[Catlike教程](https://catlikecoding.com/unity/tutorials/)』，大家自行访问一下就会发现是这位『[大神](https://www.patreon.com/catlikecoding/memberships)』写的一个 Unity 系列教程，里面由浅至深的以一个个有趣的小课题来引导大家学习 Unity 的方方面面～回想自己毕业三年都在做 Unity 游戏开发，然而看了大神的教程以后发现自己欠缺的东西非常多～真正对引擎的掌握程度非常低只是在不停的拼 UI 写业务逻辑。做这个系列呢也是希望自己可以坚持把大神的教程学完让自己变得更厉害～就酱。。

那么言归正传我们本期节目的最终目标是实现作者配图中的看起来很屌的图形，像是这样的。。。

![Animation](http://ojgpkbakj.bkt.clouddn.com/2018061201.jpg)

对比上一篇文章的函数图像，大概有以下几个关键点需要实现。

* 支持多函数叠加
* 从一条曲线变成一个曲面
* 由曲面扩展成真正的三维图形

## PART 2 支持多函数叠加

首先我们的目标是可以通过一个滑杆来控制「[上一篇](https://snatix.com/2018/06/09/020-building-a-grap/)」中的曲线显示的函数，因此先复制之前的代码改改名字比如 Graph3DController.cs 再修改类名与文件名一致。然后我们的关键是需要修改这一行

```csharp
var pos = new Vector3(x, Calc(x), 0);
```

使其变成根据滑杆中的 int 值选择 delegate 中的某个函数，如下所示，代码中主要修改的地方用注释稍微解释了下。

```csharp
// 新的 deleagate
public delegate float Function(float x, float t);

// 记得修改类名与文件名一致否则不能挂在 gameobject 上
public class Graph3DController : MonoBehaviour
{
	[Range(10, 100), SerializeField] private int _resolution;
	[SerializeField] private GameObject _cube;
	// 添加新的滑杆
	[Range(0, 1), SerializeField] private int _function;
	// 一个 delegate 数组用于保存我们接下来使用的两个函数
	private Function[] _functions;
	
	...

	// Use this for initialization
	private void Start()
	{
		// 初始化 _functions 
		_functions = new Function[] {SineFunction, MultiSineFunction};		
		...
	}

	private void Update()
	{
		_startX = -1f;
		for (int i = 0; i < _resolution; i++)
		{
			var x = _startX + i * _step;
			// 此处修改调用方法
			var pos = new Vector3(x, _functions[_function](x, Time.time), 0);
			var point = _points[i];
			point.transform.localPosition = pos;
		}
	}

	private float SineFunction(float x, float t)
	{
		return Mathf.Sin(Mathf.PI * (x + t));
	}

	private float MultiSineFunction(float x, float t)
	{
		float y = Mathf.Sin(Mathf.PI * (x + t));
		y += Mathf.Sin(2f * Mathf.PI * (x + 2f * t)) / 2f;
		y *= 2f / 3f;
		return y;
	}
}

```

于是我们实现了如下的效果～

![Animation](http://ojgpkbakj.bkt.clouddn.com/2018061801.gif)

不过作者在原文中还添加了 Enum 然后可以不用滑杆而是改用一个下拉菜单来改变要显示的函数图像。最终效果没什么不同就不再赘述了感兴趣的同学可以自行找到『[原文链接](https://catlikecoding.com/unity/tutorials/basics/mathematical-surfaces/)』查看更详细的步骤～

## PART 3 画出水滴的波纹

那么接下来开始要真正的绘制一个3D曲面了~那么首先是创建更多的小方块～我们在初始化的地方改成一个二维的 List 来保存所有的小方块

```csharp
private void Start()
{
    ...
    for (int i = 0; i < _resolution; i++)
    {
        _points.Add(new List<Transform>());
        for (int j = 0; j < _resolution; j++)
        {
            var point = Instantiate(_cube, transform);
            _points[i].Add(point.transform);
            point.transform.localScale = scale;
            point.SetActive(true);
        }
    }
}
```

在后续的遍历也对该二维数组进行遍历。

```csharp
private void Update()
{
    for (int i = 0; i < _points.Count; i++)
    {
        for (int j = 0; j < _points[i].Count; j++)
        {
            var posX = i * _step - 1;
            var posZ = j * _step - 1;
            var pos = new Vector3(posX, _functions[(int) _function](posX, posZ, Time.time), posZ);
            var point = _points[i][j];
            point.localPosition = pos;
        }
    }
}
```

最后再稍微修改下两个函数的参数就完成了从 2D 到 3D 的跳跃～如图所示

![Animation](http://ojgpkbakj.bkt.clouddn.com/2018061802.gif)

不过我们并不应该满足于此，感觉这样其实并没有充分利用 Z 轴啊，完全就是复制了很多条曲线排在一起。所以我们新建两个这样的函数。

```csharp
private float Sine2DFunction(float x, float z, float t)
{
    float y = Mathf.Sin(Mathf.PI * (x + t));
    y += Mathf.Sin(Mathf.PI * (z + t));
    y *= 0.5f;
    return y;
}

private float MultiSine2DFunction(float x, float z, float t)
{
    float y = 4f * Mathf.Sin(Mathf.PI * (x + z + t * 0.5f));
    y += Mathf.Sin(Mathf.PI * (x + t));
    y += Mathf.Sin(2f * Mathf.PI * (z + 2f * t)) * 0.5f;
    y *= 1f / 5.5f;
    return y;
}
```

那么` Sine2DFunction`可以很明显的看出是两个完全一样的正弦波分别沿 x 轴和 Z 轴传播并且直接叠加，那么第二个。。。反正很复杂语言解释不清楚大概就是 3 个波叠加起来的，大家可以一行一行注释掉看看效果就知道了～

那么如何画出一个波纹呢，首先波纹是由原点也就是`(0, 0)`点开始均匀扩散的，那么可能是一个从原点向周围扩散的正弦波。那么直觉上来说这个函数可能长这样。。

```csharp
private float Ripple (float x, float z, float t) 
{
    float d = Mathf.Sqrt(x * x + z * z);
    float y = Mathf.Sin(Mathf.PI * (d - t));
    return y;
}
```

运行下会发现完全不像，主要是因为水波在扩散的过程中是要衰减的，正弦波完全不会，因此我们需要加上衰减的控制。既然是衰减的话显然距离越大衰减的越多喽所以我们让 y 除以 `1 + 2 * Mathf.PI * d`试一试，之所以加1是为了防止在距离原点过于近的时候结果趋近于无穷大。所以现在代码变成了这样～

```csharp
private float Ripple(float x, float z, float t)
{
    float d = Mathf.Sqrt(x * x + z * z);
    float y = Mathf.Sin(Mathf.PI * (d - t));
    y = y / (1 + 2 * Mathf.PI * d);
    return y;
}
```

跑起来看一下会发现。。。emmmm

![Animation](http://ojgpkbakj.bkt.clouddn.com/2018061901.png)

所以我们再加上一些参数比如`_velocity`传播速度，`frequency`水波频率，`_amplitude`振幅，`_attenuation`衰减。代码如下。（这些参数并不是数值越大就直观意义上越大，虽然这样不太好但是懒得整理了。。。大家大概意思理解就好）

```csharp
private float Ripple(float x, float z, float t)
{
    float d = Mathf.Sqrt(x * x + z * z);
    float y = Mathf.Sin(_frequency * Mathf.PI * (d - t / _velocity));
    y *= 1 / (_amplitude + _attenuation * 2 * Mathf.PI * d);
    return y;
}
```

然后将这些参数调整到合适的值，就完成一个完美的水波了～如图所示

![Animation](http://ojgpkbakj.bkt.clouddn.com/2018061902.gif)

## PART 4 画出三维图形

显然我们不能满足于此，传入 x 和 z 来计算出唯一的 y 导致了无法有两个点拥有相同的 x 和 z，这极大的限制了我们的发挥～比如说画出一个球体之类的。所以我们接下来的目标是画出真正的三维图形～

在开始之前，我们首先要放弃传入 x 和 z 来计算 y 的设想，所以应该把所有的函数的返回值改成 Vector3，并且为了区分我们将函数的参数变成 u，v，t。

```csharp
public delegate Vector3 Function(float u, float v, float t);

public enum GraphFunctionName {
	Sine,
	MultiSine,
	Sine2D,
	MultiSine2D,
	Ripple,
}

public class Graph3DController : MonoBehaviour
{
	[Range(10, 100), SerializeField] private int _resolution;
	[SerializeField] private GameObject _cube;
	[SerializeField] public GraphFunctionName _function;

	[SerializeField] private float _amplitude = 3;
	[SerializeField] private float _frequency = 4;
	[SerializeField] private float _velocity = 2;
	[SerializeField] private float _attenuation = 6;

	private List<List<Transform>> _points;
	private float _step;

	private Function[] _functions;

	// Use this for initialization
	private void Start()
	{
		_functions = new Function[] {SineFunction, MultiSineFunction, Sine2DFunction, MultiSine2DFunction, Ripple};

		_cube.SetActive(false);
		_points = new List<List<Transform>>();
		_step = 2f / _resolution;

		var scale = Vector3.one * _step;

		for (int i = 0; i < _resolution; i++)
		{
			_points.Add(new List<Transform>());
			for (int j = 0; j < _resolution; j++)
			{
				var point = Instantiate(_cube, transform);
				_points[i].Add(point.transform);
				point.transform.localScale = scale;
				point.SetActive(true);
			}
		}

	}

	private void Update()
	{
		for (int i = 0; i < _points.Count; i++)
		{
			for (int j = 0; j < _points[i].Count; j++)
			{
				var u = i * _step - 1;
				var v = j * _step - 1;
				var point = _points[i][j];
				point.localPosition = _functions[(int) _function](u, v, Time.time);
			}
		}
	}

	private Vector3 SineFunction(float u, float v, float t)
	{
		var x = u;
		var y = Mathf.Sin(Mathf.PI * (u + t));
		var z = v;
		return new Vector3(x, y, z);
	}

	private Vector3 MultiSineFunction(float u, float v, float t)
	{
		var x = u;
		float y = Mathf.Sin(Mathf.PI * (u + t));
		y += Mathf.Sin(2f * Mathf.PI * (u + 2f * t)) / 2f;
		y *= 2f / 3f;
		var z = v;
		return new Vector3(x, y, z);
	}

	private Vector3 Sine2DFunction(float u, float v, float t)
	{
		var x = u;
		float y = Mathf.Sin(Mathf.PI * (u + t));
		y += Mathf.Sin(Mathf.PI * (v + t));
		y *= 0.5f;
		var z = v;
		return new Vector3(x, y, z);
	}

	private Vector3 MultiSine2DFunction(float u, float v, float t)
	{
		var x = u;
		float y = 4f * Mathf.Sin(Mathf.PI * (u + v + t * 0.5f));
		y += Mathf.Sin(Mathf.PI * (u + t));
		y += Mathf.Sin(2f * Mathf.PI * (v + 2f * t)) * 0.5f;
		y *= 1f / 5.5f;
		var z = v;
		return new Vector3(x, y, z);
	}

	private Vector3 Ripple(float u, float v, float t)
	{
		var x = u;
		float d = Mathf.Sqrt(u * u + v * v);
		float y = Mathf.Sin(_frequency * Mathf.PI * (d - t / _velocity));
		y *= 1 / (_amplitude + _attenuation * 2 * Mathf.PI * d);
		var z = v;
		return new Vector3(x, y, z);
	}
}

```

### 圆柱体

那么如何组成一个圆柱体呢，首先我们知道圆柱体可以认为是由许多个圆环组成的，那么如何构成一个圆环呢？我们知道 u 的取值范围是[-1, 1]，将 u * PI 即可获得 [-PI, PI] 即刚好一个圆周的弧度，对应的坐标即是`(x = sin(PI * u), z = cos(PI * u))`，按照以上思路我们完成以下代码。然后每一个点的纵座标 y 就直接取 v 的值即可形成「每个水平的圆周上有100个点，共100个圆纵向排列组成的圆柱体」了好吧感觉表述的不是特别清楚写出来跑跑看就知道了。。。

```csharp
private Vector3 Cylinder(float u, float v, float t)
{
    var x = Mathf.Sin(Mathf.PI * u);
    var y = v;
    var z = Mathf.Cos(Mathf.PI * u);
    return new Vector3(x, y, z);
}
```

运行一下发现果然是一个圆柱体，如果想要控制圆柱体的半径和高直接在 x 和 z 乘以 R，y 乘以 H 即可，如下图所示。代码就不贴了大家都会自己乘～

![Animation](http://ojgpkbakj.bkt.clouddn.com/2018062001.png)

那么如何让这个圆柱体动起来呢～比如说随便对 R 做一些手脚像下面这样

```csharp
private Vector3 InterestingCylinder(float u, float v, float t)
{
    var r = _radius * (0.8f + Mathf.Sin(Mathf.PI * (6f * u + 2f * v + t)) * 0.2f);
    var x = r * Mathf.Sin(Mathf.PI * u);
    var y = _height * v;
    var z = r * Mathf.Cos(Mathf.PI * u);
    return new Vector3(x, y, z);
}
```

尝试改变 u 和 v 的系数可以看到很多有趣的现象哦～懒得自己写的可以打开我的「[Github Repo](https://github.com/sNaticY/CatlikePractice)」直接运行时修改 FactorU 和 FactorV 的值查看结果～最终我们可以达到类似这样的效果

![Animation](http://ojgpkbakj.bkt.clouddn.com/2018062002.gif)

### 球体

我们在圆柱体的基础上稍加修改就可以获得一个球体，首先，球体跟圆柱体一样也可以认为是很多半径不同的圆环组成的，那么圆环的半径呈现怎样的变化呢，我们想象球体沿经线切开后，可以观察到一圈纬线的半径和纬线的纵座标分别对应`Cos(PI / 2 * v)`和`Sin(PI / 2 * v)`，按照这个思路我们尝试写出如下代码。

```csharp
private Vector3 Sphere(float u, float v, float t)
{
    var r = _radius * Mathf.Cos(Mathf.PI / 2 * v);
    var x = r * Mathf.Sin(Mathf.PI * u);
    var y = _radius * Mathf.Sin(Mathf.PI / 2 * v);
    var z = r * Mathf.Cos(Mathf.PI * u);
    return new Vector3(x, y, z);
}
```

运行一下发现完全没有问题～如图所示。。。

![Animation](http://ojgpkbakj.bkt.clouddn.com/2018062003.png)

所以想要让球体动起来我们可以使用同样地思路对 r 的计算进行一点点魔改，比如说这样的一个参数`factor`：

```csharp
private Vector3 InterestingSphere(float u, float v, float t)
{
    var factor = 0.8f + Mathf.Sin(Mathf.PI * (_factorU * u + t)) * 0.1f;
    factor += Mathf.Sin(Mathf.PI * (_factorV * v + t)) * 0.1f;
    var r = factor * _radius * Mathf.Cos(Mathf.PI / 2 * v);
    ...
}
		
```

调一些奇怪的参数。。。然后就出现了一坨嚅动的，。。球体。。。

![Animation](http://ojgpkbakj.bkt.clouddn.com/2018062004.gif)

### 圆环体

那么想象下一个圆环体和球体到底有什么区别呢，针对每左半条或者右半条经线圈，如果直接变成一个环，那么球体不就变成圆环了么。。。那么怎么变成圆环呢，我们之前提到

> 一圈纬线的半径和纬线的纵座标分别对应`Cos(PI / 2 * v)`和`Sin(PI / 2 * v)

所以我们把半个周期的 cos 和 sin 变成完整周期就可以了，不要除以 2 就好。。于是我们尝试着写下如下代码

```csharp
private Vector3 Torus(float u, float v, float t)
{
    var r = _radius * Mathf.Cos(Mathf.PI * v);
    var x = r * Mathf.Sin(Mathf.PI * u);
    var y = _radius * Mathf.Sin(Mathf.PI * v);
    var z = r * Mathf.Cos(Mathf.PI * u);
    return new Vector3(x, y, z);
}
```

运行一下发现还是球体啊。。这是为什么呢，仔细观察发现似乎小方块比以前稀疏了，是因为半条经线被扩展到整个周期以后变成了一整圈经线，所以和对面的那半条完全重叠了。。所以怎么解决这个问题呢？就是扩大纬线圈让相对的两个半条经线不会相互重叠甚至完全分离就可以了。所以这样修改下试试

```csharp
private Vector3 Torus(float u, float v, float t)
{
    var r = _radius * Mathf.Cos(Mathf.PI * v) + _radius2;
    ...
}
```

这里之所以是加一个`_radius2`在最外面是为了达到「无论 v 如何变化都可以是的半径无条件增加 _radius2」的效果。。。运行下会发现嗯果然没问题了。。

![Animation](http://ojgpkbakj.bkt.clouddn.com/2018062005.png)

所以最后也顺便让它动起来吧。。。

![Animation](http://ojgpkbakj.bkt.clouddn.com/2018062006.gif)

## PART 5 总结

好吧这篇真的好长，而且写的好累并且在公式功能坏掉的情况下又很难讲清楚～大家把「[Github Repo](https://github.com/sNaticY/CatlikePractice)」下载下来自己运行稍微修改下就很容易理解了～总之我们把简单的图像扩展到了三维的图形的过程还是很有趣的～虽然不知道暂时有什么用处不过对于培养数学思维也还是挺有帮助的～好吧希望下一篇早日更新～就酱。。。

---

原文链接：http://snatix.com/2018/06/20/021-mathematical-surfaces/

本文由 sNatic 发布于『[大喵的新窝](http://snatix.com)』 转载请保留本申明