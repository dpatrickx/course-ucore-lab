# Lab3 Report

---

## [练习0] 填写已有实验
#### 本实验依赖实验1/2。请把你做的实验1/2的代码填入本实验中代码中有“LAB1”,“LAB2”的注释相应部分。

使用meld工具辅助完成

lab1: kern/debug/kdebug.c kern/trap/trap.c

lab2: kern/mm/default_pmm.c kern/mm/pmm.c   

## [练习1] 给未被映射的地址映射上物理页
#### 完成do_pgfault（mm/vmm.c）函数，给未被映射的地址映射上物理页。设置访问权限 的时候需要参考页面所在 VMA 的权限，同时需要注意映射物理页时需要操作内存控制 结构所指定的页表，而不是内核的页表。

+ 设计思路

    调用get_pte函数后，对结果进行判断，如果不存在物理页，则调用`pgdir_alloc_page`函数来分配页。

+ 请描述页目录项（Pag Director Entry）和页表（Page Table Entry）中组成部分对ucore实现页替换算法的潜在用处。

     页目录项指向用来存储页表的页，其本质与页表相同。
     
     PDE高20位表示下一级页表的基址;

    PTE高20位表示物理页表的基址；

    后12位是标志位，如下所示
``` c++
/* page table/directory entry flags */
#define PTE_P           0x001                       // Present
#define PTE_W           0x002                     // Writeable
#define PTE_U           0x004                     // User
#define PTE_PWT         0x008                  // Write-Through
#define PTE_PCD         0x010                   // Cache-Disable
#define PTE_A           0x020                   // Accessed
#define PTE_D           0x040                   // Dirty
#define PTE_PS          0x080                   // Page Size
#define PTE_MBZ         0x180                   // Bits must be zero
#define PTE_AVAIL       0xE00                   // Available for software use
                                                // The PTE_AVAIL bits aren't used by the kernel or interpreted by the
                                                // hardware, so user processes are allowed to set them arbitrarily.
```
    从低地址向高依次为存在位、可写位、用户可用位等。
    
+ 如果ucore的缺页服务例程在执行过程中访问内存，出现了页访问异常，请问硬件要做哪些事情？
    
    硬件会中断当前指令，产生一个缺页中断，将一些寄存器的值还有错误码压入栈中，并查询idt表获取该异常处理例程的入口地址，之后程序跳转到该入口处，开始处理页访问异常，在处理结束后硬件回到被中断的指令处继续执行。

+ 和参考答案区别

     实现思路一致。但我没有考虑到对一些函数调用失败的处理，查看答案后做了添加。

## [练习2] 补充完成基于FIFO的页面替换算法
#### 完成vmm.c中的do_pgfault函数，并且在实现FIFO算法的swap_fifo.c中完成map_swappable和swap_out_vistim函数。通过对swap的测试。

+ 设计思路

    map_swappable函数中，直接将entry项插入到head项之前
    
    因为在队列初始化时满足`head->prev=head->next=head`，因此该队列在进行若干`add`操作后成环，而在`swap_out_vistim`函数中，只需要根据head找到最早添加项的指针即可。

+ 如果要在ucore上实现"extended clock页替换算法"请给你的设计方案，现有的swap_manager框架是否足以支持在ucore中实现此算法？如果是，请给你的设计方案。

    可行。因为目前队列结构已成环，因此只需要在FIFO算法基础上对`map_swappable`和`swap_out_vistim`进行修改即可。
    
    算法中所要求的访问位和修改位在页表项中已经存在，直接访问即可。而该算法需要一个指向某项的指针，可以在`struct mm`结构中直接添加。
    
    + 需要被换出的页的特征是什么？
    
        访问位和修改位均为0
        
    + 在ucore中如何判断具有这样特征的页？
    
        判断页表项`PTE_A`和`PTE_D`位
        
    + 何时进行换入和换出操作？
        
        换入操作：发生缺页异常，且ucore中`swap_init_OK`不为零
        
        换出操作：当物理内存接近被占满时会将一部分页换出
        
+ 和参考答案区别

    列表加入方向不一样；此外答案中以assert形式考虑了函数执行失败的情况。
    
-----

### 重要的知识点

多级页表机制，页的换入换出操作

###

