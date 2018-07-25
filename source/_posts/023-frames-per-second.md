---
title: Catlike学习笔记(1.5)-使用Unity模拟原子核
date: 2018-07-17 16:40:48
tags: [unity, 教程, 基础, csharp]
categories: Catlike学习笔记
description: Catlike Coding, Unity模拟原子核, Unity基础教程, Unity项目Fps统计
---

周末打了两天 Celeste 蔚蓝终于打穿了，作为 IGN 2018 年首款满分神作还是很名副其实的，至少关卡设计绝对是大师级的～虽然难到爆炸但是总有一种我刚才是失误下一次一定过的错觉。。事实证明像博主这样的手残党也可以在经历十几个小时的磨难后顺利通关～总的来说还是很赞的游戏，十几个小时的游戏时间绝对值回票价~是的没错说了这么多就是为了完美的解释上周末没有更新文章的原因。。好吧总之这周来到了「[Catlike Coding 第一章](https://catlikecoding.com/unity/tutorials/basics/)」的最后一篇文章～这篇主要是讲如何使用 Profiler 查看游戏的性能以及写个小工具测量帧率～按照惯例附上『[原文链接](https://catlikecoding.com/unity/tutorials/frames-per-second/)』

<!--more-->

## PART 1 概述

那么为了使用 Profiler 之类的工具可以有效的查看到性能的变化～我们需要制作一个跑得越来越慢的 Demo 的样子。。所以用 Unity 物理组件模拟一个不断增长的原子核似乎是个不错的想法。所以我们的目标大概有以下这些～

* 使用 Unity 物理组件模拟原子核并使其不断增长
* 使用 Profiler 查看游戏性能并稍加分析
* 制作一个 FPS 指示器实时显示当前 FPS

感觉需求并不是非常复杂～开工！

## PART 2 制作模拟原子核

我们并没有打算制作完全符合物理学的原子核模型～只是一些会被吸引到原点的小球而已跟真正的原子核一点关系都没有只是很有趣。所以第一步就是制作两个不同颜色的小球的 Prefab 一个代表质子另一个代表中子这样。首先我们制作一个脚本可以给小球一个由其当前位置指向世界坐标原点的力。

```csharp
[RequireComponent(typeof(Rigidbody))]
public class Nucleon : MonoBehaviour
{
    public float AttractionForce;
    private Rigidbody _body;

    private void Awake()
    {
        _body = GetComponent<Rigidbody>();
    }

    private void FixedUpdate()
    {
        _body.AddForce(transform.localPosition * -AttractionForce);
    }
}
```

大概就是这样～代码非常简单想必大家都看得懂就不解释了。。。接下来做两个不同颜色的 Material 以便区分质子和中子～比如像这样：

![picture](http://ojgpkbakj.bkt.clouddn.com/2018071701.png)

![picture](http://ojgpkbakj.bkt.clouddn.com/2018071702.png)

最后把这些东西拼在一起做成 Prefab。

![picture](http://ojgpkbakj.bkt.clouddn.com/2018071703.png)

![picture](http://ojgpkbakj.bkt.clouddn.com/2018071704.png)

完成质子和中子的 Prefab 以后我们就可以生成这些质子和中子了，添加一个空 GameObject 并挂上以下代码

```csharp
public class NucleonSpawner : MonoBehaviour
{
	public float TimeBetweenSpawns;
	public float SpawnDistance;
	public Nucleon[] NucleonPrefabs;

	private float _timeSinceLastSpawn;

	private void FixedUpdate()
	{
		_timeSinceLastSpawn += Time.deltaTime;
		if (_timeSinceLastSpawn >= TimeBetweenSpawns)
		{
			_timeSinceLastSpawn -= TimeBetweenSpawns;
			SpawnNucleon();
		}
	}

	private void SpawnNucleon()
	{
		var prefab = NucleonPrefabs[Random.Range(0, NucleonPrefabs.Length)];
		var spawn = Instantiate<Nucleon>(prefab);
		spawn.transform.localPosition = Random.onUnitSphere * SpawnDistance;
	}
}
```

最后再设置好合适的参数就可以了～比如这样：

![picture](http://ojgpkbakj.bkt.clouddn.com/2018071705.png)

到此为止我们的原子核生成器就完成了～运行效果如图所示：

![picture](http://ojgpkbakj.bkt.clouddn.com/2018071706.gif)

## PART 3 使用 Profiler 分析性能

我们一边运行一边打开 Profiler 看看～发现大概是下图的样子，博主用 Macbook 做的实验因此可以看到偶尔物理处理的部分那根柱子爆表了。。。以及偶尔会出现的 EditorOverhead 之类的干扰项。

![picture](http://ojgpkbakj.bkt.clouddn.com/2018072203.png)

我们可以在 Profiler 里面找到很多相关的数据但是并不十分准确～可以尝试打包以后再连接 Profiler 查看更准确的数据。要记得勾选`Development Build`和`Autoconnect Profiler`。

![picture](http://ojgpkbakj.bkt.clouddn.com/2018072204.png)

运行程序再在 Profiler 里面选择正确的要调试的应用。可以看到数据不像在 Editor 里那样疯狂跳动而是变得平滑一些。当然各项消耗的占比也会略有不同，有兴趣的话还可以尝试安卓或 ios 看看是不是会有更显著的差距。

![picture](http://ojgpkbakj.bkt.clouddn.com/2018072205.png)

## PART 4 计算FPS

首先我们简单的制作一个显示当前 FPS 的脚本，大概代码如下所示

```csharp
public class FPSDisplay : MonoBehaviour
{
	[SerializeField] private Text _fpsText;

	// Update is called once per frame
	void Update()
	{
		_fpsText.text = ((int)(1f / Time.unscaledDeltaTime)).ToString();
	}
}
```

然后发现，我们每一帧把 int 转换成 string 都会产生一些额外的 GC 开销，

![picture](http://ojgpkbakj.bkt.clouddn.com/2018072206.png)

因此我们尝试提前建立 int 到 string 的索引，首先把帧数显示限制在 0-100 的范围内，然后从 List 中取出相应的字符串。

```csharp
public class FPSDisplay : MonoBehaviour
{
	[SerializeField] private Text _fpsText;

	private static readonly List<string> _fpsStrings = new List<string>
	{
		"00", "01", "02", "03", "04", "05", "06", "07", "08", "09",
		"10", "11", "12", "13", "14", "15", "16", "17", "18", "19",
		"20", "21", "22", "23", "24", "25", "26", "27", "28", "29",
		"30", "31", "32", "33", "34", "35", "36", "37", "38", "39",
		"40", "41", "42", "43", "44", "45", "46", "47", "48", "49",
		"50", "51", "52", "53", "54", "55", "56", "57", "58", "59",
		"60", "61", "62", "63", "64", "65", "66", "67", "68", "69",
		"70", "71", "72", "73", "74", "75", "76", "77", "78", "79",
		"80", "81", "82", "83", "84", "85", "86", "87", "88", "89",
		"90", "91", "92", "93", "94", "95", "96", "97", "98", "99"
	};

	// Update is called once per frame
	void Update()
	{
		var curFps = Mathf.Clamp((int) (1f / Time.unscaledDeltaTime), 0, 99);
		_fpsText.text = _fpsStrings[curFps];
	}
}
```

再次使用 Profiler 发现讨厌的 GC 消失不见了~

![picture](http://ojgpkbakj.bkt.clouddn.com/2018072208.png)

不过我们的 FPS 指示器还有另外一个缺陷，就是每帧都在跳动如果变化非常剧烈的话基本上看不清显示的是什么，尽管我们可以改成每秒计算一次之类的，不过这样就没有办法感受到 FPS 在一秒之内产生怎样的变化。因此我们的做法就是求一定过去一定帧数之内的平均值。

```csharp
public class FPSDisplay : MonoBehaviour
{
	[SerializeField] private Text _fpsText;
	[SerializeField] private int _fpsRange = 60;
	
	private int[] _fpsBuffer;
	private int _fpsBufferIndex;

	...

	private void Awake()
	{
		_fpsBuffer = new int[_fpsRange];
	}

	// Update is called once per frame
	private void Update()
	{
		UpdateBuffer();
		CalcFPS();
	}

	private void UpdateBuffer()
	{
		var curFps = (int) (1f / Time.unscaledDeltaTime);
		_fpsBuffer[_fpsBufferIndex] = curFps;
		_fpsBufferIndex++;
		if (_fpsBufferIndex >= _fpsRange)
		{
			_fpsBufferIndex = 0;
		}
	}

	private void CalcFPS()
	{
		var sum = 0;
		foreach (var fps in _fpsBuffer)
		{
			sum += fps;
		}
		_fpsText.text = _fpsStrings[Mathf.Clamp(sum / _fpsRange, 0, 99)];
	}
}

```

这样就可以求过去 60 帧之内的 FPS 的平均值了。我们还可以顺手把过去 60 帧之内的最大最小 FPS 分别显示出来，稍微改一改 UI 增加两个 Text 分别用于显示最大和最小值再修改代码如下：

```csharp
public class FPSDisplay : MonoBehaviour
{
	[SerializeField] private Text _lowFpsText;
	[SerializeField] private Text _fpsText;
	[SerializeField] private Text _highFpsText;
	[SerializeField] private int _fpsRange = 60;
	
	...

	private void CalcFPS()
	{
		var lowest = int.MaxValue;
		var highest = 0;
		var sum = 0;
		foreach (var fps in _fpsBuffer)
		{
			if (fps < lowest)
			{
				lowest = fps;
			}

			if (fps > highest)
			{
				highest = fps;
			}
			sum += fps;
		}

		_lowFpsText.text = _fpsStrings[Mathf.Clamp(lowest, 0, 99)];
		_fpsText.text = _fpsStrings[Mathf.Clamp(sum / _fpsRange, 0, 99)];
		_highFpsText.text = _fpsStrings[Mathf.Clamp(highest, 0, 99)];
	}
}
```

这样我们就可以愉快的把一定时间内最大最小以及平均 FPS 显示出来了～效果不错。。。

![picture](http://ojgpkbakj.bkt.clouddn.com/2018072209.png)

最后我们为不同数值范围的 FPS 上色，使得玩家可以更直观的感受到当前 FPS 正常还是过低。首先添加一个 Struct 里面保存一个颜色以及该颜色对应最低 FPS 值。

```csharp
[Serializable]
struct FPSColor {
	public Color color;
	public int minimumFPS;
}
```

然后再稍微重构一下代码，在`CalcFps()`中每帧计算出来的 FPS 保存在类的成员变量中记录下来，然后把显示 FPS 的代码提取出来变成一个函数`DisplayFps()`每帧调用，分别用于刷新三个 Text 组件的颜色以及 FPS 数值。

```csharp
public class FPSDisplay : MonoBehaviour
{
	...
	[SerializeField] private int _fpsRange = 60;

	[SerializeField] private FPSColor[] _fpsColors;
	
	private int _lowFps;
	private int _averageFps;
	private int _highFps;
	
	private int[] _fpsBuffer;
	private int _fpsBufferIndex;
	
	...
	
	private void Update()
	{
		UpdateBuffer();
		CalcFps();
		DisplayFps(_lowFpsText, _lowFps);
		DisplayFps(_averageFpsText, _averageFps);
		DisplayFps(_highFpsText, _highFps);
	}
	
	...
	
	private void CalcFps()
	{
		...
		_lowFps = lowest;
		_averageFps = sum / _fpsRange;
		_highFps = highest;
	}

	private void DisplayFps(Text label, int fps)
	{
		label.text = _fpsStrings[Mathf.Clamp(fps, 0, 99)];
		for (var i = 0; i < _fpsColors.Length; i++) {
			if (fps < _fpsColors[i].minimumFPS) continue;
			label.color = _fpsColors[i].color;
			break;
		}
	}
}
```

最后再在 Inspector 里设置好各种颜色如图所示：

![picture](http://ojgpkbakj.bkt.clouddn.com/2018072210.png)

这样一个完美的 FPS 指示器就完成了～

![picture](http://ojgpkbakj.bkt.clouddn.com/2018072211.png)

## PART 5 总结

至此「[Catlike Coding 第一章](https://catlikecoding.com/unity/tutorials/basics/)」内容已经全部结束～目前为止都是非常基础的课程，博主大部分时间都是在整理文字并没有花太多时间在 Unity 和 c# 上，很多地方也都是大概看一下原作者的思路就差不多自己去实现了并没有完全照搬代码，根据自己的理解写一遍下来感觉还是收获颇丰的虽然很多地方有点懒就跳过了，尤其是关于 Profiler 的部分感觉有些枯燥而且博主水平有限就没有深入，感兴趣或者有不太清楚的同学可以自行前往『[原文链接](https://catlikecoding.com/unity/tutorials/frames-per-second/)』寻找更详细的讲解。希望下一篇文章不要再拖更两个礼拜了嗯就这样～

---

原文链接：https://snatix.com/2018/07/17/023-frames-per-second/

本文由 [sNatic](https://github.com/sNaticY) 发布于『[大喵的新窝](https://snatix.com)』 转载请保留本申明