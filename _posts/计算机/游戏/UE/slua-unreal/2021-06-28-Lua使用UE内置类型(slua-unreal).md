---
layout:     article
title:      "Lua使用UE内置类型"
subtitle:   " \"slua\""
date:       2021-06-28 16:25:00
author:     "Conerlius"
category: unreal
keywords: unreal
tags:
    - unreal
    - slua
---

## 大纲

- cppbinding
- 调用

## cppbinding

`slua`提供基于模板展开的lua接口绑定方法，我们称其为`cppbingding`，通过使用`cppbinding`，你可以在尽量不修改目标类文件（无侵入）的情况，将`c++`类和方法导出给lua使用。使用`cppbinding`方法导出的lua接口不使用反射机制，相当于你手写lua接口导出，所以在效率和速度上都是最优的，不用担心效率损失，因为你自己手写`lua`接口导出代码也不会再简洁了。

在`slua`的源码中，找到一下的代码:

![](/images/computer/game/ue/slua/2.png)

通过上图，我们可以了解到，`UClass`、`UScriptStruct`和`UEnum`都是直接使用了`unreal`内置的反射去进行的！所以大部分`unreal`的内置类都不需要我们手动进行`Regist`到lua中才能使用了！

## 调用

在说使用之前，需要了解到在`Unreal`的类型反射中，一些类也是不需要使用`U`、`S`和`F`开头就能反射到的，所以在`lua`中调用的时候，也是不需要加上这些的！
eg：`UParticleSystem`，在`lua`中使用时是`import('ParticleSystem')`!


```lua
print("Lua Launch is Calling……")

local KismetSystemLibrary = import("KismetSystemLibrary")
print(KismetSystemLibrary)
-- 获取粒子系统
local ParticleSystem = import'ParticleSystem'
print(ParticleSystem)
-- 获取World
local World = import'World'
print(World)
```