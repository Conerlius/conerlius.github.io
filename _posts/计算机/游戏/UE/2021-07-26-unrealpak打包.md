---
layout:     article
title:      "unrealpak打包"
subtitle:   " \"打包\""
date:       2021-07-26 11:25:00
author:     "Conerlius"
category: unreal
keywords: unreal
tags:
    - unreal
---

`UAT`AutomationTool

pak是通过挂载的形式处理的`Mount`

unrealpak.exe

`-create`构建已经cook后的资源

`-compress`生成已经压缩的资源

`encrpt -encrptindex -aes=XXX` 加密 `如果没有encrptindex就会有报错` `ase`必须是32位的


查看

`-list`可以查看pak里的资源内容

`-list -aes=XXX` 查看已经加密的内容

`-extract=your_path`解压


加载pak

FPakPlatformFile创建及初始化

FPlatformFileManager::Get().SetPlatformFile切换到Pak平台（完成后再切换回去）

将pak中的`Mount Point`切换成相对应的路径