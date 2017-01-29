---
title: Assetbundle(2)-学习Assetbundle Manager
date: 2017-01-29 15:02:53
tags:[assetBundle， unity]
categories: Unity通用框架工程

---

过完年稍微出去浪了一天终于决定重新开始好好学习天天向上了～在上一篇文章『[Assetbundle(1)-初次接触](http://snatix.com/2017/01/15/009-start-with-asset-bundle/)』中简单地讲解了 Assetbundle 最基本的使用方法，那么最基本的使用和项目中稳定的使用是有很大的区别的~有太多细节需要处理。那么今天就来学习一个神器「[AssetBundle Manager](https://www.assetstore.unity3d.com/en/#!/content/45836)」

<!--more-->

## PART 1 概述

AssetBundle Manager 其实是 Unity 官方提供的一个用于开发者方便的管理 Assetbundle 的一个插件。这里有详细的 [官方教程](https://unity3d.com/cn/learn/tutorials/topics/scripting/assetbundles-and-assetbundle-manager) ，但是这篇文章并不打算把教程翻译一遍了事 (其实是英文水平差又懒)。博主准备用自己的方式来详细讲解这个插件的用法，最后合理的把它集成在我们的 Asset Manager 中～毕竟重复造轮子不仅累而且技术含量也高。。那么接下来将会涉及到以下几个方面

1. API介绍
2. 尝试各种方式加载资源和场景
3. Variant 这种高级用法
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



---

原文链接：http://snatix.com/2017/01/29/010-using-assetbundle-manager/

本文由 sNatic 发布于『[大喵的新窝](http://snatix.com)』 转载请保留本申明