---
layout:     article
title:      "Lua文件加载"
subtitle:   " \"slua\""
date:       2021-06-28 11:25:00
author:     "Conerlius"
category: unreal
keywords: unreal
tags:
    - unreal
    - slua
---

## 大纲

- 文件加载的接口
- 文件的使用

## 文件加载的接口

```c++
LuaState->setLoadFileDelegate(&LuaFileLoader);
```

`setLoadFileDelegate`是slua提供的修改`lua`执行`require`加载时的接口,（这里要说明一下，只有`require`才有效果哦！）

一下是简单的`loader`实现

```c++
TArray<uint8> ULuaMgr::LuaFileLoader(const char* fn, FString& filepath)
{
	IPlatformFile& PlatformFile = FPlatformFileManager::Get().GetPlatformFile();
	FString path = FPaths::ProjectContentDir();
	path = FPaths::Combine(path, TEXT("/Lua"));
	FString filename = UTF8_TO_TCHAR(fn);
	path /= filename.Replace(TEXT("."), TEXT("/"));
	TArray<uint8> Content;
	TArray<FString> luaExts = { UTF8_TO_TCHAR(".lua"), UTF8_TO_TCHAR(".luac") };
	for (auto& it : luaExts) {
		auto fullPath = path + *it;

		FFileHelper::LoadFileToArray(Content, *fullPath);
		if (Content.Num() > 0) {
			filepath = fullPath;
			return MoveTemp(Content);
		}
	}

	return MoveTemp(Content);
}
```

## 文件的使用

要执行文件就简单多了，使用`doFile`函数就可以！

```c++
luaState->doFile("Launch");
```

> 或者通过`doString('require('luaFileName')')`来启动！