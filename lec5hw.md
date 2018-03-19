
## 第五讲 “连续内存分配”与视频相关的课堂练习

计54 寇明阳 2015011318

### 5.1 计算机体系结构和内存层次
#### 1.操作系统中存储管理的目标是什么？
抽象、保护、共享、虚拟化

### 5.2 地址空间和地址生成
#### 1.描述编译、汇编、链接和加载的过程是什么？

广义的代码编译过程，实际上应该细分为：预处理，编译，汇编，链接。

预处理过程，负责头文件展开，宏替换，条件编译的选择，删除注释等工作。gcc –E表示进行预处理。

编译过程，负载将预处理生成的文件，经过词法分析，语法分析，语义分析及优化后生成汇编文件。gcc –S表示进行编译。

汇编，是将汇编代码转换为机器可执行指令的过程。通过使用gcc –C或者as命令完成。

链接，负载根据目标文件及所需的库文件产生最终的可执行文件。链接主要解决了模块间的相互引用的问题，分为地址和空间分配，符号解析和重定位几个步骤。实际上在编译阶段生成目标文件时，会暂时搁置那些外部引用，而这些外部引用就是在链接时进行确定的。链接器在链接时，会根据符号名称去相应模块中寻找对应符号。待符号确定之后，链接器会重写之前那些未确定的符号的地址，这个过程就是重定位。

Linux系统通过execve()系统调用来执行程序，系统会为相应格式的文件查找合适的装载处理函数。elf格式文件的加载主要通过load_elf_binary()完成，有如下过程：
* 检查文件格式有效性，比如magic number，文件头
* 寻找动态链接的.interp段，设置动态链接器路径
* 根据elf文件描述，对elf文件进行映射，比如代码，数据
* 初始化elf进程环境，比如进程启动时EDX寄存器地址应该是DT_FINI地址
* 将系统调用的返回地址，修改成elf可执行文件入口点，该入口点取决于程序的链接方式，对于静态链接的可执行文件，这个入口点就是elf文件中e_entry所指向的地址，对于动态链接的elf，程序入口点就是动态链接器。{如何判断elf采用的是静态链接还是动态链接的？可以通过对它执行ldd命令，如果是静态链接，会输出：statically linked}
* load_elf_binary()执行完毕后，可执行文件就被装入了内存，接下来就可以执行了。

#### 2.动态链接如何使用？尝试在Linux平台上使用LD_DEBUG查看一个C语言Hello world的启动流程。 (optional)

### 5.3 连续内存分配
#### 1.什么是内碎片、外碎片？
内碎片是指分配给任务的内存大小比任务所要求的大小所多出来的内存。外碎片只分配给任务的内存之间无法利用的内存。
#### 2.最先匹配会越用越慢吗？请说明理由（可来源于猜想或具体的实验）？
会。因为最先匹配总是先找低地址空间的内存，到后期低地址空间都是大量小的不连续的内存空间，每次都要扫描低地址空间后到达高地质空间才能得到可用的内存。
#### 3.最差匹配的外碎片会比最优适配算法少吗？请说明理由（可来源于猜想或具体的实验）？
会。因为每次都找到最大的内存块进行分割，因此分割剩下的内存块也很大，往往还可以再装下一个程序。
#### 4.理解0:最优匹配，1:最差匹配，2:最先匹配，3:buddy systemm算法中分区释放后的合并处理过程？ (optional)

### 5.4 碎片整理
#### 1.对换和紧凑都是碎片整理技术，它们的主要区别是什么？为什么在早期的操作系统中采用对换技术？
区别是，紧凑是在内存中搬动进程占用的内存位置，以合并出大块的空闲块；对换是把内存中的进程搬到外存中，以空出更多的内存空闲块。
采用对换的原因是，处理简单。
#### 2.一个处于等待状态的进程被对换到外存（对换等待状态）后，等待事件出现了。操作系统需要如何响应？
将进程从硬盘中读取到内存中，在这个过程中，操作系统将该进程标为等待状态并且调度其他进程。

### 5.5 伙伴系统
#### 1.伙伴系统的空闲块如何组织？
按照内存的大小有一系列链表组织,类似于哈希表，将相同大小的内存区域首地址连接起来。
#### 2.伙伴系统的内存分配流程？伙伴系统的内存回收流程？
* 分配：当向内核请求分配(2^(i-1)，2^i]数目的页块时，按照2^i页块请求处理。如果对应的块链表中没有空闲页块，则在更大的页块链表中找。当分配的页块中有多余的页时，伙伴系统根据多余的页框大小插入到对应的空闲页块链表中。
* 回收：当释放多页的块时，内核首先计算出该内存块的伙伴的地址。内核将满足以下条件的三个块称为伙伴：(1)两个块具有相同的大小，记作b。(2)它们的物理地址是连续的。(3)第一块的第一个页的物理地址是2*(2^b)的倍数。如果找到了该内存块的伙伴，确保该伙伴的所有页都是空闲的，以便进行合并。内存继续检查合并后页块的“伙伴”并检查是否可以合并，依次类推。