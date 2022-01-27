---
layout:     article
title:      "GetPrivateStaticClass报错"
subtitle:   " \"unreal\""
date:       2022-01-27 11:25:00
author:     "Conerlius"
category: unreal
keywords: unreal
tags:
    - unreal
---

## 问题现象

![png](/images/computer/game/ue/problems/1.png)


## 解决过程：

- 一

过程：删除Intermediate后重新Generate，然后编译

结果：还是一样的异常

- 二

过程：在报错的类里添加指定的方法
```c++
UNetworkObjectBase::GetPrivateStaticClass(void)
UNetworkObjectBase::UNetworkObjectBase(class FVTableHelper &)
```
结果：还是一样的异常！


- 三

过程：添加XXXXX_API

结果：编译正常

