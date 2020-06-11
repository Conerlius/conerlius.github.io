---
layout:     article
title:      "unity SRP下shader"
subtitle:   " \"Shader教程\""
date:       2020-06-10 17:00:00
author:     "Conerlius"
category: shader
keywords: shader,unity
tags:
    - shader
---

[TOC]

## Unity的自定义管线

### Unity 自定义管线的构建（以urp为例）



### unity 自定义管线的基本流程

```mermaid
classDiagram
    class CustomRenderPipeLineAsset~RenderPipelineAsset~{
        +CreatePipeline() RenderPipeline
    }
    class CustomRenderPipeLine~RenderPipeline~{
        +CustomRenderPipeLine()
        +Render(ScriptableRenderContext context, Camera[] cameras)
    }
    class MainCameraRender{
        +RenderCamera(ScriptableRenderContext context, Camera camera)
    }
    CustomRenderPipeLineAsset-->CustomRenderPipeLine : 创建
    CustomRenderPipeLine-->MainCameraRender : 调用
```

如果上图失败
![png](/images/shader_tutorial/1.png)

## Unity Shader HLSL的基本语法

### 基本的shader结构

### Shader变量

### Shader的方法

### Shader的特性(shader_feature)

### Shader的Property声明

### Shader Property的前置标签

### Shader展示界面(ShaderGui)
