---
layout:     article
title:      "Unity Api Updater的使用"
subtitle:   "Api Updater"
date:       2020-12-08 13:22:00
author:     "Conerlius"
category: unity
keywords: 
    - unity
    - api updater
tags:
    - unity
---

## 纲要

- unity的api更新机制
- 如何接入unity的api更新

## unity的api更新机制

- unity的api更新原理
- unity的api更新流程
  
### unity的api更新原理

- class的迁移
- class内部的迁移

#### class的迁移

在`class`的头申明`MovedFrom`

#### class内部的迁移

识别`Obsolete`,判定`Obsolete`的内容里是否包含`UnityUpgradable`,有就启动`api updater`


### unity的api更新流程

## 如何接入unity的api更新

- 例子
- 代码

### 例子

### 代码

