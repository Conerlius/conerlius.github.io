---
layout:     article
title:      "lua库的移动平台重新编译"
subtitle:   " \"slua\""
date:       2021-07-29 11:25:00
author:     "Conerlius"
category: unreal
keywords: unreal
tags:
    - unreal
    - slua
---

## 大纲

- Xcode Command Line Tools安装
- cmake的安装
- Android SDK和Android NDK
- Xcode
- 下载工程
- 编译


## Xcode Command Line Tools安装

使用`macos`的终端，在终端中输入

```
git --version
```

如果电脑尚没有安装啊`git`的话，系统会提示你安装开发工具，只需要点击安装即可！

如果已经安装了，则输出`git`的版本信息;

![](/images/computer/game/ue/slua/mac_git.png)

## cmake的安装

从官网 https://cmake.org/download/ 下载对应的dmg版本;

下载安装完成后，也仅仅只是能通过app去启动；还是没有支持命令调用的！需要进行一下操作

- 打开cmake点击Tools选择How to Install For Command Line Use
- 提供了3中方式，这里使用的是第二种方式
- 执行指令
- 使用指令`camke --version`测试cmake

![](/images/computer/game/ue/slua/mac_cmake1.png)

![](/images/computer/game/ue/slua/mac_cmake2.png)

## Android SDK和Android NDK

由于`unreal`推荐使用的是`r21b`,为了防止有什么问题，这里下载使用的也是这个版本。

> 但是在`macos`下打开的界面不支持`r21`的展示从而获取下载链接，这里就直接使用`curl`去下载了！

```shell
curl https://dl.google.com/android/repository/android-ndk-r21-darwin-x86_64.dmg --ouput android-ndk.dmg
```

解压后，将`NDK`配置到环境中

```shell
vim ~/.bash_profile

export ANDROID_NDK=you_android_ndk_app_path/ndk.app/Contents/NDK
export PATH=${PATH}:${ANDROID}
```

```shell
source ~/.bash_profile
```

ndk测试

```shell
ndk-build
```

提示需要`NDK_PROJECT_PATH`即说明已经成功！

## Xcode

使用命令安装

```shell
xcode-select --install
```

如果是在app里下载`xcode`后,为了防止`cmake`出现类似`CMAKE_C_COMPILER`和`CMAKE_CXX_COMPILER` not set之类的，在`xcode`安装完成后，使用

```shell
sudo xcode-select --reset
```


## 下载工程

这里使用的是http下载tar.gz的形式！

下载完成后，直接解压都桌面即可！

## 编译

进入lua插件编译的目录

`cd ./Plugins/slua_unreal`

然后对将要执行的`make_android.sh`和`make_ios.sh`赋予执行权限，这里就直接`777`了

由于是在macos下的，所以需要对现在有的`sh`文件中`cmake`进行一些处理

```shell
cmake -DCMAKE_MAKE_PROGRAM=/usr/local/bin/cmake -DANDROID_ABI=armeabi-v7a -DCMAKE_TOOLCHAIN_FILE=${ANDROID_NDK}/build/cmake/android.toolchain.cmake -DCMAKE_C_COMPILER=`xcrun -find cc` -DCMAKE_CXX_COMPILER=`xcrun -find c++` -DANDROID_TOOLCHAIN_NAME=arm-linux-androideabi-4.9 -DANDROID_NATIVE_API_LEVEL=android-9 ..
```

```shell
cmake -DCMAKE_MAKE_PROGRAM=/usr/local/bin/cmake -DANDROID_ABI=armeabi-v7a -DCMAKE_TOOLCHAIN_FILE=../cmake/android.toolchain.cmake -DCMAKE_C_COMPILER=`xcrun -find cc` -DCMAKE_CXX_COMPILER=`xcrun -find c++` -DANDROID_TOOLCHAIN_NAME=arm-linux-androideabi-4.9 -DANDROID_NATIVE_API_LEVEL=android-9 ..
```

其他的`DCMAKE_TOOLCHAIN_FILE`修改是参考一下！

```shell
sudo chmod 777 ./make_android.sh
sudo chmod 777 ./make_ios.sh
```

最终结果是上面的仅仅只能编译ios和mac的！！

# 原因

- slua提供的编译脚本使用的是`gcc`，但是在`ndk r21`后就已经禁止使用了！所以编译是不通过的！其他的以后再测试！