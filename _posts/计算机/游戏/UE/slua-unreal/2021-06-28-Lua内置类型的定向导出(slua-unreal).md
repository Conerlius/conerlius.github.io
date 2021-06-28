---
layout:     article
title:      "Lua内置类型的定向导出"
subtitle:   " \"slua\""
date:       2021-06-28 18:25:00
author:     "Conerlius"
category: unreal
keywords: unreal
tags:
    - unreal
    - slua
---

## 大纲

- CppBinding没有标志UFunction的导出
- CppBinding的限制

## CppBinding没有标志UFunction的导出

`UObject`的`UFunction`都会通过反射的方法自动导出给`lua`，但有的时候，某些`UObject`的方法并没有标记为`UFunction`，但是我们仍然希望导出给`lua`使用，例如：

```c++
UCLASS(Abstract, editinlinenew, BlueprintType, Blueprintable, meta=( DontUseGenericSpawnObject="True", DisableNativeTick) )
class UMG_API UUserWidget : public UWidget, public INamedSlotInterface
{
	GENERATED_BODY()

	friend class SObjectWidget;
public:
	UUserWidget(const FObjectInitializer& ObjectInitializer);
	...
		/** @returns The root UObject widget wrapper */
	UWidget* GetRootWidget() const;

	/** @returns The slate widget corresponding to a given name */
	TSharedPtr<SWidget> GetSlateWidgetFromName(const FName& Name) const;

	/** @returns The uobject widget corresponding to a given name */
	UWidget* GetWidgetFromName(const FName& Name) const;

	//~ Begin UObject Interface
	virtual bool IsAsset() const;
	virtual void PreSave(const class ITargetPlatform* TargetPlatform) override;
```

可以看到`UUserWidget`的`GetSlateWidgetFromName`方法等都没有标记为`UFunction`，这样的函数在反射过程中是无法找到的，但是我们很可能在lua编程中需要使用它们，这个时候，我们可以增加扩展方法，用于扩展`uobject`的方法，例如：

```c++
// 给UObjectWidget增加名为FindWidget函数，使用GetWidgetFromName的实现
REG_EXTENSION_METHOD(UUserWidget,"FindWidget",&UUserWidget::GetWidgetFromName);

// 给UObjectWidget增加名为RemoveWidget函数，自己实现
REG_EXTENSION_METHOD_IMP(UUserWidget,"RemoveWidget",{
                CheckUD(UUserWidget,L,1);
                auto widget = LuaObject::checkUD<UWidget>(L,2);
                bool ret = UD->WidgetTree->RemoveWidget(widget);
                return LuaObject::push(L,ret);
            });

// 给UObjectWidget增加名为SpawnActor函数，因为存在重载版本，我明确给出使用哪个重载版本的函数签名
REG_EXTENSION_METHOD_WITHTYPE(UWorld,"SpawnActor",&UWorld::SpawnActor,AActor* (UWorld::*)( UClass*, FVector const*,FRotator const*, const FActorSpawnParameters&));

```

`slua`提供了三种方式:

- REG_EXTENSION_METHOD

给UObjectWidget增加名为FindWidget函数，使用GetWidgetFromName的实现

- REG_EXTENSION_METHOD_IMP

给UObjectWidget增加名为RemoveWidget函数，自己实现

- REG_EXTENSION_METHOD_WITHTYPE

给UObjectWidget增加名为SpawnActor函数，因为存在重载版本，我明确给出使用哪个重载版本的函数签名

## CppBinding的限制

目前`cppbinding`导出的`c++`类暂时不支持`down cast`操作，即从父类cast到子类，官方的说法是未来版本会增加这个支持。总之写api的时候要小心就对了!