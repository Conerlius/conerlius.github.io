---
layout:     article
title:      "dynamic_cast的过程"
subtitle:   " \"c++\""
date:       2022-03-25 14:00:00
author:     "Conerlius"
category: c++
keywords: c++
tags:
    - c++
---

为了更加的深入`RTTI`,现在轮到了`Dynamic_cast`了

## 预先说明

![png](/images/computer/lang/c++/rtti/1.png)

如果使用的是vs的话，可以通过这种方式生成`asm`的代码！
因为使用习惯的问题，我使用的是`msys2`进行的操作说明！但是底层原理都是一样的！

## 同类型转换

```c++
class DynamicClassBase {
public:
	virtual void Print() {} // 一定要使用！要不然编译报错!以后会说明！
};
class DynamicClass : public DynamicClassBase {
public:
};

void TestDynamic()
{
	DynamicClass baseValue = DynamicClass();
	DynamicClass* baseValue1 = &baseValue;
	auto dclass = dynamic_cast<DynamicClass*>(baseValue1);
}

int main() {
	TestDynamic();
	return 0;
}
```

我使用的一下指令编译的！

```gdb
g++ -g Test.cpp -o test
```
使用`gdb`查看生成的代码的汇编看看

```assemble
Dump of assembler code for function _Z11TestDynamicv:
10      {
   0x0000000100401080 <+0>:     push   %rbp
   0x0000000100401081 <+1>:     mov    %rsp,%rbp
   0x0000000100401084 <+4>:     sub    $0x20,%rsp

11              DynamicClass baseValue = DynamicClass();
   0x0000000100401088 <+8>:     lea    0x1ff1(%rip),%rax        # 0x100403080 <_ZTV12DynamicClass+16>
   0x000000010040108f <+15>:    mov    %rax,-0x18(%rbp)

12              DynamicClass* baseValue1 = &baseValue;
   0x0000000100401093 <+19>:    lea    -0x18(%rbp),%rax
   0x0000000100401097 <+23>:    mov    %rax,-0x8(%rbp)

13              auto dclass = dynamic_cast<DynamicClass*>(baseValue1);
   0x000000010040109b <+27>:    mov    -0x8(%rbp),%rax
   0x000000010040109f <+31>:    mov    %rax,-0x10(%rbp)

14      }
   0x00000001004010a3 <+35>:    nop
   0x00000001004010a4 <+36>:    add    $0x20,%rsp
   0x00000001004010a8 <+40>:    pop    %rbp
   0x00000001004010a9 <+41>:    ret

End of assembler dump.

```

看到的都是直接返回的地址，看来编译器在这种明确知道类型的情况下，已经没有使用`dynamic_cast`了！

修改一下赋值

```c++
void TestDynamic()
{
	DynamicClassBase* baseValue = new DynamicClass();
	auto dclass = dynamic_cast<DynamicClass*>(baseValue);
}
```

```assemble
Dump of assembler code for function _Z11TestDynamicv:
10      {
   0x0000000100401080 <+0>:     push   %rbp
   0x0000000100401081 <+1>:     push   %rbx
   0x0000000100401082 <+2>:     sub    $0x38,%rsp
   0x0000000100401086 <+6>:     lea    0x30(%rsp),%rbp

11              DynamicClassBase* baseValue = new DynamicClass();
   0x000000010040108b <+11>:    mov    $0x8,%ecx
   0x0000000100401090 <+16>:    call   0x100401130 <__wrap__Znwm>
   0x0000000100401095 <+21>:    mov    %rax,%rbx
   0x0000000100401098 <+24>:    movq   $0x0,(%rbx)
   0x000000010040109f <+31>:    mov    %rbx,%rcx
   0x00000001004010a2 <+34>:    call   0x100401720 <_ZN12DynamicClassC1Ev>
   0x00000001004010a7 <+39>:    mov    %rbx,-0x8(%rbp)

12              auto dclass = dynamic_cast<DynamicClass*>(baseValue);
   0x00000001004010ab <+43>:    mov    -0x8(%rbp),%rax
   0x00000001004010af <+47>:    test   %rax,%rax    // 判空
   0x00000001004010b2 <+50>:    je     0x1004010d2 <_Z11TestDynamicv+82>
   0x00000001004010b4 <+52>:    mov    $0x0,%r9d
   0x00000001004010ba <+58>:    lea    0x1f4f(%rip),%r8        # 0x100403010 <__fu0__ZTVN10__cxxabiv120__si_class_type_infoE>
   0x00000001004010c1 <+65>:    lea    0x1f68(%rip),%rdx        # 0x100403030 <__fu1__ZTVN10__cxxabiv117__class_type_infoE>
   0x00000001004010c8 <+72>:    mov    %rax,%rcx
   0x00000001004010cb <+75>:    call   0x100401100 <__dynamic_cast>
   0x00000001004010d0 <+80>:    jmp    0x1004010d7 <_Z11TestDynamicv+87>
   0x00000001004010d2 <+82>:    mov    $0x0,%eax
   0x00000001004010d7 <+87>:    mov    %rax,-0x10(%rbp)

13      }
   0x00000001004010db <+91>:    nop
   0x00000001004010dc <+92>:    add    $0x38,%rsp
   0x00000001004010e0 <+96>:    pop    %rbx
   0x00000001004010e1 <+97>:    pop    %rbp
   0x00000001004010e2 <+98>:    ret

End of assembler dump.

```