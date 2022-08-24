---
layout:     article
title:      "Unity Shader Variant应用规则"
subtitle:   " \"unity\""
date:       2022-08-24 11:25:00
author:     "Conerlius"
category: unity
keywords: unity
tags:
    - unity
---

## 起因

本来是没有要记录的打算的，但是昨天有同事突然说在`Android`上`FrameDebug`到的物体使用的`Shader Variant`没有任何`keyword`，以下是Android上的截图

![Android的截图](/images/computer/game/unity/unity_shadervariant_4.png)

> 之前还遇到过在`FrameDebug`中没有`Event`的！

正常情况下，这个`Shader Variant`应该有以下的`keyword`

```
simplelit, SubShader #0
FOG_LINEAR LIGHTMAP_ON LIGHTMAP_SHADOW_MIXING SHADOWS_SHADOWMASK _ENABLE_GI _MAIN_LIGHT_SHADOWS _MAIN_LIGHT_SHADOWS_CASCADE _SHADOWS_SOFT
```

采样工具收集到的变体集基本如下：
```yaml
- first: {fileID: 4800000, guid: a508dd1fe46c65e43b13d0815c9ca93d, type: 3}
    second:
      variants:
      - keywords: 
        passType: 8
      - keywords: _ALPHATEST_ON
        passType: 8
      - keywords: 
        passType: 11
      - keywords: _ALPHATEST_ON
        passType: 11
      - keywords: 
        passType: 13
      - keywords: _MAIN_LIGHT_SHADOWS _MAIN_LIGHT_SHADOWS_CASCADE _SHADOWS_SOFT _SHADOW_COLOR_ON
        passType: 13
      - keywords: LIGHTMAP_ON _MAIN_LIGHT_SHADOWS _MAIN_LIGHT_SHADOWS_CASCADE _SHADOWS_SOFT
          _SHADOW_COLOR_ON
        passType: 13
        ...
      - keywords: FOG_EXP LIGHTMAP_ON _ALPHATEST_ON _MAIN_LIGHT_SHADOWS _MAIN_LIGHT_SHADOWS_CASCADE
          _SHADOWS_SOFT
        passType: 13
        ...
      - keywords: FOG_EXP LIGHTMAP_ON _ENABLE_GI _MAIN_LIGHT_SHADOWS _MAIN_LIGHT_SHADOWS_CASCADE
          _NORMALMAP _SHADOWS_SOFT _SHADOW_COLOR_ON
        passType: 13
      - keywords: _ALPHATEST_ON _ENABLE_GI _MAIN_LIGHT_SHADOWS _MAIN_LIGHT_SHADOWS_CASCADE
          _SHADOWS_SOFT
        passType: 13
        ...
      - keywords: FOG_EXP LIGHTMAP_ON _ALPHATEST_ON _ENABLE_DROPMAINHIGHLIGHT _ENABLE_GI
          _MAIN_LIGHT_SHADOWS _MAIN_LIGHT_SHADOWS_CASCADE _SHADOWS_SOFT
        passType: 13
```

而另外一个`ShaderVariant`情况就不大一样了！以下是PC上的截图

![Standalone的截图](/images/computer/game/unity/unity_shadervariant_5.png)

但是Android上使用的`keyword`组却是以下的样子：

```
_Lit, SubShader #0
FOG_LINEAR LIGHTMAP_ON SHADOWS_SHADOWMASK _ADDITIONAL_LIGHTS _ENABLE_GI _MAIN_LIGHT_SHADOWS _MAIN_LIGHT_SHADOWS_CASCADE _MATCAPMODE_MULTIPLY _SHADOWS_SOFT
```

> 和PC上的对比，缺少了`LIGHTMAP_SHADOW_MIXING`!!!

为什么？

## 原则

- 首先是`keyword`的顺序不影响选择
- `keyword`最大交集多优先（交集必须和选择的变体`keyword`数量一致！）
- `ShaderVariantCollection`顺序前面优先

下面是例子：

假设是在`Forward`模式下，已经进行打包编译的`keyword`组合有一下：

```
// 按顺序从上到下
1. A B C
2. A B E
3. no keyword
```

| 正确渲染使用的`keyword`组合 | 实际调用的`keyword`组合 | 说明 |
| :-: | :-: | :- |
| A B C | A B C | 正常匹配 |
| A B | no keyword|	没有匹配的变体|
| A B C D | A B C |	调用交集中`keyword`数量多的变体 |
| A B C E| A B C | `A B C`和`A B E`交集数量一致，但是因为`A B C`排在前面，优先使用了|

到这里，基本就能理解为什么出现第一张图的原因了！而且从svc中匹配的结果也是没有最大匹配的交集，所以智能选择`no keyword`了!

> 前面也提到遇到UI渲染的时候在`FrameDebug`里连`Event`都没有找到，原因就是除了`keyword`找不到外，连`lightmode`都没有匹配，即`Forward`或者`Add`之类的！！