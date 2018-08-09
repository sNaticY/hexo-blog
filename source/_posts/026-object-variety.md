---
title: Catlike学习笔记(2.2)-为不同颜色的GameObejct开启GPU Instancing
date: 2018-08-08 16:40:48
tags: [unity, 教程, 对象管理, shader, csharp]
categories: Catlike学习笔记
description: 使用 Unity 生成并保存 GameObject 并为不同形状和颜色的 GameObject 开启 GPU Instancing
---

20180808 过生日一大早就收到超级大红包开心～中午难得跟家人一起过生日所以晚上才开始写文章，今天来到了 [Object Management](https://catlikecoding.com/unity/tutorials/object-management/) 的第二篇 Object variety，在「[上一篇](https://snatix.com/2018/08/05/025-persisting-objects/)」的基础上继续扩展，需要我们的存档文件支持保存不同形状和材质以及颜色的对象，并且在完成一系列功能后尝试开启 GPU Instancing 对比性能开销差异。

<!--more-->

## PART 1 概述

首先我们要对之前已经完成的工程进行一系列改进，当然如果只对 GPU Instancing 感兴趣的同学可以直接跳到 PART 5，我们需要逐步支持保存和加载不同形状，不同材质以及不同颜色的 GameObject，最终可以对比开启 GPU Instancing 以后的差异～所以大概有以下任务

- 生成不同形状的 GameObject
- 支持保存和加载不同形状
- 支持多种材质和随机颜色
- 开启 GPU Instancing

## PART 2 生成不同形状的 GameObject

那么为了实现这个需求我们需要继承在「[上一篇](https://snatix.com/2018/08/05/025-persisting-objects/)」文章中实现的`PersistableObject`类，在上一篇中就有提到

> 接下来`PersistableObject`的作用是挂载在 Perfab 上从而使得外部可以简单的通过`Save()`和`Load()`接口来将一个对象的所有数据一次性的保存或读取出来。后续如果我们不同种类的游戏对象需要保存和读取的数据更复杂的话就可以继承这个类并重写相关接口来实现而无需改动外部调用代码

那么在这里因为多了一个定义形状的属性所以我们可以放心的继承这个类并且无需对以前的代码进行修改，为了符合「开闭原则」什么的就不提了～总之创建如下代码：

```csharp
public class Shape : PersistableObject
{
}
```

新的`Shape`里面可以暂时什么都不需要添加等到我们需要加载和保存不同形状的时候再做。接下来我们创建一些各种形状的 Prefab 然后给他们挂上这个脚本，比如 Cube, Sphere 和 Capsule 像这样的三个 Prefab ～

![](http://ojgpkbakj.bkt.clouddn.com/2018080801.png)

然后我们还需要一个`ShapeFactory`类从而让我们可以通过调用一个接口生成不同的形状：

```csharp
using UnityEngine;

[CreateAssetMenu]
public class ShapeFactory : ScriptableObject
{
    [SerializeField] Shape[] prefabs;
    
    public Shape Get (int shapeId) {
        return Instantiate(prefabs[shapeId]);
    }
    
    public Shape GetRandom () {
        return Get(Random.Range(0, prefabs.Length));
    }
}
```

这个代码就很简单了大概就是 Instantiate 指定 Id 或者随机 Id 的形状，需要注意的是`CreateAssetMenu`选项可以让我们在 Project 窗口中任意文件夹点击右键创建这样的一个`ScriptableObject`。如果不是非常理解`ScriptableObject`的话大家可以简单的认为这个跟 Prefab 差不多只不过没有 GameObject 只有脚本中的各种属性被保存其中～更详细的解释大家可以参见「[官方文档](https://docs.unity3d.com/ScriptReference/ScriptableObject.html)」

然后我们右键创建一个`Shape Factory`然后把我们之前创建的 Prefab 拖进去如下图～

![](http://ojgpkbakj.bkt.clouddn.com/2018080802.png)

![](http://ojgpkbakj.bkt.clouddn.com/2018080803.png)

最后我们还需要稍微对之前写的`PersistentDemo`修改一下，博主为了方便在同一个工程中可以看到每一期的代码就新建一份名为`VarietyDemo`内容如下：

```csharp
public class VarietyDemo : PersistableObject
{
	public ShapeFactory shapeFactory;
	
	...

	private void CreateObject()
	{
		Shape instance = shapeFactory.GetRandom();
		var t = instance.transform;
		t.localPosition = Random.insideUnitSphere * 5f;
		t.localRotation = Random.rotation;
		t.localScale = Vector3.one * Random.Range(0.1f, 1f);
		_objectList.Add(instance);
	}

	private void BeginNewGame()
	{
		...
	}

	public override void Save(GameDataWriter writer)
	{
		...
	}

	public override void Load(GameDataReader reader)
	{
		int count = reader.ReadInt();
		for (int i = 0; i < count; i++)
		{
			Shape instance = shapeFactory.Get(0);
			instance.Load(reader);
			_objectList.Add(instance);
		}
	}
}
```

大家可以注意到，代码与昨天的`PersistentDemo`相差无无几，只有创建对象和加载的函数内部稍作修改，引用到了我们刚才写好的`ShapeFactory`和其函数`Get()`以及`GetRandom()`。那么此时我们运行一下会发现～

![](http://ojgpkbakj.bkt.clouddn.com/2018080804.png)

虽然可以随机生成了但是我们只是保存了位置旋转和缩放信息，并没有保存具体是哪种形状，因此保存后再加载会发现所有的形状全都变成了 Cube。那么我们接下来该如何解决这个问题呢？

## PART 3 支持保存和加载不同形状

那么首先为了支持旧版本的存档，也就是只有位置旋转和缩放信息的保存文件，需要引入版本号的概念～那么我们假设现在的版本号是1，我们需要做的是把版本号写到文件头部，加载存档的时候也先加载出来。所以修改`VarietyDemo`如下

```csharp
public class VarietyDemo : PersistableObject
{
	public int SaveVersion = 1;
	
	...

	public override void Save(GameDataWriter writer)
	{
		writer.Write(-SaveVersion);
		writer.Write(_objectList.Count);
		...
	}

	public override void Load(GameDataReader reader)
	{
		var version = -reader.ReadInt();
		if (version > SaveVersion) {
			Debug.LogError("Unsupported future save version " + version);
			return;
		}
		int count = version <= 0 ? -version : reader.ReadInt();
		...
	}
}
```

因为我们之前的版本写入的是保存的对象的数量，因此为了方便区分版本号与数量，我们将版本号乘以`-1`以后再写入，这样就可以通过读出来的第一个数据是否大于`0`来确定是否是之前不带有版本号的版本，从而决定时候读取下一个 int 作为保存的对象的数量。那么有了版本号，接下来我们该把物体的形状保存下来～首先在`Shape`中添加以下代码

```csharp
public class Shape : PersistableObject
{
    private int _shapeId = int.MinValue;
    
    public int ShapeId
    {
        get { return _shapeId; }
        set
        {
            if (_shapeId == int.MinValue)
            {
                _shapeId = value;
            }
            else
            {
                Debug.LogError("Not allowed to change shapeId.");
            }
        }
    }
}
```

总之就是在`Shape`中保存`ShapeId`，而且这个 Id 只能被赋值一次～完成以后继续修改`VarietyDemo`让我们每次在保存的时候把`ShapeId`写入文件，读取的时候根据`ShapeId`读取不同形状的 Prefab，代码如下

```csharp
public class VarietyDemo : PersistableObject
{
	...
	private List<Shape> _objectList;
	
	...
	
	public override void Save(GameDataWriter writer)
	{
		writer.Write(-SaveVersion);
		writer.Write(_objectList.Count);
		for (int i = 0; i < _objectList.Count; i++)
		{
			writer.Write(_objectList[i].ShapeId);
			_objectList[i].Save(writer);
		}
	}

	public override void Load(GameDataReader reader)
	{
		var version = -reader.ReadInt();
		if (version > SaveVersion) {
			Debug.LogError("Unsupported future save version " + version);
			return;
		}
		int count = version <= 0 ? -version : reader.ReadInt();
		for (int i = 0; i < count; i++)
		{
			var shapeId = version <= 0 ? 0 : reader.ReadInt();
			Shape instance = shapeFactory.Get(shapeId);
			instance.Load(reader);
			_objectList.Add(instance);
		}
	}
}
```

需要注意的是读取`ShapeId`之前先检查版本号，如果是负的版本号就表示我们并没有保存形状，所以就按照默认值 0 来处理～完成后我们运行一下看看，为了验证多版本支持功能正常运作，我们可以先运行一次上一篇文章创建的场景，保存一次再在新版本中载入试试。

![](http://ojgpkbakj.bkt.clouddn.com/2018080805.gif)

大家可以看到～我首先载入了旧版本的存档文件，生成出来的全是立方体，然后再按 C 创建了很多新的球体等其他形状的物体，保存后再加载，原来的立方体还是立方体，但是新的球体和胶囊体都保存下来了～

## PART 4 支持多种材质和随机颜色

在支持多材质和颜色之前，我们需要先重构一下存档版本的部分，因为方便起见我们最好可以在`Shape`类内部访问到要加载的存档版本，否则就被迫在`VarietyDemo`里面写处理所有版本相关的事情，非常的不直观。重构好以后我们就可以添加多个材质，在`ShapeFactory`中与形状一起处理，然后再在`Shape`中按照版本信息读取颜色，大概思路就是这样。

### 重构版本管理部分

，所以先修改`GameDataReader`使其在初始化的时候携带当前读取存档文件的版本号。当然还顺便加上了读颜色的接口方便后续操作。

```csharp
public class GameDataReader
{
    public int Version { get; private set; }
    private BinaryReader _reader;

    public GameDataReader(BinaryReader reader, int version)
    {
        _reader = reader;
        Version = version;
    }
	...
        
    public Color ReadColor () {
        Color value;
        value.r = _reader.ReadSingle();
        value.g = _reader.ReadSingle();
        value.b = _reader.ReadSingle();
        value.a = _reader.ReadSingle();
        return value;
    } 
}
```

然后再修改`PersistentStorage`让我们每次保存的时候都先写入版本号，以及每次读取的时候都优先把版本号读出来。

```csharp
public class PersistentStorage : MonoBehaviour
{
    ...
    public void Save(PersistableObject o, int version)
    {
        using (var writer = new BinaryWriter(File.Open(_savePath, FileMode.Create)))
        {
            writer.Write(-version);
            o.Save(new GameDataWriter(writer));
        }
    }

    public void Load(PersistableObject o)
    {
        using (var reader = new BinaryReader(File.Open(_savePath, FileMode.Open)))
        {
            o.Load(new GameDataReader(reader, -reader.ReadInt32()));
        }
    }
}
```

最后在`VarietyDemo`做相应的修改，大概在调用`Storage.Svae()`的时候把当前的版本号传入，然后去掉在保存时写入的版本号直接写对象数量，最后在`Load()`时可以从`reader.Version`读出版本号而无需自行`ReadInt()`

```csharp
public class VarietyDemo : PersistableObject
{
    ...

    private void Update()
    {
        ...
        else if (Input.GetKeyDown(SaveKey))
        {
            Storage.Save(this, _saveVersion);
        }
        else if (Input.GetKeyDown(LoadKey))
        {
            BeginNewGame();
            Storage.Load(this);
        }
    }
	...

    public override void Save(GameDataWriter writer)
    {
        writer.Write(_objectList.Count);
        ...
    }

    public override void Load(GameDataReader reader)
    {
        var version = reader.Version;
        ...
    }
}
```

总之一顿操作下来重构完成，运行一下试试有没有问题～博主这边表示完全正常。

### 添加多材质和多颜色

有了前面的铺垫这一步就很简单。那么首先我们需要在`GameDataReader`和`GameDataWriter`中分别添加读写`Color`的接口，从而我们可以在`Shape`中读写`Color`对象。

```csharp
public class GameDataReader
{
	...
    public Color ReadColor () {
        Color value;
        value.r = _reader.ReadSingle();
        value.g = _reader.ReadSingle();
        value.b = _reader.ReadSingle();
        value.a = _reader.ReadSingle();
        return value;
    }
}
```

```csharp

public class GameDataWriter
{
	...
    public void Write (Color value) {
        _writer.Write(value.r);
        _writer.Write(value.g);
        _writer.Write(value.b);
        _writer.Write(value.a);
    }
}

```

接下来在`Shape`中添加设置材质`SetMaterial()`和设置颜色`SetColor()`接口。从而可以在`ShapeFactory`中生成一个对象的时候为其设置随机材质和颜色。同时我们还需要 override `Save()`和`Load()`两个接口，从而读取和保存位于「位置」「旋转」和「缩放」之后的颜色属性。

```csharp

public class Shape : PersistableObject
{
	...

    public int MaterialId { get; private set; }
    public Color Color { get; private set; }

    public void SetMaterial(Material material, int materialId)
    {
        GetComponent<MeshRenderer>().material = material;
        MaterialId = materialId;
    }

    public void SetColor(Color color)
    {
        GetComponent<MeshRenderer>().material.color = color;
        Color = color;
    }
    
    public override void Save(GameDataWriter writer)
    {
        base.Save(writer);
        writer.Write(Color);
    }

    public override void Load(GameDataReader reader)
    {
        base.Load(reader);
        SetColor(reader.Version <= 0 ? Color.white : reader.ReadColor());
    }
}

```

然后创建三个材质球看起来不一样即可，博主按照「[原文链接](https://catlikecoding.com/unity/tutorials/object-management/object-variety/)」的指示选择了 Shandard Shiny Metallic，其中 Standard 就是默认的新建材质球，Shiny 是默认材质球把 Smoothness 拉到 0.9，Metallic 就是把 Metallic 和 Smoothness 同时拉到 0.9。创建好以后修改`ShapeFactory`如下~

```csharp

[CreateAssetMenu]
public class ShapeFactory : ScriptableObject
{
    [SerializeField] private Shape[] _prefabs;

    [SerializeField] private Material[] _materials;

    public Shape Get(int shapeId, int materialId)
    {
        var instance = Instantiate(_prefabs[shapeId]);
        instance.ShapeId = shapeId;
        instance.SetMaterial(_materials[materialId], materialId);
        return instance;
    }

    public Shape GetRandom()
    {
        var instance = Get(Random.Range(0, _prefabs.Length), Random.Range(0, _materials.Length));
        instance.SetColor(Random.ColorHSV(0f, 1f, 0.4f, 0.6f, 0.7f, 0.9f, 1f, 1f));
        return instance;
    }
}

```

注意我们在`Get()`中添加一个参数并在`Instantiate()`后调用`SetMaterial()`用于设置材质球，以及在`GetRandom()`中调用的`SetColor()`从而设置随机颜色之类的～最后相应的修改`VarietyDemo`如下～

```csharp

public class VarietyDemo : PersistableObject
{
    ...

    public override void Save(GameDataWriter writer)
    {
        writer.Write(_objectList.Count);
        for (int i = 0; i < _objectList.Count; i++)
        {
            writer.Write(_objectList[i].ShapeId);
            writer.Write(_objectList[i].MaterialId);
            _objectList[i].Save(writer);
        }
    }

    public override void Load(GameDataReader reader)
    {
        ...
        for (int i = 0; i < count; i++)
        {
            var shapeId = version <= 0 ? 0 : reader.ReadInt();
            var materialId = version <= 0 ? 0 : reader.ReadInt();
            Shape instance = shapeFactory.Get(shapeId, materialId);
            instance.Load(reader);
            _objectList.Add(instance);
        }
    }
}
```

全部完成后我们尝试运行一下～因为代码跨越多个文件如果不能马上理顺其中的关系的话可以尝试自行实现一遍应该会很快理解～

![](http://ojgpkbakj.bkt.clouddn.com/2018080901.gif)

## PART 5 开启 GPU Instancing

那么开启 GPU Instancing 很简单只需要在 Material 中勾选 GPU Instancing 就可以了～就像下面这样。。

![](http://ojgpkbakj.bkt.clouddn.com/2018080902.png)

所以勾选了这个真的有效果么？

![](http://ojgpkbakj.bkt.clouddn.com/2018080903.png)

![](http://ojgpkbakj.bkt.clouddn.com/2018080904.png)

那么我们注意到开启了 GPU Instancing 以后 Batches 从 402 降低到了 219，但事实上我们还有很多改进空间。因为默认的 Shadard Shader 中 Unity 只会将只有 Transform 组件不同的 GameObject 进行合批处理，因此改变颜色会导致该机制失效，那么我们首先创建一个支持将颜色声明为 instanced property 的 shader 使其支持不同颜色的 GameObject。代码如下~

```c++
Shader "Custom/InstancedColors" {
	Properties {
		_Color ("Color", Color) = (1,1,1,1)
		_MainTex ("Albedo (RGB)", 2D) = "white" {}
		_Glossiness ("Smoothness", Range(0,1)) = 0.5
		_Metallic ("Metallic", Range(0,1)) = 0.0
	}
	SubShader {
		Tags { "RenderType"="Opaque" }
		LOD 200

		CGPROGRAM
		#pragma surface surf Standard fullforwardshadows
		#pragma instancing_options assumeuniformscaling

		#pragma target 3.0

		sampler2D _MainTex;

		struct Input {
			float2 uv_MainTex;
		};

		half _Glossiness;
		half _Metallic;

		UNITY_INSTANCING_BUFFER_START(Props)
			UNITY_DEFINE_INSTANCED_PROP(fixed4, _Color)
		UNITY_INSTANCING_BUFFER_END(Props)

		void surf (Input IN, inout SurfaceOutputStandard o) {
			fixed4 c = tex2D (_MainTex, IN.uv_MainTex) *
				UNITY_ACCESS_INSTANCED_PROP(Props, _Color);
			o.Albedo = c.rgb;
			o.Metallic = _Metallic;
			o.Smoothness = _Glossiness;
			o.Alpha = c.a;
		}
		ENDCG
	}
	FallBack "Diffuse"
}

```

使用以上 Shader 时，我们需要使用[`MaterialPropertyBlock`](http://docs.unity3d.com/Documentation/ScriptReference/MaterialPropertyBlock.html)将 _Color 属性的变化告诉 Unity 使其可以将这些 GameObject 置入同一个 draw call 中。修改`Shape`代码如下

```csharp
public class Shape : PersistableObject
{
    ...
    private MeshRenderer _meshRenderer;

    private static int _colorPropertyId = Shader.PropertyToID("_Color");
    private static MaterialPropertyBlock _sharedPropertyBlock;

    void Awake () {
        _meshRenderer = GetComponent<MeshRenderer>();
    }

    public void SetMaterial(Material material, int materialId)
    {
        _meshRenderer.material = material;
        MaterialId = materialId;
    }

    public void SetColor(Color color)
    {
        if (_sharedPropertyBlock == null) {
            _sharedPropertyBlock = new MaterialPropertyBlock();
        }
        _sharedPropertyBlock.SetColor(_colorPropertyId, color);
        _meshRenderer.SetPropertyBlock(_sharedPropertyBlock);
        Color = color;
    }
	...
}
```

最后再运行一下看看～效果显著！从 219 又降到 55 非常神奇～

![](http://ojgpkbakj.bkt.clouddn.com/2018081001.png)

## PART 6 总结

又是持续两天写完的文章(~~实际上有三天因为现在已经过 12 点了~~)。。不管怎么说完成这篇文章以后博主自己还是稍微有一点点收获的，就是关于 GPU Instancing 的最最基础的使用的部分。这篇真的好长而且很多功能横跨几个文件，没太理顺的同学可以跳转「[Github项目地址](https://github.com/sNaticY/CatlikePractice)」下载工程亲自跑起来试一下会更容易理解～希望在假期结束之前也就是8月15号之前可以顺利完成「对象管理」的部分，加油～

---

原文链接：https://snatix.com/2018/08/08/026-object-variety/

本文由 [sNatic](https://github.com/sNaticY) 发布于『[大喵的新窝](https://snatix.com)』 转载请保留本申明