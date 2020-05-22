---
layout:     article
title:      "Injection的基本使用"
subtitle:   " \"physic\""
date:       2020-04-28 18:00:00
author:     "Conerlius"
category: unity
keywords: Injection
tags:
    - Injection
    - CSharp
---


##  前提

在看mono.Ceil前有几个需要提前说明的，注入的时候需要参考il指令，每个il的指令的意义可自行通过`百度`、`google`学习

### IL

> 在讲il之前，先看看怎么查看il代码

- 怎么查看il代码？

  我使用的工具是rider（在unity中使用的！其他ide只能自行百度了！）。

  - 首先是编译项目

    ![injection_3](/images\injection/injection_3.png)

    然后就是等待编译完成，成功后，`il view`会自动刷新，失败就保留上一次的结果。

  - 打开il视图

    ![injection_4](/images\injection/injection_4.png)

- il是什么？

  前面说了怎么看il代码了，那么现在我们就说说il！

  通过一个简单的例子说明吧：

  ```c#
  using UnityEngine;
  
  public class TestInject
  {
  	void main(int[] args)
  	{
  		// 声明一个int并赋值存储
  		int intValue = 1;
  		// 声明一个string并赋值存储
  		string stringValue = "1";
  		// 调用静态方法打印stringValue
  		Debug.LogError(stringValue);
  	}
  }
  ```

  这份代码编译后的il如果，我顺带不上了一下说明，方便大家理解

  ```c#
  // Type: TestInject 
  // Assembly: Assembly-CSharp, Version=0.0.0.0, Culture=neutral, PublicKeyToken=null
  // MVID: 7F10B9DF-01DB-4FCF-9D05-6C8A452F2206
  // Location: XXX\Temp\bin\Debug\Assembly-CSharp.dll
  // Sequence point data from XXX\Temp\bin\Debug\Assembly-CSharp.pdb
  
  /* .class表示的Program是一个类，extends 代表Program类继承于程序集mscorlib中的System.Object类，这就告诉我们，在C#中所有的类的父类都是Object。
   * private为访问权限，表明该类是私有的。
   * auto：表明程序加载的时候内存布局是有CLR决定的，而不是由程序本身控制的。
   * ansi:表明类的编码为ansi编码
   * beforefieldinit ：表明CLR可以在第一次访问静态字段之前的任何时刻执行构造函数，而使用beforefieldinit属性可以提高性能。
   * TestInject类型名称
   * extends 表示继承，后面跟随的是被继承者
   */
  .class public auto ansi beforefieldinit
    TestInject
      extends [netstandard]System.Object
  {
  /* .method表示这是一个方法
   * private表示这是一个私有的方法，同样的还有public
   * hidebysig指令表示如果当前类作为父类，用该指令标记的方法将不会被子类继承
   * instance表示该方法将由对象来关联这个方法
   * void返回类型
   * main方法名称
   * cil managed表明方法中的代码是IL代码，且是托管代码，即运行在CLR运行库中的代码
   */
    .method private hidebysig instance void
      main(
        int32[] args
      ) cil managed
    {
    /*.maxstack 表明执行构造函数时，评估堆栈可容纳数据项的最大个数。评估堆栈是保存方法中所需变量的值的一个内存区域，该区域在方法执行结束时会被清空，或者存储一个返回值
     */
      .maxstack 1
  /* .locals init方法中用到的变量
   * [1] string stringValue表示定义string 类型的变量，变量名成为：stringValue
   */
      .locals init (
        [0] int32 intValue,
        [1] string stringValue
      )
  /* // [6 2 - 6 3]表示下方指令在代码源文件中的位置
   * IL_0000是代码行的开头。一般在IL_标记之前的部分为变量的声明和初始化操作
   * nop不做任何操作
   */
      // [6 2 - 6 3]
      IL_0000: nop
  /* ldc.i4.1将整数值 1 作为 int32 推送到计算堆栈上
   * stloc.0弹出栈顶的值，赋值到本地变量0
   */
      // [8 3 - 8 20]
      IL_0001: ldc.i4.1
      IL_0002: stloc.0      // intValue
  /* ldstr将string放置在堆栈中
   * stloc.1弹出栈顶的值，赋值到本地变量1,这里注意到stloc.0，其实是stloc.index,表示弹出栈顶的值，赋值到本地变量index，对应.locals init
   */
      // [10 3 - 10 28]
      IL_0003: ldstr        "1"
      IL_0008: stloc.1      // stringValue
  /* ldloc.1加载本地变量 1 到堆栈中，和stloc.index一样，前者是读，后者是存
   * call调用由传递的方法说明符指示的方法
   * nop表示无操作
   */
      // [12 3 - 12 31]
      IL_0009: ldloc.1      // stringValue
      IL_000a: call         void [UnityEngine.CoreModule]UnityEngine.Debug::LogError(object)
      IL_000f: nop
    /*ret表示返回*/
      // [13 2 - 13 3]
      IL_0010: ret
  
    } // end of method TestInject::main
   
  
  /* specialname 特殊名称声明
   * .ctor 表示构造函数
   * 其他的同上面分解的！
      */
  
    .method public hidebysig specialname rtspecialname instance void
      .ctor() cil managed
    {
      .maxstack 8
  /* ldarg将参数（由指定索引值引用）加载到堆栈上
   */
      IL_0000: ldarg.0      // this
      IL_0001: call         instance void [netstandard]System.Object::.ctor()
      IL_0006: nop
      IL_0007: ret
  
    } // end of method TestInject::.ctor
  } // end of class TestInject
  ```

  如果细心的，大家估计都发现了，这就是一个过程化的程序流水！

  至于更多的il指令，可以自行去`维基百科`看看

## mono.ceil的使用

> 现在开始，我们就进入正题

### 加载dll

`AssemblyDefinition assembly  = AssemblyDefinition.ReadAssembly(DLL_Path);`

以上一个指令就能加载并解释dll了

### 输出dll

`assembly.Write(DLL_Path);`

需要覆盖还是生成新的，就看`DLL_Path`有没有资源了

### mono.ceil的注入方式

`mono.ceil`的注入其实就是在执行的il里加入（或修改）il指令,所以才有了前面说的il前提。

> 下面大致讲讲`mono.ceil`的几种常见使用

- 变量声明

  ```c#
  // 找到需要注入的method
  // method.Body方法主体
  MethodBody targetBody = method.Body;
  // 获取方法的il
  var worker = targetBody.GetILProcessor();
  // 声明方法体有局部变量
  targetBody.InitLocals = true;
  // 变量的类型
  var variable = new VariableDefinition(claDef);
  // 将变量添加到方法体中
  targetBody.Variables.Add(variable);
  ```

- 变量赋值

  比如需要对一个**string** 变量**`strValue`**进行赋值(变量声明就不说了)

  ```c#
  // 获取方法的il
  var worker = targetBody.GetILProcessor();
  // 获取要写入的位置，现在就先在第一行
  var ins = targetBody.Instructions[0];
  worker.InsertBefore(ins, worker.Create(OpCodes.ldstr, "your string value"));
  // 假设声明的变量是第一个
  worker.InsertBefore(ins, worker.Create(OpCodes.stloc.0, strValue));
  ```

- 导入引用类型

  ```C#
  assembly.MainModule.ImportReference(typeof(typName));
  ```

- 静态方法调用

  ```c#
  worker.InsertBefore(ins, worker.Create(OpCodes.Call,
  assembly.MainModule.ImportReference(typeof(typeName).GetMethod(method_name))));
  ```

- 对象方法调用

  对象方法的调用其实就是在调用call前加载一下对象

  ```c#
  worker.InsertBefore(ins, worker.Create(OpCodes.Ldloc, variable));
  worker.InsertBefore(ins, worker.Create(OpCodes.Call,
  assembly.MainModule.ImportReference(typeof(typeName).GetMethod(method_name))));
  ```

- 其他

  ```C#
  // 创建对象
  worker.InsertBefore(ins, worker.Create(OpCodes.Newobj, 类型的构造方法));
  ```

## Demo

下面举个demo说明一下!

比如我们有一下一个类型:

```C#
using UnityEngine;

[MyAttr]
public class ClassB
{
	public int Print()
	{
		Debug.LogError("ClassB");
		return 1;
	}
}
```

现在我们需要使得在打印`“ClassB”`之前，创建一个`ClassA`并执行其打印,一下就是injection的代码:

```C#
static void Inject(string path)
    {
        // 加载第一个Assembly-CSharp.dll
        AssemblyDefinition csharp  = AssemblyDefinition.ReadAssembly($"{path}/Assembly-CSharp.dll");
        // 依赖的dll
        AssemblyDefinition DLL1  = AssemblyDefinition.ReadAssembly($"{path}/DLL1.dll");
        // 因为dll1中只有一个自定义attributes，所以这里就直接取0
        var customAttribute = DLL1.CustomAttributes[0];
        // ClassA是DLL1的，在这里我们可以直接访问使用,并将之导入到Assembly-CSharp.dll
        var Constructor = csharp.MainModule.ImportReference(typeof(ClassA).GetConstructor(new Type[] { }));
        //var claDef = DLL1.MainModule.Types.Single(definition =>  definition.Name == "ClassA");
        var claDef = csharp.MainModule.ImportReference(typeof(ClassA));
        // Assembly-CSharp.dll的所有module
        foreach (var module in csharp.Modules)
        {
            // 每个module都导入ClassA,可以直接在mainModule里导入ClassA就行
            module.ImportReference(claDef);
            // 所有类型
            foreach (var moduleType in module.Types)
            {
                // 判定该类型是否需要injection
                if (CheckAttr(moduleType))
                {
                    // 类型下的所有method
                    foreach (var method in moduleType.Methods)
                    {
                        // 方法主体
                        MethodBody targetBody = method.Body;
                        // 获取il
                        var worker = targetBody.GetILProcessor();
                        // 注入局部变量
                        targetBody.InitLocals = true;
                        var variable = new VariableDefinition(claDef);
                        targetBody.Variables.Add(variable);
                        // 获取方法的第一个指令位置，也可以查找自己特定的位置,这里就不复杂处理了
                        var ins = targetBody.Instructions[0];
                        worker.InsertBefore(ins, worker.Create(OpCodes.Newobj, Constructor));
                        worker.InsertBefore(ins, worker.Create(OpCodes.Stloc, variable));
                        worker.InsertBefore(ins, worker.Create(OpCodes.Ldloc, variable));
                        worker.InsertBefore(ins, worker.Create(OpCodes.Call,
                            csharp.MainModule.ImportReference(typeof(ClassA).GetMethod("Print"))));
                    }
                }
            }
        }
        csharp.Write($"{path}/Assembly-CSharp.dll");
    }
```

injection后代码如下了！

```C#
// ClassB
using UnityEngine;

[MyAttr]
public class ClassB
{
	public int Print()
	{
		ClassA classA = new ClassA();
		classA.Print();
		Debug.LogError("ClassB");
		return 1;
	}

	public ClassB()
	{
		new ClassA().Print();
		base..ctor();
	}
}
```

最后送上悬疑图，有兴趣的可以去了解一下

![png](/images/injection/injection_5.png)