---
layout:     article
title:      "chrome降低cpu消耗"
subtitle:   "software reporter tool"
date:       2021-11-03 11:25:00
author:     "Conerlius"
category: 其他
keywords: chrome
tags:
    - 其他
---

## 大纲

- 插件消耗
- software_reporter_tool

目前遇到的`chrome`的消耗问题，通常都可以归类到这两类（`插件消耗`和`software_reporter_tool`）！

## 插件消耗

想要确认是不是插件的消耗，可以借助`chrome`内置的`任务管理器`(`Shift`+`Esc`打开)

![png](/images/other/chrome_cpu1.png)

找到对应的插件，将之kill或者卸载就可以了！

## software_reporter_tool

![png](/images/other/chrome_cpu2.png)

网上有两个比较常见的方法：

- 简单点的

在浏览器的地址栏中输入
> `chrome://flags/ `

找到`Chrome Cleanup Tool in safety check`并disable重启就可以了！
![png](/images/other/chrome_cpu3.png)

- 如果上述的并没有成功，这个一定是可以的！


我们还可以通过`win`+`r`键打开运行命令窗口并输入以下命令快速找到它

> `%localappdata%\Google\Chrome\User Data\SwReporter`

![png](/images/other/chrome_cpu4.png)

![png](/images/other/chrome_cpu5.png)

进入改目录，找到`software_reporter_tool.exe`右键进入属性,在属性中选择`安全`

![png](/images/other/chrome_cpu6.png)

选中`System`的情况下，选择`高级`,然后`禁用继承`

![png](/images/other/chrome_cpu7.png)

![png](/images/other/chrome_cpu8.png)

完成了上述的步骤后，手动在`Windows`的任务管理器中找到`software reporter tool`,并将之kill了就可以了！之后使用`chrome`的时候就消耗很低了！

> Software Reporter Tool是一个Chrome清理工具,用于清理谷歌浏览器中不必要或恶意的扩展，应用程序，劫持开始页面等等。当你安装Chrome时，Software_reporter_tool.exe也j就会被下载在SwReporter文件夹下的Chrome应用数据文件夹中。
> 
> oftware_reporter_tool.exe是谷歌浏览器的一个用于清理恶意的扩展、应用程序、劫持页面等的一个工具，在安装并使用谷歌浏览器的时候，Software_reporter_tool.exe也会自动下载在安装文件当中。
> 
> 这个软件在运行的过程中可能会长时间地占用CPU，导致高CPU使用率，我们可以通过任务管理器手动结束进程或者选择删除SRT，但这并不完全解决这个问题，过一段时间它又会再次运行，在浏览器自动更新的时候就又会重新被下载下来。
