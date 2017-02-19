---
title: Assetbundle(2)-学习Assetbundle Manager
date: 2017-01-29 15:02:53
tags: [asset bundle, unity]
categories: Unity通用框架工程

---

过完年稍微出去浪了一天终于决定重新开始好好学习天天向上了～在上一篇文章『[Assetbundle(1)-初次接触](http://snatix.com/2017/01/15/009-start-with-asset-bundle/)』中简单地讲解了 Assetbundle 最基本的使用方法，那么最基本的使用和项目中稳定的使用是有很大的区别的~有太多细节需要处理。那么今天就来学习一个神器「[AssetBundle Manager](https://www.assetstore.unity3d.com/en/#!/content/45836)」

<!--more-->

## PART 1 概述

AssetBundle Manager 其实是 Unity 官方提供的一个用于开发者方便的管理 Assetbundle 的一个插件。这里有详细的 [官方教程](https://unity3d.com/cn/learn/tutorials/topics/scripting/assetbundles-and-assetbundle-manager) ，但是这篇文章并不打算把教程翻译一遍了事 (其实是英文水平差又懒)。博主准备用自己的方式来详细讲解这个插件的用法，最后合理的把它集成在我们的 Asset Manager 中～毕竟重复造轮子不仅累而且技术含量也高。。那么接下来将会涉及到以下几个方面

1. API介绍
2. 尝试各种方式加载资源和场景
3. Variants 这种高级用法
4. 一些好玩的小功能

## PART 2 API 介绍

虽然博主是行动派喜欢自己动手试试，但是简单的介绍还是需要的，因为没太多可以讲的博主决定就抄官方文档了。。

> The AssetBundle Manager’s API includes:
>
> - ***Initialize()*** Initializes the AssetBundle manifest object.
> - ***LoadAssetAsync()*** Loads a given asset from a given AssetBundle and handles all the dependencies.
> - ***LoadLevelAsync()*** Loads a given scene from a given AssetBundle and handles all the dependencies.
> - ***LoadDependencies()*** Loads all the dependent AssetBundles for a given AssetBundle.
> - ***BaseDownloadingURL*** Sets the base downloading url which is used for automatic downloading dependencies.
> - ***SimulateAssetBundleInEditor*** Sets Simulation Mode in the Editor.
> - ***Variants*** Sets the active variant.
> - ***RemapVariantName()*** Resolves the correct AssetBundle according to the active variant.

大概就是讲有以下 API：

* **Initialize()** 初始化 AssetBundle Manifest 对象
* **LoadAssetAsync()** 从指定的一个 AssetBundle 中加载指定资源并处理所有的依赖
* **LoadLevelAsync()** 从指定的一个 AssetBundle 中加载指定场景并处理所有的依赖
* **LoadDependencies()** 加载指定的 AssetBundle 的所有依赖 AssetBundle
* **BaseDownloadingURL** 设置用来自动下载依赖的基本地址
* **SimulateAssetBundleInEditor** 在编辑器中设置模拟模式
* **Variants** 设置当前激活的 Variant
* **RemapVariantName()** 根据当前的变体决定正确的 AssetBundle

Api 似乎非常简洁正式我想要的，不知道用起来是不是很复杂呢？Asset Store 里下载下来该 [插件](https://www.assetstore.unity3d.com/en/#!/content/45836) 会自带一个 Demo 大家可以根据需求自行参考～

## PART 3 尝试加载资源和场景

接下来必然要在我们的 AssetManager 工程中来使用这个插件了，首先我们需要在 Asset store 中找到这个 [插件](https://www.assetstore.unity3d.com/en/#!/content/45836) 。

![Asset Store](http://ojgpkbakj.bkt.clouddn.com/2017012901.png)

作为一个洁癖当然不会随便的把 Example 和 README 什么的导入进来了～想看的话可以到另一个工程里看一看

![导入](http://ojgpkbakj.bkt.clouddn.com/2017012902.png)

完成以后就把整个文件夹移动到 AssetManager 文件夹下让他成为我们插件的一部分～(蜜汁觉得好像哪里不太对不过算了先这样吧)。因为这次的主要目的是使用这个插件所以我们暂时就不做任何修改了，原汁原味的使用～

### 使用插件生成一次 AssetBundle

因为之前我们已经做过一次依赖测试所以资源就不需要再添加了，不明白的同学请先看「[上一篇](https://snatix.com/2017/01/15/009-start-with-asset-bundle/)」，直接点击菜单`Assets/AssetBundles/Build AssetBundles`就可以生成想要的 AssetBundle 了。如图所示，插件在我们 Asset 同级目录下生成了`AssetBundles/OSX`目录并把生成好的 Bundle 放在里面。

![AssetBundles](http://ojgpkbakj.bkt.clouddn.com/2017012903.png)

按照相同的目录结构把生成的文件放在 http 服务器上这样准备工作就完成了~记得把`Asset/AssetBundles/Simulation Mode`关掉不然就只是模拟模式。。。

### 加载资源

直接新建一个场景 `example_3` 再新建一个脚本`Example_3.cs`就可以了开始测试了。如以下代码

```csharp
IEnumerator LoadDependenciesAndPrefab()
{
    AssetBundleManager.SetSourceAssetBundleURL(Config.ABServerRootPath);
    yield return AssetBundleManager.Initialize();

    AssetBundleLoadAssetOperation request = AssetBundleManager.LoadAssetAsync("prefab_2", "prefab_2", typeof(GameObject));
    yield return StartCoroutine(request);

    GameObject prefab = request.GetAsset<GameObject>();
    GameObject.Instantiate(prefab);
}
```

直接在`Start`函数中开启协程调用以上函数，发现已经可以了～不需要处理依赖什么的，就好像异步加载本地资源一样非常方便。

### 加载场景

首先制作一个场景，制作的步骤不说了大概就是从 Asset Store 里随便下载一个免费的场景，然后把必要的资源移动到我们规划好的文件夹里- - 最后设置好 AssetBundle Name 就好了。最终效果大概是下图这样

![场景完成](http://ojgpkbakj.bkt.clouddn.com/2017020101.png)

那么资源的具体位置不多说了总之就是按照资源的类型分别放在`Materials`,`Modles`,`Prefabs`,`Scenes`,`Textures` 等文件夹里。需要注意的是，场景文件不可以和普通的资源文件设置相同的 AssetBundle Name，因此我们把场景文件最后生成一下 Assetbundle 看看～

![生成的文件](http://ojgpkbakj.bkt.clouddn.com/2017020102.png)

生成好的文件如上图所示，`scene_4`很小只有100K左右，查看其 manifest 文件可以得出以下依赖关系（没装 visio 之类的随便用一个 vscode 插件画了一下）

![依赖关系](http://ojgpkbakj.bkt.clouddn.com/2017020103.png)

那么这种复杂的依赖关系是不是处理起来很麻烦呢~答案显然是否。。使用 AssetBundle Manager 可以轻松搞定，新建一个空场景拖一个空 GameObject 然后挂上一个名叫 Example_4 的脚本，注意看以下代码。

```csharp
IEnumerator LoadDependenciesAndScene()
{
	AssetBundleManager.SetSourceAssetBundleURL(Config.ABServerRootPath);
	yield return AssetBundleManager.Initialize();

	AssetBundleLoadOperation request = AssetBundleManager.LoadLevelAsync("scene_4", "CalibrationScene", false);
	yield return StartCoroutine(request);
}
```

同样啰嗦了很多遍的在`Start`函数里开启一个协程调用以上代码就可以加载出来场景了（可能要等待10秒钟或更久）是不是很好很强大～效果为了提高网页加载速度就不贴那么多图了跟之前的差不多～大家自己试试吧。

## PART 4 Variants 的用法

至于 Variants 到底是什么和具体使用场景就不细说了毕竟博主也没真正在项目中使用这个东西，欲了解精准概念请移步「[官方文档](https://docs.unity3d.com/Manual/BuildingAssetBundles.html)」，但是据博主观察大多数例子都举相同资源的不同质量的贴图，hd，sd之类的。以之前加载的场景为例，最终的贴图的依赖位于整个依赖链的最后，假设我们要准备两个不同贴图分辨率的场景的话，可能要从头到尾很多东西包括 material，mode 和 prefab 什么的都要重新制作一遍，造成了极大的浪费，否则就要写代码去动态加载要额外处理很多事情。那么现在 Assetbundle 就提供了这样的一个后缀一样的参数，允许你设置当前的 Variants 然后再去加载场景，这样就可以前面所有的东西都一样，只为两套不同分辨率的贴图制作两个 AssetBundle，加载的时候就根据当前的 Variants 把所需要的贴图所在的 Assetbundle 下载并 Load 所需要的贴图了。废话说了不少可能具体怎么操作大家还是不太明白，简单的尝试一下吧

首先在`Examples/Textures/`下建立两个文件夹命名不一样就行，然后把两个**相同名称**的资源分别放进去设置**相同的** AssetBundle Name，然后分别设置**不同的** Variants （以"a"，"b"为例）。最后制作 prefab 引用其中任何一个资源即可，最后制作场景添加脚本什么的就不想细说了。重点是观察以下代码

```csharp
IEnumerator LoadDependenciesAndScene()
{
	AssetBundleManager.SetSourceAssetBundleURL(Config.ABServerRootPath);
    AssetBundleManager.ActiveVariants = new string[] {"b"};
    yield return AssetBundleManager.Initialize();

    AssetBundleLoadAssetOperation request = AssetBundleManager.LoadAssetAsync("prefab_5", "prefab_5", typeof(GameObject));
    yield return StartCoroutine(request);

    GameObject prefab = request.GetAsset<GameObject>();
    GameObject.Instantiate(prefab);
}
```

需要注意的就只有`AssetBundleManager.ActiveVariants = new string[] {"b"};`这一行，其他的都是复制粘贴过来的几乎完全一样。。自己尝试该代码运行会发现加载进来的 prefab 所依赖的贴图会随着这里填写不同的 Variants 自动切换。而制作 prefab 时完全无需考虑这一点，也不用再写其他的代码加载，非常方便的样子～

## PART 5 一些有趣的小功能

### Simulation Mode

最后来探索以下 Assetbundle Manager 还有什么有趣的小功能吧，还记得之前让大家关闭的`Assets/AssetBundles/Simulation Mode`的功能么？那我们打开会如何呢？是的大家会发现即使你关掉我们用来下载 Assetbundle 的 http 服务也可以正常加载，而且无需重新生成 bundle 就可以更新资源。这极大的简化了我们的开发流程，不需要额外的资源管理机制就可以迅速看到 Assetbundle 中的改动效果而无需先重新生成一遍再放到服务器上。

### Local AssetBundle Server

只是在本地直接模拟还是不够满意的，就是想模拟远程下载怎么办，只需要开启`Assets/AssetBundles/Local AssetBundle Server`就好了，因为博主电脑是 mac 各种麻烦的原因开启的时候总会报错可能要安装高版本的 mono 或者有其他的解决方案，下次找到了方法再补充，所以现在也不知道到底有多好用～Windows 上的大家应该用起来爽爽的~

## PART 6 总结

到此为止关于神奇的插件 AssetBundle Manager 就基本上介绍完了，总的来说是一个非常方便的插件，一口气帮助我们解决了许多细节问题而且感觉非常靠谱的样子。下一步就是完成一个自动设置 AssetBundle Name 的功能了，然后针对插件进行一些魔改了。不知道多久可以做好，等思路差不多的时候就开始更新下一篇！

---

原文链接：http://snatix.com/2017/01/29/010-using-assetbundle-manager/

本文由 sNatic 发布于『[大喵的新窝](http://snatix.com)』 转载请保留本申明