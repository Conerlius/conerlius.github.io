---
layout:     article
title:      "Mixing AGameState with AGameModeBase is not compatible"
subtitle:   " \"unreal\""
date:       2022-08-03 11:00:00
author:     "Conerlius"
category: unreal
keywords: unreal
tags:
    - unreal
---

Gameplay用起来很巴适，但是有时候就会折腾一下，然后就遇到一些不注意的东西！

```
Mixing AGameState with AGameModeBase is not compatible. Change AGameModeBase subclass (bp_BaseGameMode_C) to derive from AGameMode, or make both derive from Base
```

因为项目中并没有使用`AGameMode`,使用的是`AGameModeBase`，在`AGameModeBase`的代码中

```c++
/** GameState is used to replicate game state relevant properties to all clients. */
	UPROPERTY(Transient)
	TObjectPtr<AGameStateBase> GameState;
```

而`AGameMode`中的是

```c++
void AGameMode::InitGame(const FString& MapName, const FString& Options, FString& ErrorMessage)
{
	Super::InitGame(MapName, Options, ErrorMessage);
	SetMatchState(MatchState::EnteringMap);

	if (GameStateClass == nullptr)
	{
		UE_LOG(LogGameMode, Error, TEXT("GameStateClass is not set, falling back to AGameState."));
		GameStateClass = AGameState::StaticClass();
	}
	else if (!GameStateClass->IsChildOf<AGameState>())
	{
		UE_LOG(LogGameMode, Error, TEXT("Mixing AGameStateBase with AGameMode is not compatible. Change AGameStateBase subclass (%s) to derive from AGameState, or make both derive from Base"), *GameStateClass->GetName());
	}
    ...
}
```

https://ai-gaminglife.hatenablog.com/entry/2018/07/23/115923


![png](/images/computer/game/ue/problems/mixing_gamestate.png)

所以要求是项目的MyGameMode继承的是哪个Mode，就是用对应的GameState!