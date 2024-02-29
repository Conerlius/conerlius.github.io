---
layout:     article
title:      "Unreal found Conficting Redirects"
subtitle:   " \"unreal\""
date:       2024-02-29 14:00:00
author:     "Conerlius"
category: unreal
keywords: unreal
tags:
    - unreal
---

## 事故现场

在 `cook content` 或者 `package` 的时候，出现以下异常：

```
UATHelper: Cooking (Windows): LogInit: Display: LogCoreRedirects: Error: AddRedirect(Engine) found conflicting redirects for /Script/...
```

## 解决

事故的原因是因为 `class` 重定向出现了异常！

相关配置文件（如果是项目工程的话）的路径是在

```
{项目工程根目录路}/Config/DefaultEngine.ini
```

找到 `[CoreRedirects]` 后，能看到中其中重定向的类型;将有问题的删除即可！

![png](/images/computer/game/ue/problems/found_Conficting_Redirects.jpg)

我的问题如上图所示，一个类型指定了多种，导致混乱了！删除了无效的就可以了！
