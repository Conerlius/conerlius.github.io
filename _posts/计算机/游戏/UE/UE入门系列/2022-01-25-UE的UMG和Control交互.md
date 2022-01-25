---
layout:     article
title:      "UE的UMG和Control交互"
subtitle:   " \"unreal\""
date:       2022-01-25 12:25:00
author:     "Conerlius"
category: unreal
keywords: unreal
tags:
    - unreal
---

感觉说明一下几个类的结构，大家就知道怎么处理了！

## delegate

`delegate`的方式相信很多人都能想得到吧？但是这种方式不是这次要讲的！

## 直调

UI和其他几个类型的交互离不开这几个类型！

- UUserWidget
- APlayerController
- AHUD

### UUserWidget

前面的文章 [UE的UMG简单使用]({% post_url 2022-01-24-UE的UMG简单使用 %}) 中已经说明了UMG绑定相关的了！

在这里再补充一点，HUD是和玩家绑定的！

我们看看`UUserWidget`中的一些变量

####  GetOwningPlayer

```c++
	/**
	 * Gets the local player associated with this UI.
	 * @return The owning local player.
	 */
	virtual ULocalPlayer* GetOwningLocalPlayer() const override;
	
	/**
	 * Gets the local player associated with this UI cast to the template type.
	 * @return The owning local player. May be NULL if the cast fails.
	 */
	template < class T >
	T* GetOwningLocalPlayer() const
	{
		return Cast<T>(GetOwningLocalPlayer());
	}

	/**
	 * Sets the player associated with this UI via LocalPlayer reference.
	 * @param LocalPlayer The local player you want to be the conceptual owner of this UI.
	 */
	void SetOwningLocalPlayer(ULocalPlayer* LocalPlayer);

	/**
	 * Gets the player controller associated with this UI.
	 * @return The player controller that owns the UI.
	 */
	virtual APlayerController* GetOwningPlayer() const override;
	
	/**
	 * Gets the player controller associated with this UI cast to the template type.
	 * @return The player controller that owns the UI. May be NULL if the cast fails.
	 */
	template < class T >
	T* GetOwningPlayer() const
	{
		return Cast<T>(GetOwningPlayer());
	}
```

#### GetOwningPlayerPawn

```c++
	/**
	 * Gets the player pawn associated with this UI.
	 * @return Gets the owning player pawn that's owned by the player controller assigned to this widget.
	 */
	UFUNCTION(BlueprintCallable, BlueprintCosmetic, Category="Player")
	class APawn* GetOwningPlayerPawn() const;
	
	/**
	 * Gets the player pawn associated with this UI cast to the template type.
	 * @return Gets the owning player pawn that's owned by the player controller assigned to this widget.
	 * May be NULL if the cast fails.
	 */
	template < class T >
	T* GetOwningPlayerPawn() const
	{
		return Cast<T>(GetOwningPlayerPawn());
	}
```

#### GetOwningPlayerState && GetOwningPlayerCameraManager

```c++
	/**
	 * Get the owning player's PlayerState.
	 *
	 * @return const APlayerState*
	 */
	template <class TPlayerState = APlayerState>
	TPlayerState* GetOwningPlayerState(bool bChecked = false) const
	{
		if (auto Controller = GetOwningPlayer())
		{
			return !bChecked ? Cast<TPlayerState>(Controller->PlayerState) :
			                   CastChecked<TPlayerState>(Controller->PlayerState, ECastCheckedType::NullAllowed);
		}

		return nullptr;
	}

	/**
	 * Gets the player camera manager associated with this UI.
	 * @return Gets the owning player camera manager that's owned by the player controller assigned to this widget.
	 */
	UFUNCTION(BlueprintCallable, BlueprintCosmetic, Category = "Player")
	class APlayerCameraManager* GetOwningPlayerCameraManager() const;

	/**
	 * Gets the player camera manager associated with this UI cast to the template type.
	 * @return Gets the owning player camera manager that's owned by the player controller assigned to this widget.
	 * May be NULL if the cast fails.
	 */
	template <class T>
	T* GetOwningPlayerCameraManager() const
	{
		return Cast<T>(GetOwningPlayerCameraManager());
	}
```

通过上面的几段代码，我们大致知道通过当前的UMG，我们能获取到和当前界面绑定的`PlayerController`、`PlayerState`、`Cameramanager`!

在看看`PlayerController`(继承了`AController`)

### APlayerController

```c++
	/** Heads up display associated with this PlayerController. */
	UPROPERTY()
	TObjectPtr<AHUD> MyHUD;
	/** Camera manager associated with this Player Controller. */
	UPROPERTY(BlueprintReadOnly, Category=PlayerController)
	TObjectPtr<APlayerCameraManager> PlayerCameraManager;

    // 下面是继承自AController
	/**
	 * @return this controller's PlayerState cast to the template type, or NULL if there is not one.
	 * May return null if the cast fails.
	 */
	template < class T >
	T* GetPlayerState() const
	{
		return Cast<T>(PlayerState);
	}
	/** Getter for Pawn */
	FORCEINLINE APawn* GetPawn() const { return Pawn; }

	/** Templated version of GetPawn, will return nullptr if cast fails */
	template<class T>
	T* GetPawn() const
	{
		return Cast<T>(Pawn);
	}
	/** Getter for Character */
	FORCEINLINE ACharacter* GetCharacter() const { return Character; }
```

在这里我们看到`APlayerController`能获取到`HUD`和`Pawn`、`ACharacter`、`PlayerState`、`APlayerCameraManager`!

在看看`HUD`

#### HUD

```c++
	/** Returns the PlayerController for this HUD's player.	 */
	UFUNCTION(BlueprintCallable, Category=HUD)
	APlayerController* GetOwningPlayerController() const;

	/** Returns the Pawn for this HUD's player.	 */
	UFUNCTION(BlueprintCallable, Category=HUD)
	APawn* GetOwningPawn() const;
```

#### 最后

从上面的部分源码中我们能看到`HUD`、`PlayerController`、`Pawn`和`UUserwidget`的关联！

而且因为`UMG`一般是在`HUD`进行创建的~（你也可以在其他地方创建，但是不大建议而已）。所以如果我们在自己创建的`UUserwidget`中有`Input`的回调的时候，简单粗暴的方法就是通过`GetOwningPlayer`后在进行其他的交互！