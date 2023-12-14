---
title: 现代操作系统 - 笔记
description: "Notes about Modern OS"
slug: system-bachelor-notes-book-notes-mordenOS
date: 2022-01-01 00:00:00+0000
categories:
    - OS
tags:
    - study
    - OS
    - CS
weight: 1
---

# OS

## 引论

操作系统运行在内核态，对所有硬件具有完全访问权，软件运行在用户态

资源管理：多路复用，空间复用，时间复用

流水线：同时取出多条指令
超标量：有多个执行单元

系统调用，陷入内核调用OS，TRAP指令实现用户态切换至内核态

I/O端口空间
实现输入输出：
1. 系统调用->内核翻译->过程调用，忙等待：设备驱动程序启动I/O并在连续循环中检查设备是否完成工作
2. 启动设备，完成工作后发出一个中断
3. I/O使用特殊的直接存储器访问(DMA)，CPU设置DMA、启动DMA、DMA完成后引发一个中断

启动计算机

BIOS：基本输入输出系统

