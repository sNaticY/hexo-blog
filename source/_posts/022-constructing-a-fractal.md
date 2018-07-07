---
title: Catlike学习笔记(1.4)-使用Unity构建分形
date: 2018-07-07 19:42:15
tags: [unity, tutorial, basic, csharp]
categories: Catlike学习笔记
mathjax: true
---

又两个星期没写文章了，主要是沉迷 [Screeps](https://store.steampowered.com/app/464350/Screeps/) 这个游戏，真的是太好玩了导致我这两个礼拜 [Github](https://github.com/sNaticY) 小绿点几乎天天刷。其实想开一个新坑大概把自己写 AI 的心路历程记录下，不过觉得因为要消耗太多时间暂时决定先不开，准备把过程中遇到的有趣的算法问题记录下就好了。言归正传今天来到「[构建分形](https://catlikecoding.com/unity/tutorials/constructing-a-fractal/)」 这篇文章。比较简单主要介绍递归的思想。我们就迅速一些，~~因为我还要继续沉迷 Screeps，~~因为还要继续学习嗯。。。再贴一次「[原文链接](https://catlikecoding.com/unity/tutorials/constructing-a-fractal/)」吧。。

<!--more-->

## PART 1 概述

「[分形](https://en.wikipedia.org/wiki/Fractal)」这种东西，随便了解一下大概就能想到作者要用递归的方法来完成。所以这一篇教程性质的文章主要是讲在 Unity 里使用递归完成一些事情。鉴于大家应该上大学的时候随随便便上个课就差不多了解递归这种基本概念，因此我们就进展快一些～大概要完成以下事情：

* 使用递归生成一大堆立方体和球体
* 整理一下使其变成分形
* 美化一下

## PART 2 递归生成

首先我们要在 MonoBehaviour 里面生成一个立方体，需要如下代码。

```csharp
public class Fractal : MonoBehaviour
{
	public Mesh Mesh;
	public Material Material;
	
	// Use this for initialization
	void Start ()
	{
		gameObject.AddComponent<MeshFilter>().mesh = Mesh;
		gameObject.AddComponent<MeshRenderer>().material = Material;
	}
}
```

非常简单，然后在场景里新建一个 GameObject 再挂上这个脚本，拖一些默认的 mesh 和 material 上去就好了，运行发现 OK 完美成功。那么说好的递归呢？非常简单，我们只需要在`Start()`里面创建一个新的 GameObject 再给他挂上这个 MonoBehaviour 就好。当然要记得限制递归的次数不然要爆炸～每次递归都记得调整子物体的位置和大小，最后设置一下递归深度这样就 OK 了，代码如下

```csharp
public class Fractal : MonoBehaviour
{
	public Mesh Mesh;
	public Material Material;

	public int Depth;
	
	// Use this for initialization
	void Start ()
	{
		gameObject.AddComponent<MeshFilter>().mesh = Mesh;
		gameObject.AddComponent<MeshRenderer>().material = Material;
		if (Depth > 0)
		{
			new GameObject("Fractal Child").AddComponent<Fractal>().Initialize(this);
		}
		
	}

	public void Initialize(Fractal parent)
	{
		Mesh = parent.Mesh;
		Material = parent.Material;
		Depth = parent.Depth - 1;
		transform.SetParent(parent.transform);
	}
}

```

那么这样就可以生成一大堆叠在一起的立方体了。。接下来的目标就是对这段代码修修补补让这些立方体组成看起来像是分形的样子。

## PART 3 分形

首先我们尝试让每个立方体在除了底面的每一面生成一个比他小一半的立方体。首先需要让`Initialize()`接收位置和方向以及大小的参数。

```csharp
public void Initialize(Fractal parent, float size, Vector3 pos, Vector3 rot)
{
    Mesh = parent.Mesh;
    Material = parent.Material;
    Depth = parent.Depth - 1;
    Size = size;
    transform.SetParent(parent.transform);
    transform.localPosition = pos;
    transform.localEulerAngles = rot;
    transform.localScale = Vector3.one * size;
}
```

非常简单，然后在每个立方体执行`Start()`的时候初始化 5 个小立方体，之所以我们需要设置小立方体的朝向是为了小立方体朝着其 Z 轴方向 (0, 0, 1) 生长，而不用考虑每次递归的时候的生长方向。代码如下

```csharp
private void Start ()
{
    gameObject.AddComponent<MeshFilter>().mesh = Mesh;
    gameObject.AddComponent<MeshRenderer>().material = Material;
    if (Depth <= 0) return;
    var posOffset = Size + Size / 2f;
    new GameObject("Fractal Child").AddComponent<Fractal>().Initialize(this, Size, Vector3.left * posOffset, new Vector3(0, -90, 0));
    new GameObject("Fractal Child").AddComponent<Fractal>().Initialize(this, Size, Vector3.right * posOffset, new Vector3(0, 90, 0));
    new GameObject("Fractal Child").AddComponent<Fractal>().Initialize(this, Size, Vector3.up * posOffset, new Vector3(-90, 0, 0));
    new GameObject("Fractal Child").AddComponent<Fractal>().Initialize(this, Size, Vector3.down * posOffset, new Vector3(90, 0, 0));
    new GameObject("Fractal Child").AddComponent<Fractal>().Initialize(this, Size, Vector3.forward * posOffset, new Vector3(0, 0, 0));
}
```

最后在场景里设置下`Scale = (0.5, 0.5, 0.5)`且`size = 0.5f Depth =  4`，再设置初始物体 Z 向上，即`(-90, 0, 0)`运行一下效果如图所示还不错～

![picture](http://ojgpkbakj.bkt.clouddn.com/2018070701.png)

嗯感觉还不错～再多设置一下变成 6 呢？我的 Macbook Pro 风扇开始呼呼的转。。。

![picture](http://ojgpkbakj.bkt.clouddn.com/2018070702.png)

接下来稍微重构下代码～之前的太丑了。我们把五行长得差不多的创建子物体的代码提取一下关键参数，完整版如下：

```csharp
public class Fractal : MonoBehaviour
{
	public Mesh Mesh;
	public Material Material;

	public int Depth;
	public float Size;

	private readonly Vector3[] _positions = {Vector3.left, Vector3.right, Vector3.up, Vector3.down, Vector3.forward};
	private readonly Vector3[] _rotations = {Vector3.down, Vector3.up, Vector3.left, Vector3.right, Vector3.zero};
	
	// Use this for initialization
	private void Start ()
	{
		gameObject.AddComponent<MeshFilter>().mesh = Mesh;
		gameObject.AddComponent<MeshRenderer>().material = Material;
		if (Depth <= 0) return;
		var posOffset = Size + Size / 2f;
		for (int i = 0; i < 5; i++)
		{
			new GameObject("Fractal Child").AddComponent<Fractal>().Initialize(this, Size, _positions[i] * posOffset, _rotations[i] * 90);
		}
	}

	public void Initialize(Fractal parent, float size, Vector3 pos, Vector3 rot)
	{
		Mesh = parent.Mesh;
		Material = parent.Material;
		Depth = parent.Depth - 1;
		Size = size;
		transform.SetParent(parent.transform);
		transform.localPosition = pos;
		transform.localEulerAngles = rot;
		transform.localScale = Vector3.one * size;
	}
}

```

## PART 4 美化

感觉作者写的美化一点都不美～不过我们还是按照教程顺手做一些换个 Mesh 啊随机旋转啦，生成机率之类的事情吧也算是有个交代。

### 随机 Mesh

这个非常简单了我们把 Mesh 这个字段扩充成一个数组。然后在初始化`MeshFilter`的地方从里面随机一个出来，像下面这样。然后在拖一些 Mesh 进去。

```csharp
public class Fractal : MonoBehaviour
{
	public Mesh[] Mesh;
	public Material Material;
	...
	private void Start ()
	{
		gameObject.AddComponent<MeshFilter>().mesh = Mesh[Random.Range(0, Mesh.Length)];
		gameObject.AddComponent<MeshRenderer>().material = Material;
		...
	}
	...
}

```

这样就可以了～运行起来每次都不太一样。。图就不截了变化并不大大家应该可以想象出来～

### 生成机率

也很简单，添加一个机率然后在生成的地方每次随机一下，随到了就生成。。

```csharp
public class Fractal : MonoBehaviour
{
	...
	public float Probability;
	...
	private void Start ()
	{
		...
		for (int i = 0; i < 5; i++)
		{
			if (Random.Range(0, 1f) <= Probability)
			{
				new GameObject("Fractal Child").AddComponent<Fractal>().Initialize(this, Size, _positions[i] * posOffset, _rotations[i] * 90);
			}
		}
	}
	
	public void Initialize(Fractal parent, float size, Vector3 pos, Vector3 rot)
	{
		...
		Probability = parent.Probability;
		...
	}
}

```

把机率调成 0.75 以后生成效果如下图（跟上一条随机 Mesh 一起展示了）

![picture](http://ojgpkbakj.bkt.clouddn.com/2018070703.png)

### 旋转起来吧

接下来要做的就是让这些东西全部动起来。。。并且以随机的速度。。嗯我已经可以想像出来大概是怎样的鬼畜场景了，尝试实现一下的话首先就是加一个最大旋转速度。然后在`Update()`里面随机好速度然后做一次旋转就好了～

```csharp
public class Fractal : MonoBehaviour
{
	...
	public float MaxRotateSpeed;
	...
	
	private void Start ()
	{
		...
	}

	private void Update()
	{
		var rotationSpeed = Random.Range(-MaxRotateSpeed, MaxRotateSpeed);
		transform.Rotate(0f, rotationSpeed * Time.deltaTime, 0f);
	}

	...
}

```

运行一下发现似乎总是在原地抖动的样子。。。一定是我们速度变换的频率太高了所以最终结果会趋近于原地不动，稍微限制一下加点随机。。

 ```csharp
public class Fractal : MonoBehaviour
{
	...
	public float MaxRotateSpeed;
	public float RotateSpeedChangeRate;
	private float RotateSpeed;
	...
	
	private void Update()
	{
		Random.InitState(Depth * (int)Mathf.Ceil(Time.time * 100));
		if (Random.Range(0, 1f) <= RotateSpeedChangeRate)
		{
			RotateSpeed = Random.Range(-MaxRotateSpeed, MaxRotateSpeed);
		}
		transform.Rotate(0f, 0f,  RotateSpeed * Time.deltaTime);
	}

	public void Initialize(Fractal parent, float size, Vector3 pos, Vector3 rot)
	{
		...
		MaxRotateSpeed = parent.MaxRotateSpeed;
		RotateSpeedChangeRate = parent.RotateSpeedChangeRate;
		...
	}
}
 ```

哇画面真的是太诡异了。。。

![animation](http://ojgpkbakj.bkt.clouddn.com/2018070705.gif)

## PART 5 总结

好的这一篇文章就这样成功的 ~~水过去了~~ 完成了～这一篇大概上就是递归的使用方法吧～其实没怎么看原文自己摸索的时候还是要稍微花几分钟的，不过还是非常简单啊大家随便看看应该就可以了解的很透彻了～感兴的同学的欢迎 follow 我的「[Github](https://github.com/sNaticY)」下载「[项目工程](https://github.com/sNaticY/CatlikePractice)」准备继续去玩 Screeps 喽~

---

原文链接：http://snatix.com/2018/07/07/022-constructing-a-fractal/

本文由 sNatic 发布于『[大喵的新窝](http://snatix.com)』 转载请保留本申明