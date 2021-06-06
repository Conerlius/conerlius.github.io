---
layout:     article
title:      "vscode使用cl编译c++"
subtitle:   " \"c++\""
date:       2021-06-04 11:00:00
author:     "Conerlius"
category: c++
keywords: c++
tags:
    - c++
---

## 纲要

- window下的c++编译器
- vscode的cl编译
- - 基本的环境
- - 全局环境的配置
- vscode的`gcc/g++`编译
- - msys2的安装及配置
- - vscode的编译配置

从我们学习`c++`的时候，基本说的都是使用`gcc/g++`去编译居多，但是在`window`下的话，使用的基本都是`Microsoft`的软件进行`c++`编译（有点废话了，在微软的系统下，不用它的编译ide用啥！）；

但是在使用的过程中，vs太庞大，我有个同事就巨喜欢用vscode（我也比较喜欢vscode！），他想用vscode编译`c++`项目，刚好我也有兴趣，就有了这次的内容！


### window下的c++编译器

`c++`除了我们熟悉的`gcc/g++`外，还有`clang`等，而在windows下还有`vs`使用的`cl`

### vscode的cl编译

通过` Microsoft Visual install`安装`C++ Build Tools`

![](/images/computer/lang/c++/vscode_cl1.png)

安装完成后，检查一下是否已经安装正确

![](/images/computer/lang/c++/vscode_cl2.png)

将想要编译的c++文件的根目录拖动到`vscode`中！

> 这里需要注意哦，不是单个文件，而已文件件，否则就出现文章所没有描述的状况了！

for example：
> 需要编译一个`hello world`的cpp文件

然后再`vscode`中打开hello.cpp，

> 如果你的`vscode`尚未安装c++相关的组件，需要按照提示去安装一下`C++`的扩展包!

直接使用`F5`键！

![](/images/computer/lang/c++/vscode_cl3.png)

这里我们选在的是`C++ (windows)`

![](/images/computer/lang/c++/vscode_cl4.png)

选中后，就等待编译即可，例子的编译大致如下:

![](/images/computer/lang/c++/vscode_cl5.png)

### vscode的`gcc/g++`编译

`Window`是没有`gcc`编译器的，为了能更好地使用`gcc`,需要一个中间的工具，那就是`Msys2`。

#### msys2的安装及配置

- 安装

`Msys2`的安装，这里就不在多说，具体可以自行百度谷歌。

- 配置

为`msys2`安装必要的工具；

`gcc/g++`的安装

使用`pacman -S gcc`安装。

`mingw-w64`安装
然后使用` pacman -S mingw-w64-x86_64-toolchain`

以上两个工具安装完成后，在`Msys2`的目下大致是这样的，如果失败，目录是空的！

![](/images/computer/lang/c++/vscode_gcc1.png)

因为`vscode`是安装在`window`下的，所以需要能在`window`唤起`gcc`指令才可以；

在系统环境变量中加入`Msys2`的`/mingw64/bin`路径到`path`

![](/images/computer/lang/c++/vscode_gcc2.png)

然后开始配置`vscode`的`gcc`环境

#### vscode的编译配置

将想要编译的c++文件的根目录拖动到`vscode`中！

> 这里需要注意哦，不是单个文件，而已文件件，否则就出现文章所没有描述的状况了！

for example：
> 需要编译一个`hello world`的cpp文件

![](/images/computer/lang/c++/vscode_gcc3.png)

然后再`vscode`中打开hello.cpp，

> 如果你的`vscode`尚未安装c++相关的组件，需要按照提示去安装一下`C++`的扩展包!

![](/images/computer/lang/c++/vscode_gcc4.png)

直接使用`F5`键！

![](/images/computer/lang/c++/vscode_gcc5.png)

> 这里先说明一下，`gdb`使用的默认就是`g++`了，所以直接选择`C++ (GDB/lldb)`

![](/images/computer/lang/c++/vscode_gcc6.png)

这里了我们使用的是`mingw64`的，选中后，就等待编译即可，例子的编译大致如下:

![](/images/computer/lang/c++/vscode_gcc7.png)

> 注意了！在第二次生成的时候，一定要是处于打开的cpp文件！否则会出现类似以下这样的问题！
> ![](/images/computer/lang/c++/vscode_gcc8.png)