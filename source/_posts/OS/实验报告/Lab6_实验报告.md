---
title: OS_Lab6
tags: 
  - OS
categories: 
  - Experiment
cover: https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202408121648665.png
top_img: https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202408091736667.jpg
abbrlink: 14574
---

#### **Thinking 6.1** 

>  示例代码中，父进程操作管道的写端，子进程操作管道的读端。如果现在想让父进程作为“读者”，代码应当如何修改？ 

```c
#include <stdlib.h>
#include <unistd.h>

int fildes[2];
char buf[100];
int status;

int main()
{

	status = pipe(fildes);

	if (status == -1)
	{
		printf("error\n");
	}

	switch (fork())
	{
	case -1:
		break;

	case 0:									  /* 子进程 - 作为管道的写者 */
		close(fildes[0]);					   /* 关闭不用的读端 */
		write(fildes[1], "Hello world\n", 12); /* 向管道中写数据 */
		close(fildes[1]);					   /* 写入结束，关闭写端 */
		exit(EXIT_SUCCESS);
            
	default:								   /* 父进程 - 作为管道的读者 */
        close(fildes[1]);					  /* 关闭不用的写端 */
		read(fildes[0], buf, 100);			  /* 从管道中读数据 */
		printf("parent-process read:%s", buf); /* 打印读到的数据 */
		close(fildes[0]);					  /* 读取结束，关闭读端 */
		exit(EXIT_SUCCESS);
	}
}
```



#### **Thinking 6.2** 

> 上面这种不同步修改 pp_ref 而导致的进程竞争问题在 `user/lib/fd.c` 中的 dup 函数中也存在。请结合代码模仿上述情景，分析一下我们的 dup 函数中为什么会出现预想之外的情况？

**answer:**

`dup`函数是将依据`oldfdnum`的文件描述符复制给依据`newfdnum`的文件描述符，先完成文件描述符的映射（虚拟页 -> 物理页），在完成文件数据的映射。

```c
switch (fork())
	{
	case -1:
		break;

	case 0:									  /* 子进程 */
		read(p[0], buf, sizeof(buf));
            
	default:								   /* 父进程 */
        dup(p[0], newfd);
   	 	write(p[1], "hello world!", 12)
	}    
```

上述代码为`fork`函数后，子进程先进行。但是在`read`之前发生时钟中断，故进程切换到父进程开始进行，父进程在`dup(p[0])`中，完成对`p[0`]的映射，但这时发生中断，故还未来得及完成对`pipe`的映射。此时再进行进程切换，回到子进程，进入`read`函数，结果通过`ref(p[0]) == ref(pipe) == 2`判断此时写进程关闭，故导致预想之外的情况。



#### **Thinking 6.3** 

> 阅读上述材料并思考：为什么系统调用一定是原子操作呢？如果你觉得不是所有的系统调用都是原子操作，请给出反例。希望能结合相关代码进行分析说明。

```bash
exc_gen_entry:
        SAVE_ALL
        mfc0    t0, CP0_STATUS
        and     t0, t0, ~(STATUS_UM | STATUS_EXL | STATUS_IE)
        mtc0    t0, CP0_STATUS
        mfc0    t0, CP0_CAUSE
        andi    t0, 0x7c
        lw      t0, exception_handlers(t0)
        jr      t0
```

这是系统调用`syscall`对用的异常处理程序`handle_sys`,`and t0, t0, ~(STATUS_UM | STATUS_EXL | STATUS_IE)`该行代码通过取消设置UM和EXL，并取消设置IE以全局禁用中断，故不会响应中断，不会切换进程或运行其他程序，从而保证系统调用原子操作



#### **Thinking 6.4** 

> 仔细阅读上面这段话，并思考下列问题

* 按照上述说法控制 **pipe_close** 中 `fd` 和 `pipe unmap` 的顺序，是否可以解决上述场景的进程竞争问题？给出你的分析过程。

* 我们只分析了 close 时的情形，在 `fd.c` 中有一个 dup 函数，用于复制文件描述符。试想，如果要复制的文件描述符指向一个管道那么是否会出现与 close 类似的问题？请模仿上述材料写写你的理解。

**answer:**

- 可以解决。由于需要保证`ref(p[0])`小于`ref(pipe)`始终成立，故先通过`unmap`解出`p[0]`的映射，则使得`ref(p[0])`更小于`ref(pipe)`，从而符合要求。
- `dup`函数会出现与`close`类似的问题，同时也需要保证`pipe`的引用次数总比`fd`要高。在管道的`dup`方法执行到一半时， 若先映射`fd`，再映射 `pipe` ，会导致`fd`在`pipe`引用次数增加1，会出现两者的引用次数相等的情况，故也会出现相似的问题。



#### **Thinking 6.5** 

> 思考以下三个问题。

* 认真回看 Lab5 文件系统相关代码，弄清打开文件的过程。

* 回顾 Lab1 与 Lab3，思考如何读取并加载 ELF 文件。

* 在 Lab1 中我们介绍了 `data text bss` 段及它们的含义，`data` 段存放初始化过的全局变量，`bss` 段存放未初始化的全局变量。关于 `memsize` 和 `filesize` ，我们在 Note1.3.4中也解释了它们的含义与特点。关于 Note 1.3.4，注意其中关于“`bss` 段并不在文件中占数据”表述的含义。回顾 Lab3 并思考：`elf_load_seg()` 和 `load_icode_mapper()`函数是如何确保加载 ELF 文件时，`bss` 段数据被正确加载进虚拟内存空间。`bss` 段在 ELF 中并不占空间，但 ELF 加载进内存后，`bss` 段的数据占据了空间，并且初始值都是 0。请回顾 `elf_load_seg()` 和 `load_icode_mapper()` 的实现，思考这一点是如何实现的？

(1) 

<img src="https://ooo.0x0.ooo/2024/07/03/OPpift.png" alt="image-20240529173532171" style="zoom:70%;" />

`ENV_CREATE(user_env)` 和 `ENV_CREATE(fs_serv)` 两者由 `init()` 发出创建消息后，`init()` 函数返回执行其他操作，由上述两个方法对应的两个线程 `fs` 和 `user` 完成自己的初始化工作，为异步消息。
`fs` 通过`serv_init()` 和 `fs_init()` 完成文件服务系统进程的初始化后，再进入 `serv()` 函数，循环调用 `ipc_revc()`，反复接受用户线程的处理请求，这里指open（打开文件）（在`ipc_revc`会被阻塞，直到被用户线程的 `ipc_send` 唤醒）
`user` 向 `fs`  通过`ipc_send`发送处理请求（同步消息），发送后自身进入阻塞状态等待被唤醒，此时正等待被唤醒的`fs`线程完成服务后，通过 `ipc_send` 向用户线程发送完成服务的信息，再进入上述的循环当中。（返回其文件控制块的指针）

(2)	`load_icode` 函数负责加载可执行文件 `binary` 到进程 e 的内存中。它调用的 `elf_from` 函数完成了解析 ELF 文件头的部分，elf_load_seg 负责将 ELF 文件的一个 segment 加载到内存。为了达到这一目标，elf_load_seg 的最后两个参数用于接受一个自定义的回调函数 map_page，以及需要传递给回调函数的额外参数 data。每当 elf_load_seg 函数解析到一个需要加载到内存中的页面，会将有关的信息作为参数传递给回调函数，并由它完成单个页面的加载过程，而这里 `load_icode_mapper` 就是 map_page 的具体实现。`load_icode` 函数会从 ELF 文件中解析出每个 segment 的段头 `ph`，以及其数据在内存中的起始位置 bin，再由 elf_load_seg 函数将参数指定的程序段（program segment）加载到进程的地址空间中。

(3)

```c
while (i < sgsize)
{
	if ((r = map_page(data, va + i, 0, perm, NULL, MIN(sgsize - i, PAGE_SIZE))) != 0)
	{
		return r;
	}
	i += PAGE_SIZE;
}
return 0;
```

这段代码中通过不断创建新的页，但是并不向其中加载任何内容，从而实现`bss` 段的数据占据了空间，并且初始值都是 0。



#### **Thinking 6.6** 

> 通过阅读代码空白段的注释我们知道，将标准输入或输出定向到文件，需要我们将其 dup 到 0 或 1 号文件描述符（fd）。那么问题来了：在哪步，0 和 1 被“安排”为标准输入和标准输出？请分析代码执行流程，给出答案。

在`init.c`的main函数分别设置0 和 1 被为标准输入和标准输出

```C
// stdin should be 0, because no file descriptors are open yet
if ((r = opencons()) != 0)
{
	user_panic("opencons: %d", r);
}
// stdout
if ((r = dup(0, 1)) < 0)
{
	user_panic("dup: %d", r);
}
```



#### **Thinking 6.7** 

> 在 shell 中执行的命令分为内置命令和外部命令。在执行内置命令时 shell 不需要 fork 一个子 shell，如 Linux 系统中的 cd 命令。在执行外部命令时 shell 需要 fork一个子 shell，然后子 shell 去执行这条命令。据此判断，在 MOS 中我们用到的 shell 命令是内置命令还是外部命令？请思考为什么Linux 的 cd 命令是内部命令而不是外部命令？

**answer：**

- shell命令是外部命令，因为在执行shell命令时，当前进程通过`fork`产生一个子进程，即上述所提及的子shell，然后子 shell 去执行这条命令。
- shell包含`linux`的部分命令，即一些比较简单的命令，这些指令直接在shell程序内运行，其本身即挂载在系统重。而`cd`指令符合上述特征，这样子的目的也是为了避免每次都要使用`fork`新建子shell，提高运行效率。



#### **Thinking 6.8** 

>  在你的 shell 中输入命令 `ls.b | cat.b > motd`。

* 请问你可以在你的 shell 中观察到几次 spawn ？分别对应哪个进程？

* 请问你可以在你的 shell 中观察到几次进程销毁？分别对应哪个进程？

**answer：**

- 2次`spawn`，两个子进程分别执行`ls.b`和`cat.b`
- 2次进程销毁，进行即上述分别执行`ls.b`和`cat.b`的两个子进程。

#### 难点分析

1. 掌握管道的原理与底层细节
2. 实现管道的读写
3. 复述管道竞争情景
4. 实现基本 shell
5. 实现 shell 中涉及管道的部分

#### 实验体会

经过一个学期的操作系统的学习，在熟悉linux终端以及其他工具的同时，对操作系统内的架构体系与具体细节有了更深入的体会，同时对操作系统凝结巨大智慧的艺术品顿感敬佩，也为能深入学习并坚持下来感到庆幸。最后的Lab6作为收尾，实现一个简单的程序功能（shell），看到搭建的操作系统最终能执行指令，十分雀跃，也很享受开发与应用的过程。