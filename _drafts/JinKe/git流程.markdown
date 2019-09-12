---
layout:     post
title:      "git代码版本管理"
subtitle:   " \"流程\""
date:       2019-09-12 15:45:00
author:     "Conerlius"
header-img: "img/post-bg-2015.jpg"
tags:
    - 交流
---

[TOC]
---
# git代码版本管理

## git的各种分支定义
* 划分主干分支(master)
* 开发分支(dev)
* 开发的feature/xxx分支
* release分支
* Hotfix
* Tag

### 主干分支master
> 经过测试QA验收的内容版本，其中囊括所有的地区版本，渠道版本的内容;
> 也可以不使用该分支,具体可以按项目怎么定义master的含义;

### 开发分支develop
> 该分支要是经过一定测试的内容，其中囊括所有的地区版本，渠道版本的内容;
> 因为功能具有耦合性，所以此分支还是有可能产生`bug`，没有`master`稳定
> 如果需要修复该分支上的bug，可以根据具体项目定义，另起`Hotfix`或`Bug`分支都行，建议`Bug`分支

### 开发的feature
> 具体单一一个功能一个feature分支，命名规则是`feature/功能名称`，eg: `feature/mail`；名称必须是有意义的英文命名；
> 该分支上的功能必须经过测试QA验收通过后才可以Merge到开发develop；
> 需要注意的是，merge到develop的时候，一定只能产生一条commit记录;
> 次分支在merge完成后，***必须删除!***

### release分支
> 预发布分支
> 该分支通常是做为发不版本的一个测试分支，eg:如果需要发布一个腾讯渠道的app，那么就会创建一个release分支，按照PM的要求，把当前里程碑的内容逐一merge到此release分支；
> 之测试QA在次分支上测试发布版本的内容;
> 次分支在正式发布完成后，打上发布版本的`Tag`,***并删除此分支*** ;

### Hotfix
> hotfix是紧急修复的分支，常用于Release分支；如果QA在测试release分支的时候发现***需要*** 修复的bug，则开发需要另起`Hotfix`分支来修复次问题，QA在`Hotfix`分支验收完成后，再merge到`release`分支，如果有需要，还需要merge到`develop`
> 次分支在merge完成后，***必须删除!***

### Tag
> 该标志是用了记录发布版本的，上线后，后续的发布版本都将基于此tag创建`release`分支

## 流程
> git 的代码管理流程有一种广为人知的流程，先来说明一下经典的流程
### 经典流程
> 先post一张每次讲git都会出现的图;
> **以下是盗用网络上的图**
> ![png](git_flow_chart.png)

#### 流程的说明
### 调整后的流程
> 但是上图在大型或者多渠道，多版本的项目的时候，会出现一些问题，所以需要对上图的流程进行一些不大的调整;
#### 流程说明
## 例子说明
