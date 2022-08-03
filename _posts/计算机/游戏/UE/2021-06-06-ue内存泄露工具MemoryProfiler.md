---
layout:     article
title:      "ue内存泄露工具MemoryProfiler"
subtitle:   " \"MemoryProfiler\""
date:       2021-06-16 11:25:00
author:     "Conerlius"
category: unreal
keywords: unreal
tags:
    - unreal
---

## 大纲

- MemoryProfiler

下载源码

Setup.bat

GenerateProjectFiles.bat

打开Engine\Source\Programs\MemoryProfiler2\MemoryProfiler2.sln 工程文件

Engine\Programs\MemoryProfiler2\Binaries 目录

修改引擎和项目对应的xxx.Target.cs

```
bUseMallocProfiler = true;
bOmitFramePointers = false;

```