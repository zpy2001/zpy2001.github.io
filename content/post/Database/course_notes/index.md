---
title: 数据库 - 课程笔记
description: Course notes about Database
slug: database-course
date: 2023-09-22 00:00:00+0000
categories:
    - Database
tags:
    - Database
    - CS
    - Course Notes
weight: 1
---

# 数据库系统

## 关系模型和SQL

### 关系代数

selection:

从表中选择一些行

$\sigma_{condition}(table_name)$

projection

从表中选择一些列

$\pi_{colum\_list}(table_name)$

join:

$R\bowtie_{R.a = S.b}S$

重命名：$\rho$

除：

A/B,假设A(x,y), B(y)，则$A/B=\{x|\forall y \in B, (x, y) \in A\}$

分组运算：$\gamma_{group, form[\rightarrow rename\_col]}(table)$


授权图：

(授权者，被授权者，权限，yes/no)

## 数据冗余与依赖关系

### 范式

每种范式消除了一种冗余

> 1NF:所有属性（列）都是原子类型
>> 2NF:消除函数依赖中的部分依赖
>>> 3NF:消除函数依赖中的非键传递依赖
>>>> BNF:消除所有函数依赖
>>>>> 4NF:消除多值依赖
>>>>>> 5NF:消除连接依赖

函数依赖(FD)

$R(U)$为属性集$U$上的一个关系模式，$X\subseteq U, Y\subseteq U$

函数依赖$X\rightarrow Y$：任意两个元组在X上取值相同，则他们在Y上取值也相同

候选键：一种特殊的函数依赖

$X\subseteq U$为候选键，若满足：

1. $X\rightarrow U$
2. $\forall Y\subseteq X, Y\nrightarrow U$(X为最小的)

超键只满足第一个条件

函数依赖的推导：

Armstrong规则：

- 自反律：若$Y\subseteq X$则$X\rightarrow Y$
- 增补律：若$X\rightarrow Y$，则对任意$Z, XZ\rightarrow YZ$
- 传递律：若$X\rightarrow Y$且$Y\rightarrow Z$，则$X\rightarrow Z$

- 分解：若$X\rightarrow YZ$，则$X\rightarrow Y, X\rightarrow Z$
    - 证明：$X\rightarrow YZ, YZ \rightarrow Y, X\rightarrow Y$
- 合并：若$X\rightarrow Y, X\rightarrow Z$，则$X\rightarrow YZ$
    - 证明：$X\rightarrow Z, X\rightarrow XZ, X\rightarrow Y, XZ\rightarrow YZ, X\rightarrow YZ$

分解和合并等价，因此只需找出但一属性的函数依赖关系则可得所有，但列举自反律、增补律无意义

求必报重点是传递律，创建条件可使用另两

函数依赖F的闭包

属性集的闭包算法：

```
closure  = X;
do{
    changed = false;
    if(exists(Y\rightarrow Z\in F)) and (Y\subseteq closure) and !(Z\subseteq closure){
        closure = closure\bigcup Z;
        changed = truel
    }
}while(changed);
```

属性集X$\rightarrow$单个属性A

1. 类型1:平凡依赖：${A}\subseteq X$，无数据冗余

X, A与键的关系：

X: 超键；为候选键的一部分；包含非键的属性，且X不是超键

A: 非键；键

2. 类型2：X为超键（包括候选键），无数据冗余

3. 部分依赖：X为候选键的一部分，A非键
    - 2NF

4. 非键传递依赖：X包含非键的属性，X不是超键，A为非键属性
    - 3NF

5. 对键属性的函数依赖：X不为超键，A为键属性
    - BNF

判断关系模式是否满足xNF:

1. 找出所有的函数依赖关系
2. 查看是否包含：
    - 部分依赖：
        - Y: 1NF break, N: 2NF continue
    - 非键传递依赖：
        - Y, break, N: 3NF continue
    - 键属性函数依赖
        - N: BNF

### 关系模式分解

无损分解：对R进行X和Y的分解是无损的，当且仅当：

- 分解后两个表的自然连接可以得到原表

可证：若$R1\bigcup R2=R$且$R1\bigcap R2\rightarrow R1$，则对R进行R1和R2的分解无损

$R1\bigcap R2$为join key，因此为R1的主键，R2的外键

BCNF通过关系模式的无损分解

若$X\rightarrow A$违反：

- $X\subset R$
- $A\in R$为单独属性
- $A\notin X$，不是平凡依赖

对R进行XA和R-A的分解

反复执行

## 多值依赖

$X\rightarrow \rightarrow Y$

$Z=R-XY$，X相同的情况下，Y和Z的全组合都出现

函数依赖是特例：X取一个值的时候，Y有唯一的值，Y和Z的组合全出现

R满足4NF，若对每一个R上的$X\rightarrow \rightarrow Y$下面条件有一个成立：

- $Y\subseteq X$
- $XY = R$
- X为一个超键

若R为BCNF，且存在一个单属性的候选键，则R为4NF

## 连接依赖

无损连接分解为连接了依赖

5NF：所有$Ri=R$或连接依赖为一组函数依赖所蕴含，函数依赖左侧都是候选键

R为3NF且每个候选键都是单属性，则也为5NF

## 数据存储于访问路径

索引：给定一个键，找到对应的值

### 树结构索引

B+树：每个节点可以有多个孩子，且永远是平衡的，每个节点是一个page，所有key存储在叶子节点，内部节点完全是索引作用

所有叶子逻辑上组成一个数组，叶节点从左至右从小大大排序，使用sibling pointer连起来

查找直接二分，代价：

- 总共N个key
- 每个节点child/pointer个数为B
- 树高$O(\log_{B}N)$
- 总比较次数
    - 每个节点内部二分查找：$O(\log_{2}B)
    - $O(\log_{B}N) \times O(\log_{2}B) = O(\log_{2}N)$
- 总I/O次数 = $O(\log_{B}N)$s

Insertion

Search而后在节点插入：

- 叶节点未满，插入叶节点
- 叶节点满了，节点分裂
    - 分裂算法：
    - 原始节点叶子数量为l
    - 原始节点更新后有$\lceil (l+1)/2\rceil$项
    - 新节点有$\lfloor (l+1)/2\rfloor$项
    - 将第$\lceil (l+1)/2\rceil$键移动到父节点，将新节点插入父节点
    - 重复知道无需拆分

Deletion

节点为空进行merger，或者完全不merge

### 哈希索引

Extendible Hash

采用前n位作为hash键，若对应的表项满，则采用更多的位数作为键，区分度更高

### 其他索引

#### ISAM：排序文件+多层索引

类似B+树在计算机数据文件上的应用，但插入操作与B+树不同，上面索引不变，而是在数据部分增加溢出页

#### 位图索引

横排：相同key具有多个记录

- 可在树结构或哈希表key对应的位置放不同的项，或在树结构、哈希表中允许多项具有相同的key
- 通过指针指向其余区域的间接层，间接层中存放记录
- 位图索引，简介层记录位图，key对应bitmap，bitmap每位对应一条记录，01表示取值相同与否

当1个数少时压缩位图，使用距离表示1间隔

#### 倒排索引

每条文本进行标号，每个单词记录含有此单词的文本序号

### 多维索引

单维索引不同维度的重要性是不同的，多维索引相似

Z-order：多维数据放入一维

#### 网格文件

- 空间切成一个个长方形网格
- 每个网格对应一个桶，存储在数据页中
- 用数据结构记录桶的位置

桶矩阵：横坐标是横向划分的数据区间，纵坐标是纵向划分的数据区间

部分匹配，范围查询，最近邻查询

插入时划分网格，要使改变前后网格中数据分布都很均匀

#### 分段散列

对每一维属性key进行哈希值计算，最终hash值为它们的拼接

无法有效支持不等值比较

二维上还有

#### 四叉树

#### R-Tree

每个树节点代表一个区域：MBR，是包含子树中所有对象的最小外接矩形，可能有重叠区域，希望最小

插入：

- 对每个内部节点插入时考虑
    - 选择MBR面积增大最小的孩子
    - 当面积增加相同时考虑面积最小的
    - 进入孩子节点，重复

有空位插入，否则split

## 查询处理

系统目录

存储数据的元信息：

- 表
    - 表名
    - 表存储方式
    - 属性名、类名
    - 索引名
    - 完整性约束
    - 统计信息
        - 记录数
        - 表大小：页数
        - 属性值分布特征

- 索引
    - 名称
    - 结构类型
    - key组成属性
    - 统计信息
        - key个数
        - 索引大小：页数
        - 树结构索引高度
        - 索引范围

- 视图
    - 名称
    - 定义

查询计划：Query Optimizer产生，Execution Engine按照plan执行，最终表现为一颗Operator Tree

每个节点代表一个运算，运算的输入来自孩子节点，输出送往父节点

### 处理方式

#### operator at a time

完整执行每个operator

每个operator一个函数

中间结果表的代价大

#### tuple at a time

每个operator设计成为具有统一的函数接口: open, getnext, close

主程序循环调用根的getnext，执行方式是从上至下的pull方式，父节点调用子节点getnext生成一条条结果

两类operator：

- non-blocking
- blocking: 需要预处理一遍数据

缺点是代码执行路径很长

getnext返回组记录，循环集中处理

#### 多线程流水线

push方式，operator tree划分为多个子树，每个线程负责一个子树，线程之间传递中间结果

一个线程等待孩子节点的结果来到后进行计算，产生结果，发送给父亲节点，push

### 选择

#### 无索引、未排序

顺序访问每个页和记录

IO代价$M_{R}$，即数据页数量

#### 无索引、排序

二分查找

$M_{R}$数据页数量，满足条件的记录占据D个数据页

总代价：$\log_{2}M_{R} + D$

#### $B+ Tree

- 聚簇索引：叶子节点就是数据页。对表首先利用主键组织到一颗B+树中，而后将辅助索引键组织到B+树。使用辅助键索引需要2次B+树查找

- 非聚簇索引：叶子节点存RecordID，随机读取，辅助键和主键直接索引到数据中

假设B+ Tree的查找代价为H次IO，符合条件索引项有m个叶子节点页，指向m个记录

总代价：H+L+m

#### 哈希索引

哈希表的平均查找代价为H次IO，符合条件的记录有m个

总代价：H+m

#### 多个选择条件

- 文件扫描是最通用的方法，对每个记录求解任意复杂的选择条件
- 先求解一个合取条件，转换为合取范式
- 使用位图索引，获得每个选择条件的位图，而后计算总位图，按位操作
- 利用多个索引，获取每个字条件RecordID集合，对集合操作

### 投影

- 行式数据库，在选择的基础上，对选中的记录提取指定的列
- 列式数据库，不同的列存放在不同的文件，投影即为选择不同的文件，需要把同一记录的多个列从多个文件中读取组装

### 排序

#### 内存排序

- $O(n^{2})$

冒泡，选择，插入

- $O(n\log N)$

快排，归并，堆

- $O(n)$

基数

#### 外存排序

归并

