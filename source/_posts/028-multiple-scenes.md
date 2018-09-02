---
title: Catlike学习笔记(2.4)-Unity多场景支持
date: 2018-09-02 21:52:48
tags: [unity, 教程, 对象管理, csharp]
categories: Catlike学习笔记
description: Unity多场景切换，Unity运行时创建场景，Unity中移动GameObject到其他场景，Unity同时编辑多个场景
---

Hallo(德语) 大家好我又回来了没错上个礼拜又拖更了～原因是熬夜看 TI8 然而 LGD 居然输了，明明可以拿冠军的，枉费我小绿本充到 170 级，中国队偶数年魔咒已破所以就难过的没有更新文章。。嗯我想这个理由应该非常充分了吧～然后沉迷的『[群星](https://store.steampowered.com/app/281990/Stellaris/)』刚打完一把铁人模式准将难度唯心和平亲外流，种田种了 140 年除了中期危机没跟任何人发生战争，整个银河系因我而充满了和平与爱简直屌爆了～周六又突发奇想去玩了 [VA-11 Hall-A](https://store.steampowered.com/app/447530/VA11_HallA_Cyberpunk_Bartender_Action/) ，初体验来说我还挺喜欢这个令人放松的游戏的。。。

<!--more-->

## PART 1 概述

首先贴上『[原文链接](https://catlikecoding.com/unity/tutorials/object-management/multiple-scenes/)』嗯貌似引言部分硬生生被我写成了游戏推荐环节～所以概述就稍微多讲一些正事。。在『[上一篇](https://cn.snatix.com/2018/08/19/027-reusing-objects/)』中我们完成了对象池，那么在本篇中，我们会为对象池相关的对象在运行时创建一个新的场景，然后添加两个新的场景作为两个关卡，两个关卡有着不同的灯光并且支持一键切换，最后我们需要将关卡信息也保存到存档文件中～除了我们之前已经学习过的内容以外我们会遇到以下新的知识点：

- 在 play mode 中创建场景
- 在不同场景之间移动 GameObject
- 多场景加载，卸载，制作，切换
- 创建关卡

## PART 2 对象池专用场景

首先我们注意到在主场景中加载一大堆 GameObject 以后我们的 Hierarchy 会变得异常混乱，从而很难找到某个特定 GameObject，还会拖慢 Editor 的运行效率就很烦～一个最最直观的解决方案就是把他们统一放到一个父级 GameObject 中，但这样也不是很好因为会多多少少影响我们游戏的运行效率，因此最好可以避免没有意义的嵌套。

那么更好的方法就是把所有的对象池加载出来的相关物体放置到另一个场景中，这样的话不需要嵌套也可以在编辑器里折叠起来，而且也不会降低运行效率。那么该怎么做呢？

### 运行时创建场景并放入对象

因为对象池是运行时创建的，因此我们同样需要在运行时创建一个新场景来放置对象池生成的物体，那么我们在`ShapeFactory`中添加如下代码：

```csharp
[CreateAssetMenu]
public class ShapeFactory : ScriptableObject
{
    ...
    private Scene _poolScene;

    public Shape Get(int shapeId, int materialId)
    {
        Shape instance;
        if (_recycle)
        {
            ...
            if (lastIndex >= 0)
            {
                ...
            }
            else
            {
                ...
                SceneManager.MoveGameObjectToScene(instance.gameObject, _poolScene);
            }
        }
        ...
    }

    private void CreatePools()
    {
        ...
        _poolScene = SceneManager.CreateScene(name);
    }
}
```

以上代码想必大家都很清楚，大概就是在创建对象池最后顺便也创建一个新场景，然后在创建新对象的时候放入该场景

### 重编译后恢复

虽然博主觉得这个功能好像没什么用不过还是先照做比较好～简单来说就是目前的功能可以正常运行了，但是在运行状态下修改代码重新编译的话就会造成一些问题，因为 Unity 会在编译的时候序列化所有的`MonoBehaviour`，然而`ScriptableObject`却不会被序列化，也就是说我们的对象池`List`在重新编译后就会丢失引用，会导致`CreatePool()`再次被执行，从而导致一系列问题。于是我们需要修改如下代码：

```csharp
[CreateAssetMenu]
public class ShapeFactory : ScriptableObject
{
    ...

    private void CreatePools()
    {
        _pools = new List<Shape>[_prefabs.Length];
        for (int i = 0; i < _pools.Length; i++)
        {
            _pools[i] = new List<Shape>();
        }

        if (Application.isEditor)
        {
            _poolScene = SceneManager.GetSceneByName(name);
            if (_poolScene.isLoaded)
            {
                GameObject[] rootObjects = _poolScene.GetRootGameObjects();
                for (int i = 0; i < rootObjects.Length; i++)
                {
                    Shape pooledShape = rootObjects[i].GetComponent<Shape>();
                    if (!pooledShape.gameObject.activeSelf)
                    {
                        _pools[pooledShape.ShapeId].Add(pooledShape);
                    }
                }

                return;
            }
        }
        _poolScene = SceneManager.CreateScene(name);
    }
}
```

需要注意的这种情况仅会在 Editor 中发生，因此我们需要用`Application.isEditor`来判断是否是 Editor 环境。然后获取场景并判断其是否已被加载，如果是的话就将其跟节点的所有 GameObject 加入`_pools`中这样～然后我们在运行时即使修改代码也可以直接重新编译继续运行而不会报错了！（说实话博主以前写代码从来没考虑过这个）

## PART 3 LEVEL 1

通常来说场景并不只是用来做上面这样的用来容纳 GameObject 这样的事情，更常见的做法是每一关一个场景。不过在游戏中我们经常会遇到某些不属于任何场景的对象，在这种情况下我们可以选择将此类对象放置在一个单独的场景。

### 多场景编辑

首先我们创建一个新的场景`Level 1`，创建好以后只需要把场景拖到 Hierarchy 中就可以同时编辑多个场景，如图所示。

![](http://ojgpkbakj.bkt.clouddn.com/2018090201.png)

然后我们需要删除 Level 1 中的 Main Camera 和主场景中的 Directional Light。运行一下会发现场景里的物体莫名其妙的变黑了～这主要是因为每个场景都有自己的 lighting settings，最终结果取决于我们使用哪个场景的 lighting settings。如图所示：

![](http://ojgpkbakj.bkt.clouddn.com/2018090202.png)

解决方案是右键点击 Level 1 再选择 Set Active Scene。再运行就发现恢复正常了～如图所示

![](http://ojgpkbakj.bkt.clouddn.com/2018090203.png)

### 加载场景

然而打包后，只有 index 0 会被自动加载，因此我们需要手动加载场景。添加如下代码：

```csharp
public class MultiSceneDemo: PersistableObject
{
    ...
    private void Awake()
    {
        _objectList = new List<Shape>();
        StartCoroutine(LoadLevel());
    }

   	...
   	
    private IEnumerator LoadLevel() 
    {
        SceneManager.LoadScene("Level 1", LoadSceneMode.Additive);
        yield return null;
        SceneManager.SetActiveScene(SceneManager.GetSceneByName("Level 1"));
    }
}

```

需要注意的是`LoadLevel()`中的`yield return null`，主要原因是在加载场景后需要等待一帧才能完全加载好。但是运行一下会发现，即使我们已经调用`SetActiveScene()`但是环境光还是有问题，尽管打包以后是没有问题的，但是在 Editor 中运行时加载场景时，自动生成 lighting data 会出问题～那么该如何解决呢？

### Lighting 烘培

为了确保光照数据被正常生成，我们需要取消 Auto Generate 选项，首先打开 Window / Lighting / Settings

![](http://ojgpkbakj.bkt.clouddn.com/2018090204.png)

然后打开 Level 1 场景，点击 Generate Lighting 后 Unity 会烘培光照数据并且保存在场景所在的文件夹中。

![](http://ojgpkbakj.bkt.clouddn.com/2018090205.png)

此时再运行会发现一切正常～

### 异步加载场景

加载场景所需时间取决于场景本身的大小，在我们现在的场景就只有一个平行光因此就加载很快～但是一般来说时间都会稍长一些，会导致这段时间游戏卡顿。为了解决这个问题我们需要异步加载场景的技术～代码如下：

```csharp
public class MultiSceneDemo: PersistableObject
{
    ...

    private IEnumerator LoadLevel()
    {
    	enabled = false;
        yield return SceneManager.LoadSceneAsync("Level 1", LoadSceneMode.Additive);
        SceneManager.SetActiveScene(SceneManager.GetSceneByName("Level 1"));
        enabled = true;
    }
}
```

只需要替换`LoadLevel()`中加载场景的代码为`SceneManager.LoadSceneAsync()`即可，同时为了防止在加载过程中玩家发送各种指令导致各种问题，我们需要把`component`临时关闭。

### 防止多次加载

虽然目前为止看起来好像是正常工作了，但是如果我们在游戏开始之前就加载两个场景就会出现奇怪的问题，就是场景被打开两次导致光线太明亮～虽然这种情况只有 Editor 模式中会发生但是我们还是要处理一下～

![](http://ojgpkbakj.bkt.clouddn.com/2018090206.png)

修改代码如下：

```csharp
public class MultiSceneDemo : PersistableObject
{
	...
    // private void Awake()
    private void Start()
    {
        _objectList = new List<Shape>();

        if (Application.isEditor) {
            Scene loadedLevel = SceneManager.GetSceneByName("Level 1");
            if (loadedLevel.isLoaded) {
                SceneManager.SetActiveScene(loadedLevel);
                return;
            }
        }

        StartCoroutine(LoadLevel());
    }
    ...
}

```

注意我们将原来的`Awake()`中的内容移动到`Start()`中，主要原因是在`Awake()`时场景并不会被标记为加载完成，因此在`Awake()`中判断是不行的，所以我们稍微延迟一点点放到`Start()`中就可以解决这个问题了～

## PART 4 更多关卡

某些游戏只有一个关卡，不过大部分游戏都有多个关卡，所以我们现在尝试再创建一个关卡并且可以来回切换～首先我们把 Level 1 复制一下改名 Level 2 然后把平行光的角度 x 从 50 改成 1 最后在点击 Generate Lighting。

### 检测已加载的关卡

虽然我们有可能需要同时打开多个关卡，不过一般来说同一时间只打开一个关卡的可能性更大一些～某些情况下我们可能需要同时打开多个场景进行修改之类或者复制粘贴之类的～但是一旦进入 Play Mode 我们还是希望除了主场景外只打开一个场景。如果我们在开始游戏之前就打开了 Level 2 的话就会又同时打开 Level 1。。所以为了防止这种情况发生，我们需要在`Start()`中检测关卡。代码如下：

```csharp
public class MultiSceneDemo : PersistableObject
{
    ...

    private void Start()
    {
        _objectList = new List<Shape>();

        if (Application.isEditor)
        {
            for (int i = 0; i < SceneManager.sceneCount; i++) {
                Scene loadedScene = SceneManager.GetSceneAt(i);
                if (loadedScene.name.Contains("Level ")) {
                    SceneManager.SetActiveScene(loadedScene);
                    return;
                }
            }
        }

        StartCoroutine(LoadLevel());
    }
}
```

这样的话我们就可以在任意关卡打开的时候开始游戏了～（好像是很有用的功能）

### 加载特定关卡

作为一个小游戏我们就简单的把数字键 1 和 2 作为切换场景的按键～稍微修改`LoadLevel()`即可实现，代码如下

```csharp
public class MultiSceneDemo : PersistableObject
{
    ...
    public int LevelCount;
	
	...
    private void Update()
    {
        ...
        else
        {
            for (int i = 1; i <= LevelCount; i++)
            {
                if (Input.GetKeyDown(KeyCode.Alpha0 + i))
                {
                    BeginNewGame();
                    StartCoroutine(LoadLevel(i));
                    return;
                }
            }
        }
        ...
    }
	...
    private IEnumerator LoadLevel(int levelBuildIndex)
    {
        enabled = false;
        if (_loadedLevelBuildIndex > 0)
        {
            yield return SceneManager.UnloadSceneAsync(_loadedLevelBuildIndex);
        }
        yield return SceneManager.LoadSceneAsync(levelBuildIndex, LoadSceneMode.Additive);
        SceneManager.SetActiveScene(SceneManager.GetSceneByBuildIndex(levelBuildIndex));
        _loadedLevelBuildIndex = levelBuildIndex;
        enabled = true;
    }
}

```

需要注意的是我们在加载每个场景后记录该场景的 Index，在下次加载场景时记得把之前加载的场景卸载掉～

### 保存关卡

目前为止我们可以在游戏过程中切换场景了，但是无法保存关卡本身，因此我们就可以在某个关卡保存 GameObject 然后再另一个关卡加载，这样可能会出现问题，所以还是把关卡的 index 也写入存档中比较好～

为了支持之前版本的存档文件，我们再修改存档文件的版本号为 2，然后将当前场景的 index 写入存档～如果版本号低于 2 则默认加载场景 Level 1。代码如下

```csharp
public class MultiSceneDemo : PersistableObject
{
	...
    private int _saveVersion = 2;
	...

    public override void Save(GameDataWriter writer)
    {
        writer.Write(_objectList.Count);
        writer.Write(_loadedLevelBuildIndex);
        ...
    }

    public override void Load(GameDataReader reader)
    {
        var version = reader.Version;
        if (version > _saveVersion)
        {
            Debug.LogError("Unsupported future save version " + version);
            return;
        }

        int count = version <= 0 ? -version : reader.ReadInt();
        StartCoroutine(LoadLevel(version < 2 ? 1 : reader.ReadInt()));
        ...
    }
	
	...
}

```

最终效果如下～首先切换到场景 1，创建一些对象以后切换到场景 2，再按下`l`读取存档，可以自动切换回场景 1 并加载对象，测试通过～

![](http://ojgpkbakj.bkt.clouddn.com/2018090302.gif)

## PART 5 总结

嗯本来以为这一篇结束以后『[对象管理](https://snatix.com/tags/%E5%AF%B9%E8%B1%A1%E7%AE%A1%E7%90%86/)』系列就告一段落了没想到作者又出了『[新一篇](https://catlikecoding.com/unity/tutorials/object-management/spawn-zones/)』～好吧那么只好下周再把对象管理系列完结了～嗯就这样决定了，我要继续在『[VA-11 Hall-A](https://store.steampowered.com/app/447530/VA11_HallA_Cyberpunk_Bartender_Action/)』中当一名善解人意的调酒师了大家拜拜～

---

原文链接：https://snatix.com/2018/09/02/028-multiple-scenes/

本文由 [sNatic](https://github.com/sNaticY) 发布于『[大喵的新窝](https://snatix.com)』 转载请保留本申明