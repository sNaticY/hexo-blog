---
title: 重启Hexo(3)-添加打赏功能
date: 2017-02-13 12:18:53
tags: [hexo, mac, blog]
categories: Hexo博客重启计划
donate: true

---

因为在『[关于我](https://snatix.com/about/)』页面中承诺要制作打赏功能拖延一个月还没开始动手，被某小盆友吐槽以后决定奋发图强的瞬间做好了。但是当时制作过程太紧迫就没有一步一步的记录下来，现在就稍微补一下吧～其实过程太简单所以这篇文章应该内容很少，但的确很好用～

<!--more-->

## PART 1 概述

距离系列的『[上一篇](http://snatix.com/2017/01/14/008-customize-hexo/)』过去了一个多月终于迎来了更新，那就是「打赏」功能！其实这样的需求不算少，大家随便用「hexo」「打赏」这样的关键词百度一下就会发现很多很多转载来转载去的文章，翻来覆去讲的大概就是那些内容，很多都是为某个特殊主题服务的，或者是主题框架与自己使用的主题完全不同，就会导致很多麻烦的问题。可能很多同学没有打算深入研究某个主题，因此也对主题框架一知半解，花时间解决那些问题还挺麻烦的，就想简简单单的复制一段代码就实现的打赏功能难道不存在么？当然存在

1. 在主题文章内容的 ejs 文件里插入关键代码
2. 在博客`_config.yml`中定义二维码链接
3. 在需要添加打赏功能的文章顶部添加标识

## PART 2 在关键位置插入关键代码

既然说关键代码当然很关键了，最关键的是代码是整体的。不存在一部分放在某个地方，另一部分放在其他地方然后相互调用的问题，完美方便懒得了解自己正在用的主题的框架的同学。当然，这段代码具体插在哪里还是有讲究的。总之大家先查看以下代码。

```html
<% if (page.donate) { %>
<!-- css -->
<style type="text/css">
  .center {
    text-align: center;
  }
  .hidden {
    display: none;
  }
  .donate_bar a.btn_donate{
    display: inline-block;
    width: 82px;
    height: 82px;
    background: url("http://ojgpkbakj.bkt.clouddn.com/DonateButton.gif") no-repeat;
    _background: url("http://ojgpkbakj.bkt.clouddn.com/DonateButton.gif") no-repeat;

    -webkit-transition: background 0s;
    -moz-transition: background 0s;
    -o-transition: background 0s;
    -ms-transition: background 0s;
    transition: background 0s;
  }

  .donate_bar a.btn_donate:hover{ background-position: 0px -82px;}
  .donate_bar .donate_txt {
    display: block;
    color: #9d9d9d;
    font: 14px/2 "Microsoft Yahei";
  }
  .bold{ font-weight: bold; }
</style>

<!-- /css -->
<!-- Donate Module -->
<div id="donate_module">

  <!-- btn_donate & tips -->
  <div id="donate_board" class="donate_bar center">
    <a id="btn_donate" class="btn_donate" target="_self" href="javascript:;" title="Donate 打赏"></a>
    <span class="donate_txt">
      <%= theme.donate.text %>
    </span>
  </div>
  <!-- /btn_donate & tips -->

  <!-- donate guide -->
  <div id="donate_guide" class="donate_bar center hidden">

    <a href="<%= theme.donate.wechat %>" title="用微信扫一扫哦~" class="fancybox" rel="article0">
      <img src="<%= theme.donate.wechat %>" title="微信打赏 snatic" height="190px" width="auto"/>
    </a>



    <a href="<%= theme.donate.alipay %>" title="用支付宝扫一扫即可~" class="fancybox" rel="article0">
      <img src="<%= theme.donate.alipay %>" title="支付宝打赏 snatic" height="190px" width="auto"/>
    </a>

    <span class="donate_txt">
      <%= theme.donate.text %>
    </span>

  </div>
  <!-- /donate guide -->

  <!-- donate script -->
  <script type="text/javascript">
    document.getElementById('btn_donate').onclick = function() {
      $('#donate_board').addClass('hidden');
      $('#donate_guide').removeClass('hidden');
    }

    function donate_on_web(){
      $('#donate').submit();
    }

    var original_window_onload = window.onload;
    window.onload = function () {
      if (original_window_onload) {
        original_window_onload();
      }
      document.getElementById('donate_board_wdg').className = 'hidden';
    }
  </script>
  <!-- /donate script -->

</div>
<!-- /Donate Module -->
```

还挺长的，但是排版缩进博主都整理好了大家勇敢的复制粘贴吧。那么复制粘贴到哪里呢，这的确是一个问题。通常是位于`themes/xxx/layout/_partial/`之类的目录下面，找到类似`post-content.ejs`或者看着名字像是文章正文的文件中合适的位置，一般是`<%- page.content %>`这一行的后面。。。

好吧我承认我表达的有点隐晦，不过大家都是聪明的程序猿或者攻城狮，随便试试就试出来了，当然了解结构的同学可以自行根据需要添加在文章开头也是没什么问题的～

## PART 3 填写全局配置

这一步就很简单了打开位于根目录下的`_config.yml`文件，在最后添加如下代码并且把内容改成你想要的配置就好了。

``` yaml
# Donate
donate:
  text: 一定会有好心人支持我的，一块两块的就好～
  wechat: http://ojgpkbakj.bkt.clouddn.com/WechatImage.png
  alipay: http://ojgpkbakj.bkt.clouddn.com/AlipayImage.png
```

`text`就是显示在「赏」字下面的那一行小字，然后`wechat`和`alipay`分别对应微信和支付宝的收款二维码（不要贴成付款二维码了），最好截图以后稍微放到 Photoshop 里面处理一下，毕竟美观一些大家付款的欲望也会更强的……吧。

## PART 4 为想要的文章开启打赏功能 

最后也是最关键的一步，很多人不管什么文章都喜欢在后面放一个大大的「赏」字好像这个字出现次数越多就会收到打赏的机率越高一样。。。然而博主并不这么认为，因为隐隐约约觉得好像在乞讨一样～所以可以选择性的为某篇文章开启打赏功能当然是必备的。只需要在文章顶部的`front-matter`添加`donate`即可，像是这样。

```yaml
title: 重启Hexo(3)-添加打赏功能
date: 2017-02-13 12:18:53
tags: [hexo, mac, blog]
categories: Hexo博客重启计划
donate: true
```

作为有节操的博主本来是只在「关于我」这样的页面放一个打赏的入口，心里默默计划着万一哪天真的写了一篇特别厉害的文章说不定也会加入（比如这篇？）好吧其实是放一个按钮给大家点一点让大家能够直观的看到效果。

## PART 5 总结

目前为止打赏功能就算是完成了，看起来挺简单的因为文章里省略了最耗时的过程——用 PS 处理二维码，为了美观也是竭尽全力～好吧这次本系列下次更新可能要好久好久了，因为主题也慢慢的稳定下来～以前喜欢折腾主题不喜欢写文章的习惯已经在慢慢的改变了，希望以后能再勤劳一些写更多的文章～打赏功能不出意外的话应该会显示在文章底部喽～

------

原文链接：https://snatix.com/2017/02/13/011-hexo-blog-donate-plugin/

本文由 [sNatic](https://github.com/sNaticY) 发布于『[大喵的新窝](https://snatix.com)』 转载请保留本申明