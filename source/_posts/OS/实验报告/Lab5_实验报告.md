---
title: OS_Lab5
tags: 
  - OS
categories: 
  - Experiment
cover: https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202408121648665.png
top_img: https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202408091736667.jpg
abbrlink: 14766
---

#### **Thinking 5.1** 

> 如果通过 kseg0 读写设备，那么对于设备的写入会缓存到 Cache 中。这是一种错误的行为，在实际编写代码的时候这么做会引发不可预知的问题。请思考：这么做这会引发什么问题？对于不同种类的设备（如我们提到的串口设备和 IDE 磁盘）的操作会有差异吗？可以从缓存的性质和缓存更新的策略来考虑。 

**answer：**

1. 缓存Cache是为了提高CPU访问数据的速度，但只有在外部设备进行数据更新或缓存Cache空间不足时，缓存数据才会被写入内存，故在外部设备更新数据时，CPU可能从外设（外部设备对应的内存空间）读入更新前的数据，故引发错误。
2. 串口设备通常是字符设备，按照字节或字符进行数据传输，传输数据的频率高，IDE磁盘是块设备，按块进行数据传输，传输数据的频率低，故前者引发问题的概率大于后者

#### **Thinking 5.2** 

> 查找代码中的相关定义，试回答一个磁盘块中最多能存储多少个文件控制块？一个目录下最多能有多少个文件？我们的文件系统支持的单个文件最大为多大？

**answer:**

* 宏`FILE_STRUCT_SIZE`表示文件结构体的大小`256B`，一个磁盘块的大小为`4KB`，故一个磁盘块最多能储存$4KB / 256B = 16$个文件控制块。
* 一个磁盘块储存一个目录文件，储存$4KB / 4B = 1024$ 个页目录项（`32bit`），每个页目录项指向其对应的磁盘块，储存`16`个文件控制块，故一个目录最多有$1024 * 16 = 16K$个文件。
* 文件控制块使用直接指针`f_direct`和间接指针`f_indirect`储存文件内容，直接指针和间接指针的个数总数为1024个，故单个文件最大时表示所有指针均有效并储存文件数据，根据一个指针指向一个磁盘块，单个文件最大为$1024 * 4KB = 4MB$。

**Exercise 5.4** 

> 文件系统需要负责维护磁盘块的申请和释放，在回收一个磁盘块时，需要更改位图中的标志位。如果要将一个磁盘块设置为 free，只需要将位图中对应位的值设置为 1 即可。请完成 `fs/fs.c` 中的 `free_block` 函数，实现这一功能。同时思考为什么参数`blockno` 的值不能为 0？

**answer：**

`blockno == 0`，对应的第一个磁盘块类型设为 `BLOCK_BOOT`，表示主引导扇区，作用是加载引导程序并启动操作系统，不能将其空闲化。

#### **Thinking 5.3** 

> 请思考，在满足磁盘块缓存的设计的前提下，我们实验使用的内核支持的最大磁盘大小是多少？ 

**answer：**

由于本操作系统中将 `DISKMAP`到 `DISKMAP+DISKMAX` 这一段虚存地址空间 (0x10000000-0x4FFFFFFF) 作为缓冲区，因为`DISKMAX`的大小为`0x4000_0000`，故我们实验使用的内核支持的最大磁盘大小是1GB

**Thinking 5.4** 

> 在本实验中，``fs/serv.h`、`user/include/fs.h` 等文件中出现了许多宏定义，试列举你认为较为重要的宏定义，同时进行解释，并描述其主要应用之处。 

**answer：**

* `INDEX2FD`获取文件描述符所映射到的地址（虚拟）

  ```bash
  #define INDEX2FD(i) (FDTABLE + (i)*PTMAP)
  ```

* `INDEX2DATA` 获取文件描述符线性对应的位置所映射到的地址（虚拟）

  ```bash
  #define INDEX2DATA(i) (FILEBASE + (i)*PDMAP)
  ```

#### **Thinking 5.5** 

> 在 Lab4“系统调用与 fork”的实验中我们实现了极为重要的 fork 函数。那么 fork 前后的父子进程是否会共享文件描述符和定位指针呢？请在完成上述练习的基础上编写一个程序进行验证。 

**answer：**

`fork`前后的父子进程会共享文件描述符和定位指针

**程序验证**

```C
int main() {
	int r;
	int fdnum;
	char buf[512];
	int n;

	if ((r = open("/newmotd", O_RDWR)) < 0) {
		user_panic("cannot open /newmotd: %d", r);
	}
	fdnum = r;
	debugf("open is good\n");

	if ((n = read(fdnum, buf, 511)) < 0) {
		user_panic("cannot read /newmotd: %d", n);
	}
	if (strcmp(buf, diff_msg) != 0) {
		user_panic("read returned wrong data\nbuf: %s\n n: %d\n fdnum: %d", buf, n ,fdnum);
	}
	debugf("read is good\n");
	int id;

        if ((id = fork()) == 0) {
            if ((n = read(fdnum, buf, 511)) < 0) {
                user_panic("child read /newmotd: %d", n);
            }
            if (strcmp(buf, diff_msg) != 0) {
                user_panic("child read returned wrong data");
            }
	    debugf("child read is good && child_fd == %d\n",n);
	    struct Fd *fdd;
            fd_lookup(n,&fdd);
            debugf("child_fd's offset == %d\n",fdd->fd_offset);
        }
        else {
            if((n = read(fdnum, buf, 511)) < 0) {
                user_panic("father read /newmotd: %d", n);
            }
            if (strcmp(buf, diff_msg) != 0) {
                user_panic("father read returned wrong data");
            }
            debugf("father read is good && father_fd == %d\n",r);
            struct Fd *fdd;
            fd_lookup(n,&fdd);
            debugf("father_fd's offset == %d\n",fdd->fd_offset);
        }
}
```

**实验结果**

```c
init.c: mips_init() is called
Memory size: 65536 KiB, number of pages: 16384
to memory 80430000 for struct Pages.
pmap.c:  mips vm init success
FS is running
superblock is good
read_bitmap is good
open is good
read is good
father read is good && father_fd == 0
father_fd's offset == 40
child read is good && child_fd == 0
child_fd's offset == 40
```

#### **Thinking 5.6** 

> 请解释 `File`,`Fd`, `Filefd` 结构体及其各个域的作用。比如各个结构体会在哪些过程中被使用，是否对应磁盘上的物理实体还是单纯的内存数据等。说明形式自定，要求简洁明了，可大致勾勒出文件系统数据结构与物理实体的对应关系与设计框架。 

```c
struct File {
	char f_name[MAXNAMELEN]; // 文件名称,最大长度为MAXNAMELEN（128）
    uint32_t f_size; // 文件大小
	uint32_t f_type; // 文件类型，区分文件还是目录
	uint32_t f_direct[NDIRECT]; // 直接索引,指向文件内容的磁盘块
	uint32_t f_indirect; // 间接索引，存储指向文件内容的磁盘块的指针（直接索引）
	
	struct File *f_dir; // 文件所属的文件目录
	char f_pad[FILE_STRUCT_SIZE - MAXNAMELEN - (3 + NDIRECT) * 4 - sizeof(void *)];
    //为了让文件控制块占用一个磁盘块，填充其剩余空间
} __attribute__((aligned(4), packed));

struct Fd {
     u_int fd_dev_id;     // 外设的id。
     //在fd.c中，会根据Fd的dev_id会调用相应的文件服务函数。
     u_int fd_offset;     // 文件的偏移量
     //用来找起始filebno的文件块号。
     u_int fd_omode;      // 打开方式（只读，只写，读写）
     //req和open结构体都会用到
};

struct Filefd {
     struct Fd f_fd;     // 文件描述符
     u_int f_fileid;     // 文件的id
     //用来在opentab[]里映射其对应的open结构体
     struct File f_file; // 其文件控制块
};
```

上述的结构体均为内存数据，记录了文件信息。
`Filefd` 以及 `fd` 中的保存的文件控制块 `File` 中记录的直接或间接索引对应物理实体。

#### **Thinking 5.7** 

> 下图中有多种不同形式的箭头，请解释这些不同箭头的差别，并思考我们的操作系统是如何实现对应类型的进程间通信的。 

<img src="https://ooo.0x0.ooo/2024/07/02/OPmhoI.png" alt="image-20240529173532171" style="zoom:70%;" />

`ENV_CREATE(user_env)` 和 `ENV_CREATE(fs_serv)` 两者由 `init()` 发出创建消息后，`init()` 函数返回执行其他操作，由上述两个方法对应的两个线程 `fs` 和 `user` 完成自己的初始化工作，为异步消息。
`fs` 通过`serv_init()` 和 `fs_init()` 完成文件服务系统进程的初始化后，再进入 `serv()` 函数，循环调用 `ipc_revc()`，反复接受用户线程的处理请求（在`ipc_revc`会被阻塞，直到被用户线程的 `ipc_send` 唤醒）
`user` 向 `fs`  通过`ipc_send`发送处理请求（同步消息），发送后自身进入阻塞状态等待被唤醒，此时正等待被唤醒的`fs`线程完成服务后，通过 `ipc_send` 向用户线程发送完成服务的信息，再进入上述的循环当中。

#### 难点分析

依据指导书，我们主要需要完成以下三部分的内容，

1. 外部存储设备驱动：
   * 需要理解内存映射，不同的外设有着其映射的内存空间，需要为用户进程提供相应的接口
   * 对IDE硬盘的物理结构与操作类型进行了解，在设备的物理地址读或写内存的虚拟地址，完成驱动程序的编写
2. 文件系统结构：
   * 理解磁盘文件系统布局，理解磁盘空间中不同磁盘块的功能
   * 掌握文件系统详细结构——文件控制块，理解不同对文件的服务函数对文件控制块的操作
   * 块缓存，文件系统服务是一个用户进程，其可以拥有 4GB 的虚拟内存空间，本操作系统中将 DISKMAP 到DISKMAP+DISKMAX 这一段虚存地址空间 (0x10000000-0x4FFFFFFF) 作为缓冲区，当磁盘读入内存时，用来映射相关的页。
3. 文件系统的用户接口：
   * 文件描述符：当用户进程试图打开一个文件时，文件系统服务进程通过文件描述符来存储文件的基本信息和用户进程中关于文件的状态；同时，文件描述符也起到描述用户对于文件操作的作用。
   * 文件系统服务：文件系统服务通过 IPC 的形式供其他进程调用，进行文件读写操作。

**理念的理解：**

1. 将传统操作系统的文件系统移出内核，使用用户态的文件系统服务程序以及一系列用户库来实现。用户进程通过进程间通信(IPC) 来请求文件系统的相关服务。
2. 操作系统将一些内核数据暴露到用户空间，使得进程不需要切换到内核态就能访问。MOS将进程页表映射到用户空间，此处文件系统服务进程访问自身进程页表即可判断磁盘缓存中是否存在对应块。
3. 将传统操作系统的设备驱动移出内核，作为用户程序来实现。微内核体现在，内核在此过程中仅提供读写设备物理地址的系统调用。

#### 实验体会

本次`lab5-2`实验中，完成exam的实验部分，其整个过程就是上述三个部分的实现，在充分掌握课下内容的情况下，exam较为容易完成。

**注：**注意`memcpy`的源地址和目标地址

没有完成`extra`部分，完成该部分的同学描述该部分与exam较为相似，但建议的实现过程相较之下并不详细，代码量较大，需要注意的点较多。
