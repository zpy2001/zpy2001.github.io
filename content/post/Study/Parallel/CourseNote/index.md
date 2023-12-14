---
title: 并行 - 课程笔记
description: "Course notes about Parallel Computing"
slug: parallel-course-note
date: 2023-06-24 00:00:00+0000
categories:
    - Parallel Computing
tags:
    - study
    - Parallel Computing
    - CS
    - Course Notes
weight: 1
---

# SIMD

运算操作及运算数数据类型一致

SIMD Types:

|features|MMX|SSE|AVX[/2]|AVX512|
|:-:|:-:|:-:|:-:|:-:|
|vector size(bit)|64|128|256|512|
|data types(bit)|8,16,32|int: 8,16,32,64<br>float: 32,64|int: 8,16,32,64<br>float: 32,64|int: 32<br>float: 32,64|

avoid mixing SSE and AVX

intel intrinsics compile:

```C
gcc -march=native
```

icc generate report:

To generate a vectorization report, use the qopt-report-phase=vec compiler options together with qopt-report=1 or qopt-report=2.