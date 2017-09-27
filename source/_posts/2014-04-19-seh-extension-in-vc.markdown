---
layout: post
title: "VC中对SEH的扩展"
date: 2014-04-19 16:30:54 +0800
comments: true
categories:
---
这篇文章是早前跟某同事探讨Windows SEH中`__finally`实现时研究的内容, 根据某书介绍, 异常处理函数都是通过`_EXCEPTION_REGISTRATION_RECORD`内的回调函数`Handler`实现的:

``` c
typedef struct _EXCEPTION_REGISTRATION_RECORD
{
   struct _EXCEPTION_REGISTRATION_RECORD *Next;
   PEXCEPTION_ROUTINE                     Handler;
} EXCEPTION_REGISTRATION_RECORD, *PEXCEPTION_REGISTRATION_RECORD;

__try
{
    ...
}
__except( XXX )
{
    // 这里是对应那个_EXCEPTION_REGISTRATION_RECORD 的第二个成员Handler
    ...
}
```

但是对于`__finally`却没有介绍具体的实现方式. 通过调试器trace不到`__finally`的调用路径.
<!-- more -->

限于当时搜商有限, 没有找到比较好的资料, 只能结合<<软件调试>>中介绍的内容自己进行摸索实验.
写了一段包含`__try`, `__finally`的简单代码进行反汇编, 发现VC对SEH做了些扩展, 往ExceptionList中添加的链表节点并不是书上描述的`_EXCEPTION_REGISTRATION_RECORD`,而是`_EH3_EXCEPTION_REGISTRATION`, 并且多向栈里压了8个字节. 同时异常处理函数也不是`__except`或者`__finally`里面的代码,而是统一的`__except_handler3`(老的编译器可能是`__except_handler2`, 还有那个某网站上贴的代码是`__except_handler4`, 不同版本VC扩展的结构不太一样, 但基本原理都差不多), 真正的`__finally`和`__except`中的处理代码放在`_EXCEPTION_REGISTRATION_RECORD`里的`_SCOPETABLE_ENTRY`中:

``` c
struct _EH3_EXCEPTION_REGISTRATION
{
    // 前两个字段同_EXCEPTION_REGISTRATION_RECORD
	_EH3_EXCEPTION_REGISTRATION * Next;
	void * ExceptionHandler;
	_SCOPETABLE_ENTRY * ScopeTable;
	DWORD TryLevel;
};
```

``` c
typedef struct _SCOPETABLE_ENTRY
{
	DWORD EnclosingLevel;

    // 这个是过滤函数的地址, 就是__except(xxx)括号中得那部分代码.
    // 可以是函数指针或者一个常数, 如果是常数的话一般是mov + ret这两条指令
    // 另外如果FilterFunc是NULL, 那下面HandlerFunc中指向的就是__finally而不是__except
	PVOID FilterFunc;

    // 这里保存的是__finally或__except包含的代码块
	PVOID HandlerFunc;
} SCOPETABLE_ENTRY, *PSCOPETABLE_ENTRY;
```

下面整段 `__try`, `__finally`的反汇编:

``` c
int func_in()
{
009F13C0  push        ebp  
009F13C1  mov         ebp,esp
                      // 从这里开始向ExceptionList加入新的节点, 先压栈的是TryLevel
009F13C3  push        0FFFFFFFFh
                      // 接着压栈ScopeTable,保存了__finally里的代码段
009F13C5  push        offset ___rtc_tzz+108h (9F6B40h)
                      // 后面压入的就是_EXCEPTION_REGISTRATION_RECORD里的两个字段, 首先是回调函数, 一直是__except_handler3
009F13CA  push        offset @ILT+115(__except_handler3) (9F1078h)  
                      // 最后压入的是ExceptionList的头结点
009F13CF  mov         eax,dword ptr fs:[00000000h]
009F13D5  push        eax  
009F13D6  mov         dword ptr fs:[0],esp  
009F13DD  add         esp,0FFFFFF38h  
009F13E3  push        ebx  
009F13E4  push        esi  
009F13E5  push        edi  
009F13E6  lea         edi,[ebp-0D8h]  
009F13EC  mov         ecx,30h  
009F13F1  mov         eax,0CCCCCCCCh  
009F13F6  rep stos    dword ptr es:[edi]  
	__try
009F13F8  mov         dword ptr [ebp-4],0  
	{
		printf("fin try!\n");
009F13FF  mov         esi,esp  
009F1401  push        offset string "fin try!\n" (9F574Ch)  
009F1406  call        dword ptr [__imp__printf (9F82C0h)]  
009F140C  add         esp,4  
009F140F  cmp         esi,esp  
009F1411  call        @ILT+320(__RTC_CheckEsp) (9F1145h)  
		__leave;
009F1416  jmp         func_in+62h (9F1422h)  
		*(int*)0=0;
009F1418  mov         dword ptr ds:[0],0  
	}
	__finally
009F1422  mov         dword ptr [ebp-4],0FFFFFFFFh
009F1429  call        $LN5 (9F1430h)  
009F142E  jmp         $LN8 (9F1448h)  
	{
		printf("fin finally!\n");
		              // 从这里开始是__finally中代码块的起始地址, 注意地址是0x9F1430
009F1430  mov         esi,esp
009F1432  push        offset string "fin finally!\n" (9F573Ch)  
009F1437  call        dword ptr [__imp__printf (9F82C0h)]  
009F143D  add         esp,4  
009F1440  cmp         esi,esp  
009F1442  call        @ILT+320(__RTC_CheckEsp) (9F1145h)  
$LN6:
009F1447  ret  
	}
	return 0;
009F1448  xor         eax,eax  
}
```

函数在进入`__try`块前, 首先在栈上安装了自己的异常处理函数, 并把`__finally`或者`__except`的放入ScopeTable做为参数同样压栈, 通过调试器分析压入的ScopeTable地址`0x9F6B40`处的内容:

![SEH Memory Dump](/images/2014/04/seh-memory-dump.png)

`0x9F6B40 + 8`也就是`_SCOPETABLE_ENTRY::HandlerFunc`正好指向`0x009f1430`, 对应`__finally`里代码段的地址. 如果是`__except`, 那这里对应的就是`__except`中的代码段.

<<软件调试>>中有一部分对SEH的分析, 该书的确是本好书, 即使不做Windows开发拿来学习操作系统也是相当不错的. 并且作者十分负责, 出版之后发布了一个电子版的补编, 补全了很多没有写入书中的内容(包括该`__finally`的实现).
