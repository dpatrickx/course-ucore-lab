# Lab5 Report
---

## [练习0] 填写已有实验
### 本实验依赖实验1/2/3/4。请把你做的实验1/2/3/4的代码填入本实验中代码中有“LAB1”/“LAB2”/“LAB3”/“LAB4”的注释相应部分。

使用meld工具辅助完成

lab1: kern/debug/kdebug.c kern/trap/trap.c

lab2: kern/mm/default_pmm.c kern/mm/pmm.c   

lab3: kern/mm/vmm.c kern/mm/swap_fifo.c

lab4: kern/proc/process/proc.c

-----

## [练习1] 加载应用程序并执行

#### 设计思路

按照注释修改load_icode函数，和答案基本一致。

#### 描述当创建一个用户态进程并加载了应用程序后，CPU是如何让这个应用程序最终在用户态执行起来的。即这个用户态进程被ucore选择占用CPU执行（RUNNING态）到具体执行应用程序第一条指令的整个经过。

cpu会调用do_execve函数，在函数中完成对cr3寄存器的赋值、对旧的内存的清除，并调用函数load_icode函数。

load_icode函数会从ELF格式的文件中获取新process的内容；该函数会进行内存的分配及映射、代码段和数据段的拷贝、用户态堆栈的创建、中断帧的修改等工作。

之后，进程就开始被调度。CPU选择调度进程的话，会通过```proc_run```来唤醒进程，在```proc_run```函数中会调用```switch_to```函数，在```switch_to```函数最后，将context.eip压入栈中，在```switch_to```函数运行结束后，会执行```forkret```函数，执行汇编函数```forkrets```，```forkrets```函数会将proc的trapframe作为参数，然后跳转到__trapret。在返回过程中，就跳到了代码入口。

``` assembly
# switch_to()
movl 4(%esp), %eax          # not 8(%esp): popped return address already
...
pushl 0(%eax)  
```

``` assembly
# forkrets
# set stack to this new process's trapframe
movl 4(%esp), %esp
jmp __trapret
```

-----

## [练习2] 父进程复制自己的内存空间给子进程

#### 设计思路

基本按照注释的提示来实现即可，需要对lab1/lab2/lab4的代码进行一些更改，其中在trap.c中要注释掉```count_ticks()```函数中才能成功```make grade```；实现和答案逻辑一致。

#### 简要说明如何设计实现”Copy on Write 机制“，给出概要设计。

对于用户的读操作，直接分配给用户访问该数据的指针；而对于用户的写操作，则重新申请一个页面，并将用户申请的页面的内容拷贝到新申请的页面中，将新页面的指针返回给用户。

在多个用户同时进行修改时可能会存在死锁问题，可以加入lock机制。

-----

## [练习3] 阅读分析源代码，理解进程执行 fork/exec/wait/exit 的实现，以及系统调用的实现

#### 简要说明对 fork/exec/wait/exit函数的分析

#### 请分析fork/exec/wait/exit在实现中是如何影响进程的执行状态的？

#### 请给出ucore中一个用户态进程的执行状态生命周期图

摘自proc.c

```
process state changing:
                                            
  alloc_proc                                 RUNNING
      +                                   +--<----<--+
      +                                   + proc_run +
      V                                   +-->---->--+ 
PROC_UNINIT -- proc_init/wakeup_proc --> PROC_RUNNABLE -- try_free_pages/do_wait/do_sleep --> PROC_SLEEPING --
                                           A      +                                                           +
                                           |      +--- do_exit --> PROC_ZOMBIE                                +
                                           +                                                                  + 
                                           -----------------------wakeup_proc----------------------------------
```

-----

## 本实验中重要的知识点

cpu加载进程；

进程切换；

进程不同状态间转换。