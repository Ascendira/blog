---
title: OS_Lab1
tags: 
  - OS
categories: 
  - Experiment
cover: https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202408121648665.png
top_img: https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202408091736667.jpg
description: 我服了哇！
abbrlink: 64175
---

#### Thinking 1.1

> 请阅读 附录中的编译链接详解，尝试分别使用实验环境中的原生 x86 工具链（gcc、ld、readelf、objdump 等）和 MIPS 交叉编译工具链（带有 mips-linux-gnu-前缀），重复其中的编译和解析过程，观察相应的结果，并解释其中向 objdump 传入的参数的含义。 

**新建 + 预处理**



<img src="https://ooo.0x0.ooo/2024/07/02/OPgEeF.png" alt="image-20240319225251242" style="zoom:60%;" />

<img src="https://ooo.0x0.ooo/2024/07/02/OPgGE6.png" alt="image-20240325155144959" style="zoom:80%;" />

<img src="https://ooo.0x0.ooo/2024/07/02/OPgQIP.png" alt="image-20240325155120435" style="zoom:60%;" />

**只编译而不链接 + 反汇编**

> 传入参数为test.o,此处仅生成test.c的目标文件，未链接其他目标文件，故printf 的具体实现依然不在我们的程序中。
>
> <img src="https://ooo.0x0.ooo/2024/07/02/OPg0hb.png" alt="image-20240325155355528" style="zoom:80%;" />

<img src="https://ooo.0x0.ooo/2024/07/02/OPgcRl.png" alt="image-20240325155335069" style="zoom:70%;" />

<img src="https://ooo.0x0.ooo/2024/07/02/OPgdjg.png" alt="image-20240325155454391" style="zoom:70%;" />

**编译 + 反汇编**

> 传入的参数为a.out，此处为test.c编译出的可执行文件，在链接（**Link**）阶段，printf 的实现被插入到最终的可执行文件中的。

<img src="https://ooo.0x0.ooo/2024/07/02/OPgVWB.png" alt="image-20240325155627357" style="zoom:67%;" />

<img src="https://ooo.0x0.ooo/2024/07/02/OPgYys.png" alt="image-20240319225638334" style="zoom:60%;" />

<img src="https://ooo.0x0.ooo/2024/07/02/OPgfsK.png" alt="image-20240325155700777" style="zoom:70%;" />

**objdump 传入的参数的含义**

```bash
-D, --disassemble-all    Display assembler contents of all sections
      --disassemble=<sym>  Display assembler contents from <sym>
-S, --source             Intermix source code with disassembly
      --source-comment[=<txt>] Prefix lines of source code with <txt>
```



#### Thinking 1.2

> 思考下述问题：
>
> * 尝试使用我们编写的 readelf 程序，解析之前在 target 目录下生成的内核 ELF 文件。
> * 也许你会发现我们编写的 readelf 程序是不能解析 readelf 文件本身的，而我们刚才介绍的系统工具 readelf 则可以解析，这是为什么呢？（提示：尝试使用 readelf-h，并阅读 tools/readelf 目录下的 Makefile，观察 readelf 与 hello 的不同）

<img src="https://ooo.0x0.ooo/2024/07/02/OPg9oa.png" alt="image-20240320151001117" style="zoom:80%;" />

**原因：**

> <img src="https://ooo.0x0.ooo/2024/07/02/OPgjeS.png" alt="image-20240320152413342" style="zoom:80%;" />
>
> <img src="https://ooo.0x0.ooo/2024/07/02/OPg2GN.png" alt="image-20240320152343555" style="zoom:80%;" />
>
> 生成readelf可执行文件未加上“-m32"的选项，默认生成64位的目标代码，生成hello可执行文件加上“-m32”的选项，生成32位的目标代码。
>
> 在elf.h源文件中，储存的变量为32位或者16位，不能解析64位的readelf的目标文件。

#### Thinking 1.3 

>  在理论课上我们了解到，MIPS 体系结构上电时，启动入口地址为 0xBFC00000（其实启动入口地址是根据具体型号而定的，由硬件逻辑确定，也有可能不是这个地址，但一定是一个确定的地址），但实验操作系统的内核入口并没有放在上电启动地址，而是按照内存布局图放置。思考为什么这样放置内核还能保证内核入口被正确跳转到？（提示：思考实验中启动过程的两阶段分别由谁执行。） 

* MIPS 体系结构加电之后，最先进入kseg1的启动入口地址，因为其是非cache存取的，支持“非翻译无需转换”，唯一能在系统重启后正常工作的地址空间，而由于内核所需内存过大，需要cache存取，需放入kseg0的地址空间，故实验操作系统的内核入口并没有放在上电启动地址。
* 在Stage1阶段，通过启动入口地址，完成初始化Cache，设置堆栈等工作后，跳转至C语言入口。
* 进入Stage2阶段，完成时钟、环境变量、串口速率和内存的初始化工作，进行内存划分，对堆和栈初始化，并留出Bootloader代码大小的空间，将代码从flash上搬到ram上，执行Bootloader的代码，加载操作系统的内核到内存中的指定位置（即按照内存布局图放置的位置）。加载完成后，引导程序会跳转到内核的入口点，开始执行操作系统内核的初始化代码。

#### 难点分析

<img src="https://ooo.0x0.ooo/2024/07/02/OPg6IC.png" alt="image-20240327161916712" style="zoom:60%;" />



#### 实验体会

* **Lab1课下：**

> 课下主要难点在上述的难点分析中，在实际完成Lab1课下时，实际花费在理解理论知识的时间要大于完成思考题和练习题的时间，随之感受到的是对操作系统内核的启动流程，其中的关键节点有了更深刻的理解。补充一句，练习题中的实战printk真的很有意思，真切体会到C语言与ISA层联系之紧密，通过自行实现printk，理解在链接阶段，动态链接库中跳转printf的地址在哪里了。

* **lab1-exam:**

> 描述：在vprintfmt函数中实现一个标识符P，其作用是读入`x`,`y`两个变量，输出`(x,y,|(x-y)*(x+y)|)`，注意`x`,`y`的正负。
>
> 课下仔细完成printk，并且课下没有bug的话，这道题挺容易完成的

* **lab1-extra:**

> 描述：实现一个scanfk函数，可读入标识符为`%d`,`%x`,`%c`,`%s`，并保存在其后的变量中，不同变量间通过间隔符来分隔，变量前至少有一个间隔符，读入`%x`,`%d`时可能会出现前导0和负号，读入`%s`时要注意字符串以`'\0'`结尾
>
> `eg：`
>
> ```c
> scanf("%s%x%d%c",&str ,&X ,&D ,&cha);
> 	 43tfe  -0a0 
>   3256 +
> str:43tf3\0
> X:-160
> D:3256
> cha:+
> ```
>
> **注意：**
>
> 难度并不是很大，主要是要细心。
>
> 在进行处理时，需进行分步处理，先除去变量前的间隔符，再判断是否有正负号，然后除去前导0，之后读入变量值。
>
> 千万不要写成如下代码，简直就是离谱，不仅出现大量重复代码，而且在用肉眼法debug时，出现判断失误的地方大大增加，同时修改bug时要多处同时更改......
>
> ```C
> if (ch == '-')
> {
>     in(data, &ch, 1);
>     while (ch == '0')
>     {
>         in(data, &ch, 1);
>     }
>     int num = 0;
>     int temp = 16;
>     while (('0' <= ch && ch <= '9') || ('a' <= ch && ch <= 'f'))
>     {
>         num *= temp;
>         if ('0' <= ch && ch <= '9')
>         {
>             num += (ch - '0');
>         }
>         else
>         {
>             num += (ch - 'a' + 10);
>         }
>         in(data, &ch, 1);
>     }
>     num *= -1;
>     int *pos = (int *)va_arg(ap, int *);
>     *pos = num;
> }
> else
> {
>     ......(与上面处理方法类似)
> }
> ```





