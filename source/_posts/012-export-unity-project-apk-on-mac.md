---
title: Unity自动化(1)-在Mac上手动打包Apk
date: 2017-03-12 22:41:48
tags: [unity, mac, android]
categories: Unity通用框架工程

---

失踪人口终于回归啦！好久没写文章了，因为找工作跳槽搬家入职适应环境什么的各种事情耽搁了好几周没抽出时间来静下心来研究一些东西。因为新项目中工作环境比较原始，打包 apk 和 ipa 都需要完全手动导出，甚至有些文件还需要自己手动复制很是麻烦，有些怀念之前用 jenkins 动动手指在任意电脑打开浏览器添加一系列任务吃个饭回来 30 个包就 OK 的日子，所以突然很想要把所有的一切自动化起来。

<!--more-->

## PART 1 概述

虽说我们的目标是完全自动化，但第一节课还是从入门开始(其实就是偷懒想少写一些内容)，也就是先把手动打包的流程整理清楚，因为考虑到如果想要同一台电脑想要同时打包 apk 和 ipa 所以我们的选择就只有 mac，mac 上打包 ipa 很常见了，那么如何在 mac 上为 unity 打包一个 apk 呢。其实坑还是挺多的，当然以下提到的一切都建立在可以「科学上网」的基础上。其实步骤很简单

1. 安装 JDK
2. 安装 Android SDK
3. 安装 NDK
4. 使用 Android 虚拟机验证

## PART 2 安装 JDK

博主使用的是 Unity5.5.0f3 的版本，其实配置起来还是比较方便的。当然前提是我们已经安装 Unity Android Support以后，切换到 Android Platform 打开 Unity Preference 以后可以看到需要配置的地方都提供了`Download`按钮，点击一下就可以跳转到具体的官网页面下载了当然如果没有「科学上网」的帮助的话很可能这些页面完全打不开或者下载速度不忍直视。

![配置环境](http://ojgpkbakj.bkt.clouddn.com/2017031201.png)

首先我们第一步是下载 JDK，点击对应的「Download」就可以调转到 [ORACLE官网下载页](http://www.oracle.com/technetwork/java/javase/downloads/index.html) 进行下载了。貌似没有太多复杂的东西好说，找到 JDK 也就是 Java SE Development Kit 的下载页进入以后选择好相应的平台下载，最后一路安装就没问题了，英语水平正常的话完全没有任何要注意的坑所以就不截图了。安装好一般来说路径就跟博主图里的路径是差不多的。

最后在 Terminal 里输入`java -version`试试有版本号信息的话出现就表示安装成功了。最后确认好位置填入到 Unity Prefrences 里面 JDK 一项就好了。

## PART 3 安装 Android Studio 

这一步是安装 Android SDK，目前来说安装 Android SDK 的一般方式就是安装好 Android Studio 然后打开 SDK Manager 来管理所有的 Android SDK，当然也可以直接下载好放在某个位置设置路径就行了但是博主不喜欢来历不明比较奇怪的东西，毕竟以后要接渠道 SDK 也用得到 Android Studio 顺便安装一个也省心。

 点击 Unity Prefrences 中 SDK 项后面的`Download`进入 [Android Studio 官方下载页](https://developer.android.com/sdk/index.html#Other) 直接点击下载即可

![官方下载页](http://ojgpkbakj.bkt.clouddn.com/2017031202.png)

这个官网还挺好看的就截一张图吧。一路安装好以后首次打开会帮你下载好最新的 Andriod SDK 和 SDK Tools 之类的东西，很方便。最后在 Unity Prefrences 中配置路径就好了就跟博主的还是差不多的路径。

什么这样就结束了么？当然会有坑的不然这篇文章也没必要写了。提示 NDK 如果不用 IL2CPP 的话留白就好了，但这时候我们点击`Build`会发现并不能正常出包，大概会报如下的错

```text
CommandInvokationFailure: Unable to list target platforms. Please make sure the android sdk path is correct. See the Console for more details. 
/Library/Java/JavaVirtualMachines/jdk1.8.0_121.jdk/Contents/Home/bin/java -Xmx2048M -Dcom.android.sdkmanager.toolsdir="/Users/snatic/Library/Android/sdk/tools" -Dfile.encoding=UTF8 -jar "/Applications/Unity/PlaybackEngines/AndroidPlayer/Tools/sdktools.jar" -

stderr[
Error:Invalid command android
]
stdout[

]
exit code: 64
UnityEditor.Android.Command.Run (System.Diagnostics.ProcessStartInfo psi, UnityEditor.Android.WaitingForProcessToExit waitingForProcessToExit, System.String errorMsg)
UnityEditor.Android.AndroidSDKTools.RunCommandInternal (System.String javaExe, System.String sdkToolsDir, System.String[] sdkToolCommand, Int32 memoryMB, System.String workingdir, UnityEditor.Android.WaitingForProcessToExit waitingForProcessToExit, System.String errorMsg)
UnityEditor.Android.AndroidSDKTools.RunCommandSafe (System.String javaExe, System.String sdkToolsDir, System.String[] sdkToolCommand, Int32 memoryMB, System.String workingdir, UnityEditor.Android.WaitingForProcessToExit waitingForProcessToExit, System.String errorMsg)
UnityEditor.HostView:OnGUI()

```

那么是哪里出了问题呢，博主在这个地方头疼好久最终找到了解决方案，就是回到刚才的  [Android Studio 官方下载页](https://developer.android.com/sdk/index.html#Other) 拉到底部，找到「仅获取命令行工具」的地方选择对应平台的下载，解压后复制到 SDK目录下，例如`/Users/snatic/Library/Android/sdk`，再替换原来的`tools`文件夹。

最后设置环境变量，在 Terminal 中输入

```bash
export ANDROID_SDK_HOME="/Users/snatic/Library/Android/sdk"
```

需要注意的是，下次打开 Android Studio 的时候可能会提示 SDK Tools 有更新，点击忽略即可。不然下次打包又会悲剧的重新替换一遍不要问我怎么知道的。。。

## PART 4 安装 NDK

不了解 IL2CPP 或者只打算用 mono 的同学到目前位置应该差不多可以打包了，如果想要使用 IL2CPP 的话我们还需要下载 NDK，需要注意的是我们并不能使用 Android Studio 中的 SDK Manager 来安装 NDK。需要按照 Unity Prefrence 的指引，我们会得到一个名为`android-ndk-r10e-darwin-x86_64.bin`的文件，然而这个文件如何使用呢？首先进入到该文件所在目录，输入以下命令更改文件的读写权限

```bash
chmod a+x android-ndk-r10e-darwin-x86_64.bin
```

再运行以下命令执行该文件

```bash
./android-ndk-r10e-darwin-x86_64.bin
```

这样我们就会得到一个文件夹就是我们所需要的 NDK 了，把它移动到喜欢的目录就好了，博主将它和 Android SDK 放在一起了，最后在 Unity Prefrences 中填写路径，完成配置，可以使用 IL2CPP 打包 APK 了。

## PART 5 使用 Android Studio 自带的虚拟机

生成 APK 文件以后当然是想要迫不及待的验证一下到底对不对了，那么如何创建 Android 虚拟机呢？首先打开 Android Studio 随便新建一个工程如图所示。

![Android Studio](http://ojgpkbakj.bkt.clouddn.com/2017031203.png)

依次打开`Tools -> Android -> AVD Manager`，按照指引选择机型后下载对应版本的系统镜像文件并创建虚拟机，最后点击绿色箭头运行即可。

![Android Studio](http://ojgpkbakj.bkt.clouddn.com/2017031204.png)

最后把生成好的 APK 拖到虚拟机中即可安装运行了，随便搭一个场景写点代码试一试效果吧~

## PART 6 总结

好像这么结束的确有点水不过作为系列的第一篇还是情有可原的。替换 tools 文件夹真的坑了我好久，最后还是在万能的 stackoverflow「[这个页面](http://stackoverflow.com/questions/42538433/not-finding-android-sdk-unity#)」 找到了解决方案。以后的版本也许就会修复这个问题不用这么麻烦了，但是目前为止因为没有 Assetbundle 要处理，也没有渠道 SDK 资源替换之类的操作，所以本质上就是一键打包 apk 的，但是 ipa 就不同了需要我们先导出 xcode 工程，搞定烦人的证书以后再导出 ipa 安装。很想自动化处理这一切，一起拭目以待吧。

------

原文链接：https://snatix.com/2017/03/12/012-export-unity-project-apk-on-mac/

本文由 [sNatic](https://github.com/sNaticY) 发布于『[大喵的新窝](https://snatix.com)』 转载请保留本申明