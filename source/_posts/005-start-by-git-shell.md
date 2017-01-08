---
layout: post
title: "探索Git(1)-初体验之git shell"
date: 2014-08-21 14:55
categories: git探索之路
tags: [git, windows, github]
---

既然开始用 github 托管 octopress ，那么就少不了接触 git 这个分布式版本管理神器。当初选择在 ubuntu 下搭 octopress 是因为不想把我心爱的 windows 装一些麻烦的东西(主要还是 windows 下搭起来太麻烦了)。但是平时编码都是需要在 windows 下完成的。博主用不了 vim 这些高端的东西只习惯 visual studio 这类一键式神器。嗯那么我们就从最简单的"GitHub for Windows"开始学习。

<!--more-->

## PART 1 概述

好吧好吧请原谅博主土鳖没有使用 msysgit 或者 cygwin 。而是使用了未登录之前的 [github主页](https://github.com) 提供的“[GitHub for windows](http://github-windows.s3.amazonaws.com/GitHubSetup.exe)"。那么接下来我们就一边使用 git 一边用 visual studio 创建项目然后开始学习吧。篇目有限我们接下来应该会尝试这些内容。

- 创建新项目
- 初始化`git init`
- 跟踪文件或加入暂存区`git add`
- 检查当前项目状态`git status`
- 将暂存区内容提交到git`git commit -m`
- 查看当前状态与上次commit之间的差别`git diff`
- 直接将已跟踪的文件当前状态提交`git commit -a -m`
- 查看提交记录`git log`
- 回滚整个项目到之前的某次commit`git reset --hard`(慎用)

## PART 2 开始

安装好『GitHub for Windows』就会发现桌面上有"Git shell"的快捷方式了。双击打开即可。然后我们开始尝试用 visual studio 创建一个 c# 控制台项目，然后使用git来管理他的版本

### 创建新项目

这个没什么可说的、博主新项目配置如下
![配置新项目](http://ojgpkbakj.bkt.clouddn.com/2014082101.png)
完成以后发现`f:/dev/HelloGit`里已经创建了一些文件了，不过只有一个`Program.cs`是我们的代码文件，且内容未被修改。我们打开 git  尝试如何管理。

``` powershell
F:\dev> cd .\HelloGit
```

### git初始化

初始化会在当前文件夹下建立一个“.git”文件夹。该文件夹里将会用于保存整个项目的信息。整个项目文件夹的文件添加删除修改等操作都会被 git 捕捉到。
进入到我们要管理的目录以后使用`git init`命令初始化我们的 git

``` powershell
F:\dev\HelloGit> git init
Initialized empty Git repository in F:/dev/HelloGit/.git/
```

### 跟踪文件

git 会捕捉项目文件夹下每个文件的变动。但并不会记录他们。需要使用`git add`命令来告诉git 哪个文件需要被跟踪。事实上只有`Program.cs`需要被跟踪。该文件位于`HelloGit/Program.cs`因此我们使用命令`git add`来跟踪这个文件

``` powershell
F:\dev\HelloGit [master +3 ~0 -0 !]> git add .\HelloGit\Program.cs
```

### 查看项目状态

`git status`命令会输出当前项目的状态，如『修改了但并没有被跟踪的文件』『已跟踪且修改了但并没有添加到暂存区的文件』『已经被添加到暂存区等待提交的文件』等等许多信息。现在我们来尝试一下。

``` powershell
F:\dev\HelloGit [master +1 ~0 -0 | +7 ~0 -0 !]> git status
# On branch master
#
# Initial commit
#
# Changes to be committed:
#   (use "git rm --cached <file>..." to unstage)
#
#       new file:   HelloGit/Program.cs
#
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#
#       HelloGit.sln
#       HelloGit.v12.suo
#       HelloGit/App.config
#       HelloGit/HelloGit.csproj
#       HelloGit/Properties/
#       HelloGit/bin/
#       HelloGit/obj/
```

我们发现上一条命令`git add`的作用是开始跟踪`Program.cs`并将其当前状态加入到暂存区。

### 将暂存区内容提交

我们发现"Changes to be committed"（暂存区）中出现了"new file"，正是我们需要跟踪的那个文件。以后每次执行`git commit`命令都会把整个『暂存区』提交为项目的一个版本。那些“Untracked files”暂时先不管了毕竟是visual studio自动生成的文件。

``` powershell
F:\dev\HelloGit [master +1 ~0 -0 | +7 ~0 -0 !]> git commit -m "initial project version"
[master (root-commit) 0727b5a] initial project version
 1 file changed, 15 insertions(+)
 create mode 100644 HelloGit/Program.cs
```

我们使用`git status`命令看看现在的状态

``` powershell
F:\dev\TouchGit\HelloGit [master +7 ~0 -0 !]> git status
# On branch master
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#
#       HelloGit.sln
#       HelloGit.v12.suo
#       HelloGit/App.config
#       HelloGit/HelloGit.csproj
#       HelloGit/Properties/
#       HelloGit/bin/
#       HelloGit/obj/
nothing added to commit but untracked files present (use "git add" to track)
```

这样就成功的提交了我们项目的第一个版本。当然只跟踪了`Program.cs`这个文件。

### 文件被修改后`git status`的变化

接下来我们修改这个文件试试看

``` csharp
static void Main(string[] args)
{
    Console.WriteLine("Hello Git!");
    Console.WriteLine("按任意键继续...");
    Console.Read();
}
```

运行一下，嗯没错我们的程序可以运行啦，赶快用`git status`看看发生了什么变化

``` powershell
F:\dev\HelloGit [master +7 ~0 -0 !]> git status
# On branch master
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git checkout -- <file>..." to discard changes in working directory)
#
#       modified:   HelloGit/Program.cs
#
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#
#       HelloGit.sln
#       HelloGit.v12.suo
#       HelloGit/App.config
#       HelloGit/HelloGit.csproj
#       HelloGit/Properties/
#       HelloGit/bin/
#       HelloGit/obj/
no changes added to commit (use "git add" and/or "git commit -a")
```

哦~它提示我们`Program.cs`这个文件被修改了，需要用`git add`命令来更新。但在此之前我们可能需要确认一下到底被修改了什么。
### 使用`git diff`进行比较
`git diff`命令会输出当前文件状态与上次提交之间的差别。

``` powershell
F:\dev\HelloGit [master +7 ~1 -0 !]> git diff
diff --git a/HelloGit/Program.cs b/HelloGit/Program.cs
index 6932e36..bb42dc2 100644
--- a/HelloGit/Program.cs
+++ b/HelloGit/Program.cs
@@ -10,6 +10,9 @@ class Program
     {
         static void Main(string[] args)
         {
+            Console.WriteLine("Hello Git!");
+            Console.WriteLine("按任意键继续...");
+            Console.Read();
         }
     }
 }
```

嗯，就是添加这三行代码。确实是我们想要的，但我们需要使用`git add`命令将这个文件加入到『暂存区』，感觉很麻烦的样子。

### 直接提交已跟踪的文件。

这次并没有新文件需要跟踪所以直接使用一个命令把所有已跟踪的文件(就只有Program.cs)添加到『暂存区』然后提交。省去`add`过程稍微方便一些

``` powershell
F:\dev\HelloGit [master +7 ~1 -0 !]> git commit -a -m  "HelloGit version 1.0"
[master 944b367] HelloGit version 1.0
 1 file changed, 3 insertions(+)
```

这样就已经提交成功了。

### 查看提交历史

`git log`命令会输出每次提交的历史。内容包括`commit ID`,『作者』，『提交时间』和每次提交时填写的『message』。那么我们来看看提交历史。

``` powershell
F:\dev\HelloGit [master +7 ~0 -0 !]> git log
commit 944b3671b5ad2456089e73f29d3ee3a184d274ba
Author: snatic <snatic0@126.com>
Date:   Tue Aug 19 20:11:24 2014 +0800

    HelloGit version 1.0

commit 0727b5a638b0403c5ab82ba4fa8f249869351b12
Author: snatic <snatic0@126.com>
Date:   Tue Aug 19 19:36:56 2014 +0800

    initial project version
```

### 回滚版本(慎用)

突然我发现其实整个"HelloGit"项目都有问题必须回滚到之前的某个版本。。怎么办呢？例如我们要回滚到"initial project version"。commit ID 是"0727b5a638b0403c5ab82ba4fa8f249869351b12"

``` powershell
F:\dev\HelloGit [master +7 ~0 -0 !]> git reset --hard 0727b5a638b0403c5ab82ba4fa8f249869351b12
HEAD is now at 0727b5a initial project version
```

回到vs提示文件有外部修改什么的。重新载入以后发现`Program.cs`真的回滚到以前的版本了。但是之前的『commit』都不见了，貌似不能再滚回去了。当然这个操作在 git 中并不提倡。正确的方法应该是进行分支。

## PART 3 总结

这次探索结束了，应该基本掌握 git 的正常用法了可以在写完代码以后一次一次的提交确保每次的信息都被完完整整的记录下来防止意外。发生意外以后可以顺利恢复到以前的版本。但直接回滚貌似并不是非常好的办法。git 的最精髓的部分『分支』并没有涉及到。关于分支什么的内容等待下次探索~

---
原文链接：http://snatix.com/2014/08/21/005-start-by-git-shell/

本文由 sNatic 发布于『[大喵的新窝](http://snatix.com)』 转载请保留本申明