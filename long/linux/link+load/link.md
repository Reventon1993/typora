# 链接器比编译器更早

在早期编程人们使用机器语言在条带上编程时就面临一个问题：每次程序加载到内存中的位置是不一样的。所以需要手动对代码中的跳转指令中的地址等进行重新计算。这个过程就叫做重定位(Relocation)。

后来出现了汇编语言，可以使用符号来表示目标地址，比如jmp target。符号(Symbol)的概念正是来源于此。随着汇编语言的出现，程序的规模变得越来越大，一个程序被分成了多个模块。多个模块的相互联系的过程叫做链接(linking)。

链接器(linker)实现了链接的过程，其中包括了重定位的过程。

链接器的主要工作为：重定位(relocation)、符号决议(symbol resolution)/符号绑定(symbol binding)、地址和空间分配(address and storage allocation)

## 符号决议

一般在静态链接中使用术语：符号决议(symbol resolution)

在动态链接中使用术语：符号绑定(symbol binding)/名称绑定(name binding)

在不特指的情况下，可以把这些词看成一个概念。

# 现代编译系统



# linux中的链接

## ld

linux使用ld(The GNU linker)作为链接器。



## 目标文件&ELF

编译后还没链接的文件被成为目标文件。linux中使用ELF(format of Executable and Linking Format )格式。

在windows使用PE格式，它和ELF一样，都是COFF格式的变种。

早期unix系统的可执行文件格式为a.out格式，后来提出的COFF的格式规范。现在gcc默认生成的可执行文件名就是a.out。

目标文件和可执行文件的与OS和编译器密切相关，目标文件和可执行文件几乎反应了OS的发展。

### ELF格式分类

normal executable files、relocatable  object  files、core  files、shared libraries.



### ELF结构

```
The header file <elf.h> defines the format of ELF executable binary files.  Amongst these files
are normal executable files, relocatable object files, core files, and shared objects.

An executable file using the ELF file format consists of an ELF header, followed by  a  program
header  table  or  a section header table, or both.  The ELF header is always at offset zero of
the file.  The program header table and the section header  table's  offset  in  the  file  are
defined  in  the  ELF  header.   The two tables describe the rest of the particularities of the
file.

This header file describes the above mentioned headers as C structures and also includes struc‐
tures for dynamic sections, relocation sections and symbol tables.
```

具体查看man elf

### section

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20211121153504.png" alt="image-20211121144400353" style="zoom: 80%;" />

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20211121153512.png" alt="image-20211121144341580" style="zoom: 80%;" />

### 调试信息

P107



## 静态链接

现代链接器一般采用一种两步链接(Two-pass linking):

**1虚拟地址空间的分配**

**2符号解析和重定位**

![image-20211121153742148](https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20211121153742.png)

### 虚拟地址空间的分配

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201216195320.png" alt="image-20201216195319974" style="zoom:33%;" />

### 符号解析与重定位

#### 符号表

An  object file's symbol table holds information needed to locate and relocate a program's sym‐
bolic definitions and references.  A symbol table index is a subscript into this array.

    typedef struct {
        uint32_t      st_name;
        Elf32_Addr    st_value;
        uint32_t      st_size;
        unsigned char st_info;
        unsigned char st_other;
        uint16_t      st_shndx;
    } Elf32_Sym;
    
    typedef struct {
        uint32_t      st_name;
        unsigned char st_info;
        unsigned char st_other;
        uint16_t      st_shndx;
        Elf64_Addr    st_value;
        uint64_t      st_size;
    } Elf64_Sym;

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201222083940.png" alt="image-20201222083933607" style="zoom:33%;" />

##### 特殊符号

使用ld作为链接器时，ld会自定义一些符号，可以在程序中直接使用这些符号。这些符号可以当成ld的内置符号。

常见的有：

![image-20211121145543599](https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20211121153520.png)

##### 若符号&强符号

编译器默认函数和初始化了的全局变量为强符号，未初始化的全局变量为弱符号。

也可以使用\__attribute__((weak))指定符号位弱符号。

##### 强引用&弱引用

强引用：找不到符号的定义报错

弱引用：找不到符号的定义不会报错，默认为特殊值或者0，使用这个符号时才会报错。

![image-20211121151238860](https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20211121153010.png)

![image-20211121151519166](https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20211121153020.png)

#### 重定位表

relocation table，表项为重定位入口(relocation entry)

.rel.text、.rel.data

Relocation  is  the process of connecting symbolic references with symbolic definitions.  Relo‐
catable files must have information that describes how to modify their section  contents,  thus
allowing  executable and shared object files to hold the right information for a process's pro‐
gram image.  Relocation entries are these data.

Relocation structures that do not need an addend:

    typedef struct {
        Elf32_Addr r_offset;
        uint32_t   r_info;
    } Elf32_Rel;
    
    typedef struct {
        Elf64_Addr r_offset;
        uint64_t   r_info;
    } Elf64_Rel;

Relocation structures that need an addend:

    typedef struct {
        Elf32_Addr r_offset;
        uint32_t   r_info;
        int32_t    r_addend;
    } Elf32_Rela;
    
    typedef struct {
        Elf64_Addr r_offset;
        uint64_t   r_info;
        int64_t    r_addend;
    } Elf64_Rela;

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201222181748.png" alt="企业微信截图_b6740cdf-fcb6-426b-bfc2-7c526f42172b" style="zoom:33%;" />

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201222181821.png" alt="企业微信截图_c8996e0c-a378-452d-98ec-093d90fa72f2" style="zoom:33%;" />

#### 指令修正方式

![image-20211121155154869](https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20211121155154.png)

### COMMON

弱符号(未初始化的全局变量)会被标记为一个common块。在目标文件中，不会在bss、data段中分配空间。只会在符号表中有一个common类型的表项标识它。

原因是弱符号在链接之前并不能确定其大小，无法为其分配空间。也可以说本质原因是链接器不支持符号类型，所以导致了处理弱符号情况时，无法决定其大小。

### 变量所在的位置

![image-20211121170817782](https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20211121170817.png)

1: .text 2: .data 3: .bss 4: .rodata



## 动态链接

静态库的问题：没有模块化。

因此出现了动态链接，动态链接使得程序被成分了多个模块：可执行文件和共享对象。

模块化提出了一个问题:多个程序都使用同一个共享对象时，如何在物理内存中共享？

动态链接的实现是和操作系统有关的，他也算是操作系统的一部分

早期的OS实现是静态共享库，在每个程序中，静态共享模块的地址空间是一样的。

显然静态共享库并不是很好的方案，后又提出了动态共享库，这个实现基于PIC技术。



gcc -shared 编译共享模块，不共享物理内存，在x86-64中好像已经不能使用

gcc -shared -fPIC 动态共享模块

下文只对动态共享模块进行讨论。

动态共享模块不需要是pic的，可执行文件可以不是pic的，但是目前默认是使用pic的。





### 共享模块的编译

gcc -shared -fPIC 命令编译动态共享模块可以指定多个c文件。对于每个输入的c文件来说，

对于非静态变量和函数，他无法区分他是来自别的模块还是本模块的其他c文件，无法区分的原因在于编译共享模块的具体实现。在我看来，具体实现中没有“链接过程”，所以无法区分。



### 位置无关代码

position-independent code，PIC

共享模块在物理内存中，只有代码(只读段)是可以共享，数据(非可读段)是写时复制的。

PIC技术是代码段可以被加载到任何地址空间中。

PIC技术引入了GOT段，实际在elf中会有.got和.got.plt段(使用plt技术)。

本质上来说：把之前代码段中需要重定位的条目移到了GOT中。



### 延迟绑定

延迟绑定(Lazy Binding)：对GOT代码的重定位可以推迟到它第一次使用时。

ELF使用PLT(Procedure Linkage Table)的方法来实现。

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20211203153427.png" alt="企业微信截图_8e27e3d8-11dc-4b0e-820a-6a7b043018d1" style="zoom: 50%;" />

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20211203153508.png" alt="企业微信截图_5c65af6d-5d2d-45ee-8faa-e189d62f0d00" style="zoom:50%;" />



### 动态链接的过程

1.可执行文件的生成和静态链接一样，链接的过程只处理静态符号。ELF默认可执行文件使用PIC和

   延迟绑定技术。

2.可执行文件中.interp存放了动态链接器文件的地址。

3.ELF文件.dynamic(key/value表)存放ld需要的信息。

4.ELF文件.rela.dyn记录.got和.data中需要重定位的项，.rela.plt记录.got.plt的重定位的项。

5.ELF文件.dynsym为动态符号表，.symtab是他的超集。

6.ELF文件.dynstr为动态符号名表，类似.strtab

7.ELF文件.hash为辅助.dynstr的section





## 装载

**装载考虑的是如何分配物理内存**

虚拟存储(虚拟内存)概念之前，一般采用覆盖装入的方式

虚拟存储(虚拟内存)概念之后，一般采用页映射(分页)的方式

覆盖载入(overlay)

overlay manager 分配和管理物理内存

页映射(paging)

paging是虚拟内存概念的一部分，也需要一个manager，一般就是OS。

现代操作系统都是用页映射的方式，下面只论述linux采用的页映射的装载方式

### 装载过程

装载的过程实际就是OS创建进程的过程:

1 创建虚拟内存和物理内存的映射关系，其实这里只需要分配一个页目录即可，具体的页可以后面在使用的时候分配

2 创建物理内存和可执行文件(还有空文件)的映射关系，这一步是最核心的步骤。因此可执行文件也被称为映像文件 (Image)为此，linux提出了VMA的概念。windows有相对应的virtual section概念。

3 跳转至ELF的入口地址

### VMA

VMA(virtual memory area)表示一个进程的一段连续的虚拟内存，内核认为这段虚拟内存有这同样的属性。

ELF中的segment就是与之相关的一个概念。加载器根据segment信息去创建一块虚拟

内存区域和物理内存的映射。**注意不是所有的VMA并不完全和segment一致**。

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201216111926.png" style="zoom:33%;" />

LOAD表示要装载的segment

### 栈的初始化

#### 静态链接的栈

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20211122093209.png" alt="image-20211122093209803" style="zoom:33%;" />

**栈的初始化部分由加载器完成，部分由库函数(_start)完成。具体情况还不清楚。**

#### 动态链接的栈

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20211122093805.png" alt="image-20211122093804885" style="zoom:50%;" />

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20211122093601.png" alt="image-20211122093601089" style="zoom:50%;" />

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20211122093651.png" alt="image-20211122093651193" style="zoom:50%;" />

### 具体装载过程

todo

### 动态链接器的实现

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201226163500.png" alt="image-20201226163500137" style="zoom:50%;" />

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201226163548.png" alt="image-20201226163548539" style="zoom:50%;" />

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201226163636.png" alt="image-20201226163635971" style="zoom:50%;" />

### 动态链接的装载过程

#### 动态链接器的自举

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201226162023.png" alt="image-20201226162023822" style="zoom:50%;" />

动态链接首先运行**他自身**的自举代码

#### 装载共享对象

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201226162608.png" alt="image-20201226162608282" style="zoom:50%;" />

#### 重定位和初始化

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201226163028.png" alt="image-20201226163028404" style="zoom:50%;" />

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201226163057.png" alt="image-20201226163057765" style="zoom:50%;" />



## 动态链接栈初始化

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201225001511.png" alt="image-20201225001511481" style="zoom:33%;" />

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201225001544.png" alt="image-20201225001544142" style="zoom:33%;" />

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201225001620.png" alt="image-20201225001620232" style="zoom:33%;" />



# 

# GLIBC

linux下的C运行库，是C标准库(ISO C)的超集。linux中还有其他的C运行库：linux libc、uClibc。

libc.a/libc.so只是glibc中的文件之一。

## 静态库

一组目标文件的集合，很多目标文件经过压缩打包后形成的一个文件

ar -t 可以查看包含哪些目标文件

ar -x 可以解压包含的目标文件到当前目录

**静态库中的每个目标文件只包含一个函数**



# 堆

# TODO

core dump file

lds

vsdo