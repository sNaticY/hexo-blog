---
layout: post
title: "Git探索之路(2)-.gitignore和git分支的使用"
date: 2014-08-23 19:41
categories: git探索之路
tags: [git, windows, github]
---

在上一篇博客『[Git探索之路(1)-git初体验之git Shell](http://http://www.snatic.tk/blog/2014/08/21/start-by-git-shell/)』中。我们探索了一些 git 的基本操作比如 git 初始化、追踪文件、加入暂存区、提交、比较、查看提交记录以及很暴力的回滚项目。那么 git 的精髓『分支』该如何使用呢？在此之前我们可能还注意到上次的工作目录太『不干净』了每次使用`git status`命令总会输出一大堆"untracked files"很影响心情。怎样才能彻底忽略掉那一大堆不需要跟踪的自动生成的文件呢？

<!--more-->

## PART 1 概述
在介绍分支之前我们首先要让我们的工作目录“干净”一些，最基本的方法就是创建并使用`.gitignore`文件将项目中不需要被显示的文件忽略掉。

- `.gitignore`的创建及使用方法

在进行版本管理时我们偶尔会需要在两个版本之间进行切换，但又不想暴力的把某个版本之后的所有工作全都抛弃，所以我们会用到“分支”这个概念。在分支这一块我们会进行以下几种情况的尝试

- 创建分支`git branch`
- 转换到某个分支`git checkout`
- 创建一个分支并转换到该分支`git checkout -b`
- 合并分支`git merge`
- 管理分支`git branch -v`
- 删除分支`git branch -d`
- 合并冲突分支

## PART 2 忽略某些文件

首先我们`git status`输出一下当前项目信息

``` powershell
F:\dev\HelloGit [master +7 ~0 -0 !]> git status
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

好多“Untracked files”。看起来极度不美观。我们创建一个名为`.gitignore`的文件。好像在 windows 中不能重命名为"."开头的文件名。我们直接使用 visual studio 新建一个文件再另存为就可以了。尝试在`.gitignore`文件中添加如下内容

``` text
#自定义需要忽略的文件
*.sln
*.suo
*.csproj
*.config
.gitignore
HelloGit/obj/
HelloGit/Properties/
HelloGit/bin/
```

再试试`git status`输出结果

``` powershell
F:\dev\HelloGit [master +1 ~0 -0 !]> git status
# On branch master
nothing to commit, working directory clean
```

嗯这样就成功了。该忽略的文件都不再显示了，这个世界清净的许多。`.gitignore`文件规则如下。(来源于《progit》)

>文件 .gitignore 的格式规范如下：

>- 所有空行或者以注释符号 ＃ 开头的行都会被 Git 忽略。
- 可以使用标准的 glob 模式匹配。
- 匹配模式最后跟反斜杠（/）说明要忽略的是目录。
- 要忽略指定模式以外的文件或目录，可以在模式前加上惊叹号（!）取反。

>所谓的 glob 模式是指 shell 所使用的简化了的正则表达式。星号（*）匹配零个或多个任 意字符；[abc] 匹配任何一个列在方括号中的字符（这个例子要么匹配一个 a，要么匹配一 个 b，要么匹配一个 c）；问号（?）只匹配一个任意字符；如果在方括号中使用短划线分 隔两个字符，表示所有在这两个字符范围内的都可以匹配（比如 [0-9] 表示匹配所有 0 到 9 的数字）。 我们再看一个 .gitignore 文件的例子：

>	```
	# 此为注释 – 将被 Git 忽略
	*.a # 忽略所有 .a 结尾的文件
	!lib.a # 但 lib.a 除外
	/TODO # 仅仅忽略项目根目录下的 TODO 文件，不包括 subdir/TODO
	build/ # 忽略 build/ 目录下的所有文件
	doc/*.txt # 会忽略 doc/notes.txt 但不包括 doc/server/arch.txt
>	```

不过貌似像博主这样后来才编辑`.gitignore`文件貌似不是很好。很容易误操作把不需要的文件加入到暂存区里，或者把不该忽略的目录全部忽略掉。于是博主发现一个很巧(tou)妙(lan)的方法就是在『GitHub for Windows』的GUI版本里创建一个新的 Repo 然后在`Git ignore`中选择“Visual studio”这样就可以使用预设的`.gitignore`文件自动忽略掉不该显示的文件类型和目录了。当然如果大家使用其他的语言的IDE什么的都可以自己选择需要的，非常方便。

![创建visual Studio默认.gitgnore](http://ojgpkbakj.bkt.clouddn.com/2014082301.png)

## PART 3 Git的分支系统

在 git 中新建分支、删除分支、合并分支等等操作的时间代价非常小，因此我们在开发的过程中可以尽可能的利用这个优势。

有时候会遇到这样的情景，我们的“HelloGit”项目主要功能已经趋于稳定并且投入运行。正在开发一个新的功能“输出Developer”、然而在新功能尚未完成的时候突然发现当前已经运行的版本有一个bug需要尽快修复。但是为了开发新功能原始的项目已经改的面目全非编译都不能通过了。难道必须等到新功能完成再回头去修复bug或者试图手动把改的乱七八糟的代码再改回来么。

在 git 的辅助下科学的做法是这样。首先我们的`Program.cs`文件如下。该版本已经基本稳定并投入运行。该版本位于默认的 master 分支。

``` csharp
static void Main(string[] args)
{
    Console.WriteLine("Hello Git!");
    Console.ReadKey();
}
```

### 创建新的分支
在我们开发新功能”输出developer“之前应该新建分支“develop”(名字随便取自己最后能认识就行)

``` powershell
F:\dev\HelloGit [master]> git branch develop
```

这样就成功建立了新的分支但该分支目前与 master 分支内容一致

### 切换分支

虽然建立了新的分支但是我们依然在 master 分支上工作。需要用`git checkout`命令来切换到新建的分支

``` powershell
F:\dev\HelloGit [master]> git checkout develop
Switched to branch 'develop'
F:\dev\HelloGit [develop]>
```

我们发现方括号里的 master 已经变成 develop。意味着我们现在开始进行的所有修改都是在develop分支进行的了。

接下来。在我们的辛勤劳动下，新功能的实现指日可待了。。(用这么大篇幅贴我写出来的渣代码这样真的好吗？)

``` csharp
class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine("Hello Git!");
        Console.ReadKey();
        Developer me = new Developer("sNaticY","www.snatic.tk");
        me.Output();
        Console.ReadKey();
    }
}

class Developer
{
    public string Name;
    public string Blog;
    public Developer(string n, string b)
    {
        Name = n;
        Blog = b;
    }
    public void Output()
    {
        Console.
        //这的代码还没写完，编译不通过
    }
}
```

就在这即将完成的关键时刻我们突然发现必须马上修复已经在运行的那个版本的一个错误。但是又不能等到新功能开发完成以后再修复。而且现在程序也不能运行了就算强行修复了也无法调试。怎么办，辛亏我们之前已经切换到了新的分支。果断提交以后再说。

``` powershell
F:\dev\HelloGit [develop +0 ~1 -0]> git commit -a -m "Unfinished updates"
[develop 3a04743] Unfinished updates
 1 file changed, 18 insertions(+)
```

### 创建新分支并切换到该分支

先切换到 master 分支。再在 master 分支的基础上新建一个分支用于修复 bug，使用`git checkout -b`

``` powershell
F:\dev\HelloGit [develop]> git checkout master
Switched to branch 'master'
F:\dev\HelloGit [master]> git checkout -b debug
Switched to a new branch 'debug'
F:\dev\HelloGit [debug]>
```

该命令相当于`git branch debug`+`git checkout debug`。然后我们回到 visual studio 发现文件已经恢复新功能开发之前的样子了。我们来修复这个“必须马上修改晚一秒会死人”的"bug"

``` csharp
static void Main(string[] args)
{
    Console.WriteLine("Hello Git!");
    Console.WriteLine("按任意键继续...");
    Console.ReadKey();
}
```

嗯，终于改好了人生从此圆满了。经过一系列复杂的测试以后你确信本次 debug 工作非常成功，所以我们接下来要将它合并到 master 分支以便发布并投入运行。

### 合并分支

首先切换到 master 分支，然后使用`git merge`命令将 debug 分支合并进来。

``` powershell
F:\dev\HelloGit [debug]> git checkout master
Switched to branch 'master'
```

``` powershell
F:\dev\HelloGit [master]> git merge debug
Updating 152e89c..f15ab97
Fast-forward
 HelloGit/Program.cs | 1 +
 1 file changed, 1 insertion(+)
```

>请注意，合并时出现了 “Fast forward”（快进）提示。由于当前 master 分支所在的 commit 是要并入的 hotfix 分支的直接上游，Git 只需把指针直接右移。换句话说，如果 顺着一个分支走下去可以到达另一个分支，那么 Git 在合并两者时，只会简单地把指针前 移，因为没有什么分歧需要解决，所以这个过程叫做快进（Fast forward）。 --《progit》

合并以后我们位于 master 分支。回到 visual studio 发现新的代码已经被添加我们的debug顺利完成。

### 分支管理

现在我们貌似分支分了几个而且合并几个操作感觉很混乱。来尝试查看一下当前项目分支信息吧`git brance -v`

``` powershell
F:\dev\HelloGit [master]> git branch -v
  debug   f15ab97 debug over
  develop 3a04743 Unfinished updates
* master  f15ab97 debug over
```

我们发现已经合并了的 debug 分支依然存在。而且与 master 分支指向了同一次 commit 。好像也没什么用处了

### 删除分支

既然 debug 分支没什么用了就删掉好了。等到下次有 Bug 再新建也不迟。不然项目显得乱糟糟的。

``` powershell
F:\dev\HelloGit [master]> git branch -d debug
Deleted branch debug (was f15ab97).
```

### 继续开发新功能

删除分支以后心情非常愉快，继续刚才未完成的工作吧。回到 develop 分支继续工作。

``` csharp
public void Output()
{
    Console.WriteLine("Developer: {0}",Name);
    Console.WriteLine("Blog: http://{0}/", Blog);
}
```

经过一系列测试发现新功能实现了，我们把提交然后合并到 master 分支吧。

``` powershell
F:\dev\HelloGit [develop]> git commit -a -m "update finished"
[develop 047c04a] update finished
F:\dev\HelloGit [develop]> git checkout master
Switched to branch 'master'
```

``` powershell
F:\dev\HelloGit [master]> git merge develop
Auto-merging HelloGit/Program.cs
Merge made by the 'recursive' strategy.
 HelloGit/Program.cs | 19 +++++++++++++++++++
 1 file changed, 19 insertions(+)
```

嗯嗯不错，并没有出现合并冲突的情况。而是成功的把以旧版 master 为原型的 develop 分支和已经进化了的 master 分支合并在一起。如下

``` csharp
static void Main(string[] args)
{
    Console.WriteLine("Hello Git!");
    Console.WriteLine("按任意键继续...");
    Console.ReadKey();
    Developer me = new Developer("sNaticY","www.snatic.tk");
    me.Output();
    Console.ReadKey();
}
```

### 合并冲突

但是有时候合并操作并不会这么顺利。如果修改了两个分支里同一部分，那么就会出现合并冲突的情况。比如我们在 master 分支中的`Hello Git!`后多加一个"!"，而在 develop 分支中去掉"!"。那么合并时就会出现如下状况。(前面一大堆修改提交省略)

``` powershell
F:\dev\HelloGit [master]> git merge develop
Auto-merging HelloGit/Program.cs
CONFLICT (content): Merge conflict in HelloGit/Program.cs
Automatic merge failed; fix conflicts and then commit the result.
```

提示我们`Program.cs`文件合并失败了。打开这个文件看看哪里失败了。

``` csharp
<<<<<<< HEAD
            Console.WriteLine("Hello Git!!");
            Console.WriteLine("按任意键继续...");
=======
            Console.WriteLine("Hello Git");
>>>>>>> develop
```

git 为我们在冲突的地方使用一些符号标记。”=======“上半部分是HEAD(即master分支)中的内容，而下方是 develop 的内容，我们只需二选一或者用自己的方式将他们整合。像这样

``` csharp
Console.WriteLine("Hello Git");
Console.WriteLine("按任意键输出developer...");
```

总之删掉特殊的标记然后修改代码使其可以按照你的意愿运行，随后`commit`即可。最后贴一张我们的项目完全版运行结果。

![HelloGit完整版运行结果](http://ojgpkbakj.bkt.clouddn.com/2014082302.png)

## PART 4 总结

终于，关于git shell的本地使用的大部分内容就这样学习了。结合上一篇博客『[Git探索之路(1)-git初体验之git Shell](http://http://www.snatic.tk/blog/2014/08/21/start-by-git-shell/)』中讲到的内容应该可以成功的使用『git shell』进行本地的版本管理了。接下来我们可能会继续探索 Github 与 Git shell 结合使用。以及GUI版本的『GitHub for Windows』的使用方法。敬请期待。

---

原文链接：http://snatix.com/2014/08/23/006-gitignore-and-git-branch/

本文由 sNatic 发布于『[大喵的新窝](http://snatix.com)』 转载请保留本申明