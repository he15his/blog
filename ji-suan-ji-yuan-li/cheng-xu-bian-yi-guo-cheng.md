# c程序的执行过程



1.hello程序的生命周期是从一个高级c语言程序开始的，然后为了在系统上运行hello.c程序，每条c语句都必须被其他程序转化为一系列的低级机器语言指令。

![img](https://pic002.cnblogs.com/images/2012/422101/2012082017581666.png)

2.预处理阶段。预处理器（cpp）根据以字符#开头的命令，修改原始的C程序。#include <stdio.h>命令告诉预处理器读取系统头文件stdio.h的内容，并将它直接插入到程序文本中。结果就得到另一个C程序，通常以.i作为文件扩展名。

3.编译阶段。编译器（ccl）将文本文件hello.i翻译成文本文件hello.s。它包含一个汇编语言程序。汇编语言程序中的每条语句都以一种标准的文本格式确切地描述了一条低级机器语言指令。汇编语言为不同编译器提供了通用的输出语言。

4.汇编阶段。汇编器（as）将hello.s
翻译成机器语言指令。并将结果保存在目标文件hello.o中。hello.o是一种二进制文件。它的字节编码是机器语言指令而不是字符。

5.连接阶段。hello程序调用printf函数。它是c编译器都会提供的标准c库中的一个函数。printf函数存在于一个名为printf.o的单独的预编译好的目标文件中，而这个文件必须以某种方式合并到我们的hello.o程序中。连接器就是负责这种合并的。