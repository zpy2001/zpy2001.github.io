---
title: Linux内核设计的艺术 - 问题
description: "Questions about the art of Linux kernel design"
slug: system-linux-art-questions
date: 2023-12-10 00:00:00+0000
categories:
    - Linux
tags:
    - OS
    - CS
    - Linux
weight: 1
---

# Questions of Linux Art

## 进程2(Shell)

为何要提前打开stdio文件，然后再关闭

## IA32 Assembly

```asm
push
pop
cld;rep;stosl
std
```

# Homework Questions

## 第二次作业

{{% hint degree="warning" title="问题" %}}
1. 为什么开始启动计算机的时候，执行的是BIOS代码而不是操作系统自身的代码？
{{% /hint %}}

刚启动计算机的时候内存里没有任何合法数据，需要从BIOS芯片开始执行，从磁盘搬运操作系统代码到内存之后才可以执行。

{{% hint degree="warning" title="问题" %}}
2. 为什么BIOS只加载了一个扇区，后续扇区却是由`bootsect`代码加载？为什么BIOS没有直接把所有需要加载的扇区都加载？
{{% /hint %}}

不同操作系统的实现是不同的，`bootsect`之外的代码大小和位置都不容易确定，因此主板BIOS和操作系统遵循统一设计约定，只将最多一个扇区(512B)的`bootsect`放在起始位置，BIOS仅负责加载`bootsect`，剩下的操作系统内核程序由`bootsect`自行/自由加载

{{% hint degree="warning" title="问题" %}}
3. 为什么BIOS把`bootsect`加载到`0x07c00`，而不是`0x00000`？加载后又马上挪到`0x90000`处，是何道理？为什么不一次加载到位？
{{% /hint %}}

1. 内存起始处已经设置了中断向量表，将`bootsect`加载到`0x0`会引起冲突；
2. 操作系统可以自由决定内存布局，原先BIOS所在的`0x7c00`处将会有其他的数据结构占用，需要移往更高的安全地址，Linux选取的是`0x90000`；
3. 不一次加载到`0x90000`处是因为BIOS设计者向前兼容了内存只有640KB或更少内存的旧设备，将BIOS的寻址空间作了限制，而两段加载的方式给予操作系统设计者更加自由的内存安排选择。

{{% hint degree="warning" title="问题" %}}
4. `bootsect`、`setup`、`head`程序之间是怎么衔接的？给出代码证据。
{{% /hint %}}

{{% hint degree="warning" title="问题" %}}
5. setup程序的最后是`jmpi 0,8` ，为什么这个8不能简单的当作阿拉伯数字8看待，究竟有什么内涵？
{{% /hint %}}

{{% hint degree="warning" title="问题" %}}
6. 保护模式在“保护”什么？它的“保护”体现在哪里？特权级的目的和意义是什么？分页有“保护”作用吗？
{{% /hint %}}

{{% hint degree="warning" title="问题" %}}
7. 在`setup`程序里曾经设置过`gdt`，为什么在`head`程序中将其废弃，又重新设置了一个？为什么设置两次，而不是一次搞好？
{{% /hint %}}

{{% hint degree="warning" title="问题" %}}
8. 内核的线性地址空间是如何分页的？画出从`0x000000`开始的7个页（包括页目录表、页表所在页）的挂接关系图，就是页目录表的前四个页目录项、第一个个页表的前7个页表项指向什么位置？给出代码证据。
{{% /hint %}}

{{% hint degree="warning" title="问题" %}}
9. 根据内核分页为线性地址恒等映射的要求，推导出四个页表的映射公式，写出页表的设置代码。
{{% /hint %}}

{{% hint degree="warning" title="问题" %}}
10. 为什么不用`call`，而是用`ret`“调用”`main`函数？画出调用路线图，给出代码证据。
{{% /hint %}}

## 第三次作业
  
{{% hint degree="warning" title="问题" %}}
1、计算内核代码段、数据段的段基址、段限长、特权级。
{{% /hint %}}

{{% hint degree="warning" title="问题" %}}
2、计算进程0的代码段、数据段的段基址、段限长、特权级。
{{% /hint %}}

{{% hint degree="warning" title="问题" %}}
3、`fork`进程1之前，为什么先调用`move_to_user_mode()`？用的是什么方法？解释其中的道理。
{{% /hint %}}

{{% hint degree="warning" title="问题" %}}
4、根据什么判定`move_to_user_mode()`中`iret`之后的代码为进程0的代码。
{{% /hint %}}

{{% hint degree="warning" title="问题" %}}
5、进程0的`task_struct`在哪？具体内容是什么？给出代码证据。
{{% /hint %}}

{{% hint degree="warning" title="问题" %}}
6、在`system.h`里  

```C
#define _set_gate(gate_addr,type,dpl,addr) \  
__asm__ ("movw %%dx,%%ax\n\t" \  
    "movw %0,%%dx\n\t" \  
    "movl %%eax,%1\n\t" \  
    "movl %%edx,%2" \  
    : \  
    : "i" ((short) (0x8000+(dpl%%13)+(type%%8))), \  
    "o" (*((char *) (gate_addr))), \  
    "o" (*(4+(char *) (gate_addr))), \  
    "d" ((char *) (addr)),"a" (0x00080000))

#define set_intr_gate(n,addr) \  
    _set_gate(&idt[n],14,0,addr)

#define set_trap_gate(n,addr) \  
    _set_gate(&idt[n],15,0,addr)

#define set_system_gate(n,addr) \  
    _set_gate(&idt[n],15,3,addr)
```

读懂代码。这里中断门、陷阱门、系统调用都是通过`set_gate`设置的，用的是同一个嵌入汇编代码，比较明显的差别是dpl一个是3，另外两个是0，这是为什么？说明理由。
{{% /hint %}}

{{% hint degree="warning" title="问题" %}}
7、分析`get_free_page()`函数的代码，叙述在主内存中获取一个空闲页的技术路线。
{{% /hint %}}

{{% hint degree="warning" title="问题" %}}
8、`copy_process`函数的参数最后五项是：`long eip, long cs, long eflags, long esp, long ss`。查看栈结构确实有这五个参数，奇怪的是其他参数的压栈代码都能找得到，确找不到这五个参数的压栈代码，反汇编代码中也查不到，请解释原因。详细论证其他所有参数是如何传入的。
{{% /hint %}}

{{% hint degree="warning" title="问题" %}}
9、详细分析Linux操作系统如何设置保护模式的中断机制。
{{% /hint %}}

{{% hint degree="warning" title="问题" %}}
10、分析Linux操作系统如何剥夺用户进程访问内核及其他进程的能力。
{{% /hint %}}

{{% hint degree="warning" title="问题" %}}
11、
```C
_system_call:  
    cmpl $nr_system_calls-1,%eax  
    ja bad_sys_call
```

分析后面两行代码的意义。
{{% /hint %}}

## 第四次作业

{{% hint degree="warning" title="问题" %}}
1、`copy_process`函数的参数最后五项是：`long eip, long cs, long eflags, long esp, long ss`。查看栈结构确实有这五个参数，奇怪的是其他参数的压栈代码都能找得到，确找不到这五个参数的压栈代码，反汇编代码中也查不到，请解释原因。
{{% /hint %}}

{{% hint degree="warning" title="问题" %}}
2、分析`get_free_page()`函数的代码，叙述在主内存中获取一个空闲页的技术路线。
{{% /hint %}}

{{% hint degree="warning" title="问题" %}}
3、分析`copy_page_tables()`函数的代码，叙述父进程如何为子进程复制页表。
{{% /hint %}}

{{% hint degree="warning" title="问题" %}}
4、进程0创建进程1时，为进程1建立了`task_struct`及内核栈，第一个页表，分别位于物理内存两个页。请问，这两个页的位置，究竟占用的是谁的线性地址空间，内核、进程0、进程1、还是没有占用任何线性地址空间？说明理由（可以图示）并给出代码证据。
{{% /hint %}}

{{% hint degree="warning" title="问题" %}}
5、假设：经过一段时间的运行，操作系统中已经有5个进程在运行，且内核为进程4、进程5分别创建了第一个页表，这两个页表在谁的线性地址空间？用图表示这两个页表在线性地址空间和物理地址空间的映射关系。
{{% /hint %}}

{{% hint degree="warning" title="问题" %}}
6、

```C
#define switch_to(n) {\  
struct {long a,b;} __tmp; \  
__asm__("cmpl %%ecx,_current\n\t" \  
    "je 1f\n\t" \  
    "movw %%dx,%1\n\t" \  
    "xchgl %%ecx,_current\n\t" \  
    "ljmp %0\n\t" \  
    "cmpl %%ecx,_last_task_used_math\n\t" \  
    "jne 1f\n\t" \  
    "clts\n" \  
    "1:" \  
    ::"m" (*&__tmp.a),"m" (*&__tmp.b), \  
    "d" (_TSS(n)),"c" ((long) task[n])); \  
}
```

代码中的`ljmp %0\n\t `很奇怪，按理说`jmp`指令跳转到得位置应该是一条指令的地址，可是这行代码却跳到了`m (*&__tmp.a)`，这明明是一个数据的地址，更奇怪的，这行代码竟然能正确执行。请论述其中的道理。
{{% /hint %}}

{{% hint degree="warning" title="问题" %}}
7、进程0开始创建进程1，调用`fork()`，跟踪代码时我们发现，fork代码执行了两次，第一次，执行`fork`代码后，跳过`init()`直接执行了`for(;;) pause()`，第二次执行`fork`代码后，执行了`init()`。奇怪的是，我们在代码中并没有看到向转向`fork`的`goto`语句，也没有看到循环语句，是什么原因导致`fork`反复执行？请说明理由（可以图示），并给出代码证据。
{{% /hint %}}

{{% hint degree="warning" title="问题" %}}
8、详细分析进程调度的全过程。考虑所有可能（`signal`、`alarm`除外）
{{% /hint %}}

{{% hint degree="warning" title="问题" %}}
9、分析`panic`函数的源代码，根据你学过的操作系统知识，完整、准确的判断`panic`函数所起的作用。假如操作系统设计为支持内核进程（始终运行在0特权级的进程），你将如何改进`panic`函数？
{{% /hint %}}