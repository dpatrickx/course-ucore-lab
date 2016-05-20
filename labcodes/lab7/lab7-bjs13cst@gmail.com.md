# Lab6 Report
---

## [练习0] 填写已有实验
### 本实验依赖实验1/2/3/4/5/6。请把你做的实验1/2/3/4/5/6的代码填入本实验中代码中有“LAB1”/“LAB2”/“LAB3”/“LAB4”/“LAB5”/“LAB6”的注释相应部分。并确保编译通过。

使用meld工具辅助完成

lab1: kern/debug/kdebug.c kern/trap/trap.c

lab2: kern/mm/default_pmm.c kern/mm/pmm.c   

lab3: kern/mm/vmm.c kern/mm/swap_fifo.c

lab4: kern/proc/process/proc.c

lab5: kern/proc/process/proc.c

lab6: kern/proc/process/proc.c kern/schedule/default_sched.c

-----

## [练习1] 理解内核级信号量的实现和基于内核级信号量的哲学家就餐问题

**内核级信号量的设计描述**

基于semaphore_t实现, semaphore_t包含两部分: 记录资源数量的value和记录等待资源的队列.

``` c++
typedef struct {
    int value;
    wait_queue_t wait_queue;
} semaphore_t;
```

信号量相关操作有三个: up, down, try_down.

其中up操作给对应信号量加一, 并且取出等待队列中最靠前的进程并唤醒; 

down操作会判断信号量资源是否为正, 若为正, 则将其减一并返回, 否则使进程进入等待状态并加入到等待队列中;

try_down操作和down操作类似, 区别在于如果信号量资源不为正, try_down操作会直接返回.

**用户态进程/线程提供信号量机制的设计方案**

可以对内核态信号量机制进行封装来实现用户态信号量机制, 即用sem_init来获取一个内核态信号量的id, 用wait/push操作来通过系统调用调用内核态信号量的down和up操作即可.

**哲学家问题信号量实现**

定义有一个信号量数组, 每个信号量对应一个哲学家. 每个哲学家可以进行两个操作: phi_take_forks_sema(拿筷子)和phi_put_forks_sema(放筷子).

phi_take_forks_sema操作会将哲学家状态标为HUNGRY, 并且尝试获得筷子; 如果两边的筷子都可以获得, 哲学家状态会标注为EATING并up信号量, 尝试结束后会down自己的信号量, 如果获得了筷子, 信号量资源就会释放, 进程继续运行, 否则进程就会进入等待队列;

phi_put_forks_sema操作会将哲学家的筷子放下, 并把其状态标为THINKING, 再提醒相邻哲学家去尝试获得筷子.

-----

## [练习2] 完成内核级条件变量和基于内核级条件变量的哲学家就餐问题

**内核级条件变量的设计描述**

一个管程会保持一个保护临界区的信号量mutex, 一个信号量next用于唤醒在等待队列中的进程, 一个32位整数next_count来记录等待进程的个数, 以及一个条件变量的指针cv用于管理该管程下的条件变量.

``` c++
typedef struct monitor{
    semaphore_t mutex; 
    semaphore_t next;
    int next_count;
    condvar_t *cv;
} monitor_t;
```

条件变量包括一个信号量sem, 一个记录等待状态的进程数量的32位整数count, 和一个管程指针owner, 指向持有自己的管程. 条件变量相关操作有两个: cond_signal和cond_wait.

cond_signal会使得对应条件变量中的条件量的资源加1, 同时让next资源减1;

cond_wait会检查是否有处于等待状态的进程, 若有则将next资源加1并将该进程加入调度.

**用户态进程/线程提供条件变量机制的设计方案**

利用内核态条件变量进行封装, 使用get_cond得到内核态中对应条件变量的id, 使用signal和wait通过系统调用来调用内核态下的cond_signal和cond_wait.

**哲学家问题条件变量实现**

同上, 哲学家有两个操作: phi_take_forks_sema和phi_put_forks_sema.

phi_take_forks_sema操作会将哲学家状态标为HUNGRY, 并且尝试获得筷子; 如果两边的筷子都可以获得, 哲学家状态会标注为EATING并signal自己的条件变量, 尝试结束后会wait自己的条件变量, 如果获得了筷子, 进程继续运行, 否则进程就会进入等待队列;

phi_put_forks_sema操作会将哲学家的筷子放下, 并把其状态标为THINKING, 再提醒相邻哲学家去尝试获得筷子.