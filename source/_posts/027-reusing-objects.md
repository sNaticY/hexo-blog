---
title: Catlike学习笔记(2.3)-Unity对象池与重用
date: 2018-08-19 18:34:48
tags: [unity, 教程, 对象管理, csharp]
categories: Catlike学习笔记
description: 在 Unity 中使用对象池，使用 UGUI 创建和销毁对象。
---

短暂又美好的假期瞬间就过去了，已经上班 3 天了感觉每天早上都很困很想睡懒觉～而且也没有按照原计划在假期内完成「[对象管理](https://catlikecoding.com/unity/tutorials/object-management/)」的部分感觉好像有那么一点点惭愧（~~其实完全不惭愧反正已经不知道拖延过多少次弃坑过更多次早就习惯了~~)咳咳，然后最近几天沉迷『[群星(Stellaris)](https://store.steampowered.com/app/281990/Stellaris/)』（~~睿智的P社玩家+1~~） 话说为什么自从开始要求自己每个礼拜更新一篇博客以后比以前更容易沉迷游戏了。。。不管那些总之这一篇来到了 [Reusing Object](https://catlikecoding.com/unity/tutorials/object-management/reusing-objects/) ，主要讲了简单的对象池实现和使用，希望可以给大家带来一点点启发～

<!--more-->

## PART 1 概述

虽然主角是对象池但是如果没有一个好的可以跑起来的工程用来测试的话也不能发挥出对象池的威力～所以接『[上一篇](https://snatix.com/2018/08/08/026-object-variety/)』和『[上上篇](https://snatix.com/2018/08/05/025-persisting-objects/)』我们完成了一键创建随机材质颜色和形状的 GameObject 还可以实现保存和重新加载，最后还开启了 GPU Instancing，如果直接看这一篇有点莫名其妙的同学可以从第一篇过一遍～当然不关心场景是怎么创建出来的只想看看对象池的简单实现的话可以直接跳到「PART 4 对象池」。总之本篇的目标大概是以下这些：

- 销毁对象
- 自动创建与销毁
- 创建简单的 UI 控制自动创建和销毁的速度
- 使用 Profiler 追踪内存分配
- 使用对象池回收

## PART 2 添加一键删除随机对象

在之前的教程中我们完成了一键创建随机材质颜色和形状的 GameObject 并保存在`_objectList`中，那么相应的删除对象也很简单，大概就是从 List 中随机一个 index，销毁后再从 List 中删除即可，因此我们添加如下代码：

```csharp

public class ReusingDemo : PersistableObject
{
	...
    public KeyCode CreateKey = KeyCode.C;
    public KeyCode DestroyKey = KeyCode.X;
    ...

    private void Update()
    {
    	...
        else if (Input.GetKeyDown(DestroyKey))
        {
            DestroyObject();
        }
    }

	...

    private void DestroyObject()
    {
        if (_objectList.Count > 0)
        {
            int index = Random.Range(0, _objectList.Count);
            Destroy(_objectList[index].gameObject);
            _objectList.RemoveAt(index);
        }
    }

	...
}

```

简单的来说就是添加一个新的按键叫`DestroyKey`，在`Update()`中检测到该按键被按下时从`_objectList`中取出相应的 GameObject 并销毁，最后从`_objectList`中移除。不过这样有一个小小的不足之处，学过数据结构的同学都知道，在我们从 List / Array 之类的结构中移除一个对象的时候，会依次将后面的对象往前移动，这样就造成了不必要的性能开销～那么我们还可以稍微优化一下，代码如下：

```csharp

public class ReusingDemo : PersistableObject
{
    ...
    private void DestroyObject()
    {
        if (_objectList.Count > 0)
        {
            int index = Random.Range(0, _objectList.Count);
            Destroy(_objectList[index].gameObject);
            int lastIndex = _objectList.Count - 1;
            _objectList[index] = _objectList[lastIndex];
            _objectList.RemoveAt(lastIndex);
        }
    }
	...
}

```

大概思路就是下面这样，假设当前 List 中有 9 个对象`A-I`然后 `\0`代表结尾的话。

|  0   |  1   |  2   |  3   |   4   |  5   |  6   |  7   |   8   |      |
| :--: | :--: | :--: | :--: | :---: | :--: | :--: | :--: | :---: | :--: |
|  A   |  B   |  C   |  D   |   E   |  F   |  G   |  H   |   I   |  \0  |
|  A   |  B   |  C   |  D   |   /   |  F   |  G   |  H   | **I** |  \0  |
|  A   |  B   |  C   |  D   | **I** |  F   |  G   |  H   |   I   |  \0  |
|  A   |  B   |  C   |  D   | **I** |  F   |  G   |  H   |  \0   |  -   |

所以大概思路就是将`E`销毁掉以后，把 List 中的最后一个也就是`I`放到`E`原来的位置，然后再把`List`的最后一个移除这样。避免了大量的对象移动操作。完成以后看看效果～

![](http://ojgpkbakj.bkt.clouddn.com/2018081900.gif)

## PART 3 自动化创建和销毁对象

目前为止可以一键创建和销毁对象了，但是如果我们想要持续化的反复创建和销毁的话，一直敲键盘感觉会很累的样子，所以做一个小功能来自动化的创建和销毁比较好～所以我们可能需要创建一个拉杆允许我们调整生成和销毁的速度，总之随便用 UGUI 做一下就可以了～就像这样。

![](http://ojgpkbakj.bkt.clouddn.com/2018081901.png)

然后我们需要做一些设置再添加一些代码让这个进度条工作起来。那么首先在`Game`中添加

```csharp
public class ReusingDemo : PersistableObject
{
	...
    public float CreationSpeed { get; set; }
    public float DestructionSpeed { get; set; }
    ...
}
```

我们需要设置这两个属性来接收 Slider 的值。像这样设置

![](http://ojgpkbakj.bkt.clouddn.com/2018081902.png)

除了下方的 On Value Changed 的地方设置好以外还要记得把 Max Value 改成 10，不然拖到最大每秒也只能生成一个就很慢。。。把两个 Slider 分别绑定到`CreationSpeed`和`DestructionSpeed`以后，继续添加代码：

```csharp

public class ReusingDemo : PersistableObject
{
    ...
    public float CreationSpeed { get; set; }
    public float DestructionSpeed { get; set; }

    private float _creationProgress;
    private float _destructionProgress;
    ...

    private void Update()
    {
        ...

        _creationProgress += Time.deltaTime * CreationSpeed;
        while (_creationProgress >= 1f)
        {
            _creationProgress -= 1f;
            CreateObject();
        }

        _destructionProgress += Time.deltaTime * DestructionSpeed;
        while (_destructionProgress >= 1f)
        {
            _destructionProgress -= 1f;
            DestroyObject();
        }
    }
	...
}

```

大概思路。。。不用解释了吧这个代码也太简单了总之就累加到大于 1 就创建或者销毁就这样。。。那么运行一下看看效果吧～

![](http://ojgpkbakj.bkt.clouddn.com/2018081903.gif)



## PART 4 对象池

目前为止我们的准备工作终于完成了。在使用对象池之前，我们先打个包然后观察一下 Profiler 看看。

![](http://ojgpkbakj.bkt.clouddn.com/2018081906.png)

跟之前一样打包并且勾选 Autoconnect Profiler 并且运行起来，把自动生成和销毁速度都调成最高，我们可以观察到每次 Instantiate 对象的时候都会导致 GC Alloc 的产生。因此我们要做的是使用对象池把要被销毁的对象保存起来以便下次再重用从而避免 GC Alloc。

众所周知对象池就是一个“池子”里面放着我们所需要的对象，在需要使用的时候从里面取出，发现取空的时候才创建对象。用完以后通过某个接口把对象重新放回池里。按照这个思路，我们针对`ShapeFactory`进行修改：

```csharp
[CreateAssetMenu]
public class ShapeFactory : ScriptableObject
{
	...
    [SerializeField] private bool _recycle;
    private List<Shape>[] _pools;

    public Shape Get(int shapeId, int materialId)
    {
        Shape instance;
        if (_recycle)
        {
            if (_pools == null)
            {
                CreatePools();
            }

            var pool = _pools[shapeId];
            var lastIndex = pool.Count - 1;
            if (lastIndex >= 0)
            {
                instance = pool[lastIndex];
                instance.gameObject.SetActive(true);
                pool.RemoveAt(lastIndex);
            }
            else
            {
                instance = Instantiate(_prefabs[shapeId]);
                instance.ShapeId = shapeId;
            }
        }
        else
        {
            instance = Instantiate(_prefabs[shapeId]);
            instance.ShapeId = shapeId;
        }

        instance.SetMaterial(_materials[materialId], materialId);
        instance.SetColor(Random.ColorHSV(0f, 1f, 0.4f, 0.6f, 0.7f, 0.9f, 1f, 1f));

        return instance;
    }
	...
	
    public void Reclaim(Shape shapeToRecycle)
    {
        if (_recycle)
        {
            if (_pools == null)
            {
                CreatePools();
            }

            _pools[shapeToRecycle.ShapeId].Add(shapeToRecycle);
            shapeToRecycle.gameObject.SetActive(false);
        }
        else
        {
            Destroy(shapeToRecycle.gameObject);
        }
    }

    private void CreatePools()
    {
        _pools = new List<Shape>[_prefabs.Length];
        for (int i = 0; i < _pools.Length; i++)
        {
            _pools[i] = new List<Shape>();
        }
    }
}

```

简单来说，就是我们为每种形状准备一个对象池，每次调用`Get()`获取一个新的 GameObject 的时候先检查对象池是否存在，确保对象池存在以后将 List 的最后一个对象返回。调用`Reclaim()`将一个正在被使用的对象放回 List 中，并设置`SetActive(false)`。

最后再修改本来销毁对象的地方，改成调用`Reclaim()`即可：

```csharp
public class ReusingDemo : PersistableObject
{    
	...
	private void DestroyObject()
    {
        if (_objectList.Count > 0)
        {
            int index = Random.Range(0, _objectList.Count);
            shapeFactory.Reclaim(_objectList[index]);
            int lastIndex = _objectList.Count - 1;
            _objectList[index] = _objectList[lastIndex];
            _objectList.RemoveAt(lastIndex);
        }
    }

    private void BeginNewGame()
    {
        for (int i = 0; i < _objectList.Count; i++)
        {
            shapeFactory.Reclaim(_objectList[i]);
        }

        _objectList.Clear();
    }
    ...
}
```

完成以后运行一下看看，发现运行一小段时间后不再有新的对象产生，而是通过已创建对象的反复开启和关闭来模拟创建和删除对象。

![](http://ojgpkbakj.bkt.clouddn.com/2018081907.gif)

## PART 5 总结

嗯就这样完成了，话说之前好像有同学提醒说要再写详细一点不知道这一篇够不够详细，再贴一遍非常详细的『[原文链接](https://catlikecoding.com/unity/tutorials/object-management/reusing-objects/)』和我的『[项目地址](https://github.com/sNaticY/CatlikePractice)』方便大家看更详细的教程和下载代码直接跑～如果有需要交流的同学可以直接在我的[博客](https://snatix.com/2018/07/17/023-frames-per-second/)的评论区留言这样我会更容易看到回复也会更快一些～好了不多说了我要继续跟室友联机群星继续我的银河帝国的征程了就酱～

---

原文链接：https://snatix.com/2018/08/19/027-reusing-objects/

本文由 [sNatic](https://github.com/sNaticY) 发布于『[大喵的新窝](https://snatix.com)』 转载请保留本申明