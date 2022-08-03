---
layout:     article
title:      "移除Addressable catalog中的hash"
subtitle:   " \"unity\""
date:       2022-03-28 15:00:00
author:     "Conerlius"
category: unity
tags:
    - unity
    - addressable
---

## 背景说明

起因如下：
项目中使用了`lua`，而`lua`文件我们在开发的过程中肯定不会是直接使用`encode`后的文件，而真的打包发布的是时候我们也不可能直接将`lua`的明文发布出去的，所以在使用`addressable`打包的时候，我们会对工程内的`Lua`进行`Encode`后在`Package`