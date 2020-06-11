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

## 渲染管线

渲染管线的几个基本流程

```mermaid
graph LR;
    A[应用程序阶段]
    B[几何阶段]
    C[光栅化阶段]
    A-->B
    B-->C
```

### 应用程序阶段

> 计算机CPU的处理，包含CPU传递内容到`VRAM`(显存)

### 几何阶段

> 基本都是对顶点的处理

```mermaid
graph LR;
    A[模型视点变换]
    B[顶点着色]
    C[投影]
    D[裁剪]
    E[屏幕映射]
    A-->B
    B-->C
    C-->D
    D-->E
```

- 模型视点变换
> 将模型自身坐标转换到视觉空间坐标
> 
> 借助`《real-time rendering》`的图
![png](/images/shader_tutorial/2.png)
- 顶点着色
> 
- 投影
> 
- 裁剪
> 超出视觉空间的进行裁剪
- 屏幕映射
> 映射到屏幕空间

### 光栅化阶段

```mermaid
graph LR;
    A[三角形设定]
    B[三角形遍历]
    C[像素着色]
    D[融合]
    A-->B
    B-->C
    C-->D
```

- 三角形设定
- 三角形遍历
- 像素着色
- 融合

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
