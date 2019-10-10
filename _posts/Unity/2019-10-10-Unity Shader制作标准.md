---
layout:     post
title:      "Unity Shader制作标准"
subtitle:   " \"制作标准\""
date:       2019-10-10 16:00:00
author:     "Conerlius"
category: unity
keywords: unity,shader
tags:
    - unity
    - shader
---

## 材质球标准
### 命名
- 英文（或拼音）、_、数字组成
- 不能出现中文的名称

### 材质球贴图
#### 贴图文件格式和尺寸
- 原始贴图不带通道采用24位tga或者jpg格式，带通道的为32位tga或者png，尺寸最大别超过1024，贴图文件尺寸须为2的N次方 （8、16、32、64、128、256、512、1024）最大贴图尺寸不能超过1024x1024，特殊情况下尺寸可在这些范围内做调整。

#### 贴图材质应用规则
- 贴图不能为中文命名，不能有重名；
- 材质球命名与物体名称一致，材质球的父子层级的命名必须一致；
- 战斗中的角色，除了特殊部件（翅膀等）一律不使用透明材质；贴图长宽相等，符合pot，128-256为主，最大不超过512；
- 同一个贴图除特殊情况外，都只能对应一个材质球；

### 材质球的GPU Instancing和Batch
- 优先Batch,其次才是Instance；

### Shader
#### Shader的应用规则
- Unity默认自带的Shader不能直接使用；如果需要使用，应该从Unity里拷贝出来，修改前置归类后才可放入工程中使用；
- Unity的Surface Sahder能不用就不用！优先使用Vertex Fragment Shader;
- Shader文件应该进行功能性分类（eg:战斗才使用的放置在Fight目录）；
- 从其他地方弄来的shader需要先经过检查没问题后方可使用，不可以直接使用
  
#### Shader的书写规则
- Shader的`Shader "Features/NoFog_Unlit_name"`应该有归类及其特性说明；
- 如果该shader不需要使用Fog，就关闭fog(`Fog {Mode Off}`)；其他特性雷同；
- Shader尽可能不要多Pass；
- Shader的`FallBack`尽可能不要使用unity默认的（或者不使用`FallBack`）；如果没有十足的把握，可以改为`FallBack "Custom/DiffuseMy"`；
- Shader里尽可能使用`GPU Instancing`;
- 尽可能使用特性对应的shader，eg:如果不需要fog的就不要使用带有fog特性的shader；
- Shader的变种应该尽可能控制在1个，如果一定要做多个，尽可能控制在10个以内；
- Shader的高阶、复杂的函数尽可能不使用（eg:sin、cos等）；