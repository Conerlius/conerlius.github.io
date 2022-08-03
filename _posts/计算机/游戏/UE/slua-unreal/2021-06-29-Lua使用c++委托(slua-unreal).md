---
layout:     article
title:      "Lua使用c++委托"
subtitle:   " \"slua\""
date:       2021-06-29 11:25:00
author:     "Conerlius"
category: unreal
keywords: unreal
tags:
    - unreal
    - slua
---

## 大纲

- slua的代理对象（回调函数）
- unreal的委托

## slua的代理对象（回调函数）

`cppbinding`提供`TFunction`作为回调函数来使用;

c++.h

```c++
// Fill out your copyright notice in the Description page of Project Settings.

#pragma once

#include "CoreMinimal.h"
#include "slua.h"

using namespace NS_SLUA;
/**
 * 
 */
class CustomDelegate
{
	LuaClassBody()
public:
	CustomDelegate();
	virtual ~CustomDelegate()
    {
        callback = nullptr;
    }
	void setCallback(const TFunction<void(int)>& func);
	void doCall(int num);

	TFunction<void(int)> callback;
	static LuaOwnedPtr<CustomDelegate> Create();
};
DefLuaClass(CustomDelegate)
	DefLuaMethod(setCallback, &CustomDelegate::setCallback)
	DefLuaMethod(doCall, &CustomDelegate::doCall)
EndDef(CustomDelegate, &CustomDelegate::Create)
```

lua使用

```lua
local cd = CustomDelegate()
cd:setCallback(function(i) print("callback from cppbinding",i) end)
cd:doCall(1111)
```

如果需要使用多播，就需要自己写容器去记录保存handle了！用完记得移除`handle`

## unreal的委托

- 单播
`Bind(function)` 设置一个function作为代理回到函数，返回一个`handler`，对应的lua变量被置空后回收内存，但在lua尚未回收时，可能导致回调任然有效。

`Clear()` 立刻删除代理回调

例如

```lua
local Test=import('SluaTestCase');

-- test coroutine
local co = coroutine.create( function()
    ccb = slua.createDelegate(function(p)
        print("LoadAssetClass callback in coroutine",p) 
    end)
    Test.LoadAssetClass(ccb)
    ccb = nil
end )
coroutine.resume( co )
```

- 多播

`Add(function) `添加一个function作为代理回到函数，返回一个`handler`

`Remove(handler) `移除对应的函数代理

`Clear() `清空所有代理

`多播`和`单播`，他们的使用基本相同，但都需要注意需要移除对应的`handle`，保证没有内存泄露。