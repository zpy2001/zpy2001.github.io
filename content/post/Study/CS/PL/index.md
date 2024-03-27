---
title: 并行处理
description: 国科大谭光明老师开设的并行处理课程
slug: Parallel-Processing
date: 2024-02-27
categories:
  - Parallel
tags:
  - CS
  - PL
  - class
weight: 1
---
# 并行处理

## Lecture 02: Parallelism and Locality
**并行的局部性**
- 峰值性能$$P_{chip}=n_{core} . n^{FP}_{super}.n_{FMA}.n_{simd}.f$$
### Part I: Parallel Execution
#### Exploit Parallelism
- ILP SISD
- SIMD
- MIMD SPMD

## Lecture 03

### 提纲
- 应如何描述应用和系统的性能要求
- 什么是用户对性能和成本要求
- 如何测量应用程序的性能
- 当并行计算机上执行并行程序时，应如何描述系统性能特征
- 应如何量化和分析系统可扩展性
（锐评图形学，每个文章都挺好，跑花花公子的图片很漂亮，但没有benchmark，不知道做到什么程度了）
### 系统和应用的基准程序
#### 基准程序的定义
- 基准程序组(benchmark suite)是一套基准程序和控制测试条件及过程的一组特定规则，包括输入数据、输出结果以及性能指标
#### 基准程序的分类
- 按应用类型:科学计算、人工智能、商业应用、网络服务、多媒体应用、信号处理等
- 按复杂程度:宏基准程序和微基准程序
##### 代表性的微和宏基准程序组

| 类型        | 名称                   | 测量           |
| --------- | -------------------- | ------------ |
| **微基准程序** | Linpack、HPCG         | 数值计算(稠密、稀疏)  |
|           | Graph500             | 非数值计算(图计算)   |
|           | AIPerf               | 人工智能         |
|           | Lmbench              | 系统调用和数据移动    |
|           | Stream               | 存储器带宽        |
|           | OSU                  | 大规模网络消息传递    |
| **宏基准程序** | NAS NPB              | 并行计算(CFD)    |
|           | ECP Proxy Apps Suite | 超算代理应用       |
|           | SPEC                 | CPU混合基准程序系列  |
|           | TPC                  | 事务处理和数据库商业应用 |
###### Linpack
- 采用LU分解的高斯消元法求解大型稠密线性方程组
- 成为评测高性能计算机系统性能的**事实标准**
###### HPCG
- [HPCG Benchmark](https://hpcg-benchmark.org/)
- 共轭梯度法求解大型系数矩阵方程组（稀疏矩阵）
- 这类方程源自非定常数非线性偏微分方程组，迭代求解过程中需要频繁地存取不规则数据，因此 HPCG 对计算机系统要求高带宽、低延时、高 CPU (核)主频

### 可扩展性和加速比分析
#### Amdahl定律
- 设一个问题的固定工作负载为W，假设此工作负载W可分为两部分$$W = \alpha W+ (1-\alpha)W$$
  则有加速比为$$S_n = \frac{W}{\alpha W +(1-\alpha)W\frac{W}{n}}=\frac{n}{1+(n-1)\alpha}$$
- 扩展Amdahl定律
### How to achieve the peal
https://github.com/PAA-NCIC/PE.git


## Lecture 04
#### Introduction: why data access is so important
#### Introduction: conceptual goals of the study
#### Introduction: pragmatic goals of the study
 - **Latency**: how many cycles it takes to get data from main memory and caches
 - **Bandwidth**: how much data CPU can bring from main memory and caches
 - what does “memory bandwidth" we see in a processor spec sheet really mean? e.g.,
	 - The processor data sheet of E5-2698(68GB/s)
	 - In general:8 bytes * DDR frequency * memory channel, per CPU socket
	 - The “big” CPU (Skylake-X Gold 6130)
### **Memory bound algorithms**
### What does memory performance imply for FLOPS?
### Memory-bound algorithms (applications)
- There are two obvious lower bounds on the time to complete the algorithm
	$T \geq \frac{C}{the\ peak\ FLOAPS}$
	$T \geq \frac{N}{the\ peak\ mem\ band}$

