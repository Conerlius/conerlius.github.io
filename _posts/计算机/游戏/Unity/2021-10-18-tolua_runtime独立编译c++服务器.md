---
layout:     article
title:      "tolua_runtime独立编译c++服务器"
subtitle:   " \"unity\""
date:       2021-10-18 11:25:00
author:     "Conerlius"
category: unity
keywords: unity
tags:
    - unity
    - lua
---

## 大纲
- 背景交代
- 编译独立的win库
- 问题及修改

## 背景交代

项目想要前端和后端使用的相通的`lua`文件作为逻辑处理或者是验证！这样能避免不少重复的工作量；
而`c++`调用`lua`的效率是比较高的；但是项目使用的是`tolua`的，为了在目前不动（少改动）的前提下，最好是将`tolua`编译一份放置到后端！

***这里不考虑lua 多线程的问题！***

## 编译独立的win库

- 下载tolua-runtime
  
  直接从`github`的[tolua_runtime](https://github.com/topameng/tolua_runtime/tree/master)下载就可以了！

- 配置生成tolua工程
  
  因为`tolua_runtime`使用的的是`Cross-compiling`,但是我需要的是`vs2019`编译,所以我直接使用了`cmake`来生成！

  这里就不重复说明`cmake`的作用了！

  ```cmakelist
  cmake_minimum_required(VERSION 2.8)
  
  set(CMAKE_BUILD_TYPE Debug)
  
  set(CMAKE_CXX_STANDARD 99)
  
  project(tolua)
  
  
  set(LUA_INC_PATH ${EXT_SRC_ROOT}/lua)
  set(LUA_SRC_PATH ${EXT_SRC_ROOT}/lua)
  
  set(LUA_SRC_FILES 
  	tolua.c
  	int64.c
  	uint64.c
  	pb.c
  	lpeg.c
  	struct.c
  	cjson/strbuf.c
  	cjson/lua_cjson.c
  	cjson/fpconv.c
  	luasocket/auxiliar.c
  	luasocket/buffer.c
  	luasocket/except.c
  	luasocket/inet.c
  	luasocket/io.c
  	luasocket/luasocket.c
  	luasocket/mime.c
  	luasocket/options.c
  	luasocket/select.c
  	luasocket/tcp.c
  	luasocket/timeout.c
  	luasocket/udp.c
  	luasocket/wsocket.c
  )
  
  include_directories(
      luajit-2.1/src
  	luasocket
  )
  
  add_library(tolua SHARED
      ${LUA_SRC_FILES}
  )
  
  link_libraries(tolua ws2_32)
  ```

  然后通过
  ```bat
  mkdir build_win32 & pushd build_win32
  
  cmake -G "Visual Studio 16 2019" -A Win32 ..
  ```
  以上就能在`build_win32`中生成对应的vs工程了！

- 生成luajit

  这里使用的是`x86`的，所以无需在进行其他参数控制;使用`Visual Studio Command Prompt`执行一下指令（先`cd`到luajit的目录下）

  ```bat
  cd src
  msvcbuild.bat
  ```

  一般都没有什么问题！如果需要添加参数，可以直接看`msvcbuild.bat`的代码！

## 问题及修改

因为前面写的`cmakelist`只是简简单单填写了一些，还需要打开工程修改一下配置！

- 将工程从`使用工具`切换成为`动态库`
- 添加`_WIN32_WCE`宏
- 添加库路径
  `luajit-2.1\src`
- 添加依赖的lib
  `luajit.lib` `lua51.lib` `ws2_32.lib`

完成上述步骤后，编译还有一个`strcasecmp`找不到符号！这里需要在`lua_cjson.c`中添加

```c
#ifdef _MSC_VER
#define strcasecmp stricmp
#define strncasecmp  strnicmp
#endif
```

到此`tolua_runtime`的库已经编译成功！