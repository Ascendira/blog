---
title: OS_Lab4
tags: 
  - OS
categories: 
  - Experiment
cover: https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202408121648665.png
top_img: https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202408091736667.jpg
abbrlink: 63855
---

#### Thinking 4.1

> 思考并回答下面的问题：
>
> * 内核在保存现场的时候是如何避免破坏通用寄存器的？
> * 系统陷入内核调用后可以直接从当时的 `$a0-$a3 `参数寄存器中得到用户调用 `msyscall`留下的信息吗？
> * 我们是怎么做到让 sys 开头的函数“认为”我们提供了和用户调用 `msyscall` 时同样的参数的？
> * 内核处理系统调用的过程对 `Trapframe` 做了哪些更改？这种修改对应的用户态的变化是什么？

**answer:**

* 内核通过调用宏`SAVE_ALL`，先保存`$sp`,`$v0`两个寄存器，协助将通用寄存器保存在栈空间`Trapframe`中
* 可以，因为系统陷入内核调用时，当时的 `$a0-$a3 `参数寄存器并未被修改，可以得到用户调用 `msyscall`留下的信息。
* 用户函数`syscall_*`到内核函数`sys_*`，`$a0-$a3`寄存器不会被破坏，也可直接通过内核栈获取

* ```bash
  tf->cp0_epc += 4;
  tf->regs[2] = func(arg1, arg2, arg3, arg4, arg5);
  ```

  前者保证跳转回用户态函数，执行异常指令（`syscall`）的下一条指令，后者保证用户态可通过`Trapframe`中储存的返回值得到`sysno`对应的系统调用函数

#### Thinking 4.2 

> 思考 envid2env 函数: 为什么 envid2env 中需要判断 `e->env_id != envid`的情况？如果没有这步判断会发生什么情况？

**answer:**

* 可通过`envid`的后十位得到数组`envs`中对应的进程控制块`env`，由于函数`mkenvid`通过`++i`生成`envid`，有不确定性，若不进行判断`e->env_id != envid`，则可能取到错误或者已经被销毁的进程控制块

#### **Thinking 4.3** 

> 思考下面的问题，并对这个问题谈谈你的理解：请回顾 `kern/env.c` 文件中 `mkenvid()` 函数的实现，该函数不会返回 0，请结合系统调用和 IPC 部分的实现与`envid2env()` 函数的行为进行解释。 

**answer:**

* 函数`mkenvid()`由于`++i`不会返回0
* 函数`envid2env()`当`envid == 0`返回`curenvid`
* 系统调用：用户态通过函数`syscall_*()`，直接通过`envid = 0` 调用内核的当前进程，在内核变量不可见的情况下调用内核接口
* IPC通过`e->env_ipc_recving = 0`传递得到当前进程的`envid`。

#### **Thinking 4.4** 

> 关于 fork 函数的两个返回值，下面说法正确的是：
>
> A、fork 在父进程中被调用两次，产生两个返回值
>
> B、fork 在两个进程中分别被调用一次，产生两个不同的返回值
>
> C、fork 只在父进程中被调用了一次，在两个进程中各产生一个返回值
>
> D、fork 只在子进程中被调用了一次，在两个进程中各产生一个返回值

**answer:**

* C

#### **Thinking 4.5** 

> 我们并不应该对所有的用户空间页都使用 `duppage` 进行映射。那么究竟哪些用户空间页应该映射，哪些不应该呢？请结合 `kern/env.c` 中 `env_init` 函数进行的页面映射`include/mmu.h` 里的内存布局图以及本章的后续描述进行思考。 

**answer:**

* `UTOP`到`UVPT`之间储存内核的页表项，页和页控制块等信息，在`env_setup_vm`函数（`env_init`函数）中，将上述地址空间映射到进程的地址空间上。
* `USTACKTOP`到`UTOP`之间为`user expception stack`（异常处理栈），为异常处理的地方，故无需共享此部分的内存
* 故需映射的为`USTACKTOP`以下的部分，需要保证页目录项与页表项的有效性

#### **Thinking 4.6** 

> 在遍历地址空间存取页表项时你需要使用到 `vpd` 和 `vpt` 这两个指针，请参考 `user/include/lib.h` 中的相关定义，思考并回答这几个问题：
>
> * `vpt` 和 `vpd` 的作用是什么？怎样使用它们？
> * 从实现的角度谈一下为什么进程能够通过这种方式来存取自身的页表？
> * 它们是如何体现自映射设计的？
> * 进程能够通过这种方式来修改自己的页表项吗？

**answer:**

* 作用：`vpt`和`vpd`分别是指向用户页表和用户页目录的基地址的指针，可看作储存页表项和页目录项的数组，可用来访问其存储的页表项和页目录项的内容。

  * 使用`vpt`时，`vpt`加上页表项的偏移量（`va`对应的页表项，高12~31位），再使用`*`得到页表项储存的内容，`eg:vpt[va >> PAGESIZE] 或 *(vpt) + (va >> PAGESIZE)  `
  * 使用`vpd`时，`vpd`加上页目录项的偏移量（`va`对应的页目录项，高22~31位），再使用`*`得到页目录项储存的内容，`eg:vpt[va >> (PAGESIZE + 10)] 或 *(vpt) + (va >> (PAGESIZE + 10))  `

* ```bash
   #define vpt ((const volatile Pte *)UVPT)
   #define vpd ((const volatile Pde *)(UVPT + (PDX(UVPT) << PGSHIFT)))
  ```

  上述代码定义宏`vpt`和`vpd`，`vpt`指向储存页表空间的初始地址，`vpd`将页目录的基址映射到页表项的某一个页表项，完成自映射

* 不能，该区域对于用户态而言是只读不写的，想要修改页表项，需要陷入内核态方可

#### **Thinking 4.7** 

> 在 `do_tlb_mod` 函数中，你可能注意到了一个向异常处理栈复制 `Trapframe`运行现场的过程，请思考并回答这几个问题：
>
> * 这里实现了一个支持类似于“异常重入”的机制，而在什么时候会出现这种“异常重入”？
> * 内核为什么需要将异常的现场 `Trapframe` 复制到用户空间？

**answer:**

* 当发生缺页写入异常时，调用`do_tlb_mod`函数，在处理上述异常时，又发生缺页写入异常，重新调用`do_tlb_mod`函数，故发生“异常重入”
* 上述需要在用户态下完成页写入异常的处理，是不能直接使用正常情况下的用户栈的（因为发生页写入异常的也可能是正常栈的页面），所以用户进程就需要一个单独的栈来执行处理程序，我们把这个栈称作异常处理栈，即`Trapframe`。

#### **Thinking 4.8** 

> 在用户态处理页写入异常，相比于在内核态处理有什么优势？

* 按照微内核的设计理念，功能尽可能实现在用户空间中，其包括了页写入异常的处理，更加灵活

* 内核处理页写入异常时，若发生错误，会导致整个系统崩溃

#### **Thinking 4.9** 

> 请思考并回答以下几个问题：
>
> * 为什么需要将 `syscall_set_tlb_mod_entry` 的调用放置在 `syscall_exofork` 之前？
> * 如果放置在写时复制保护机制完成之后会有怎样的效果？

* `syscall_exofork`创建子进程，同时子进程在父进程创建子进程的语句后继续执行，由于子进程需要对设置共享页面的COW权限，存在COW写入异常，故先设置 `syscall_set_tlb_mod_entry`，以防出现异常后，能调用异常处理函数进行处理。
* 父进程在使用写时复制保护机制时，可能会出现缺页写入异常，但此时未设置异常处理函数，导致不能处理。

#### 难点分析

1. 系统调用的实例：在用户态执行内核态提供的接口，使进程陷入内核，执行异常处理程序

2. 系统调用机制：`syscall_*` -> `msyscall` -> `syscall` -> `handle_sys` -> `do_syscall` -> `sys_*`

   `syscall` -> `handle_sys`：通过`CP0_CAUSE`寄存器中的异常码从数组`exception_handlers`中取得异常处理函数`handle_sys`

   `do_syscall` -> `sys_*`：

   * 修改`epc`使得结束系统调用后返回**自陷指令**的下一条指令

   * 通过`a0`参数获得内核的系统调用函数`func(arg1, arg2, arg3, arg4, arg5) `，即`sys_*`

3. 栈帧（`stackframe`）：调用函数在调用被调用函数时，使用栈指针对需要保存的数据通过压栈储存在栈帧中，被调用函数直接从栈帧中获取需要使用的数据（需要被传送的）
4. 理解如何实现`fork`指令，异常写入的处理，写时复制技术，一次调用+两次返回，子进程运行前设置
5. 理解如何实现进程内通信：信息接受（`ipc_resv`），信息发送（`ipc_send`）

#### 实验体会

**lab4-1-exam：**由于进程进行循环调用某个函数（不与其他进程相关联）时，不能将`env_status`设置为`ENV_UN_RUNNABLE`，而其他进程未将`env_status`重新设置为`ENV_RUNNABLE`，导致该进程不能被调度，无法进行再循环调用了。

**lab4-2-exam：**在构建提供给用户态的接口时，类似于`ipc_send`，可通过`env`直接获得当前运行的函数。

两次实验均为做到`extra`部分
