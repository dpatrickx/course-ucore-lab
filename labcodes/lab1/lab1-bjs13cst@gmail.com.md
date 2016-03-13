# Lab1 Report
### 白家松 计33 2013011339
---

## [练习1]
#### [练习1.1] 操作系统镜像文件 ucore.img 是如何一步一步生成的?(需要比较详细地解释 Makefile 中 每一条相关命令和命令参数的含义,以及说明命令导致的结果)

**ucore.img生成过程**

执行命令make -n和make "V="，得到make运行时所执行的命令如下

**cc kern/init/init.c**

对于所有c和汇编文件使用gcc编译命令如下
```
gcc -Ikern/init/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/
-Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/init/init.c -o obj/kern/init/init.o
```
+ 编译时包含目录：
    + kern/init/
    + kern/debug/
    + kern/mm/
    + kern/trap/
    + libs/
+ -fno-builtin：禁止使用不是两个下划线开头的内建函数
+ -fno-stack-protector：禁止使用gcc的堆栈保护
+ -Wall：打开所有警告
+ -ggdb：添加gdb调试信息
+ -m32：32位编译
+ -gstabs ：输出stabs 格式的调试信息
+ -nostdinc ：不在系统默认的头文件目录搜索
+ -c：指定源文件名
+ -o：指定输出程序名

其余c和汇编程序的编译类似，包括：

+ cc kern/iibs/readline.c
+ cc kern/libs/readline.c
+ cc kern/libs/stdio.c
+ cc kern/debug/kdebug.c
+ cc kern/debug/kmonitor.c
+ cc kern/debug/panic.c
+ cc kern/driver/clock.c
+ cc kern/driver/console.c
+ cc kern/driver/intr.c+ cc kern/driver/picirq.c
+ cc kern/trap/trap.c
+ cc kern/trap/trapentry.S
+ cc kern/trap/vectors.S
+ cc kern/mm/pmm.c
+ cc libs/printfmt.c
+ cc libs/string.c
+ cc boot/bootasm.S
+ cc boot/bootmain.c

**ld bin/kernel**

将全部object文件链接为bin/kernel
```
ld -m    elf_i386 -nostdlib -T tools/kernel.ld -o bin/kernel  obj/kern/init/init.o
obj/kern/libs/readline.o obj/kern/libs/stdio.o obj/kern/debug/kdebug.o
obj/kern/debug/kmonitor.o obj/kern/debug/panic.o
obj/kern/driver/clock.o obj/kern/driver/console.o obj/kern/driver/intr.o
obj/kern/driver/picirq.o
obj/kern/trap/trap.o obj/kern/trap/trapentry.o obj/kern/trap/vectors.o
obj/kern/mm/pmm.o  obj/libs/printfmt.o obj/libs/string.o
```
其中参数含义如下：
+ -m elf_i386：设置机器类型为elf_i386
+ -nostdlib：不使用标准库文件进行链接
+ -T tools/kernel.ld：使用tools/kernel.ld作为链接脚本
+ -o：指定输出程序名

**ld bin/bootblock**
```
ld -m elf_i386 -nostdlib -N -e start -Ttext 0x7C00 obj/boot/bootasm.o
obj/boot/bootmain.o -o obj/bootblock.o
```
将obj/boot/bootasm.o和obj/boot/bootasm.o链接为obj/bootblock.o。

额外参数意义为：

+ -N： 将text和data段设置为可读写权限，不对data数据段进行页对齐
+ -e start ：指定入口地址为start
+ -Ttext 0x7C00 :使用0x7C00为text代码段的起始地址

**objcopy -S -O binary obj/bootblock.o obj/bootblock.out**

使用objcopy工具将bootblock.o改写为bootblock.out可执行文件。

+ -S ：不拷贝符号信息和重定位信息
+ -O binary ：输出格式为可执行文件

**bin/sign obj/bootblock.out bin/bootblock**

使用前一阶段编译的sign工具将bootblock.out改写为符合规范的引导扇区。

sign工具检查bootblock.out的大小是否小于510字节，若大于则报错退出。 若小于则写入到一个512字节的数组中，在最后两字节写入0xAA55。最后将512字节大小的引导扇区写入bin/bootblock文件。

 **dd if=/dev/zero of=bin/ucore.img count=10000**
 
使用dd工具生成一个5000KB的的空白镜像ucore.img。

**dd if=bin/bootblock of=bin/ucore.img conv=notrunc**

将编译好的bootblock文件写入到ucore.img其实部分，conv=notrunc表示若bootblock大小大于ucore.img，不截断内容。

**dd if=bin/kernel of=bin/ucore.img seek=1 conv=notrunc**

将编译好的kernal文件写入到ucore.img起始部分的512字节后（即bootloader后），seek表示从输出文件开头跳过1个block（512字节）后开始复制。


#### [练习1.2] 一个被系统认为是符合规范的硬盘主引导扇区的特征是什么?
查看tools/sign.c的代码
``` c++
char buf[512];
buf[510] = 0x55;
buf[511] = 0xAA;
```
可以看到硬盘主引导扇区大小为512byte；
其中第510个（倒数第二个）字节是0x55，第511个（倒数第一个）字节是0xAA。
---

## [练习2]
#### [练习2.1] 从 CPU 加电后执行的第一条指令开始,单步跟踪 BIOS 的执行。
1 启动终端，用以下命令在lab1目录下启动qemu
``` c++
qemu -S -s -hda bin/ucore.img -monitor stdio
```
2 在另一个终端中启动gdb，并远程连接到qemu
```
(gdb) target remote:1234
Remote debugging using :1234
0x0000fff0 in ?? ()
```
3 在qemu调试界面中执行如下命令来查看当前指令
```
(qemu) x /6i $pc
0xfffffff0:  ljmp   $0xf000,$0xe05b
0xfffffff5:  xor    %dh,0x322f
0xfffffff9:  xor    (%bx),%bp
0xfffffffb:  cmp    %di,(%bx,%di)
0xfffffffd:  add    %bh,%ah
0xffffffff:  add    %al,(%bx,%si)
```
4 在gdb终端中执行如下命令来跟踪BIOS的执行
```
si
```
5 再在qemu终端中查看当前指令，发现执行指令已经变化，即第一条指令跳转到的地方
```
(qemu) x /6i $pc
0x000fe05b:  cmpl   $0x0,%cs:0x6574
0x000fe062:  jne    0xfd2b6
0x000fe066:  xor    %ax,%ax
0x000fe068:  mov    %ax,%ss
0x000fe06a:  mov    $0x7000,%esp
0x000fe070:  mov    $0xf3c24,%edx
```

#### [练习2.2]在初始化位置0x7c00设置实地址断点,测试断点正常。
1 启动qemu与gdb，并建立远程连接
2 在gdb终端设置断点，并执行
```
(gdb) b *0x7c00
Breakpoint 1 at 0x7c00
(gdb) c
Continuing.
```
3 执行完毕，在qemu终端查看当前指令
```  
(qemu) x /10i $pc
0x00007c00:  cli    
0x00007c01:  cld    
0x00007c02:  xor    %ax,%ax
0x00007c04:  mov    %ax,%ds
0x00007c06:  mov    %ax,%es
0x00007c08:  mov    %ax,%ss
0x00007c0a:  in     $0x64,%al
0x00007c0c:  test   $0x2,%al
0x00007c0e:  jne    0x7c0a
0x00007c10:  mov    $0xd1,%al
```
可以正常运行到断点，测试完成
#### [练习2.3]从0x7c00开始跟踪代码运行,将单步跟踪反汇编得到的代码与bootasm.S和 bootblock.asm进行比较。
使用单步跟踪所得代码如下
``` assembly
(qemu) x /10i $pc
0x00007c00:  cli    
0x00007c01:  cld    
0x00007c02:  xor    %ax,%ax
0x00007c04:  mov    %ax,%ds
0x00007c06:  mov    %ax,%es
0x00007c08:  mov    %ax,%ss
0x00007c0a:  in     $0x64,%al
0x00007c0c:  test   $0x2,%al
0x00007c0e:  jne    0x7c0a
0x00007c10:  mov    $0xd1,%al
```
bootasm.s相应代码如下
``` assembly
start:
.code16                                             # Assemble for 16-bit mode
    cli                                             # Disable interrupts
    cld                                             # String operations increment

    # Set up the important data segment registers (DS, ES, SS).
    xorw %ax, %ax                                   # Segment number zero
    movw %ax, %ds                                   # -> Data Segment
    movw %ax, %es                                   # -> Extra Segment
    movw %ax, %ss                                   # -> Stack Segment

    # Enable A20:
    #  For backwards compatibility with the earliest PCs, physical
    #  address line 20 is tied low, so that addresses higher than
    #  1MB wrap around to zero by default. This code undoes this.
seta20.1:
    inb $0x64, %al                                  # Wait for not busy(8042 input buffer empty).
    testb $0x2, %al
    jnz seta20.1

    movb $0xd1, %al                                 # 0xd1 -> port 0x64
```
bootblock.asm相应部分代码如下
```
start:
.code16                                             # Assemble for 16-bit mode
    cli                                             # Disable interrupts
    7c00:	fa                   	cli    
    cld                                             # String operations increment
    7c01:	fc                   	cld    

    # Set up the important data segment registers (DS, ES, SS).
    xorw %ax, %ax                                   # Segment number zero
    7c02:	31 c0                	xor    %eax,%eax
    movw %ax, %ds                                   # -> Data Segment
    7c04:	8e d8                	mov    %eax,%ds
    movw %ax, %es                                   # -> Extra Segment
    7c06:	8e c0                	mov    %eax,%es
    movw %ax, %ss                                   # -> Stack Segment
    7c08:	8e d0                	mov    %eax,%ss
```
比较可以发现，反汇编所得代码比较简洁；bootasm.s中带有一些注释；而bootblock.asm中不仅带有注释，还有代码所在的位置，与反汇编所得位置一致。

+ 需要注意的是，在bootblock.asm中call bootmain的地址是`0x7cd1`；而在第一次反汇编中得到地址不一致。在观察makefile代码后，发现如果将qemu启动公式设为`qemu -S -s -hda bin/ucore.img -parallel stdio`，即让ucore在qemu模拟的x86硬件环境中执行，则所得结果也为`0x7cd1`。

#### [练习2.4]自己找一个bootloader或内核中的代码位置，设置断点并进行测试。
在gdb终端中载入./bin/kernel，设置断点memset，执行，得到如下结果
```
(gdb) b memset
Breakpoint 1 at 0x1030b3: file libs/string.c, line 273.
(gdb) r
Starting program: /home/dpatrickx/desktop/lab/labcodes/lab1/bin/kernel 

Breakpoint 1, memset (s=0x10ea16, c=0 '\000', n=4874) at libs/string.c:273
273	    return __memset(s, c, n);
```
---

## 练习３
#### BIOS将通过读取硬盘主引导扇区到内存，并转跳到对应内存中的位置执行bootloader。请分析bootloader是如何完成从实模式进入保护模式的。

**1. 为何开启A20，以及如何开启A20?**

不开启A20只能使用20根地址总线，而开启后寻址能力可以达到4GB；
开启方法：查看0x64端口，如果不忙，则把0xd1写入0x64端口，把0xdf写入0x60端口

**2. 如何初始化GDT表?**

直接载入即可

``` assembler
lgdt gdtdesc
...
gdtdesc:
    .word 0x17                                      # sizeof(gdt) - 1
    .long gdt                                       # address gdt

```

**3. 如何使能和进入保护模式？**

``` assembler
movl %cr0, %eax
orl $CR0_PE_ON, %eax
movl %eax, %cr0
```
通过将cr0寄存器最后一位置1来开启保护模式

``` assembler
ljmp $PROT_MODE_CSEG, $protcseg
```

然后跳转到保护模式段执行

---

## 练习4
#### 通过阅读bootmain.c，了解bootloader如何加载ELF文件。通过分析源代码和通过qemu来运行并调试bootloader&OS

**1. bootloader如何读取硬盘扇区的？**

读取硬盘扇区通过readsect函数完成，函数接收一个指针和一个无符号整数secno，执行将secno对应的扇区的内容拷贝至指针处。代码如下，分析如注释：
``` c
static void
readsect(void *dst, uint32_t secno) {
    // 等待磁盘准备好
    waitdisk();

    outb(0x1F2, 1);                                      // 向0x1F2输出1，即要读取扇区数目
    outb(0x1F3, secno & 0xFF);                  // 向0x1F3输出secno的0~7位
    outb(0x1F4, (secno >> 8) & 0xFF);       // 向0x1F4输出secno的8~15位
    outb(0x1F5, (secno >> 16) & 0xFF);      // 向0x1F5输出secno的16~23位
    outb(0x1F6, ((secno >> 24) & 0xF) | 0xE0);  // 向0x1F6输出1110+secno的24~27位
    outb(0x1F7, 0x20);                                  // 向0x1F7输出0x20，表示读数据

    // 等待磁盘准备好
    waitdisk();

    // 调用insl函数将磁盘数据拷贝至内存指针位置
    insl(0x1F0, dst, SECTSIZE / 4);
}
```
其中insl函数代码如下：
``` c
insl(uint32_t port, void *addr, int cnt) {
    asm volatile (
            "cld;"              // 复位方向标志位
            "repne; insl;"  // 反复执行insl指令，每次拷贝32bit的内容
            : "=D" (addr), "=c" (cnt)
            : "d" (port), "0" (addr), "1" (cnt)
            : "memory", "cc");
}
```
**2. bootloader是如何加载ELF格式的OS？**

bootloader通过bootmain函数完成对ELF格式的加载，代码如下
``` c
void
bootmain(void) {
    // 1. 从kernel头读取8个扇区至ELFHEDR处
    readseg((uintptr_t)ELFHDR, SECTSIZE * 8, 0);
    // 2. 判断开头是否合法（开头32位等于ELF_MAGIC）
    if (ELFHDR->e_magic != ELF_MAGIC) {
        goto bad;
    }
    
    struct proghdr *ph, *eph;
    // 3. 如果合法，从ELFHDR + ELFHDR->e_phoff读出section header
    ph = (struct proghdr *)((uintptr_t)ELFHDR + ELFHDR->e_phoff);
    eph = ph + ELFHDR->e_phnum;
    // 4. 循环调用readseg函数，从ELF文件中读出程序片段
    for (; ph < eph; ph ++) {
        readseg(ph->p_va & 0xFFFFFF, ph->p_memsz, ph->p_offset);
        }
    // 5. 读取入口函数指针，跳转进入OS
    ((void (*)(void))(ELFHDR->e_entry & 0xFFFFFF))();

bad:
    outw(0x8A00, 0x8A00);
    outw(0x8A00, 0x8E00);
    while (1);
}
```

---

## 练习5

#### 我们需要在lab1中完成kdebug.c中函数print_stackframe的实现，可以通过函数print_stackframe来跟踪函数调用堆栈中记录的返回地址。

输出中，堆栈最深层为
```
ebp:0x00007bf8 eip:0x00007d68 args:0xc031fcfas 0xc08ed88es 0x64e4d08es 0xfa7502a8s 
    <unknow>: -- 0x00007d67 --
```

%ebp 0x7bf8的地址对应的是bootmain函数入口0x7c00之前，args中的数值均为指令。

---

## 练习6

#### 请完成编码工作和回答如下问题：

**1. 中断描述符表（也可简称为保护模式下的中断向量表）中一个表项占多少字节？其中哪几位代表中断处理代码的入口？**

中断向量表一个表项占8个字节，从高到低表示为 Byte7 ~ Byte0，

Byte7, Byte6 表示中断处理代码入口的高 16 位

Byte1, Byte0 表示中断处理代码入口的低 16 位

Byte3, Byte2 表示中断处理代码入口的段选择子

**2. 请编程完善kern/trap/trap.c中对中断向量表进行初始化的函数idt_init。在idt_init函数中，依次对所有中断入口进行初始化。使用mmu.h中的SETGATE宏，填充idt数组内容。每个中断的入口由tools/vectors.c生成，使用trap.c中声明的vectors数组即可**

参见代码

**3. 请编程完善trap.c中的中断处理函数trap，在对时钟中断进行处理的部分填写trap函数中处理时钟中断的部分，使操作系统每遇到100次时钟中断后，调用print_ticks子程序，向屏幕上打印一行文字”100 ticks”。**

参见代码

---
