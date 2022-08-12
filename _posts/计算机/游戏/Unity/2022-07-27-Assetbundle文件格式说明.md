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
    uint32 CompressBlockAndCabInfoSize
    uint32 UnCompressBlockAndCabInfoSize
    uint32 Flag 
    align 16 // 16位对齐
    byte[] CompressData1
    byte[] CompressData2
}
```

`CompressData1`解压后的格式`UnCompressData1`

```mermaid
classDiagram
class UnCompressData1 {
    byte[16] DataHash
    int32 BlocksInfoCount
    List~Class BlockInfo~ 数量为BlocksInfoCount
    int32 CabInfoCount
    List~Class CabInfo~ 数量为CabInfoCount
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
class CabInfo {
    int64 Offset
    int64 Size
    uint32 Flag
    string Path
}
```

![](/images/computer/game/unity/assetbundle/1.png)

使用前面得到的BlockInfo,对ComPressData2进行解压和重新生成
新的buffer


```mermaid
classDiagram
class CompressData2 {
    byte[] Buffer //大小为所有的BlockInfo.CompressedSize
}
```
`CompressData2`解压后的格式`UnCompressData2`,解压后的`UnCompressData2`内容如下:

![](/images/computer/game/unity/assetbundle/2.png)


```mermaid
classDiagram
class UnCompressData2{
    List~class CabContent~  // 数量 UnCompressData1.CabInfoCount
}
```

每个cab的大小在CabInfo中能找到(CabInfo.Size和CabInfo.Offset)

然后来看看CabContent

```mermaid
classDiagram
class CabContent {
    uint32 old_MetaDataSize
    uint32 old_FileSize
    uint32 Version
    unit32 old_DataOffset
    byte Endianess
    byte Reserved
    unit32 MetadataSize
    int64 FileSize
    int64 DataOffset
    int64 unknow
    string unityVersion
    int32 targetPlatform
    bool typeTreeEnable
    int32 typeCount
    List~TypeTree~ typeTree
    int32 objectCount
    List~Object~ objects
    int32 scriptCount
    List~Script~ scripts
    int32 externalCount
    List~externalRef~ externalRefs
    int32 refTypesCount
    List~externalRef~ refTypeTree
    string userInfo
}
```
![](/images/computer/game/unity/assetbundle/3.png)

`All ObjectData`里存放的是`List<Object> objects`的数据！不同的文件格式就不一样！

