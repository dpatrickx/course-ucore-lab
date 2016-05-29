# Lab8 Report
---

## [练习0] 填写已有实验
### 本实验依赖实验1/2/3/4/5/6/7。请把你做的实验1/2/3/4/5/6/7的代码填入本实验中代码中
  
使用meld工具辅助完成

lab1: kern/debug/kdebug.c kern/trap/trap.c

lab2: kern/mm/default_pmm.c kern/mm/pmm.c   

lab3: kern/mm/vmm.c kern/mm/swap_fifo.c

lab4: kern/proc/process/proc.c

lab5: kern/proc/process/proc.c

lab6: kern/proc/process/proc.c kern/schedule/default_sched.c

lab7: kern/sync/check_sync.c kern/sync/monitor.c kern/process/proc.c

-----

## [练习1] 完成读文件操作的实现
### 首先了解打开文件的处理流程，然后参考本实验后续的文件读写操作的过程分析，编写在sfs_inode.c中sfs_io_nolock读文件中数据的实现代码。

需要填写sfs_inode.c中的sfs_io_nolock函数, 会用到两个已经实现的函数, 一个是sfs_bmap_load_nolock, 他可以根据内存中的block索引, 得到磁盘上的block号, 另一个是sfs_buf_op, 事实上是一个函数指针.

对于可写操作, 指向sfs_wbuf, 对于只读操作, 指向sfs_rbuf, 负责将buf中的内容写入, 或将内容读出, 写入bug当中.

**设计实现”UNIX的PIPE机制“的概要设方案**

PIPE机制即进程通过内存来进行交互, 可以看做进程对特定文件进行读写来交互, 因此只需要设计实现对该特殊文件的read/write/open/close函数即可.

-----

## [练习2] 完成基于文件系统的执行程序机制的实现
### 改写proc.c中的load_icode函数和其他相关函数，实现基于文件系统的执行程序机制。

与原有的代码几乎相同, 不同的地方在于原来从binary读入的数据, 改为由load_icode_read从文件系统读入.

**设计实现基于”UNIX的硬链接和软链接机制“的概要设方案**

一个软链接对应一个新的索引节点, 其中的块储存被链接文件的绝对路径;

一个硬链接对应着一个目录, 其中的编号就是被链接文件的数据块索引值.