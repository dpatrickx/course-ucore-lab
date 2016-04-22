# Lab4 Report
### 白家松 计33 2013011339
---

## [练习0] 填写已有实验
### 本实验依赖实验1/2/3。请把你做的实验1/2/3的代码填入本实验中代码中有“LAB1”,“LAB2”,“LAB3”的注释相应部分。

使用meld工具辅助完成

lab1: kern/debug/kdebug.c kern/trap/trap.c

lab2: kern/mm/default_pmm.c kern/mm/pmm.c   

lab3: kern/mm/vmm.c kern/mm/swap_fifo.c

-----

## [练习1] 分配并初始化一个进程控制块

#### alloc_proc函数（位于kern/process/proc.c中）负责分配并返回一个新的struct proc_struct结构，用于存储新建立的内核线程的管理信息。ucore需要对这个结构进行最基本的初始化，你需要完成这个初始化过程。

+ 设计思路

    按照注释对proc_struct各成员变量进行初始化即可
    
+ 请说明proc_struct中struct context context和struct trapframe *tf成员变量含义和在本实验中的作用是啥？

    struct context context在进程切换时，保存上下文，用于恢复被中断进程运行现场；
    struct trapframe tf用于在同一进程进行内核态和用户态切换时，保存中断现场。
    
-----

## [练习2] 为新创建的内核线程分配资源

+ 实现过程
    
    按照注释提示，依次进行struct proc_struct的分配及复制等操作。
    
+ 请说明ucore是否做到给每个新fork的线程一个唯一的id？请说明你的分析和理由。

    可以。get_pid可以做到遍历全局进程链表，来确保分配到一个不和之前任一进程重复的pid。
    
-----

## [练习3] 理解 proc_run 函数和它调用的函数如何完成进程切换的。

+ 代码分析

    在函数中首先会判断当前进程是否是proc，如果不是，再进行进程切换，切换部分代码如下；和参考答案逻辑相同，但开始写的时候没有过多考虑屏蔽中断/分配失败等问题。
    
``` c++
local_intr_save(intr_flag);
{
    current = proc;
    load_esp0(next->kstack + KSTACKSIZE);
    lcr3(next->cr3);
    switch_to(&(prev->context), &(next->context));
}
```

    首尾两句话完成了中断屏蔽和中断恢复。
    
    切换逻辑包括将当前进程置为proc，改变esp，改变cr3寄存器，调用switch_to函数进行上下文切换。

+ 在本实验的执行过程中，创建且运行了几个内核线程？
    
    创建了两个内核线程，在proc_init中创建了idleproc和initproc。
    
+ 语句```local_intr_save(intr_flag);....local_intr_restore(intr_flag);```在这里有何作用?请说明理由。

    local_intr_save屏蔽中断，local_intr_restore恢复中断。用在这里是因为在进程切换时需要屏蔽中断，否则在做保护和恢复寄存器时会造成混乱导致系统崩溃。
    
-----

###  重要的知识点

进程切换，进程运行现场保存。
