---
layout: post
title: gnu ld 学习笔记
date: 2015-10-07 22:58:00
---

##GNU ld 学习笔记##

学习的主要资料是gcc-arm-none-eabi-4_9-2015q3/share/doc/gcc-arm-none-eabi/pdf/ld.pdf(version 2.24.0)


###3 Linker Scripts###
每个链接由一个linker script控制，这个脚本使用链接命令语言编写。

链接脚本的主要用途是用来描述 input files 中的section如何被映射到output files，并且控制output files 在存储器中的位置。大多数的链接脚本就做这些事情。然而，在必要时，链接脚本也能使用下面所述的命令引导连接器去执行许多其它操作。

连接器经常使用一个linker script。如果没有提供，连接器将会使用编译到连接器程序的一个默认的脚本。你可以使用 --verbose 命令行选项来显示默认的链接脚本。某些命令行选项，如-r或-N，将会影响默认的链接脚本。

你可以通过使用-T选项提供自己的linker script。当你这样 做后，你的linker script将会替换默认的脚本。

你也可以隐含的使用linker script(即不用使用-T选项指定，链接器默认链接),将linker script命名为input files，这样它们会被链接（3.11）。

###3.1 Basic Linker Script Concepts###

为了描述链接脚本语言，我们需要定义一些基本概念及术语表。

连接器把input files 融合成一个单一的 output files。output files以及每个input files处于一种特殊的数据格式，叫做object file格式。每个文件被叫做一个object file。output file 经常被叫做executable,但从我们的观点我们仍叫它object file. 每个object file 在各种东西中有一个section列表。我们一般引用input file中的section作为input section;类似，output file中的section叫做output section.


object file中的每个section 有一个name 和size.大多数的section同时有一个绑定的data块，叫做section contents.一个 section可以被标记为loadable,意味着当output file运行时,contents应当被loaded into memory。没有contents的section可以叫做allocable,意思是存储器中的一块应当被保留，但没有什么特别的东西被加载到这里（有些情况下这些存储必须被zeroed out）。一个section既不是loadable又不是allocable，通常包括一些调试信息。


每个loadable或allocable output section 有两个地址.第一种是VMA,即virtual memory address.这是当output file 运行时section的地址。第二种是LMA,即load memory address.这是secton 在何处加载的地址。大多数情况下，两种地址相同。可能不同的一个示例是，当一个data section装载到ROM中，当程序启动时拷贝到RAM中（这种技术通常用来在一个基于ROM的系统中初始化全局变量）。这种情况下ROM地址即是LMA，RAM地址即使VMA.

你可以使用objdump -h 来看到一个object file中的section.每个ojbect file也有一个symbols表，一个symbol可以使defined或undefined.在各种信息中，每个symbol有一个name,每个defined symbol有一个地址。如果你编译一个C 或C++程序到一个object file,对于每一个 defined function 及 global或static variable，你将会得到一个 defined symbol。input file中引用的每个undefined function或global variable 将会变成一个undefined symbol.

使用nm程序可以看到object file中的symbol，或者使用objdump -t.

###3.2 Linker Script Format###

Linker scripts 是文本文件。

用一系列命令来写linker script。每个命令或者是一个keyword，跟随arguments；或者是an assignment to a symbol.你可以使用分号分隔命令.空白被忽略。

file 或 format names 字符串可以被直接输入。如果一个file name 包括一个字符如逗号，逗号另一方面用作file names的分隔，你可以把file name 放在双引号中。没有办法在文件名中使用双引号。

你可以在linker script 中包括注释信息，就像C中的，通过‘/*’ 和 ‘*/’ 限定。就像C，注释在句法上等同于空白。

###3.3 Simple Linker Script Example###

许多链接脚本相当简单。

可能的最简单的链接脚本只有一条命令：'SECTIONS'.你使用这个命令来描述输出文件的内存分布。

'SECTIONS'命令是一个强大的命令。这里我们描述一个简单使用。让我们假定你的代码仅包括code，initialized data，和uninitialized data.这些将会分别在'.text','.data',和‘.bss’  sections中.让我们更进一步假定在你的input files这些是仅有的sections.

例如，我们说代码应当加载在0x10000,data加载在0x800 0000。这是一个执行这些操作的linker script：

SECTIONS
{
. = 0x10000;
.text : { *(.text) }
. = 0x8000000;
.data : { *(.data) }
.bss : { *(.bss) }
}

你用关键字 SECTIONS来写SECTIONS命令，跟随在闭合的花括号中的一系列symbol assignments和output section 描述。
上面例子中SECTIONS命令中的第一行设置特殊symbol '.'的值，它是location counter.如果你不用其它方法（其它方法稍后描述）指定一个output section的地址，地址被设置为location counter的当前值。location counter 按照output section的大小增加。在SECTIONS命令的起始处，location counter的值为0.

第二行定义了一个output section，'.text'.冒号需要的语法现在可以忽略。在output section name后边的花括号中，你列出了需要放在output section中的input sections的名字。'*'是一个通配符，匹配任何文件名。表达式'*(.text)'表示在所有的input files中的所有'.text' input sections.

... ....

3.4 Simple Linker Script Commands

3.4.4 Assign alias names to memory regions

别名可以被加到由MEMORY命令（3.7）创建的已存在的存储区域.每个name最多对应一个存储区域。
REGION_ALIAS(alias , region )

REGION_ALIAS 函数为region存储区域创建一个alias别名.这允许一个灵活的output sections映射到存储区域。下面是示例.

猜想我们有一个具有多种memory storage devices的嵌入式系统应用.都有一个通用意图的，volatile memory RAM 允许代码执行和数据存储.一些可能具有只读，non-volatile memory ROM允许代码执行和只读数据访问.最后变种是一个只读，non-volatile memory ROM2 具有只读数据访问但没有执行代码的能力.我们有四个output sections：

• .text program code;
• .rodata read-only data;
• .data read-write initialized data;
• .bss read-write zero initialized data.

目标是提供一个包括不依赖系统部分链接命令文件.我们的嵌入式系统具有三种内存设置A,B和C：

#基本理解，待完善#





###3.6 SECTIONS Command###
#已理解#
3.6.1 Output Section Description
#已理解#
3.6.2 Output Section Name
#4,5行不太理解，其它已理解#
3.6.3 Output Section Address






3.6.8.6 #已理解#


###3.7 MEMORY Command###

连接器默认配置允许分配所有可见存储。你可以使用MEMORY命令覆盖这种配置。

MEMORY命令描述target内存储块的location和size.你可以使用它来描述那些存储区域可以被链接器使用，哪些存储器区域必须避免。然后你可以将sections分配到特定的存储区域。

连接器依靠存储区域将会设置section地址，且将会警告区域满。连接器不会放置sections到可见区域。

一个 linker script 最多可能包含使用一个MEMORY 命令.然而，你可以在其中定义你期望的存储块数量。语法为：

MEMORY
{
name [(attr )] : ORIGIN = origin , LENGTH = len
...
}

name是在linker script中是使用的名字，用来引用那个区域。 region name 在linker script外无意义。region names在单独的name space中存储，不会和symbol names,file names,section names产生冲突。每个存储区域在MEMORY命令中必须有一个确切名字.然而，你可以稍后对存在的存储区域增加别名。（3.4.4）

attr字符串是一个可选的属性列表，指明了是否要对一个input section which is not explicitly mapped in the linker script使用一个特定存储区域.(3.6) .如果你不对一些input secton指定一个output section，链接器将会创建一个和input section相同名字的output section。如果你定义了区域属性，连接器将会使用它来为创建的output section选择存储区域.

attr字符串必须仅由以下字符组成：

‘R’ Read-only section
‘W’ Read/write section
‘X’ Executable section
‘A’ Allocatable section
‘I’ Initialized section
‘L’ Same as ‘I’
‘!’ Invert the sense of any of the attributes that follow

如果一个 unmapped section 匹配除'!'之外列出的任何属性,它将会被放置在存储区域中。'!' 属性反转这种test，因此一个unmapped section将会放置在只有当它和列出的属性不匹配的存储区域。

origin是一个数值表达式，表示存储区域的起始地址。表达式必须为常量值并且不能包含任何symbols.关键字ORIGIN可以被缩写为org或o(但不可以，比如，ORG).

len是一个表达式，用字节表示存储区域的大小。和origin表达式一样，必须是数值常量表达式.关键字LENGTH可以被缩写为len或l.

在下面的示例中，我们指定有两个存储区域可分配：一个在0处起始，有256KBytes，另外一个在0x4000 0000起始，有4Mbytes.链接器将会把没有准确映射到存储区域并且为只读或可执行的每个section放置到rom存储区域.连接器将会把其它没有准确映射到存储区域的section放置到ram存储区域.

MEMORY
{
rom (rx) : ORIGIN = 0, LENGTH = 256K
ram (!rx) : org = 0x40000000, l = 4M
}

一旦你定义了存储区域，你可以通过使用>region output section属性来引导链接器放置特定的output sections到存储区域.例如,如果你已经有一个内存区域叫做mem，你可以在output section definition 中使用>mem.(3.6.8.6).如果没有为output section指明地址,链接器将会把地址设置为在存储区域中下个可用地址.如果融合的output sections对于要导向的存储区域太大，连接器将会报告一个错误消息.

在表达式中通过ORIGIN(memory)和LENGTH(memory)函数，可以访问到存储的origin和length：

_fstack = ORIGIN(ram) + LENGTH(ram) - 4;
