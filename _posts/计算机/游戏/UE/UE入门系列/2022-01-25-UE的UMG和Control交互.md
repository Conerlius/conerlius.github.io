---
layout:     article
title:      "UE的UMG和Control交互"
subtitle:   " \"unreal\""
date:       2022-01-25 12:25:00
author:     "Conerlius"
category: unreal
keywords: unreal
tags:
    - unreal
---

感觉说明一下几个类的结构，大家就知道怎么处理了！

## delegate

`delegate`的方式相信很多人都能想得到吧？但是这种方式不是这次要讲的！

## 直调

UI和其他几个类型的交互离不开这几个类型！

- UUserWidget
- AHUD
- APlayerController
- AGameModeBase

### UUserWidget

前面的文章 [UE的UMG简单使用]({% post_url 2022-01-24-UE的UMG简单使用 %}) 中已经说明了UMG绑定相关的了！