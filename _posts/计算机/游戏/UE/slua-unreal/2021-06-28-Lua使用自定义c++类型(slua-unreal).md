---
layout:     article
title:      "Lua使用自定义c++类型"
subtitle:   " \"slua\""
date:       2021-06-28 18:25:00
author:     "Conerlius"
category: unreal
keywords: unreal
tags:
    - unreal
    - slua
---

这一章节的内容就只有一个，那就是如何将自定义的类型注册到lua中使用！

# 大纲
- 书写自定义class
- 标注导出
- 注意事项
- lua中使用

## 书写自定义class

创建一个lua使用的c++类

```c++
class CustomClass
{
	LuaClassBody()
public:
	virtual ~CustomClass();

	void TestMethod();
};
```

`LuaClassBody`这个必须的，因为在后面写导出的时候必须依赖这个！
方法体的具体事项这里就不做说明！

## 标注导出

```c++
DefLuaClass(CustomClass) 
	DefLuaMethod(TestMethod,&CustomClass::TestMethod)
EndDef(CustomClass,nullptr)
```

`DefLuaClass`是实现c++注入到lua的类开头标志！以`EndDef`结束！

`EndDef`中常见有两个参数，一个是类名，第二个是`lua`中创建该类的时候调用的构造方法，这个方法必须是静态方法！

使用`DefLuaClass`导出一个c++类，如果存在父类，可以在，后面填写，支持多继承，如果存在多个就写多个，例如：
```c++
DefLuaClass(Base)
DefLuaClass(Foo,Base) 
DefLuaClass(Foo,Base,Object) 
```

`DefLuaMethod`导出类方法，同时支持支持`static`方法和成员方法，格式为：

```c++
DefLuaMethod(方法名字，函数指针)
```
如果c++函数存在多个重载版本，可以使用`DefLuaMethod_With_Type`宏 明确到底使用哪个版本的函数，例如：

```c++
DefLuaMethod_With_Type(getFruit_1,&Foo::getFruit,Fruit (Foo::*) ())
DefLuaMethod_With_Type(getFruit_2,&Foo::getFruit,Fruit (Foo::*) (int))
```

如果你希望使用lambda表达式作为函数实现，导出给lua，可是使用 `DefLuaMethod_With_Lambda `宏，例如：

```c++
DefLuaMethod_With_Lambda(helloWorld,false,[]()->void {
            slua::Log::Log("Hello World from slua");
        })
```

## 注意事项

前文说的`EndDef`构造方法，如果在方法中返回了类指针，默认lua是不负责生命周期管理的，你需要自己管理类的生命周期，比如自己提供对应`destroy`方法去销毁对象。如果你希望lua管理生命周期，在lua没有引用后自行销毁，你需要返回`LuaOwnedPtr`封装的对象指针，这样则lua会管理对象生命周期，同时在c++中不应该有任何代码去销毁对应的对象，否则会造成不可知的错误，例如：

```c++
	// constructor function for lua
    // LuaOwnedPtr will hold ptr by lua and auto collect it;
    static LuaOwnedPtr<Foo> create(int v) {
        return new Foo(v);
    }

    // raw ptr not hold by lua, lua not collect it
    static Foo* getInstance() {
        static Foo s_inst(2048);
        return &s_inst;
    }
```
`create`函数返回的`new Foo(v)` 收到lua管理，而`getInstance`返回的`Foo*`不受lua管理，lua不会回收。

完整示例及说明

```c++
#pragma once

#include "CoreMinimal.h"
// 必须include slua.h
#include "slua.h"

// 这个是必须要的！！
using namespace NS_SLUA;
/**
 * 
 */
class CustomClass
{
    // 这个也是必须的！
	LuaClassBody()
public:
	virtual ~CustomClass()
    {
        Log::Log("Base object %p had been freed", this);
    }

	void TestMethod()
    {
        Log::Log("TestMethod call");
    }
    static LuaOwnedPtr<CustomClass> CreateInstance()
    {
        return new CustomClass();
    }
};
// 这些可以不放置在头文件中，放置在.cpp中都可以！
DefLuaClass(CustomClass)
	DefLuaMethod(TestMethod, &CustomClass::TestMethod)
EndDef(CustomClass, &CustomClass::CreateInstance)
```

## lua中使用

```lua
local cc = CustomClass()
print(cc)

cc.TestMethod()
```
