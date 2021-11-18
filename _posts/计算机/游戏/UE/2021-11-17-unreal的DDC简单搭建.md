---
layout:     article
title:      "unreal的DDC简单搭建"
subtitle:   " \"缓存\""
date:       2021-11-17 11:25:00
author:     "Conerlius"
category: unreal
keywords: unreal
tags:
    - unreal
---


## 大纲

- DDC的简单说明
- DDC简单搭建使用
  - DDC文件夹的挂载
  - Unreal中使用DDC

## DDC的简单说明

这里就直接上link就可以了！https://docs.unrealengine.com/4.27/zh-CN/ProductionPipelines/DerivedDataCache/

## DDC简单搭建使用

在`UnrealProject\Config\DefaultEngine.ini`文件中添加

```config
[DerivedDataBackendGraph]
; Keep files for at least 7 days
MinimumDaysToKeepFile=7
; Configure the root entry. It uses a KeyLength node to hash long strings then passes requests to AsyncPut
Root=(Type=KeyLength, Length=120, Inner=AsyncPut)
; Configure the AsyncPut entry. It uses an AsyncPut node that then passes requests to Hierarchy
AsyncPut=(Type=AsyncPut, Inner=Hierarchy)
; Configure the hierarchy entry. This uses multiple nodes that are used in order until a read is found (writes go to all writable entries)
Hierarchy=(Type=Hierarchical, Inner=Boot, Inner=Pak, Inner=EnginePak, Inner=Local, Inner=Shared)
; Configure the Boot node. This holds 512MB of data used to speed up booting.
Boot=(Type=Boot, Filename="%GAMEDIR%DerivedDataCache/Boot.ddc", MaxCacheSize=512)
; Configure the local node. This is a filesystem node with the following set -
;   Readonly: Can data be written to this layer
;   Clean: Perform a cleanup of old files on launch
;   Flush: Nuke the DDC and start over
;   PurgeTransient: Don't keep transient data in the DDC
;   DeleteUnused: Clean up old files (happens on a background thread)
;   UnusedFileAge: Age after which files are removed
;   FoldersToClean: Max number of folders to clean in a session. -1 = Unlimited
;   MaxFileChecksPerSec: How many files to check a second.
;   Path: path to use for the filesystem DDC
;   EnvPathOverride: An environment variable that if set will be used instead of path. E.g. UE-LocalDataCachePath=d:\DDC. ('None' disables the DDC)
;   CommandLineOverride: A command line argument used in preference to the default / envvar setting. E.g. -SharedDataCachePath=\\someshare\folder
;   EditorOverrideSetting: Editor user setting that overrides the default/envvar/command line values
Local=(Type=FileSystem, ReadOnly=false, Clean=false, Flush=false, PurgeTransient=true, DeleteUnused=true, UnusedFileAge=34, FoldersToClean=-1, Path=%ENGINEDIR%DerivedDataCache, EnvPathOverride=UE-LocalDataCachePath, EditorOverrideSetting=LocalDerivedDataCache)
; Configure the shared DDC that is accessed after local. It's a filesystem DDC and the parameters are explained above
Shared=(Type=FileSystem, ReadOnly=false, Clean=false, Flush=false, DeleteUnused=true, UnusedFileAge=10, FoldersToClean=10, MaxFileChecksPerSec=1, Path=?EpicDDC, EnvPathOverride=UE-SharedDataCachePath, EditorOverrideSetting=SharedDerivedDataCache, CommandLineOverride=SharedDataCachePath)
; Configure an alternate shared DDC that is accessed after local. It's a filesystem DDC and the parameters are explained above
AltShared=(Type=FileSystem, ReadOnly=true, Clean=false, Flush=false, DeleteUnused=true, UnusedFileAge=23, FoldersToClean=10, MaxFileChecksPerSec=1, Path=?EpicDDC2, EnvPathOverride=UE-SharedDataCachePath2)
; Configure a Project Pak node. This is a pre-generated DDC data file for the project that can be distributed to reduce runtime fetches/generation
; See documentation for how to create a DDP via the DerivedDataCache commandlet
Pak=(Type=ReadPak, Filename="%GAMEDIR%DerivedDataCache/DDC.ddp")
; Configure a Project Pak node. This is a pre-generated DDC data file for the engine that can be distributed to reduce runtime fetches/generation
EnginePak=(Type=ReadPak, Filename=%ENGINEDIR%DerivedDataCache/DDC.ddp)
```

### 挂载DDC文件夹

https://support.microsoft.com/en-us/windows/map-a-network-drive-in-windows-10-29ce55d1-34e3-a7e2-4801-131475f9557d

### Unreal中使用DDC

然后使用指令启动项目工程

```cmd
Start %Custom_Compiled_UE4Editor% "%current_path%/UnrealProject/UnrealProject.uproject" -run=DerivedDataCache -fill
```

然后再项目工程中

![ ](/images/computer/game/ue/ddc_config1.png)

选择已经挂载的`ddc`目录，重启unreal就可以了！

![ ](/images/computer/game/ue/ddc_config2.png)