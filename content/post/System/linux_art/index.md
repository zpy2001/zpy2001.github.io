---
title: Linux内核设计的艺术 - 笔记
description: "Notes about the art of Linux kernel design"
slug: LinuxArt
date: 2023-10-10 00:00:00+0000
categories:
    - Linux
tags:
    - OS
    - CS
    - Linux
weight: 1
---

# Linux内核设计的艺术

## 启动

实模式：8086/80x86兼容的cpu操作模式，特性是一个20位的存储器地址空间，1MB

加电后入口：0xFFFF0 - BIOS

BIOS程序在内存最开始的位置(0x0000)用1KB的内存空间(0x0000~ 0x003FF)构建中断向量表，在紧挨着它的位置用256字节的内存空间构建BIOS数据区(0x00400~0x004FF)，并在大约57KB以后的位置(OxOE05B)加载了8KB左右的与中断向量表相应的若干中断服务程序

Linux 0.11，分三批加载内核代码，第一批由BIOS中断`int 0x19`将第一扇区bootsect加载到内存，而后第二、三批由bootsect指挥加载4/240个扇区至内存

`int 0x19`指向`0x0E6F2`，加载bootsect扇区的中断服务程序入口地址

1. 加载bootsect
2. 加载setup
	1. bootsect对内存规划
	2. bootsect将自身从0x7c00复制到0x90000
	3. 将setup程序加载到内存
		1. 借助BIOS提供(bootsect执行)的`int 0x13`中断向量指向的中断服务程序完成
3. 加载system模块
	1. 仍然是`int 0x13`中断
4. 设置根设备
5. `jmpi 0 SETUPSEG`跳转到0x90200执行setup程序
6. setup程序利用BIOS中断服务程序从设备上提取内核运行所需的机器系统数据，包括光标位置、显示页面等，加载的部分数据会覆盖bootsect程序，而且不能被覆盖，0x90000~0x901fd，510字节
7. 向32为模式转变（实模式$\rightarrow$保护模式）
	1. 关中断(EFLAGS IF位clear，命令`cli`，开中断`sti`)，将位于0x10000的system代码移动到内存起始位置0x0，覆盖BIOS中断向量表和BIOS数据区
	2. 设置中断描述符表(IDT, IDTR)和全局描述符表(GDT, GDTR)，GDT是系统中唯一存放段描述符的数组，为所有进程的总目录表，存放每个任务LDT和TSS地址，GDT可位于任意位置，地址在GDTR中。此时内核尚未运行，GDT第一项为空，第二/三项位内核代码/数据段描述符，而IDT是空表。两步进行：
		1. 设计内核代码时，将表写好，数据填写完成
		2. 将IDTR/GDTR指向表
	3. 打开A20`mov al,#0xdf`，实现32位寻址，最大寻址空间4GB，实模式下0~0xfffff，1MB寻址空间，需要20根地址线，保护模式后使用32根地址线，因此选通21(A20)~32根地址线

16/32位中断机制对比：16位使用位于0x0的中断向量表，32位使用中断描述符表，位置不固定

