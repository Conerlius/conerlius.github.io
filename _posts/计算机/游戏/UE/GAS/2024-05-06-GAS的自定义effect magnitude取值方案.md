---
layout:     article
title:      "GAS的自定义effect magnitude取值方案"
subtitle:   " \"gas\""
date:       2024-05-06 11:25:00
author:     "Conerlius"
category: unreal
keywords: unreal
tags:
    - unreal
---

## 背景交代

产品策划想要使用独立的配置表去控制effect的数值，在不同的关卡中使用同一个effect的时候，能跟进不同的关卡effect产生不一样的数值影响；

### 简单的测试例子

在一个Level内，两个区域内，这两个区域有叠加的部分，两个区域的effect都是对其中一个属性进行减法，但是减去的值不一样，希望是站在重叠的区域内的时候，这两个区域的effect里，对某条属性减去最大的数字的保留生效，其他的不不能生效，但是在退出最大值的区域回到单个区域的时候，希望该区域的effect能全部生效；

## 方案

### 同一个Effect产生不一样的数值效果的问题

GAS里是有两种比较简单的方式可以做到这个事情，一个是`SetByCaller`,一个是`Custom Calculation Class`

![png](/images/computer/game/ue/gas/gas_magnitude_calculate_type.jpg)

和项目探讨过，`SetByCaller` 满足不了需求；那就只有 `Custom Calculation Class` 了；

`Custom Calculation Class` 需要的调用使用的 `GetDefaultObject` 来执行的，所以需要在 `Effect Context` 中记录Effect需要引用的查找配置的id，由项目程序的调用发起者指定id；这样就能满足项目的需求了。

```cpp
class FDemoGameplayEffectContext;

float UDemoModMagnitude::CalculateBaseMagnitude_Implementation(const FGameplayEffectSpec& Spec) const
{
    FDemoGameplayEffectContext* Context = static_cast<FDemoGameplayEffectContext*>(Spec.GetContext().Get());
    FString Va = Context->Get(0);
    FString NunString = Va.Right(1);
    int num = FCString::Atoi(*NunString);
    Context->PrintDatas();
    return 100*num;
}
```

以上就是一个从context中根据透传的string的最后一个数值放大100后生效！

### Stack导致的Context覆盖问题

上述的代码逻辑中， `Context` 明显很重要的，所以针对 `Context` 的处理要小心；

#### `Context` 自定义

`EffectContext` 并不能通过 `ini` 来指定，必须要重载 `UAbilitySystemGlobals::AllocGameplayEffectContext()` 以实现构建自定义的；

`UAbilitySystemGlobals` 可以通过 `ini` 来指定！

```ini
[/Script/GameplayAbilities.AbilitySystemGlobals]
AbilitySystemGlobalsClassName="/Script/Test_gas.DemoAbilitySystemGlobals"
```

代码：

```cpp
FGameplayEffectContext* UDemoAbilitySystemGlobals::AllocGameplayEffectContext() const
{
    return new FDemoGameplayEffectContext();
}
```

GAS提供的Stack有三种方式， `Aggregate` 聚合相关的就不说明了，自己看文档；

![png](/images/computer/game/ue/gas/gas_effect_stack.jpg)

但是 `Aggregate` 在使用的时候，会导致 `Spec` 覆盖，后面应用的会覆盖前面的，然后通过 `Stack count` 对当前的 `Magnitude` 值乘以count，所以如果要想保留上一个的Context，那么就不能使用 `Aggregate` ，需要使用 `None` ；

### 保留指定的最大/最小的GameplayModifier

由于想尽可能不改动源码，但最后发现有很多函数都是private的，不改不行！

简单的原理说明（eg: 最大）：

在 `ApplyGameplayEffectSpecToSelf` 的时候，遍历所有的effect，非最大值的将已经计算好的 `FModifierSpec` 重新赋值成 $0$ ，使得仅剩最大值的 `Effect` 生效！

在 `RemoveActiveGameplayEffect` 时，先通知指定类型的 `Effect` 重新计算 `FModifierSpec` ，然后再取最大值；

> 代码放置在最后！

## 结果

![gif](/images/computer/game/ue/gas/gas_effect_demo.gif)

## 代码

### 源码改动

在 `FActiveGameplayEffectsContainer` 中添加以下两个方法即可！

```cpp
    void UpdateAllAggregatorModMagnitudesDemo(FActiveGameplayEffect& ActiveEffect){ UpdateAllAggregatorModMagnitudes(ActiveEffect); }

    void UpdateAggregatorModMagnitudesDemo(const TSet<FGameplayAttribute>& AttributesToUpdate, FActiveGameplayEffect& ActiveEffect) { UpdateAggregatorModMagnitudes(AttributesToUpdate, ActiveEffect); }
```

代码仅是示例，具体使用可以自行考量

```cpp
FActiveGameplayEffectHandle UDemoAbilitySystemComponent::ApplyGameplayEffectSpecToSelf(
	const FGameplayEffectSpec& GameplayEffect, FPredictionKey PredictionKey)
{
	FGameplayEffectQuery FindEffectClassQuery;
	FindEffectClassQuery.EffectDefinition = GameplayEffect.Def.GetClass();
	TArray<FActiveGameplayEffectHandle> Array = ActiveGameplayEffects.GetActiveEffects(FindEffectClassQuery);
	for (FActiveGameplayEffectHandle& ActiveEffectHandle : Array)
	{
		FActiveGameplayEffect* Effect = ActiveGameplayEffects.GetActiveGameplayEffect(ActiveEffectHandle);
		Effect->Spec.CalculateModifierMagnitudes();
	}
	// reset pre Effect
	FActiveGameplayEffectHandle ActiveGameplayEffectHandle = Super::ApplyGameplayEffectSpecToSelf(GameplayEffect, PredictionKey);
	if(!ActiveGameplayEffectHandle.WasSuccessfullyApplied())
		return ActiveGameplayEffectHandle;
		
	Array = ActiveGameplayEffects.GetActiveEffects(FindEffectClassQuery);
	int TotalAttribute = GameplayEffect.Def->Modifiers.Num();
	TArray<float> AllMaxValue = TArray<float>();
	TArray<FActiveGameplayEffect*> AllMaxEffect = TArray<FActiveGameplayEffect*>();
	AllMaxValue.AddZeroed(TotalAttribute);
	AllMaxEffect.AddZeroed(TotalAttribute);
	TSet<FGameplayAttribute> set;
	for (FActiveGameplayEffectHandle& ActiveEffectHandle : Array)
	{
		FActiveGameplayEffect* Effect =ActiveGameplayEffects.GetActiveGameplayEffect(ActiveEffectHandle);
		int index=0;
		for (FGameplayModifierInfo GameplayModifierInfo : Effect->Spec.Def->Modifiers)
		{
			float magnitude = Effect->Spec.Modifiers[index].GetEvaluatedMagnitude();
			if(!set.Contains(GameplayModifierInfo.Attribute))
				set.Emplace(GameplayModifierInfo.Attribute);
			
			if (AllMaxValue[index] <= magnitude)
			{
				AllMaxValue[index] = magnitude;
				Effect->Spec.Modifiers[index] = FModifierSpec();
				AllMaxEffect[index] = Effect;
			}
			else
			{
				Effect->Spec.Modifiers[index] = FModifierSpec();
			}
			index++;
		}
		ActiveGameplayEffects.UpdateAggregatorModMagnitudesDemo(set,  *Effect);
	}
	// revert the max
	for (int index = 0; index < set.Num(); index++)
	{
		AllMaxEffect[index]->Spec.Modifiers[index] = FModifierSpec(AllMaxValue[index]);
		ActiveGameplayEffects.UpdateAggregatorModMagnitudesDemo(set,  *AllMaxEffect[index]);
	}
	
	return ActiveGameplayEffectHandle;
}

bool UDemoAbilitySystemComponent::RemoveActiveGameplayEffect(FActiveGameplayEffectHandle Handle, int32 StacksToRemove)
{
	FActiveGameplayEffect* RemoveEffect =ActiveGameplayEffects.GetActiveGameplayEffect(Handle);
	UClass* EffectClass = RemoveEffect->Spec.Def.GetClass();
	bool isSuccess = Super::RemoveActiveGameplayEffect(Handle, StacksToRemove);
	if (!isSuccess)
	{
		return isSuccess;
	}
	FGameplayEffectQuery FindEffectClassQuery;
	FindEffectClassQuery.EffectDefinition = EffectClass;	
	TArray<FActiveGameplayEffectHandle> Array = ActiveGameplayEffects.GetActiveEffects(FindEffectClassQuery);
	int MaxValue=-1.f;
	int TotalAttribute = RemoveEffect->Spec.Def->Modifiers.Num();
	TArray<float> AllMaxValue = TArray<float>();
	TArray<FActiveGameplayEffect*> AllMaxEffect = TArray<FActiveGameplayEffect*>();
	AllMaxValue.AddDefaulted(TotalAttribute);
	AllMaxEffect.AddDefaulted(TotalAttribute);
	TSet<FGameplayAttribute> set;
	
	for (FActiveGameplayEffectHandle& ActiveEffectHandle : Array)
	{
		FActiveGameplayEffect* Effect = ActiveGameplayEffects.GetActiveGameplayEffect(ActiveEffectHandle);
		Effect->Spec.CalculateModifierMagnitudes();
		int index=0;
		for (FGameplayModifierInfo GameplayModifierInfo : Effect->Spec.Def->Modifiers)
		{
			float magnitude = Effect->Spec.Modifiers[index].GetEvaluatedMagnitude();
			if(!set.Contains(GameplayModifierInfo.Attribute))
				set.Emplace(GameplayModifierInfo.Attribute);
			if (AllMaxValue[index] <= magnitude)
			{
				AllMaxValue[index] = magnitude;
				Effect->Spec.Modifiers[index] = FModifierSpec();
				AllMaxEffect[index] = Effect;
			}
			else
			{
				Effect->Spec.Modifiers[index] = FModifierSpec();
			}
			index++;
		}
		ActiveGameplayEffects.UpdateAggregatorModMagnitudesDemo(set,  *Effect);
	}
	// revert the max
	for (int index = 0; index < set.Num(); index++)
	{
		AllMaxEffect[index]->Spec.Modifiers[index] = FModifierSpec(AllMaxValue[index]);
		ActiveGameplayEffects.UpdateAggregatorModMagnitudesDemo(set,  *AllMaxEffect[index]);
	}
	return isSuccess;
}

```