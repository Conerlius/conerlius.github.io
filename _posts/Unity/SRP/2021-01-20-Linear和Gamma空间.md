---
layout:     article
title:      "Linear和Gamma空间"
subtitle:   " \"Profile\""
date:       2021-01-20 11:25:00
author:     "Conerlius"
category: unity
keywords: unity,srp
tags:
    - unity
---

## 纲要

- Gamma
- Linear
- Gamma、Linear区别
- Unity中的应用

## Gamma

- 基本原理
- CRT & LCD

### 基本原理

关于`Gamma`的说明，这里就先上一个wikipedia的[link](https://en.wikipedia.org/wiki/Gamma_correction)。如果已经看过link的内容，可以直接跳过该段落，直接到下一个段落。

这里就再简单说明一下`Gamma`

需要展示的像素的颜色计算完成后（每个像素都包含一个代表亮度的值，该值在0和1之间），最后都会输出到显示器上，显示器有一个物理特性就是两倍的输入电压产生的不是两倍的亮度。输入电压产生约为输入电压的指数级的亮度，这个指数就叫做显示器的Gamma。
$$
设备输出亮度 = 电压^{Gamma}
$$
上面的公式，我们知道`Gamma`是用来表征显示器亮度的一个参数。

无论捕捉还是显示设备都不是线性的。因此，需要对`Framebuff`进行一定的处理后才能争取显示。 `Gamma Correction(Gamma矫正) `产生了。

> 显示器是非线性的，但是渲染器确实线性的！

下图是其中比较具有代表性的CRT设备的亮度图：

![png](/images/Unity/srp/gamma1.png)

将x,y轴归一化，并进行模拟：

![png](/images/Unity/srp/gamma2.png)

> https://zhuanlan.zhihu.com/p/111981195 这里也是有相关的说明

进过大量的数据后，得到的是在值大约是2.2;

> 并不是所有的都是2.2，有些可能是1.4或其他的！
>
> http://www6.uniovi.es/hypgraph/color/gamma_correction/gamma_intro.html 这里就有相关的说明

到此，我们知道`gamma`是个什么样的值的，但是gamma是怎么作用的，在哪个阶段生效的！下面我们开始说说原理!

在讲原理前，有几个前提需要提前说明一下

- 最原始的数据是线性的！(其他的后面会说到)
- 最终需要展示原始的视图!

首先需要了解图片的输入到输出过程;

- 原始图像数据(这个是线性的信号)
- 因为会经过`Gamma 矫正`，所有在`Gamma 矫正`前，我们先逆向操作，以保证后面的`Gamma 矫正`后能变成原始图像的

![png](/images/Unity/srp/gamma3.png)


### CRT & LCD

LCD并没有直接继承CRT的特征，但是也会模拟CRT的效果！（是不很无语，感觉是历史遗留问题，但是现实就是这样，至于为什么这里就不深究了！）

## Linear

## Gamma、Linear区别

## Unity中的应用