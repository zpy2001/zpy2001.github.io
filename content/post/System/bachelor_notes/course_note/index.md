---
title: 操作系统 - 课程笔记
description: "Course notes about OS"
slug: OS-Course
date: 2022-01-01 00:00:00+0000
categories:
    - OS
tags:
    - OS
    - CS
    - Course Notes
weight: 1
---

# Course Note

## 进程与线程

### 进程

- 进程是指一个具有一定独立功能的程序在一个数据集合上的一次动态执行过程
- 进程地址空间刻画了一个程序运行所需要的资源

|进程地址空间|
|:-:|
|段表|
|共享库|
|栈|
|$\downarrow$|
|$\uparrow$|
|堆|
|初始化数据|
|代码|

### 进程控制块(PCB)

包含信息：

- CPU相关进程管理信息
	- 状态
		- 就续态
		- 正在运行
		- 阻塞态
	- 寄存器，EFLAGS
- 内存管理信息
	- 栈，代码段和数据段
	- 段，页表 
- I/O和文件管理
	- 通信端口，目录，文件描述符

进程原语(Primitives)

原语：一般是指由若干条指令组成的程序段，用来实现某个特定功能，在执行过程中不可被中断

- 创建和终止
	- `exec, fork, wait, kill`
- 信号(?)
- 操作
	- 阻塞，放弃CPU控制权(yield)
- 同步

- 创建进程 = 创建与初始化PCB
	– 将数据和代码加载至内存
	– 创建一个空的调用栈
	– 初始化进程的状态
	– 把进程标志为就绪态
- 克隆 = 复制与修改PCB
	– 停止当前进程，并保存其状态
	– 备份当前代码和数据，代码，栈和OS的状态
	– 把进程标志为就绪态

UNIX:
– fork克隆出一个进程
– exec覆盖掉当前进程

进程操作：
- 创建
	- when:系统初始化、用户请求、当前进程执行系统调用 
- 执行：从就绪态选择，分时间片
- 等待（阻塞）：主动
	- 请求并等待系统服务
	- 启动某种操作，无法马上完成
	- 需要的数据未到达
- 抢占（抢占式）
	- 高优先级进程就绪
	- 时间片用完
- 唤醒（抢占式）
- 结束
	- 正常主动
	- 错误主动
	- 错误强制
	- 被kill，强制

进程上下文切换

保存上下文：

- 寄存器，协处理器

### 线程

线程是进程的一部分，描述指令流执行状态，是CPU调度的基本单位

线程在同一个进程的地址空间内，可共享变量

线程控制块(TCB)

线程API
- 创建
	- fork(pthread_create), join
- 互斥
	- acquire(上锁), release(解锁)
- 条件变量
	- wait, signal, broadcast
- 警报
	- alert, alertwait, testalert

线程上下文切换可能除法进程上下文切换

过程调用中调用者或被调用者保存部分上下文

- 线程可能乱序恢复
	- 无法用栈保存状态，每个线程有自己的栈
- 线程切换不会太频繁
- 线程可以异步
- 多线程并行，过程顺序执行

- 用户线程：通过用户级线程库函数管理
- 内核线程：内核系统调用，完成线程创建终止管理
- 轻量级进程：内核支持用户线程

#### 非抢占式线程

内核拥有自己的地址空间，并与所有进程共享

内核包含
- 引导加载程序
- BIOS
- 核心驱动
- 线程
- 调度器
	- 使用就绪队列存放所有就绪线程
	- 在相同地址空间内调度(thread)
	- 在新的地址空间内进行调度(process)

- 用户级线程：
  - 用户级线程库实现线程上下文切换
  - 时钟中断会引入抢占
  - 当用户级线程被I/O事件阻塞时，整个进程都会被阻塞
- 内核级线程
	- 内核级线程被内核调度器调度
	- 内核级线程的上下文切换开销远大于用户级（跨越保护边界）

### 调度

#### 调度算法

- when：

- 进程/线程创建
- 进程/线程退出
- I/O阻塞、同步
- I/O中断
- 时钟中断

##### FCFS 先到先服务

一直运行，直到：
- 结束
- 阻塞
- 主动放弃CPU
- 非抢占式

##### STCF/SRTCF

最短时间优先：非抢占
最短剩余时间优先：抢占

##### 时间片轮转RR, Round Robin

增加时间片，结束时调度器按FCFS算法切换到下一个就绪进程

抢占式

注意时间片时间很短，可以看作当第一个任务完成的时候所有任务都运行了相同的时间

##### 虚拟轮转算法

引入辅助队列FIFO

等待I/O的任务完成后会进入辅助队列以被调度（I/O密集型）

引入优先级，辅助队列比就绪队列优先级高

##### 多级队列

将就绪队列分为多个独立的子队列，每个队列有自己的调度算法

前台RR，后台FCFS

每个队列分配一个优先级与相应时间片
队列间按照时间片调度

##### 多级反馈队列(MLFQ)

进程可在不同队列间移动的多级队列算法

- 时间片大小岁优先级别增加而增加
- 进程在当前时间片没有完成，则降到下一个优先级
- CPU密集型进程优先级下降很快，I/O密集型进程停留在高优先级

##### 彩票调度

- 给每个作业一定数量的彩票
- 随机抽取彩票
- 给短作业更多的彩票
- 每个作业都要拥有彩票
- 相互合作的进程可以交换彩票

##### 公平共享调度(FSS)

FSS控制用户对系统资源的访问

- 保证不重要的用户组无法垄断资源
- 未使用的资源按比例分配
- 没有达到资源利用率的组获得更高优先级

### 实时调度

只有当系统保证所有进程实时性前提下，新的实时进程被接纳

若满足：$\sum\frac{C_{i}}{T_{i}} \leq 1$，则作业可调度，其中$C_{i}$为计算时间，$T_{i}$为周期

## 同步与通信

临界区：进程中访问临界资源的一段需要互斥执行的代码

进入临界区：
- 检查可否进入临界区的一段代码
- 可进入，设置正在访问标志

退出临界区：
- 拔出标志

原子操作：不存在任何中断致使失败的操作，或成功或未执行

临界区操作必须为原子操作

### 临界区保障

#### 软件

线程通过共有变量同步行为

Peterson

共享变量：
```C
int turn;	// 表示谁进入临界区
bool flag[];	//表示进程是否准备进入临界区`
```

进入区代码：
```C
flag[i] = true;
turn = j;
while (flag[j] && turn == j){
	;
}
```

退出区代码：
```C
flag[i] = false;
```

#### 硬件

##### 屏蔽中断（上下文切换）实现互斥

获取锁：
```C
Acquire(lock){
	disable interrupts;
	while(lock.value == BUSY){
		enqueue(q, TCB);
		Yield();
	}
	lock.value = BUSY;
	enable interrupts;
}
```

释放锁：
```C
Release(lock){
	disable interrupts; 
	if(!is_empty(queue)){
		dequeue(q, thread T);
		wakeup(T);
	}
	else{
		lock.value = FREE;
	}
	enable interrupts;
}
```
##### 原子操作

- 测试和置位(test and set)
	- 从内存单元中读取值
	- 测试改值是否为1，返回真或假
	- 内存单元值设置为1

```C
bool TestAndSet(bool *target){
	bool rv = *target;
	*target = true;
	return rv;
}
```

锁：

```java
class Lock{
	int value = 0;
	waitQueue q;
}

lock::Acquire(){
	while(test_and_set(value)) //spin or
	{
		enqueue(q, TCB);
		do_schedule();
	}
}

Lock::Release(){
	value = 0;
	dequeue(q, thread t);
	wakeup(t);
}
```

- 交换指令：交换寄存器与内存

`Fetch_and_add/Fetch_and_op`
`Load_linked/Conditional_Store`(LL/SC)
store时检查load linked是否被修改，若有，重load

```C
void Enchange(bool *a, bool *b){
	bool tmp = *a;
	*a = *b;
	*b = tmp;
}
```

- 软件方法
	– 实现复杂
- 中断
	– 实现之后，只能用于单核处理器
- 原子操作指令与锁
	– 大多数时间在用户层自旋
	– 线程数比处理器数目多

## 死锁

认为进程线程等价

- 资源：请求、使用、释放
	- 可抢占：CPU
	- 不可抢占：磁盘，文件，互斥锁

饥饿：进程无限等待
死锁：若进程集合中所有进程都在等待其余进程退出，但是所有进程不提前退出

若A在持有R的时候请求S，B在持有S的时候请求R

死锁的条件：

互斥：所有资源都被分配给恰好一个进程
占有和等待：持有资源的进程可以请求新资源
不可抢占：资源不可被夺走
环路等待：如上例描述

银行家算法，调度方案，使所有进程能够完成

- 单个资源
	- 每个进程有个贷款额度
	- 总的资源可能不能满足所有贷款额度
	- 跟着分配的和仍然需要的资源
	- 分配时检查安全性

数据结构：

- `n`线程数量，`m`资源类型数量
- `available`剩余空闲量：长度为`m`的向量
- `allocaion`已分配量`n*m`矩阵
- `need`未来需要量：`n*m`矩阵

算法：
```
初始化: 
Request[i] 线程T[i]的资源请求向量
Request[i][j] 线程T[i]请求资源R[j]的实例

循环:依次处理线程T[i], i=0, 1, 2, …
1. 如果 Request[i] <= Need[i], 转到步骤2。
否则, 拒绝资源申请, 因为线程已经超过了其最大要求
2. 如果 Request[i] <= Available, 转到步骤3。
否则, T[i] 必须等待, 因为资源不可用
3. 通过安全状态判断来确定是否分配资源给T[i]:
生成一个需要判断状态是否安全的资源分配环境
	Available = Available -Request[i];
	Allocation[i]= Allocation[i] + Request[i];
	Need[i]= Need[i]–Request[i];
调用安全状态判断
	如果返回结果是安全，将资源分配给T[i]
	如果返回结果是不安全，系统会拒绝T[i]的资源请求
```

安全状态判断

两阶段加锁

试图对所有所需的资源进行加锁
- 若成功则使用资源，而后释放资源；
- 否则释放所有资源，从头开始

## 信号量

Semaphores

一个整形变量，表示系统资源数量

```java
class Semaphore{
	int sem;
	waitqueue q;
}
```

两个原子操作：

等待信号量为正后信号量-1：P(down/wait)

```C
P(s){
	while(s <= 0){
		;
	}
	s--;
}
```

```java
Semaphore::P(){
	sem--;
	if(sem < 0){
		enqueue(q, thread t);
		block(t);
	}
}
```

信号量+1：V(up/signal)

```C
V(s){
	s++;
}
```

```java
Semaphore::V(){
	sem++;
	if(sem<=0){
		dequeue(q, t);
		wakeup(t);
	}
}
```

有界缓冲区

一个或多个生产者生成数据后放在一个缓冲区中，但个消费者从缓冲区去除数据处理

- 互斥访问：任何时刻都只能有一个生产者或消费者访问缓冲区
- 条件同步：缓冲区空时，消费者必须等待生产者；缓冲区满时反向等待

信号量实现生产者-消费者问题

```java
class BoundedBuffer{
	mutex = new Semaphore(1);
	emptyBuffer = new Semaphore(0);
	fullBuffer = new Semaphore(n);
}

BoundedBuffer::Deposit(c){
	fullBuffer->P();
	mutex->P();
	add c to buffer;
	mutex->V();
	emptyBuffer->V();
}

BoundedBuffer::Remove(c){
	emptyBuffer->P();
	mutex->P();
	remove c from buffer;
	mutex->V();
	fullBuffer->V();
}
```

## 管程

管程是一种用于多线程互斥访问共享资源的程序结构

- 任何时刻最多只有一个线程执行管程代码
- 正在管程中的线程可以临时放弃管程的互斥访问，等待事件出现时恢复

管程组成：

一个锁：控制管程代码互斥访问
0或者多个条件变量：控制管理数据的并发访问

条件变量是管程内的等待机制
每个条件变量表示一种等待原因，对应一个等待队列

- `wait()`操作
	- 将自己阻塞在等待队列
	- 唤醒一个等待者或者释放管程的互斥操作
- `signal()`操作
	- 将等待队列中的一个线程唤醒
	- 若等待队列为空，则等同空操作

### 条件变量实现

```java
class Condition{
	int numWaiting = 0;
	waitqueue q;
}

Condition::Wait(lock){
	numWaiting++;
	enqueue(q, thread t);
	release(lock);
	do_schedule();	// need mutex
	acquire(lock);
}
// Mesa风格管程：
// 原子解锁mutex并加入condition对应的等待队列
// 被唤醒时重新锁定mutex

Condition::Signal(){
	if(numWaiting > 0){
		dequeue(q, thread t);
		wakeup(t);
		numWaiting--;
	}
}
// Mesa风格管程：
// 当没有线程阻塞于该条件变量时，do nothing
// 如果有被阻塞的线程，至少唤醒一个
```

等待资源：

```
Acquire(mutex);
while(没有资源){
	wait(mutex,cond);
}
使用资源
release(mutex);
```

释放资源：

```
Acquire(mutex);
使资源可用
Signal(cond);
release(mutex);
```

管程实现生产者消费者问题：

```java
class BoundedBuffer{
	Lock lock;
	int count = 0;	// 利用共享变量确定缓冲区资源数
	Condition notFull, notEmpty;
}

BoundedBuffer::Deposit(c){
	lock->Acquire();
	// Mesa: while(count == n)
	while(count == n){
		notFull.wait(&lock);	// 当缓冲区满时将线程enqueue，注意锁先释放，调度后立刻获取
	}
	add c to the buffer;
	count++;
	/* Mesa: 
	if(count == 1){
		Signal(empty);
   }*/
	notEmpty.Signal();
	lock->release();
}

BoundedBuffer::Remove(c){
	lock->Acquire();
	// Mesa: while(!count)
	while(count == 0){
		notEmpty.wait(&lock);	// 当缓冲区空时将线程enqueue
	}
	remove c from buffer;
	count--;
	/* Mesa: 
	if(count == 1){
		Signal(full);
   }*/
	notFull.Signal();
	lock->release();
}
```

signal之后的选择：
- 被唤醒的线程立即执行，刮起发送方
- 退出管程
- 继续执行

### 屏障原语

指定一个屏障变量
广播给其他$n-1$个线程
若屏障变量的值达到$n$，则继续

### 读者-写者问题

读者：只读取数据，不修改
写者：读取和修改数据

对共享数据的读写：
- 读读允许：同一时刻允许多个读者同时读取
- 读写互斥：读和写不能同时进行
- 写写互斥：没有其他写者时写者才能写

信号量：writemutex，控制读写操作的互斥，初始化为1
读者计数：rcount，正在进行读操作的读者树木，初始化为0
信号量：countmutex，控制对读者计数的互斥修改，初始化为1

读者优先：

```C
P(countmutex);

if(rcount == 0){
	P(writemutex);
}
++rcount;

V(countmutex);

p(countmutex);

--rcount;
if(rcount == 0){
	V(writemutex);
}

V(countmutex);
```

## 进程通信

两个原语：send, receive

### 消息队列

每个消息是一个字节序列
相同标识的消息组成按先进先出的顺序组成一个消息队列

系统调用
- `msgget(key, flags)`
- `msgsnd(QID, buf, size, flags)`
- `msgrcv(QID, buf, size, type, flags)`
- `msgctl()`

### 共享内存

操作系统将同一个物理内存区域同时映射到多个进程的内存地址空间的通信机制

每个进程将共享内存区域映射到私有地址空间

优点：快速、方便共享数据
缺点：必须使用额外同步机制协调数据访问

系统调用
- `shmget(key, size, flags)`
- `shmat(shmid, *shmaddr, flags)`
- `shmdt(*shmaddr)`
- `shmctl()`

需要信号量等机制协调共享内存的访问冲突

### 管道

进程间给予内存文件的通信机制

子进程从父进程继承文件描述符
0-stdin, 1-stdout, 2-stderr

进程不关心另一端，可能从键盘、文件、程序读取，可能写入终端、文件、程序

系统调用

- 读管道：`read(fd, buffer, nbytes)`
- 写管道：`write(fd, buffer, nbytes)`
- 创建管道：`pipe(rgfd)`，`rgfd`为两个文件描述符组成的数组，`rgfd[0]`为读文件描述符，`rgfd[1]`为写文件描述符

### 信号

信号的接收处理

- 不活
- 忽略
- 屏蔽

