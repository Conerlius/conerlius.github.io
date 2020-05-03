---
layout:     post
title:      "GitLab CI在windows下的部署"
subtitle:   " \"GitLab CI在windows下的部署\""
date:       2019-08-05 17:26:00
author:     "Conerlius"
category: 其他
keywords: CI
tags:
    - Tool
---
* content
{:toc}

## 准备
> 该文章主要是以unity为例，unity项目需要在打包机上准备一下几个工具
> * GitBash
> * GitLab_runner
> * Gradle
> * JDK

> 如果是其他项目，只需要保存基本的几个工具就可以
> * Gitbash
> * GitLab_runner

> 具体的安装方式就不说了，相信需要看的奆奆都是会的

## 环境
> JAVA_HOME
>> android打包需要的环境，具体安装可以自行google
>
> ANDROID_HOME
>> unity在导出和打包的时候需要的sdk工具
>
> GRADLE_HOME
>> gradle打包是使用的工具

> 如果不是unity的打包，可以根据需要的情况安装工具

> 注1

## gitlab-runner
> 从官网上download gitlab-runner.exe,按照说明修改为`gitlab-runner`<br>
> 使用`powershell`进入`gitlab-runner的目录`
### gitlab-runner 安装
`./gitlab-runner install`
### gitlab-runner 卸载
`./gitlab-runner uninstall`
### gitlab-runner 注册
`./gitlab-runner register`
> ```
> Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com )
> https://gitlab.com(在工程的runner展示界面上有地址)
> ```
> ```
> Please enter the gitlab-ci token for this runner
> xxx
> ```
> ```
> Please enter the gitlab-ci description for this runner
> android
> ```
> ```
> Please enter the gitlab-ci tags for this runner (comma separated):
> android
> ```
> ```
> Please enter the executor: ssh, docker+machine, docker-ssh+machine, kubernetes, docker, parallels, virtualbox, docker-ssh, shell:
> shell
> ```
> 注2

## Shell在gitlab-runner的配置
> 配置文件一般生成在gitlab-runner的同一个目录下`config.toml`<br>
> 这里就只给出一些比较特殊的，其他的一般直接让gitlab-runner生成就可以了
### shell
> shell标签可以指定一些外部的执行工具，比如windows下的powershell.eg:
> ```
> [[runners]]
>  shell="powershell"
> ```
### clone_url
> 如果你的runner在执行clone的时候，发现connect不上，确定是端口问题，那么需要用到这个标签
> ```
> [[runners]]
>   clone_url = "http://XXXX:8888"
> ```
## Q&A.
### 注1
> 如果未配置好环境，会导致在打包或者使用工具的情况下找不到工具类
### 注2
> 个人比较懒，windows直接使用shell可以少配置不少东西,如果使用docker,可以自行去官网看看docker的相关配置