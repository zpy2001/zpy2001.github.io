---
title: OSTEP - 笔记
description: "Notes about OSTEP"
slug: system-bachelor-notes-book-notes-OSTEP
date: 2022-01-01 00:00:00+0000
categories:
    - OS
tags:
    - study
    - OS
    - CS
weight: 1
---

# OSTEP

## 抽象：进程

进程就是运行中的程序

提供有许多CPU可用的假象，通过虚拟化CPU，让一个进程只运行一个时间片，切换其他进程

分时共享CPU

进程的机器状态：程序在运行时可以读取和更新的部分
- 地址空间：进程可访问的内存
- 寄存器
    - 特殊寄存器：PC,SP(stack pointer),FP(frame pointer)

### 进程API

- 创建(create)
- 销毁(destroy)
- 等待(wait)
- 其他控制(miscellaneous control)
    - 例如暂停进程一段时间然后恢复运行
- 状态(statu)
    - 获得进程信息，运行时间与状态

`fork()`系统调用，父进程获得子进程的PID，子进程获得0，不使用`wait()`时，父子进程完成顺序不确定

`exec()`给定可执行程序的名称及需要的参数，函数会从可执行程序中加载代码和静态数据，并用它覆盖自己代码段、静态数据、堆、栈，然后OS执行该程序，子进程执行函数之后无返回值

分离`fork()`与`exec()`给了shell在此二者之间运行代码的机会，可以在运行新程序之前改变环境

shell程序首先显示一个提示符(prompt)，通过`fork`创建新进程，`exec`变体执行用户输入的程序，`wait`等待子进程结束，执行结束后shell返回并且再次输出一个提示符，等待下一次用户输入

分离`fork,wait`后的有趣功能：shell可以将标准输出关闭，重定向到某些文件中

UNIX pipe也是如此，输出被连接到内核管道pipe上的队列，另一个进程的输入也被链接

### 进程创建

1. 将代码和所有静态数据（如初始化变量）加载到进程的地址空间中。现代OS，惰性，尽在需要的时候加载
2. 为进程的运行时栈分配内存，可能会用参数初始化栈，填入`main()`
3. 为程序的堆(heap)分配一些内存，C中用于显式请求动态分配的数据
4. 其他，特别是I/O任务，默认为每个进程打开3个文件描述符（标准输入、输出、错误），轻松读取来自终端输入、打印输出
5. 启动程序，到入口运行，CPU控制权移交新创建例程

### 进程状态

- 运行
- 就绪
- 阻塞

进程从就绪到运行：被调度
从运行转移到就绪：被取消调度
I/O发起后进程从运行转移到阻塞
I/O完成后从阻塞转移到就绪（或立即再次运行）

还可能的状态：
- initial:进程在创建时
- final/zombie:进程处于已退出但并未清理的状态，允许其他进程检查进程的返回代码查看进程是否成功执行

## 机制：受限直接执行

只需直接在CPU上运行程序即可

直接运行协议（无限制）

操作系统|程序
:-|:-
在进程列表上创建条目<br>为程序分配内存<br>像程序加载到内存中<br>根据`argc/argv`设置程序栈|-
清除寄存器<br>执行`call main()`方法|-
-|执行`main()`<br>从`main`中执行`return`
释放进程的内存<br>将进程从列表删除|-

### 受限操作

> 用户模式下应用程序不能完全访问硬件资源，内核模式下操作系统可以。提供陷入(trap)内核和从陷阱返回(trapret)到用户模式程序的特别说明，以及一些指令使操作系统告诉硬件所谓陷阱表(trap table)在内存中的位置

`trap`指令跳入内核并且提升特权级别为内核模式，完成后`trapret`指令降低特权级别回到用户模式

x86上处理器会将程序计数器、标志、寄存器推送到每个进程内核栈上
返回陷阱从栈弹出这些值

内核必须谨慎控制在陷阱上执行的代码

内核通过在启动时设置陷阱表(trap table)来实现。机器启动时，在特权（内核）模式下执行，OS告诉硬件发生异常时应该运行什么代码，通过特殊指令，通知硬件陷阱处理程序的位置。

受限运行协议

操作系统@启动（内核模式）|硬件|程序
:-|:-|:-
初始化陷阱表|-|-
-|记住系统调用处理程序的位置|-
在进程列表上创建条目<br>为程序分配内存<br>像程序加载到内存中<br>根据`argc/argv`设置程序栈<br>用寄存器/程序计数器填充内核栈<br>从陷阱返回|-|-
-|从内核栈恢复寄存器<br>转向用户模式<br>跳到main|-
-|执行`main()`<br>调用系统调用<br>陷入内核
-|将寄存器保存到内核栈<br>转向内核模式<br>跳到陷阱处理程序|-
处理陷阱<br>做系统调用工作<br>陷阱返回|-|-
-|-|<br>从`main`中执行`return`<br>陷入，通过exit()
释放进程的内存<br>将进程从列表删除|-|-

### 进程切换

#### 协作方式：等待系统调用（非抢占式）

若无系统调用

#### 非写作方式：OS控制（抢占式）

始终中断，始终设备每隔几毫秒产生一次中断，进程停滞，预先配置的中断处理程序运行，OS获得CPU控制权，停止当前进程运行，启动另一进程

启动过程操作系统需要启动时钟

上下文切换：OS为当前执行进程保存寄存器值（到它的内核栈），并为即将执行的进程（从它内核栈）恢复一些寄存器值

会调用汇编保存/恢复寄存器和PC

操作系统@启动（内核模式）|硬件|程序
:-|:-|:-
初始化陷阱表|-|-
-|记住系统调用处理程序的位置|-
启动中断时钟|-|-
-|启动时钟，每隔x ms终端CPU|-
在进程列表上创建条目<br>为程序分配内存<br>像程序加载到内存中<br>根据`argc/argv`设置程序栈<br>用寄存器/程序计数器填充内核栈<br>从陷阱返回|-|-
-|从内核栈恢复寄存器<br>转向用户模式<br>跳到进程A`main`|-
-|-|进程A<br>执行`main()`
-|时钟中断，将寄存器(A)保存到内核栈(A)<br>转向内核模式<br>跳到陷阱处理程序|-
处理陷阱<br>调用`switch()`历程<br>&nbsp;&nbsp;将寄存器(A)保存到进程结构(A)<br>&nbsp;&nbsp;将进程结构(B)恢复到寄存器(B)|-|-
-|从内核栈(B)恢复寄存器(B)<br>转向用户模式<br>跳到B的PC|-
-|-|进程B

OS可以简单决定在中断处理期间禁止中断，但会导致丢失中断

加锁方案，保护对内部数据结构的并发访问

## 进程调度（上层策略）

### 工作负载假设

对运行的进程作出如下假设：
1. 每一个工作运行相同的时间
2. 所有的工作同时到达
3. 一旦开始，每个工作保持运行直到完成
4. 所有的工作只利用CPU（无I/O）
5. 每个工作运行时间是已知的

### 调度指标

指标：周转时间
定义：任务完成一件减去任务到达系统的时间
$$T_{周转时间}= T_{完成时间}-T_{到达时间}$$

因为假设2，所有任务同时到达，$T_{到达时间}=0$，因此$T_{周转时间}=T_{完成时间}$

性能 & 公平
矛盾

### 策略

#### FIFO

当先来的进程运行时间长时平均周转时间长
护航效应

#### SJF（最短任务优先）

针对假设2最优，但是假设2不成立时仍然需等待

#### STCF（最短完成时间任务优先）

向SJF添加抢占，抢占式最短作业优先

### 新指标：响应时间

分时系统
响应时间：任务到达系统到首次运行时间
$$T_{响应时间}= T_{首次运行}-T_{到达时间}$$

STCF响应时间不好

### 轮转(RR)

RR在一个时间片内运行一个工作，只关心周转时间，RR最差

### 接合I/O

I/O到来中断，进程进入阻塞队列；I/O完成，产生中断，进程从阻塞队列进入就绪队列
使用重叠可以更好使用资源，将A的任务分解，不占用CPU的时段运行B

### 无法预知

调度程序不知道作业长度？？？

## 调度：多级反馈队列(MLFQ)

应用于兼容时分共享系统，利用历史经验预测未来

MLFQ中有许多独立的队列，每个队列有不同的优先级，任何时刻一个工作只能存在于一个队列中，MLFQ总是优先执行高优先级的工作

## 调度：比例份额

调度程序的最终目标是确保每个工作获得一定比例的CPU时间，而不是优化周转时间和响应时间

彩票调度，每隔一段时间抽奖，频繁运行的进程机会更多

彩票数(ticket)代表进程占有某个资源的份额，彩票数多抽中概率大

机制：
- 彩票货币：允许用户以灵活的彩票数分配给工作
- 彩票转让：进程可以临时将自己的彩票交给另一个进程

实现：生成随机数count，遍历列表，知道找到第一个值大于count的任务，将列表项按照彩票数递减排列更有效率





## 分页

空间划分为固定长度的分片

地址空间虚拟页映射到物理内存特定页帧

记录进程地址空间每个虚拟页放在物理内存中的位置，为进程保存页表，进行地址转换

将虚拟页转换，偏移量不变

## 锁

### 使用

通过锁来提供互斥进入临界区的函数

```C
int pthread_mutex_lock(pthread_mutex_t *mutex);
int pthread_mutex_unlock(pthread_mutex_t *mutex);

pthread_mutex_t lock;
pthread_mutex_lock(&lock);
x=x+1;
pthread_mutex_lock(&lock);
```

若在调用lock的时候没有其他线程持有锁，线程将获取该锁并进入临界区；
若另一个线程持有该锁，则尝试获取该锁的进程将等待知道获得该锁（另一个进程解锁）

锁需要正确初始化，确保具有正确的值
POSIX线程，初始化法1:使用`PTHREAD_MUTEX_INITIALIZER`
动态方法，在运行时调用`pthread_mutex_init()`：

```C
int rc = pthread_mutex_init(&lock,NULL);
assert(rc==0);
```

使用完锁之后调用：`pthread_mutex_destroy()`

```C
int pthread_mutex_trylock(pthread_mutex_t *mutex);
int pthread_mutex_timedlock(pthread_mutex_t *mutex, struct timespec *abs_timeout);
```
超时失败

锁变量，保存了锁在某一时刻的状态

- 可用的：线程持有锁
- 被占用的：表示线程持有锁，并且处于临界区

可保存持有锁的线程/请求获取锁的线程队列

mutex互斥量，POSIX库

评价锁，完成功能，公平，性能

### 实现锁

#### 控制中断

互斥解决方案，临界区关中断，进入临界区的代码不会被中断

这种方法要求我们允许所有调用线程执行特权操作，开关中断；不支持多核处理器；关闭中断导致中断丢失；效率低

Peterson算法

```C
int flag[2];
int turn;
void init() {
    flag[0] = flag[1] = 0; turn = 0;
}
void lock() {
    flag[self] = 1;
    turn = 1 - self;
    while ((flag[1-self] == 1) && (turn == 1 - self))
    ; // spin-wait
}
void unlock() {
    flag[self] = 0; // simply undo your intent 
}
```

原理/解释：

```C

bool flag[2] = {false, false};
int turn;

P0:      flag[0] = true;
P0_gate: turn = 1;
         while (flag[1] == true && turn == 1)
         {
             // busy wait
         }
         // critical section
         ...
         // end of critical section
         flag[0] = false;

P1:      flag[1] = true;
P1_gate: turn = 0;
         while (flag[0] == true && turn == 0)
         {
             // busy wait
         }
         // critical section
         ...
         // end of critical section
         flag[1] = false;
```

#### 测试并设置指令

硬件支持：原子交换(test-and-set instruction)

需要避免错误：中断切换线程时标志相同

SPARC上实现指令为`ldstub`，x86上为`xchg`

```C
int TestAndSet(int *old_ptr, int new) {
    int old = *old_ptr; // fetch old value at old_ptr
    *old_ptr = new; // store 'new' into old_ptr
    return old; // return the old value
}

typedef struct lock_t {
    int flag;
} lock_t;

void init(lock_t *lock) {
    // 0 indicates that lock is available, 1 that it is held lock->flag = 0;
}
void lock(lock_t *lock) {
    while (TestAndSet(&lock->flag, 1) == 1)
    ; // spin-wait (do nothing)
}
void unlock(lock_t *lock) {
    lock->flag = 0;
}
```

既可以测试旧值，又可以设置新值，使用它在`lock()`函数，能够实现简单的自旋锁

单核上需要抢占式调度器

中断打断之后仍然判断1==1，因此始终循环直到前一进程

自旋锁无公平性

### 比较并交换

另一个硬件源语，`compare-and-swap`，`x86`为`cmpxchg`

```C
int CompareAndSwap(int *ptr, int expected, int new) {
    int actual = *ptr;
    if (actual == expected)
        *ptr = new;
    return actual;
}
```

使用`CompareAndSwap(&lock->flag, 0, 1) == 1`可实现锁

#### 连接加载和条件存储

MIPS，`lc/sc`

LL取数且置系统LLbit为1
LL为1时，处理器检查响应单元是否被修改，如果其他处理器或设备访问了相应但愿或执行了entry操作，LLbit置为0

执sc时若`LLbit`为1，则存数成功，将目标寄存器置为1；否则不成功，置0

在两次修改寄存器中的值的期间，没有任何一个处理器改变它内存中的值

```C
int LoadLinked(int *ptr) {
    return *ptr;
}
int StoreConditional(int *ptr, int value) {
    if (no one has updated *ptr since the LoadLinked to this address) {
        *ptr = value;
        return 1; // succes
    } else {
        return 0; // failed to update
    }
}

void lock(lock_t *lock){
    while(1){
        while(LoadLinked(&lock->flag) == 1){
            ;
        }
        if(StoredConditional(&lock->flag,1)==1){
            return;
        }
    }
}
```

#### 获取并增加

`fetch-and-add`，能原子地返回特定地址的旧值，并且让该值自增1

```C
int FetchAndAdd(int *ptr){
    int old = *ptr;
    *ptr = old + 1;
    return old;
}

typedef struct lock_t {
    int ticket;
    lock->turn;
} lock_t;

void lock(lock_t *lock){
    int myturn = FetchAndAdd(&lock->ticket);
    while(lock->turn != myturn){
        ;
    }
}

void unlock(lock_t *lock){
    FetchAndAdd(&lock->turn);
}
```

获取锁ticket值后ticket自增1，原值作为线程的`turn`，顺位

线程的myturn灯与锁的turn时，进入临界区

解锁增加锁的turn，使下一个线程ticket能够与turn相等

本方法保证所有线程都能抢到锁，只要线程获得了ticket则最终会被调度

### 自旋过多

N个线程去竞争1个锁，会浪费N-1个时间片

### 操作系统配合

#### yield()让出

如果临界区线程发生上下文切换，则其他线程只能一直自旋，浪费时间片

自旋的时候放弃CPU

```C
void init(){
    flag = 0;
}

void lock(){
    while(TestAndSet(&flag,1)==1){
        yield(); // give up CPU
    }
}

void unlock(){
    flag = 0;
}
```

`yield()`系统调用能够使运行态变为就绪态，允许其他线程运行
线程取消调度了自己

上下文切换成本大

#### 使用队列替代自旋：休眠代替自旋

添加控制，决定锁释放时谁能抢到锁，使用队列保存等待锁的线程

Solaris，`park()`使调用进程休眠，`unpark(threadID)`唤醒threadID标识的线程

```C
typedef struct lock_t{
    int flag;
    int guard;
    queue_t *q;
} lock_t;   

void lock_init(lock_t *m){
    m->flag = 0;
    m->guard = 0;
    queue_init(m->q);
}

void lock(lock_t *m){
    while(TestAndSet(&m->guard,1) == 1){
        ;
    }
    if(m->flag == 0){
        m->flag = 1;
        m->guard = 0;
    }
    else{
        queue_add(m->q,gettid());
        m->guard = 0;
        park();
    }
}

void unlock(lock_t *m){
    while(TestAndSet(&m->guard,1) == 1){
        ;
    }
    if(queue_empty(m->q){
        m->flag = 0;
    }
    else{
        unpark(queue_remove(m->q));
    }
    m->guard = 0;
}
```

启动下一个线程后锁不用动`flag`，直接是隶属于下一个线程，下一线程无需调用`lock`

竞争条件，线程将`park`切换到另一个持有锁的线程，若持有锁的线程释放了锁，`unpark`不能唤醒下一线程，会出现错误，之后的所有线程`park`

Solaris通过`setpark()`，一个线程表面自己将要park，若另一个线程被调度、调用unpark，则后续park调用会直接返回，而非一直睡眠

在`m->guard=0`之前使用`setpark()`

guard进入内核采取预防

linux futex

每个futex关联一个特定的物理内存位置，有一个事先建好的内核队列

调用`futex_wait(address,expected)`，若address处的值等于expected，则让线程睡眠，否则调用返回

`futex_wake(address)`唤醒等待队列中一个线程

```C
void mutex_lock (int *mutex) {
    int v;
    /* Bit 31 was clear, we got the mutex (this is the fastpath) */
    if (atomic_bit_test_set (mutex, 31) == 0)
        return;
    atomic_increment (mutex);
    while (1) {
        if (atomic_bit_test_set (mutex, 31) == 0) {
            atomic_decrement (mutex);
            return;
        }
        /* We have to wait now. First make sure the futex value we are monitoring is truly negative (i.e. locked). */
        v = *mutex;
        if (v >= 0) continue;
        futex_wait (mutex, v);
    }
}

void mutex_unlock (int *mutex) {
    /* Adding 0x80000000 to the counter results in 0 if and only if
        there are not other interested threads */
    if (atomic_add_zero (mutex, 0x80000000))
        return;
    /* There are other threads waiting for this mutex, wake one of them up. */
    futex_wake (mutex);
}
```

nptl库中`lowlevellock.h`，利用证书记录锁是否被持有，以及等待者的个数

##