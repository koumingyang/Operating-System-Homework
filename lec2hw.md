计54 寇明阳 2015011318

## 思考题

- 你理解的对于类似ucore这样需要进程/虚存/文件系统的操作系统，在硬件设计上至少需要有哪些直接的支持？至少应该提供哪些功能的特权指令？

支持中断、用户模式与特权模式、支持访问虚存和文件系统、支持多进程。

需要提供的特权指令：
1. 修改虚存的段表和页表
2. 允许和禁止中断，控制中断禁止屏蔽位
3. 在进程间切换处理
4. 存取用于主存保护的寄存器
5. 执行I/O操作
6. 停止一个中央处理器的工作
7. 清理内存
8. 设置时钟

- 你理解的x86的实模式和保护模式有什么区别？物理地址、线性地址、逻辑地址的含义分别是什么？

1. 可访问的物理内存空间大小不同，实模式下可访问的物理内存空间不超过1MB，在保护方式下，全部32条地址线有效，可寻址高达4G字节的物理地址空间。
2. 保护模式和实模式的根本区别是进程内存是否受保护，实模式将整个物理内存看成分段的区域,程序代码和数据位于不同区域，系统程序和用户程序没有区别对待，而且每一个指针都是指向"实在"的物理地址。这样一来，用户程序的一个指针如果指向了系统程序区域或其他用户程序区域，并改变了值，那么对于这个被修改的系统程序或用户程序，其后果就很可能是灾难性的。为了克服这种低劣的内存管理方式，处理器厂商开发出保护模式。这样，物理内存地址不能直接被程序访问，程序内部的地址（虚拟地址）要由操作系统转化为物理地址去访问，程序对此一无所知。
3. CPU启动环境为16位实模式，之后切换到保护模式，这主要是因为下x86需要向下兼容。
* 物理地址：是处理器提交到总线上用于访问计算机系统中的内存和外设的最终地址。一个计算机系统中只有一个物理地址空间。
* 逻辑地址：在有地址变换功能的计算机中,访问指令给出的地址叫逻辑地址。
* 线性地址：线性地址是逻辑地址到物理地址变换之间的中间层，处理器通过段(Segment)机制控制下的形成的地址空间。

- 理解list_entry双向链表数据结构及其4个基本操作函数和ucore中一些基于它的代码实现（此题不用填写内容）

- 对于如下的代码段，请说明":"后面的数字是什么含义
```
 /* Gate descriptors for interrupts and traps */
 struct gatedesc {
    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment
    unsigned gd_ss : 16;            // segment selector
    unsigned gd_args : 5;            // # args, 0 for interrupt/trap gates
    unsigned gd_rsv1 : 3;            // reserved(should be zero I guess)
    unsigned gd_type : 4;            // type(STS_{TG,IG32,TG32})
    unsigned gd_s : 1;                // must be 0 (system)
    unsigned gd_dpl : 2;            // descriptor(meaning new) privilege level
    unsigned gd_p : 1;                // Present
    unsigned gd_off_31_16 : 16;        // high bits of offset in segment
 };
```

每一个成员变量的field(域)在struct(结构体)中所占的位(bit)数。

- 对于如下的代码段，

```
#define SETGATE(gate, istrap, sel, off, dpl) {            \
    (gate).gd_off_15_0 = (uint32_t)(off) & 0xffff;        \
    (gate).gd_ss = (sel);                                \
    (gate).gd_args = 0;                                    \
    (gate).gd_rsv1 = 0;                                    \
    (gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;    \
    (gate).gd_s = 0;                                    \
    (gate).gd_dpl = (dpl);                                \
    (gate).gd_p = 1;                                    \
    (gate).gd_off_31_16 = (uint32_t)(off) >> 16;        \
}
```
如果在其他代码段中有如下语句，
```
unsigned intr;
intr=8;
SETGATE(intr, 1,2,3,0);
```
请问执行上述指令后， intr的值是多少？

0x10002

### 课堂实践练习

#### 练习一

请在ucore中找一段你认为难度适当的AT&T格式X86汇编代码，尝试解释其含义。

  - [Intel格式和AT&T格式汇编区别](http://www.cnblogs.com/hdk1993/p/4820353.html)

  - ##### [x86汇编指令集  ](http://hiyyp1234.blog.163.com/blog/static/67786373200981811422948/)

  - ##### [PC Assembly Language, Paul A. Carter, November 2003.](https://pdos.csail.mit.edu/6.828/2016/readings/pcasm-book.pdf)

  - ##### [*Intel 80386 Programmer's Reference Manual*, 1987](https://pdos.csail.mit.edu/6.828/2016/readings/i386/toc.htm)

  - ##### [[IA-32 Intel Architecture Software Developer's Manuals](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)]

#### 练习二

宏定义和引用在内核代码中很常用。请枚举ucore中宏定义的用途，并举例描述其含义。

 > 利用宏进行复杂数据结构中的数据访问；
 > 利用宏进行数据类型转换；如 to_struct,
 > 常用功能的代码片段优化；如  ROUNDDOWN, SetPageDirty
