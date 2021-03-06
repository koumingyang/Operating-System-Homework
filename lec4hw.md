计54 寇明阳　2015011318

# lab1 SPOC思考题

## 思考题

### 启动顺序

1. 段寄存器的字段含义和功能有哪些？

分四个字段，CS，DS，ES和SS。CS是代码段，DS是数据段，SS是堆栈段，ES是附加段。
段寄存器CS指向存放程序的内存段，IP是用来存放下条待执行的指令在该段的偏移量，把它们合在一起可在该内存段内取到下次要执行的指令。
段寄存器SS指向用于堆栈的内存段，SP是用来指向该堆栈的栈顶，把它们合在一起可访问栈顶单元。另外，当偏移量用到了指针寄存器BP，则其缺省的段寄存器也是SS，并且用BP可访问整个堆栈，不仅仅是只访问栈顶。
段寄存器DS指向数据段，ES指向附加段，在存取操作数时，二者之一和一个偏移量合并就可得到存储单元的物理地址。该偏移量可以是具体数值、符号地址和指针寄存器的值等之一，具体情况将由指令的寻址方式来决定。
通常，缺省的数据段寄存器是DS，只有一个例外，即：在进行串操作时，其目的地址的段寄存器规定为ES。当然，在一般指令中，我们还可以通过改变前缀中的“段取代”字段来改变操作数的段寄存器。

2. 描述符特权级DPL、当前特权级CPL和请求特权级RPL的含义是什么？在哪些寄存器中存在这些字段？对应的访问条件是什么？

CPL是当前执行的程序或任务的特权级。它被存储在CS和SS的第0位和第1位上。通常情况下，CPL代表代码所在的段的特权级。当程序转移到不同特权级的代码段时，处理器将改变CPL。只有0和3两个值，分别表示用户态和内核态。
DPL表示段或门的特权级。它被存储在段描述符或者门描述符的DPL字段中，当当前代码段试图访问一个段或者门（这里大家先把门看成跟段一样，下面我们会介绍），DPL将会和CPL以及段或者门选择子的RPL相比较，根据段或者门类型的不同，DPL将会区别对待。
RPL是通过段选择子的第0和第1位表现出来的。RPL是代码中根据不同段跳转而确定，以动态刷新CS里的CPL，在代码段选择符中。而且RPL对每个段来说不是固定的，两次访问同一段时的RPL可以不同。操作系统往往用RPL来避免低特权级应用程序访问高特权级段内的数据，即便提出访问请求的段有足够的特权级，如果RPL不够也是不行的，当RPL的值比CPL大的时候，RPL将起决定性作用。也就是说，RPL相当于附加的一个权限控制，只有当RPL>DPL的时候，才起到实际的限制作用。

3. 分析可执行文件格式elf的格式（无需回答）

### 4.2 C函数调用的实现

### 4.4 x86中断处理过程

1. 中断处理中硬件压栈内容？用户态中断和内核态中断的硬件压栈有什么不同？

压栈内容为中断上下文。进程上下文，就是一个进程传递给内核的那些参数和CPU的所有寄存器的值、进程的状态以及堆栈中的内容，也就进程在进入内核态之前的运行环境。所以在切换到内核态时需要保存当前进程的所有状态，即保存当前进程的上下文，以便再次执行该进程时，能够恢复切换时的状态，继续执行。同理，硬件通过触发信号，导致内核调用中断处理程序，进入内核空间。这个过程中，硬件的一些变量和参数也要传递给内核，内核通过这些参数进行中断处理，中断上下文就可以理解为硬件传递过来的这些参数和内核需要保存的一些环境。
一个进程的上下文可以分为三个部分:用户级上下文、寄存器上下文以及系统级上下文。
（1）用户级上下文: 正文、数据、用户堆栈以及共享存储区；
（2）寄存器上下文: 通用寄存器、程序寄存器(IP)、处理器状态寄存器(EFLAGS)、栈指针(ESP)；
（3）系统级上下文: 进程控制块task_struct、内存管理信息(mm_struct、vm_area_struct、pgd、pte)、内核栈。
两种中断的中断上下文不同，内核态中断没有用户级上下文，同时两种中断的上下文内容也不一样。

2. 为什么在用户态的中断响应要使用内核堆栈？

因为中断响应的过程其实就是从用户态切换到内核态的过程，并保存中断上下文。而内核堆栈是用于内核态的堆栈，有更高的权限，保证用户态时无法操作内核中的数据。由于中断响应时切换到了内核态，因此需要使用内核堆栈。

3. trap类型的中断门与interrupt类型的中断门有啥设置上的差别？如果在设置中断门上不做区分，会有什么可能的后果?

中断门（Interrupt gate）
其类型码为110，中断门包含了一个中断或异常处理程序所在段的选择符和段内偏移量。当控制权通过中断门进入中断处理程序时，处理器清IF 标志，即关中断，以避免嵌套中断的发生。中断门中的DPL（Descriptor Privilege Level）为0，因此，用户态的进程不能访问Intel 的中断门。所有的中断处理程序都由中断门激活，并全部限制在内核态。
陷阱门（Trap gate）
其类型码为111，与中断门类似，其唯一的区别是，控制权通过陷阱门进入处理程序时维持IF 标志位不变，也就是说，不关中断。
可能导致在中断处理程序时出现中断，导致嵌套中断，出现系统崩溃。

### 4.8 练习四和五 ucore内核映像加载和函数调用栈分析

1. 在kdebug.c文件中用到的函数`read_ebp`是内联的，而函数`read_eip`不是内联的。为什么要设计成这样？

read_ebp设计成内联的可以加快程序运行速度，在编译时讲函数代码直接进行汇编、连接。read_eip不是内联的可能是因为防止程序出现变量冲突，未保存原先寄存器的值就修改，出现错误。

### 4.9 练习六 完善中断初始化和处理

1. CPU加电初始化后中断是使能的吗？为什么？

不是。在bootloader加载内核之后，操作系统才获得控制权，之后中断就是使能的。

## 开放思考题

1. 如何修改lab1, 实现在出现除零异常时显示一个字符串的异常服务例程？
2. 在lab1/bin目录下，通过`objcopy -O binary kernel kernel.bin`可以把elf格式的ucore kernel转变成体积更小巧的binary格式的ucore kernel。为此，需要如何修改lab1的bootloader, 能够实现正确加载binary格式的ucore OS？ (hard)
3. GRUB是一个通用的bootloader，被用于加载多种操作系统。如果放弃lab1的bootloader，采用GRUB来加载ucore OS，请问需要如何修改lab1, 能够实现此需求？ (hard)
4. 如果没有中断，操作系统设计会有哪些问题或困难？在这种情况下，能否完成对外设驱动和对进程的切换等操作系统核心功能？

## 课堂实践
### 练习一
在Linux系统的应用程序中写一个函数print_stackframe()，用于获取当前位置的函数调用栈信息。实现如下一种或多种功能：函数入口地址、函数名信息、参数调用参数信息、返回值信息。

### 练习二
在ucore内核中写一个函数print_stackframe()，用于获取当前位置的函数调用栈信息。实现如下一种或多种功能：函数入口地址、函数名信息、参数调用参数信息、返回值信息。
