---
title: Linux内核设计的艺术 - 问题
description: "Questions about the art of Linux kernel design"
slug: system-IA32-notes-linux-art-questions
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