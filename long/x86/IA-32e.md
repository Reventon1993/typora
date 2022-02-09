# 运行模式

![image-20220209172011928](https://raw.githubusercontent.com/Reventon1993/pictures/master/picgo/20220209172012.png)

![企业微信截图_a715618a-09b2-412a-b86a-586ca00644ac](https://raw.githubusercontent.com/Reventon1993/pictures/master/picgo/20220209172036.png)



## 模式转换图

![企业微信截图_2bb2e5b5-b1c8-4891-8515-55aa3a6222b3](https://raw.githubusercontent.com/Reventon1993/pictures/master/picgo/20220209172113.png)

## BIG REAL MODE

一种特殊的实模式，该模式下用户大于1M的数据访址能力。

**一般在系统启动初期会进入该模式，此时系统既希望拥有大于1M的数据访址能力，又希望可以使用bios提供的实模式下的中断服务。**

### 进入big real mode

```assembly
        ;=== 开启A20地址
        push    ax
        in      al,     92h
        or      al,     00000010b
        out     92h,    al
        pop     ax

        cli
     
        ;=== 置位CR0.PE进入保护模式
        lgdt    [GdtPtr]

        mov     eax,    cr0
        or      eax,    1
        mov     cr0,    eax
				
				;=== 加载2^32空间的代码段
        mov     ax,     SelectorData32
        mov     fs,     ax
        
        ;=== 复位CR0.PE返回到实模式
        ;=== 此时fs的缓存部分中已经加载了32位代码段的信息，即使回到实模式，也依旧用于32位代码段的
        ;访址能力
        
        mov     eax,    cr0
        and     al,     11111110b
        mov     cr0,    eax

        sti
        
        
        LABEL_GDT:              dd      0,0
				LABEL_DESC_CODE32:      dd      0x0000FFFF,0x00CF9A00
				LABEL_DESC_DATA32:      dd      0x0000FFFF,0x00CF9200

				GdtLen  equ     $ - LABEL_GDT
				GdtPtr  dw      GdtLen - 1
        				dd      LABEL_GDT       
```

# CPUID

通过cpuid指令我们可以得知CPU当前是否支持某项特性。

**通过标志寄存器的ID标志位可以查看CPU是否支持该指令。**

**cpuid在任何模式下执行的效果均相同**

### CPUID

![image-20220209175121122](https://raw.githubusercontent.com/Reventon1993/pictures/master/picgo/20220209175121.png)

![企业微信截图_a9783b07-ae9a-4314-817f-ebe604e36953](https://raw.githubusercontent.com/Reventon1993/pictures/master/picgo/20220209175157.png)

# 寄存器

## 通用寄存器

### <img src="/Users/jiangchuan/Library/Application Support/typora-user-images/image-20220209214322736.png" alt="image-20220209214322736" style="zoom: 33%;" />

![image-20220209214755545](/Users/jiangchuan/Library/Application Support/typora-user-images/image-20220209214755545.png)

### RBP栈帧寄存器

GAS汇编器对过程的编译：

```assembly
push %rbp
mov %rsp,%rbp
...
...
...
leaveq
retq
```

leavq指令相当于mov %rbp,%rsp ; pop %rbp

## 段寄存器

<img src="/Users/jiangchuan/Library/Application Support/typora-user-images/image-20220209220107095.png" alt="image-20220209220107095" style="zoom: 50%;" />

段寄存器分为16位的“可见部分”(装载段选择子)和不可见的缓存部分。

长模式/64位CPU的段寄存器依旧分为这两个部分，且可见部分一样，缓存部分内部结构可能不一样。

## 标志寄存器

<img src="/Users/jiangchuan/Library/Application Support/typora-user-images/image-20220209220839265.png" alt="image-20220209220839265" style="zoom: 50%;" />

VM、NT位保留标志位表示:64位模式下不在支持，32位下支持。

## 控制寄存器

<img src="/Users/jiangchuan/Library/Application Support/typora-user-images/image-20220209221029219.png" alt="image-20220209221029219" style="zoom: 25%;" />

## MSR寄存器组

<img src="/Users/jiangchuan/Library/Application Support/typora-user-images/image-20220209221133100.png" alt="image-20220209221133100" style="zoom: 25%;" />

## IP/IDTR/LGTR/TR/GTDR/其他寄存器

https://wiki.osdev.org/CPU_Registers_x86-64#General_Purpose_Registers



# 地址空间

## 虚拟地址&物理地址

<img src="/Users/jiangchuan/Library/Application Support/typora-user-images/image-20220209223221258.png" alt="image-20220209223221258" style="zoom: 33%;" />

**未启用分页机制：逻辑地址→虚拟地址=物理地址**

**启用分页机制：逻辑地址→虚拟地址→物理地址**

## 分段机制

保护模式中引入，在分页功能引入后，分段机制已经不再常用。长模式直接大大简化了分段机制。

系统段和数据段依旧是64位，但大大简化：

### 代码段&数据段

<img src="/Users/jiangchuan/Library/Application Support/typora-user-images/image-20220209231208438.png" alt="image-20220209231208438" style="zoom:33%;" />



<img src="/Users/jiangchuan/Library/Application Support/typora-user-images/image-20220209231239090.png" alt="image-20220209231239090" style="zoom:33%;" />

### 系统段描述符

**系统段描述符扩展位128位，不再有任务门描述符**

linux不使用调用门、LDT，就不再描述。中断门/陷阱门放在中断章节描述

#### TSS描述符

<img src="/Users/jiangchuan/Library/Application Support/typora-user-images/image-20220209231627580.png" alt="image-20220209231627580" style="zoom:33%;" />



TSS段，有较大改动

<img src="/Users/jiangchuan/Library/Application Support/typora-user-images/image-20220209231709950.png" alt="image-20220209231709950" style="zoom:33%;" />

IST为终端栈表，在中断部分描述。

## 分页机制

<img src="/Users/jiangchuan/Library/Application Support/typora-user-images/image-20220209232606716.png" alt="image-20220209232606716" style="zoom:50%;" />

<img src="/Users/jiangchuan/Library/Application Support/typora-user-images/image-20220209232551256.png" alt="image-20220209232551256" style="zoom:50%;" />



## IO地址空间

TODO

# 中断机制

<img src="/Users/jiangchuan/Library/Application Support/typora-user-images/image-20220209232357556.png" alt="image-20220209232357556" style="zoom:33%;" />





