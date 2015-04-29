title: "【翻译】开发者工具时间轴：现在提供了全部的故事"
date: 2015-04-29 10:23:09
categories: 翻译
tags:
s: devtools-timeline-improvements
---

原文地址：[http://updates.html5rocks.com/2015/04/devtools-timeline-improvements](http://updates.html5rocks.com/2015/04/devtools-timeline-improvements)

开发者工具的[timeline panel](https://developer.chrome.com/devtools/docs/timeline)一直以来都是在性能优化路上最佳的第一站。 这个关于你的应用程序活动的集中化概览，可以帮助你分析花费在loading、scripting、rendering和repainting上的时间。 最近，我们升级了timeline面板，增加了更多的使用方法，以便你可以更加深入地了解你的应用程序性能。

我们添加了以下特性：

* 集成[Javascript profiler](#Integrated_JavaScript_Profiler)（包括了火焰图表）
* [frame viewer](#Frame_Viewer)，帮助你查看组合图层。
* [paint profiler](#paint-profiler)，入木三分地剖析了浏览器渲染活动。

注意，使用本文中描述的__Paint__捕获选项会导致一些性能负担，所以只有你需要它们的时候，才去勾选它们。

<!-- more -->

## Integrated JavaScript Profiler

如果你曾经玩过__Profiles__面板，那么你应该很熟悉[Javascript CPU Profiler](https://developer.chrome.com/devtools/docs/cpu-profiling)。这个工具估量了执行时间都花费在你的Javascript函数哪些地方。通过火焰图表来查看Javascript profiles，你可以可视化你的Javascript随着时间推移的执行过程。

现在，你可以在__Timeline__面板上获得细粒度的Javascript执行故障。通过勾选__JS Profiler__捕获选项，你可以在时间轴上看到你的Javascript调用栈，同时还有一些其他浏览器事件。 添加这些特性到时间轴上，可以帮助你流程化你的调试工作。但是不仅如此，它还允许你查看上下文中的Javascript代码，并且还可以标识出影响页面加载时间和渲染的那部分代码。

除了该Javascript profiler，我们还集成了一个火焰图表到__Timeline__面板中。你现在可以通过两种方式查看你的应用程序活动，一种是经典的瀑布图，一种是一个火焰图表。 面板上的火焰图标<span class="article-inline-img">{% asset_img flame-icon.png %}</span>允许你在这两种视图之间切换。

{% asset_img javascript-profiler.png %}

<p class="article-center-p">在时间轴上，使用JS Profiler捕获选项和火焰图表视图来研究调用栈。</p>

提示：可以使用按键`WASD`来缩放和平移火焰图表。`Shift + 拖动`可以画一个选择框。

## Frame Viewer

[layer compositing](http://www.html5rocks.com/en/tutorials/speed/layers/)的艺术是浏览器不为人知的另一方面。当节俭并小心地使用时，图层可以帮助避免一些昂贵的重绘操作，并且产生巨大的性能收益。但是往往无法明显地预测浏览器将如何组合你的内容。 使用Timeline面板中新的__Paint__捕获选项，你可以显式地查看一个记录中每一帧的组合图层。

当你选择在__Main Thread__上方的某一个灰色的帧条时，它的图层面板提供了一个图层的可视化模型，这些图层组成了你的应用程序。

你可以缩放、旋转、拖动这些图层来查看它们的内容。光标移到一个图层上，会揭示出它在页面上的当前位置。右击一个图层可以让你跳到__Elements__面板中相应的节点上。这些功能向你展示了什么被提升为一个图层。如果你选择一个图层，你也可以在签名为__Compositing Reasons__的行中看到它为什么被提升了。

{% asset_img compositing-reasons.png %}

<p class="article-center-p">从[Codrops' Scattered Polaroids Gallery](http://tympanus.net/Development/ScatteredPolaroidsGallery/)里检查一个图层来揭晓浏览器组合图层的原因。</p>

_ps: 一个元素可以被提升到一个图层中，当它具备某些特性时，详见[compositing layers](http://www.html5rocks.com/en/tutorials/speed/layers/)。_

## Paint Profiler

最后但并非不重要，我们已经添加了paint profiler来帮助你找出由于昂贵的绘制引发的卡顿。这个特性充实了时间轴，我们可以看到更多关于Chrome在绘制事件期间的工作信息。

对于初学者，现在可以更容易地辨别可见内容与绘制事件之间的对应关系。当你在时间轴上选择一个绿色的绘制事件（__Main
Thread__下方），__Details__面板将为你展示已经绘制的实际像素的预览。

{% asset_img paint-capture.png %}

<p class="article-center-p">使用__paiint__捕获选项预览浏览器已经绘制的像素。</p>

如果你真想深究，切换到__Paint Profiler__面板上。这个profiler为你展示了浏览器在选中的绘制事件中所执行的准确的绘制命令。 为了帮助你关联这些本地命令到你应用程序中的实际内容，你可以右击__draw*__调用并直接跳转到__Elements__面板上的对应的节点。

{% asset_img draw-calls.png %}

那个横跨在面板顶部上的小时间轴可以让你重播绘制进程，然后让你对哪些浏览器绘制操作是昂贵的有所了解。绘图操作是颜色编码的，如下：<span
style="color: pink;">pink</span>(shapes)、<span style="color: blue;">blue</span>(bitmap)、<span style="color:
green;">green</span>(text)、<span style="color:
purple">purple</span>(misc.)。条形高表示调用过程，所以研究高的条形可以帮助你了解一个特定的绘制是怎么昂贵的。

## Profile and profit!

当涉及到性能优化时，浏览器的知识是难以置信地有用。管中窥豹，这些时间轴更新足以帮助阐明你的代码和Chrome渲染进程之间的关系。试试时间轴上的这些更新吧，然后看看Chrome DevTools能做些什么来增强你的解决卡顿问题的工作流程。


_ps: 原文有几个视频，推荐大家去看看。_
