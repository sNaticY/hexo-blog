---
title: Catlike学习笔记(2.1)-保存和加载GameObject
date: 2018-08-05 16:40:48
tags: [unity, 教程, 对象管理, csharp]
categories: Catlike学习笔记
description: 在Unity中随机生成GameObject并将数据保存至 PersistentData 并加载。
---

终于放假回国了～然而热到爆炸几乎不能出门，充了B站大会员在家看了花了两天补完「[博人传 火影忍者新时代](https://www.bilibili.com/bangumi/media/md5978/?from=search&seid=7643644285500463201)」和「[齐木楠雄的灾难](https://www.bilibili.com/bangumi/media/md8812/?from=search&seid=4918824649219054592)」第二季，今天终于要开始努力学习了嗯。。。那么今天来到了「[Catlike教程](https://catlikecoding.com/unity/tutorials/)」的第二部分——[对象管理](https://catlikecoding.com/unity/tutorials/object-management/)，那么这是一系列与创建，追踪，保存和加载对象的教程。今天就不多说了快速进入正题～

<!--more-->

## PART 1 概述

既然做要保存和加载 GameObject 那么我们肯定要先做生成的功能，生成完以后保存下来，然后清除掉，再把之前保存的内容加载出来这样～所以我们的任务目标是以下这些：

- 一键生成随机方块并一键清除
- 保存 GameObject 的状态写入文件
- 加载已经保存的数据重新生成 GameObject
- 重构一下将保存与加载抽象成独立模块

## PART 2 完成游戏逻辑

我们的需求非常简单，大概就是按下一个按键就生成一个小方块在随机位置旋转和缩放，然后按下某个按键就可以清空场景内所有方块。那么首先我们随便创建一个方块的 Prefab，然后创建脚本名为`PersistentDemo`然后添加如下代码。

```csharp
public class PersistentDemo : MonoBehaviour
{
	public Transform Prefab;
	public KeyCode CreateKey = KeyCode.C;

	private List<Transform> _objectList;

	private void Awake()
	{
		_objectList = new List<Transform>();
	}

	private void Update()
	{
		if (Input.GetKeyDown(CreateKey))
		{
			CreateObject();
		}
	}

	private void CreateObject()
	{
		Transform t = Instantiate(Prefab);
		t.localPosition = Random.insideUnitSphere * 5f;
		t.localRotation = Random.rotation;
		t.localScale = Vector3.one * Random.Range(0.1f, 1f);
		_objectList.Add(t);
	}
}
```

代码特别简单大概就是在`Update()`中检测按键，然后在随机位置生成一个随机旋转和大小的方块，然后加到`ObjectList`中。写好以后我们在场景中创建一个空 GameObject 取名叫`Game`然后挂上该脚本，再把之前制作好的 Cube Prefab 拖到脚本中。运行一下看看～

![picture](http://ojgpkbakj.bkt.clouddn.com/2018080601.gif)

嗯效果不错～然后我们需要设置一个快捷键可以一键清除所有方块以便重新开始生成。那么继续添加如下代码。大概就是检测到玩家按下按键后就遍历`ObjectList`中的所有 GameObject 并 Destroy，最后清空`ObjectList`。

```csharp
using System.Collections.Generic;
using UnityEngine;

public class PersistentDemo : MonoBehaviour
{
	...
	public KeyCode NewGameKey = KeyCode.N;
	...
	
	private void Update()
	{
		...
		else if (Input.GetKey(NewGameKey))
		{
			BeginNewGame();
		}
	}

	...

	private void BeginNewGame()
	{
		for (int i = 0; i < ObjectList.Count; i++)
		{
			Destroy(ObjectList[i].gameObject);
		}

		ObjectList.Clear();
	}
}
```

运行效果就不截图了总之就是所有的小方块都消失了～那么我们现在就完成了第一步。

## PART 3 保存和读取

保存和读取的思路非常简单，我们在这里就不用 PlayerPref 之类的东西，而是采取更简单易懂的直接在 PersistentData 目录中创建一个文件把 GameObject 的信息写在里面就好了。那么事实上我们需要保存的数据就只有方块的数量以及每个方块其各自的位置，旋转和大小。那么我们尝试在`PersistentDemo`中添加如下代码。

```csharp
public class PresistentDemo : MonoBehaviour
{
	...
	public KeyCode SaveKey = KeyCode.S;

	private List<Transform> _objectList;
	private string _savePath;

	private void Awake()
	{
		_objectList = new List<Transform>();
		_savePath = Path.Combine(Application.persistentDataPath, "saveFile");
	}

	private void Update()
	{
		...
		else if (Input.GetKeyDown(SaveKey))
		{
			Save();
		}
	}

	...

	private void Save()
	{
		using (var writer = new BinaryWriter(File.Open(_savePath, FileMode.Create)))
		{
			writer.Write(_objectList.Count);
			for (int i = 0; i < _objectList.Count; i++)
			{
				Transform t = _objectList[i];
				writer.Write(t.localPosition.x);
				writer.Write(t.localPosition.y);
				writer.Write(t.localPosition.z);
			}
		}
	}
}
```

代码内容也非常简单，大概就是检测按键后在预设好的路径中创建文件，然后写入当前创建的方块的数量，并依次写入每个方块的`Position`。这样一来读取的代码也呼之欲出了，大概就是从预设路径的文件中取出方块的数量最后按照相应的位置信息生成 GameObject 

```csharp
public class PresistentDemo : MonoBehaviour
{
	...
	public KeyCode LoadKey = KeyCode.L;
	...

	private void Update()
	{
		...
		else if (Input.GetKeyDown(LoadKey)) {
			Load();
		}
	}

	...
	private void Load()
	{
		BeginNewGame();
		using (var reader = new BinaryReader(File.Open(_savePath, FileMode.Open)))
		{
			int count = reader.ReadInt32();
			for (int i = 0; i < count; i++)
			{
				Vector3 p;
				p.x = reader.ReadSingle();
				p.y = reader.ReadSingle();
				p.z = reader.ReadSingle();
				Transform t = Instantiate(Prefab);
				t.localPosition = p;
				_objectList.Add(t);

			}
		}
	}
}
```

运行一下看看效果～在这里博主特意把按键指令也显示在屏幕左下角方便大家看清楚发生了什么～

![picture](http://ojgpkbakj.bkt.clouddn.com/2018080602.gif)

那么大家会注意到，最后按下`l`的时候，所有的小方块的旋转和缩放信息都不见了，只剩下位置还是正确的，那么讲道理我们可以继续修改代码把旋转和缩放也写到文件里面，不过这样代码会变得异常丑陋，我们稍微重构一下代码再把旋转和缩放补全吧。

## PART 4 抽象与重构

那么该如何抽象呢，大概的思路就是先创建一个`Writer`和`Reader`可以让我们方便的从`Binary`中读出`Vector3`和`Quaternion`，然后创建`PersistableObject`挂载在我们的 Prefab 上，可以使外部方便的调用`Save()`和`Load()`接口就可以把 Prefab 中的所有重要信息，如 Position Rotation Scale 等保存或读取出来。最后我们从`PresistentDemo`把保存和读取相关的代码提取出来单独作为一个`PersistentStorage`类，由`PresistentDemo`调用。那么方案确定下来以后就开始实施～

### Writer 和 Reader

Writer 和 Reader 的作用就是允许我们方便的调用一个接口就可以把相应比较复杂的数据结构写入到 Binary 中或从中读取，从而避免大量的重复的类似`writer.Write(t.localPosition.x)`这样的代码。这两部分代码非常相似而且很简单，就不多解释了随便贴一下。。。。

```csharp
public class GameDataReader
{
	private BinaryReader _reader;

	public GameDataReader(BinaryReader reader)
	{
		_reader = reader;
	}

	public float ReadFloat()
	{
		return _reader.ReadSingle();
	}

	public int ReadInt()
	{
		return _reader.ReadInt32();
	}

	public Quaternion ReadQuaternion()
	{
		Quaternion value;
		value.x = _reader.ReadSingle();
		value.y = _reader.ReadSingle();
		value.z = _reader.ReadSingle();
		value.w = _reader.ReadSingle();
		return value;
	}

	public Vector3 ReadVector3()
	{
		Vector3 value;
		value.x = _reader.ReadSingle();
		value.y = _reader.ReadSingle();
		value.z = _reader.ReadSingle();
		return value;
	}
}
```

```csharp
public class GameDataWriter
{
	private BinaryWriter _writer;

	public GameDataWriter(BinaryWriter writer)
	{
		_writer = writer;
	}

	public void Write(float value)
	{
		_writer.Write(value);
	}

	public void Write(int value)
	{
		_writer.Write(value);
	}

	public void Write(Quaternion value)
	{
		_writer.Write(value.x);
		_writer.Write(value.y);
		_writer.Write(value.z);
		_writer.Write(value.w);
	}

	public void Write(Vector3 value)
	{
		_writer.Write(value.x);
		_writer.Write(value.y);
		_writer.Write(value.z);
	}
}
```

### Persistable Object

接下来`PersistableObject`的作用是挂载在 Perfab 上从而使得外部可以简单的通过`Save()`和`Load()`接口来将一个对象的所有数据一次性的保存或读取出来。后续如果我们不同种类的游戏对象需要保存和读取的数据更复杂的话就可以继承这个类并重写相关接口来实现而无需改动外部调用代码，不过这都是后话了，目前我们需要保存的就是`localPosition`，`localRotation`和`localScale`这样。所以代码如下

```csharp
[DisallowMultipleComponent]
public class PersistableObject : MonoBehaviour
{
	public virtual void Save(GameDataWriter writer)
	{
		writer.Write(transform.localPosition);
		writer.Write(transform.localRotation);
		writer.Write(transform.localScale);
	}

	public virtual void Load(GameDataReader reader)
	{
		transform.localPosition = reader.ReadVector3();
		transform.localRotation = reader.ReadQuaternion();
		transform.localScale = reader.ReadVector3();
	}
}

```

是不是这样组织代码比在 `PresistentDemo`中实现所有功能要清晰很多呢，而且很容易扩展和修改～最后不要忘记挂在我们的 Prefab 上面。

### Persistent Storage

最后`PersistentStorage`存在的意义是将文件操作相关代码从主逻辑中剥离出来，并没有很多内容。。。

```csharp
public class PersistentStorage : MonoBehaviour
{
	private string _savePath;

	void Awake()
	{
		_savePath = Path.Combine(Application.persistentDataPath, "saveFile");
	}

	public void Save(PersistableObject o)
	{
		using (var writer = new BinaryWriter(File.Open(_savePath, FileMode.Create)))
		{
			o.Save(new GameDataWriter(writer));
		}
	}

	public void Load(PersistableObject o)
	{
		using (var reader = new BinaryReader(File.Open(_savePath, FileMode.Open)))
		{
			o.Load(new GameDataReader(reader));
		}
	}
}
```

将其挂在 Game 上后修改`PresistentDemo`，完整代码如下～注意我们 Override 的`Save()`和`Load()`函数部分。以及按下 SaveKey 和 LoadKey 后调用的`Storage.Save(this)`和`Storage.Load(this)`

```csharp
public class PresistentDemo : PersistableObject
{
	public PersistableObject Prefab;

	public KeyCode CreateKey = KeyCode.C;
	public KeyCode NewGameKey = KeyCode.N;
	public KeyCode SaveKey = KeyCode.S;
	public KeyCode LoadKey = KeyCode.L;

	private List<PersistableObject> _objectList;

	public PersistentStorage Storage;

	private void Awake()
	{
		_objectList = new List<PersistableObject>();
	}

	private void Update()
	{
		if (Input.GetKeyDown(CreateKey))
		{
			CreateObject();
		}
		else if (Input.GetKey(NewGameKey))
		{
			BeginNewGame();
		}
		else if (Input.GetKeyDown(SaveKey))
		{
			Storage.Save(this);
		}
		else if (Input.GetKeyDown(LoadKey))
		{
			BeginNewGame();
			Storage.Load(this);
		}
	}

	private void CreateObject()
	{
		PersistableObject o = Instantiate(Prefab);
		var t = o.transform;
		t.localPosition = Random.insideUnitSphere * 5f;
		t.localRotation = Random.rotation;
		t.localScale = Vector3.one * Random.Range(0.1f, 1f);
		_objectList.Add(o);
	}

	private void BeginNewGame()
	{
		for (int i = 0; i < _objectList.Count; i++)
		{
			Destroy(_objectList[i].gameObject);
		}
		_objectList.Clear();
	}

	public override void Save(GameDataWriter writer)
	{
		writer.Write(_objectList.Count);
		for (int i = 0; i < _objectList.Count; i++)
		{
			_objectList[i].Save(writer);
		}
	}

	public override void Load(GameDataReader reader)
	{
		int count = reader.ReadInt();
		for (int i = 0; i < count; i++)
		{
			PersistableObject o = Instantiate(Prefab);
			o.Load(reader);
			_objectList.Add(o);
		}
	}
}
```

最后把各种东西引用都拖好大概像这样～

![picture](http://ojgpkbakj.bkt.clouddn.com/2018080603.png)

最后运行下看看～

![picture](http://ojgpkbakj.bkt.clouddn.com/2018080604.gif)



## PART 5 总结

持续一边划水玩手机一边吃零食一边写文章花了两天终于完成了～自己回顾下来感觉代码有点多讲的不够细，但是都是非常简单的代码呀相信各位同学可以轻轻松松搞定的～嗯明天就开始下一篇！

---

原文链接：https://snatix.com/2018/08/05/025-persisting-objects/

本文由 [sNatic](https://github.com/sNaticY) 发布于『[大喵的新窝](https://snatix.com)』 转载请保留本申明