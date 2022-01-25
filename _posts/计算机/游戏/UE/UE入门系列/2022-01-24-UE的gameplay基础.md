---
layout:     article
title:      "UE的gameplay基础"
subtitle:   " \"unreal\""
date:       2022-01-24 11:25:00
author:     "Conerlius"
category: unreal
keywords: unreal
tags:
    - unreal
---

[TOC]

## Unreal的几个基本概念

- GameInstance
- World
- Level
- GameMode
- HUD
- Pawn
- Controller

首先先上一下ue的经典图片，不要求一下子看懂，知道大致流程就好，后面会简单说一下！

![png](/images/computer/game/ue/simple_tutorial/GameFlowChart.png)

### GameInstance

这里可以理解成游戏的单例，一个游戏里默认就只有一个`GameInstance`!所有在这个系列里我们可以单做游戏开发的入口！

> 只是目前而已，后续深入有其他的！

`GameInstance`提供了几个比较实用的可重载的接口！

- ApplicationXXXDelegate 是UE监听的各种事件的委托；eg:进入后台等！
- Init 初始化的函数！
- Shutdown 游戏关闭的函数！
- StartGameInstance 开始！其中会调用`OnStart`
- OnStart 启动！

一般而言，我们都是使用`UPlatformGameInstance`多，`UPlatformGameInstance`就是集成`GameInstance`的！


### World

在Unreal Editor的界面中，目之所及就能看到一个`World Outliner`

![png](/images/computer/game/ue/simple_tutorial/1.png)

> world不是Level（即不是创建的地图！）；一个World里可以有多个Level的！你可以理解这是个宇宙！

![png](/images/computer/game/ue/simple_tutorial/2.png)


### Level

这个就能简单粗暴地先理解成地图，如果`World`是宇宙的话，那么一个`Level`就是一个星球！而星球内就有很多大陆（也是Level）！

### GameMode

一个`World`就对应一个`GameMode`；可以简单地理解成宇宙的基本法则，定义world运行的规则！（包含了玩家出生的规则等）

比较常用的就是指定以下属性

- HUDClass

> 游戏UI界面

- PlayerStateClass

> 为游戏连接的每一个用户的状态数据！（可以是本地的单机的，也可以是走`UE Dedicate Server`的）

- DefaultPawnClass

> 这里暂时可以理解成玩家角色模型的生成蓝图！（暂定，实际上不是！）

- PlayerControllerClass

> 玩家控制器，监听玩家输入，引用玩家hud，角色模型等！

![png](/images/computer/game/ue/simple_tutorial/3.png)

### HUD

玩家的界面；一般的用法是`PlayerControllerClass`接受到玩家的输入后，通过`HUD`对ui进行处理的！

### Pawn

这里暂时可以理解成玩家角色模型的生成蓝图！（暂定，实际上不是！后面会在例子中逐步展开）

### Controller

`PlayerControllerClass`就是集成于它（虽然很多pawn能做的，他也能做，但是还是简单把其当做是控制器就可以了！）。