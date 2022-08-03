---
layout:     article
title:      "Assetbundle文件格式说明"
subtitle:   " \"unity\""
date:       2022-07-27 11:25:00
author:     "Conerlius"
category: unity
keywords: unity
tags:
    - unity
---


前段时间趁着工作需要，顺便把assetbundle的文件格式看了~在此做一下记录。

```mermaid
classDiagram
class AssetBundle{
    string Signature
    uint32 Version
    string UnityVersion
    string UnityRevision
    int64 Size
    uint32 CompressBlockAndDirectoryInfoSize
    uint32 UnCompressBlocksInfoSize
    uint32 Flag 
    align 16 // 16位对齐
    byte[] CompressBlockAndDirectoryInfos
    List~Class CompressBlockContent~
}
```

`List<Class CompressBlockContent>`是指有多个的意思，具体数量`CompressBlockAndDirectoryInfoSize`

`byte[] CompressBlockAndDirectoryInfos` 大小为`AssetBundle.CompressBlockAndDirectoryInfoSize`


`CompressBlockAndDirectoryInfos`解压后的格式`UnCompressBlockAndDirectoryInfos`

```mermaid
classDiagram
class UnCompressBlockAndDirectoryInfos{
    byte[16] DataHash
    int32 BlocksInfoCount
    List~Class BlockInfo~ 数量为BlocksInfoCount
    int32 DirectoryCount
    List~Class DirectoryInfo~ 数量为DirectoryCount
}
```

```mermaid
classDiagram
class BlockInfo {
    uint32 UnCompressedSize
    uint32 CompressedSize
    uint16 Flag
}
```

```mermaid
classDiagram
class DirectoryInfo {
    int64 Offset
    int64 Size
    uint32 Flag
    string Path
}
```

```mermaid
classDiagram
class CompressBlockContent {
    byte[] Buffer //大小为BlockInfo.CompressedSize
}
```
`CompressBlockContent`解压后的格式`UnCompressBlockContent`

```mermaid
classDiagram
class UnCompressBlockContent{
    List~class DirectoryContent~ 数量 UnCompressBlockAndDirectoryInfosContent.DirectoryCount
}
```