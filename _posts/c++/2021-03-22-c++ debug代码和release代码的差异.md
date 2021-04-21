---
layout:     article
title:      "c++ debug代码和release代码的差异"
subtitle:   " \"编译\""
date:       2021-03-22 18:00:00
author:     "Conerlius"
category: c++
keywords: c++
tags:
    - c++
---

## 大纲

- `exe`的`debug`和`release`怎么生成的？
- `debug`和`release`在汇编结构上的差异
- `debug`生成的对象怎么直接转换成`release`

## `exe`的`debug`和`release`怎么生成的？

`gas`是`gcc`的汇编器

x86的指令eg
```
mov eax 32
```

`GAS`的指令eg
```
movl $32 %eax
```

[<Programming From The Ground Up>](https://download-mirror.savannah.gnu.org/releases/pgubook/ProgrammingGroundUp-1-0-booksize.pdf)

http://savannah.nongnu.org/

## `debug`和`release`在汇编结构上的差异


## `debug`生成的对象怎么直接转换成`release`