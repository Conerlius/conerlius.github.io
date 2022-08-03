---
layout:     article
title:      "Unreal ModuleRules的一些使用技巧"
subtitle:   " \"unreal\""
date:       2022-08-03 14:00:00
author:     "Conerlius"
category: unreal
keywords: unreal
tags:
    - unreal
---

在UE中，建立`Module`的时候，通常都会伴随一个`XXXX.build.cs`文件，使用C#处理工程的配置！

今天问了一下同事，有些配置有些人不大清楚（以为是都知道，现在发现还是简单记录一下比较好！）

```c++
public class XXXXX : ModuleRules
{
    public XXXXX(ReadOnlyTargetRules Target) : base(Target)
    {
        PCHUsage = PCHUsageMode.UseExplicitOrSharedPCHs;
        ...
    }
}
```

这里简单说明一些稍微常见的，其他的可以看源码！

|函数/属性/字段|描述| 用例 |
|- |-|-|
| OptimizeCode | 代码编译优化相关|OptimizeCode = CodeOptimization.Never|
|CppStandard | cpp的版本| CppStandard = CppStandardVersion.Latest|
|bEnableExceptions | 启用异常处理 | bEnableExceptions = true|


这是官方的文档说明！

https://docs.unrealengine.com/4.27/zh-CN/ProductionPipelines/BuildTools/UnrealBuildTool/ModuleFiles/