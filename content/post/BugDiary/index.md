---
title: 踩坑日记
description: "Bug Diary"
slug: Bug-Diary
date: 2023-10-24 00:00:00+0000
categories:
    - Bug
tags:
    - Bug
    - CS
weight: 1
---

# Bug Diary

## Windows

## Mac

### X11

mac上使用X11与Linux通信，需要：

1. 安装运行[XQuartz](https://www.xquartz.org)，设置 $\rightarrow$ 安全性 $\rightarrow$ 勾选允许从网络客户端链接
2. **运行`xhost +`**，允许所有链接
3. 使用-X连接remote或者在`~/.ssh/config`中`Host`下加入`ForwardX11 yes`
4. remote使用`DISPLAY`环境变量：`export DISPLAY=${HOSTNAME}:0`

## Linux