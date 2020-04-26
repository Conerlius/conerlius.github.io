---
layout:     post
title:      "confluence上写markdown"
subtitle:   " \"physic\""
date:       2020-04-26 11:22:00
author:     "Conerlius"
category: 其他
keywords: markdown
tags:
    - markdown
    - confluence
---

## Confluence
Confluence是目前公司的的集成wiki，但是由于其markdown的宏对真正的markdown的支持很有限，（各种样式都没有）。
## wiki标志
confluence的markdown宏使用不了的情况下，但是为了统一而且好看的样式，源文档就必须是使用markdown写了，但是又如何将markdown放置到wiki上呢；这个时候就需要`wiki标志`,将markdown转换成wiki标志。
## markdown转换成wiki标志
https://zhuyi731.github.io/m2c-webpage/ 这个是工具的link,当然也可以将其clone下来，自己仿照其写另外一个工具 https://github.com/Zhuyi731/m2c-webpage
我自己就仿照写了一个python的[link](https://github.com/Conerlius/ConvertMarkdownToConfluence)

## 注意
- markdown上的图片引用，如果是本地的资源引用，在wiki上还需要手动修正（即上行后引用）(也可以在markdown上直接引用wiki的绝对资源路径)