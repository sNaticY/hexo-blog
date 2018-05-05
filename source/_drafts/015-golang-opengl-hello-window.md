---
title: 学习Opengl(1)-Hello Window
date: 2018-01-02 15:51:00
tags: [golang, mac, opengl, glfw]
categories: 随便玩玩Opengl

---

2017随随便便的就过去了，无数没填完且有可能一辈子都填不上的坑也无法阻止我新年开新坑。要玩就玩大的我们今年一起来玩 opengl 吧希望年底的时候可以稍微懂一点皮毛就好。这次打算跟着 https://learnopengl.com 的教程一步一步来，并不是想逐字翻译，就当作是课后笔记吧。但是由于博主不喜欢 c++ 所以我们将选择方便快捷的 golang 来尝试完成原教程中的所有课程。

<!--more-->

## PART 1 概述

通常来说，教程开始之前都会有一篇介绍 opengl 到底是什么的 [文章](https://learnopengl.com/#!Getting-started/OpenGL)，但是我们这里以实用为主就不翻译那些东西了直接跳过，有需要的小伙伴可以打开看看。首先按照 [原始教程](https://learnopengl.com/#!Getting-started/Hello-Window) 中的要求，我们要在本篇结束的时候使用 golang 创建一个窗口并在里面涂上一个背景色，并且可以接收esc键指令关闭窗口。似乎是很简单的任务呢，我们要做的大概是：

1. 配置环境 golang 以及在 golang 中使用 gl 和 glfw 库
2. 开启一个窗口
3. 设置视口 Viewport
4. 接收esc指令关闭窗口
5. 设置背景色

## PART 2 环境配置

go相关的配置比c++简单太多了，就不展开讲了。遇到问题大家可以在评论区提问。按顺序 [**Golang 安装官方教程**](https://golang.org/doc/install)、 [**go-gl/gl**](https://github.com/go-gl/gll)、[**go-gl/glfw**](https://github.com/go-gl/glfw) 将这些事情都做好以后新建文件夹如以下结构

``` tex
任意路径下
|--任意文件夹名称如 mygopath
	|--bin
	|--pkg
	|--src
		|--learnopengl-go
			|--001-HelloWindow
				|--main.go
```

将 mygopath 加入 gopath 后新建好文件`main.go`，就算是配置完毕了，如果不确定可以跑个 Helloworld 试试。

## PART 3 开启窗口

首先我们需要创建一个 main 函数，我们将在这个函数中初始化 GLFW 窗口。需要注意的是不同于 c++，我们需要额外调用 `runtime.LockOsThread()`来确保`main()`执行在主线程中，因此我们的 main.go 目前会是这样：

``` go
package main

import (
	"runtime"
	"github.com/go-gl/glfw/v3.2/glfw"
)

func init() {
	runtime.LockOSThread()
}

func main() {
    err := glfw.Init()
    if err != nil {
      panic(err)
    }
    defer glfw.Terminate()
    if err := gl.Init(); err != nil {
      log.Fatalln(err)
    }
}

```

`init()`函数会在 package 被导入时自动执行用于初始化，因此我们无需手动调用。在`main()`中，我们初始化 glfw 对象并 defer 调用 `glfw.Terminate()`确保函数执行结束后可以正确释放资源，最后初始化 gl 对象以便后续使用。

接下来我们尝试使用 glfw 创建一个 window 对象，该对象持有了所有的窗口相关数据因此该对象接下来会被很频繁的调用到。

```go
window, err := glfw.CreateWindow(800, 600, "LearnOpenGL", nil, nil)
if err != nil {
  panic(err)
}
window.MakeContextCurrent()
```

`glfw.CreateWindow()`的前两个参数是窗口的宽和高，第三个参数是窗口标题。我们暂时使用"LearnOpenGL"当然大家也可以随便换自己喜欢的名字。该函数返回一个 `glfw.window` 对象，我们接下来会在其他 glfw 操作中用到该对象。在创建完窗口后，我们将创建好的窗口的上下文设置为当前线程的 glfw 主上下文。

## PART 4 ViewPort 视口

在开始渲染之前必须先告诉 OpenGL 渲染窗口的大小，从而让 OpenGL 可以正确的根据窗口的大小来显示数据。我们可以通过调用`gl.Viewport()`设置窗口大小。





------

原文链接：http://snatix.com/2018/01/02/015-golang-opengl-hello-window/

本文由 sNatic 发布于『[大喵的新窝](http://snatix.com)』 转载请保留本申明