---
title: OS_Lab0
date: 2024-7-4 21:18:00
tags: 
  - OS
categories: 
  - Experiment
cover: https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202408121648665.png
top_img: https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202408091736667.jpg
description: 实验报告 —— 思考题+难点分析+实验体会
abbrlink: 14958
---

#### Thinking 0.1

> 思考下列有关 Git 的问题：
>
> * 在前述已初始化的 ~/learnGit 目录下，创建一个名为 README.txt 的文件。执行命令 git status > Untracked.txt（其中的 > 为输出重定向，我们将在 0.6.3 中详细介绍）。
> * 在 README.txt 文件中添加任意文件内容，然后使用 add 命令，再执行命令 git status > Stage.txt。
> * 提交 README.txt，并在提交说明里写入自己的学号。
> * 执行命令 cat Untracked.txt 和 cat Stage.txt，对比两次运行的结果，体会README.txt 两次所处位置的不同。
> * 修改 README.txt 文件，再执行命令 git status > Modified.txt。
> * 执行命令 cat Modified.txt，观察其结果和第一次执行 add 命令之前的 status 是否一样，并思考原因。

**不一样**

**原因：**

> 因为创建README.txt文件时，README.txt处于未跟踪的状态，不能跟踪该文件的变化
>
> 在执行git add命令后，可将文件提交到暂存区，变为暂存状态，即可跟踪README.txt，故再对README.txt文件进行修改后，可跟踪其的变化。

#### Thinking 0.2

> 仔细看看0.10，思考一下箭头中的 add the file 、stage the file 和commit 分别对应的是 Git 里的哪些命令呢？ 

**add the file:** `git add <file>`

**stage the file:** `git add <file>`

**commit:** `git commit -m "message"`

#### Thinking 0.3

> 思考下列问题：
>
> 1. 代码文件 print.c 被错误删除时，应当使用什么命令将其恢复？
>
> 2. 代码文件 print.c 被错误删除后，执行了 git rm print.c 命令，此时应当使用什么命令将其恢复？
>
> 3.  无关文件 hello.txt 已经被添加到暂存区时，如何在不删除此文件的前提下将其移出暂存区？

1. git checkout -- print.c
2. * git reset HEAD print.c
   * git checkout -- print.c 
3. git rm --cached print.c

#### Thinking 0.4

> 思考下列有关 Git 的问题：
>
> * 找到在 /home/22xxxxxx/learnGit 下刚刚创建的 README.txt 文件，若不存在则新建该文件。
> * 在文件里加入 Testing **1**，git add，git commit，提交说明记为 **1**。
> * 模仿上述做法，把 1 分别改为 2 和 3，再提交两次。
> * 使用 git log 命令查看提交日志，看是否已经有三次提交，记下提交说明为 3 的哈希值*a*。
> * 进行版本回退。执行命令 git reset --hard HEAD^ 后，再执行 git log，观察其变化。
> * 找到提交说明为 1 的哈希值，执行命令 git reset --hard <hash> 后，再执行 git log，观察其变化。现在已经回到了旧版本，为了再次回到新版本，执行 git reset --hard <hash>，再执行 git log，观察其变化。

* **vim README.txt：**进入README.txt编辑

* **git add README.txt**

* **git commit README.txt "[123]"**

* **git log:**  提交说明为”1 2 3“均会显示

<img src="https://ooo.0x0.ooo/2024/07/02/OPgrvj.png" alt="image-20240311235422573" style="zoom:50%;" />



* **git reset --hard HEAD^ + git log**: 仅显示”1 2“的提交说明

<img src="https://ooo.0x0.ooo/2024/07/02/OPgAdp.png" alt="image-20240311235631711" style="zoom:50%;" />



* **git reset --hard e27c0fddee6312baa600c4ab7a462ffad96e1abc(提交说明为“1"的哈希值) + git log**：仅显示”1“的提交说明

<img src="https://ooo.0x0.ooo/2024/07/02/OPgZaU.png" alt="image-20240312000028929" style="zoom:50%;" />

 

* **git reset --hard bf6889e550c6a9c0266d99ee648bfd37a7e00(提交说明为“3) + git log:** 提交说明为”1 2 3“均会显示

<img src="https://ooo.0x0.ooo/2024/07/02/OPgvNY.png" alt="image-20240311235747484" style="zoom:50%;" />

#### Thinking 0.5

> 执行如下命令, 并查看结果
>
> * echo first
> * echo second > output.txt
> * echo third > output.txt
> * echo forth >> outp

<img src="https://ooo.0x0.ooo/2024/07/02/OPgyEv.png" alt="image-20240312000703998" style="zoom:80%;" />



```bash
#生成output.txt文件，将second重定向至output.txt
echo second > output.txt

#将second重定向并覆盖output.txt内容
echo third > output.txt

#将four重定向并添加到output.txt
echo four >> output.txt
```

#### Thinking 0.6

> 使用你知道的方法（包括重定向）创建下图内容的文件（文件命名为 test），将创建该文件的命令序列保存在 command 文件中，并将 test 文件作为批处理文件运行，将运行结果输出至 result 文件中。给出 command 文件和 result 文件的内容，并对最后的结果进行解释说明（可以从 test 文件的内容入手）. 具体实现的过程中思考下列问题: echoecho Shell Start 与 echo `echo Shell Start` 效果是否有区别; echo echo $c>file1与 echo `echo $c>file1` 效果是否有区别.

**commend：**

<img src="https://ooo.0x0.ooo/2024/07/02/OPggIq.png" alt="image-20240312110258910" style="zoom:62%;" />



**test:**

<img src="https://ooo.0x0.ooo/2024/07/02/OPgpFc.png" alt="image-20240312110636472" style="zoom:80%;" />

**result:**

> $[[]]: 数值计算
>
> \>|>>重定向

<img src="https://ooo.0x0.ooo/2024/07/02/OPgDdI.png" alt="image-20240312110809788" style="zoom:80%;" />

1. `echo echo Shell Start` 与 `echo echo Shell Start` 效果是否有区别

* 没有区别
* 两者都是直接标准输出echo Shell Start

2. `echo echo $c>file1`与 `echo echo $c>file1` 效果是否有区别

* 有区别
* 前者将echo 3(\$c的值)输出到file，后者标准输出echo $c>file1

#### Thinking	A.1

> 在现代的 64 位系统中，提供了 64 位的字长，但实际上不是 64 位页式存储系统。假设在 64 位系统中采用三级页表机制，页面大小 4KB。由于 64 位系统中字长为8B，且页目录也占用一页，因此页目录中有 512 个页目录项，因此每级页表都需要 9 位。因此在 64 位系统下，总共需要 3 *×* 9 + 12 = 39 位就可以实现三级页表机制，并不需要 64位。
>
> 现考虑上述 39 位的三级页式存储系统，虚拟地址空间为 512 GB，若三级页表的基地址为 PTbase，请计算：
>
> * 三级页表页目录的基地址。
> * 映射到页目录自身的页目录项（自映射）。

假设39位的[0:11],[12:20],[21:30],[31,39]依此表示页，三级页表，二级页表，一级页表

1. * 如果三级页表页目录的基地址表示二级页表的基地址：$ PT_{base} + (PT_base >> 12)*8$
   * 如果三级页表页目录的基地址表示二级页表的基地址：$ PT_{base} + (PT_base >> 12)*8 +(PT_{base} >> 21)*8 $
2. $ PT_{base}[31:39] $表示映射到页目录自身的页目录项（一级页表）内的偏移量



#### 难点分析

<img src="https://ooo.0x0.ooo/2024/07/02/OPgLaD.png" alt="image-20240314105505370" style="zoom:15%;" />

#### 实验体会

* **lab0课下：**借助实验讲解视频，实验指导书巩固linux操作命令，Vim常用功能，GCC编译器的使用，Makefile的编写，Git管理，重定向和管道，shell脚本编程等知识点，利用上述知识点基本覆盖课下实验，但是遇到gcc编译并链接两个不同文件夹的.c文件时，需使用-I选线，引入源文件的文件夹。

* **lab0-exam：**lab0课下实验的知识点覆盖这一部分练习

  > 注意生成文件的位置
  >
  > 在实验的准备阶段仔细阅读实验相关要求，例如执行命令bash init.sh

* **lab0-extra：**仔细理解题意，实验中需要的完成的任务较为简单，对`sort`，`grep`，`awk`三条命令的使用

  > 在.sh文件（shell脚本文件）编写命令时需注意传入参数的使用，不要直接用文件名，例如grep \${CMD} \${FILE}（\$CMD—寻找字符串 $FLIE—目标文件）
  >
  > 变量赋值：nowpid=\$(awk -F " " '\$2=='\$nowpid' {print \$3}' ${FILE})