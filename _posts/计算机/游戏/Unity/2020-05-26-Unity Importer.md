---
layout:     article
title:      "Unity importer"
subtitle:   " \"unity\""
date:       2020-05-26 15:00:00
author:     "Conerlius"
category: unity
keywords: unity,addressableimporter
tags:
    - unity
---

之所以写这一片文章记录，是因为unity addressable的lua文件的读取支持问题！

## ScriptedImporter

ScriptedImporter是Unity2017.1引入的功能，使用它，我们可以让Unity识别很多它无法识别的文件，比如Excel(.xlsx)文件，自定义的一些后缀的文件。

所以我们需要在unity导入资源的时候，将其转换成unity能识别的资源类型就可以了！


```c#
using System.IO;
using UnityEditor.Experimental.AssetImporters;
using UnityEngine;

namespace DefaultNamespace
{
	[ScriptedImporter(1, "lua")]
	public class LuaScriptImporter:ScriptedImporter
	{
		public override void OnImportAsset(AssetImportContext ctx)
		{
			var text = File.ReadAllText(ctx.assetPath);
			var asset = new TextAsset(text);
			ctx.AddObjectToAsset(Path.GetFileName(ctx.assetPath), asset);
			ctx.SetMainObject(asset);
		}
	}
}
```