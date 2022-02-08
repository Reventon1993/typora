# Boot

In [computing](https://en.wikipedia.org/wiki/Computing), **booting** is the process of starting a [computer](https://en.wikipedia.org/wiki/Computer). 

*Boot* is short for [*bootstrap*](https://en.wikipedia.org/wiki/Bootstrapping)[[1\]](https://en.wikipedia.org/wiki/Booting#cite_note-2)[[2\]](https://en.wikipedia.org/wiki/Booting#cite_note-3) or *bootstrap load* and derives from the phrase *[to pull oneself up by one's bootstraps](https://en.wikipedia.org/wiki/Bootstrapping#Etymology)*.[[3\]](https://en.wikipedia.org/wiki/Booting#cite_note-4)[[4\]](https://en.wikipedia.org/wiki/Booting#cite_note-5) The usage calls attention to the requirement that, if most software is loaded onto a computer by other software already running on the computer, some mechanism must exist to load the initial software onto the computer.[[5\]](https://en.wikipedia.org/wiki/Booting#cite_note-6)

boot的实现方式有很多种，我们只讨论现代计算机系统最常用的使用bios和uefi启动的方式。

# bios+mbr

bios+mbr的方式是IBM提出的一套框架，里面不仅仅涉及booting。还涉及disk partitioning等。这套方案只适用于 [IBM PC-compatible](https://en.wikipedia.org/wiki/IBM_PC-compatible) systems and beyond



## boot sector

A **boot sector** is the [sector](https://en.wikipedia.org/wiki/Disk_sector) of a persistent [data storage device](https://en.wikipedia.org/wiki/Data_storage_device) (e.g., [hard disk](https://en.wikipedia.org/wiki/Hard_disk), [floppy disk](https://en.wikipedia.org/wiki/Floppy_disk), [optical disc](https://en.wikipedia.org/wiki/Optical_disc), etc.) which contains [machine code](https://en.wikipedia.org/wiki/Machine_code) to be loaded into [random-access memory](https://en.wikipedia.org/wiki/Random-access_memory) (RAM) and then executed by a [computer system](https://en.wikipedia.org/wiki/Computer_system)'s built-in [firmware](https://en.wikipedia.org/wiki/Firmware) (e.g., the [BIOS](https://en.wikipedia.org/wiki/BIOS)).

### mbr

A **master boot record** (**MBR**) is a special type of [boot sector](https://en.wikipedia.org/wiki/Boot_sector) at the very beginning of [partitioned](https://en.wikipedia.org/wiki/Disk_partitioning) computer [mass storage devices](https://en.wikipedia.org/wiki/Mass_storage_device) like [fixed disks](https://en.wikipedia.org/wiki/Fixed_disk) or [removable drives](https://en.wikipedia.org/wiki/Removable_drive) intended for use with [IBM PC-compatible](https://en.wikipedia.org/wiki/IBM_PC-compatible) systems and beyond. The concept of MBRs was publicly introduced in 1983 with [PC DOS 2.0](https://en.wikipedia.org/wiki/PC_DOS_2.0).

### vbr/pbr

A **volume boot record** (**VBR**) (also known as a **volume boot sector**, a **partition boot record** or a **partition boot sector**) is a type of [boot sector](https://en.wikipedia.org/wiki/Boot_sector) introduced by the [IBM Personal Computer](https://en.wikipedia.org/wiki/IBM_Personal_Computer). It may be found on a [partitioned](https://en.wikipedia.org/wiki/Disk_partitioning) [data storage device](https://en.wikipedia.org/wiki/Data_storage_device), such as a [hard disk](https://en.wikipedia.org/wiki/Hard_disk), or an unpartitioned device, such as a [floppy disk](https://en.wikipedia.org/wiki/Floppy_disk), and contains [machine code](https://en.wikipedia.org/wiki/Machine_code) for [bootstrapping](https://en.wikipedia.org/wiki/Bootstrapping_(computing)) [programs](https://en.wikipedia.org/wiki/Computer_program) (usually, but not necessarily, [operating systems](https://en.wikipedia.org/wiki/Operating_system)) stored in other parts of the device

## bios

bios主要工作如下：

1.POST

2.建立IVT

  **optional bios**:

  https://en.wikipedia.org/wiki/Option_ROM 

![企业微信截图_7eb7a434-1783-4f9e-a05b-8c020146de4d](/Users/jiang/Library/Containers/com.tencent.WeWorkMac/Data/Library/Application Support/WXWork/Temp/ScreenCapture/企业微信截图_7eb7a434-1783-4f9e-a05b-8c020146de4d.png)

![企业微信截图_1c395993-1fe4-477c-855b-e435050b1632](/Users/jiang/Library/Containers/com.tencent.WeWorkMac/Data/Library/Application Support/WXWork/Temp/ScreenCapture/企业微信截图_1c395993-1fe4-477c-855b-e435050b1632.png)

3.交接给boot sector

### to boot sector

bios根据引导顺序，寻找设备的第一个sector，如果第一个扇区最后俩字节是0X55AA。那么

把控制权给改扇区的第一个字节。

对于硬盘，第一个sector就是MBR程序，

对于软盘等不可分区的设备，第一个sector就是VBR程序

## MBR

MBR可以看成一种规范化的数据或者程序，这种规范规定了booting和disk partitioning。

MBR有多种形式，最常见的MBR形式：

446字节的引导代码+4*16分区表+0x55aa

分区表结构：

![企业微信截图_3bac87a2-acbd-477d-8d3d-e958108b3b8f](/Users/jiang/Library/Containers/com.tencent.WeWorkMac/Data/Library/Application Support/WXWork/Temp/ScreenCapture/企业微信截图_3bac87a2-acbd-477d-8d3d-e958108b3b8f.png)

第一个字节为active flag,活动分区表示该分区中有内核/second-stage bootloader

第五个字节为分区类型，表示分区的类型。

### Disk identity

In addition to the bootstrap code and a partition table, master boot records may contain a [disk signature](https://en.wikipedia.org/wiki/Master_boot_record#DISK_ID). This is a 32-bit value that is intended to identify uniquely the disk medium

### partition vs filesystem

分区和文件系统并不是1对1的关系。比如lvm分区：

多个lvm分区(pv)可以构成一个vg，vg上面划分vm,vm上面创建一个文件系统。

# bootloader

A **bootloader**, also spelled as **boot loader**[[1\]](https://en.wikipedia.org/wiki/Bootloader#cite_note-GRUB-1)[[2\]](https://en.wikipedia.org/wiki/Bootloader#cite_note-systemd-2) or called **boot manager**[[2\]](https://en.wikipedia.org/wiki/Bootloader#cite_note-systemd-2) and **bootstrap loader**, is a [computer program](https://en.wikipedia.org/wiki/Computer_program) that is responsible for [booting](https://en.wikipedia.org/wiki/Booting) a computer.

bootloader也是一个宽泛的概念。用于booting的程序都是bootloader。

在上述bios+mbr的booting方案中，mbr就是一个bootloader。因为mbr大小有限，能完成的功能较少。为了解决这个

问题，mbr会把控制权给其他的bootloader(文件系统中)。

## Single Stage Bootloader

## Two-Stage Bootloader

### First-stage bootloader

In the first-stage bootloader, we must do the following:

- Setup the memory segments and stack used by the bootloader code
- Reset the disk system
- Display a string saying “Loading OS…”
- Find the second-stage boot loader in the FAT directory
- Read the second-stage boot loader image into memory at 1000:0000
- Transfer control to the second-stage bootloader

### Second-stage bootloader

In the second-stage bootloader, we must do the following:

- Copy the boot sector data bytes to a local memory area, as they will be overwritten
- Find the kernel image in the FAT directory
- Read the kernel image into memory at 2000:0000
- Reset the disk system
- Enable the A20 line
- Setup the interrupt descriptor table at 0000:0000
- Setup the global descriptor table at 0000:0800
- Load the descriptor tables into the CPU
- Switch to protected mode
- Clear the prefetch queue
- Setup protected mode memory segments and stack for use by the kernel code
- Transfer control to the kernel code using a long jump

# UEFI+GPT

## UEFI

uefi是bios的后继，booting和bios不相同。

uefi提供兼容了对bios的兼容:CSM模式

UEFI不再依赖mbr，他可以自行找到文进系统中的bootloader文件。

## GPT

gpt是mbr的后继，优化了booting、disk partition等。

<img src="/Users/jiang/Library/Containers/com.tencent.WeWorkMac/Data/Library/Application Support/WXWork/Temp/ScreenCapture/企业微信截图_801b1cf0-132c-45b2-b121-25011ee5c126.png" alt="企业微信截图_801b1cf0-132c-45b2-b121-25011ee5c126" style="zoom:50%;" />

gpt中第一个sector保留为mbr做兼容。在这种模式下，mbr中的分区表为空。mbr中的引导程序要能识别

这种场景才能正确booting。所以在gpt兼容mbr场景中，关键是第一个扇区中的启动代码要做好兼容。



# 总结

1 booting的整个方案涉及多个方面:固件、磁盘的分区方式、文件系统、bootloader

2 bios+mbr的方式最早由IBM公司提出，该标准后成了标准方案，所以后续的机器实际上都是IBM PC兼容机

3 boot sector(包括mbr)上的bootloader代码需要能解析引导扇区的文件系统，这样才能交接给后续的bootloader文件/kernel文件。

4 uefi+gpt是bios+mbr的后继，gpt的出现更多的是为了解决mbr对磁盘容量的限制。uefi开启csm模式可以引导mbr。

  gpt的第一个扇区的bootloader代码做兼容可以实现bios+gpt的方案。

5 gpt可以识别多个文件系统，他可以直接把控制权交接给bootloader文件/kernel文件。boot过程就不再需要boot sector

   的参与。



