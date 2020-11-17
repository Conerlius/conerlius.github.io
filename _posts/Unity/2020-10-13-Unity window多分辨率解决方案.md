---
layout:     article
title:      "Unity window多分辨率解决方案"
subtitle:   "window多分辨率"
date:       2020-10-12 18:22:00
author:     "Conerlius"
category: unity
keywords: unity
tags:
    - python
    - unity
---

## 说明

unity的最近的版本因为移除了以前的多分辨率启动选择界面，改成了在打包`exe`的时候直接选择，不能像以前一样在打包完成后，按需要调整启动的窗口大小了！

经过一系列的测试，常见的方法有以下几个：

- 修改注册表
- 调整启动参数

### 调整启动参数

通过逆向，我找到了一下几个可以通过`cmd`传递的参数

- screen-fullscreen 是否全屏

> -screen-fullscreen false

- screen-quality 质量

> -screen-quality 1

- screen-width width 宽

> -screen-width 960

- screen-height height 高

> -screen-height 640

- monitor index 模拟

> -monitor 1

这些是目前unity支持的！更加具体的参数和数据说明可以自行百度了！

## demo

```cmd
UnityProj.exe -screen-fullscreen false -screen-width 960 -screen-height 640
```