---
title: OS_挑战性任务-Sigaction
tags: 
  - OS
categories: 
  - Experiment
cover: https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202408121648665.png
top_img: https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202408091736667.jpg
abbrlink: 14987
---

### 挑战性任务报告

> 首先想要~~吐槽一点~~，由于要求不能对`Makefile`进行修改，导致不能将自己编写的文件链接入可执行文件当中，导致只能把自己写的函数挤压进已有文件当中，显得~~有点丑陋~~

本实验需要实现异步通信`Sigaction`，仅需关注32个普通信号中的6个信号，按照其信号处理函数或默认处理动作来处理，其余忽略即可。

#### 信号数据存放位置

由于考虑到不同进程需要存储自己的信号，便于在从内核态返回用户态时直接查询使用，故放置在进程控制块Env结构体内

<img src="https://ooo.0x0.ooo/2024/07/02/OPmgqK.png" alt="image-20240627161211466" style="zoom:80%;" />

#### 信号选择

由于指导书要求**同一普通信号在进程中最多只存在一个**，故只要考虑不同信号间的优先级即可，同时要考虑当前进程正在处理的信号的屏蔽集。（ 对于`SIGKILL`信号，该信号不可被阻塞）

<img src="https://ooo.0x0.ooo/2024/07/02/OPmmAa.png" alt="image-20240627162006488" style="zoom:80%;" />

#### 内核态到用户态的准备

参考助教在讨论区的建议，需要进程从内核态返回至用户态过程中检查是否有信号进行处理，若有，则处理完后再返回中断位置，这就需要使用重新申请一个栈帧用于当前信号的处理，设置传参等，以及保存进程发生中断时的现场。（一个比较好玩的地方在于当完成信号处理后会调用系统调用`syscall_set_trapframe`恢复原来的栈帧，这个时候既恢复现场（重新设置`cp0_epc`），又可以通过`eret`直接返回啦）

<img src="https://ooo.0x0.ooo/2024/07/02/OPmpQS.png" alt="image-20240627165156071" style="zoom:80%;" />

#### 掩码栈

因为考虑到进程正在处理信号会被打断，其他进程发送新的信号，返回至原进程时会转而处理优先级更高的信号，但是每个信号都有自己的掩码（需屏蔽同类型信号以及屏蔽集中的信号），进程需要根据当前正在处理的信号的掩码阻塞其余信号，但是上述情况下需要使用掩码栈通过压栈出栈的方式利用栈顶掩码解决。

<img src="https://ooo.0x0.ooo/2024/07/02/OPmJCN.png" alt="image-20240627163105484" style="zoom:80%;" />

#### 用户态函数入口设置

由于信号的处理函数位于用户态，需要像写时复制技术（COW）那样调用`cow_entry`解决（这个主要是因为微内核设计），需要使用相似的设计，于是采用`sigaction_entry`作为依据信号状态选择处理方式的用户态函数入口。

选择放置在`libos.c`下的`libmain`调用系统调用`syscall_set_sigaction_entry`完成第一个进程的用户态信号处理函数入口的设置，子进程直接通过继承的方式进行设置。

<img src="https://ooo.0x0.ooo/2024/07/02/OPmRiC.png" alt="image-20240627163639269" style="zoom:80%;" />

系统调用`sys_exofork`：

<img src="https://ooo.0x0.ooo/2024/07/02/OPmt4L.png" alt="image-20240627163738125" style="zoom:80%;" />

#### 用户态函数入口

需要检查是否有相应的信号处理函数，若无，依据信号类型选择默认处理动作

<img src="https://ooo.0x0.ooo/2024/07/02/OPm46i.png" alt="image-20240627164937015" style="zoom:80%;" />

#### 信号注册函数 & 信号发送函数 & 信号集处理函数

* 对参数进行非法检查
* 需要对进程的信号数据进行修改则构建系统调用
* 信号发送函数对于`SIGILL`，`SIGSEGV`，`SIGCHLD`，`SIGSYS`在MOS原实现中直接panic，但在使用`sigaction`之后可直接使用`kill`发送信号

**SIGILL**

<img src="https://ooo.0x0.ooo/2024/07/02/OPmHbX.png" alt="image-20240627171145292" style="zoom:80%;" />

**SIGCHLD**

<img src="https://ooo.0x0.ooo/2024/07/02/OPmLmt.png" alt="image-20240627171403575" style="zoom:80%;" />

**SIGSEGV**

<img src="https://ooo.0x0.ooo/2024/07/02/OPmEYx.png" alt="image-20240627171108207" style="zoom:80%;" />

**SIGSYS**

<img src="https://ooo.0x0.ooo/2024/07/02/OPmQqj.png" alt="image-20240627171442352" style="zoom:80%;" />

#### 信号处理完毕

* 调用系统调用`syscall_sig_over`将取消相应信号的置位以及掩码栈的出栈
* 调用系统调用`syscall_set_trapframe`设置为原栈帧直接放回至中断位置即可。

#### 错误点

* 在异常返回函数中调用`do_sigaction`进行信号处理时，记得传参数（a0)

<img src="https://ooo.0x0.ooo/2024/07/02/OPmSAp.png" alt="image-20240627170044658" style="zoom:80%;" />

* 在信号集处理函数中记得对`__set`，`__signo`等参数进行细致的非法检查，但在`int sigprocmask(int __how, const sigset_t * __set, sigset_t * __oset);`信号集处理函数中传入`__set`，`__oset`即使是NULL并不非法，不存入即可。

  
