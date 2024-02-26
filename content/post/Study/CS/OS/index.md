---
title: 现代操作系统课程
description: 杨力祥老师的国科大研究生现代操作系统思考题
slug: Modern-Operation-System
date: 2024-02-25
categories:
  - OS
tags:
  - OS
  - class
  - CS
---
### 考前复习
#### 第二次作业
**1.为什么开始启动计算机的时候，执行的是BIOS代码而不是操作系统自身的代码？**
在计算机上电启动时，操作系统程序位于磁盘上，而电脑只能运行RAM中的程序，因此需要执行RAM中的BIOS将操作系统代码搬运到RAM上
***
**2.为什么BIOS只加载了一个扇区，后续扇区却是由bootsect代码加载？为什么BIOS没有直接把所有需要加载的扇区都加载？**
BIOS的工作是加载第一个扇区，也即bootsect的程序到RAM上，剩余的工作可以由bootsect完成；这样设计是为了便于操作系统的设计者更自由的安排操作系统的后续内容，只要BIOS和操作系统的设计者遵循同样的约定，将bootsect放在第一个扇区的512B内，后续的安排则完全由操作系统设计者自行裁量
***
**3.为什么BIOS把bootsect加载到0x07c00，而不是0x00000？加载后又马上挪到0x90000处，是何道理？为什么不一次加载到位？**
0x00000处会安排极为重要的中断向量表和BIOS数据，如果一开始加载到此处，在后续的运行中会被覆盖；而0x07c00在后续则会被其他加载的内容（缓冲区）覆盖，因而必须后移；不一次加载到位的原因更多是历史原因，对于内存有限的（小于640KB）的旧设备而言，BIOS的寻址空间受限，这种两段加载的方式给操作系统设计者更大的自由；
***
**4.bootsect、setup、head程序之间是怎么衔接的？给出代码证据。**
bootsect在完成前四个（1-4）扇区的加载等先期工作后，会通过
```C
jmpi 0,SETUPSEG
```
跳转到setup
而setup在完成后，会通过
```C
jmpi 0,8
```
跳转至GDT的第一项（前面还有第0项）
***
**5.setup程序的最后是jmpi 0,8 ，为什么这个8不能简单的当作阿拉伯数字8看待，究竟有什么内涵？**
在这里是根据段选择子跳转，0是offset，而8则应理解为二进制下的...01000，其中最低二位表示特权级为0特权，第三位为0表示是GDT，而高位则是索引，这里是GDT的第1项（前面有第0项）
***
**6.保护模式在“保护”什么？它的“保护”体现在哪里？特权级的目的和意义是什么？分页有“保护”作用吗？**
```C
mov ax,#0x0001 ! protected mode (PE) bit
lmsw ax        ! This is it!
```
保护内核/用户程序和地址空间不被其他用户程序非法执行或访问，通过硬件完成间接寻址、将虚拟地址映射到用户无法预估的物理地址、从而隐藏真实物理地址。

打开了保护模式后，CPU的寻址模式发生了变化，需要依赖于GDT去获取代码或数据段的基址。从GDT可以看出，保护模式除了段基址外，还有段限长，这样相当于增加了一个段位寄存器。既有效地防止了对代码或数据段的覆盖，又防止了代码段自身的访问超限，明显增强了保护作用。

特权级的目的是限制用户对于系统关键组件的修改（例如GDTR/LDTR等），防止用户自定义数据结构覆盖内核指定的数据结构，引发非法访问的安全隐患。特权级影响范围是段，所有段选择符最后两位标识特权级。

分页也有保护作用，保证内核可以访问所有用户进程的空间，但用户进程不能够互相访问或访问内核。内核分页线性递增，能够知道虚地址实际对应的物理地址，用户程序的地址空间从高到低分配，进程的动态执行与临时分页使得页分配具有随机性，无法被预测。
***
**7.在setup程序里曾经设置过gdt，为什么在head程序中将其废弃，又重新设置了一个？为什么设置两次，而不是一次搞好？**
setup设置的gdt是在setup的内存空间上，在之后会被缓冲区覆盖，因此需要重新设置。如果一开始就把gdt安排在指定位置，则会与head程序的地址安排发生冲突；若先设置gdt，则gdt会被覆盖，反之则会覆盖部分head代码，因此需要用后运行head重新找一个安全的位置设置gdt
***
**8.内核的线性地址空间是如何分页的？画出从0x000000开始的7个页（包括页目录表、页表所在页）的挂接关系图，就是页目录表的前四个页目录项、第一个个页表的前7个页表项指向什么位置？给出代码证据。**
第一个页是页目录项，第二三四五页是页表项，后续则为正常页，页目录项分别指向各个页表，第二页即第一个页表页分别指向前1K的页表
***
**9.根据内核分页为线性地址恒等映射的要求，推导出四个页表的映射公式，写出页表的设置代码。**
内核分页还不考虑开启保护模式支持16MB，分在4个页表中，共4K页，因此采用
2:10:12的方式完成映射，2为页目录，10为页表项，12为offset
***
**10.为什么不用call，而是用ret“调用”main函数？画出调用路线图，给出代码证据。**
ret在弹出eip后是不会返回的，而call会先把call下方的指令压栈，随后在call跳转去的程序执行完毕后，返回执行其保存的“下一条”指令，而操作系统是一个持续执行的程序，除非强制下电等特殊情况，是不会退出的，因此无须返回，直接用ret即可。

#### 第三次作业
**1、计算内核代码段、数据段的段基址、段限长、特权级**
这个需要给出代码和数据结构
![[os3-1.png]]

在head中有gdt的安排
0x00c09a0000000fff
0x00c0920000000fff
首先看第一段（代码段）
0000|0000|1010|0000|1001|1010|0000|0000
0000|0000|0000|0000|0000|1111|1111|1111
段基址为0，段限长为16MB，特权级为0
第二段为数据段，结果同上
***
**2、计算进程0的代码段、数据段的段基址、段限长、特权级。**
在sched里的INIT TASK中给出了了结构
```C
/* ldt */ {0x9f,0xc0fa00}, \
		  {0x9f,0xc0f200}, \
```
计算方式完全同上，base为0，G为1，特权级为3，段限长为640KB
***
**3、fork进程1之前，为什么先调用move_to_user_mode()？用的是什么方法？解释其中的道理。**
Linux系统规定，任何非0进程必须由先前的进程在用户态特权级下创建；在move to user mode的作用是将此时的进程0利用中断机制和iret从0特权级翻到3特权级；
```C
#define move_to_user_mode() \
__asm__ ("movl %%esp,%%eax\n\t" \
    "pushl $0x17\n\t" \
    "pushl %%eax\n\t" \
    "pushfl\n\t" \
    "pushl $0x0f\n\t" \
    "pushl $1f\n\t" \
    "iret\n" \
    "1:\tmovl $0x17,%%eax\n\t" \
    "movw %%ax,%%ds\n\t" \
    "movw %%ax,%%es\n\t" \
    "movw %%ax,%%fs\n\t" \
    "movw %%ax,%%gs" \
    :::"ax")
```
具体方法是手动伪造中断的压栈，把SS ESP EFLAGS CS和EIP分别进行压栈，其中SS和CS分别为0x17和0x0f，即00010111和00001111，可见是LDT且特权级为3，在iret到eip存储地址继续执行后，特权级已经是3了
***
**4、根据什么判定move_to_user_mode()中iret之后的代码为进程0的代码。**
基于上题的解释，此时会去ldt寻找代码段执行，而在move之前的sched中，已经有如下代码
```C
ltr(0);
lldt(0);
```
使得ldt此时存储的就是进程0的信息
***
**5、进程0的task_struct在哪？具体内容是什么？给出代码证据。**
在sched.h中，具体内容如下
```C
/*tss*/    {0,PAGE_SIZE+(long)&init_task,0x10,0,0,0,0,(long)&pg_dir,\
     0,0,0,0,0,0,0,0, \
     0,0,0x17,0x17,0x17,0x17,0x17,0x17, \
     _LDT(0),0x80000000, \
        {} \
    }, \
```
***
**6、在system.h里**  
```C
#define _set_gate(gate_addr,type,dpl,addr) \  
__asm__ ("movw %%dx,%%ax\n\t" \  
    "movw %0,%%dx\n\t" \  
    "movl %%eax,%1\n\t" \  
    "movl %%edx,%2" \  
    : \  
    : "i" ((short) (0x8000+(dpl<<13)+(type<<8))), \  
    "o" (*((char *) (gate_addr))), \  
    "o" (*(4+(char *) (gate_addr))), \  
    "d" ((char *) (addr)),"a" (0x00080000))

#define set_intr_gate(n,addr) \  
    _set_gate(&idt[n],14,0,addr)

#define set_trap_gate(n,addr) \  
    _set_gate(&idt[n],15,0,addr)

#define set_system_gate(n,addr) \  
    _set_gate(&idt[n],15,3,addr)  
```
**读懂代码。这里中断门、陷阱门、系统调用都是通过_set_gate设置的，用的是同一个嵌入汇编代码，比较明显的差别是dpl一个是3，另外两个是0，这是为什么？说明理由。**
中断和陷阱门都是内核调用的，而系统调用则是要给用户态使用的，因而系统调用的特权级为3
***
**7、分析get_free_page()函数的代码，叙述在主内存中获取一个空闲页的技术路线。**
```C
unsigned long get_free_page(void)
{
register unsigned long __res asm("ax");

__asm__("std ; repne ; scasb\n\t" #从最后向前扫描时候有空页
    "jne 1f\n\t"#若没有则直接跳转到1f，返回一个0
    "movb $1,1(%%edi)\n\t"#找到后把对应的mem_map项置一
    "sall $12,%%ecx\n\t"#相对页面起始地址
    "addl %2,%%ecx\n\t"#实际物理起始地址
    "movl %%ecx,%%edx\n\t"#把物理地址放在edx里
    "movl $1024,%%ecx\n\t"#给计数寄存器放在ecx
    "leal 4092(%%edx),%%edi\n\t"#把页面末端存在edi里
    "rep ; stosl\n\t"#清零
    "movl %%edx,%%eax\n"#把物理地址放在返回值
    "1:"
    :"=a" (__res)
    :"0" (0),"i" (LOW_MEM),"c" (PAGING_PAGES),
    "D" (mem_map+PAGING_PAGES-1)
    :"di","cx","dx");
return __res;
}
```
***
**8、copy_process函数的参数最后五项是：long eip,long cs,long eflags,long esp,long ss。查看栈结构确实有这五个参数，奇怪的是其他参数的压栈代码都能找得到，确找不到这五个参数的压栈代码，反汇编代码中也查不到，请解释原因。详细论证其他所有参数是如何传入的。**
这五个参数是通过int 0x80硬件压栈
```C
int copy_process(int nr,long ebp,long edi,long esi,long gs,long none,
        long ebx,long ecx,long edx,
        long fs,long es,long ds,
        long eip,long cs,long eflags,long esp,long ss)
```
ds到ebx是在system call中压栈的
none是在sys call table时压栈的
gs到nr是_sys_fork中压栈的，其中nr即eax
***
**9、详细分析Linux操作系统如何设置保护模式的中断机制。**
在head中设置IDT，使用lidt idt_descr将256项idt数组位置等信息加载到idtr
在`trap_init`中设置IDT表项，即中断、系统陷入门，使得x86硬件能够正确识别和跳转到对应的处理函数
各个处理函数正确处理栈，完成处理逻辑后通过iret返回

- 在setup中cli关中断，设置idtr，开启32位和保护模式
- 在head中初始化idt，使用lidt descr将256项idt数组位置等信息加载置为0
- 在main中的trap init中设置idt表项，将中断服务程序和idt挂接，把idt中对应的表项按格式填好，其中包括中断服务程序的地址，dpl等信息；此外还会挂载一些硬件相关的中断，如时钟中断和外设的中断，在sched中挂载0x80中断，作为系统调用使用，使得x86硬件能够正确识别和跳转到对应的函数
- 全部完成后sti开中断
- 另外各个函数正确处理栈，完成处理逻辑后通过iret返回
***
**10、分析Linux操作系统如何剥夺用户进程访问内核及其他进程的能力。**
通过特权级和分页机制；前者保证用户进程无法修改一些关键寄存器，比如ldtr和gdtr等，保证内核运行的机制不被破坏；分段分页机制和特权级相协同，通过权限检查使得进程只能访问被授权访问的代码段，数据段和栈段；不同段的读写权限又限制了使用方式，每个进程相关的页面只在每个进程的页表中，而硬件支持的寻址使得每个进程只能访问分配给自己的页；
***
**11、分析后面两行代码的意义。**
```C
_system_call:  
    cmpl $nr_system_calls-1,%eax  
    ja bad_sys_call
```
nr_system_calls为支持的系统调用数量，在这里是检查系统调用号是否合法，如果非法则直接进入bad sys call处理。

#### 第四次作业
**3、分析copy_page_tables（）函数的代码，叙述父进程如何为子进程复制页表。**
```C
int copy_page_tables(unsigned long from,unsigned long to,long size)
{
    unsigned long * from_page_table;
    unsigned long * to_page_table;
    unsigned long this_page;
    unsigned long * from_dir, * to_dir;
    unsigned long nr;

    if ((from&0x3fffff) || (to&0x3fffff))
        panic("copy_page_tables called with wrong alignment");
    from_dir = (unsigned long *) ((from>>20) & 0xffc); /* _pg_dir = 0 */
    to_dir = (unsigned long *) ((to>>20) & 0xffc);
    size = ((unsigned) (size+0x3fffff)) >> 22;
    for( ; size-->0 ; from_dir++,to_dir++) {
        if (1 & *to_dir)
            panic("copy_page_tables: already exist");
        if (!(1 & *from_dir))
            continue;
        from_page_table = (unsigned long *) (0xfffff000 & *from_dir);
        if (!(to_page_table = (unsigned long *) get_free_page()))
            return -1;    /* Out of memory, see freeing */
        *to_dir = ((unsigned long) to_page_table) | 7;
        nr = (from==0)?0xA0:1024;
        for ( ; nr-- > 0 ; from_page_table++,to_page_table++) {
            this_page = *from_page_table;
            if (!(1 & this_page))
                continue;
            this_page &= ~2;
            *to_page_table = this_page;
            if (this_page > LOW_MEM) {
                *from_page_table = this_page;
                this_page -= LOW_MEM;
                this_page >>= 12;
                mem_map[this_page]++;
            }
        }
    }
    invalidate();
    return 0;
}
```

- 首先检查f和t是否在4Mb内存边界地址
- 求fd和td的起始页目录项，并求要转移的页表数（页目录项数）
- 检查ft的present位，应当是前者为1（存在）后者为0（不存在）
- 随后通过gfp来获取空闲内存页存放
- 设置目的目录项信息，7为usr R/W present
- 判断当前处理的页表是否是在内核空间，若是则只用处理160页（640KB），反之要处理1K页
- 随后开始迁移页表，获取源页表项，如果f中有未使用页表则无须复制
- 复位页表项中的R/W位
- 复制
- 后续代码视情况执行，若页表项所指的页面地址在1MB以上，则需要执行COW操作，而在初次fork时，都是内核空间，因而会直接跳过这部分；本部分内容为令原页表也只读，并对mem map进行设置（引用次数+1）
- 重写CR3刷新TLB
***
**4、进程0创建进程1时，为进程1建立了task_struct及内核栈，第一个页表，分别位于物理内存两个页。请问，这两个页的位置，究竟占用的是谁的线性地址空间，内核、进程0、进程1、还是没有占用任何线性地址空间？说明理由（可以图示）并给出代码证据。**
占用的内核线性地址空间
具体代码在fork操作的copy mem中，规定了64MB分段给一个进程，不同进程的页表完全隔离
在head中则规定了内核线性地址和物理地址一一对应
task struct和内核栈占用的是最后两个物理页（16MB的最后两页）
进程0占0-640KB线性地址空间
进程1则在64MB之后
***
**5、假设：经过一段时间的运行，操作系统中已经有5个进程在运行，且内核为进程4、进程5分别创建了第一个页表，这两个页表在谁的线性地址空间？用图表示这两个页表在线性地址空间和物理地址空间的映射关系。**
内核线性地址空间
每个进程16个页表项
$4\times 64$MB
$5\times 64$MB
页目录表0x40和0x50索引到页目录，随后根据21-12位索引到页表，再根据offset确定物理地址空间即可
***
6、**代码中的"ljmp %0\n\t" 很奇怪，按理说jmp指令跳转到得位置应该是一条指令的地址，可是这行代码却跳到了"m" (*&__tmp.a)，这明明是一个数据的地址，更奇怪的，这行代码竟然能正确执行。请论述其中的道理。**
```C
#define switch_to(n) {\
struct {long a,b;} __tmp; \
__asm__("cmpl %%ecx,_current\n\t" \
    "je 1f\n\t" \
    "movw %%dx,%1\n\t" \
    "xchgl %%ecx,_current\n\t" \
    "ljmp %0\n\t" \
    "cmpl %%ecx,_last_task_used_math\n\t" \
    "jne 1f\n\t" \
    "clts\n" \
    "1:" \
    ::"m" (*&__tmp.a),"m" (*&__tmp.b), \
    "d" (_TSS(n)),"c" ((long) task[n])); \
}
```

这里涉及到limp指令相关的问题；理论上ljmp指令后面需要跟一个四字节或六字节参数，其中有二或四字节是eip，二字节是cs，但这里是涉及到任务切换的长跳转，因而tmp a存储的四字节offset信息是无用的，核心是其后存储的tmp b，而内存实际情况和此处的表现是相反的，因此后续跟的参数实质上就是新任务的段选择子tmp b
***
**7、进程0开始创建进程1，调用fork（），跟踪代码时我们发现，fork代码执行了两次，第一次，执行fork代码后，跳过init（）直接执行了for(;;) pause()，第二次执行fork代码后，执行了init（）。奇怪的是，我们在代码中并没有看到向转向fork的goto语句，也没有看到循环语句，是什么原因导致fork反复执行？请说明理由（可以图示），并给出代码证据。**
这个问题有一点误导性，第二次执行fork的说法并不严谨，因为确实没有出现显式的第二执行整个fork的流程，而是在代码段中的中间部分开始执行；
进程0通过系统调用执行fork时，会创建进程1并使之在eip保存的地址处开始执行，而进程0本身会返回一个子进程id，即1，因而会进入for pause，而在中断后的调度后，进程1从前文所述的地址开始执行，这就是第二次执行fork，最后会返回其tss eax中存储的0，从而进入init，开始初始化。
***
**8、详细分析进程调度的全过程。考虑所有可能（signal、alarm除外）**
Linux的进程调度是抢占式的，但内核进程执行时不被抢占，在执行调度时，schedule函数会扫描所有就绪态进程的滴答计数counter的值，该值越大说明被执行的时间越短，即选中counter最大的进程进行调度；
如果所有就绪态进程的时间片都被用完，则系统会根据优先级重新计算所有进程（包括睡眠进程）的counter，具体公式是c = c/2+p；然后调度程序重新扫描就绪态进程进行调度；
***
**9、分析panic函数的源代码，根据你学过的操作系统知识，完整、准确的判断panic函数所起的作用。假如操作系统设计为支持内核进程（始终运行在0特权级的进程），你将如何改进panic函数？**
```C
volatile void panic(const char * s)
{
    printk("Kernel panic: %s\n\r",s);
    if (current == task[0])
        printk("In swapper task - not syncing\n\r");
    else
        sys_sync();
    for(;;);
}
```
首先判断是否是0进程，若是说明系统是在空闲状态下遇到的严重问题，此时不会进行同步，而是直接输出提示信息后开始for;;，因为这种情境下说明系统的问题极其严重，尝试同步甚至可能损坏磁盘；
若不是则将缓冲区的信息同步到磁盘上，此时系统随时可能死掉，及时同步信息是明智的
如果要支持内核进程，我会增加一些更详细的打印信息，并增加中断控制

#### 第五次作业
**1、getblk函数中，申请空闲缓冲块的标准就是b_count为0，而申请到之后，为什么在wait_on_buffer(bh)后又执行if（bh->b_count）来判断b_count是否为0？**
申请缓冲块的这一操作是抢占式的，在一个时刻会有许多进程在申请缓冲块，由于wait中有sleep函数，这意味着进程被暂时中断，任何从sleep恢复的进程在执行下一步前都应当对抢占式的资源进行一次是否仍然合法的检查，即缓冲块是否被占用
***
**2、b_dirt已经被置为1的缓冲块，同步前能够被进程继续读、写？给出代码证据。**
在缓冲块读写的相关代码中，并没有对dirt位的检查，换言之可以被继续读写；
dirt只意味着是否需要向磁盘写回，若为1则只有在写回后才能被用作新的缓冲区
只有在sys_sync sync_dev getblk和低级磁盘读写中会检查dirt
具体读写操作主要通过`bread(a)`来获取缓存块，其中只会检查`b_uptodate`即缓冲块内容是否领先于内存，若为0需要将同步读到缓冲
***
**3、wait_on_buffer函数中为什么不用if（）而是用while（）？**
本质上是前置判断和后置判断问题，若用if则只进行了一次前置检查，在sleep后sb块时候解锁并不确定，而while在sleep后会进行一次后置检查，此时若sb块已经解锁，才进行恢复中断操作
***
**4、分析ll_rw_block(READ,bh)读硬盘块数据到缓冲区的整个流程（包括借助中断形成的类递归），叙述这些代码实现的功能。**
```C
void ll_rw_block(int rw, struct buffer_head * bh)
{
    unsigned int major;

    if ((major=MAJOR(bh->b_dev)) >= NR_BLK_DEV ||
    !(blk_dev[major].request_fn)) {
        printk("Trying to read nonexistent block-device\n\r");
        return;
    }
    make_request(major,rw,bh);
}
```
***

首先检查块设备号是否合法，即是否有对应的设备；若有则发起请求`make_request(major,rw,bh);`
```C
static void make_request(int major,int rw, struct buffer_head * bh)
{
    struct request * req;
    int rw_ahead;

/* WRITEA/READA is special case - it is not really needed, so if the */
/* buffer is locked, we just forget about it, else it's a normal read */
    if (rw_ahead = (rw == READA || rw == WRITEA)) {
        if (bh->b_lock)
            return;
        if (rw == READA)
            rw = READ;
        else
            rw = WRITE;
    }
    if (rw!=READ && rw!=WRITE)
        panic("Bad block dev command, must be R/W/RA/WA");
    lock_buffer(bh);
    if ((rw == WRITE && !bh->b_dirt) || (rw == READ && bh->b_uptodate)) {
        unlock_buffer(bh);
        return;
    }
repeat:
/* we don't allow the write-requests to fill up the queue completely:
 * we want some room for reads: they take precedence. The last third
 * of the requests are only for reads.
 */
    if (rw == READ)
        req = request+NR_REQUEST;
    else
        req = request+((NR_REQUEST*2)/3);
/* find an empty request */
    while (--req >= request)
        if (req->dev<0)
            break;
/* if none found, sleep on new requests: check for rw_ahead */
    if (req < request) {
        if (rw_ahead) {
            unlock_buffer(bh);
            return;
        }
        sleep_on(&wait_for_request);
        goto repeat;
    }
/* fill up the request-info, and add it to the queue */
    req->dev = bh->b_dev;
    req->cmd = rw;
    req->errors=0;
    req->sector = bh->b_blocknr<<1;
    req->nr_sectors = 2;
    req->buffer = bh->b_data;
    req->waiting = NULL;
    req->bh = bh;
    req->next = NULL;
    add_request(major+blk_dev,req);
}
```
发起请求时先判断请求类型，若为预读写而缓冲块已经上锁，则直接退出；反之按照读写进行处理；随后寻找空闲请求项，若没有则sleeop进行等待；一旦找到后则填入请求配置，
```C
static void add_request(struct blk_dev_struct * dev, struct request * req)
{
    struct request * tmp;

    req->next = NULL;
    cli();
    if (req->bh)
        req->bh->b_dirt = 0;
    if (!(tmp = dev->current_request)) {
        dev->current_request = req;
        sti();
        (dev->request_fn)();
        return;
    }
    for ( ; tmp->next ; tmp=tmp->next)
        if ((IN_ORDER(tmp,req) ||
            !IN_ORDER(tmp,tmp->next)) &&
            IN_ORDER(req,tmp->next))
            break;
    req->next=tmp->next;
    tmp->next=req;
    sti();
}
```
在`add_request`中，若设备空闲（即当前进程为第一个挂载上的任务）则直接调用req fn进行处理，反之则用电梯排序挂载到设备的等待列表中。这里重点讨论前者，和req fn相关的情况，即*借助中断实现的类递归*
Linux目前支持软盘，硬盘和虚拟盘，以硬盘为例，其还有一个辅助函数
```C
static void hd_out(unsigned int drive,unsigned int nsect,unsigned int sect,
        unsigned int head,unsigned int cyl,unsigned int cmd,
        void (*intr_addr)(void))
{
    do_hd = intr_addr;
}

```
do_hd_request调用do out在设置硬盘读写数据配置的同时将一个异常处理函数设置好，等待中断时调用，相关硬盘中断处理函数如下：
```C
_hd_interrupt:
    xchgl _do_hd,%edx
    call *%edx        # "interesting" way of handling intr.
```
注意到在`do_hd_request`中，每种处理函数(`read_intr/write_intr/recal_intr`)内部最后都调用了`do_hd_request`，形成递归，即处理当前请求完成后再根据请求设置设备状态、等候中断处理，完成闭环

**5、分析包括安装根文件系统、安装文件系统、打开文件、读文件在内的文件操作**
- 安装根文件系统使用mount root函数
	- 检查并初始化设备与文件系统数据结构
	- 复制根设备sp
	- 读取虚拟盘中根inode放入itable，并且挂载到sp
	- 根文件系统与进程1关联
- 安装文件系统（mount命令，sys mount函数）
	- namei通过设备路径获取设备文件dev下的inode，读取文件根设备号，读取sp
	- namei获取mnt目录下挂载点文件inode
	- 将sp与根文件系统mnt目录挂载
- 打开文件：确定进程操作哪个文件
	- 将用户进程task struct中的filep与内核file table挂载
	- 用open namei根据用户提供的路径名找到用户要打开文件的inode
	- 将inode在file table中挂载
- 读文件sys read到file read
	- 调用bmp确定文件数据块在外设上的逻辑块号
	- 根据直接间接数据块找到需要读取的数据在外设中的位置
	- bread把数据块读入缓冲块
	- 数据复制到进程地址空间

****

**6、在创建进程、从硬盘加载程序、执行这个程序的过程中，sys_fork、do_execve、do_no_page分别起了什么作用？**
fork的由父进程创建一个子进程，根据返回值可以判断父子进程；execvec加载可执行文件，子进程将跳转执行可执行文件，在这一过程中由于cow机制，一旦需要写页面（数据段代码段栈段）则需要do no page辅助处理缺页中断，申请内存并重新设置页表
### 关于考试
- 四道大题，思考题全搞懂就没事