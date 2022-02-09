# ELF

ELF: format of Executable and Linking Format (ELF) files。

## 历史和发展

目标文件和可执行文件的与OS和编译器密切相关，目标文件和可执行文件几乎反应了OS的发展。

unix最早的可执行文件格式为a.out,a.out没有共享库的概念。所以出现了COFF格式。ELF和windows的PE(

Portable Executable)都是COFF的变种。

## 结构

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



### ELF header (Ehdr)

    The ELF header is described by the type Elf32_Ehdr or Elf64_Ehdr:
    
        #define EI_NIDENT 16
    
        typedef struct {
            unsigned char e_ident[EI_NIDENT];
            uint16_t      e_type;
            uint16_t      e_machine;
            uint32_t      e_version;
            ElfN_Addr     e_entry;
            ElfN_Off      e_phoff;
            ElfN_Off      e_shoff;
            uint32_t      e_flags;
            uint16_t      e_ehsize;
            uint16_t      e_phentsize;
            uint16_t      e_phnum;
            uint16_t      e_shentsize;
            uint16_t      e_shnum;
            uint16_t      e_shstrndx;
        } ElfN_Ehdr;
       e_type    This member of the structure identifies the object file type:
    
                 ET_NONE         An unknown type.
                 ET_REL          A relocatable file.
                 ET_EXEC         An executable file.
                 ET_DYN          A shared object.
                 ET_CORE         A core file.
### Program header (Phdr)

```
An executable or shared object file's program header table is  an  array  of  structures,  each
describing  a  segment  or other information the system needs to prepare the program for execu‐
tion.  An object file segment contains one or more sections.  Program  headers  are  meaningful
only for executable and shared object files.  A file specifies its own program header size with
the ELF header's e_phentsize and e_phnum members.  The ELF program header is described  by  the
type Elf32_Phdr or Elf64_Phdr depending on the architecture:

        typedef struct {
            uint32_t   p_type;
            Elf32_Off  p_offset;
            Elf32_Addr p_vaddr;
            Elf32_Addr p_paddr;
            uint32_t   p_filesz;
            uint32_t   p_memsz;
            uint32_t   p_flags;
            uint32_t   p_align;
        } Elf32_Phdr;

        typedef struct {
            uint32_t   p_type;
            uint32_t   p_flags;
            Elf64_Off  p_offset;
            Elf64_Addr p_vaddr;
            Elf64_Addr p_paddr;
            uint64_t   p_filesz;
            uint64_t   p_memsz;
            uint64_t   p_align;
        } Elf64_Phdr;
```



   ### Section header (Shdr)

    A file's section header table lets one locate all the file's sections.  The section header  ta‐
    ble  is an array of Elf32_Shdr or Elf64_Shdr structures.  The ELF header's e_shoff member gives
    the byte offset from the beginning of the file to the section header table.  e_shnum holds  the
    number  of  entries  the section header table contains.  e_shentsize holds the size in bytes of
    each entry.
    
        The section header has the following structure:
    
            typedef struct {
                uint32_t   sh_name;
                uint32_t   sh_type;
                uint32_t   sh_flags;
                Elf32_Addr sh_addr;
                Elf32_Off  sh_offset;
                uint32_t   sh_size;
                uint32_t   sh_link;
                uint32_t   sh_info;
                uint32_t   sh_addralign;
                uint32_t   sh_entsize;
            } Elf32_Shdr;
    
            typedef struct {
                uint32_t   sh_name;
                uint32_t   sh_type;
                uint64_t   sh_flags;
                Elf64_Addr sh_addr;
                Elf64_Off  sh_offset;
                uint64_t   sh_size;
                uint32_t   sh_link;
                uint32_t   sh_info;
                uint64_t   sh_addralign;
                uint64_t   sh_entsize;
            } Elf64_Shdr;
### section

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201215120401.png" style="zoom: 33%;" />

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201215130625.png" style="zoom:33%;" />

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201215130812.png" style="zoom:33%;" />



# 静态链接

现代链接器一般采用一种两步链接(Two-pass linking):

**1虚拟地址空间的分配**

**2符号解析和重定位**

## 虚拟地址空间的分配

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201216195320.png" alt="image-20201216195319974" style="zoom:33%;" />

## 符号解析与重定位

### 重定位表

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

### 符号表

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

### 字符串表

    String  table  sections hold null-terminated character sequences, commonly called strings.  The
    object file uses these strings to represent symbol and section names.  One references a  string
    as  an index into the string table section.  The first byte, which is index zero, is defined to
    hold a null byte ('\0').  Similarly, a string table's last byte is defined to hold a null byte,
    ensuring null termination for all strings.
### 强符号和弱符号

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201217092918.png" alt="企业微信截图_66c7f1db-b424-421f-b9e6-28a50021c95b" style="zoom:33%;" />

### 强引用&弱引用

强引用：找不到符号的定义报错

弱引用：找不到符号的定义不会报错，默认为特殊值或者0，使用这个符号时才会报错。

弱符号和弱引用一般用户库的链接场景中

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201217105513.png" alt="企业微信截图_9d04e7ca-8b77-4fa6-9943-a5edec741b4b" style="zoom:33%;" />

### 静态库

一组目标文件的集合，很多目标文件经过压缩打包后形成的一个文件

ar -t 可以查看包含哪些目标文件

ar -x 可以解压包含的目标文件到当前目录

**静态库中的每个目标文件只包含一个函数**

# 动态链接

## 静态链接的问题

1.没有模块化，任一处的修改需要重新编译

2.没有代码复用

3.不能动态地更新代码

## 模块

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201217184431.png" alt="企业微信截图_7f68a172-8cb6-42c5-baa4-18c803ed9069" style="zoom:33%;" />

1.每个模块对应1-n个目标文件

2.程序只包括一个程序主模块和可以多个共享模块

3.程序主模块和动态链接库由目标文件生成的过程是不一样的。--shared表示生成共享模块

4.--fpic不是必要的生成共享模块的参数，也就是说代码位置无关不是共享模块必要的技术。实际上代码位置无关

   是为了静态链接的问题2。

## 装载时重定位

解决问题1

主程序模块在生成时候，依旧有静态链接过程，依旧会对静态符号进行重定位。

如何区分是静态符号还是动态符号：

链接的时候回去查看所有输入的共享模块、动态链接库。如果符号在其中，那么就当成动态符号。



共享模块在生成的时候，不会进行任何重定位。所以他存在以下问题：

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201217185801.png" alt="image-20201217185801372" style="zoom:33%;" />

这种场景下，global被当成定义在外部模块。动态链接的时候动态链接器去具体地重定位该符号。



装载的时候，动态链接器对主程序模块和共享模块中的符号进行重定位。

## 位置无关代码

解决问题2

相关段GOT

## PLT

一定要配合GOT,引入一个PLT段，是一个可读可执行的代码段。

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201222185737.png" alt="企业微信截图_46f24c24-9d73-4eda-8887-8e29218280ab" style="zoom:33%;" />

PLT[1]调用了__libc_start_main

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master/picgo/20220209161708.png" alt="image-20201223161050423" style="zoom:33%;" />

## 相关section

interp

dynamic

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201223160323.png" alt="image-20201223160323091" style="zoom:33%;" />

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201223160441.png" alt="image-20201223160441791" style="zoom:33%;" />



dynsym

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master/picgo/20220209161728.png" alt="image-20201223161304788" style="zoom:33%;" />

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201223161337.png" alt="image-20201223161337272" style="zoom:33%;" />

hash & dynstr

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master/picgo/20220209161749.png" alt="image-20201223161412561" style="zoom:33%;" />

rel.dyn & rel.plt

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201223161514.png" alt="image-20201223161514534" style="zoom:33%;" />

## 运行时链接

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201226163736.png" alt="image-20201226163736638" style="zoom:50%;" />

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201226163756.png" alt="image-20201226163756692" style="zoom:50%;" />

## 共享库

### 命名

![image-20201231072022620](https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201231072028.png)

![image-20201231073217219](https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201231073217.png)

### so-name

![image-20201231073616717](https://raw.githubusercontent.com/Reventon1993/pictures/master/picgo/20220209161805.png)

![image-20201231073649709](https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201231073649.png)

### linux共享库路径



# 装载

**装载考虑的是如何分配物理内存**

虚拟存储(虚拟内存)概念之前，一般采用覆盖装入的方式

虚拟存储(虚拟内存)概念之后，一般采用页映射(分页)的方式

## 覆盖载入(overlay)

overlay manager 分配和管理物理内存

## 页映射(paging)

paging是虚拟内存概念的一部分，也需要一个manager，一般就是OS

### 装载过程

paging的装载过程有三个步骤

1 创建虚拟内存和物理内存的映射关系，其实这里只需要分配一个页目录即可，具体的页可以后面在使用的时候分配

2 创建物理内存和可执行文件(还有空文件)的映射关系，这一步是最核心的步骤。因此可执行文件也被称为映像文件 (Image)为此，linux提出了VMA的概念。windows有相对应的virtual section概念。

3 跳转至ELF的入口地址

### VMA

VMA(virtual memory area)表示一个进程的一段连续的虚拟内存，内核认为这段虚拟内存有这同样的属性。

ELF中的segment就是与之相关的一个概念。加载器根据segment信息去创建一块虚拟

内存区域和物理内存的映射。注意不是所有的VMA并不完全和segment一致。

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201216111926.png" style="zoom:33%;" />



LOAD表示要装载的segment

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201216122217.png" alt="image-20201216122217249" style="zoom:33%;" />





## 装载过程

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201222235828.png" alt="image-20201222235828069" style="zoom: 50%;" />

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201222235848.png" style="zoom:50%;" />

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201222235926.png" alt="image-20201222235926418" style="zoom:50%;" />

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201223131748.png" alt="image-20201223131747870" style="zoom:33%;" />

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

## 静态链接栈初始化

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201226160832.png" alt="企业微信截图_189c7701-ef81-4948-af00-7feb7b98c20f" style="zoom: 50%;" />

## 动态链接栈初始化

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201225001511.png" alt="image-20201225001511481" style="zoom:33%;" />

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201225001544.png" alt="image-20201225001544142" style="zoom:33%;" />

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201225001620.png" alt="image-20201225001620232" style="zoom:33%;" />



# 运行环境

## 内存

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201226172757.png" alt="image-20201226172757747" style="zoom: 67%;" />

## 

### 栈

#### ebp

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201226173140.png" alt="image-20201226173140583" style="zoom:50%;" />

ebp作为帧指针只是编译器的策略

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201226173248.png" alt="image-20201226173248764" style="zoom:67%;" />

#### 乱码“烫”和“屯”

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201226173450.png" alt="image-20201226173449895" style="zoom: 67%;" />

### 堆

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201226174005.png" alt="image-20201226174005647" style="zoom:67%;" />

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201226174047.png" alt="image-20201226174047384" style="zoom:67%;" />

#### 堆分配算法

堆分配算法是运行库的算法，主要以下三种实现方式：

- 空闲链表
- 位图
- 对象池

## 运行库

### 运行库vs标准库

https://www.cnblogs.com/grooovvve/p/12907659.html

## 系统调用

# 工具&命令

r

objdump:可以进行反汇编

objcopy

nm

![image-20201217232128338](https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201217232128.png)

# 链接控制过程



vsdo vma 

core dump

108 特殊符号

109 函数名

115 __ attribute __((weak))

118 gcc -g DWARF

145 libc.a不包括crtl.o

176 pae 32cpu 36位地址线

220 so是否为pic pic&pie

222 共享数据段&TLS

229 内核虚拟对象

365、368 标准库、运行库、crtl.o

375 crt

409 系统调用

422 dd mem

423 syscenter