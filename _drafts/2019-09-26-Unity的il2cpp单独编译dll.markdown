---
layout:     article
title:      "Unity的il2cpp单独编译dll"
subtitle:   " \"il2cpp\""
date:       2019-09-26 15:00:00
author:     "Conerlius"
category: unity
keywords: unity,il2cpp
tags:
    - unity
---

## il2cpp是什么

## unity的il2cpp命令

[link](https://blogs.unity3d.com/cn/2015/05/06/an-introduction-to-ilcpp-internals/)

## 怎么查看il2cpp后的文件

首先需要让unity通过il2cpp生成相应的cpp文件，一般是在`project_Path\Temp\StagingArea\Data\il2cppOutput`下，而项目的代码通常也是在`Assembly-CSharp.cpp`文件里，有文本打开后，因为生成的文件有备注，所以直接使用C#的类名+方法名称就能定位到生成的cpp对应方法！