---
layout:     article
title:      "ConsoleCommand的指令添加方案"
subtitle:   " \"unreal\""
date:       2024-05-10 11:25:00
author:     "Conerlius"
category: unreal
keywords: unreal
tags:
    - unreal
---

## 扩展说明

### 指令集（Set）的创建

在 `/Game/Editor/` 的目录下创建一个 `UAbilitySystemCheatExt` 的子蓝图对象

![png](/images/computer/game/ue/tools/cheat_ext_create.jpg)

![png](/images/computer/game/ue/tools/cheat_ext_name_spell.jpg)

> 关于命名，是和提示相关的；
> 
> 插件会剥离 `BP_` 字符，然后将剩下的名称当做前缀去调用函数；
> 
> eg: `BP_ASC` ，那么在 `Command` 中需要使用 `ASC.` 获取到能执行的指令！
> 
> 文件名是基于整个 `/Game/Editor/` 做的不重名添加，重名的话，没试过，但是如果是在子目录的话，理论应该可以！
> 

### 指令函数

指令集添加完成后，打开自定义的指令集蓝图，在其中添加自定函数即可！

![png](/images/computer/game/ue/tools/cheat_ext_set_item_create.jpg)

虽然完成了函数的创建，但是这个函数尚无法反射到command使用，还需要进行以下的配置：

![png](/images/computer/game/ue/tools/cheat_ext_set_item_config.jpg)

> `Exec` 的选项必须要勾选上
> 
> `Description` 是选填，用于在 `Command` 窗口找到该指令的时候，进行提示！
> 

完成以上两步后， `Compile` 蓝图，然后运行游戏，打开 `Command` 后就能看到该指令了！

> 注意： 一定要 `Compile` 目前写的代码里是依赖于蓝图的编译完成事件的（虽然直接运行的时候Editor也会执行蓝图编译，但还是建议手动点一下编译！！！）

![png](/images/computer/game/ue/tools/cheat_ext_result.jpg)

## 插件的内部流程说明

插件内部使用的是 `CheatManager` ，在Character构建 `CheatManager` 的时候，添加Ext进行使用！

### CheatManager的构建回调

为了解耦，使用的是 `UGameInstanceSubsystem` 进行注册！

```cpp
void UCheatRegisterSystem::Initialize(FSubsystemCollectionBase& Collection)
{
    Super::Initialize(Collection);

#if UE_WITH_CHEAT_MANAGER
    CheatManagerCreateHandle = UCheatManager::RegisterForOnCheatManagerCreated(
        FOnCheatManagerCreated::FDelegate::CreateUObject(this, &UCheatRegisterSystem::OnCheatManagerCreate));
#endif
}
```

`UE_WITH_CHEAT_MANAGER` 是Unreal自带的一个宏，能有效地避免作弊指令等对shipping环境的影响！

### UCheatManagerExtension的蓝图遍历添加

在 `CheatManager` 创建后，将项目工程中构建的指令集 `UAbilitySystemCheatExt` 进行添加，需要对资源进行遍历！

```cpp
        UObjectLibrary* ObjectLibrary = UObjectLibrary::CreateLibrary(UObject::StaticClass(), true, GIsEditor);
        ObjectLibrary->AddToRoot();
        ObjectLibrary->LoadAssetDataFromPath(TEXT("/Game/Editor"));
        ObjectLibrary->LoadAssetsFromAssetData();
        TArray<FAssetData> AssetDataArray;
        ObjectLibrary->GetAssetDataList(AssetDataArray);
        for (auto AssetData : AssetDataArray)
		{
			auto ObjectPath = AssetData.GetSoftObjectPath();
			UObject* obj = ObjectPath.ResolveObject();
			auto bp = Cast<UBlueprint>(obj);
			CheatManager->AddCheatManagerExtension(NewObject<UAbilitySystemCheatExt>(CheatManager,bp->GeneratedClass));
		}

		ObjectLibrary->RemoveFromRoot();
```

完成了上述的功能后，运行游戏时已经能在 `ConsoleCommand` 中直接输入指令执行了！但是有一个问题：

> 它不会提示！！！！

### 添加ConsoleCommand提示

按照官方的文档，以下是一个简单的例子：

```cpp
static void ShowGivedGa(const TArray<FString>& Strings, UWorld* World)
{
	UE_LOG(LogTemp, Error, TEXT("仅仅测试使用的！"));
}

FAutoConsoleCommandWithWorldAndArgs AbilitySystemCheatExtTestCmd(
	TEXT("asc.test"),
	TEXT("打印玩家已经被赋有的技能"),
	FConsoleCommandWithWorldAndArgsDelegate::CreateStatic(ShowGivedGa)
);
```

ue通过 `FAutoConsoleCommandWithWorldAndArgs` 的构造，对 `IConsoleManager` 执行了 `RegisterConsoleCommand` !

按照这套方式，插件在蓝图编译完成后，进行反射，将蓝图中声明的函数注册到 `IConsoleManager` 。

```cpp
FString FixName = BlueprintClass->GetName().Replace(TEXT("BP_"), TEXT(""));
FString CommandName = FString::Printf(TEXT("%s.%s"), *FixName, *Graph->GetName());
FConsoleCommandWithWorldAndArgsDelegate Command = FConsoleCommandWithWorldAndArgsDelegate::CreateUFunction(this, GraphName);
IConsoleCommand* ConsoleCommand = ConsoleManager.RegisterConsoleCommand(*CommandName, *(node->MetaData.ToolTip.ToString()), Command, ECVF_Default);
ConsoleCommands.Add(ConsoleCommand);
```

## 高级扩展

### 透传参数

目前插件是基于 `FAutoConsoleCommandWithWorldAndArgs` 注册的指令，而 `UWorld` 暂时不支持在蓝图上开放，所以暂时只用透传 `TArray<FString>` 的函数！

> 后续有需求再加吧！

![png](/images/computer/game/ue/tools/cheat_ext_adv_config.jpg)

### 测试例子

![png](/images/computer/game/ue/tools/cheat_ext_adv_demo.jpg)

> ASC.TestFunctionWithArgs string1 string2
