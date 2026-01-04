---
title: OS_Lab3
tags: 
  - OS
categories: 
  - Experiment
cover: https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202408121648665.png
top_img: https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202408091736667.jpg
abbrlink: 15150
---

#### Thinking 3.1

> 请结合 MOS 中的页目录自映射应用解释代码中 e->env_pgdir[PDX(UVPT)] = PADDR(e->env_pgdir) | PTE_V 的含义。 

**解释：**

该代码设置进程控制块e的页目录下虚拟地址UVPT映射到其本身的物理地址，并设置为只读权限。该页目录项所指向的为一个二级页表，但通过地址判断还是一个一级页表，即页目录自身，这就是页目录的自映射。

**分析：**

读取该虚拟地址PDX(UVPT)[31:22]PDX(UVPT)[31:22]0000_0000_0000_0000的数据，首先读取页目录项（e->env_pgdir[PDX(UVPT)]），得到二级页表的基地址（物理地址）PADDR(e->env_pgdir) | PTE_V，该二级页表与一级页表为同一个页表，所读取的页表项与一开始读取的页目录项相同，为e->env_pgdir[PDX(UVPT)]，最终得到数据PADDR(e->env_pgdir) | PTE_V，所得到的页表为页目录本身，即为自映射。

#### **Thinking 3.2**

> elf_load_seg 以函数指针的形式，接受外部自定义的回调函数 map_page。请你找到与之相关的 data 这一参数在此处的来源，并思考它的作用。没有这个参数可不可以？为什么？ 

* elf_load_seg 定义在文件lib/elfloader.c当中

<img src="https://ooo.0x0.ooo/2024/07/02/OPm53v.png" alt="Screenshot 2024-04-17 200558" style="zoom:80%;" />

* load_icode中调用elf_load_seg传入的data参数为传入load_icode的参数e

<img src="https://ooo.0x0.ooo/2024/07/02/OPmTiq.png" alt="Screenshot 2024-04-17 200218" style="zoom:80%;" />

* 不可以没有该参数。

  因为在回调函数load_icode_mapper当中，需要使用进程控制块中的页目录基地址（env->env_pgdir）和进程asid信息（env->env_asid），加入进程的虚拟地址va当中。

  <img src="https://ooo.0x0.ooo/2024/07/02/OPmUtc.png" alt="Screenshot 2024-04-17 200932" style="zoom:80%;" />



#### **Thinking 3.3**

> 结合 elf_load_seg 的参数和实现，考虑该函数需要处理哪些页面加载的情况。

* 加载的虚拟地址不与页对齐，申请额外空间存放开头发生偏移的部分（非整页的情况），映射到页中。
* 加载该段的.text和.data，通过循环不断将数据加载到页上。
* 已申请段空间小于段需载入内存大小，继续创建新页，但不向其中加载任何内容（NULL）。

#### **Thinking 3.4** 

> 思考上面这一段话，并根据自己在 **Lab2** 中的理解，回答：
>
> * 你认为这里的 env_tf.cp0_epc 存储的是物理地址还是虚拟地址?

虚拟地址

#### **Thinking 3.5** 

>  试找出 0、1、2、3 号异常处理函数的具体实现位置。8 号异常（系统调用）涉及的 do_syscall() 函数将在 Lab4 中实现。 

**0 号异常 —— handle_int**

kern/genex.S

<img src="https://ooo.0x0.ooo/2024/07/02/OPmW2r.png" alt="Screenshot 2024-04-21 223336" style="zoom:80%;" />

<img src="https://ooo.0x0.ooo/2024/07/02/OPmkbM.png" alt="Screenshot 2024-04-21 224213" style="zoom:80%;" />

**1 号异常 —— handle_mod**

**2 号异常 —— handle_tlb**

**3 号异常 —— handle_tlb**

上述三个异常通过BUILD_HANDLER实现

![Screenshot 2024-04-21 224203](https://ooo.0x0.ooo/2024/07/02/OPmxgG.png)

#### Thinking 3.6

> 阅读 entry.S、genex.S 和 env_asm.S 这几个文件，并尝试说出时钟中断在哪些时候开启，在哪些时候关闭。 

在调度执行每一个进程之前，所调用的env_run函数中，调用**env_asm.S文件**中的env_pop_tf函数，其调用了宏 RESET_KCLOCK，将 Count 寄存器清零并将 Compare 寄存器配置为我们所期望的计时器周期数，再调用**genex.S文件**中的ret_from_exception函数，将当前的 CPU 现场（上下文）保存到内核的异常栈中，并跳转至EPC（原处），在宏 RESTORE_ALL 中恢复了 Status 寄存器，**开启了时钟中断**；当 Count 寄存器的值与 Compare 寄存器的值相等且非 0时，**时钟中断产生**，通过**entry.S中的异常分发函数**，判断当前异常为时钟中断异常，随后进入0号异常（handle_int），后根据schedule函数决定是否要**关闭时钟中断**（没有程序运行）

#### Thinking 3.7 

> 阅读相关代码，思考操作系统是怎么根据时钟中断切换进程的。

在env_run函数中，在启动线程的最后，通过调用env_asm.S文件中的env_pop_tf函数，其调用了宏 RESET_KCLOCK，将 Count 寄存器清零并将 Compare 寄存器配置为我们所期望的计时器周期数，再调用genex.S文件中的ret_from_exception函数，将当前的 CPU 现场（上下文）保存到内核的异常栈中，并跳转至EPC（原处），在宏 RESTORE_ALL 中恢复了 Status 寄存器，开启了时钟中断；

当 Count 寄存器的值与 Compare 寄存器的值相等且非 0时，时钟中断产生，触发MIPS中断，cpu 就会自动跳转到虚拟地址 `0x80000180`，即跳转到 .text.exc_gen_entry代码段（.text），通过entry.S中的异常分发函数，判断当前异常为时钟中断异常，随后跳转0号异常的中断处理函数—— handle_int，进而判断为IM7中断，再跳转至enable_irq函数，调用schedule函数进行调度处理。

在schedule函数中，将Count寄存器的值清零，并取出当前运行的进程控制块curenv。若满足以下任一情况，则进行进程切换：

* 尚未调度过任何进程（curenv 为空指针）；
* 当前进程已经用完了时间片；
* 当前进程不再就绪（如被阻塞或退出）；
* yield 参数指定必须发生切换。则进行进程切换：

进行进程切换时，需判断当前进程是否仍然就绪，如果是则将其移动到调度链表的尾部，再选中调度链表首部的进程来调度运行，将剩余时间片长度设置为其优先级。

无论是否进行进程切换，均要对当前进程的可用时间片数量减一（即 count--;）并调用env_run函数（运行e）

即通过时钟中断完成进程切换

#### 难点分析

![难点分析](https://ooo.0x0.ooo/2024/07/02/OPmzV1.png)

#### 实验体会

**课下：**

仔细学习进程的相关知识，通过各种函数掌握进程的相关信息，完成加载文件，创建进程，进程运行与切换；学习中断与异常，掌握异常的分发，异常向量组，时钟中断，进程调度。跟之前的`lab`相似，仍需要反复学习，较长时间才能掌握上述知识

**课上：**

未能完成`exam`，未理解题意