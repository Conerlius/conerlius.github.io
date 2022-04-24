---
layout:     article
title:      "UE使用cpp代码打开EditorUtilityWidget"
subtitle:   " \"unreal\""
date:       2022-04-24 11:25:00
author:     "Conerlius"
category: unreal
keywords: unreal
tags:
    - unreal
---

平时写ue小工具的基本都是使用`EditorUtilityWidgetBlueprint`；
如果工具稍微一多，直接通过`ContentBrowser`的话，不是很方便！

![png](/images/computer/game/ue/cpp_editorultilitywidget/1.png)

所以我比较喜欢在`Menu`中

```c++
void FMyEditorModule::OpenFrameworkConfig()
{
    // 因为我是放置在plugin中的，所以我这里不是/Game
	FString assetPath = TEXT("EditorUtilityWidgetBlueprint'/MyPlugin/Editor/BP_FrameworkConfig.BP_FrameworkConfig'");
	auto asset = UEditorAssetLibrary::LoadAsset(assetPath);
	UEditorUtilityWidgetBlueprint* EditorWidget = Cast<UEditorUtilityWidgetBlueprint>(asset);
	UEditorUtilitySubsystem* EditorUtilitySubsystem = GEditor->GetEditorSubsystem<UEditorUtilitySubsystem>();
	EditorUtilitySubsystem->SpawnAndRegisterTab(EditorWidget);
}
```

`assetPath`中，测试过在直接使用`/MyPlugin/Editor/BP_FrameworkConfig.BP_FrameworkConfig`加载也是没有问题的