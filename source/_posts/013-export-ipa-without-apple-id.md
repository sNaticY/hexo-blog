---
title: Unity自动化(2)-无需企业开发帐号密码打包ipa
date: 2017-03-18 13:24:08
tags: [unity, mac, ipa, ios]
categories: Unity通用框架工程

---

终于迎来一个不用加班的周六可以愉快的写文章了。换了工位换了两台一样的显示器心情无比舒爽觉得终于要开展全新生活啦～之前两台显示器型号高矮全都不一样让我这样的强迫症患者超级难受 - -。言归正传，接下来我们来到了导出 ipa 的环节。通常在稍微规模大一点的公司是不会给开发者提供他们的具有开发者账户权限的 Apple Id 和密码的，因此想要导出 ipa 就只能通过公司提供的 p12 证书和 mobileprovision 描述文件，那么具体要怎么操作呢？

<!--more-->

## PART 1 概述

在「[上一期](https://snatix.com/2017/03/12/012-export-unity-project-apk-on-mac/)」中我们介绍了在 mac 上导出 apk 的方法，那么现在来到了导出 ipa 的环节。正如引言所述，通常我们是没有企业开发帐号和密码的，只有 mobileprovision 和 p12 文件。目前 Xcode 版本已经达到 8.2 但是很多教程还停留在 7.x 甚至 6 的时代，有很多地方已经不适用了。所以接下来我们根据不同的 Xcode 版本来尝试打包一个带有企业证书的 inhouse 包并发布到服务器上可以允许任意用户远程安装。具体步骤如下：

1. 准备好一个 Unity 工程并将其导出 Xcode 工程
2. 导入 p12 证书和 mobileprovision 描述文件
3. 生成对应的 app 文件
4. 将 app 文件转换成 ipa 文件
5. 制作 plist 文件并发布到服务器

## PART 2 将Unity工程导出xcode工程

为了给大家演示又方便打包博主就简单的制作一个 Unity 工程吧，如图所示。。

![示例工程](http://ojgpkbakj.bkt.clouddn.com/2017032001.png)

相信大家一眼就看出来了就只有一行代码把`Time.time`显示在屏幕中间。。好吧我们现在开始导出 xcode 工程。Build Settings 面板如下，点击`Build`再选一个路径就可以导出了，要注意的是`Run in Xcode as`必须要选择`Debug`才行，还有就是别忘了修改`Bundle Identifier`与证书一致，如果不知道的话可以留到 Xcode 工程中再修改。

![BuildSettings](http://ojgpkbakj.bkt.clouddn.com/2017032002.png)

最后打开导出的目录双击`Unity-iPhone.xcodeproj`打开 Xcode 工程，成功！

![Xcode工程](http://ojgpkbakj.bkt.clouddn.com/2017032004.png)

## PART 2 导入.p12证书和.mobileprovision

接下来的步骤可能不同版本的 Xcode 会不太一样，以博主的 Xcode 8.2.1 为例。打开以后如上图所示。可以看到有一个报错叫做`"Signing for "Unity-iPhone" requires a development team"`，一般来说如果自己拥有开发者帐号的话直接按照各种教程的步骤来就没问题了。但是很多时候我们并没有对应的企业开发者帐号的使用权限，只有如下图的两个文件。

![证书](http://ojgpkbakj.bkt.clouddn.com/2017032005.png)

关于这两个文件的作用和来源这里就不再赘述了，总之很麻烦而且博主也没有企业级开发者帐号没有亲手操作过～通常公司的平台组会为我们准备好这两个文件。首先回到 Xcode 工程中，切换到`Signing`部分，取消`Automatically manage signing`。此时会出现多个要求导入`Provisioning Profile`的部分，如下图所示。

![证书](http://ojgpkbakj.bkt.clouddn.com/2017032006.png)

接下来的操作很简单，只需要在每一个地方点击`Import Profile...`并选择之前的`example.mobileprovision`文件即可。如果之前没有修改`Bundle Identifier`的话，会有报错信息提示：`Provisioning profile "xxxxxx" has app ID "com.xxx.xxx", which does not match the bundle ID "com.snatic.example".` 按要求修改`Bundle Identifier`即可。

那么修复了这条报错以后，还会提示`No "iOS Distribution" signing certificate matching team ID "xxxxxxxxx" with a private key was found.`那么此时我们需要导入的就是刚才的 .p12 文件了，双击以后输入密码即可导入很方便。那么目前为止所有的报错全都解决了可以安全导出 ipa 了么？

## PART 3 生成app文件

理论上此时我们就可以直接插上一台 iPhone 开始运行了，但是我们的目标是导出 ipa，因此按照正常流程往下走的话就是点击 Product -> Archive 然后 Export 就好了，这个流程在 Xcode 7 中完全没有问题。但在 Xcode 8 中，会做如下提示。

![添加帐号](http://ojgpkbakj.bkt.clouddn.com/2017032007.png)

在我们没有开发者帐号以及企业子帐号的情况下这一步没办法绕过了，在 Xcode 7 中可以直接选择`Use local signing assets`的选项但是这里并不会直接出现还是需要先登录。那么我们只能剑走偏锋了。

插上一台 iPhone 直接点击运行，等待编译和安装完成以后确认在设备上运行无误，然后我们就会在左侧看到 `XXXX.app`，如图所示。

![导出app](http://ojgpkbakj.bkt.clouddn.com/2017032008.png)

右键点击该文件，选择`Show in Finder`即可找到我们至关重要的 app 文件。

## PART 4 将app转换为ipa

转换的方法有几种，限于篇幅(其实是太懒)博主就先介绍三种简单的方法，关于高级的自动化脚本的方法下次单独开一篇文章来介绍吧～

### 方法一 拖入iTunes

打开 iTunes 切换到应用页，然后直接将上一步得到的 app 文件拖进来，就能得到如下图的应用。

![导出app](http://ojgpkbakj.bkt.clouddn.com/2017032201.png)

右键点击应用 -> 在 Finder 中显示 就可以得到我们所需要的 ipa 文件了。

### 方法二 手动压缩改后缀

1. 新建“Payload"文件夹，注意大小写
2. 将 app 文件放到`Payload`文件夹中
3. 在`Payload`文件夹右键压缩成 zip 后将生成的 zip 文件后缀为 ipa

感觉简单粗暴，这个过程明显可以写一个脚本来完成的。下次再慢慢讲脚本的问题～理想状态是可以不用打开 xcode 不用插手机就能直接导出，这样就可以一键导出 ipa 包了。

## PART 5 Inhouse发布 

其实发布 Inhouse 下载非常简单，如果按照正常打包流程的话 Xcode 是会为我们自动生成所需要的 plist 文件的，但是我们的流程比较奇怪所以需要新建一个文件，粘贴以下内容最后修改后缀为`plist`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>items</key>
	<array>
		<dict>
			<key>assets</key>
			<array>
				<dict>
					<key>kind</key>
					<string>software-package</string>
					<key>url</key>
					<string>THE URL FOR YOUR IPA: ex: https://go.com/appname.ipa</string>
				</dict>
				<dict>
					<key>kind</key>
					<string>full-size-image</string>
					<key>needs-shine</key>
					<true/>
					<key>url</key>
					<string>THE URL FOR INSTALLATION @2x ICON: ex: https://go.com/Icon@2x.png</string>
				</dict>
				<dict>
					<key>kind</key>
					<string>display-image</string>
					<key>needs-shine</key>
					<true/>
					<key>url</key>
					<string>THE URL FOR INSTALLATION ICON: ex: https://go.com/Icon.png</string>
				</dict>
			</array>
			<key>metadata</key>
			<dict>
				<key>bundle-identifier</key>
				<string>YOUR BUNDLE ID (Take it from your Xcode Project)</string>
				<key>bundle-version</key>
				<string>1.2.3 Your app version</string>
				<key>kind</key>
				<string>software</string>
				<key>title</key>
				<string>The Title To Present To The User installing the app</string>
			</dict>
		</dict>
	</array>
</dict>
</plist>

```

按照说明修改其中的字段以后将该文件与 ipa 一起上传支持 https 协议的服务器，没有的话就在 oschina 或者 coding,net 之类的新建一个 Git 仓库然后上传就没问题了，记得图标也要上传不然会有奇怪的无法下载的问题，图标大小不大于 500*500 就好。最后我们在任意一个网页上插入如下代码，在`url=`后面填写刚才上传的 plist 文件地址就好了。

```html
<a href="itms-services://?action=download-manifest&url=https://mydomain.com/apps/MyInHouseApp.plist" id="text">Install the In-House App</a>  
```

用手机打开这个页面试试。点击链接就可以下载了果然很方便的样子。

## PART 6 总结

拖了小半周零零碎碎的总算是写完了，貌似没什么干货反而让截图占了很大的篇幅。但是为了把整个流程讲清楚多截几张图也还好。也许新版本的  Xcode 发布以后这篇教程就过期了～当前的 Xcode 版本是 8.2.1 不知道苹果会不会在以后的版本中采取更严格的方式限制打包。不过目前手动打包 ipa 的方法并不是我们的重点，下周我们一起研究自动打包脚本～

------

原文链接：https://snatix.com/2017/03/18/013-export-ipa-without-apple-id/

本文由 [sNatic](https://github.com/sNaticY) 发布于『[大喵的新窝](https://snatix.com)』 转载请保留本申明

