---
title: Assetbundle(1)-初次接触
date: 2017-01-15 15:01:25
tags: [asset bundle, unity]
categories: Unity通用框架工程

---

这理论上应该是一个冗长无比的系列，但是既然决定开这个深坑了就要好好填～其实一直以来都希望有一套自己熟悉的框架来自行维护，有什么想法的时候拿起来就能写，同时也有机会接触到 Unity 的方方面面而不是只会拼 UI 写业务系统。不管是 ulua 还是 slua，目的就是通过 AssetBundle 来更新，因此我们首先从 Assetbundle 开始。

<!--more-->

## PART 1 概述

其实随便搜一搜就会发现关于 Unity5.x 的 Assetbundle 的文章超多，但是很多都是在结合 Unity4 的 Assetbundle 来对比着讲的，或者是讲的稍微有点深，某些很基础很的概念还没有普及到就开始使用了，看着很不爽。经过一小段时间的研究以后终于有了一些感觉，那么我来尝试用我自己的语言来向零基础的初次接触 Assetbundle 的同学们从头开始讲～

1. 什么是 Assetbundle，为什么要使用这个鬼东西
2. 尝试随便写点代码用一下看看效果
3. 据说可以自动处理依赖，试一试

不知道为什么今天的文风比较随意，大家将就看吧~

## PART 2 Assetbundle 介绍

其实我很讨厌长篇大论的写东西来介绍某个概念，一是读起来太麻烦，二是我也写不出来。但是考虑到完整性，介绍一下又是不得不做的事情，我就不复制那些随便一搜一大堆的东西了，大概用自己的语言讲一下，讲的不对大家就当没看见就好了或者在评论区稍微提醒我修改～真想了解精准定义和概念的请移步「[官方文档](https://docs.unity3d.com/Manual/AssetBundlesIntro.html)」

### 什么是 Assetbundle

Asset 大家都知道是什么其实就是游戏用到的模型，贴图，场景，Prefab之类的资源。那么 Assetbundle 顾名思义就是把这些东西打成一个 Bundle。。。所以其实就是 Unity 官方提供了一种机制允许开发者把这些游戏资源打包放在自己的服务器，然后运行时再把 bundle 下载下来，把 Asset 解出来再使用。

### 为什么要用 Assetbundle

首先就是可以减少包体大小，有些公司的产品很在意这些因为某商店的安装包超过 100M 时就默认不允许玩家使用流量下载，因此可以把很大的模型场景贴图之类的移出去，然后玩家安装好游戏以后在游戏里继续下载直到全部资源都下载完毕才可以开始玩。

其次就是更新资源了，贴图模型和 Prefab 全都可以更新，用上 [slua](https://github.com/pangweiwei/slua) 或 [tolua](https://github.com/topameng/tolua) 的话连代码都可以更新，麻麻再也不怕我写出 bug 了~而且可以热更新的话就不需要每次修复 bug 或添加新特性都要更整包了有效提高玩家的用户体验。

大概的概念介绍到此结束，想了解更多的同学请自行翻阅「[官方文档](https://docs.unity3d.com/Manual/AssetBundlesIntro.html)」，俗话说 Talk is cheap, show me the code。所以我们接下来就用代码说话。

## PART 3 简单的尝试 Assetbundle

首先做实验的第一步是生成 Assetbundle，那么新建工程新建场景脚本什么不说了直接观察目录结构

```tex
|---AssetManager	//Unity工程目录
	|---Assets	//资源目录
	|	|---AssetManager	//插件目录(博主最终想要把整个作成插件的形式)
	|		|---Editor		//Editor脚本放置目录
	|		|---Examples	//一大堆示例
	|		|	|---Prefabs
	|		|	|---Scenes
	|		|	|---Textures
	|		|
	|		|---Scripts		//插件脚本
	|	
	|---Bundls	//放置生成的AssetBundle的目录
```

### 写一句生成 Assetbundle 的代码

那么如何生成 Assetbundle 呢？超级简单，核心就一个函数`BuildPipeline.BuildAssetBundles`。

```csharp
using UnityEditor;
namespace AssetManager
{
    public class Menu
    {
        [MenuItem("AssetManager/Build AssetBundles")]
        public static void BuildAssetBundles()
        {
            BuildPipeline.BuildAssetBundles("outputPath", BuildAssetBundleOptions.None, BuildTarget.StandaloneOSXUniversal);
        }
    }
}
```

在`AssetManager/Editor`下放置该文件就可以了。简单的说这样做可以让标题栏出现一个菜单`AssetManager/Build AssetBundles`，点一下就会自动生成 Assetbundle 了。是不是很方便，第一个参数是输出的路径，如我们项目结构的规划放置在与`Assets`同级的`Bundles`目录下，不出意外的话这个目录以后应该会反复用到，为了方便以后扩展最好做一个静态类 config 作为配置。

```csharp
using UnityEngine;
namespace AssetManager
{
    public static class Config
    {
        public static string ABRootName = "Bundles";
        public static string ABOutputPath = Application.dataPath + "/../" + ABRootName;
        public static string ABServerRootPath = "http://192.168.31.40/";
    }
}
```

这样就可以把第一段代码中的`"outputPath"`替换成`Config.ABOutputPath`了。那么生成的代码写完了，下一步就是做个 prefab 生成一下 Assetbundle 试试~

### 随便制作一个 Prefab 做实验

制作 Prefab 的过程不多说了，大概就是随便放一张图在`Example/Textures`然后在做一个把一个带有 Sprite 的 GameObject 做成 Prefab，再把图片拖进去就行了，像是这样

![制作一个Prefab](http://ojgpkbakj.bkt.clouddn.com/2017012101.png)

为了等一下观察生成的 Assetbundle 的大小，我特意放了一张很大的图（博主的壁纸，在 Unity 中显示大小为 2.2M ）。最后把 Prefab 放在了`Example/Prefabs`文件夹，设置`Assetbundle Name`为`prefab_1`。这样这一步就完成了。

### 生成一个 Assetbundle 并放在服务器上

第一步顺利完成的话，应该会看到菜单栏出现了名为`AssetManager`的菜单，点击`AssetManager/Build AssetBundles`，等待几秒钟就生成好了。

![生成中](http://ojgpkbakj.bkt.clouddn.com/2017012102.png)

生成好了以后应该可以在`Bundles`这个文件夹下看到四个文件，观察一下发现名为`prefab_1`的文件很大大约有1.4M，而`Bundles`这个文件却很小只有不到1K。为什么一个 Prefab 打成的 Assetbundle 会这么大呢？答案是这个 Bundle 里包含了 Prefab 依赖的全部资源，因此那张很大的壁纸也被包含在里面了。我们把两个不带`.manifest`后缀的文件上传到自己的 http 服务器上确保用浏览器可以访问到。

启动 Http 服务的方法很多，博主在局域网内另一台 Ubuntu 上某个文件夹直接运行以下命令：

``` bash
$ sudo python -m SimpleHTTPServer 80
```

就可以在当前文件夹下开启 Http 服务了，mac 想模拟的话大家自行寻找方法。上传文件也不说了博主就随便开启了个 ftp 就传了反正只是做实验。下一步是写一些代码把这个 Prefab 加载进来。

### 加载 Assetbundle 中的 Prefab

新建一个场景`Example_1`，保存在`Example/Scenes`中，再新建一个 GameObject 名为`Example_1`，新建一个脚本名为`Example_1`，保存在`Example/Scripts`中，再挂在上一步创建好的 Prefab 上，开始写代码。

首先思路是使用`www`把这个文件下载下来，再把里面的内容 Load 进来，最后在场景中 Instanciate 这个 Prefab。实现这一套思路最简单的代码是这样的。

```csharp
using System.Collections;
using UnityEngine;

public class Example_1 : MonoBehaviour
{
    void Start()
    {
        StartCoroutine(LoadPrefabFromAssetBundle());
    }

    IEnumerator LoadPrefabFromAssetBundle()
    {
        WWW www = new WWW( Config.ABServerRootPath + "prefab_1");
        yield return www;

        GameObject prefab = www.assetBundle.LoadAsset<GameObject>("prefab_1");
        GameObject.Instantiate(prefab);
    }
}
```

运行一下，等待几秒后成功的把该显示的加载出来了~（可能有10秒）图就不截了大家应该都想象得出来，就跟上面制作的一模一样。到此为止我们的 Assetbundle 最基础的用法已经掌握了。

## PART 4 自动依赖管理

为什么会有依赖的问题呢？大家应该都发现了，把一个 Prefab 打成 Assetbundle 是会在生成的 Assetbundle 文件中包含所有依赖的资源，那么如果博主的壁纸在两个 Prefab 中使用呢？简单测试一下会发现生成的两个 Assetbundle 大小都约为1.4M。那么就会对玩家的流量造成极大的浪费，而且加载速度也会变慢。所以我们需要把他们依赖的公共资源单独打包至一个 Assetbundle 中。好的现在开始动手试试～

首先放一张新的图在`Examples/Textures`下随便放两张完全一样的贴图命名为`texture_2`和`texture_3`，然后设置`texture_2`的 AssetBundle Name 为`texrure_2`，`texture_3`不进行任何设置作为对照组，然后做两个 prefab 分别引用这张图片，再给 prefab 分别设置 AssetBundle Name 为 `prefab_2`和`prefab_3`。最后生成一下，好的具体步骤就不演示了~生成结果如下图所示

![生成结果](http://ojgpkbakj.bkt.clouddn.com/2017012601.png)

大家可以观察到生成的名为`prefab_2`的文件大小为 2KB ，小于`prefab_3`的 3KB，这充分表明他所依赖的资源已经被分离出去了因此打好的 Assetbundle 大小会变小。那么如果用之前加载 prefab 的方法把 prefab_2 加载出来会发生什么事情呢？简单的修改代码尝试一下会发现，结果如图所示

![丢失Sprite](http://ojgpkbakj.bkt.clouddn.com/2017012602.png)

我们的 Sprite 丢失了~那么正确的处理方式是怎样呢？答案是要先把他所依赖的 Assetbundle 加载好。那么如何获取依赖的 Assetbundle 呢？

```csharp
IEnumerator LoadDependenciesAndPrefab()
{
	WWW www = new WWW(Config.ABServerRootPath + Config.ABRootName);
	yield return www;

	AssetBundleManifest manifest = www.assetBundle.LoadAsset<AssetBundleManifest>("AssetBundleManifest");

	string[] dependenceBundleName = manifest.GetAllDependencies("prefab_2");
	foreach (string name in dependenceBundleName)
	{
    	www = new WWW(Config.ABServerRootPath + name);
    	yield return www;
    	www.assetBundle.LoadAllAssets();
  	}

  	www = new WWW(Config.ABServerRootPath + "prefab_2");
  	yield return www;

 	GameObject prefab = www.assetBundle.LoadAsset<GameObject>("prefab_2");
	GameObject.Instantiate(prefab);
}
```

在`Start()`里开启一个协程调用以上函数试试～这应该是可以把一层依赖关系的 Assetbundle 解出来的最直观的代码了，不管好不好看至少可以非常清晰的理解思路。事实上不管多层依赖都只需要递归地把所有有依赖关系的 AssetBundle 用同样地方法解出来最后加载你想要的 Prefab 就可以了。

## PART 5 总结

这一篇写的非常基础也非常水，其实这也算是博主自己探索 Assetbundle 的过程的一个笔记～希望可以对像博主一样完全初次接触 Assetbundle 的同学有帮助。文章配套的工程已经放在博主的 [github](https://github.com/sNaticY/unity-asset-manager) 了。接下来就要随着学习的进行一步一步的构建我们的插件，希望最终可以实现配置好路径就可以完成自动设置 Assetbundle Name，生成和上传 Assetbundle，下载和加载资源等等操作而不需要过多的关注这些细节。不知道最终多久可以完成，希望不会太久远～加油！

---

原文链接：https://snatix.com/2017/01/15/009-start-with-asset-bundle/

本文由 [sNatic](https://github.com/sNaticY) 发布于『[大喵的新窝](https://snatix.com)』 转载请保留本申明