---
layout:     article
title:      "如何生成C#版的protobuf"
subtitle:   " \"protobuf\""
date:       2019-07-31 18:22:00
author:     "Conerlius"
category: 其他
keywords: CSharp
tags:
    - CSharp
---
* content
{:toc}

## 环境

> 首先说明一下生成的系统环境，及其一些配置

因为项目是unity的，所以我会把生成的方式集成到unity菜单下，方便使用。
系统： windows 10

## 步骤

将proto文件编译成.cs文件。.net版的protobuf来源于proto社区，有两个版本：
> 一个版本叫protobuf-net，下载地址为：[https://github.com/mgravell/protobuf-net](https://github.com/mgravell/protobuf-net)  写法上比较符合c#一贯的写法，而且效率更高。
>
> 另一个为protobuf-csharp-sport ， 官方站点：[http://code.google.com/p/protobuf-csharp-port/](http://code.google.com/p/protobuf-csharp-port/) 写法上跟java上的使用极其相似，比较遵循Google 的原生态写法，跨平台选择此版本比较好。
> 
这里使用的是protobuf-net，下载解压后，用vs打开工程，将编译配置调成unity。在工程里我们只需要编译两个文件就可以了precompile和ProtoGen（我用的是r668,最新版本试过编译会报错），编译成功后将Precompile\precompile.exe 以及 ProtoGen\protogen.exe 两个文件加入到环境变量中，打开cmd，使用方法如下：

```
protogen -i:input.proto -o:output.cs  
protogen -i:input.proto -o:output.xml -t:xml  
protogen -i:input.proto -o:output.cs -p:datacontract -q  
protogen -i:input.proto -o:output.cs -p:observable=true  
```