---
layout:     post
title:      "Sounds in Unity"
subtitle:   " \"Unity Sounds\""
date:       2019-08-05 14:00:00
author:     "Conerlius"
header-img: "img/post-bg-2015.jpg"
tags:
    - Unity;
    - Tool;
---

# unity比较常见的声音插件
* Fmod
* Wwise
* ADX2
* Unity
# 各个插件的收费价格比较
> 价格上，unity就不参与了，毕竟unity是集成了音效的！

Wwise

  | 入门级 | 专业级 | 白金级 | 企业级 
--- | --- | --- | --- | --- |
 制作成本 | 单项目&lt;150K | 单项目&lt;1.5M | 单项目&gt;1.5M | 多项目 
 媒体素材 | 500 | 无限制 | 无限制 | 无限制 
 费用 | 免费 | 7200+$/项目 | 21600+$/项目 | 议价 

Fmod

| | Indie | Basic | Premium |
|---|---|---|---|
| 制作成本 | 单项目&lt;500K | 500K&lt;单项目&lt;1.5M | 单项目&gt;1.5M |
| 费用 | 免费 | 5000$/项目 | 15000$/项目 |

ADX2

| | win/mac | ios/android |
|---|---|---|
| 费用 | 79/99$ | 议价 |


# 各插件的优势
1. Unity

> unity其实也是集成的fmod，不过unity开放的api比较有限
> 易于理解的API，如AudioSource，AudioClip
> 使用Unity的标准分析器进行优化
> 与AssetBundle的高亲和力

2. ADX2

> 通常可以做任何事情
> 专有的编解码器压缩率很高
> Android上出色的低延迟播放（约15毫秒，而标准40毫秒）
> Profiler易于使用
> 包装系统也可提供

3. Wwise

> 通常可以做任何事情
> 由于它是世界标准，有各种插件等
> 对VR解决方案的反应也很快
> 许多布局预设适合用户的目的
> 官方教程非常详细

4. Fmod

> UX易于理解的创作工具
> 声音控制，3D声音，随机播放，互动...

# 各插件的劣势
1. Unity

> 在尝试控制精细发音时，有必要编写脚本
> 当Wav包含在版本控制中时，项目变得臃肿，因此存在许多问题，例如将其作为子模块或将其变为AssetBundle

2. ADX2

> 在商业中，Unity附带的样品等很少
> 限量版没有Mac编辑器

3. Wwise

> 驻留内存量略大于其他驱动程序

4. Fmod

> 文档非常差