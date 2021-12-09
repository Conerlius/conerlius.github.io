---
layout:     article
title:      "unreal的UGameInstance的Tick执行问题"
subtitle:   " \"UGameInstacne\""
date:       2021-12-06 11:25:00
author:     "Conerlius"
category: unreal
keywords: unreal
tags:
    - unreal
---

## 大纲

- 背景
- 解决方案
  - 方案1
  - 方案2
  

## 背景

在unreal中，`UGameInstance`通常会有两个，一个是在`Editor World`中使用的，一个是`PIE`中使用的！

这就导致同一个`GameInstance`类在运行的时候，有两个实例在跑，如果配合`Tick`的时候，那么`EditorWorld`容易因为一些运行时才初始化的对象而造成闪退！

基于上述原因，需要对`Tick`的执行进行一定的处理，使得，`Tick`能够分情况执行！

## 解决方案

先看看`Tick`的执行逻辑！(Tickable.cpp)

```c++
for (const FTickableObjectEntry& TickableEntry : Statics.TickableObjects)
{
	if (FTickableGameObject* TickableObject = static_cast<FTickableGameObject*>(TickableEntry.TickableObject))
	{
		// If it is tickable and in this world
		if (((TickableEntry.TickType == ETickableTickType::Always) || TickableObject->IsTickable()) && (TickableObject->GetTickableGameObjectWorld() == World))
		{
			const bool bIsGameWorld = InTickType == LEVELTICK_All || (World && World->IsGameWorld());
			// If we are in editor and it is editor tickable, always tick
			// If this is a game world then tick if we are not doing a time only (paused) update and we are not paused or the object is tickable when paused
			if ((GIsEditor && TickableObject->IsTickableInEditor()) ||
				(bIsGameWorld && ((!bIsPaused && TickType != LEVELTICK_TimeOnly) || (bIsPaused && TickableObject->IsTickableWhenPaused()))))
			{
				FScopeCycleCounter Context(TickableObject->GetStatId());
				TickableObject->Tick(DeltaSeconds);
				// In case it was removed during tick
				if (TickableEntry.TickableObject == nullptr)
				{
					bNeedsCleanup = true;
				}
			}
		}
	}
	else
	{
		bNeedsCleanup = true;
	}
}
```

在上面的源码中，不难看到有几种选择！

先把基本的实现放出来！

```c++
UCLASS(BlueprintType, Blueprintable)
class UNREALPROJECT_API UMyGameInstance : public UGameInstance, public FTickableGameObject
{
	GENERATED_BODY()
public:
    virtual TStatId GetStatId() const override{return Super::GetStatID();}
    virtual void Tick(float DeltaTime) override;
    // 后面修改的都是这里分线以下的！
	virtual bool IsTickable() const override
	{
		return true;
	}
}
```

### 方案1

```c++
virtual bool IsTickable() const override
{
	if (!GetWorld())
		return false;
	return GetWorld()->HasBegunPlay();
}
```

这里值得注意的是，不能直接使用`GWorld`；在判定`World`是否已经启动之前，要对`GetWorld`进行判空处理，在非运行时，`GetWorld`是空的！

### 方案2

```c++
virtual UWorld* GetTickableGameObjectWorld() const override
{
	return GetOuter()->GetWorld();
}
```

重载`GetTickableGameObjectWorld`，对`Actor`所在`World`和现在使用的World是否一直判定，`PIE`在启动的时候，会通过`Context`构建一个运行时使用的World;


## 2021-12-09斧正

经过一轮时间的测试，发现上述的并不是真的不Tick了，而是因为在我实际使用的工程中不在报错而已！那么如何才能做到真的停掉`Tick`呢？

因为

```c++
if (((TickableEntry.TickType == ETickableTickType::Always) || TickableObject->IsTickable()) && (TickableObject->GetTickableGameObjectWorld() == World))
```

在`TickableObject->IsTickable()`恒定为`false`的情况下，`(TickableObject->GetTickableGameObjectWorld() == World)`是默认的（大多情况下是`true`），那么整个`Tick`就是由`TickableEntry.TickType`决定这个`tick`是否执行后续的逻辑了！

```c++
virtual void Init() override;
virtual void Tick(float DeltaTime) override;
virtual bool IsTickable() const override
{
	return  false;
}
virtual ETickableTickType GetTickableTickType() const override { return TickCondition; }
virtual TStatId GetStatId() const override{return Super::GetStatID();}
ETickableTickType TickCondition;
```

因为`Init`函数只有在Play的时候才执行，那么只要在`Init`里将`TickCondition`设置为`TickCondition = ETickableTickType::Always;`就可以了！