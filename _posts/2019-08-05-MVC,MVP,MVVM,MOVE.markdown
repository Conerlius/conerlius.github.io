---
layout:     post
title:      "MVC,MVP,MVVM,MOVE"
subtitle:   " \"MVC,MVP,MVVM,MOVE\""
date:       2019-08-05 17:05:00
author:     "Conerlius"
category: 其他
keywords: CSharp
tags:
    - 交流
---

> 对于ui设计模式，好多人都知道大概是怎么回事，去没有很好地区别;<br>
> 下面我们来盘点一下目前比较常见的ui设计模式:
# MVC
> M(Model)<br>
> 独立于UI,它一般用于管理数据
> 
> V(View)<br>
> 图元或视图、UI
>
> C(Control)<br>
> 接收交互，并把交互转换成`Model`或`View`的指令<br>

![png](https://upload.wikimedia.org/wikipedia/commons/thumb/a/a0/MVC-Process.svg/200px-MVC-Process.svg.png)
![png](https://upload.wikimedia.org/wikipedia/commons/f/f0/ModelViewControllerDiagramZh.png)
# HMVC
> 也叫做`LayeredMVC`<br>
> 意义和上面的mvc差不多，不过存在以下的一些区别
> 1. HMVC可以说是按功能块之类的规则进行了划分各管各的
> 2. HMVC如果需要和其他View交互，就必须通过上层Control

![png](https://upload.wikimedia.org/wikipedia/commons/thumb/4/4a/Pac-schema.png/400px-Pac-schema.png)
>> 硬要说MVC和MVC的区别，那就是是否有分层！
# MVA
> `MV`就是`MVC`的`MV`,但是其中将`C`替换成`A`(Adapt)的概念。<br>
> 而View和Model在MVA中是不直连的，但是View和Model却是相互监听的！
> 而且`A`一般是处理成接口的形式
![png](/images/MVA.png)
# MVP
> MVP看起来和MVA很像，但是`presenter`却不是接口！
![png](https://upload.wikimedia.org/wikipedia/commons/thumb/d/dc/Model_View_Presenter_GUI_Design_Pattern.png/220px-Model_View_Presenter_GUI_Design_Pattern.png)
# MVVM
> 这个其实和MVA,MVP差别都不大，但是呢，`VM`是执行的是数据绑定，执行的内容上和`presenter`、`adapter`不同
![png](https://upload.wikimedia.org/wikipedia/commons/8/87/MVVMPattern.png)

# 注
> 本来还有个`Move`的，但是看看wikipedia上没有，就不写了！