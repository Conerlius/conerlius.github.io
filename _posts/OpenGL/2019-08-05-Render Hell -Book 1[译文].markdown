---
layout:     post
title:      "Render Hell -Book 1[译文]"
subtitle:   " \"Graphic\""
date:       2019-08-5 11:25:00
author:     "Conerlius"
category: opengl
keywords: opengl
tags:
    - 译文
---
* content
{:toc}

本文翻译自：[link](https://simonschreibt.de/gat/renderhell-book1/)

*如果视频无法播放，[Message me](mailto:971302351@qq.com)*

*美术必须注意的是:从计算级的角度来看，你的所有资源都是一系列的顶点和纹理数据。将原始数据转换成下一代的图像主要由你的系统处理器（CPU）和图形处理器（GPU）完成。*
## 1. 将数据复制到内存中，以方便系统快速地访问
> 首先，所有必要的数据都从硬盘驱动器（HDD）加载到系统内存（RAM）中，以便更快地访问。然后把必要的网格和纹理都加载到显存（VRAM）中。这是因为显卡访问VRAM比访问内存(RAM)更快。

<html>
<video width="100%" controls="" loop="" preload="" src="/vedios/copy_data_from_hdd_to_ram_vram_01.mp4" type="video/mp4"></video>
</html>

当VRAM已经加载完成以后，如果纹理资源不需要重复使用，我们就可以吧该纹理资源从RAM里移除。（注意：你必须确定你不在需要该资源，因为重新从HDD加载到RAM需要消耗大量的时间）。一般情况下，顶点数据都会常驻在RAM中，主要是CPU基本都有可能需要访问访问，比如碰撞检测。

<html>
<video width="100%" controls="" loop="" preload=""  src="/vedios/delete_data_in_ram.mp4" type="video/mp4"></video>
</html>
<html>

完成上一步的时候，数据已经缓存在显存中了(VRAM)。但是从VRAM到GPU的传输速度还是显得慢，GPU在传输数据上还可以更快的！<br>

因此硬件驱动允许把较小的内存块数据直接传递到处理芯片(the processor chips)，一般称之为“芯片缓存”（on-chip caches）。因为如果要将该缓存植入到处理芯片上需要极为昂贵的成本，所以导致了该缓存的大小极为有限。GPU从缓存中复制部分当前必须的数据.

<html>
<video width="100%" controls="" loop="" preload="" src="/vedios/copy_data_from_vram_to_l2_01.mp4" type="video/mp4"></video>
</html>

数据复制完成后，这些已经复制的数据会存放在一个叫L2的缓存区。L2一般都比较小（在NVIDIA GM204: 2048 KB），而L2的位于GPU上，并且GPU访问L2的速度比访问VRAM快了很多。<br>


尽管如此，对于需要更高效的GPU来说，还是很慢!因此出现了一个更小的缓存L1（在NVIDIA GM204: 384 KB (4 x 4 x 24 KB)），它不仅位于GPU上，而且更接近核心！

<html>
<video width="100%" controls="" loop="" preload="" src="/vedios/copy_data_from_l2_to_l1_01.mp4" type="video/mp4"></video>
</html>

另外，还有另一块内存用来存放GPU核的输入和输出数据的：寄存器和寄存器文件。在此GPU核心将截取部分数据（比如2个值）去计算并且把计算的结果放置到寄存器里（基本上都是一些寄存器上的内存地址指针）；

这些寄存器上的结果都会传送和存储到L1/L2/VRAM里，以便为新的计算在寄存器文件中腾出空间。作为开发者一般都不需要太过于担心这个过程。
<html>
<video width="100%" controls="" loop="" preload="" src="/vedios/copy_data_from_l1_to_register_01.mp4" type="video/mp4"></video>
</html>

为什么需要如此麻烦，如上所述，这些都和访问消耗的时间有关！如果你去比较这些访问时间，比如：HDD和L1,你会发现差距超级大！具体数据可以参考
[link](https://gist.github.com/hellerbarde/2843375)

在渲染开始之前，CPU会设置一些全局值，这些值描述了如何渲染网格。这些值称为“渲染状态”(Render State)。<br>


## 2. 设置渲染状态（Set the Render State）
渲染状态是一种用来定义mesh如何去渲染的全局标志。它包含以下信息：

`` “vertex and pixel shader, texture, material, lighting, transparency, etc. […]” ``

重点：CPU命令GPU绘制的每个网格都将在这些条件下渲染！你可以渲染石头，椅子或剑； 如果在渲染下一个网格之前不更改渲染状态，它们都会获得相同的渲染值（例如材质）。
<html>
<video width="100%" controls="" loop="" preload="" src="/vedios/renderstate.mp4" type="video/MP4"></video>
</html>

准备完成后，CPU最终可以调起GPU并告诉它要绘制什么。 此命令称为：Draw Call。<br>

## 3. Draw Call
Draw call是一个通知渲染**一个**mesh的指令。它是由cpu发起的，由gpu接受，该指令只指向将要渲染的mesh，并不包含任何材质信息，因为材质信息已经通过渲染状态定义了。这些mesh位于VRAM中。
<html>
<video width="100%" controls="" loop="" preload="" src="/vedios/cpu_calls_gpu.mp4" type="video/mp4"></video>
</html>

当这些指令发出去之后，gpu会获取渲染状态的数据（材质、纹理、shader……）和定点数据，通过一些代码将这些数据转换成漂亮的像素到屏幕上。这个处理过程就称之为Pipeline。<br>

## 4. Pipeline
正如文章开头说言，资源或多或少就是一张定点和纹理数据的列表。要将这些转换成令人兴奋的图形，显卡需要基于顶点创建三角形，计算他们怎么放置，绘制纹理像素在三角形上等。这个称之为state,Pipeline States。

基于现在本文阅读到的位置，你会发现大部分工作都是由gpu完成的，但有时他们会说，例如三角形创建和片段创建是由图形卡的其他部分完成的。
> 这个管道示例非常简化，只能被视为粗略概述或“逻辑”管道：每个三角形/像素都贯穿逻辑步骤，但实际发生的情况有点不同。所以请不要太认真，并考虑我在本文底部链接的所有其他来源。或者随时邮寄，推特或facebook我，以便我可以改进动画/解释。:)

以下是硬件绘制一个三角形的步骤示例：

<html>
<video width="100%" controls="" loop="" preload="" src="/vedios/pipeline_overview.mp4" type="video/mp4"></video>
</html>

渲染一般要处理大量的小任务，比如为了把数千个顶点或绘制数百万像素到荧屏上而进行一些计算。至少30fps(希望是)。


在同一时间能处理大量的Pipeline流程是非常必须的，而且不需要每个处理任务（顶点或像素）串行执行也是非常必要的。在以前，处理器有且仅只有一个核心在GPU上。

现代的CPU有4-8个内核,能一次性处理4-8个float操作，假设有64个浮点执行单元，而GPU却有几千个。仅仅比较GPU核心和CPU核心数量是不公平的，因为不同的架构和组织方式。GPU供应商倾向于将“核心”用于最小的执行单元，而CPU供应商则用于高级单元。 第二册介绍了GPU中高级到低级组织的一些细节。

当数据（比如一堆顶点）被放入Pipeline时，转换点/像素的工作被分发到几个核心，然后由很多这些小元素并行形成一张大图片：

<html>
<video width="100%" controls="" loop="" preload="" src="/vedios/pipeline_overview_multicore.mp4" type="video/mp4"></video>
</html>

现在我们都知道GPU可以并行处理。但是CPU和GPU是怎么通信的呢？难道CPU要等待GPU执行完所有的任务并且能接受新的指令？

![gif](/images/tut_cpu_commands_gpu_zzz.gif)

幸好不是！原因是这种通信会产生瓶颈（当cpu不能足够快速地划分指令时）和是并行工作成为不可能。该解决方案是维护一个列表，由CPU添加命令而GPU读取 - 彼此独立！ 此列表称为：Command Buffer。<br>

## 5. Command Buffer
> Command Buffer使得CPU和GPU独立工作成为了可能。当cpu需要渲染某样东西的时候，会把command放置到队列里，当gpu需要渲染的时候，gpu会从队列里取出并执行（FIFO，先进先出）。

<html>
<video width="100%" controls="" loop="" preload="" src="/vedios/commandbuffer_communication.mp4" type="video/mp4"></video>
</html>

顺便一提：有可能有一些比较特殊的command，一个是Draw call,另一个是更换render state。

That’s it for the first book. Now you should have an overview about asset data during rendering, draw calls, render states and the communication between CPU and GPU.