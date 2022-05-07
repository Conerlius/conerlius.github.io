---
layout:     article
title:      "Internal的跨dll访问"
subtitle:   " \"c#\""
date:       2022-05-07 12:00:00
author:     "Conerlius"
category: unity
keywords: unity
tags:
    - unity
    - CSharp
---

## 背景

在`Unity`的项目中，经常遇到各种internal的方法或者是类，但是我又需要继承或者使用它，目前我的解决方案有一下几个：

- Reflection
- InternalsVisibleToAttribute
- IgnoresAccessChecksToAttribute
- 修改dll

### Reflection

反射，通过type获取其`fileInfo`、`MethodInfo`等

- Field

```C#
private T GetReflectionField<T>(Type superType, string fieldName)
{
	var reflectFileInfo =superType.GetField(fieldName,
		BindingFlags.Default | BindingFlags.Instance | BindingFlags.NonPublic);
	if (reflectFileInfo == null)
	{
		Debug.LogError($"反射{fieldName} field失败!!");
		return default;
	}
	var reflectData = (T)reflectFileInfo.GetValue(this);
	if (reflectData == null)
	{
		Debug.LogError($"获取{fieldName}失败!!");
		return default;
	}
	return reflectData;
}
```

反射的时候注意一下`BindingFlags`，如果在`Type`中能看得改成员，但是`GetField`获取不到的话，自己尝试一下更换`BindingFlags`

- Method

```c#
private MethodInfo GetReflectionMethod(Type superType, string methodName)
{
	var method = superType.GetMethod(methodName,
		BindingFlags.Default | BindingFlags.Instance | BindingFlags.NonPublic);
	if (method == null)
	{
		Debug.LogError($"反射方法{methodName}失败！！！");
		return default;
	}
	return method;
}
```

其他的还有静态方法之类的，毕竟随便百度一下就能找到了，这里就不再说了！

### InternalsVisibleToAttribute

我们试一下打开看看`Unity.Addressables.Editor.dll`看看

![png](/images/computer/lang/csharp/internal/1.png)

我们留意到以下信息

```
[assembly: CompilationRelaxations(8)]
[assembly: RuntimeCompatibility(WrapNonExceptionThrows = true)]
[assembly: Debuggable(/*Could not decode attribute arguments.*/)]
[assembly: InternalsVisibleTo("Unity.Addressables.Editor.Tests")]
[assembly: InternalsVisibleTo("Unity.Addressables.Tests")]
[assembly: InternalsVisibleTo("PerformanceTests.Editor")]
[assembly: AssemblyVersion("0.0.0.0")]
```
具有这些名称的程序集是 `Unity.Addressables.Editor.dll` 中的`友集`。

我们先将`Addressables`从`Library/PackageCache`移动到`Pacakges`下，然后在`Addressables`中添加一个`CustomAssembly.cs`

```c#
using System.Runtime.CompilerServices;

[assembly: InternalsVisibleTo("自己的dll名称")]
```

编译完成后，我们从dll中看到信息多了一条

![png](/images/computer/lang/csharp/internal/2.png)

然后我们建立自己工程的dll（注意名字要和`CustomAssembly`中的一致）

这样处理完成后，我们就能自己的dll中直接访问`Addressables`中的各种`internal`方法、类等了。


### IgnoresAccessChecksToAttribute

这个方法不是我想要的！所以我放弃了，这里就只是简单把我看的link放出来！

https://qiita.com/mob-sakai/items/f3bbc0c45abc31ea7ac0

https://www.strathweb.com/2018/10/no-internalvisibleto-no-problem-bypassing-c-visibility-rules-with-roslyn/

### 修改dll

```c#
static void Begin()
{
    string dllsDir = "Library/ScriptAssemblies";
    AssemblyDefinition assembly = AssemblyDefinition.ReadAssembly($"{dllsDir}/Unity.Addressables.Editor.dll");
    var attributes = assembly.CustomAttributes;
    foreach (var attribute in attributes)
    {
        Debug.Log(attribute.AttributeType);
    }
    var newAttr = assembly.MainModule.ImportReference(typeof(InternalsVisibleToAttribute).GetConstructor(new []{typeof(string)}));
    var attr = new CustomAttribute(newAttr);
    attr.ConstructorArguments.Add(new CustomAttributeArgument(assembly.MainModule.ImportReference(typeof(string)), "MyNewDllName"));
    assembly.CustomAttributes.Add(attr);
    // 为了做出区别，特意改成不一样的名字！
    assembly.Write($"{dllsDir}/Unity.Addressables.Editor11111.dll");
}
```

使用`mono.cecil`强行对已经生成的`dll`写入`InternalsVisibleToAttribute`,这样就和前面的`InternalsVisibleToAttribute`方案一样了！

![png](/images/computer/lang/csharp/internal/3.png)