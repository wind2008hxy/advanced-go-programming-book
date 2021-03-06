# 3.2. 计算机结构

汇编语言是直面计算机的编程语言，因此理解计算机结构是掌握汇编语言的前提。当前流行的计算机基本采用的是冯·诺伊曼计算机体系结构（在某些特殊领域还有哈佛体系架构）。冯·诺依曼结构也称为普林斯顿结构，采用的是一种将程序指令和数据存储在一起的存储结构。冯·诺伊曼计算机中的指令和数据存储器其实指的是计算机中的内存，然后在配合CPU处理器就组成了一个最简单的计算机了。

汇编语言其实是一种非常简单的编程语言，因为它面向的计算机模型就是非常简单的。让人觉得汇编语言难学主要有几个原因：不同类型的CPU都有自己的一套指令；即使是相同的CPU，32位和64位的运行模式依然会有差异；不同的汇编工具同样有自己特有的汇编指令；不同的操作系统和高级编程语言和底层汇编的调用规范并不相同。本节将描述几个有趣的汇编语言模型，最后精简出一个适用于AMD64架构的精简指令集，以便于Go汇编语言的学习。


## 图灵机和BF语言

图灵机是由图灵提出的一种抽象计算模型。机器有一条无限长的纸带，纸带分成了一个一个的小方格，每个方格有不同的颜色，这类似于计算机中的内存。同时机器有一个探头在纸带上移来移去，类似于通过内存地址来读写内存上的数据。机器头有一组内部计算状态，还有一些固定的程序（更像一个哈佛结构）。在每个时刻，机器头都要从当前纸带上读入一个方格信息，然后根据自己的内部状态和当前要执行的程序指令将信息输出到纸带方格上，同时更新自己的内部状态并进行移动。

图灵机虽然不容易编程，但是非常容易理解。有一种极小化的BrainFuck计算机语言，它的工作模式和图灵机非常相似。BrainFuck由Urban Müller在1993年创建的，简称为BF语言。Müller最初的设计目标是建立一种简单的、可以用最小的编译器来实现的、符合图灵完全思想的编程语言。这种语言由八种状态构成，早期为Amiga机器编写的编译器（第二版）只有240个字节大小！

就象它的名字所暗示的，brainfuck程序很难读懂。尽管如此，brainfuck图灵机一样可以完成任何计算任务。虽然brainfuck的计算方式如此与众不同，但它确实能够正确运行。这种语言基于一个简单的机器模型，除了指令，这个机器还包括：一个以字节为单位、被初始化为零的数组、一个指向该数组的指针（初始时指向数组的第一个字节）、以及用于输入输出的两个字节流。这种 语言，是一种按照“Turing complete（完整图灵机）”思想设计的语言，它的主要设计思路是：用最小的概念实现一种“简单”的语言，BrainF**k 语言只有八种符号，所有的操作都由这八种符号的组合来完成。

下面是这八种状态的描述，其中每个状态由一个字符标识：

| 字符 | C语言类比          | 含义
| --- | ----------------- | ------
| `>` | `++ptr;`          | 指针加一
| `<` | `--ptr;`          | 指针减一
| `+` | `++*ptr;`         | 指针指向的字节的值加一
| `-` | `--*ptr;`         | 指针指向的字节的值减一
| `.` | `putchar(*ptr);`  | 输出指针指向的单元内容（ASCⅡ码）
| `,` | `*ptr = getch();` | 输入内容到指针指向的单元（ASCⅡ码）
| `[` | `while(*ptr) {}`  | 如果指针指向的单元值为零，向后跳转到对应的 `]` 指令的次一指令处
| `]` |                   | 如果指针指向的单元值不为零，向前跳转到对应的 `[` 指令的次一指令处

下面是一个 brainfuck 程序，向标准输出打印"hi"字符串：

```
++++++++++[>++++++++++<-]>++++.+.
```

理论上我们可以将BF语言当作目标机器语言，将其它高级语言编译为BF语言后就可以在BF机器上运行了。

## 人力资源机器游戏

《人力资源机器》（Hunman Resource Machine）是一款设计精良汇编语言编程游戏。在游戏中，玩家扮演一个职员角色，来模拟人力资源机器的运行。通过完成上司给的每一份任务来实现晋升的目标，完成任务的途径就是用游戏提供的11个机器指令编写正确的汇编程序，最终得到正确的输出结果。人力资源机器的汇编语言可以认为是跨平台、跨操作系统的通用的汇编语言，因为在macOS、Windows、Linux和iOS上该游戏的玩法都是完全一致的。

人力资源机器的机器模型非常简单：INBOX命令对应输入设备，OUTBOX对应输出设备，玩家小人对应一个寄存器，临时存放数据的地板对应内存，然后是数据传输、加减、跳转等基本的指令。总共有11个机器指令:

| 名称      | 解释 |
| -------- | ---
| INBOX    | 从输入通道取一个整数数据，放到手中(寄存器)
| OUTBOX   | 将手中（寄存器）的数据放到输出通道，然后手中将没有数据（此时有些指令不能运行）
| COPYFROM | 将地板上某个编号的格子中的数据复制到手中（手中之前的数据作废），地板格子必须有数据
| COPYTO   | 将手中（寄存器）的数据复制到地板上某个编号的格子中，手中的数据不变
| ADD      | 将手中（寄存器）的数据和某个编号对应的地板格子的数据相加，新数据放到手中（手中之前的数据作废）
| SUB      | 将手中（寄存器）的数据和某个编号对应的地板格子的数据相减，新数据放到手中（手中之前的数据作废）
| BUMP+    | 自加一
| BUMP-    | 自减一
| JUMP     | 跳转
| JUMP =0  | 为零条件跳转
| JUMP <0  | 为负条件跳转

除了机器指令外，游戏中有些环节还提供类似寄存器的场所，用于存放临时的数据。人力资源机器游戏的机器指令主要分为以下几类：

- 输入/输出(INBOX, OUTBOX): 输入后手中将只有1份新拿到的数据, 输出后手中将没有数据。
- 数据传输指令(COPYFROM/COPYTO): 主要用于仅有的1个寄存器（手中）和内存之间的数据传输，传输时要确保源数据是有效的
- 算术相关(ADD/SUB/BUMP+/BUMP-)
- 跳转指令: 如果是条件跳转，寄存器中必须要有数据

主流的处理器也有类似的指令。除了基本的算术和逻辑预算指令外，再配合有条件跳转指令就可以实现分支、循环等常见控制流结构了。

下图是某一层的任务：将输入数据的0剔除，非0的数据依次输出，右边部分是解决方案。

![](../images/ch3-02-arch-hsm-zero.jpg)

整个程序只有一个输入指令、一个输出指令和两个跳转指令共四个指令：

```
LOOP:
	INBOX
	JUMP-if-zero LOOP
	OUTBOX
	JUMP LOOP
```

首先通过INBOX指令读取一个数据包；然后判断包裹的数据是否为0，如果是0的话就跳转到开头继续读取下一个数据包；否则将输出数据包，然后再跳转到开头。以此循环无休止地处理数据包裹，直到任务完成晋升到更高一级的岗位，然后处理类似的但更复杂的任务。


## 精简X86-64指令集

X86其实是是80X86的简称（后面三个字母），包括Intel 8086、80286、80386以及80486等指令集合，因此其架构被称为x86架构。x86-64是AMD公司于1999年设计的x86架构的64位拓展，向后兼容于16位及32位的x86架构。X86-64目前正式名称为AMD64，也就是Go语言中GOARCH环境变量指定的AMD64。如果没有特殊说明的话，本章中的汇编程序都是针对64位的X86-64环境。

很多汇编语言的教程都会强调汇编语言是不可移植的。严格来说很多汇编语言在不同的CPU类型、或不同的操作系统环境、或不同的汇编工具链下是不可移植的。而这种不可移植性正是汇编语言普及的一个极大的障碍。虽然CPU指令集的差异是导致不好移植的较大因素，但是汇编语言的相关工具链对此也有不可推卸的责任。而源自Plan9的Go汇编语言对此做了一定的改进：首先Go汇编语言在相同CPU架构上是完全一致的，也就是屏蔽了操作系统的差异；同时Go汇编语言将一些基础并且类似的指令抽象为相同名字的伪指令，从而减少不同CPU架构下汇编代码的差异（当然，寄存器名字和数量的差异是一直存在的）。本节的目的也是找出一个较小的精简指令集，以简化Go汇编语言的学习。

下面是X86/AMD架构图：

![](../images/ch3-arch-amd64-01.ditaa.png)

寄存器是CPU中最重要的资源，每个要处理的内存数据原则上需要先放到寄存器中才能由CPU处理，同时寄存器中处理完的结果需要再存入内存。X86中除了状态寄存器和指令指令两个特殊的寄存器外，还有AX、BX、CX、DX、SI、DI、BP、SP几个通用寄存器。在X86-64中又增加了八个以R8-R15方式命名的通用寄存器。因为历史的原因R0-R7并不是通用寄存器，它们只是X87开始引入的MMX指令专有的寄存器。在通用寄存器中BP和SP是两个比较特殊的寄存器：其中BP用于记录当前函数帧的开始位置，和函数调用相关的指令会隐式地影响SP的值；SP则对应当前栈指针的位置，和栈相关的指令会隐式地影响SP的值。

X86是一个极其复杂的系统，有人统计x86-64中指令有将近一千个之多。不仅仅如此，X86中的很多单个指令的功能也非常强大，比如有论文证明了仅仅一个MOV指令就可以构成一个图灵完备的系统。以上这是两种极端情况，太多的指令和太少的指令都不利于汇编程序的编写。通用的基础机器指令大概可以分为数据传输指令、算术运算和逻辑运算指令、控制流指令等几类。因此我们将尝试精简出一个X86-64指令集，以便于Go汇编语言的学习。

基础的数据传输指令有MOV、LEA、PUSH、POP等几个。其中MOV指令可以用于将字面值移动到寄存器、字面值移到内存、寄存器之间的数据传输、寄存器和内存之间的数据传输。需要注意的是，MOV传输指令的内存操作数只能有一个，可以通过某个临时寄存器达到类似目的。LEA指令将标准参数格式中的内存地址加载到寄存器（而不是加载内存位置的内容）。PUSH和POP分别是压栈和出栈指令，通用寄存器中的SP为栈指针，栈是向低地址方向增长的。

| 名称    | 解释 |
| ------ | ---
| MOV    | 数据转移
| LEA    | 取地址
| PUSH   | 压栈
| POP    | 出栈


基础算术指令有ADD、SUB、MUL、DIV等指令。其中ADD、SUB、MUL、DIV用于加、减、乘、除运算，最终结果存入目标寄存器。基础的逻辑运算指令有AND、OR和NOT等几个指令，对应逻辑与、或和取反等几个指令。


| 名称    | 解释 |
| ------ | ---
| ADD    | 加法
| SUB    | 减法
| MUL    | 乘法
| DIV    | 除法
| AND    | 逻辑与
| OR     | 逻辑或
| NOT    | 逻辑取反

控制流指令有CMP、JMP-if-x、JMP、CALL、RET等指令。CMP指令用于两个操作数做减法，根据比较结果设置状态寄存器的符号位和零位，可以用于有条件跳转的跳转条件。JMP-if-x是一组有条件跳转指令，常用的有JL、JLZ、JE、JNE、JG、JGE等指令，对应小于、小于等于、等于、不等于、大于和大于等于等条件时跳转。JMP指令则对应无条件跳转，将要跳转的地址设置到IP指令寄存器就实现了跳转。而CALL和RET指令分别为调用函数和函数返回指令。


| 名称      | 解释 |
| -------- | ---
| JMP      | 无条件跳转
| JMP-if-x | 有条件跳转，JL、JLZ、JE、JNE、JG、JGE
| CALL     | 调用函数
| RET      | 函数返回


为了简单我们省略了位运算指令，很多高级指令。完整的X86指令在 https://github.com/golang/arch/blob/master/x86/x86.csv 文件定义。同时Go汇编还正对一些指令定义了别名，具体可以参考这里 https://golang.org/src/cmd/internal/obj/x86/anames.go 。

## Go汇编中的伪寄存器

Go汇编为了简化汇编代码的编写，引入了PC、FP、SP、SB四个伪寄存器。四个伪寄存器和X86/AMD64的内存和寄存器的相互关系如下图：

![](../images/ch3-arch-amd64-02.ditaa.png)

在AMD64环境，伪PC寄存器其实是IP指令计数器寄存器的别名。伪FP寄存器对应的是函数的帧指针，一般用来访问函数的参数和返回值。伪SP栈指针对应的是当前函数栈帧的底部（不保护参数和返回值部分），一般用于定位局部变量。伪SP是一个比较特殊的寄存器，因为还存在一个同名的SP真寄存器。真SP寄存器对应的是栈的顶部，一般用于定位调用其它函数的参数和返回值。

当需要区分伪寄存器和真寄存器的时候只需要记住一点：伪寄存器一般需要一个标识符和偏移量为前缀，如果没有标识符前缀则是真寄存器。比如`(SP)`、`+8(SP)`没有标识符前缀为真SP寄存器，而`a(SP)`、`b+8(SP)`有标识符为前缀表示伪寄存器。


## Go函数调用规范

和C语言函数不同，Go语言函数的参数和返回值完全通过栈传递。下面是Go函数调用时栈的布局图：

![](../images/ch3-func-stack-frame-layout-01.ditaa.png)

在汇编定义函数时，我们需要关注framesize和argsize，分别对应函数帧大小和函数参数和返回值的大小。

因为函数的参数和返回值大小可以通过Go函数的签名解析得到，因此argsize一般是可以省略的。需要注意的是，输入参数和返回值依此从低地址向高地址顺序排列。同时每个参数的类型需要满足地址对齐要求。

帧大小相对复杂一点：其中包含函数的局部变量和调用其它函数时的参数和返回值空间。局部变量也是从低地址向高地址顺序排列的，因此它们和栈增长方向是相反的。

在最下面灰色的部分是调用函数后的返回地址。当执行CALL指令时，会自动将SP向下移动，并将返回地址和SP寄存器存入栈中。然后被调用的函数执行RET返回指令时，先从栈恢复BP和SP寄存器，接着取出的返回地址跳转到对应的指令执行。

## MOV指令

MOV指令是最重要的机器指令，它不仅仅用于在寄存器和内存之间传输数据，而且还可以用于处理数据的扩展和截断操作。

最简单的是忽略符号位的数据传输操作，386和AMD64指令一样，不同的1、2、4和8字节宽度有不同的指令：

| Data Type | 386/AMD64   | Comment       |
| --------- | ----------- | ------------- |
| [1]byte   | MOVB        | B => Byte     |
| [2]byte   | MOVW        | W => Word     |
| [4]byte   | MOVL        | L => Long     |
| [8]byte   | MOVQ        | Q => Quadword |

但是当数据宽度和寄存器的宽度不同又需要处理符号位时，386和AMD64有各自不同的指令：

| Data Type | 386     | AMD64   | Comment       |
| --------- | ------- | ------- | ------------- |
| int8      | MOVBLSX | MOVBQSX | sign extend   |
| uint8     | MOVBLZX | MOVBQZX | zero extend   |
| int16     | MOVWLSX | MOVWQSX | sign extend   |
| uint16    | MOVWLZX | MOVWQZX | zero extend   |

比如当需要将一个int64类型的数据转为bool类型时，则需要使用MOVBQZX指令处理。
