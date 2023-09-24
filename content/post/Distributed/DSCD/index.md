---
title: 分布式系统概念与设计 - 笔记 [TODO]
description: "Notes about Distributed System: Concept and Design"
slug: distributed-system-concept-design
date: 2023-10-07 00:00:00+0000
categories:
    - Distributed System
tags:
    - Distributed System
    - CS
weight: 1
---

# 分布式系统概念与设计

## 特征

### Feature

- 并发
- 缺乏全局时钟：交换消息以协作
- 故障独立性

### Challenges

- 异构性：网络、硬件、操作系统、编程语言、开发者；sol: 中间件
- 开放性：发布关键接口，基于一致的通信机制与接口访问共享资源，兼容不同异构软硬件
- 安全性
- 可伸缩性：用户数量与资源数量激增保持有效
- 故障处理
- 并发
- 透明性，访问（本地、远程相同）、位置、复制、故障、移动、性能、伸缩

## 系统模型

### 体系结构模型

- 通信实体：通信末端为结点或线程
- 通信范型：进程间通信、远程调用、间接通信
	- 远程调用：请求应答协议，client-server；远程过程调用(RPC)
	- 间接通信：组通信，发布-订阅，消息队列，元组空间，分布式共享内存(DSM)


## 远程调用

RPC

- 接口编程
	- 接口定义语言(IDL)允许以不同语言实现过程以便相互调用