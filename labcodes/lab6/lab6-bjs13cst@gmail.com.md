# Lab6 Report
---

## [练习0] 填写已有实验
### 本实验依赖实验1/2/3/4/5。请把你做的实验2/3/4/5的代码填入本实验中代码中有“LAB1”/“LAB2”/“LAB3”/“LAB4”“LAB5”的注释相应部分。并确保编译通过。注意：为了能够正确执行lab6的测试应用程序，可能需对已完成的实验1/2/3/4/5的代码进行进一步改进。

使用meld工具辅助完成

lab1: kern/debug/kdebug.c kern/trap/trap.c

lab2: kern/mm/default_pmm.c kern/mm/pmm.c   

lab3: kern/mm/vmm.c kern/mm/swap_fifo.c

lab4: kern/proc/process/proc.c

lab5: kern/proc/process/proc.c

-----

## [练习1] 使用 Round Robin 调度算法

#### 请理解并分析sched_calss中各个函数指针的用法，并接合Round Robin 调度算法描ucore的调度执行过程

+ RR_init: 初始化调度队列run_list;

+ RR_enqueue: 将进程加入到调度队列末尾, 初始化时间片长度;

+ RR_dequeue: 将进程从调度队列中删去;

+ RR_pick_next: 返回调度队列中最靠前的进程;

+ RR_proc_tick: 时钟中断时调用, 将当前时间片减一, 如果时间片已经为0, 则该进程已经可以被调度.

在每次时钟中断后, cuore会修改current进程的剩余时间片, 当剩余时间片为0时, 该进程进入可被调度状态.

在init函数中会调用cpu_idle函数, 该函数会判断当前进程是否需要被调度, 如果需要, 则调用schedule函数, 该函数会将current进程加入调度队列, 而将调度队列中的下一个进程取出执行.

#### 请在实验报告中简要说明如何设计实现”多级反馈队列调度算法“，给出概要设计

将ucore中的run_list改为数组, 记录多个不同的调度队列, 对不同的调度队列设置不同的时间片长度. 不同的调度队列也设置有不同的优先级, 在pick_next的时候优先从高优先级队列中选取.

-----

## [练习2] 实现 Stride Scheduling 调度算法

#### 设计实现过程

Stride Scheduling调度算法基本思路就是对于每个进程都记录一个步长stride和一个前进步数pass. 每次都选择当前步长最短的进程. 这样, stride就相当与这个进程的优先级.

在实现中, 会确定一个大常数. 在每次选择进程时, 都用这个大常数来除以当前进程的优先级. 需要注意的是大常数的合适选取可以方便步长的比较.

按照注释提示基本可以实现调度算法的各函数.