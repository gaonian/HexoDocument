---
title : Mac OS X ABI Mach-O File Format Reference
categories: iOS
---

# Overview

This document describes the structure of the Mach-O (Mach object) file format, which is the standard
used to store programs and libraries on disk in the Mac OS X application binary interface (ABI). To
understand how the Xcode tools work with Mach-O files, and to perform low-level debugging tasks,
you need to understand this information.

本文档描述了Mach- o (Mach object)文件格式的结构，这是Mac OS X应用程序二进制接口(ABI)中用于在磁盘上存储程序和库的标准格式。要理解Xcode工具如何处理Mach-O文件，并执行 low-level 调试任务，您需要理解这些信息。



The Mach-O file format provides both intermediate (during the build process) and final (after linking
the final product) storage of machine code and data. It was designed as a flexible replacement for the
BSD a.out format, to be used by the compiler and the static linker and to contain statically linked
executable code at runtime. Features for dynamic linking were added as the goals of Mac OS X evolved,
resulting in a single file format for both statically linked and dynamically linked code.

Mach-O文件格式提供了机器代码和数据的中间(在构建过程中)和最终(在链接最终产品之后)存储。它被设计为一个灵活的替代BSD `a.out`输出格式，由编译器和静态链接器使用，并在运行时包含静态链接的可执行代码。随着Mac OS X目标的发展，动态链接的特性也被添加进来，从而为静态链接和动态链接代码提供了一种单一的文件格式。



## Basic Structure

A Mach-O file contains three major regions (as shown in Figure 1):

一个Mach-O文件包含三个主要区域(如图1所示):

- At the beginning of every Mach-O file is a **header structure** that identifies the file as a Mach-O file. The header also contains other basic file type information, indicates the target architecture, and contains flags specifying options that affect the interpretation of the rest of the file. 

  在每个Mach-O文件的开头都有一个头结构，它将文件标识为一个Mach-O文件。头文件还包含其他基本文件类型信息，指示目标体系结构，并包含指定影响文件其余部分解释的选项的标志。

- Directly following the header are a series of variable-size **load commands** that specify the layout and linkage characteristics of the file. Among other information, the load commands can specify: 

  头文件后面是一系列大小可变的**load commands**，它们指定文件的布局和链接特性。在其他信息中，load commands可以指定:

  - The initial layout of the file in virtual memory 

    文件在虚拟内存中的初始布局

  - The location of the symbol table (used for dynamic linking) 

    符号表的位置(用于动态链接)

  - The initial execution state of the main thread of the program

    程序主线程的初始执行状态

  - The names of shared libraries that contain definitions for the main executable’s imported
    symbols

    包含主可执行文件导入符号定义的共享库的名称

- Following the load commands, all Mach-O files contain the data of one or more segments. Each **segment** contains zero or more sections. Each **section** of a segment contains code or data of some particular type. Each segment defines a region of virtual memory that the dynamic linker maps into the address space of the process. The exact number and layout of segments and sections is specified by the load commands and the file type. 

  按照load commands, Mach-O文件都包含一个或多个**segment**的数据。每个段包含零个或多个**sections**。段的每个**section**都包含某种特定类型的代码或数据。每个**segment**定义一个虚拟内存区域，动态链接器将该区域映射到进程的地址空间。**segment**和**sections**的确切数量和布局由`load commands`和文件类型指定。

- In user-level fully linked Mach-O files, the last segment is the **link edit** segment. This segment contains the tables of link edit information, such as the symbol table, string table, and so forth, used by the dynamic loader to link an executable file or Mach-O bundle to its dependent libraries. 

  在用户级完全链接的Mach-O文件中，最后一个段是链接编辑段。此段包含链接编辑信息的表，如符号表、字符串表等，动态加载程序使用这些表将可执行文件或Mach-O包链接到其依赖库。

  

  **Figure 1** Mach-O file format basic structure 

  ![mach-o](https://raw.githubusercontent.com/gaonian/HexoDocument/master/iOS/macho_image/machoformat2.png)

Various tables within a Mach-O file refer to sections by number. Section numbering begins at 1 (not
0) and continues across segment boundaries. Thus, the first segment in a file may contain sections 1
and 2 and the second segment may contain sections 3 and 4.

Mach-O文件中的各种表都是按编号引用节的。节编号从1(不是0)开始，并跨越段边界。因此，文件中的第一个段可以包含第1节和第2节，第二个段可以包含第3节和第4节。



When using the Stabs debugging format, the symbol table also holds debugging information. When
using DWARF, debugging information is stored in the image’s corresponding dSYM file, specified
by the uuid_command (page 20) structure

当使用Stabs调试格式时，符号表还包含调试信息。使用DWARF时，调试信息存储在image对应的dSYM文件中，该文件由uuid_command 结构指定



## Header Structure and Load Commands

A Mach-O file contains code and data for one architecture. The header structure of a Mach-O file
specifies the target architecture, which allows the kernel to ensure that, for example, code intended
for PowerPC-based Macintosh computers is not executed on Intel-based Macintosh computers.

一个Mach-O文件包含一个架构的代码和数据。Mach-O文件的头结构指定了目标体系结构，这允许内核确保，例如，针对基于powerpc的Macintosh计算机的代码不会在基于intel的Macintosh计算机上执行。



You can group multiple Mach-O files (one for each architecture you want to support) in one binary
using the format described in “Universal Binaries and 32-bit/64-bit PowerPC Binaries” (page 55).

您可以使用“Universal Binaries and 32-bit/64-bit PowerPC Binaries”中描述的格式将多个Mach-O文件(每个体系结构一个)分组到一个二进制文件中。



> **Note:** Binaries that contain object files for more than one architecture are not Mach-O files. They
> archive one or more Mach-O files.
>
> 注意:包含多个体系结构的目标文件的二进制文件不是Mach-O文件。他们存档一个或多个Mach-O文件。



Segments and sections are normally accessed by name. Segments, by convention, are named using
all uppercase letters preceded by two underscores (for example, _TEXT); sections should be named
using all lowercase letters preceded by two underscores (for example, __text). This naming convention is standard, although not required for the tools to operate correctly.

`Segments`和`sections`通常按名称访问。按照惯例，`Segments`的命名使用所有大写字母加上两个下划线(例如，__ TEXT);  `sections`的命名应该使用所有小写字母加上两个下划线(例如，__ text)。这只是一个规范的命名约定，不强制要求必须这样做



## Segments

A segment defines a range of bytes in a Mach-O file and the addresses and memory protection
attributes at which those bytes are mapped into virtual memory when the dynamic linker loads the
application. As such, segments are always virtual memory page aligned. A segment contains zero or
more sections.

`segment`在Mach-O文件中定义一个字节范围，以及地址和内存保护属性，当动态链接器加载应用程序时，这些字节被映射到虚拟内存中。因此，`segments`始终是虚拟内存页面对齐的。一个`segment`包含零个或多个`sections`。



Segments that require more memory at runtime than they do at build time can specify a larger
in-memory size than they actually have on disk. For example, the __PAGEZERO segment generated by
the linker for PowerPC executable files has a virtual memory size of one page but an on-disk size of 0.

Because __PAGEZERO contains no data, there is no need for it to occupy any space in the executable
file.

运行时比构建时需要更多内存的段可以指定比它们在磁盘上实际拥有的更大的内存大小。例如，链接器为PowerPC可执行文件生成的__ PAGEZERO段的虚拟内存大小为一页，而磁盘大小为0。由于__ PAGEZERO不包含数据，因此不需要占用可执行文件中的任何空间。



> **Note:** Sections that are to be filled with zeros must always be placed at the end of the segment.
> Otherwise, the standard tools will not be able to successfully manipulate the Mach-O file.
>
> 注意:要用零填充的部分必须始终放在段的末尾。否则，标准工具将无法成功地操作Mach-O文件。



For compactness, an intermediate object file contains only one segment. This segment has no name;
it contains all the sections destined ultimately for different segments in the final object file. The data
structure that defines a section (page 23) contains the name of the segment the section is intended
for, and the static linker places each section in the final object file accordingly.

为了紧凑性，中间对象文件只包含一个`segment`。这个`segment`没有名字;它包含为最终目标文件中的不同`segment`指定的所有`sections`。定义`section`的数据结构(第23页)包含节的目标段的名称，静态链接器将每个节相应地放在最终目标文件中。



For best performance, segments should be aligned on virtual memory page boundaries—4096 bytes
for PowerPC and x86 processors. To calculate the size of a segment, add up the size of each section,
then round up the sum to the next virtual memory page boundary (4096 bytes, or 4 kilobytes). Using
this algorithm, the minimum size of a segment is 4 kilobytes, and thereafter it is sized at 4 kilobyte
increments.

为了获得最佳性能，段应该对齐到虚拟内存页边界上—PowerPC和x86处理器的4096字节。要计算段的大小，请将每个段的大小相加，然后将总和四舍五入到下一个虚拟内存页边界(4096字节，或4 kb)。使用这种算法，一个段的最小大小是4 kb，然后以4 kb的增量进行调整。



The header and load commands are considered part of the first segment of the file for paging purposes.
In an executable file, this generally means that the headers and load commands live at the start of the __ TEXT segment because that is the first segment that contains data. The __PAGEZERO segment contains no data on disk, so it’s ignored for this purpose.

出于分页的目的，`header`和`load commands`被认为是文件第一`segment`的一部分。在一个可执行文件中，这通常意味着`headers`和`load commands`位于`__ TEXT` `segment`的开头，因为这是包含数据的第一个`segment`。由于`__PAGEZERO` `segment`没有包含任何数据，因此忽略了它。



These are the segments the standard Mac OS X development tools (contained in the Xcode Tools CD)
may include in a Mac OS X executable:

以下是标准Mac OS X开发工具(包含在Xcode tools CD中)可能包含在Mac OS X可执行文件中的部分:

- The static linker creates a __ PAGEZERO segment as the first segment of an executable file. This
  segment is located at virtual memory location 0 and has no protection rights assigned, the
  combination of which causes accesses to NULL, a common C programming error, to immediately
  crash. The __ PAGEZERO segment is the size of one full VM page for the current architecture (for
  Intel-based and PowerPC-based Macintosh computers, this is 4096 bytes or 0x1000 in hexadecimal).
  Because there is no data in the __PAGEZERO segment, it occupies no space in the file (the file size
  in the segment command is 0).

  静态链接器创建一个__PAGEZERO `segment` 作为可执行文件的第一个`segment`。这个`segment`位于虚拟内存位置0，没有分配任何保护权限，两者的组合会导致对NULL的访问立即崩溃，这是一个常见的C编程错误。对于当前的体系结构(对于基于intel和基于powerpc的Macintosh计算机，这是4096字节，或者十六进制是0x1000)， _ PAGEZERO`segment`的大小相当于一个完整的VM页面。因为在PAGEZERO段中没有数据，所以它在文件中不占空间(segment命令中的文件大小为0)。

- The __ TEXT segment contains executable code and other read-only data. To allow the kernel to map it directly from the executable into sharable memory, the static linker sets this segment’s virtual memory permissions to disallow writing. When the segment is mapped into memory, it can be shared among all processes interested in its contents. (This is primarily used with frameworks, bundles, and shared libraries, but it is possible to run multiple copies of the same executable in Mac OS X, and this applies in that case as well.) The read-only attribute also means that the pages that make up the __ TEXT segment never need to be written back to disk. When the kernel needs to free up physical memory, it can simply discard one or more __TEXT pages and re-read them from disk when they are next needed. 

  TEXT `segment`包含可执行代码和其他只读数据。为了允许内核将它直接从可执行文件映射到可共享内存，静态链接器将这个段的虚拟内存权限设置为不允许写入。当段被映射到内存中时，它可以在对其内容感兴趣的所有进程之间共享。(这主要用于框架、捆绑包和共享库，但也可以在Mac OS X中运行同一可执行文件的多个副本，在这种情况下也是适用的。)只读属性还意味着组成TEXT段的页面永远不需要写回磁盘。当内核需要释放物理内存时，它可以简单地丢弃一个或多个TEXT页面，然后在下次需要时从磁盘重新读取它们。

- The __ DATA segment contains writable data. The static linker sets the virtual memory permissions of this segment to allow both reading and writing. Because it is writable, the __ DATA segment of a framework or other shared library is logically copied for each process linking with the library. When memory pages such as those making up the __DATA segment are readable and writable, the kernel marks them copy-on-write; therefore when a process writes to one of these pages, that process receives its own private copy of the page. 

  DATA段包含可写数据。静态链接器设置此段的虚拟内存权限为允许读写。因为它是可写的，所以framework或其他共享库的DATA segment在逻辑上被复制到与库链接的每个进程。当组成DATA段的内存页可读可写时，内核将它们标记为copy-on-write; 因此，当一个进程写到其中一个页面时，该进程将收到该页面的私有副本。

- The __OBJC segment contains data used by the Objective-C language runtime support library. 

  __OBJC段包含OC运行时支持库使用的数据。

- The __IMPORT segment contains symbol stubs and non-lazy pointers to symbols not defined in 

  the executable. This segment is generated only for executables targeted for the IA-32 architecture. 

  __IMPORT段含符号存根和指向可执行文件中未定义的符号的非懒加载指针。此段只在IA-32架构下的可执行文件中生成

- The __LINKEDIT segment contains raw data used by the dynamic linker, such as symbol, string, and relocation table entries. 

  __LINKEDIT段包含动态链接器使用的原始数据，例如符号、字符串和重定位表条目。



## Sections

The __ TEXT and __ DATA segments may contain a number of standard sections, listed in Table 1, Table 2 (page 11), and Table 3 (page 11). The __OBJC segment contains a number of sections that are private to the Objective-C compiler. Note that the static linker and file analysis tools use the section type and attributes (instead of the section name) to determine how they should treat the section. The section name, type and attributes are explained further in the description of the section (page 23) data type. 

TEXT和DATA `segments`可以包含许多标准`sections`，如表1、表2和表3中列出。__OBJC段包含许多对Objective-C编译器私有的部分。注意，静态链接器和文件分析工具使用`section`类型和属性(而不是section name)来确定它们应该如何处理这个`section`。`section`名称、类型和属性将在section数据类型描述中进一步解释。



**Table 1** The sections of a __TEXT segment 

| Segment and section name  | Contents                                                     |
| ------------------------- | ------------------------------------------------------------ |
| __ TEXT, __text           | Executable machine code. The compiler generally places only executable<br/>code in this section, no tables or data of any sort.<br>可执行的机器代码。编译器通常只将可执行代码放在这一节中，没有任何类型的表或数据。 |
| __ TEXT, __cstring        | Constant C strings. A C string is a sequence of non-null bytes that ends<br/>with a null byte ('\0'). The static linker coalesces constant C string values,<br/>removing duplicates, when building the final product.<br>常数C字符串。C字符串是一个以空字节('\0')结尾的非空字节序列。静态链接器在构建最终产品时合并常量C字符串值，删除重复项。 |
| __ TEXT, __picsymbol_stub | Position-independent indirect symbol stubs. See “Dynamic Code<br/>Generation” in *Mach-O Programming Topics* for more information.<br>位置无关的间接符号存根。有关更多信息，请参阅Mach-O编程主题中的“动态代码生成”。 |
| __ TEXT, __symbol_stub    | Indirect symbol stubs. See “Dynamic Code Generation” in *Mach-O<br/>Programming Topics* for more information.<br>间接的象征存根。有关更多信息，请参阅Mach-O编程主题中的“动态代码生成”。 |
| __ TEXT, __const          | Initialized constant variables. The compiler places all nonrelocatable data<br/>declared const in this section. (The compiler typically places uninitialized constant variables in a zero-filled section.)<br>初始化常数变量。编译器将在本节中放置所有声明的const的不可重定位数据。(编译器通常将未初始化的常量变量放在一个零填充的部分。) |
| __ TEXT, __literal4       | 4-byte literal values. The compiler places single-precision floating point<br/>constants in this section. The static linker coalesces these values, removing<br/>duplicates, when building the final product. With some architectures,<br/>it’s more efficient for the compiler to use immediate load instructions<br/>rather than adding to this section.<br>4字节的文本值。编译器在本节中放置单精度浮点常量。静态链接器在构建最终产品时合并这些值，删除重复项。对于某些架构，编译器使用立即加载指令比添加到本节更有效。 |
| __ TEXT, __literal8       | 8-byte literal values. The compiler places double-precision floating point<br/>constants in this section. The static linker coalesces these values, removing<br/>duplicates, when building the final product. With some architectures,<br/>it’s more efficient for the compiler to use immediate load instructions<br/>rather than adding to this section.<br>8字节文字值。编译器在本节中放置双精度浮点常量。静态链接器在构建最终产品时合并这些值，删除重复项。对于某些架构，编译器使用立即加载指令比添加到本节更有效。 |



**Table 2**	The sections of a __DATA segment 

| Segment and section name | Contents                                                     |
| ------------------------ | ------------------------------------------------------------ |
| __ DATA, __data          | Initialized mutable variables, such as writable C strings and data arrays.<br>初始化可变变量，如可写C字符串和数据数组。 |
| __ DATA, __la_symbol_ptr | Lazy symbol pointers, which are indirect references to functions<br/>imported from a different file. See “Dynamic Code Generation” in<br/>*Mach-O Programming Topics* for more information.<br>懒加载符号指针，是对从不同文件导入的函数的间接引用。有关更多信息，请参阅Mach-O编程主题中的“动态代码生成”。 |
| __ DATA, __nl_symbol_ptr | Non-lazy symbol pointers, which are indirect references to data items imported from a different file. See “Dynamic Code Generation” in Mach-O Programming Topics for more information.<br>非懒加载符号指针，是对从其他文件导入的函数的间接引用。有关详细信息，请参阅[Mach-O Programming Topics](https://developer.apple.com/library/archive/documentation/DeveloperTools/Conceptual/MachOTopics/0-Introduction/introduction.html#//apple_ref/doc/uid/TP40001827-SW1)中的“Dynamic Code Generatio”。 |
| __ DATA, __dyld          | Placeholder section used by the dynamic linker.<br>动态链接器使用的占位section |
| __ DATA, __const         | Initialized relocatable constant variables.<br>初始化可重定位常量变量。 |
| __ DATA, __mod_init_func | Module initialization functions. The C++ compiler places static<br/>constructors here.<br>模块初始化方法，c++的静态构造函数在这里放 |
| __ DATA, __mod_term_func | Module termination functions.<br>退出方法，类似析构函数      |
| __ DATA, __bss           | Data for uninitialized static variables (for example, static int i;).<br>未初始化静态变量的数据（例如，static int i；） |
| __ DATA, __common        | Uninitialized imported symbol definitions (for example, int i;)<br/>located in the global scope (outside of a function declaration).<br>位于全局范围（函数声明之外）中的未初始化全局符号的定义（例如，int i；) |



**Table 3**  The sections of a __IMPORT segment 

| Segment and section name | Contents                                                     |
| ------------------------ | ------------------------------------------------------------ |
| __ IMPORT, __jump_table  | Stubs for calls to functions in a dynamic library.<br>动态库中函数调用的存根。 |
| __ IMPORT, __pointers    | Non-lazy symbol pointers, which are direct references to functions<br/>imported from a different file.<br>非懒加载符号指针，它们直接引用从其他文件导入的函数 |

>  Note: Compilers or any tools that create Mach-O files are free to define additional section names. These additional names do not appear in Table 
>
> 注意：编译器或任何创建mach-o文件的工具都可以自由定义附加的节名。这些附加名称不出现在表中



# Data Types

This reference describes the data types that compose a Mach-O file. Values for integer types in all Mach-O data structures are written using the host CPU’s byte ordering scheme, except for `fat_header` (page 56) and `fat_arch` (page 56), which are written in big-endian byte order.

此参考描述组成mach-o文件的数据类型。所有mach-o数据结构中的整数类型的值都是使用主机cpu的字节顺序方案编写的，除了`fat_header`（第56页）和`fat_arch`（第56页）之外，它们都是以大端字节顺序编写的。



## Header Data Structure

### mach_header

Specifies the general attributes of a file. Appears at the beginning of object files targeted to 32-bit architectures. Declared in /usr/include/mach-o/loader.h. See also mach_header_64 (page 14).

```c
/*
 * The 32-bit mach header appears at the very beginning of the object file for
 * 32-bit architectures.
 */
struct mach_header {
	uint32_t	magic;		/* mach magic number identifier */
	cpu_type_t	cputype;	/* cpu specifier */
	cpu_subtype_t	cpusubtype;	/* machine specifier */
	uint32_t	filetype;	/* type of file */
	uint32_t	ncmds;		/* number of load commands */
	uint32_t	sizeofcmds;	/* the size of all the load commands */
	uint32_t	flags;		/* flags */
};

/* Constant for the magic field of the mach_header (32-bit architectures) */
#define	MH_MAGIC	0xfeedface	/* the mach magic number */
#define MH_CIGAM	0xcefaedfe	/* NXSwapInt(MH_MAGIC) */
```

- **magic**

  An integer containing a value identifying this file as a 32-bit Mach-O file. Use the constant MH_MAGIC if the file is intended for use on a CPU with the same endianness as the computer on which the compiler is running. The constant MH_CIGAM can be used when the byte ordering scheme of the target machine is the reverse of the host CPU.

  一个整数将文件标识为32位Mach-O文件。如果文件要在与运行编译器的计算机具有相同端点的CPU上使用，请使用常量MH_MAGIC。当目标机器的字节排序方案与主机cpu相反时，可以使用常数MH_CIGAM。(大小端值不相同)

- **cputype**

  An integer indicating the architecture you intend to use the file on. Appropriate values include:

  一个整数，指示要在其上使用文件的体系结构。适当的值包括：

  ```c
  #include <mach/machine.h>
  
  /*
   *	Machine types known by all.
   */
   
  #define CPU_TYPE_ANY		((cpu_type_t) -1)
  
  #define CPU_TYPE_VAX		((cpu_type_t) 1)
  /* skip				((cpu_type_t) 2)	*/
  /* skip				((cpu_type_t) 3)	*/
  /* skip				((cpu_type_t) 4)	*/
  /* skip				((cpu_type_t) 5)	*/
  #define	CPU_TYPE_MC680x0	((cpu_type_t) 6)
  #define CPU_TYPE_X86		((cpu_type_t) 7)
  #define CPU_TYPE_I386		CPU_TYPE_X86		/* compatibility */
  #define	CPU_TYPE_X86_64		(CPU_TYPE_X86 | CPU_ARCH_ABI64)
  
  /* skip CPU_TYPE_MIPS		((cpu_type_t) 8)	*/
  /* skip 			((cpu_type_t) 9)	*/
  #define CPU_TYPE_MC98000	((cpu_type_t) 10)
  #define CPU_TYPE_HPPA           ((cpu_type_t) 11)
  #define CPU_TYPE_ARM		((cpu_type_t) 12)
  #define CPU_TYPE_ARM64		(CPU_TYPE_ARM | CPU_ARCH_ABI64)
  #define CPU_TYPE_MC88000	((cpu_type_t) 13)
  #define CPU_TYPE_SPARC		((cpu_type_t) 14)
  #define CPU_TYPE_I860		((cpu_type_t) 15)
  /* skip	CPU_TYPE_ALPHA		((cpu_type_t) 16)	*/
  /* skip				((cpu_type_t) 17)	*/
  #define CPU_TYPE_POWERPC		((cpu_type_t) 18)
  #define CPU_TYPE_POWERPC64		(CPU_TYPE_POWERPC | CPU_ARCH_ABI64)
  ```

- **cpusubtype**

  An integer specifying the exact model of the CPU. To run on all PowerPC or x86 processors supported by the Mac OS X kernel, this should be set to CPU_SUBTYPE_POWERPC_ALL or CPU_SUBTYPE_I386_ALL.

  一个整数，指定CPU的确切型号。要在Mac OS X内核支持的所有PowerPC或x86处理器上运行，应将其设置为CPU_Subtype_PowerPC_All或CPU_Subtype_i386_All。

  ```c
  #define CPU_SUBTYPE_ARM_ALL             ((cpu_subtype_t) 0)
  #define CPU_SUBTYPE_X86_ALL		((cpu_subtype_t)3)
  ```

- **filetype**

  An integer indicating the usage and alignment of the file. Valid values for this field include:

  ```c
  /*
   * A core file is in MH_CORE format and can be any in an arbritray legal
   * Mach-O file.
   *
   * Constants for the filetype field of the mach_header
   */
  #define	MH_OBJECT	0x1		/* relocatable object file */
  #define	MH_EXECUTE	0x2		/* demand paged executable file */
  #define	MH_FVMLIB	0x3		/* fixed VM shared library file */
  #define	MH_CORE		0x4		/* core file */
  #define	MH_PRELOAD	0x5		/* preloaded executable file */
  #define	MH_DYLIB	0x6		/* dynamically bound shared library */
  #define	MH_DYLINKER	0x7		/* dynamic link editor */
  #define	MH_BUNDLE	0x8		/* dynamically bound bundle file */
  #define	MH_DYLIB_STUB	0x9		/* shared library stub for static */
  					/*  linking only, no section contents */
  #define	MH_DSYM		0xa		/* companion file with only debug */
  					/*  sections */
  #define	MH_KEXT_BUNDLE	0xb		/* x86_64 kexts */
  ```

  - The `MH_OBJECT` file type is the format used for intermediate object files. It is a very compact format containing all its sections in one segment. The compiler and assembler usually create one `MH_OBJECT` file for each source code file. By convention, the file name extension for this format is .o.

    `MH_OBJECT`文件类型是用于中间对象文件的格式。它是一种非常紧凑的格式，将其所有部分都包含在一个段中。编译器和汇编程序通常为每个源代码文件创建一个`MH_OBJECT`文件。按照惯例，此格式的文件扩展名为.o。

  - The `MH_EXECUTE` file type is the format used by standard executable programs. 

    `MH_EXECUTE` file type是标准可执行程序使用的格式。

  - The `MH_BUNDLE` file type is the type typically used by code that you load at runtime (typically called bundles or plug-ins). By convention, the file name extension for this format is .bundle.

    `MH_BUNDLE`文件类型是运行时加载的代码通常使用的类型（通常称为bundle或插件）。按照惯例，此格式的文件扩展名为.bundle。 

  - The `MH_DYLIB` file type is for dynamic shared libraries. It contains some additional tables to support multiple modules. By convention, the file name extension for this format is .dylib, except for the main shared library of a framework, which does not usually have a file name extension. 

    `MH_DYLIB`文件类型用于动态共享库。它包含一些额外的表来支持多个模块。按照惯例，此格式的文件扩展名为.dylib，但框架的主共享库除外，它通常没有文件扩展名。

  - The `MH_PRELOAD` file type is an executable format used for special-purpose programs that are not loaded by the Mac OS X kernel, such as programs burned into programmable ROM chips. Do not confuse this file type with the `MH_PREBOUND` flag, which is a flag that the static linker sets in the header structure to mark a prebound image. 

    `MH_PRELOAD`文件类型是一种可执行格式，用于mac os x内核未加载的特殊用途程序，例如烧录到可编程ROM芯片中的程序。不要将此文件类型与`MH_PREBOUND`标志混淆，`MH_PREBOUND`标志是静态链接器在头结构中设置的用于标记预绑定图像的标志。

  - The `MH_CORE` file type is used to store core files, which are traditionally created when a program crashes. Core files store the entire address space of a process at the time it crashed. You can later run gdb on the core file to figure out why the crash occurred. 

    `MH_CORE`文件类型用于存储核心文件，这些文件通常在程序崩溃时创建。核心文件存储进程崩溃时的整个地址空间。您可以稍后在核心文件上运行gdb来找出崩溃的原因。

  - The `MH_DYLINKER` file type is the type of a dynamic linker shared library. This is the type of the dyld file. 

    `MH_DYLINKER`文件类型是动态链接器共享库的类型。这是dyld文件的类型。

  - The `MH_DSYM` file type designates files that store symbol information for a corresponding binary file. 

    `MH_DSYM`文件类型指定存储相应二进制文件的符号信息的文件。

- **ncmds**

  An integer indicating the number of load commands following the header structure.

  指定架构load commands的个数

- **sizeofcmds** 

  An integer indicating the number of bytes occupied by the load commands following the header structure. 所有load commands的size

- **flags**

  An integer containing a set of bit flags that indicate the state of certain optional features of the
  Mach-O file format. These are the masks you can use to manipulate this field:

  一个整数，包含一组位标志，指示mach-o文件格式某些可选功能的状态。以下是可用于操作此字段的掩码：

  ```c
  /* Constants for the flags field of the mach_header */
  #define	MH_NOUNDEFS	0x1		/* the object file has no undefined
  					   references */
  #define	MH_INCRLINK	0x2		/* the object file is the output of an
  					   incremental link against a base file
  					   and can't be link edited again */
  #define MH_DYLDLINK	0x4		/* the object file is input for the
  					   dynamic linker and can't be staticly
  					   link edited again */
  #define MH_BINDATLOAD	0x8		/* the object file's undefined
  					   references are bound by the dynamic
  					   linker when loaded. */
  #define MH_PREBOUND	0x10		/* the file has its dynamic undefined
  					   references prebound. */
  #define MH_SPLIT_SEGS	0x20		/* the file has its read-only and
  					   read-write segments split */
  #define MH_LAZY_INIT	0x40		/* the shared library init routine is
  					   to be run lazily via catching memory
  					   faults to its writeable segments
  					   (obsolete) */
  #define MH_TWOLEVEL	0x80		/* the image is using two-level name
  					   space bindings */
  #define MH_FORCE_FLAT	0x100		/* the executable is forcing all images
  					   to use flat name space bindings */
  #define MH_NOMULTIDEFS	0x200		/* this umbrella guarantees no multiple
  					   defintions of symbols in its
  					   sub-images so the two-level namespace
  					   hints can always be used. */
  #define MH_NOFIXPREBINDING 0x400	/* do not have dyld notify the
  					   prebinding agent about this
  					   executable */
  #define MH_PREBINDABLE  0x800           /* the binary is not prebound but can
  					   have its prebinding redone. only used
                                             when MH_PREBOUND is not set. */
  #define MH_ALLMODSBOUND 0x1000		/* indicates that this binary binds to
                                             all two-level namespace modules of
  					   its dependent libraries. only used
  					   when MH_PREBINDABLE and MH_TWOLEVEL
  					   are both set. */ 
  #define MH_SUBSECTIONS_VIA_SYMBOLS 0x2000/* safe to divide up the sections into
  					    sub-sections via symbols for dead
  					    code stripping */
  #define MH_CANONICAL    0x4000		/* the binary has been canonicalized
  					   via the unprebind operation */
  #define MH_WEAK_DEFINES	0x8000		/* the final linked image contains
  					   external weak symbols */
  #define MH_BINDS_TO_WEAK 0x10000	/* the final linked image uses
  					   weak symbols */
  
  #define MH_ALLOW_STACK_EXECUTION 0x20000/* When this bit is set, all stacks 
  					   in the task will be given stack
  					   execution privilege.  Only used in
  					   MH_EXECUTE filetypes. */
  #define MH_ROOT_SAFE 0x40000           /* When this bit is set, the binary 
  					  declares it is safe for use in
  					  processes with uid zero */
                                           
  #define MH_SETUID_SAFE 0x80000         /* When this bit is set, the binary 
  					  declares it is safe for use in
  					  processes when issetugid() is true */
  
  #define MH_NO_REEXPORTED_DYLIBS 0x100000 /* When this bit is set on a dylib, 
  					  the static linker does not need to
  					  examine dependent dylibs to see
  					  if any are re-exported */
  #define	MH_PIE 0x200000			/* When this bit is set, the OS will
  					   load the main executable at a
  					   random address.  Only used in
  					   MH_EXECUTE filetypes. */
  #define	MH_DEAD_STRIPPABLE_DYLIB 0x400000 /* Only for use on dylibs.  When
  					     linking against a dylib that
  					     has this bit set, the static linker
  					     will automatically not create a
  					     LC_LOAD_DYLIB load command to the
  					     dylib if no symbols are being
  					     referenced from the dylib. */
  #define MH_HAS_TLV_DESCRIPTORS 0x800000 /* Contains a section of type 
  					    S_THREAD_LOCAL_VARIABLES */
  
  #define MH_NO_HEAP_EXECUTION 0x1000000	/* When this bit is set, the OS will
  					   run the main executable with
  					   a non-executable heap even on
  					   platforms (e.g. i386) that don't
  					   require it. Only used in MH_EXECUTE
  					   filetypes. */
  
  #define MH_APP_EXTENSION_SAFE 0x02000000 /* The code was linked for use in an
  					    application extension. */
  
  #define	MH_NLIST_OUTOFSYNC_WITH_DYLDINFO 0x04000000 /* The external symbols
  					   listed in the nlist symbol table do
  					   not include all the symbols listed in
  					   the dyld info. */
  
  #define	MH_SIM_SUPPORT 0x08000000	/* Allow LC_MIN_VERSION_MACOS and
  					   LC_BUILD_VERSION load commands with
  					   the platforms macOS, macCatalyst,
  					   iOSSimulator, tvOSSimulator and
  					   watchOSSimulator. */
  					   
  #define MH_DYLIB_IN_CACHE 0x80000000	/* Only for use on dylibs. When this bit
  					   is set, the dylib is part of the dyld
  					   shared cache, rather than loose in
  					   the filesystem. */
  ```

  - `MH_NOUNDEFS`—The object file contained no undefined references when it was built. 
  
    ​							对象文件在生成时不包含未定义的引用
  
  - `MH_INCRLINK` The object file is the output of an incremental link against a base file and
    cannot be linked again
  
    对象文件是对基本文件的增量链接的输出，并且无法再次链接
  
  - `MH_DYLDLINK` 
  
    - The file is input for the dynamic linker and cannot be statically linked again.
  
      该文件是动态链接器的输入，不能再次静态链接。  
  
  - `MH_TWOLEVEL`  
  
    - The image is using two-level namespace bindings 
  
      Image正在使用两级命名空间绑定
  
  - `MH_BINDATLOAD `
  
    - The dynamic linker should bind the undefined references when the file 
  
      is loaded. 
  
      动态链接器应该在文件加载时绑定未定义的引用
  
  - `MH_PREBOUND ` 
  
    - The file’s undefined references are prebound. 
  
      文件的未定义引用是预先绑定的。
  
  - `MH_PREBINDABLE ` 
  
    - This file is not prebound but can have its prebinding redone. Used only when MH_PREBEOUND is not set 
    - 此文件不是预绑定的，但可以重新进行预绑定。仅在未设置MH_PREBEOUND时使用
  
  - `MH_NOFIXPREBINDING`
  
    - The dynamic linker doesn’t notify the prebinding agent about this executable. 
  
      动态链接器不会将此可执行文件通知预绑定代理。
  
  - `MH_ALLMODSBOUND`
  
    - Indicates that this binary binds to all two-level namespace modules of its dependent libraries. Used only when MH_PREBINDABLE and MH_TWOLEVEL are set. 
    - 指示此二进制文件绑定到其依赖库的所有两级命名空间模块。仅当设置了MH_PREBINDABLE和MH_TWOLEVEL时使用。
  
  - `MH_CANONICAL`
  
    - This file has been canonicalized by unprebinding—clearing prebinding information from the file. See the redo_prebinding man page for details 
    - 此文件已通过取消绑定从文件中清除预绑定信息来规范化。有关详细信息，请参阅重做预绑定手册页
  
  - `MH_SPLIT_SEGS`
  
    - The file has its read-only and read-write segments split. 
    - 文件的只读段和读写段被拆分。
  
  - `MH_FORCE_FLAT`
  
    - The executable is forcing all images to use flat namespace bindings. 
    - 可执行文件正在强制所有映像使用平面命名空间绑定。
  
  - `MH_SUBSECTIONS_VIA_SYMBOLS`
  
    - The sections of the object file can be divided into individual blocks. These blocks are dead-stripped if they are not used by other code. See “Linking” in Xcode User Guide for details. 
    - 对象文件的各个部分可以划分为单独的块。如果其他代码不使用这些块，则它们将被完全剥离。有关详细信息，请参阅Xcode用户指南中的“链接”。
  
  - `MH_NOMULTIDEFS`
  
    - This umbrella guarantees there are no multiple definitions of symbols in its subimages. As a result, the two-level namespace hints can always be used. 
    - 这把伞保证了它的子图像中没有符号的多重定义。因此，始终可以使用两级命名空间提示。
  
  - `MH_PIE` 
  
    - When this bit is set, the OS will load the main executable at a random address.  Only used in `MH_EXECUTE` filetypes
    - 随机地址分布，ASLR 用于MH_EXECUTE filetype

- **Special Considerations**

  For all file types, except `MH_OBJECT`, segments must be aligned on page boundaries for the given CPU
  architecture: 4096 bytes for PowerPC and x86 processors. This allows the kernel to page virtual memory
  directly from the segment into the address space of the process. The header and load commands must
  be aligned as part of the data of the first segment stored on disk (which would be the __TEXT segment,
  in the file types described in filetype).

  对于除MH_OBJECT以外的所有文件类型，对于给定的cpu体系结构，段必须在页面边界上对齐：对于powerpc和x86处理器，为4096字节。这允许内核将虚拟内存直接从段分页到进程的地址空间。`header`和`load commands`存储在磁盘上的第一个段的数据的一部分对齐（在filetype中描述的文件类型中，第一个段是__TEXT段)

  

## Load Command Data Structures

The load command structures are located directly after the header of the object file, and they specify
both the logical structure of the file and the layout of the file in virtual memory. Each load command
begins with fields that specify the command type and the size of the command data.

加载命令结构直接位于对象文件头的后面，它们指定文件的逻辑结构和虚拟内存中文件的布局。每个加载命令都以指定命令类型和命令数据大小的字段开头。

### load_command

Contains fields that are common to all load commands.

包含所有加载命令通用的字段。

```c
struct load_command {
	uint32_t cmd;		/* type of load command */
	uint32_t cmdsize;	/* total size of command in bytes */
};
```

**Fields**

- `cmd` 

  An integer indicating the type of load command. Table 4 lists the valid load command types.

- `cmdsize`

  An integer specifying the total size in bytes of the load command data structure. Each load
  command structure contains a different set of data, depending on the load command type, so
  each might have a different size. In 32-bit architectures, the size must always be a multiple of
  4; in 64-bit architectures, the size must always be a multiple of 8. If the load command data
  does not divide evenly by 4 or 8 (depending on whether the target architecture is 32-bit or
  64-bit, respectively), add bytes containing zeros to the end until it does.

  一个整数，指定加载命令数据结构的总大小（字节）。每个加载命令结构都包含不同的数据集，具体取决于加载命令类型，因此每个数据集的大小可能不同。在32位体系结构中，大小必须始终是4的倍数；在64位体系结构中，大小必须始终是8的倍数。如果加载命令数据没有被4或8等分（分别取决于目标体系结构是32位还是64位），请在末尾添加包含零的字节，直到它等分为止。

**Discussion**

Table 4 lists the valid load command types, with links to the full data structures for each type.

表4列出了有效的加载命令类型，每种类型都有指向完整数据结构的链接。



**Table 4** Mach-O load commands

| Commands                | Data Structures        | Purpose                                                      |
| ----------------------- | ---------------------- | ------------------------------------------------------------ |
| LC_UUID                 | uuid_command           | Specifies the 128-bit UUID for an image <br>or its corresponding dSYM file.<br>为image或其相应的dSYM文件指定128位uuid |
| LC_SEGMENT              | segment_command        | Defines a segment of this file to be<br/>mapped into the address space of the<br/>process that loads this file. It also includes<br/>all the sections contained by the segment.<br>定义要映射到加载此文件的进程的地址空间中的此文件段。它还包括该段所包含的所有部分。 |
| LC_SEGMENT_64           | segment_command_64     | Defines a 64-bit segment of this file to be<br/>mapped into the address space of the<br/>process that loads this file. It also includes<br/>all the sections contained by the segment. |
| LC_SYMTAB               | symtab_command         | Specifies the symbol table for this file.<br/>This information is used by both static<br/>and dynamic linkers when linking the<br/>file, and also by debuggers to map<br/>symbols to the original source code files<br/>from which the symbols were generated.<br>指定此文件的符号表。静态和动态链接器在链接文件时都会使用此信息，调试器也会使用此信息将符号映射到生成符号的原始源代码文件。 |
| LC_DYSYMTAB             | dysymtab_command       | Specifies additional symbol table<br/>information used by the dynamic linker.<br>指定动态链接器使用的其他符号表信息。 |
| LC_THREAD LC_UNIXTHREAD | thread_command         | For an executable file, the LC_UNIXTHREAD<br/>command defines the initial thread state<br/>of the main thread of the process.<br/>LC_THREAD is similar to LC_UNIXTHREAD<br/>but does not cause the kernel to allocate<br/>a stack.<br>对于可执行文件，LC_UNIXTHREAD命令定义进程主线程的初始线程状态。LC_THREAD类似于LC_UNIXTHREAD，但不会导致内核分配堆栈。 |
| LC_LOAD_DYLIB           | dylib_command          | Defines the name of a dynamic shared<br/>library that this file links against.<br>定义此文件链接所针对的动态共享库的名称。 |
| LC_ID_DYLIB             | dylib_command          | Specifies the install name of a dynamic<br/>shared library.<br>指定动态共享库的安装名称。 |
| LC_PREBOUND_DYLIB       | prebound_dylib_command | For a shared library that this executable<br/>is linked prebound against, specifies the<br/>modules in the shared library that are<br/>used.<br>对于此可执行文件链接到的共享库，请指定共享库中使用的模块。 |
| LC_LOAD_DYLINKER        | dylinker_command       | Specifies the dynamic linker that the<br/>kernel executes to load this file.<br>指定内核执行以加载此文件的动态链接器。 |
| LC_ID_DYLINKER          | dylinker_command       | Identifies this file as a dynamic linker.<br>将此文件标识为动态链接器。 |
| LC_ROUTINES             | routines_command       | Contains the address of the shared library<br/>initialization routine (specified by the<br/>linker’s -init option).<br>包含共享库初始化例程的地址（由链接器的-init选项指定）。 |
| LC_ROUTINES_64          | routines_command_64    | Contains the address of the shared library<br/>64-bit initialization routine (specified by<br/>the linker’s -init option). |
| LC_TWOLEVEL_HINTS       | twolevel_hints_command | Contains the two-level namespace lookup<br/>hint table.<br>包含两级命名空间查找提示表。 |
| LC_SUB_FRAMEWORK        | sub_framework_command  | Identifies this file as the implementation<br/>of a subframework of an umbrella<br/>framework. The name of the umbrella<br/>framework is stored in the string<br/>parameter.<br>将此文件标识为伞形框架的子框架的实现。伞形框架的名称存储在字符串参数中。 |
| LC_SUB_UMBRELLA         | sub_umbrella_command   | Specifies a file that is a subumbrella of<br/>this umbrella framework.<br>指定作为此伞形框架的子伞形结构的文件。 |
| LC_SUB_LIBRARY          | sub_library_command    | Identifies this file as the implementation<br/>of a sublibrary of an umbrella framework.<br/>The name of the umbrella framework is<br/>stored in the string parameter. Note that<br/>Apple has not defined a supported<br/>location for sublibraries.<br>将此文件标识为伞形框架的子库的实现。伞形框架的名称存储在字符串参数中。请注意，Apple尚未为子库定义支持的位置。 |
| LC_SUB_CLIENT           | sub_client_command     | A subframework can explicitly allow<br/>another framework or bundle to link<br/>against it by including an LC_SUB_CLIENT<br/>load command containing the name of<br/>the framework or a client name for a<br/>bundle.<br>子框架可以通过包含lc_sub_client load命令显式地允许另一个框架或捆绑包与之链接，该命令包含框架的名称或捆绑包的客户端名称。 |



- **uuid_command**

  Specifies the 128-bit universally unique identifier (UUID) for an image or for its corresponding dSYM file.

  为image或其相应的dSYM文件指定128位uuid

  ```c
  /*
   * The uuid load command contains a single 128-bit unique random number that
   * identifies an object produced by the static link editor.
   */
  struct uuid_command {
      uint32_t	cmd;		/* LC_UUID */
      uint32_t	cmdsize;	/* sizeof(struct uuid_command) */
      uint8_t	uuid[16];	/* the 128-bit uuid */
  };
  ```

  **Fields** 

  - `cmd`
    		Set to LC_UUID for this structure. 

  - `cmdsize`
     		Set to sizeof(uuid_command). 

  - `uuid `

    ​		128-bit unique identifier

  **Declared In** 

  /usr/include/mach-o/loader.h

  

- **segment_command**

  Specifies the range of bytes in a 32-bit Mach-O file that make up a segment. Those bytes are mapped by the loader into the address space of a program. Declared in /usr/include/mach-o/loader.h.

  指定组成段的32位mach-o文件中的字节范围。这些字节由加载程序映射到程序的地址空间中

  ```c
  struct segment_command { /* for 32-bit architectures */
  	uint32_t	cmd;		/* LC_SEGMENT */
  	uint32_t	cmdsize;	/* includes sizeof section structs */
  	char		segname[16];	/* segment name */
  	uint32_t	vmaddr;		/* memory address of this segment */
  	uint32_t	vmsize;		/* memory size of this segment */
  	uint32_t	fileoff;	/* file offset of this segment */
  	uint32_t	filesize;	/* amount to map from the file */
  	vm_prot_t	maxprot;	/* maximum VM protection */
  	vm_prot_t	initprot;	/* initial VM protection */
  	uint32_t	nsects;		/* number of sections in segment */
  	uint32_t	flags;		/* flags */
  };
  
  /*
   * The 64-bit segment load command indicates that a part of this file is to be
   * mapped into a 64-bit task's address space.  If the 64-bit segment has
   * sections then section_64 structures directly follow the 64-bit segment
   * command and their size is reflected in cmdsize.
   */
  struct segment_command_64 { /* for 64-bit architectures */
  	uint32_t	cmd;		/* LC_SEGMENT_64 */
  	uint32_t	cmdsize;	/* includes sizeof section_64 structs */
  	char		segname[16];	/* segment name */
  	uint64_t	vmaddr;		/* memory address of this segment */
  	uint64_t	vmsize;		/* memory size of this segment */
  	uint64_t	fileoff;	/* file offset of this segment */
  	uint64_t	filesize;	/* amount to map from the file */
  	vm_prot_t	maxprot;	/* maximum VM protection */
  	vm_prot_t	initprot;	/* initial VM protection */
  	uint32_t	nsects;		/* number of sections in segment */
  	uint32_t	flags;		/* flags */
  };
  ```

  **Fields**

  - `cmd`

    Common to all load command structures. Set to LC_SEGMENT for this structure.

  - `cmdsize`

    Common to all load command structures. For this structure, set this field to
    sizeof(segment_command) plus the size of all the section data structures that follow
    (sizeof(segment_command + (sizeof(section) * segment->nsect))).

  - `segname`

    A C string specifying the name of the segment. The value of this field can be any sequence of
    ASCII characters, although segment names defined by Apple begin with two underscores and
    consist of capital letters (as in ___TEXT and ___DATA). This field is fixed at 16 bytes in length.

    指定段名称的C字符串。此字段的值可以是任意一个ASCII字符序列，尽管Apple定义的段名称以两个下划线开头，并由大写字母组成（如在“_______TEXT”和"_____DATA"中）。此字段的长度固定为16字节。

  - `vmaddr`

    Indicates the starting virtual memory address of this segment.

    指示此段的起始虚拟内存地址。

  - `vmsize`

    Indicates the number of bytes of virtual memory occupied by this segment. See also the
    description of filesize, below.

    指示此段占用的虚拟内存字节数。另请参见下面的文件大小说明。

  - `fileoff`

    Indicates the offset in this file of the data to be mapped at vmaddr.

    指示此文件中要在vmaddr映射的数据的偏移量。

  - `filesize`

    Indicates the number of bytes occupied by this segment on disk. For segments that require
    more memory at runtime than they do at build time, vmsize can be larger than filesize. For
    example, the __ PAGEZERO segment generated by the linker for MH_EXECUTABLE files has a
    vmsize of 0x1000 but a filesize of 0. Because _ PAGEZERO contains no data, there is no need
    for it to occupy any space until runtime. Also, the static linker often allocates uninitialized data
    at the end of the __DATA segment; in this case, the vmsize is larger than the filesize. The
    loader guarantees that any memory of this sort is initialized with zeros.

    指示磁盘上此段占用的字节数。对于运行时比构建时需要更多内存的段，vmsize可以大于filesize。例如，链接器为MH_EXECUTABLE文件生成的u pagezero段的vmsize为0x1000，而filesize为0。因为pagezero不包含任何数据，所以在运行之前不需要占用任何空间。此外，静态链接器通常在数据段末尾分配未初始化的数据；在这种情况下，vmsize大于filesize。加载程序保证这类内存都是用零初始化的。

  - `maxprot`

    Specifies the maximum permitted virtual memory protections of this segment.

    指定此段允许的最大虚拟内存保护。

  - `initprot`

    Specifies the initial virtual memory protections of this segment.

    指定此段的初始虚拟内存保护。

  - `nsects`

    Indicates the number of section data structures following this load command.

    指示此load commands命令下的section数量

  - `flags`

    Defines a set of flags that affect the loading of this segment:

    - SG_HIGHVM—The file contents for this segment are for the high part of the virtual memory space; the low part is zero filled (for stacks in core files). 
    - SG_NORELOC—This segment has nothing that was relocated in it and nothing relocated to it. It may be safely replaced without relocation. 
    - 定义一组影响此段加载的标志：
      SG_HIGHVM—此段的文件内容用于虚拟内存空间的高部分；低部分为零填充（用于核心文件中的堆栈）。
      SG_NORELOC-该段没有重新安置的任何东西，也没有重新安置的任何东西。可安全更换，无需重新安置。

- **section**

  Defines the elements used by a 32-bit section. Directly following a segment_command data structure
  is an array of section data structures, with the exact count determined by the nsects field of the
  segment_command (page 20) structure. Declared in /usr/include/mach-o/loader.h. 

  定义32位节使用的元素。段命令数据结构的正后方是一个段数据结构数组，其精确计数由段命令（第20页）结构的nsects字段确定。在/usr/include/mach-o/loader.h中声明。

  ```c
  struct section { /* for 32-bit architectures */
  	char		sectname[16];	/* name of this section */
  	char		segname[16];	/* segment this section goes in */
  	uint32_t	addr;		/* memory address of this section */
  	uint32_t	size;		/* size in bytes of this section */
  	uint32_t	offset;		/* file offset of this section */
  	uint32_t	align;		/* section alignment (power of 2) */
  	uint32_t	reloff;		/* file offset of relocation entries */
  	uint32_t	nreloc;		/* number of relocation entries */
  	uint32_t	flags;		/* flags (section type and attributes)*/
  	uint32_t	reserved1;	/* reserved (for offset or index) */
  	uint32_t	reserved2;	/* reserved (for count or sizeof) */
  };
  
  struct section_64 { /* for 64-bit architectures */
  	char		sectname[16];	/* name of this section */
  	char		segname[16];	/* segment this section goes in */
  	uint64_t	addr;		/* memory address of this section */
  	uint64_t	size;		/* size in bytes of this section */
  	uint32_t	offset;		/* file offset of this section */
  	uint32_t	align;		/* section alignment (power of 2) */
  	uint32_t	reloff;		/* file offset of relocation entries */
  	uint32_t	nreloc;		/* number of relocation entries */
  	uint32_t	flags;		/* flags (section type and attributes)*/
  	uint32_t	reserved1;	/* reserved (for offset or index) */
  	uint32_t	reserved2;	/* reserved (for count or sizeof) */
  	uint32_t	reserved3;	/* reserved */
  };
  ```

  **Fields**

  - `sectname`

    A string specifying the name of this section. The value of this field can be any sequence of
    ASCII characters, although section names defined by Apple begin with two underscores and
    consist of lowercase letters (as in __text and __data). This field is fixed at 16 bytes in length.

    指定此节名称的字符串。此字段的值可以是任意一个ASCII字符序列，尽管Apple定义的节名称以两个下划线开头，并且由小写字母组成（如文本和数据）。此字段的长度固定为16字节。

  - `segname`

    A string specifying the name of the segment that should eventually contain this section. For
    compactness, intermediate object files—files of type MH_OBJECT—contain only one segment,
    in which all sections are placed. The static linker places each section in the named segment
    when building the final product (any file that is not of type MH_OBJECT).

    指定最终应包含此节的段的名称的字符串。对于紧凑性，MH_OBJECT类型的中间对象文件只包含一个段，其中放置了所有的段。静态链接器在生成最终产品（任何不属于MH_OBJECT类型的文件）时将每个部分放置在命名段中。

  - `addr`

    An integer specifying the virtual memory address of this section.

    指定此节的虚拟内存地址的整数。

  - `size`

    An integer specifying the size in bytes of the virtual memory occupied by this section.

    一个整数，指定此节占用的虚拟内存的字节大小。

  - `offset`

    An integer specifying the offset to this section in the file.

    一个整数，指定文件中此节的偏移量。

  - `align`

    An integer specifying the section’s byte alignment. Specify this as a power of two; for example,
    a section with 8-byte alignment would have an align value of 3 (2 to the 3rd power equals 8).

    指定节字节对齐方式的整数。将其指定为2的幂；例如，具有8字节对齐的节的对齐值为3（2到3的幂等于8）。

  - `reloff`

    An integer specifying the file offset of the first relocation entry for this section.

    一个整数，指定此节的第一个重定位项的文件偏移量。

  - `nreloc`

    An integer specifying the number of relocation entries located at reloff for this section.

    一个整数，指定位于此节的reloff处的重定位条目数。

  - `flags`

    An integer divided into two parts. The least significant 8 bits contain the section type, while
    the most significant 24 bits contain a set of flags that specify other attributes of the section.
    These types and flags are primarily used by the static linker and file analysis tools, such as
    otool, to determine how to modify or display the section. These are the possible types:

    分成两部分的整数。最低有效8位包含节类型，而最高有效24位包含指定节的其他属性的标志集。这些类型和标志主要由静态链接器和文件分析工具（如otool）使用，以确定如何修改或显示节。以下是可能的类型：

    - `S_REGULAR`

      This section has no particular type. The standard tools create a __TEXT,__text section of this type. 

      此节没有特定类型。标准工具会创建此类型的文本节。

    - `S_ZEROFILL`

      Zero-fill-on-demand section—when this section is first read from or written to, each page within is automatically filled with bytes containing zero. 

      零按需填充节第一次读取或写入此节时，其中的每一页都会自动填充包含零的字节。

    - `S_CSTRING_LITERALS` 

      This section contains only constant C strings. The standard tools create a __TEXT,__cstring section of this type. 

      这个section仅包含C常量字符串。标准工具在此type下创建了__TEXT, __string

    - `S_4BYTE_LITERALS`

      This section contains only constant values that are 4 bytes long. The standard tools create a __TEXT,__literal4 section of this type. 

    - `S_8BYTE_LITERALS `

      This section contains only constant values that are 8 bytes long. The standard tools create a __TEXT,__literal8 section of this type. 

    - `S_LITERAL_POINTERS `

      This section contains only pointers to constant values. 

      本节仅包含指向常量值的指针。

    - `S_NON_LAZY_SYMBOL_POINTERS`

      This section contains only non-lazy pointers to symbols. 

      The standard tools create a section of the __DATA,__nl_symbol_ptrs section of this type.

      此部分仅包含指向符号的非懒加载指针。

    - `S_LAZY_SYMBOL_POINTERS`

      This section contains only lazy pointers to symbols. The 

      standard tools create a __DATA,__la_symbol_ptrs section of this type. 

       此部分仅包含指向符号的懒加载指针。

    - `S_SYMBOL_STUBS` 

      This section contains symbol stubs. The standard tools create __TEXT,__symbol_stub and __TEXT,__picsymbol_stub sections of this type. See “Dynamic Code Generation” in *Mach-O Programming Topics* for more information. 

      本节包含符号存根

    - `S_MOD_INIT_FUNC_POINTERS` 

      This section contains pointers to module initialization functions. The standard tools create __DATA,__mod_init_func sections of this type. 

      包含模块初始化方法的指针

    - `S_MOD_TERM_FUNC_POINTERS `

      This section contains pointers to module termination functions. The standard tools create __DATA,__mod_term_func sections of this type. 

      包含模块结束方法的指针

    - `S_COALESCED`

      This section contains symbols that are coalesced by the static linker and possibly the dynamic linker. More than one file may contain coalesced definitions of the same symbol without causing multiple-defined-symbol errors. 

      此部分包含由静态链接器（可能还有动态链接器）合并的符号。多个文件可能包含同一符号的合并定义，但不会导致多个定义的符号错误。

    - `S_GB_ZEROFILL `

      This is a zero-filled on-demand section. It can be larger than 4 GB. This section must be placed in a segment containing only zero-filled sections. If you place a zero-filled section in a segment with non–zero-filled sections, you may cause those sections to be unreachable with a 31-bit offset. That outcome stems from the fact that the size of a zero-filled section can be larger than 4 GB (in a 32-bit address space). As a result of this, the static linker would be unable to build the output file. See segment_command (page 20) for more information. 

      这是一个零填充的按需部分。它可以大于4 GB。此节必须放在只包含零填充节的段中。如果将零填充部分放在具有非零填充部分的段中，则可能会导致以31位偏移量无法访问这些部分。这一结果源于这样一个事实：零填充部分的大小可以大于4GB（在32位地址空间中）。因此，静态链接器将无法生成输出文件。有关更多信息，请参阅StEngSub命令（第20页）。

      

    The following are the possible attributes of a section:

    - `S_ATTR_PURE_INSTRUCTIONS` 

      This section contains only executable machine instructions. The standard tools set this flag for the sections __TEXT,__text, __TEXT,__symbol_stub, and __TEXT,__picsymbol_stub. 

      本节仅包含可执行的机器指令。

    - `S_ATTR_SOME_INSTRUCTIONS `

      This section contains executable machine instructions. 

    - `S_ATTR_NO_TOC`

      This section contains coalesced symbols that must not be placed in the 

      table of contents (SYMDEF member) of a static archive library. 

    - `S_ATTR_EXT_RELOC` 

      This section contains references that must be relocated. These references refer to data that exists in other files (undefined symbols). To support external relocation, the maximum virtual memory protections of the segment that contains this section must allow both reading and writing. 

      本节包含必须重新定位的引用。这些引用引用其他文件中存在的数据（未定义的符号）。为了支持外部重定位，包含此节的段的最大虚拟内存保护必须允许读写。

    - `S_ATTR_LOC_RELOC`

      This section contains references that must be relocated. These references refer to data within this file. 

      本节包含必须重新定位的引用。这些引用引用此文件中的数据。

    - `S_ATTR_STRIP_STATIC_SYMS `

      The static symbols in this section can be stripped if the MH_DYLDLINK flag of the image’s mach_header (page 12) header structure is set. 

    - `S_ATTR_NO_DEAD_STRIP `

      This section must not be dead-stripped. See “Linking” in *Xcode User Guide* for details. 

    - `S_ATTR_LIVE_SUPPORT `

      This section must not be dead-stripped if they reference code that is live, but the reference is undetectable. 

  - `reserved1`

    An integer reserved for use with certain section types. For symbol pointer sections and symbol stubs sections that refer to indirect symbol table entries, this is the index into the indirect table for this section’s entries. The number of entries is based on the section size divided by the size of the symbol pointer or stub. Otherwise, this field is set to 0. 

    保留用于某些节类型的整数。对于引用间接符号表项的符号指针节和符号存根节，这是指向该节项的间接表的索引。条目的数目基于节大小除以符号指针或存根的大小。否则，此字段设置为0。

  - `reserved2`

    For sections of type S_SYMBOL_STUBS, an integer specifying the size (in bytes) of the symbol 

    stub entries contained in the section. Otherwise, this field is reserved for future use and should be set to 0. 

    对于S_SYMBOL_STUBS类型的section，使用整数指定符号的大小(以字节为单位)
    section中包含的stub条目。否则，此字段将保留以供将来使用，并应设置为0。

    

  **Discussion** 

  Each section in a Mach-O file has both a type and a set of attribute flags. In intermediate object files, the type and attributes determine how the static linker copies the sections into the final product. Object file analysis tools (such as otool) use the type and attributes to determine how to read and display the sections. Some section types and attributes are used by the dynamic linker. 

  These are important static-linking variants of the symbol type and attributes:

  mach-o文件中的每个部分都有一个类型和一组属性标志。在中间对象文件中，类型和属性决定静态链接器如何将节复制到最终产品中。对象文件分析工具（如otool）使用类型和属性来确定如何读取和显示节。动态链接器使用某些节类型和属性。
  这些是符号类型和属性的重要静态链接变体： 

  - **Regular sections**. In a regular section, only one definition of an external symbol may exist in intermediate object files. The static linker returns an error if it finds any duplicate external symbol definitions. 

    在常规部分中，中间对象文件中只能存在外部符号的一个定义。如果发现任何重复的外部符号定义，静态链接器将返回错误。

  - **Coalesced sections**. In the final product, the static linker retains only one instance of each symbol defined in coalesced sections. To support complex language features (such as C++ vtables and RTTI) the compiler may create a definition of a particular symbol in every intermediate object file. The static linker and the dynamic linker would then reduce the duplicate definitions to the single definition used by the program. 

    在最终产品中，静态链接器只保留合并部分中定义的每个符号的一个实例。为了支持复杂的语言特性（如C++ VTABLE和RTTI），编译器可以在每个中间对象文件中创建一个特定符号的定义。静态链接器和动态链接器将把重复的定义减少到程序使用的单个定义。

  - **Coalesced sections with weak definitions** Weak symbol definitions may appear only in coalesced sections. When the static linker finds duplicate definitions for a symbol, it discards any coalesced symbol definition that has the weak definition attribute set (see nlist (page 39)). If there are no non-weak definitions, the first weak definition is used instead. This feature is designed to support C++ templates; it allows explicit template instantiations to override implicit ones. The C++ compiler places explicit definitions in a regular section, and it places implicit definitions in a coalesced section, marked as weak definitions. Intermediate object files (and thus static archive libraries) built with weak definitions can be used only with the static linker in Mac OS X v10.2 and later. Final products (applications and shared libraries) should not contain weak definitions if they are expected to be used on earlier versions of Mac OS X. 

    弱符号定义只能出现在合并部分中。当静态链接器发现符号的重复定义时，它将丢弃任何具有弱定义属性集的合并符号定义（请参阅nlist（第39页））。如果没有非弱定义，则使用第一个弱定义。该特性被设计为支持C++模板；它允许显式模板实例化来重写隐式模板。C++编译器将显式定义放在一个常规的部分中，它将隐含的定义放在一个合并的区段中，标记为弱定义。使用弱定义构建的中间对象文件（以及静态存档库）只能与Mac OS X v10.2及更高版本中的静态链接器一起使用。如果最终产品（应用程序和共享库）预期在早期版本的MacOSX上使用，则不应包含弱定义。

    

- **twolevel_hints_command**

  Defines the attributes of a LC_TWOLEVEL_HINTS load command

  ```c
  /*
   * The twolevel_hints_command contains the offset and number of hints in the
   * two-level namespace lookup hints table.
   */
  struct twolevel_hints_command {
      uint32_t cmd;	/* LC_TWOLEVEL_HINTS */
      uint32_t cmdsize;	/* sizeof(struct twolevel_hints_command) */
      uint32_t offset;	/* offset to the hint table */
      uint32_t nhints;	/* number of hints in the hint table */
  };
  ```

  **Fields**

  - `cmd`

    Common to all load command structures. Set to LC_TWOLEVEL_HINTS for this structure.

  - `cmdsize`

    Common to all load command structures. For this structure, set to
    sizeof(twolevel_hints_command).

  - `offset`

    An integer specifying the byte offset from the start of this file to an array of 

    twolevel_hint (page 30) data structures, known as the two-level namespace hint table. 

    在文件内的offset

  - `nhints`

    The number of twolevel_hint data structures located at offset.

    位于偏移量的二级提示数据结构的数目。

    

  **Discussion** 

  The static linker adds the LC_TWOLEVEL_HINTS load command and the two-level namespace hint table to the output file when building a two-level namespace image. 

  讨论
  静态链接器在生成两级命名空间映像时将lc_twowlevel_hints load命令和两级命名空间提示表添加到输出文件中。

  **Special Considerations**
  By default, ld does not include the LC_TWOLEVEL_HINTS command or the two-level namespace hint table in an MH_BUNDLE file because the presence of this load command causes the version of the dynamic linker shipped with Mac OS X v10.0 to crash. If you know the code will run only on Mac OS X v10.1 and later, you should explicitly enable the two-level namespace hint table. See -twolevel_namespace_hints in the ld man page for more information.

  特殊注意事项
  默认情况下，ld不会在MH_BUNDLE文件中包含lc_twolevel_hint命令或两级名称空间提示表，因为这个load命令的存在会导致Mac OS X v10.0附带的动态链接器版本崩溃。如果您知道代码只在Mac OS X v10.1及更高版本上运行，那么应该显式启用两级名称空间提示表。有关更多信息，请参见ld手册页中的- twolevel_namespace_tips。

  

- **twolevel_hint**

  Specifies an entry in the two-level namespace hint table. Declared in /usr/include/mach-o/loader.h.

  ```c
  struct twolevel_hint {
      uint32_t 
  	isub_image:8,	/* index into the sub images */
  	itoc:24;	/* index into the table of contents */
  };
  ```

  **Fields**

  - `isub_image`

    The subimage in which the symbol is defined. It is an index into the list of images that make
    up the umbrella image. If this field is 0, the symbol is in the umbrella image itself. If the image is not an umbrella framework or library, this field is 0.

    定义符号的subimage。它是组成雨伞图像的图像列表的索引。如果此字段为0，则符号在伞图像本身中。如果图像不是伞形框架或库，则此字段为0。

  - `itoc`

    The symbol index into the table of contents of the image specified by the isub_image field.

    由isub_image字段指定的图像目录中的符号索引。

    

  **Discussion** 

  The two-level namespace hint table provides the dynamic linker with suggested positions to start searching for symbols in the libraries the current image is linked against. 

  Every undefined symbol (that is, every symbol of type N_UNDF or N_PBUD) in a two-level namespace image has a corresponding entry in the two-level hint table, at the same index. 

  The static linker adds the LC_TWOLEVEL_HINTS load command and the two-level namespace hint table to the output file when building a two-level namespace image. 

  By default, the linker does not include the LC_TWOLEVEL_HINTS command or the two-level namespace hint table in an MH_BUNDLE file, because the presence of this load command causes the version of the dynamic linker shipped with Mac OS X v10.0 to crash. If you know the code will run only on Mac OS X v10.1 and later, you should explicitly enable the two-level namespace hints. See the linker documentation for more information. 
  两级命名空间提示表为动态链接器提供建议的位置，以便开始在当前图像所链接的库中搜索符号。
  两级命名空间映像中的每个未定义符号（即，类型为n_undf或n_pbud的每个符号）在两级提示表中的同一索引处都有相应的项。
  静态链接器在生成两级命名空间映像时将lc_twowlevel_hints load命令和两级命名空间提示表添加到输出文件中。
  默认情况下，链接器不包括mh_捆绑包文件中的lc_two level_hints命令或两级命名空间提示表，因为此加载命令的存在会导致mac os x v10.0附带的动态链接器版本崩溃。如果您知道代码将只在macosxv10.1及更高版本上运行，那么应该显式地启用两级命名空间提示。有关详细信息，请参阅链接器文档。

  

- **lc_str**

  Defines a variable-length string. Declared in /usr/include/mach-o/loader.h.

  ```c
  union lc_str {
  	uint32_t	offset;	/* offset to the string */
  #ifndef __LP64__
  	char		*ptr;	/* pointer to the string */
  #endif 
  };
  ```

  **Fields**

  - `offset`

    A long integer. A byte offset from the start of the load command that contains this string to
    the start of the string data.

    一个长整数。从包含此字符串的加载命令开始到字符串数据开始的字节偏移量。

  - `ptr`

    A pointer to an array of bytes. At runtime, this pointer contains the virtual memory address
    of the string data. The ptr field is not used in Mach-O files.

    指向字节数组的指针。在运行时，此指针包含字符串数据的虚拟内存地址。在mach-o文件中不使用ptr字段。

  **Discussion** 

  Load commands store variable-length data such as library names using the lc_str data structure. Unless otherwise specified, the data consists of a C string. 

  The data pointed to is stored just after the load command, and the size is added to the size of the load command. The string should be null terminated; any extra bytes to round up the size should be null. You can also determine the size of the string by subtracting the size of the load command data structure from the cmdsize field of the load command data structure. 
  `Load Commands`使用`lc_str`数据结构存储可变长度的数据，例如库名称。除非另有说明，否则数据由C字符串组成。
  指向的数据存储在`load command`之后，大小将添加到`load command`的大小中。字符串应以空结尾；要舍入大小的任何额外字节都应为空。还可以通过从加载命令数据结构的cmdSize字段中减去加载命令数据结构的大小来确定字符串的大小。



- **dylib**

  Defines the data used by the dynamic linker to match a shared library against the files that have linked to it. Used exclusively in the dylib_command (page 32) data structure. Declared in
  /usr/include/mach-o/loader.h.

  定义动态链接器用于将共享库与链接到该库的文件匹配的数据

  ```c
  /*
   * Dynamicly linked shared libraries are identified by two things.  The
   * pathname (the name of the library as found for execution), and the
   * compatibility version number.  The pathname must match and the compatibility
   * number in the user of the library must be greater than or equal to the
   * library being used.  The time stamp is used to record the time a library was
   * built and copied into user so it can be use to determined if the library used
   * at runtime is exactly the same as used to built the program.
   */
  struct dylib {
      union lc_str  name;			/* library's path name */
      uint32_t timestamp;			/* library's build time stamp */
      uint32_t current_version;		/* library's current version number */
      uint32_t compatibility_version;	/* library's compatibility vers number*/
  };
  ```

  **Fields**

  - `name`

    A data structure of type lc_str (page 31). Specifies the name of the shared library.

    指定共享库的名称 是一个lc_str的结构类型

  - `timestamp`

    The date and time when the shared library was built.

    创建共享库的日期和时间。

  - `current_version`

    The current version of the shared library.

    共享库的当前版本

  - `compatibility_version`

    The compatibility version of the shared library.

    共享库的兼容版本

    

- **dylib_command**

  Defines the attributes of the `LC_LOAD_DYLIB` and `LC_ID_DYLIB` load commands. Declared in
  /usr/include/mach-o/loader.h.

  ```c
  /*
   * A dynamically linked shared library (filetype == MH_DYLIB in the mach header)
   * contains a dylib_command (cmd == LC_ID_DYLIB) to identify the library.
   * An object that uses a dynamically linked shared library also contains a
   * dylib_command (cmd == LC_LOAD_DYLIB, LC_LOAD_WEAK_DYLIB, or
   * LC_REEXPORT_DYLIB) for each library it uses.
   */
  struct dylib_command {
  	uint32_t	cmd;		/* LC_ID_DYLIB, LC_LOAD_{,WEAK_}DYLIB,
  					   LC_REEXPORT_DYLIB */
  	uint32_t	cmdsize;	/* includes pathname string */
  	struct dylib	dylib;		/* the library identification */
  };
  ```

  **Fields**

  - `cmd`

    Common to all load command structures. For this structure, set to either` LC_LOAD_DYLIB`,
    `LC_LOAD_WEAK_DYLIB`, or `LC_ID_DYLIB`.

  - `cmdsize`

    Common to all load command structures. For this structure, set to sizeof(dylib_command)
    plus the size of the data pointed to by the name field of the dylib field.

  - `dylib`

    A data structure of type dylib (page 31). Specifies the attributes of the shared library.

    dylib类型的数据结构。指定共享库的属性。

    

  **Discussion** 

  For each shared library that a file links against, the static linker creates an LC_LOAD_DYLIB command and sets its dylib field to the value of the dylib field of the LC_ID_DYLD load command of the target library. All the LC_LOAD_DYLIB commands together form a list that is ordered according to location in the file, earliest LC_LOAD_DYLIB command first. For two-level namespace files, undefined symbol entries in the symbol table refer to their parent shared libraries by index into this list. The index is called a *library ordinal*, and it is stored in the n_desc field of the nlist (page 39) data structure. 
  对于文件链接所针对的每个共享库，静态链接器创建一个LC_LOAD_DYLIB命令，并将其dylib字段设置为目标库的LC_ID_DYLD load commands的dylib字段的值。所有LC_LOAD_DYLIB命令一起形成一个列表，该列表根据文件中的位置进行排序，首先是最早的LC_LOAD_DYLIB命令。对于两级命名空间文件，符号表中未定义的符号项通过索引指向该列表中的父共享库。索引称为库序号，它存储在nlist数据结构的n_desc字段中。

  

  At runtime, the dynamic linker uses the name in the dyld field of the LC_LOAD_DYLIB command to locate the shared library. If it finds the library, the dynamic linker compares the version information of the LC_LOAD_DYLIB load command against the library’s version. For the dynamic linker to successfully link the shared library, the compatibility version of the shared library must be less than or equal to the compatibility version in the LC_LOAD_DYLIB command.

  在运行时，动态链接器使用LC_LOAD_DYLIB命令的dyld字段中的名称来定位共享库。如果找到库，动态链接器将LC_LOAD_DYLIB `load commands`的版本信息与库的版本进行比较。要使动态链接器成功链接共享库，共享库的兼容版本必须小于或等于LC_LOAD_DYLIB命令中的兼容版本。

  

  The dynamic linker uses the timestamp to determine whether it can use the prebinding information.
  The current version is returned by the function NSVersionOfRunTimeLibrary to allow you to determine the version of the library your program is using.

  动态链接器使用时间戳来确定是否可以使用预绑定信息。函数NSVersionOfRunTimeLibrary返回当前版本，允许您确定程序正在使用的库的版本。



- **dylinker_command**

  Defines the attributes of the `LC_LOAD_DYLINKER` and `LC_ID_DYLINKER` load commands. Declared in
  /usr/include/mach-o/loader.h.

  ```c
  /*
   * A program that uses a dynamic linker contains a dylinker_command to identify
   * the name of the dynamic linker (LC_LOAD_DYLINKER).  And a dynamic linker
   * contains a dylinker_command to identify the dynamic linker (LC_ID_DYLINKER).
   * A file can have at most one of these.
   * This struct is also used for the LC_DYLD_ENVIRONMENT load command and
   * contains string for dyld to treat like environment variable.
   */
  struct dylinker_command {
  	uint32_t	cmd;		/* LC_ID_DYLINKER, LC_LOAD_DYLINKER or
  					   LC_DYLD_ENVIRONMENT */
  	uint32_t	cmdsize;	/* includes pathname string */
  	union lc_str    name;		/* dynamic linker's path name */
  };
  ```

  **Fields**

  - `cmd`

    Common to all load command structures. For this structure, set to either `LC_ID_DYLINKER` or
    `LC_LOAD_DYLINKER`.

  - `cmdsize`

    Common to all load command structures. For this structure, set to sizeof(dylinker_command),
    plus the size of the data pointed to by the name field.

  - `name`

    A data structure of type lc_str (page 31). Specifies the name of the dynamic linker

  **Discussion** 

  Every executable file that is dynamically linked contains a LC_LOAD_DYLINKER command that specifies the name of the dynamic linker that the kernel must load in order to execute the file. The dynamic linker itself specifies its name using the LC_ID_DYLINKER load command. 

  每个动态链接的可执行文件都包含一个LC_LOAD_DYLINKER命令，该命令指定内核为了执行文件必须加载的动态链接器的名称。动态链接器本身使用LC_ID_DYLINKER load command指定其名称。



- **prebound_dylib_command**

  Defines the attributes of the `LC_PREBOUND_DYLIB` load command. For every library that a prebound executable file links to, the static linker adds one LC_PREBOUND_DYLIB command. Declared in
  /usr/include/mach-o/loader.h.

  ```c
  /*
   * A program (filetype == MH_EXECUTE) that is
   * prebound to its dynamic libraries has one of these for each library that
   * the static linker used in prebinding.  It contains a bit vector for the
   * modules in the library.  The bits indicate which modules are bound (1) and
   * which are not (0) from the library.  The bit for module 0 is the low bit
   * of the first byte.  So the bit for the Nth module is:
   * (linked_modules[N/8] >> N%8) & 1
   */
  struct prebound_dylib_command {
  	uint32_t	cmd;		/* LC_PREBOUND_DYLIB */
  	uint32_t	cmdsize;	/* includes strings */
  	union lc_str	name;		/* library's path name */
  	uint32_t	nmodules;	/* number of modules in library */
  	union lc_str	linked_modules;	/* bit vector of linked modules */
  };
  ```

  **Fields**

  - `cmd`

    Common to all load command structures. For this structure, set to LC_PREBOUND_DYLIB.

  - `cmdsize`

    Common to all load command structures. For this structure, set to
    sizeof(prebound_dylib_command) plus the size of the data pointed to by the name and
    linked_modules fields.

  - `name`

    A data structure of type lc_str (page 31). Specifies the name of the prebound shared library.

    预绑定共享库的名称

  - `nmodules`

    An integer. Specifies the number of modules the prebound shared library contains. The size
    of the linked_modules string is (nmodules / 8) + (nmodules % 8).

    一个整数。指定预绑定共享库包含的模块数。链接的模块字符串的大小为（nmodules/8）+（nmodules%8）

  - `linked_modules`

    A data structure of type lc_str (page 31). Usually, this data structure defines the offset of a
    C string; in this usage, it is a variable-length bitset, containing one bit for each module. Each
    bit represents whether the corresponding module is linked to a module in the current file, 1
    for yes, 0 for no. The bit for the first module is the low bit of the first byte

    lc_str类型的数据结构（第31页）。通常，此数据结构定义C字符串的偏移量；在这种用法中，它是一个可变长度的位集，每个模块包含一个位。每个位表示对应的模块是否链接到当前文件中的模块，1表示是，0表示否。第一个模块的位是第一个字节的低位

    

- **thread_command**

  Defines the attributes of the` LC_THREAD` and `LC_UNIXTHREAD` load commands. The data of this command is specific to each architecture and appears in thread_status.h, located in the architecture’s directory in /usr/include/mach. Declared in /usr/include/mach-o/loader.h.

  ```c
  struct thread_command {
  	uint32_t	cmd;		/* LC_THREAD or  LC_UNIXTHREAD */
  	uint32_t	cmdsize;	/* total size of this command */
  	/* uint32_t flavor		   flavor of thread state */
  	/* uint32_t count		   count of uint32_t's in thread state */
  	/* struct XXX_thread_state state   thread state for this flavor */
  	/* ... */
  };
  ```

  **Fields**

  - `cmd`

    Common to all load command structures. For this structure, set to `LC_THREAD` or `LC_UNIXTHREAD`.

  - `cmdsize`

    Common to all load command structures. For this structure, set to sizeof(thread_command) 

    plus the size of the flavor and count fields plus the size of the CPU-specific thread state data structure. 

  - `flavor`

    Integer specifying the particular flavor of the thread state data structure. See the thread_status.h file for your target architecture.

    整数，指定线程状态数据结构的特殊类型。请参阅目标体系结构的thread_status.h文件。

  - `count`

    Size of the thread state data, in number of 32-bit integers. The thread state data structure must be fully padded to 32-bit alignment.

    线程状态数据的大小，以32位整数为单位。线程状态数据结构必须完全填充为32位对齐。



- **routines_command**

  Defines the attributes of the `LC_ROUTINES` load command, used in 32-bit architectures. Describes the location of the shared library initialization function, which is a function that the dynamic linker calls before allowing any of the routines in the library to be called. Declared in
  /usr/include/mach-o/loader.h. See also routines_command_64 

  定义32位体系结构中使用的LC_ROUTINES load commands的属性。描述共享库初始化函数的位置，该函数是动态链接器在允许调用库中的任何例程之前调用的函数。在/usr/include/mach-o/loader.h中声明。另请参见例程命令（第36页）。

  ```c
  /*
   * The routines command contains the address of the dynamic shared library 
   * initialization routine and an index into the module table for the module
   * that defines the routine.  Before any modules are used from the library the
   * dynamic linker fully binds the module that defines the initialization routine
   * and then calls it.  This gets called before any module initialization
   * routines (used for C++ static constructors) in the library.
   */
  struct routines_command { /* for 32-bit architectures */
  	uint32_t	cmd;		/* LC_ROUTINES */
  	uint32_t	cmdsize;	/* total size of this command */
  	uint32_t	init_address;	/* address of initialization routine */
  	uint32_t	init_module;	/* index into the module table that */
  				        /*  the init routine is defined in */
  	uint32_t	reserved1;
  	uint32_t	reserved2;
  	uint32_t	reserved3;
  	uint32_t	reserved4;
  	uint32_t	reserved5;
  	uint32_t	reserved6;
  };
  
  /*
   * The 64-bit routines command.  Same use as above.
   */
  struct routines_command_64 { /* for 64-bit architectures */
  	uint32_t	cmd;		/* LC_ROUTINES_64 */
  	uint32_t	cmdsize;	/* total size of this command */
  	uint64_t	init_address;	/* address of initialization routine */
  	uint64_t	init_module;	/* index into the module table that */
  					/*  the init routine is defined in */
  	uint64_t	reserved1;
  	uint64_t	reserved2;
  	uint64_t	reserved3;
  	uint64_t	reserved4;
  	uint64_t	reserved5;
  	uint64_t	reserved6;
  };
  ```

  **Fields**

  - `cmd`

    Common to all load command structures. For this structure, set to LC_ROUTINES.

  - `cmdsize`

    Common to all load command structures. For this structure, set to sizeof(routines_command).

  - `init_address`

    An integer specifying the virtual memory address of the initialization function.

    指定初始化函数的虚拟内存地址的整数。

  - `init_module`

    An integer specifying the index into the module table of the module containing the initialization function.

    一个整数，指定包含初始化函数的模块的模块表中的索引。

  - `reserved1`

    Reserved for future use. Set this field to 0.

    保留以备将来使用。将此字段设置为0

  - `reserved2`

    Reserved for future use. Set this field to 0.

  - `reserved3`

    Reserved for future use. Set this field to 0.

  - `reserved4`

    Reserved for future use. Set this field to 0.

  - `reserved5`

    Reserved for future use. Set this field to 0.

  - `reserved6`

    Reserved for future use. Set this field to 0.

  **Discussion** 

  The static linker adds an LC_ROUTINES command when you specify a shared library initialization function using the -init option (see the ld man page for more information). 
  使用-init选项指定共享库初始化函数时，静态链接器会添加LC_ROUTINES命令（有关详细信息，请参阅ld手册页）。

  

- **sub_framework_command**

  Defines the attributes of the LC_SUB_FRAMEWORK load command. Identifies the umbrella framework of which this file is a subframework. Declared in /usr/include/mach-o/loader.h.

  ```c
  /*
   * A dynamically linked shared library may be a subframework of an umbrella
   * framework.  If so it will be linked with "-umbrella umbrella_name" where
   * Where "umbrella_name" is the name of the umbrella framework. A subframework
   * can only be linked against by its umbrella framework or other subframeworks
   * that are part of the same umbrella framework.  Otherwise the static link
   * editor produces an error and states to link against the umbrella framework.
   * The name of the umbrella framework for subframeworks is recorded in the
   * following structure.
   */
  struct sub_framework_command {
  	uint32_t	cmd;		/* LC_SUB_FRAMEWORK */
  	uint32_t	cmdsize;	/* includes umbrella string */
  	union lc_str 	umbrella;	/* the umbrella framework name */
  };
  ```

  **Fields**

  - `cmd`

  - `cmdsize`

  - `umbrella`

    A data structure of type lc_str (page 31). Specifies the name of the umbrella framework of
    which this file is a member.

    

- **sub_umbrella_command**

  Defines the attributes of the LC_SUB_UMBRELLA load command. Identifies the named framework as a subumbrella of this framework. Unlike a subframework, any client may link to a subumbrella. Declared
  in /usr/include/mach-o/loader.h.

  ```c
  /*
   * A dynamically linked shared library may be a sub_umbrella of an umbrella
   * framework.  If so it will be linked with "-sub_umbrella umbrella_name" where
   * Where "umbrella_name" is the name of the sub_umbrella framework.  When
   * staticly linking when -twolevel_namespace is in effect a twolevel namespace 
   * umbrella framework will only cause its subframeworks and those frameworks
   * listed as sub_umbrella frameworks to be implicited linked in.  Any other
   * dependent dynamic libraries will not be linked it when -twolevel_namespace
   * is in effect.  The primary library recorded by the static linker when
   * resolving a symbol in these libraries will be the umbrella framework.
   * Zero or more sub_umbrella frameworks may be use by an umbrella framework.
   * The name of a sub_umbrella framework is recorded in the following structure.
   */
  struct sub_umbrella_command {
  	uint32_t	cmd;		/* LC_SUB_UMBRELLA */
  	uint32_t	cmdsize;	/* includes sub_umbrella string */
  	union lc_str 	sub_umbrella;	/* the sub_umbrella framework name */
  };
  ```

  **Fields**

  - `cmd`

  - `cmdsize`

  - `sub_umbrella`

    A data structure of type lc_str (page 31). Specifies the name of the umbrella framework of
    which this file is a member.

    

- **sub_library_command**

  Defines the attributes of the LC_SUB_LIBRARY load command. Identifies a sublibrary of this framework and marks this framework as an umbrella framework. Unlike a subframework, any client may link to a sublibrary. Declared in /usr/include/mach-o/loader.h.

  ```c
  /*
   * A dynamically linked shared library may be a sub_library of another shared
   * library.  If so it will be linked with "-sub_library library_name" where
   * Where "library_name" is the name of the sub_library shared library.  When
   * staticly linking when -twolevel_namespace is in effect a twolevel namespace 
   * shared library will only cause its subframeworks and those frameworks
   * listed as sub_umbrella frameworks and libraries listed as sub_libraries to
   * be implicited linked in.  Any other dependent dynamic libraries will not be
   * linked it when -twolevel_namespace is in effect.  The primary library
   * recorded by the static linker when resolving a symbol in these libraries
   * will be the umbrella framework (or dynamic library). Zero or more sub_library
   * shared libraries may be use by an umbrella framework or (or dynamic library).
   * The name of a sub_library framework is recorded in the following structure.
   * For example /usr/lib/libobjc_profile.A.dylib would be recorded as "libobjc".
   */
  struct sub_library_command {
  	uint32_t	cmd;		/* LC_SUB_LIBRARY */
  	uint32_t	cmdsize;	/* includes sub_library string */
  	union lc_str 	sub_library;	/* the sub_library name */
  };
  ```

  

- **sub_client_command**

  Defines the attributes of the LC_SUB_CLIENT load command. Specifies the name of a file that is allowed to link to this subframework. This file would otherwise be required to link to the umbrella framework of which this file is a component. Declared in /usr/include/mach-o/loader.h.

  ```c
  /*
   * For dynamically linked shared libraries that are subframework of an umbrella
   * framework they can allow clients other than the umbrella framework or other
   * subframeworks in the same umbrella framework.  To do this the subframework
   * is built with "-allowable_client client_name" and an LC_SUB_CLIENT load
   * command is created for each -allowable_client flag.  The client_name is
   * usually a framework name.  It can also be a name used for bundles clients
   * where the bundle is built with "-client_name client_name".
   */
  struct sub_client_command {
  	uint32_t	cmd;		/* LC_SUB_CLIENT */
  	uint32_t	cmdsize;	/* includes client string */
  	union lc_str 	client;		/* the client name */
  };
  ```

  **Special Considerations** 

  The ld tool generates a sub_client_command load command in the built product if you pass the option -allowable_client <name>, where <name> is the install name of a framework or the client name of a bundle. See the ld man page, specifically about the options -allowable_client and -client_name, for more information. 

  如果传递选项-allowable_client<name>，其中<name>是框架的安装名称或捆绑包的客户端名称，则ld工具会在生成的产品中生成一个sub_client_command load commands。有关详细信息，请参阅ld手册页，特别是关于选项-允许的客户机和-客户机名称。



## Symbol Table and Related Data Structures

Two load commands, LC_SYMTAB and LC_DYSYMTAB, describe the size and location of the symbol
tables, along with additional metadata. The other data structures listed in this section represent the
symbol tables themselves.

两个加载命令`LC_SYMTAB`和`LC_DYSYMTAB`描述了符号表的大小和位置以及其他元数据。本节中列出的其他数据结构表示符号表本身。

- **symtab_command**

  Defines the attributes of the` LC_SYMTAB` load command. Describes the size and location of the symbol table data structures. Declared in /usr/include/mach-o/loader.h.

  定义lc_symtab load commands的属性。描述符号表数据结构的大小和位置。在/usr/include/mach-o/loader.h中声明。

  ```c
  /*
   * The symtab_command contains the offsets and sizes of the link-edit 4.3BSD
   * "stab" style symbol table information as described in the header files
   * <nlist.h> and <stab.h>.
   */
  struct symtab_command {
  	uint32_t	cmd;		/* LC_SYMTAB */
  	uint32_t	cmdsize;	/* sizeof(struct symtab_command) */
  	uint32_t	symoff;		/* symbol table offset */
  	uint32_t	nsyms;		/* number of symbol table entries */
  	uint32_t	stroff;		/* string table offset */
  	uint32_t	strsize;	/* string table size in bytes */
  };
  ```

  **Fields**

  - `cmd`

  - `cmdsize`

  - `symoff`

    An integer containing the byte offset from the start of the file to the location of the symbol table entries. The symbol table is an array of nlist (page 39) data structures.

    包含从文件开始到符号表项位置的字节偏移量的整数。符号表是一个由nlist(第39页)数据结构组成的数组。

  - `nsyms`

    An integer indicating the number of entries in the symbol table.

    符号表中条目数

  - `stroff`

    An integer containing the byte offset from the start of the image to the location of the string
    table.

    包含从图像开始到字符串表位置的字节偏移量。

  - `strsize`

    An integer indicating the size (in bytes) of the string table.

    表示字符串表的大小（字节）。

  **Discussion** 

  LC_SYMTAB should exist in both statically linked and dynamically linked file types. 

  `LC_SYMTAB`应该同时存在于静态链接和动态链接的文件类型中。

  

- **nlist**

  Describes an entry in the symbol table for 32-bit architectures. Declared in
  /usr/include/mach-o/nlist.h. See also nlist_64

  ```c
  struct nlist {
  	union {
  #ifndef __LP64__
  		char *n_name;	/* for use when in-core */
  #endif
  		uint32_t n_strx;	/* index into the string table */
  	} n_un;
  	uint8_t n_type;		/* type flag, see below */
  	uint8_t n_sect;		/* section number or NO_SECT */
  	int16_t n_desc;		/* see <mach-o/stab.h> */
  	uint32_t n_value;	/* value of this symbol (or stab offset) */
  };
  
  /*
   * This is the symbol table entry structure for 64-bit architectures.
   */
  struct nlist_64 {
      union {
          uint32_t  n_strx; /* index into the string table */
      } n_un;
      uint8_t n_type;        /* type flag, see below */
      uint8_t n_sect;        /* section number or NO_SECT */
      uint16_t n_desc;       /* see <mach-o/stab.h> */
      uint64_t n_value;      /* value of this symbol (or stab offset) */
  };
  ```

  **Fields**

  - `n_un`

    A union that holds an index into the string table, n_strx. To specify an empty string (""), set
    this value to 0. The n_name field is not used in Mach-O files.

    共用体保存着在string table中的index，n_strx。若要指定空字符串（“”），请将此值设置为0。mach-o文件中不使用n_name字段。

  - `n_type`

    A byte value consisting of data accessed using four bit masks:

    一个字节值，由使用四位掩码访问的数据组成：

    ```c
    /*
     * The n_type field really contains four fields:
     *	unsigned char N_STAB:3,
     *		      N_PEXT:1,
     *		      N_TYPE:3,
     *		      N_EXT:1;
     * which are used via the following masks.
     */
    #define	N_STAB	0xe0  /* if any of these bits set, a symbolic debugging entry */
    #define	N_PEXT	0x10  /* private external symbol bit */
    #define	N_TYPE	0x0e  /* mask for the type bits */
    #define	N_EXT	0x01  /* external symbol bit, set for external symbols */
    
    /*
     * Only symbolic debugging entries have some of the N_STAB bits set and if any
     * of these bits are set then it is a symbolic debugging entry (a stab).  In
     * which case then the values of the n_type field (the entire field) are given
     * in <mach-o/stab.h>
     */
    
    /*
     * Values for N_TYPE bits of the n_type field.
     */
    #define	N_UNDF	0x0		/* undefined, n_sect == NO_SECT */
    #define	N_ABS	0x2		/* absolute, n_sect == NO_SECT */
    #define	N_SECT	0xe		/* defined in section number n_sect */
    #define	N_PBUD	0xc		/* prebound undefined (defined in a dylib) */
    #define N_INDR	0xa		/* indirect */
    ```

    - `N_STAB (0xe0)`—If any of these 3 bits are set, the symbol is a symbolic debugging table (stab) entry. In that case, the entire n_type field is interpreted as a stab value. See /usr/include/mach-o/stab.h for valid stab values. 

      如果设置了这3位中的任何一位，则符号是符号调试表（stab）项。在这种情况下，整个n_type字段被解释为stab值。有关有效的stab值，请参见/usr/include/mach-o/stab.h。

    - `N_PEXT (0x10)`—If this bit is on, this symbol is marked as having limited global scope. When the file is fed to the static linker, it clears the N_EXT bit for each symbol with the N_PEXT bit set. (The ld option -keep_private_externs turns off this behavior.) With Mac OS X GCC, you can use the __private_extern__ function attribute to set this bit. 

      如果该位为开，则该符号被标记为具有有限的全局范围。当文件被馈送到静态链接器时，它会为每个设置了n_-pext位的符号清除n_-ext位。（ld选项-keep_private_externs关闭此行为）使用mac os x gcc，可以使用u private_u extern_uu函数属性设置此位。

    - `N_TYPE (0x0e)`—These bits define the type of the symbol. 

      定义符号的类型

    - `N_EXT (0x01)`—If this bit is on, this symbol is an external symbol, a symbol that is either defined outside this file or that is defined in this file but can be referenced by other files. 

      如果该位为开，则该符号为外部符号，即在此文件外部定义的，或在此文件中定义但可以被其他文件引用的。

      

    Values for the `N_TYPE` field include: 

    - `N_UNDF (0x0)`—The symbol is undefined. Undefined symbols are symbols referenced in this module but defined in a different module. The n_sect field is set to NO_SECT. 

      符号未定义。未定义符号是指在此模块中引用但在其他模块中定义的符号。n_sect字段设置为NO_SECT。

    - `N_ABS (0x2)`—The symbol is absolute. The linker does not change the value of an absolute symbol. The n_sect field is set to NO_SECT. 

      符号是绝对的。链接器不会更改绝对符号的值。n_sect字段设置为NO_SECT。

    - `N_SECT (0xe)`—The symbol is defined in the section number given in n_sect. 

    - `N_PBUD (0xc)`—The symbol is undefined and the image is using a prebound value for the symbol. The n_sect field is set to NO_SECT. 

    - `N_INDR ( 0xa)`—The symbol is defined to be the same as another symbol. The n_value field is an index into the string table specifying the name of the other symbol. When that symbol is linked, both this and the other symbol have the same defined type and value. 

      该符号被定义为与另一个符号相同。n_value字段是字符串表的索引，用于指定另一个符号的名称。链接该符号时，此符号和其他符号都具有相同的定义类型和值。

  - `n_sect`

    An integer specifying the number of the section that this symbol can be found in, or NO_SECT 

    if the symbol is not to be found in any section of this image. The sections are contiguously numbered across segments, starting from 1, according to the order they appear in the LC_SEGMENT load commands. 

    一个整数，指定可以在其中找到该符号的节的数目，或NO_SECT
    如果在图像的任何部分都找不到符号。根据它们在LC_SEGMENT load commands中出现的顺序，分段在分段之间连续编号，从1开始。

  - `n_desc`

    A 16-bit value providing additional information about the nature of this symbol for non-stab
    symbols. The reference flags can be accessed using the REFERENCE_TYPE mask (0xF) and are
    defined as follows:

    一个16位值，为非stab符号提供关于此符号性质的附加信息。可以使用REFERENCE_TYPE掩码(0xF)访问引用标志，定义如下:

  - `n_value`

    An integer that contains the value of the symbol. The format of this value is different for each type of symbol table entry (as specified by the n_type field). For the N_SECT symbol type, n_value is the address of the symbol. See the description of the n_type field for information on other possible values.

    包含符号值的整数。对于每种类型的符号表条目(由n_type字段指定)，此值的格式是不同的。对于N_SECT符号类型，n_value是符号的地址。有关其他可能值的信息，请参见n_type字段的描述。

  **Discussion** 

  Common symbols must be of type N_UNDF and must have the N_EXT bit set. The n_value for a common symbol is the size (in bytes) of the data of the symbol. In C, a common symbol is a variable that is declared but not initialized in this file. Common symbols can appear only in MH_OBJECT Mach-O files. 

  公共符号必须是N_UNDF类型的，并且必须设置N_EXT位。公共符号的n_value是符号数据的大小(以字节为单位)。在C语言中，公共符号是在该文件中声明但未初始化的变量。通用符号只能出现在MH_OBJECT Mach-O文件中。

  

- **dysymtab_command**

  The data structure for the `LC_DYSYMTAB` load command. It describes the sizes and locations of the parts of the symbol table used for dynamic linking. Declared in /usr/include/mach-o/loader.h.

  ```c
  struct dysymtab_command {
      uint32_t cmd;	/* LC_DYSYMTAB */
      uint32_t cmdsize;	/* sizeof(struct dysymtab_command) */
  
      /*
       * The symbols indicated by symoff and nsyms of the LC_SYMTAB load command
       * are grouped into the following three groups:
       *    local symbols (further grouped by the module they are from)
       *    defined external symbols (further grouped by the module they are from)
       *    undefined symbols
       *
       * The local symbols are used only for debugging.  The dynamic binding
       * process may have to use them to indicate to the debugger the local
       * symbols for a module that is being bound.
       *
       * The last two groups are used by the dynamic binding process to do the
       * binding (indirectly through the module table and the reference symbol
       * table when this is a dynamically linked shared library file).
       */
      uint32_t ilocalsym;	/* index to local symbols */
      uint32_t nlocalsym;	/* number of local symbols */
  
      uint32_t iextdefsym;/* index to externally defined symbols */
      uint32_t nextdefsym;/* number of externally defined symbols */
  
      uint32_t iundefsym;	/* index to undefined symbols */
      uint32_t nundefsym;	/* number of undefined symbols */
  
      /*
       * For the for the dynamic binding process to find which module a symbol
       * is defined in the table of contents is used (analogous to the ranlib
       * structure in an archive) which maps defined external symbols to modules
       * they are defined in.  This exists only in a dynamically linked shared
       * library file.  For executable and object modules the defined external
       * symbols are sorted by name and is use as the table of contents.
       */
      uint32_t tocoff;	/* file offset to table of contents */
      uint32_t ntoc;	/* number of entries in table of contents */
  
      /*
       * To support dynamic binding of "modules" (whole object files) the symbol
       * table must reflect the modules that the file was created from.  This is
       * done by having a module table that has indexes and counts into the merged
       * tables for each module.  The module structure that these two entries
       * refer to is described below.  This exists only in a dynamically linked
       * shared library file.  For executable and object modules the file only
       * contains one module so everything in the file belongs to the module.
       */
      uint32_t modtaboff;	/* file offset to module table */
      uint32_t nmodtab;	/* number of module table entries */
  
      /*
       * To support dynamic module binding the module structure for each module
       * indicates the external references (defined and undefined) each module
       * makes.  For each module there is an offset and a count into the
       * reference symbol table for the symbols that the module references.
       * This exists only in a dynamically linked shared library file.  For
       * executable and object modules the defined external symbols and the
       * undefined external symbols indicates the external references.
       */
      uint32_t extrefsymoff;	/* offset to referenced symbol table */
      uint32_t nextrefsyms;	/* number of referenced symbol table entries */
  
      /*
       * The sections that contain "symbol pointers" and "routine stubs" have
       * indexes and (implied counts based on the size of the section and fixed
       * size of the entry) into the "indirect symbol" table for each pointer
       * and stub.  For every section of these two types the index into the
       * indirect symbol table is stored in the section header in the field
       * reserved1.  An indirect symbol table entry is simply a 32bit index into
       * the symbol table to the symbol that the pointer or stub is referring to.
       * The indirect symbol table is ordered to match the entries in the section.
       */
      uint32_t indirectsymoff; /* file offset to the indirect symbol table */
      uint32_t nindirectsyms;  /* number of indirect symbol table entries */
  
      /*
       * To support relocating an individual module in a library file quickly the
       * external relocation entries for each module in the library need to be
       * accessed efficiently.  Since the relocation entries can't be accessed
       * through the section headers for a library file they are separated into
       * groups of local and external entries further grouped by module.  In this
       * case the presents of this load command who's extreloff, nextrel,
       * locreloff and nlocrel fields are non-zero indicates that the relocation
       * entries of non-merged sections are not referenced through the section
       * structures (and the reloff and nreloc fields in the section headers are
       * set to zero).
       *
       * Since the relocation entries are not accessed through the section headers
       * this requires the r_address field to be something other than a section
       * offset to identify the item to be relocated.  In this case r_address is
       * set to the offset from the vmaddr of the first LC_SEGMENT command.
       * For MH_SPLIT_SEGS images r_address is set to the the offset from the
       * vmaddr of the first read-write LC_SEGMENT command.
       *
       * The relocation entries are grouped by module and the module table
       * entries have indexes and counts into them for the group of external
       * relocation entries for that the module.
       *
       * For sections that are merged across modules there must not be any
       * remaining external relocation entries for them (for merged sections
       * remaining relocation entries must be local).
       */
      uint32_t extreloff;	/* offset to external relocation entries */
      uint32_t nextrel;	/* number of external relocation entries */
  
      /*
       * All the local relocation entries are grouped together (they are not
       * grouped by their module since they are only used if the object is moved
       * from it staticly link edited address).
       */
      uint32_t locreloff;	/* offset to local relocation entries */
      uint32_t nlocrel;	/* number of local relocation entries */
  
  };	
  ```

  **Fields**

  - `cmd`

    Common to all load command structures. For this structure, set to LC_DYSYMTAB.

  - `cmdsize`

    Common to all load command structures. For this structure, set to sizeof(dysymtab_command).

  - `ilocalsym`

    An integer indicating the index of the first symbol in the group of local symbols.

    一个整数，表示本地符号组中第一个符号的索引

  - `nlocalsym`

    An integer indicating the total number of symbols in the group of local symbols.

    表示本地符号组中符号总数

  - `iextdefsym`

    An integer indicating the index of the first symbol in the group of defined external symbols.

    一个整数，表示定义的外部符号组中第一个符号的索引。

  - `nextdefsym`

    An integer indicating the total number of symbols in the group of defined external symbols.

    一个整数，表示定义的外部符号组中的符号总数。

  - `iundefsym`

    An integer indicating the index of the first symbol in the group of undefined external symbols.

    一个整数，表示未定义外部符号组中第一个符号的索引。

  - `nundefsym`

    An integer indicating the total number of symbols in the group of undefined external symbols

    表示未定义外部符号组中符号总数

  - `tocoff`

    An integer indicating the byte offset from the start of the file to the table of contents data.

    一个整数，表示从文件开始到目录数据的字节偏移量。

  - `ntoc`

    An integer indicating the number of entries in the table of contents.

    表示目录中条目数的整数。

  - `modtaboff`

    An integer indicating the byte offset from the start of the file to the module table data.

    一个整数，表示从文件开始到模块表数据的字节偏移量。

  - `nmodtab`

    An integer indicating the number of entries in the module table.

    表示模块表中条目数的整数。

  - `extrefsymoff`

    An integer indicating the byte offset from the start of the file to the external reference table
    data.

    一个整数，表示从文件开始到外部引用表数据的字节偏移量。

  - `nextrefsyms`

    An integer indicating the number of entries in the external reference table.

    表示外部引用表中条目数的整数。

  - `indirectsymoff`

    An integer indicating the byte offset from the start of the file to the indirect symbol table data.

    一个整数，表示从文件开头到间接符号表数据的字节偏移量。

  - `nindirectsyms`

    An integer indicating the number of entries in the indirect symbol table.

    指示间接符号表中条目数的整数。

  - `extreloff`

    An integer indicating the byte offset from the start of the file to the external relocation table data.

    一个整数，表示从文件开始到外部重定位表数据的字节偏移量

  - `nextrel`

    An integer indicating the number of entries in the external relocation table.

    一个整数，指示外部重新定位表中的条目数

  - `locreloff`

    An integer indicating the byte offset from the start of the file to the local relocation table data.

    一个整数，表示从文件开始到本地重定位表数据的字节偏移量

  - `nlocrel`

    An integer indicating the number of entries in the local relocation table.

    一个整数，指示本地重新定位表中的条目数

  **Discussion** 

  The LC_DYSYMTAB load command contains a set of indexes into the symbol table and a set of file offsets that define the location of several other tables. Fields for tables not used in the file should be set to 0. These tables are described in “Dynamic Code Generation” in *Mach-O Programming Topics*. 
  LC_DYSYMTAB load command包含符号表中的一组索引和一组文件偏移量，它们定义了其他几个表的位置。文件中未使用的表的字段应设置为0。这些表在*Mach-O Programming Topics*.中的“Dynamic Code Generation”中进行了描述。

  

- **dylib_table_of_contents**

  Describes an entry in the table of contents of a dynamic shared library. Declared in
  /usr/include/mach-o/loader.h.

  ```c
  /* a table of contents entry */
  struct dylib_table_of_contents {
      uint32_t symbol_index;	/* the defined external symbol
  				   (index into the symbol table) */
      uint32_t module_index;	/* index into the module table this symbol
  				   is defined in */
  };	
  ```

  **Fields**

  - `symbol_index`

    An index into the symbol table indicating the defined external symbol to which this entry
    refers.

    符号表中的索引，指示此项所引用的已定义外部符号。

  - `module_index`

    An index into the module table indicating the module in which this defined external symbol
    is defined.

    模块表中的索引，指示在其中定义此定义的外部符号的模块。

    

- **dylib_module**

  Describes a module table entry for a dynamic shared library for 32-bit architectures. Declared in
  /usr/include/mach-o/loader.h. 

  ```c
  /* a module table entry */
  struct dylib_module {
      uint32_t module_name;	/* the module name (index into string table) */
  
      uint32_t iextdefsym;	/* index into externally defined symbols */
      uint32_t nextdefsym;	/* number of externally defined symbols */
      uint32_t irefsym;		/* index into reference symbol table */
      uint32_t nrefsym;		/* number of reference symbol table entries */
      uint32_t ilocalsym;		/* index into symbols for local symbols */
      uint32_t nlocalsym;		/* number of local symbols */
  
      uint32_t iextrel;		/* index into external relocation entries */
      uint32_t nextrel;		/* number of external relocation entries */
  
      uint32_t iinit_iterm;	/* low 16 bits are the index into the init
  				   section, high 16 bits are the index into
  			           the term section */
      uint32_t ninit_nterm;	/* low 16 bits are the number of init section
  				   entries, high 16 bits are the number of
  				   term section entries */
  
      uint32_t			/* for this module address of the start of */
  	objc_module_info_addr;  /*  the (__OBJC,__module_info) section */
      uint32_t			/* for this module size of */
  	objc_module_info_size;	/*  the (__OBJC,__module_info) section */
  };	
  
  /* a 64-bit module table entry */
  struct dylib_module_64 {
      uint32_t module_name;	/* the module name (index into string table) */
  
      uint32_t iextdefsym;	/* index into externally defined symbols */
      uint32_t nextdefsym;	/* number of externally defined symbols */
      uint32_t irefsym;		/* index into reference symbol table */
      uint32_t nrefsym;		/* number of reference symbol table entries */
      uint32_t ilocalsym;		/* index into symbols for local symbols */
      uint32_t nlocalsym;		/* number of local symbols */
  
      uint32_t iextrel;		/* index into external relocation entries */
      uint32_t nextrel;		/* number of external relocation entries */
  
      uint32_t iinit_iterm;	/* low 16 bits are the index into the init
  				   section, high 16 bits are the index into
  				   the term section */
      uint32_t ninit_nterm;      /* low 16 bits are the number of init section
  				  entries, high 16 bits are the number of
  				  term section entries */
  
      uint32_t			/* for this module size of */
          objc_module_info_size;	/*  the (__OBJC,__module_info) section */
      uint64_t			/* for this module address of the start of */
          objc_module_info_addr;	/*  the (__OBJC,__module_info) section */
  };
  ```

  **Fields**

  - `module_name`

    An index to an entry in the string table indicating the name of the module.

    指向字符串表中指示模块名称的项的索引。

  - `iextdefsym`

    The index into the symbol table of the first defined external symbol provided by this module.

    此模块提供的第一个定义的外部符号的符号表索引。

  - `nextdefsym`

    The number of defined external symbols provided by this module.

    此模块提供的已定义外部符号的数目

  - `irefsym`

    The index into the external reference table of the first entry provided by this module.

    此模块提供的第一个条目的外部引用表的索引

  - `nrefsym`

    The number of external reference entries provided by this module.

    此模块提供的外部引用条目数。

  - `ilocalsym`

    The index into the symbol table of the first local symbol provided by this module.

    此模块提供的第一个本地符号的符号表索引。

  - `nlocalsym`

    The number of local symbols provided by this module.

    此模块提供的本地符号数。

  - `iextrel`

    The index into the external relocation table of the first entry provided by this module.

    此模块提供的第一个条目的外部重新定位表的索引

  - `nextrel`

    The number of entries in the external relocation table that are provided by this module.

    此模块提供的外部重新定位表中的条目数。

  - `iinit_iterm`

    Contains both the index into the module initialization section (the low 16 bits) and the index into the module termination section (the high 16 bits) to the pointers for this module.

    包含指向模块初始化部分（低16位）的索引和指向此模块指针的模块终止部分（高16位）的索引。

  - `ninit_nterm`

    Contains both the number of pointers in the module initialization (the low 16 bits) and the
    number of pointers in the module termination section (the high 16 bits) for this module.

    包含模块初始化中的指针数（低16位）和此模块的模块终止部分中的指针数（高16位）。

  - `objc_module_info_addr`

    The statically linked address of the start of the data for this module in the __module_info section in the __OBJC segment.

    在objc段中的模块信息部分中，此模块的数据起始的静态链接地址。

  - `objc_module_info_size`

    The number of bytes of data for this module that are used in the __module_info section in the __OBJC segment.

    此模块在objc段中的模块信息部分中使用的数据字节数。

    

- **dylib_reference**

  Defines the attributes of an external reference table entry for the external reference entries provided by a module in a shared library. Declared in /usr/include/mach-o/loader.h.

  ```c
  /* 
   * The entries in the reference symbol table are used when loading the module
   * (both by the static and dynamic link editors) and if the module is unloaded
   * or replaced.  Therefore all external symbols (defined and undefined) are
   * listed in the module's reference table.  The flags describe the type of
   * reference that is being made.  The constants for the flags are defined in
   * <mach-o/nlist.h> as they are also used for symbol table entries.
   */
  struct dylib_reference {
      uint32_t isym:24,		/* index into the symbol table */
      		  flags:8;	/* flags to indicate the type of reference */
  };
  ```

  **Fields**

  - `isym`

    An index into the symbol table for the symbol being referenced

    符号表中被引用符号的索引

  - `flags`

    A constant for the type of reference being made. Use the same REFERENCE_FLAG constants as described in the nlist (page 39) structure description.

    引用类型的常量。使用与nlist(第39页)结构描述中描述的相同的REFERENCE_FLAG常量。



## Relocation Data Structures

**Relocation** is the process of moving symbols to a different address. When the static linker moves a
symbol (a function or an item of data) to a different address, it needs to change all the references to
that symbol to use the new address. The **relocation entries** in a Mach-O file contain offsets in the file
to addresses that need to be relocated when the contents of the file are relocated. The addresses stored
in CPU instructions can be absolute or relative. Each relocation entry specifies the exact format of the
address. When creating the intermediate object file, the compiler generates one or more relocation
entries for every instruction that contains an address. Because relocation to symbols at fixed addresses,
and to relative addresses for position independent references, does not occur at runtime, the static
linker typically removes some or all the relocation entries when building the final product.

重定位是将符号移动到另一个地址的过程。当静态链接器将符号(函数或数据项)移动到另一个地址时，需要更改对该符号的所有引用，以使用新地址。Mach-O文件中的重定位条目包含文件中的偏移量，这些偏移量指向当文件内容重定位时需要重定位的地址。CPU指令中存储的地址可以是绝对地址，也可以是相对地址。每个重定位条目指定地址的确切格式。在创建中间对象文件时，编译器为包含地址的每条指令生成一个或多个重定位项。由于在运行时不会对固定地址的符号和位置独立引用的相对地址进行重定位，因此静态链接器通常会在构建最终产品时删除部分或所有重定位项。

> Note: In the Mac OS X x86-64 environment scattered relocations are not used. Compiler-generated code uses mostly external relocations, in which the r_extern bit is set to 1 and the r_symbolnum field contains the symbol-table index of the target label.
>
> 注意:在Mac OS X x86-64环境中不使用分散重定位。编译器生成的代码主要使用外部重定位，其中r_extern位设置为1,r_symbolnum字段包含目标标签的符号表索引。



- **relocation_info**

  Describes an item in the file that uses an address that needs to be updated when the address is changed.
  Declared in /usr/include/mach-o/reloc.h.

  描述文件中使用地址的项，该地址在地址更改时需要更新。在/usr/include/mach-o/reloc.h中声明。

  ```c
  /*
   * Format of a relocation entry of a Mach-O file.  Modified from the 4.3BSD
   * format.  The modifications from the original format were changing the value
   * of the r_symbolnum field for "local" (r_extern == 0) relocation entries.
   * This modification is required to support symbols in an arbitrary number of
   * sections not just the three sections (text, data and bss) in a 4.3BSD file.
   * Also the last 4 bits have had the r_type tag added to them.
   */
  struct relocation_info {
     int32_t	r_address;	/* offset in the section to what is being
  				   relocated */
     uint32_t     r_symbolnum:24,	/* symbol index if r_extern == 1 or section
  				   ordinal if r_extern == 0 */
  		r_pcrel:1, 	/* was relocated pc relative already */
  		r_length:2,	/* 0=byte, 1=word, 2=long, 3=quad */
  		r_extern:1,	/* does not include value of sym referenced */
  		r_type:4;	/* if not 0, machine specific relocation type */
  };
  ```

  **Fields**

  - `r_address`

    In MH_OBJECT files, this is an offset from the start of the section to the item containing the address requiring relocation. If the high bit of this field is set (which you can check using the R_SCATTERED bit mask), the relocation_info structure is actually a scattered_relocation_info (page 52) structure. 

    In images used by the dynamic linker, this is an offset from the virtual memory address of the data of the first segment_command (page 20) that appears in the file (not necessarily the one with the lowest address). For images with the MH_SPLIT_SEGS flag set, this is an offset from the virtual memory address of data of the first read/write segment_command (page 20). 

    在MH_OBJECT文件中，这是从节的开始到包含需要重新定位的地址的项的偏移量。如果设置了该字段的高位(可以使用r_scatter位掩码检查)，则relocation_info结构实际上是scattered_relocation_info(第52页)结构。
    在动态链接器使用的图像中，这是与文件中出现的第一个segment_command(第20页)的数据的虚拟内存地址的偏移量(不一定是地址最低的那个)。对于设置了MH_SPLIT_SEGS标志的图像，这是第一个读/写段_command(第20页)数据的虚拟内存地址的偏移量。

  - `r_symbolnum`

    Indicates either an index into the symbol table (when the r_extern field is set to 1) or a section
    number (when the r_extern field is set to 0). As previously mentioned, sections are ordered
    from 1 to 255 in the order in which they appear in the LC_SEGMENT load commands. This field
    is set to R_ABS for relocation entries for absolute symbols, which need no relocation.

    表示符号表中的索引(当r_extern字段设置为1时)或节号(当r_extern字段设置为0时)。这个字段被设置为R_ABS，用于绝对符号的重新定位条目，绝对符号不需要重新定位。

  - `r_pcrel`

    Indicates whether the item containing the address to be relocated is part of a CPU instruction that uses PC-relative addressing. 

    For addresses contained in PC-relative instructions, the CPU adds the address of the instruction to the address contained in the instruction. 

    指示包含要重新定位的地址的项是否属于使用pc相对寻址的CPU指令的一部分。
    对于pc相关指令中包含的地址，CPU将指令的地址添加到指令中包含的地址中。

  - `r_length`

    Indicates the length of the item containing the address to be relocated. The following table lists
    r_length values and the corresponding address length.

    指示包含要重新定位的地址的项的长度。下表列出了r_length值和相应的地址长度。

    | Value | Address length                                               |
    | ----- | ------------------------------------------------------------ |
    | 0     | 1 byte                                                       |
    | 1     | 2 bytes                                                      |
    | 2     | 4 bytes                                                      |
    | 3     | 4 bytes. See description for the PPC_RELOC_BR14 r_type in<br/>scattered_relocation_info (page 52). |

  - `r_extern`

    Indicates whether the r_symbolnum field is an index into the symbol table (1) or a section
    number (0).

    指示r_symbolnum字段是符号表(1)的索引还是节号(0)的索引。

  - `r_type`

    

- **scattered_relocation_info** 

  Describes an item in the file—using a nonzero constant in its relocatable expression or two addresses in its relocatable expression—that needs to be updated if the addresses that it uses are changed. This information is needed to reconstruct the addresses that make up the relocatable expression’s value in order to change the addresses independently of each other. Declared in /usr/include/mach-o/reloc.h. 

  在可重定位表达式中使用非零常量或在可重定位表达式中使用两个地址来描述文件中的项——如果使用的地址发生更改，则需要更新这些常量。需要此信息来重构构成可重定位表达式值的地址，以便独立地更改地址。中声明/usr/include/mach-o/reloc.h.

  ```c
  struct scattered_relocation_info {
  #ifdef __BIG_ENDIAN__
     uint32_t	r_scattered:1,	/* 1=scattered, 0=non-scattered (see above) */
  		r_pcrel:1, 	/* was relocated pc relative already */
  		r_length:2,	/* 0=byte, 1=word, 2=long, 3=quad */
  		r_type:4,	/* if not 0, machine specific relocation type */
     		r_address:24;	/* offset in the section to what is being
  				   relocated */
     int32_t	r_value;	/* the value the item to be relocated is
  				   refering to (without any offset added) */
  #endif /* __BIG_ENDIAN__ */
  #ifdef __LITTLE_ENDIAN__
     uint32_t
     		r_address:24,	/* offset in the section to what is being
  				   relocated */
  		r_type:4,	/* if not 0, machine specific relocation type */
  		r_length:2,	/* 0=byte, 1=word, 2=long, 3=quad */
  		r_pcrel:1, 	/* was relocated pc relative already */
  		r_scattered:1;	/* 1=scattered, 0=non-scattered (see above) */
     int32_t	r_value;	/* the value the item to be relocated is
  				   refering to (without any offset added) */
  #endif /* __LITTLE_ENDIAN__ */
  };
  ```

  **Fields**

  - `r_scattered`

    If this bit is 0, this structure is actually a relocation_info (page 49) structure.

    如果这个位是0，那么这个结构实际上是一个relocation_info(第49页)结构。

  - `r_address`

    In MH_OBJECT files, this is an offset from the start of the section to the item containing the address requiring relocation. If the high bit of this field is clear (which you can check using the R_SCATTERED bit mask), this structure is actually a relocation_info (page 49) structure. 

    In images used by the dynamic linker, this is an offset from the virtual memory address of the data of the first segment_command (page 20) that appears in the file (not necessarily the one with the lowest address). For images with the MH_SPLIT_SEGS flag set, this is an offset from the virtual memory address of data of the first read/write segment_command (page 20). 

    Since this field is only 24 bits long, the offset in this field can never be larger than 0x00FFFFFF, thus limiting the size of the relocatable contents of this image to 16 megabytes. 

    在MH_OBJECT文件中，这是从节的开始到包含需要重新定位的地址的项的偏移量。如果这个字段的高位是清除的(可以使用r_scatter位掩码检查)，那么这个结构实际上是一个relocation_info(第49页)结构。
    在动态链接器使用的图像中，这是与文件中出现的第一个segment_command(第20页)的数据的虚拟内存地址的偏移量(不一定是地址最低的那个)。对于设置了MH_SPLIT_SEGS标志的图像，这是第一个读/写段_command(第20页)数据的虚拟内存地址的偏移量。
    由于该字段只有24位长，因此该字段中的偏移量永远不能大于0x00FFFFFF，从而将此图像的可重定位内容的大小限制为16 mb。

  - `r_pcrel`

    Indicates whether the item containing the address to be relocated is part of a CPU instruction that uses PC-relative addressing. 

    For addresses contained in PC-relative instructions, the CPU adds the address of the instruction to the address contained in the instruction. 

    指示包含要重新定位的地址的项是否属于使用pc相对寻址的CPU指令的一部分。
    对于pc相关指令中包含的地址，CPU将指令的地址添加到指令中包含的地址中。

  - `r_length`

    Indicates the length of the item containing the address to be relocated. A value of 0 indicates
    a single byte; a value of 1 indicates a 2-byte address, and a value of 2 indicates a 4-byte address.

    指示包含要重新定位的地址的项的长度。值为0表示单个字节;值1表示2字节地址，值2表示4字节地址。

  - `r_type`

    Indicates the type of relocation to be performed. Possible values for this field are shared between
    this structure and the relocation_info data structure; see the description of the r_type field
    in the relocation_info (page 49) data structure for more details.

    指示要执行的重定位类型。这个字段的可能值在这个结构和relocation_info数据结构之间共享;有关详细信息，请参阅relocation_info(第49页)数据结构中r_type字段的描述。

  - `r_value`

    The address of the relocatable expression for the item in the file that needs to be updated if the
    address is changed. For relocatable expressions with the difference of two section addresses,
    the address from which to subtract (in mathematical terms, the minuend) is contained in the
    first relocation entry and the address to subtract (the subtrahend) is contained in the second
    relocation entry.

    如果地址更改，则需要更新文件中项的可重定位表达式的地址。对于两个节地址不同的可重定位表达式，要减去的地址(数学术语为被减数)包含在第一个重定位项中，要减去的地址(减数)包含在第二个重定位项中。

  **Discussion** 

  Mach-O relocation data structures support two types of relocatable expressions in machine code and data: 

  Mach-O重定位数据结构支持两种类型的可重定位表达式:

  - **Symbol address + constant**. The most typical form of relocation is referencing a symbol’s address with no constant added. In this case, the value of the constant expression is 0. 

    最典型的重定位形式是引用没有添加常量的符号地址。在本例中，常量表达式的值为0。

  - **Address of section y** **–** **address of section x + constant**. The section difference form of relocation. This form of relocation supports position-independent code. 

    节差形式的移位。这种形式的重新定位支持与位置无关的代码。



## Static Archive Libraries

This section describes the file format used for static archive libraries. Mac OS X uses a format derived from the original BSD static archive library format, with a few minor additions. See the discussion for the ranlib data structure for more information. 

本节描述用于静态存档库的文件格式。Mac OS X使用的格式是从原始的BSD静态存档库格式派生出来的，只添加了一些小功能。有关更多信息，请参阅ranlib数据结构的讨论。



- **ranlib** 

  Defines the attributes of a static archive library symbol table entry. Declared in /usr/include/mach-o/ranlib.h. 

  ```c
  /*
   * Structure of the __.SYMDEF table of contents for an archive.
   * __.SYMDEF begins with a uint32_t giving the size in bytes of the ranlib
   * structures which immediately follow, and then continues with a string
   * table consisting of a uint32_t giving the number of bytes of strings which
   * follow and then the strings themselves.  The ran_strx fields index the
   * string table whose first byte is numbered 0.
   */
  struct	ranlib {
      union {
  	uint32_t	ran_strx;	/* string table index of */
  #ifndef __LP64__
  	char		*ran_name;	/* symbol defined by */
  #endif
      } ran_un;
      uint32_t		ran_off;	/* library member at this offset */
  };
  ```

  **Fields** 

  - `ran_strx`

    The index number (zero-based) of the string in the string table that follows the array of ranlib data structures. 

    在ranlib数据结构数组后面的字符串表中字符串的索引号(从零开始)。

  - `ran_name`

    The byte offset, from the start of the file, at which the symbol name can be found. This field is not used in Mach-O files. 

    从文件开始的字节偏移量，在该偏移量处可以找到符号名。此字段在Mach-O文件中不使用。

  - `ran_off`

    The byte offset, from the start of the file, at which the header line for the member containing this symbol can be found. 

    从文件开始的字节偏移量，在此位置可以找到包含此符号的成员的头行。

    

  **Discussion** 

  A static archive library begins with the file identifier string !<arch>, followed by a newline character (ASCII value 0x0A). The file identifier string is followed by a series of member files. Each member consists of a fixed-length header line followed by the file data. The header line is 60 bytes long and is divided into five fixed-length fields, as shown in this example header line: 

  静态存档库以文件标识符字符串!<arch>开始，后跟一个换行符(ASCII值0x0A)。文件标识符字符串后面跟着一系列成员文件。每个成员由一个固定长度的头行和文件数据组成。头行为60字节长，分为5个固定长度的字段，如下例头行所示:

  ```shell
            grapple.c       999514211   501   20    100644  167       `
  ```

  The last 2 bytes of the header line are a grave accent (`) character (ASCII value 0x60) and a newline
  character. All header fields are defined in ASCII and padded with spaces to the full length of the field.
  All fields are defined in decimal notation, except for the file mode field, which is defined in octal.
  These are the descriptions for each field:

  头行最后两个字节是一个重重音(')字符(ASCII值0x60)和一个换行字符。所有头字段都是用ASCII定义的，并用空格填充到字段的完整长度。所有字段都是用十进制记数法定义的，文件模式字段除外，它是用八进制定义的。以下是每个领域的描述:

  - The name field (16 bytes) contains the name of the file. If the name is either longer than 16 bytes or contains a space character, the actual name should be written directly after the header line and the name field should contain the string #1/ followed by the length. To keep the archive entries aligned to 8 byte boundaries, the length of the name that follows the #1/ is rounded to 8 bytes and the name that follows the header is padded with null bytes. 

    name字段(16字节)包含文件的名称。如果名称大于16字节或包含空格字符，则实际名称应该直接写在标题行之后，name字段应该包含字符串#1/后跟长度。为了使归档条目对齐到8个字节的边界，#1/后面的名称的长度四舍五入为8个字节，头部后面的名称用空字节填充。

  - The modified date field (12 bytes) is taken from the st_time field returned by the stat system call. 

    修改后的日期字段(12字节)取自stat系统调用返回的st_time字段。

  - The user ID field (6 bytes) is taken from the st_uid field returned by the stat system call. 

    用户ID字段(6字节)取自stat系统调用返回的st_uid字段。

  - The group ID field (6 bytes) is taken from the st_gid field returned by the stat system call. 

    组ID字段(6字节)取自stat系统调用返回的st_gid字段。

  - The file mode field (8 bytes) is taken from the st_mode field returned by the stat system call. This field is written in octal notation. 

    文件模式字段(8字节)取自stat系统调用返回的st_mode字段。这个字段用八进制符号表示。

  - The file size field (8 bytes) is taken from the st_size field returned by the stat system call. 

    文件大小字段(8字节)取自stat系统调用返回的st_size字段。

  The first member in a static archive library is always the symbol table describing the contents of the rest of the member files. This member is always called either __.SYMDEF or __.SYMDEF SORTED (note the two leading underscores and the period). The name used depends on the sort order of the symbol table. The older variant—__.SYMDEF—contains entries in the same order that they appear in the object files. The newer variant—__.SYMDEF SORTED— contains entries in alphabetical order, which allows the static linker to load the symbols faster. 

  静态存档库中的第一个成员总是描述其余成员文件内容的符号表。这个成员总是被称为__。SYMDEF或__。SYMDEF排序(注意两个前导下划线和句号)。使用的名称取决于符号表的排序顺序。__年长的变体。symdef -包含与它们在目标文件中出现的顺序相同的条目。__新的变体。按字母顺序包含条目，这允许静态链接器更快地加载符号。

  

  The __.SYMDEF and .__SORTED SYMDEF archive members contain an array of ranlib data structures preceded by the length in bytes (a long integer, 4 bytes) of the number of items in the array. The array is followed by a string table of null-terminated strings, which are preceded by the length in bytes of the entire string table (again, a 4-byte long integer). 

  __。已排序的SYMDEF archive成员包含一个ranlib数据结构数组，前面是数组中项数的字节长度(一个长整数，4个字节)。数组后面是一个以null结尾的字符串字符串表，它的前面是整个字符串表的字节长度(同样是一个4字节长的整数)。

  The string table is an array of C strings, each terminated by a null byte.
   The ranlib declarations can be found in /usr/include/mach-o/ranlib.h. 

  **Special Considerations** 

  Prior to the advent of libtool, a tool called ranlib was used to generate the symbol table. ranlib has since been integrated into libtool. See the man page for libtool for more information. 

  在libtool出现之前，使用了一个名为ranlib的工具来生成符号表。ranlib已经集成到libtool中。有关更多信息，请参见libtool的手册页。



## Universal Binaries 32-bit/64-bit PowerPC Binaries

The standard development tools accept as parameters two kinds of binaries: 

标准开发工具接受两种二进制文件作为参数:

- Object files targeted at one architecture. These include Mach-O files, static libraries, and dynamic libraries. 

  目标文件针对一个体系结构。其中包括Mach-O文件、静态库和动态库。

- Binaries targeted at more than one architecture. These binaries contain compiled code and data for one of these system types: 

  针对多个体系结构的二进制文件。这些二进制文件包含以下系统类型的编译代码和数据:

  - PowerPC-based (32-bit and 64-bit) Macintosh computers. Binaries that contain code for both
    32-bit and 64-bit PowerPC-based Macintosh computers are are known as **PPC/PPC64 binaries**.

    基于powerpc(32位和64位)的Macintosh计算机。包含两者代码的二进制文件
    基于32位和64位powerpc的Macintosh计算机被称为**PPC/PPC64二进制文件**。

  - Intel-based and PowerPC-based (32-bit, 64-bit, or both) Macintosh computers. Binaries that
    contain code for both Intel-based and PowerPC-based Macintosh computers are known as
    **universal binaries**.

    基于intel和基于powerpc(32位、64位或两者都有)的Macintosh计算机。二进制文件
    包含基于intel和基于powerpc的Macintosh计算机的代码 通用二进制文件

  Each object file is stored as a continuous set of bytes at an offset from the beginning of the binary.
  They use a simple archive format to store the two object files with a special header at the beginning
  of the file to allow the various runtime tools to quickly find the code appropriate for the current
  architecture.

  每个目标文件都存储为一组连续的字节，以二进制文件开头的偏移量为单位。它们使用一种简单的归档格式来存储这两个目标文件，并在文件的开头使用一个特殊的头，以允许各种运行时工具快速找到适合当前体系结构的代码。

A binary that contains code for more than one architecture always begins with a fat_header (page
56) data structure, followed by two fat_arch (page 56) data structures and the actual data for the
architectures contained in the file. All data in these data structures is stored in big-endian byte order.

包含多个体系结构代码的二进制文件总是以fat_header(第56页)数据结构开始，然后是两个fat_arch(第56页)数据结构和文件中包含的体系结构的实际数据。这些数据结构中的所有数据都以大端字节顺序存储。



- **fat_header**

  Defines the layout of a binary that contains code for more than one architecture. Declared in the
  header /usr/include/mach-o/fat.h.

  ```c
  #define FAT_MAGIC	0xcafebabe
  #define FAT_CIGAM	0xbebafeca	/* NXSwapLong(FAT_MAGIC) */
  
  struct fat_header {
  	uint32_t	magic;		/* FAT_MAGIC or FAT_MAGIC_64 */
  	uint32_t	nfat_arch;	/* number of structs that follow */
  };
  ```

  **Fields**

  - `magic`

    An integer containing the value 0xCAFEBABE in big-endian byte order format. On a big-endian
    host CPU, this can be validated using the constant FAT_MAGIC; on a little-endian host CPU, it
    can be validated using the constant FAT_CIGAM.

    包含值0xCAFEBABE的整数，采用大端字节顺序格式。在大端主机CPU上，可以使用常量FAT_MAGIC验证这一点;在little-endian主机CPU上，可以使用常量FAT_CIGAM验证它。

  - `nfat_arch`

    An integer specifying the number of fat_arch (page 56) data structures that follow. This is
    the number of architectures contained in this binary.

    一个整数，指定后面的fat_arch(第56页)数据结构的数量。这是这个二进制文件中包含的体系结构的数量。

    

  **Discussion** 

  The fat_header data structure is placed at the start of a binary that contains code for multiple architectures. Directly following the fat_header data structure is a set of fat_arch (page 56) data structures, one for each architecture included in the binary. 

  Regardless of the content this data structure describes, all its fields are stored in big-endian byte order. 

  fat_header数据结构位于包含多个体系结构代码的二进制文件的开头。直接跟随fat_header数据结构的是一组fat_arch(第56页)数据结构，每个结构都包含在二进制文件中。
  不管这个数据结构描述的内容是什么，它的所有字段都以大端字节顺序存储。

  

- **fat_arch**

  Describes the location within the binary of an object file targeted at a single architecture. Declared in /usr/include/mach-o/fat.h.

  描述以单一架构为目标的目标文件二进制文件中的位置。

  ```c
  struct fat_arch {
  	cpu_type_t	cputype;	/* cpu specifier (int) */
  	cpu_subtype_t	cpusubtype;	/* machine specifier (int) */
  	uint32_t	offset;		/* file offset to this object file */
  	uint32_t	size;		/* size of this object file */
  	uint32_t	align;		/* alignment as a power of 2 */
  };
  ```

  **Fields**

  - `cputype`

    An enumeration value of type cpu_type_t. Specifies the CPU family.

    类型cpu_type_t的枚举值。指定CPU族

  - `cpusubtype`

    An enumeration value of type cpu_subtype_t. Specifies the specific member of the CPU family
    on which this entry may be used or a constant specifying all members.

    类型cpu_subtype_t的枚举值。指定可以使用此条目的CPU家族的特定成员，或指定所有成员的常量。

  - `offset`

    Offset to the beginning of the data for this CPU.

    偏移到此CPU的数据开头。

  - `size`

    Size of the data for this CPU.

    这个CPU的数据大小。

  - `align`

    The power of 2 alignment for the offset of the object file for the architecture specified in cputype 

    within the binary. This is required to ensure that, if this binary is changed, the contents it retains are correctly aligned for virtual memory paging and other uses. 

    对cputype中指定的体系结构的目标文件的偏移量进行2对齐的能力
    在二进制。这是为了确保，如果这个二进制文件被更改，它保留的内容被正确对齐，以用于虚拟内存分页和其他用途。

    

  **Discussion** 

  An array of fat_arch data structures appears directly after the fat_header (page 56) data structure of a binary that contains object files for multiple architectures. 

  Regardless of the content this data structure describes, all its fields are stored in big-endian byte order. 

  fat_arch数据结构数组直接出现在包含多个体系结构目标文件的二进制文件的fat_header(第56页)数据结构之后。
  不管这个数据结构描述的内容是什么，它的所有字段都以大端字节顺序存储。

  