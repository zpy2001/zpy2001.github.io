---
title: IA32 - 笔记
description: "Notes about IA32"
slug: IA32
date: 2023-09-27 00:00:00+0000
categories:
    - OS
tags:
    - OS
    - CS
weight: 1
---

# IA-32 Manual

## Architecture

IA-32处理器在加电或者复位后首先进入实地址模式，而后由软件完成到保护模式下的切换。

寄存器与数据结构：

![registers](photos/registers.png)

### 全局和局部描述符表

保护模式下，所有内存访问通过：

- 全局描述符表(GDT)
- 局部描述符表(LDT)

描述符表里的项为段描述符(riscv: PTE)，段描述符包含

- 一个段的基地址
- 访问权限
- 类型和用法信息

每个段描述符都有一个与之相关的段选择子(riscv: satp)。段选择子包含：

- 一个对GDT或LDT的索引
- 一个全局/局部标志
- 访问权限信息

访问段中的一个字节，必须同时提供一个段选择子和一个偏移，段选择子提供对段描述符的访问，处理器从段描述符中获取段在线性地址空间里的地址，而后加上偏移。根据处理器当前特权级(CPL)决定能否访问段。

### 系统段、段描述符和门

进程运行环境：

- 代码
- 数据
- 堆栈段
- 任务状态段(TSS)
- LDT

GDT不通过段选择子和段描述符访问

IA-32门：特殊描述符（意思是入口）

- 调用门：访问与当前代码段特权级更高或相同的代码段，若调用需要切换特权级，处理器就切换到对应特权级的堆栈（新堆栈的段选择子从当前任务的TSS中获得）
- 中断门
- 陷阱门
- 任务门

提供高于用户特权级的系统过程/处理程序来进行保护性访问

### 任务状态段和任务门

TSS(riscv: Context)定义任务执行环境的状态，包含：

- 通用寄存器
- 段寄存器
- EFLAGS
- EIP
- 段选择子
- 三个堆栈段（三个特全级各一个）指针
- LDT选择子
- 页表基地址

### 中断和异常处理

外部、软件中断和异常通过中断描述符表(IDT)处理，包含访问中断和异常处理程序门描述符的集合

IDT和GDT不是段，IDT线性基地址包含在IDT寄存器(IDTR)中

IDT中描述符可以是中断门、陷阱门或任务门，处理器从int/bound指令收到一个中断向量，才去访问中断或异常处理程序。中断向量是IDT中门描述符的索引。任务门：任务切换访问处理程序

### 内存管理

支持物理寻址也支持虚拟分页

页目录基地址保存在CR3

分页只有2级：页目录$\rightarrow$页表$\rightarrow$

### 系统寄存器

- EFLAGS寄存器：
	- 系统标志和IOPL域控制任务和模式的切换、中断处理、指令跟踪和访问权限
- 控制寄存器
- 调试寄存器
- (G/L/I)DTR寄存器包含表的线性地址和限长
- 任务寄存器包含当前任务TSS的线性地址和大小
- 模型相关寄存器

### 运行模式

- 保护模式
- 实模式
- 系统管理态(SMM)
- 虚拟8086模式

上电后位于实模式，由控制寄存器CR0中的PE标志决定

### EFLAGS

![EFLAGS](photos/eflags.png)

### 内存管理寄存器

![mmreg](photos/mmreg.png)

限长以字节为单位

`lgdt`与`sgdt`分别装载和保存GDTR寄存器

### 控制寄存器

- CR0: 包含系统控制标志
- CR2: 包含页故障线性地址
- CR3: 页目录基地址寄存器(PDBR)，包含页目录表的物理基址和两个标志(PCD和PWT)
- CR4: 包含一组标志，控制扩展启用

![CR](photos/CR.png)

CR0:

- PG (page): CR0[31]，分页
- CD (cache disable): CR0[30]，为1 disable cache
- NW (no-direct write): CR0[29]，写操作写回/写穿透
- AM (align mask): CR0[18]，对齐屏蔽
- WP (write protection): CR0[16]，管理程序能否写用户级的只读页；fork时COW依赖
- NE (number error)：CR0[5]，浮点错误
- PE (protection enable): CR0[0]，启用保护模式，启用段级保护

CR3:

- PCD (page cache disable): CR3[4]，禁用页级高速缓存
- PWD (page write transparent): CR3[3]，控制页目录表的写回/写穿透

CR4:

- VME
- PVI (protection-mode virtual interrupt)
- TSD (timestamp disable): RDTSC指令
- DE (debug extension)
- PSE (page size extension): 4K->4M
- PAE (physical address extension): 32->36位物理寻址
- MCE (machine check enable)
- PGE (page global enable)
- PCE (performance counter enable)

CR4的位能够通过CPUID指令获取

## 保护模式内存管理

分段隔离每个进程，分段功能不可关闭

只有逻辑地址（远指针）确定字节在特定段中的位置

逻辑地址：段选择子+偏移

![seg_page](photos/seg_page.png)

保护模式，物理地址空间4GB：0~0xffffffff

![address_translation](photos/address_translation.ong)

逻辑地址中段选择子在段寄存器中，一个程序内所有访存共享；用户只提供偏移

![segment_selector](photos/segment_selector.png)

索引[3:15]：选择GDT或LDT中8192个描述符中的某个，处理器将索引值*8 + GDT/LDT base

GDT第一项无效，为空段选择子，装载至段寄存器时无异常，但用来访存时产生异常，但装载CS或SS产生一般保护异常(#GP)

### 段寄存器

- CS: 代码
- DS, ES, FS, GS: 数据
- SS: 堆栈

段寄存器有可见、不可见部分，段选择子可见，不可见称为描述符高速缓存/影子寄存器