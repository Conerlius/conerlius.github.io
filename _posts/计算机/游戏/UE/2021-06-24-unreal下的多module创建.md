---
layout:     article
title:      "unreal下的多module创建"
subtitle:   " \"多module\""
date:       2021-06-25 12:25:00
author:     "Conerlius"
category: unreal
keywords: unreal
tags:
    - unreal
---

在默认的情况下，unreal中创建的plugin还是项目中,基本都是只有一个module，但是有时候我们不可能吧什么都放在一个module中！所以就有了对多module的需求


在已有的plugin中，
XXX.uplugin文件添加
```
"Modules": [
	...
		{
			"Name": "名称",
			"Type": "Editor",
			"LoadingPhase": "Default"
		}
	]
```
如果是项目工程的话，就在`XXXX.uproject`中添加以上形式的代码。

- 创建build.cs

在plugin或者在项目工程的`Source`目录下，创建和module同名的目录（我这里使用的是MyEditorTools）！

![](/images/computer/game/ue/multi_modules/1.png)

> plugin类似！

在该目录下创建`{module名称}.Build.cs`(MyEditorTools.Build.cs)

内容基本如下，可以自行考虑修改
```c#
public class MyEditorTools : ModuleRules
{
	public MyEditorTools(ReadOnlyTargetRules Target) : base(Target)
	{
		PCHUsage = PCHUsageMode.UseExplicitOrSharedPCHs;

		PublicDependencyModuleNames.AddRange(new string[] { "Core", "CoreUObject", "Engine", "InputCore" });

		PrivateDependencyModuleNames.AddRange(new string[] {  });
		PublicIncludePaths.Add(ModuleDirectory);

		// Uncomment if you are using Slate UI
		// PrivateDependencyModuleNames.AddRange(new string[] { "Slate", "SlateCore" });
		
		// Uncomment if you are using online features
		// PrivateDependencyModuleNames.Add("OnlineSubsystem");

		// To include OnlineSubsystemSteam, add it to the plugins section in your uproject file with the Enabled attribute set to true
	}
}
```

- 创建Module的类文件

在`Source/{module名称}`里创建两个文件

- {module名称}.h
- {module名称}.cpp

头文件简单如下

```c++
class  FMyEditorTools : public IModuleInterface
{
public:
	virtual void StartupModule() override;
	virtual void ShutdownModule() override;
};
```

> 自行更换名称即可


实现的cpp里

```c++
#define LOCTEXT_NAMESPACE "FMyEditorTools"

void FMyEditorTools::StartupModule()
{
	IModuleInterface::StartupModule();
}

void FMyEditorTools::ShutdownModule()
{
	IModuleInterface::ShutdownModule();
}
#undef LOCTEXT_NAMESPACE

IMPLEMENT_MODULE(FMyEditorTools, MyEditorTools)
```

有时候有些人（我自己），为了方便，基本都是从现有的模块中拷贝这几个文件的，然后发现项目工程的Module类文件中只有一句
```c++
IMPLEMENT_PRIMARY_GAME_MODULE( FDefaultGameModuleImpl, UnrealGameDemo, "UnrealGameDemo" );
```
这是ue已经提供的一套入口模板，使用的注册方法不同的，需要注意一下！