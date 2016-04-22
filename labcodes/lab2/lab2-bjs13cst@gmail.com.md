# Lab2 Report
---

## [练习0]
#### 练习0：填写已有实验。本实验依赖实验1。请把你做的实验1的代码填入本实验中代码中有“LAB1”的注释相应部分。

使用meld工具辅助完成

-----

## [练习1] 实现 first-fit 连续物理内存分配算法

#### 实现过程
在练习1中需要对`default_init`，`default_init_memmap`，`default_alloc_pages`，`default_free_pages`进行更改。

+ default_init函数
    
    完成对free_list的初始化工作，不需要对之前代码做更改
    
 + default_init_memmap函数
 
    对从struct Page* base开始的连续n个page进行初始化，将property设为0，同时设置PG_reserved和PG_property；
   
    因为一个free的block只需要把第一个page加入到list中，因此在设置完成各页后，只需将*base页的property置为n并加入到free_list中。
    
+ default_alloc_pages函数
    在list中找出first fit的block，因为list在插入过程中是按照地址大小排列的，因此只需要做一次循环即可。
    
    在找到block后将block中前n个page的PG_reserved和PG_property进行更改；
    
    如果block有多于n个page，需要把多出来的page重新整理成一个block再加入到list中，这里需要按照首页的地址寻找合适的rank再插入。
    
+ default_free_pages函数
    首先将base开始的n个page的PG_reserved和PG_property进行更改；
       
    之后需要判断是否存在可以和首页为base的block合并的block，如果存在进行合并；
    
    合并后将base插入list中，这里同样需要按照地址来寻找合适的rank。
    
#### 和参考答案的区别

参考答案中将block中的每一个page都加入了free_list，但实际上只需要把第一个page加入到list中就可以完成first-fit的分配，我的实现和参考答案在理解上不一样。

-----

## [练习2]实现寻找虚拟地址对应的页表项

#### 设计实现过程

+ 从虚拟地址中获取pde
+ 根据pde首地址读出内容
+ 根据pde的存在位(0x001)判断是否需要创建一个新的页
+ 如果需要创建则创建新页，并初始化
+ 从此页中获取所需的pte

#### 请描述页目录项（Pag Director Entry）和页表（Page Table Entry）中每个组成部分的含义和以及对ucore而言的潜在用处。

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

#### 如果ucore执行过程中访问内存，出现了页访问异常，请问硬件要做哪些事情？

硬件会中断当前指令，产生一个缺页中断，然后查询idt表获取该异常处理例程的入口地址，之后程序跳转到该入口处，开始处理页访问异常，在处理结束后硬件回到被中断的指令处继续执行。

#### 和参考答案区别

实现逻辑基本一致，在刚开始实现时没有考虑对alloc_page()失败的处理，查看参考答案后补充了该条语句。

-----

## [练习3]释放某虚地址所在的页并取消对应二级页表项的映射

#### 设计实现过程

+ 检查此pte对应页是否存在，若不存在直接退出
+ 通过pte获取该页，将该页引用减1
+ 若该页引用变为0，释放该页
+ 更新pte，刷新tlb

#### 数据结构Page的全局变量（其实是一个数组）的每一项与页表中的页目录项和页表项有无对应关系？如果有，其对应关系是啥？

有关系，Page每一项都对应一个物理页面，可以看做一个大的一级页表；页表中的每个页目录项和页表项也对应一个物理页面；对于Page每一项所对应的物理地址，其对应的虚拟地址高10位就是pde的index，之后10位是pte的index。

#### 如果希望虚拟地址与物理地址相等，则需要如何修改lab2完成此事？

取消目前lab2所采用的段映射，采用对等映射。

#### 和参考答案区别

实现逻辑基本一致，个别语句有差别

-----

#### 列出你认为本实验中重要的知识点，以及与对应的OS原理中的知识点

+ 连续物理内存分配的first-fit算法
+ 非连续物理内存分配算法中的页机制，采用二级页表


#### 列出你认为OS原理中很重要，但在实验中没有对应上的知识点

+ 无建议


