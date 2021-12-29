---
layout:     article
title:      "unity shader编译打包丢失变体"
subtitle:   " \"unity\""
date:       2021-12-28 11:25:00
author:     "Conerlius"
category: unity
keywords: unity
tags:
    - unity
    - shader
---

## 大纲
- 背景
- 背景下解决方案
- 问题

## 背景

unity的`shader_feature`对于在代码中`Enable`或者项目中各种复杂动态坏境的情境下，打包就成了噩梦！！

如果是使用`multi_compile`之流，那么$n^2$的变体量就可怕了！

所以我想了一下，使用的是的`multi_compile`，但是利用`Unity`提供的一套机制，在`multi_compile`后生成的所有变体中尝试剔除一下不使用的`shadervariant`，这样从理论上应该能解决问题！

## 背景下解决方案

先给出一下`Unity`的接口
```c#
public void OnProcessShader(Shader shader, ShaderSnippetData snippet, IList<ShaderCompilerData> data)
```

> `snippet`是当前的编译信息
> 
> `data`存储了需要编译的所有变体，如果有不需要的，直接从该列表中移除就可以了！

## 问题

但是在使用了一段时间后，发现有个大坑（估计是我踩到了`违反unity使用范例`吧）,然后发现带`FOG_EXP2` `keyword`的变体在打包`shader ab`的时候总是没有能打包进`ab`！

这口锅就有点大了!

- 猜想一：有没有可能是被自己剔除了?
  因为`Shader`的编译是经过自己写的`OnProcessShader`剔除后才编译的，想着会不会是自己把该变体剔除了；但是结果打印了一下，发现不是！
- 猜想二：有没有可能是因为没有使用`multi_compile`去定义`FOG_EXP2`?
  本来`Shader`里对雾使用的是`#pragma multi_compile_fog`,在想是不是没有生效？，于是尝试使用了
  ```hlsl
  #pragma multi_compile _FOG
  #pragma multi_compile FOG_LINEAR FOG_EXP FOG_EXP2
  ```
  但是结果还是和猜想一一样！在`unity`的`OnProcessShader`调用的时候，并没有带有`FOG_EXP2`的变体
- 猜想三: 其他的`OnProcessShader`把`FOG_EXP2`给剔除了？
  全局检索了一下，可见的只有两个，一个是我写的,一个`URP`的，但是`URP`的那个已经被我禁掉了！说明已经没有其他的改动了！
- 猜想四：`Unity`有其他设置搞掉了这个？
  ![ ](/images/computer/game/unity/unity优化/unity_exp2_drop.png)
  在`unity`的`Project Setting`>`Graphics`中，有个选项，发现开启了这个后 ，`OnProcessShader`就能把到有`FOG_EXP2`的变体传输过来，并且进行了正确的打包！