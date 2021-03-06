# 程序的机器级表示

计算机执行机器代码，用字节序列编码低级的操作，包括处理数据、管理内存、读写存储设备上，以及利用网络通信。编译器基于编译语言的规则、目标机器的指令集和操作系统遵循的惯例，经过一系列的阶段生成机器代码。

GCC C语言编译器以汇编代码的形式产出输出，汇编代码是机器代码的文本显示，给出程序中的每一条指令。然后 GCC 调用汇编器和链接器，根据汇编代码生成可执行的机器代码。

当我们使用高级语言编程的时候，机器屏蔽了程序的细节，即机器级的实现。能够阅读和理解汇编代码对于程序来说是一项很重要的技能，通过阅读这些汇编代码，我们能够理解汇编器的优化能力，并分析代码中隐含的低效率。

## 历史观点

Intel 处理器系列简称 x86，每个后继处理器的设计都是后向兼容的 —— 较早版本上编译的代码可以在较新的处理器上运行。

> 摩尔定律 —— 芯片上的晶体每年都会翻一番。（见下图）

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/28.png)

## 程序编码

考虑以下编译过程（如下图）

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/29.png)

在上图中，gcc （GCC C 编译器）命令调用了一整套的程序，将源代码转为可执行代码。
  - 首先，C 预处理器扩展源代码，插入所有用 #include 命令指定的文件，并扩展所有用 #define 声明指定的宏。
  - 其次，编译器产生两个源文件的汇编代码，名字分别为 p1.s 和 p2.s。
  - 接下来，汇编器会将汇编代码转化为二进制目标代码文件 p1.o 和 p2.o。目标代码是机器代码的一种形式，它包含所有指令的二进制表示，但是还没有填入全局值的地址。
  - 最后，链接器将两个目标代码文件与实现库函数（例如 printf）的代码合并，并产生最终的可执行代码文件 p。（由命令指示符 -o p 指定的）。可执行代码是我们要考虑的机器代码的第二种形式，也就是处理器执行的代码格式。

> GCC 编译器快捷命令
> gcc -Og -S main.c

### 机器级代码

计算机系统使用了多种不同形式的抽象，对于机器级编程来说，其中两种抽象尤为重要：

  1. 第一种是由 `指令集体系结构或指令集架构（Instruction Set Architecture, ISA）` 来定义机器级程序的格式和行为，它定义了处理器状态、指令的格式，以及每条指令对状态的影响。
  2. 第二种抽象是机器级程序使用的内存地址是虚拟地址，提供的内存模型看上去是一个非常大的字节数组。

汇编代码非常接近于机器代码。与机器代码的二进制相比，汇编代码的主要特点是它用可读性更好的文本格式表示。

x86-64 的机器代码和原始的 C 代码差别特别大。一些对 C 语言程序员隐藏的处理器状态都是可见的：

  - `程序计数器`（通常称为 “PC”，在 x86-64 中用 `%rip` 来表示）给出将要执行的下一条指令在内存中的地址。
  - `整数寄存器` 文件包含 16 个命名的位置，分别存储 64 位的值。这些寄存器可以存储地址（对应 C 语言的指针）或整数数据。有的寄存器被用来记录某些重要的程序状态，而其他的寄存器用来保存临时数据，例如过程的参数和局部变量，以及函数的返回值。
  - `条件码寄存器` 保存着最近执行的算术或逻辑指令的状态信息。它们用来实现控制或数据流中的条件变化，比如用来实现 if 或 while 语句。
  - 一组 `向量寄存器` 可以存放一个或多个整数或浮点数值。

程序内存包含：程序的可执行机器代码、操作系统需要的一些信息、用来管理过程调用和返回的运行时栈、用户分配的内存块（比如用 malloc 库函数分配的）。

### 代码示例

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/53.png)

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/54.png)

机器执行的代码只是一个字节序列，它是对一系列指令的编码，机器对产生这些指令的源代码几乎一无所知。

要查看机器代码文件的内容，有一类称为 `反汇编器（disassembler）` 的程序非常有用，这些程序根据机器代码产生一种类似于汇编代码的格式。（如下图）

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/55.png)

其中一些关于机器代码和它的反汇编的表示的特性值得注意：

  - x86-64 的指令长度从 1 到 15 个字节不等。常用的指令以及操作数减少的指令所需的字节数少，而那些不太常用或操作数较多的指令所需的字节数较多。
  - 设计指令格式的方法是，从某个给定位置开始，可以将字节唯一解码成机器指令。例如，只有指令 pushq %rbx 是以字节值 53 开头的。
  - 反汇编器只是基于机器代码文件中的字节序列来确定汇编代码。它不需要访问该程序的源代码或汇编代码。
  - 反汇编器使用的指令命名规则与 GCC 生成的汇编代码使用的有些细微的差别。在我们的示例中，它省略了很多指令结尾的 `q`。这些后缀是大小指示符，在大多数情况中可以省略。相反，反汇编器 call 和 ret 指令添加了 `q` 后缀，同样，省略这些后缀也没有问题。

### 关于格式的注解

所有以 `.` 开头的行都是指导汇编器和链接器工作的伪指令。我们通常可以忽略这些行。另一方面，也没有关于指令的用途以及它们与源代码之间关系的解释说明。

在 C 程序中插入汇编代码有两种方法。第一种是，我们可以编写完整的函数，放进一个独立的汇编代码文件中，让汇编器和链接器把它和 C 语言书写的代码结合起来。第二种方法是，我们可以使用 GCC 的内联汇编（inline assembly）特性，用 asm 伪命令可以在 C 程序中包含简短的汇编代码 —— 这种方法的好处是减少了与机器相关的代码量。

## 数据格式

由于是从 16 位体系结构扩展成 32 位，Intel 用术语 `字（word）` 表示 16 位数据类型。下面是 C 语言数据类型在 x86-64 中的大小（见下图）

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/56.png)

大多数 GCC 生成的汇编代码指令都有一个字符的后缀，表明操作数的大小。例如，数据传送指令有四个变种：movb（传送字节）、movw（传送字）、movl（传送双字）、movq（传送四字）。后缀 `l` 用来表示双字，汇编代码也使用后缀 `l` 来表示 4 字节整数和 8 字节双精度浮点数。这不会产生歧义，因为浮点数使用的是一组完全不同的指令和寄存器。

## 访问信息

一个 x86-64 的中央处理单元（CPU）包含一组 16 个存储 64 位值的通用目的寄存器，这些寄存器用来存储整数数据和指针。它们的名字都以 %r 开头。（如下图）

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/57.png)

如上图所示，指令可以对这 16 个寄存器的低位字节中存放的不同大小的数据进行操作。字节级操作可以访问最低的字节，16 位操作可以访问最低的两个字节，32 位操作可以访问最低的 4 字节，而 64 位操作可以访问整个寄存器。

### 操作数指示符

大多数指令有一个或多个操作数（operand），指示出执行一个操作中要使用的源数据值，以及放置结果的目的地址。

操作数一般有三种类型：

  1. `立即数（immediate）`：立即数用来表示常数值。（比如 $-577 或 $0x1F）
  2. `寄存器（register）`：它表示某个寄存器中的内容，16 个寄存器的低位 1 字节、2 字节、4 字节或8 字节中的一个作为操作数。
  3. `内存引用`：内存引用会根据计算出来的地址（通常称为有效地址）访问某个内存位置。

内存寻址有多种不同的寻址方式（如下图）

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/58.png)

### 数据传送指令

最频繁使用的指令是将数据从一个位置复制到另一个位置的指令。

最简单的数据传送指令是 MOV 类，这些指令把数据从源位置复制到目的位置，不做任何变化（如下图）

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/59.png)

源操作数指定的值是一个立即数，存储在寄存器或者内存中。目的操作数指定一个位置，要么是一个寄存器要么是一个内存地址。x86-64 加了一条限制，传送指令的两个操作数不能都指向内存位置。将一个值从一个内存位置复制到另一个内存位置需要两条指令 —— 第一条指令将源值加载到寄存器中，第二条指令将寄存器值写入目的位置。

下面的 MOV 指令示例给出了源和目的类型的五种可能的组合。（第一个是源操作数，第二个是目的操作数）

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/60.png)

在将较小的源复制到较大的目的地时使用 MOVZ 和 MOVS 类。MOVZ 类中的指令把目的中剩余的字节填充为 0，而 MOVS 类中的指令通过符号扩展来填充，把源操作的最高位进行复制。（如下图）

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/61.png)

### 数据传送示例

下面是一个使用数据传送指令的示例。（如下图）

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/62.png)

如上图所示，函数 `exchange` 由三条指令实现：两条数据传送（movq），加上一条函数返回指令被调用点的指令（ret）。

当过程开始执行时，过程参数 `xp` 和 `y` 分别存储在寄存器 `%rdi` 和 `%rsi` 中。然后，指令 2 从内存中读出 x，把它放到寄存器 `%rax` 中，直接实现了 C 程序中的操作 `x = *xp`。稍后，用寄存器 `%rax` 从这个函数中返回一个值，因而返回值就是 x。指令 3 将 y 写入到寄存器 %rdi 中的 xp 指向的内存位置，直接实现了操作 `*xp=y`。这个例子说明了如何用 MOV 指令从内存中读值到寄存器（第 2 行），如何从寄存器中写到内存（第 3 行）。

这段汇编代码有两点值得注意。首先，我们看到 C 语言中所谓的 “指针” 其实就是地址。间接引用指针就是将该指针放在一个寄存器中，然后在内存使用中使用这个寄存器。其次，像 x 这样的局部变量通常是保存在寄存器中，而不是内存中。访问寄存器要比访问内存要快得多。

#### C 语言解析

`long x = *xp`

上面这条语句表示我们将读存储在 `xp` 所指位置中的值，并把它存放到名字为 `x` 的局部变量中。这个读操作称作指针的间接引用（pointer dereferencing），C 操作符 * 执行指针的间接引用。

`*xp = y`

上面这条语句将参数 `y` 的值写到 `xp` 所指的位置。这也是指针间接引用的一种形式（所以有操作符 *），但是它表明的是一个写操作，因为它在赋值语句的左边。

`exchange(&a, 3)`

C 操作符 `&`（成为 “取址” 操作符）创建一个指针，在本例中，该指针指向保存局部变量 `x` 的位置。

### 压入和弹出栈数据

`pushq` 指令的功能是把数据压入到栈上，而 `popq` 指令是弹出数据。这些指令都只有一个操作数——压入的数据源和弹出的数据目的。

将一个四字值压入栈中，首先要将栈指针减少 8，然后将值写到新的栈顶地址。（下图是一个案例）

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/64.png)

弹出一个四字的操作包括从栈顶位置读出数据，然后将栈指针增加 8。


## 算术和逻辑操作

大多数整数和逻辑操作都分成了指令类，这些指令有各种带不同大小操作数的变种。（如下图）

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/65.png)

### 加载有效地址

加载有效地址（load effective address）指令 leaq 实际上是 movq 指令的变形。该指令并不是从指定的位置读入数据，而是将有效地址写入到目的操作数，目的操作数必须是一个寄存器。（下图是案例）

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/66.png)

### 一元和二元操作

一元操作只有一个操作数，既是源又是目的。这个操作数可以是一个寄存器，也可以是一个内存位置。比如说，指令 `incq(%rsp)` 会使栈顶的 8 字节元素加 1。（类似于 ++ 运算符）

二元操作有两个操作数，第二个操作数既是源又是目的（类似于赋值运算符，x += y）。第一个操作数可以立即数、寄存器或内存位置，第二个操作数可以是寄存器或是内存位置。

### 移位操作

移位操作有两个操作数，先给出移位量，然后第二项给出的是要移位的数。

下图是一个执行算术操作的函数示例（如下图）

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/67.png)

在上面这个案例中，寄存器 `%rax` 中的值先后对应于程序值 3 * z、z * 48 和 t4（作为返回值）。通常，编译器产生的代码中，会用一个寄存器存放多个程序值，还会在寄存器之间传送程序值。


## 控制

机器代码提供两种基本的低级机制来实现有条件的行为：测试数据值，然后根据测试的结果来改变控制流或者数据流。

### 条件码

除了整数寄存器，CPU 还维护着一组单个位的条件码（condition code）寄存器，它们描述了最近的算术或逻辑操作的属性。（如下图）

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/68.png)

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/69.png)

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/70.png)

### 访问条件码

条件码通常不会直接读取，常用的使用方法有三种：

  - 可以根据条件码的某种组合，将一个字节设置为 0 或者 1；
  - 可以条件跳转到程序的某个其他的部分；
  - 可以有条件地传送数据；

根据条件码的某种组合，将一个字节设置为 0 或者 1，这一整类指令称为 SET 指令（如下图）。

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/71.png)

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/72.png)

### 跳转指令

`跳转(jump)` 指令会导致执行切换到程序中一个全新的位置。在汇编代码中，这些跳转的目的地通常用一个标号（label）标明（如下图）。

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/73.png)

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/74.png)

### 将条件表达式来实现条件分支

用条件表达式和语句从 C 语言翻译成机器代码，最常用的方式是结合有条件和无条件跳转。

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/75.png)

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/76.png)

### 用条件传送来实现条件分支

数据的条件转移计算一个条件操作的两种结果，然后再根据条件是否满足从中选取一个。只有在一些受限制的情况中，这种策略才可行，但是如果可行，就可以用一条简单的 `条件传送` 指令来实现它，条件传送指令更符合现代处理器的性能特性（如下图）。

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/77.png)

基于条件数据传送的代码会比基于条件控制转移的代码性能要好，这是因为现代处理器通过使用流水线（pipelining）来获得高性能，在流水线中，一条指令的处理要经过一系列的阶段，每个阶段执行所需操作的一小部分（例如，从内存取指令、确定指令类型、从内存读数据、执行算术运算、向内存写数据，以及更新程序计数器）。这种方法通过重叠连续指令的步骤来获得高性能，例如，在取一条指令的同时，执行它前面一条指令的算术运算。要做到这一点，要求能够实现确定要执行的指令序列，这样才能保证流水线中充满了待执行的指令。当机器遇到条件跳转（也称为“分支”时），只有当分支条件求值完成之后，才能决定分支往哪边走。

条件传送指令（如下图）

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/78.png)

同条件跳转不同，处理器无需预测测试的结果就可以执行条件传送。处理器只是读源值（可能是从内存中），检查条件码，然后要么更新目的寄存器，要么保持不变（如下图）。

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/79.png)

总的来说，条件数据传送提供了一种用条件控制转移来实现条件操作的替代策略，它们只能用于非常受限制的情况。

### 循环

#### do while

C 语言提供了多种循环结构，即 `do-while、while 和 for`。汇编中没有响应的指令存在，可以用条件测试和跳转组合起来实现循环的效果（如下图）。

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/80.png)

> 根据我们的经验，GCC 常常做的一些变换，非但不能带来性能好处，反而甚至可能降低代码性能。

#### while

第一种方法，叫做跳转到中间（jump to middle）（如下图）

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/113.png)

第二种方法，叫做 `guarded-do`。（如下图）

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/114.png)

#### for

GCC 为 for 循环产生的代码是 while 循环的两种翻译之一，这取决于优化的等级。


### switch 语句

switch（开关）语句可以根据一个整数索引值进行多重分支（multiway branching）。（如下图）

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/115.png)

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/116.png)



## 过程

过程是软件中一种很重要的抽象。它提供了一种封装代码的方式，用一组指定的参数和可选的返回值实现了某种功能。然后，可以在程序不同的地方调用这个函数。

要提供对过程的机器级支持，必须要处理许多不同的属性。为了讨论方便，假设过程 P 调用过程 Q，Q 执行后返回到 P。这些动作包括下面一个或多个机制：

  - 传递控制。在进入过程 Q 的时候，程序计数器必须被设置为 Q 的代码的起始地址，然后在返回时，要把程序计数器设置为 P 中调用 Q 后面那条指令的地址。
  - 传递数据。P 必须能够向 Q 提供一个或多个参数，Q 必须能向 P 返回一个值。
  - 分配和释放内存。在开始时，Q 可能需要为局部变量分配空间，而在返回前，又必须释放这些存储空间。

### 运行时栈

C 语言过程调用机制的一个关键特性在于使用了栈数据结构提供的后进先出的内存管理原则。（如下图）

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/117.png)

### 转移控制

将控制从函数 P 转移到函数 Q 只需要简单地把程序计数器（PC）设置为 Q 的代码的起始位置。不过，当稍后从 Q 返回的时候，处理器必须处记录好它需要继续 P 的执行的代码的位置（如下图）。

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/118.png)

### 数据传送

当调用一个过程时，除了要把控制传递给它并在过程返回时再传递回来之外，过程调用还可能包括把数据作为参数传递，而从过程返回还有可能包括返回一个值，而从过程返回还有可能包括返回一个值。x86-64 中，大部分过程间的数据传送是通过寄存器实现的（如下图）。

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/119.png)

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/120.png)

### 栈上的局部存储

在某些时候，局部数据必须存放在内存中，常见的情况包括：

  - 寄存器不足够存放所有的本地数据。
  - 对一个局部变量使用地址运算符 `&`，因此必须能够为它产生一个地址。
  - 某些局部变量是数组或结构，因此必须能够通过数组或结构引用被访问到。

示例：

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/121.png)

### 寄存器中的局部存储空间

寄存器组是唯一被所有过程共享的资源。虽然在给定时刻只有一个过程是活动的，我们仍然必须确保当一个过程（调用者）调用另一个过程（被调用者）时，被调用者不会覆盖调用者稍后会使用的寄存器值（如下图）。

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/122.png)

### 递归过程

寄存器和栈的惯例使得 x86-64 过程能够递归地调用它们自身。每个过程调用在栈中都有它自己的私有空间，因此多个未完成调用的局部变量不会相互影响。此外，栈的原则很自然地就提供了适当的策略，当过程被调用时分配局部存储，当返回时释放存储（如下图）。

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/123.png)



## 数组的分配和访问

C 语言中的数组是一种将标量数据聚集成更大数据类型的方式。C 语言实现数组的方式非常简单，因此很容易翻译成机器代码。C 语言一个不同寻常的特点是可以产生指向数组中元素的指针，并对这些指针进行运算。在机器代码中，这些指针会被翻译成地址计算。

### 基本原则

对于数据类型 T 和整型常数 N，声明如下：

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/124.png)

### 指针运算

C 语言允许对指针进行运算，而计算出来的值会根据该指针引用的数据类型的大小进行伸缩（如下图）。

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/125.png)

### 嵌套的数组

要访问多维数组的元素，编译器会以数组起始为基地址，（可能需要经过伸缩的）偏移量为索引，产生计算期望的元素的偏移量，然后使用某种 MOV 指令。通常来说，对于一个声明如下的数组：

T D[R][C]

它的数组元素 D[i][j] 的内存地址为：

&D[i][j] = x_D + L(C * i + j)

例如 `int A[5][3]`，存储地址如下图：

![image](http://shadows-mall.oss-cn-shenzhen.aliyuncs.com/images/assets/cs/126.png)

### 定长数组

