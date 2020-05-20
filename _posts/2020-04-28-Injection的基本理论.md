---
layout:     post
title:      "Injection的基本理论"
subtitle:   " \"physic\""
date:       2020-04-28 17:50:00
author:     "Conerlius"
category: unity
keywords: Injection
tags:
    - Injection
    - CSharp
---
* content
{:toc}

## Injection的基本理论

### 代码的大致运行流程

这里就假设你是一个会写C#的，我们通过`csc`或者`donet`指令将写好的C#代码编译成我们需要的`exe`或`dll`，然后目标机器启动加载编译的代码`dll`之类的。

但是如果dll没有加密，并且我们也需要在其中注入自己想要的执行代码的时候（比如我想把原本打印的信息修改了~别理会为什么！），那么我们从理论上应该怎么做呢？

如下图所见(如果项目需要跑脚本的话)
![png](/images/injection/injection_1.png)

基于上图，我们试想一下，如果在生成`dll`后，对`dll`进行解释，然后再`dll`里加入我们想要的代码后，再交给目标机器去执行，这样就能达成我们想要的效果了！

如下图所示

![png](/images/injection/injection_2.png)

按照这样的思路，如果我们注入的是`hotfix`，那会如何？

## `Mono.ceil`

`Mono.Cecil`库对进行C#层编译出来的`dll`程序集进行了解释，还能写入，而且还支持`IL`！

详细的`mono.ceil`的使用，请移步[Injection的基本使用]({% post_url 2020-04-28-Injection的基本使用 %})

