---
layout:     article
title:      "LuaState(slua-unreal)"
subtitle:   " \"slua\""
date:       2021-06-22 12:25:00
author:     "Conerlius"
category: unreal
keywords: unreal
tags:
    - unreal
    - slua
---

## 大纲

- 简述LuaState
- LuaState的创建和使用

## 简述LuaState

`luastate`是可见的`lua vm`实例，在`luastate`上还有一层`globalstate`的，`luastate`之间是存在环境隔离的，只有少部分是可以共享的；以下是`luastate的结构`

![](/images/computer/game/ue/slua/1.png)

在上面的代码中，能看到一个比较重要的点：

- 共享的`global_State`

通常在lua的使用中，一般而言一个`thread`使用的是一个`lua state`,而`lua state`之间的数据共享，除非你有其他的思路，要不然基本都比较麻烦；目前知道的是`skynet`是改过后，才实现`luastate`间通信的！

## LuaState的创建和使用

这里只说`slua-unreal`中的`luastate`方面的！

- LuaState的创建

```c++
luaState = new NS_SLUA::LuaState("MainState", gameinstance);
```

这里的`GameInstance`是必须的，`MainState`指的是当前创建的`luastate`的名称。`slua-unreal`一般是建议在游戏的`GameInstance`中继承`LuaState`,我这里仅仅是说明，并不是一定要这样！

上面创建完成`luastate`后，还需要对`luastate`进行初始化和一些选择性的处理！

```c++
luaState->init()
```

到这里，`luastate`的简单创建已经完成了！

- LuaState的使用

前文的`luastate`已经创建完成了，那么要如何使用？这里先简单说一下：

```c++
luaState->doString("printf('hello world')")
```

直接通过`dostring`方法就能使用了！

> 还有其他的方式，比如调用lua文件等，这里就不做说明，会在其他文档作说明！