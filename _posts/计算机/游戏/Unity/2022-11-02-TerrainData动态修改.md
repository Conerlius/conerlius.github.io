---
layout:     article
title:      "TerrainData动态修改"
subtitle:   " \"unity\""
date:       2022-11-02 11:25:00
author:     "Conerlius"
category: unity
keywords: unity
tags:
    - unity
---
[TOC]

## 简单的创建

- 空模板
- 通过 `HeightMap` 创建
- 通过 `raw` 文件创建

### 空模板

空白模板的创建有两种方式：
- 使用右键菜单中的 `3D Object` --> `Terrain` 直接创建

![png](/images/computer/game/unity/terrain/create1.png)

- 通过 `Terrain ToolBox` 创建

`Terrain ToolBox` 的包需要通过 `Package Manager` 下载安装。
安装完成后，在顶部菜单 `Window` 中就能找到该工具了！

![png](/images/computer/game/unity/terrain/create2.png)

![png](/images/computer/game/unity/terrain/create3.png)

具体的参数说明可以自行在官方文档中找到！

### 通过 `HeightMap` 创建

> `HeightMap` 的方式只能通过 `Terrain ToolBox` 去创建！

`Terrain ToolBox` 提供了3种创建的方式
![png](/images/computer/game/unity/terrain/create4.png)

#### Global

使用单张`HeightMap`构建；这里有个高度图就可以了，不细说！

#### Batch

值得注意的是[官方文档](https://docs.unity3d.com/Packages/com.unity.terrain-tools@5.0/manual/toolbox-import-heightmap.html#import-modes),这里使用的文件名个文件格式！
![png](/images/computer/game/unity/terrain/create5.png)

> 在houdini做好切割的文件名和格式都不符合unity的要求，除非直接使用houdini的unity插件！我这里为了排除dcc对程序流程的印象，所以没有直接使用houdini的unity插件！

但是！
格式不一样就不代表就不能houdini中切割了！

> Unity中也有很多第三方工具可以直接切割的！这个可以自己找找，挺好检索的！

在切割之前，需要了解一下houdini和unity在一些格式和要求！

https://www.sidefx.com/docs/unity17.0/_terrain.html#Terrain_Size

按照要求设置后，使用`Heightfield_output`节点输出，我这里简单设置一下！
![png](/images/computer/game/unity/terrain/create6.png)
![png](/images/computer/game/unity/terrain/create7.png)

> 切割完成后，自己写个小工具按照`xy`的索引，用`global`的方式创建并拼接起来就可以了！
> 
> 需要注意的是，`Terrain Height`的参数并不是默认的，我这里单个导入使用的是110
> 
> 还有就是在使用的时候，要注意的是16bit fixed！
>  
> https://forum.unity.com/threads/floating-point-textures.821553/

至于使用`PhotoShop`将png转成raw!!重新命名后就可以直接使用 `Batch` 的方式导入，我简单测试了一下，不行！纹理大小不对，不知道以后有没有能发现解决方法！（就算取巧使得导出的大小匹配，加载出来也是平面！）

> 需要注意的是，有一个`HeightMap Byte Order`，在`Ps`中要和Unity的对应！
> 导入的时候注意在unity中的纹理格式！

#### Tiles

### 通过 `raw` 文件创建

上文有说到从png转到raw，所以在这里使用raw的形式导入就可以了。

Unity在使用`Terrain`处理地形完成后，也能导出`raw`
![png](/images/computer/game/unity/terrain/create8.png)


## 1X1 Tile VS nXn Tile

这里就简单通俗点比较一下性能的差异，没有太细！

![png](/images/computer/game/unity/terrain/create9.png)

## 数据修改

### 编辑器修改

目前看到公司的wiki说，如果切成 `Tile` 后，一个刷子一次刷一个Tile，不能实现跨 `Tile`， 我简单测试了一下，确实如此，但是如果是流程上进行调整的话，还是可以做到的，统一一个 `Tile` 刷完后，导出`Raw`或者`HeightMap`后，重新在使用`Tile`的形式导入生成就可以了！
> 即保留一个`1X1 Tile`作为原始的资源，然后再输出Tile

待完成

### 运行时修改

目前能想到的只有一下几点：
- 修改`Terrain` 高度信息
- 修改树种植的区域和相关参数
- 修改草的种植区域和相关参数
- 修改TerrainLayer的信息
- Terrain地形和Mesh的融合

这里是各个点的尝试及实现过程。

#### 修改`Terrain` 高度信息
#### 修改树种植的区域和相关参数
#### 修改草的种植区域和相关参数
#### 修改TerrainLayer的信息
#### Terrain地形和Mesh的融合


待办。

## 扩展使用

由于一些游戏想法延伸出来的一些技术问题，及解决方案。

### Houdini输出信息在Terrain中的处理

