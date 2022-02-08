# 简介

- **BIOS**  an [acronym](https://en.wikipedia.org/wiki/Acronym) for **Basic Input/Output System** and also known as the **System BIOS**, **ROM BIOS** or **PC BIOS**) is [firmware](https://en.wikipedia.org/wiki/Firmware) used to perform hardware initialization during the booting process (power-on startup), and to provide runtime services for operating systems and programs

- Originally, BIOS firmware was stored in a [ROM](https://en.wikipedia.org/wiki/Read-only_memory) chip on the PC motherboard. In modern computer systems, the BIOS contents are stored on [flash memory](https://en.wikipedia.org/wiki/Flash_memory) so it can be rewritten without removing the chip from the motherboard. This allows easy, end-user updates to the BIOS firmware so new features can be added or bugs can be fixed, but it also creates a possibility for the computer to become infected with BIOS [rootkits](https://en.wikipedia.org/wiki/Rootkit). Furthermore, a BIOS upgrade that fails may [brick](https://en.wikipedia.org/wiki/Brick_(electronics)) the motherboard.

- [Unified Extensible Firmware Interface](https://en.wikipedia.org/wiki/Unified_Extensible_Firmware_Interface) (UEFI) is a successor to the legacy PC BIOS, aiming to address its technical limitations.[[6\]](https://en.wikipedia.org/wiki/BIOS#cite_note-Bradley-6)

- BIOS是与主板相关的，通产由主板开放商提供。一般由汇编编写，不开放源码。

# power off → BIOS

Early Intel processors started at physical address 000FFFF0h. Systems with later processors provide logic to start running the BIOS from the system ROM. [[14\]](https://en.wikipedia.org/wiki/BIOS#cite_note-14)

# POST(power-on self-test)

## POST流程

不同的BIOS的POST内容是不一样的，下面是IBM-compatible PC POST的流程：

The principal duties of the main BIOS during POST are as follows:

- verify CPU registers
- verify the integrity of the BIOS code itself
- verify some basic components like DMA, timer, interrupt controller
- find, size, and verify system [main memory](https://en.wikipedia.org/wiki/Main_memory)
- initialize BIOS
- pass control to other specialized extension BIOSes (if installed)
- identify, organize, and select which devices are available for booting

The functions above are served by the POST in all BIOS versions back to the very first. In later BIOS versions, POST will also:

- discover, initialize, and catalog all [system buses](https://en.wikipedia.org/wiki/System_bus) and devices
- provide a [user interface](https://en.wikipedia.org/wiki/User_interface) for system's configuration
- construct whatever system environment is required by the target [operating system](https://en.wikipedia.org/wiki/Operating_system)



注意:

- BIOS会通过更新中断向量表提供一系列基础的输入/输出接口(BIOS的由来)，这些接口BootLoader和os系统会使用，os加载会后一般会覆盖中断向量表，应用程序一般不会去使用BIOS提供的中断接口。开机时的选择操作系统、进入windows紧急模式等界面都使用了BIOS提供的中断。
- BIOS调用INT 19h寻找并把控制权交付给BootLoader。BIOS根据boot order去依次寻找[mass storage](https://en.wikipedia.org/wiki/Mass_storage)的第一个扇区，并校验最后两字节(0x55 0xAA)，交付给BootLoader后，BootLoader有绝对控制权。如果没有找到，则调用INT 18h。
- POST过程会对一些设备进行初始化的设置。
- extension BIOSes 详见下面

## extension BIOSes 

外围设备可以提供一个rom，被称为extension BIOS。BIOS在把控制权给extension BIOSes时流程中，会用如下方法依次查找所有的rom,找到后把控制权交给extension BIOS，extension BIOSes 拥有绝对控制权：

After the motherboard BIOS completes its POST, most BIOS versions search for option ROM modules, also called BIOS extension ROMs, and execute them. The motherboard BIOS scans for extension ROMs in a portion of the "[upper memory area](https://en.wikipedia.org/wiki/Upper_memory_area)" (the part of the x86 real-mode address space at and above address 0xA0000) and runs each ROM found, in order. To discover memory-mapped [ISA](https://en.wikipedia.org/wiki/Industry_Standard_Architecture) option ROMs, a BIOS implementation scans the real-mode address space from `0x0C0000` to `0x0F0000` on 2 [KiB](https://en.wikipedia.org/wiki/KiB) boundaries, looking for a two-byte ROM *signature*: 0x55 followed by 0xAA. In a valid expansion ROM, this signature is followed by a single byte indicating the number of 512-byte blocks the expansion ROM occupies in real memory, and the next byte is the option ROM's [entry point](https://en.wikipedia.org/wiki/Entry_point) (also known as its "entry offset"). A [checksum](https://en.wikipedia.org/wiki/Checksum) of the specified number of 512-byte blocks is calculated, and if the ROM has a valid checksum, the BIOS transfers control to the entry address, which in a normal BIOS extension ROM should be the beginning of the extension's initialization routine.

extension BIOS的常见用法如下：

- The motherboard BIOS typically contains code to access hardware components necessary for bootstrapping the system, such as the keyboard, display, and storage. In addition, plug-in adapter cards such as [SCSI](https://en.wikipedia.org/wiki/SCSI), [RAID](https://en.wikipedia.org/wiki/RAID), [network interface cards](https://en.wikipedia.org/wiki/Network_interface_card), and video boards often include their own BIOS (e.g. [Video BIOS](https://en.wikipedia.org/wiki/Video_BIOS)), complementing or replacing the system BIOS code for the given component. Even devices built into the motherboard can behave in this way; their option ROMs can be stored as separate code on the main BIOS [flash chip](https://en.wikipedia.org/wiki/Flash_chip), and upgraded either in tandem with, or separately from, the main BIOS.(初始化设备)
- An add-in card requires an option ROM if the card is not supported by the main BIOS and the card needs to be initialized or made accessible through BIOS services before the operating system can be loaded (usually this means it is required in the bootstrapping process). Even when it is not required, an option ROM can allow an adapter card to be used without loading driver software from a storage device after booting begins – with an option ROM, no time is taken to load the driver, the driver does not take up space in RAM nor on hard disk, and the driver software on the ROM always stays with the device so the two cannot be accidentally separated. Also, if the ROM is on the card, both the peripheral hardware and the driver software provided by the ROM are installed together with no extra effort to install the software.(提供设备驱动)
- A non-disk device such as a [network adapter](https://en.wikipedia.org/wiki/Network_adapter) attempts booting by a procedure that is defined by its [option ROM](https://en.wikipedia.org/wiki/Option_ROM) or the equivalent integrated into the motherboard BIOS ROM. As such, option ROMs may also influence or supplant the boot process defined by the motherboard BIOS ROM.(提供从网络启动系统的功能)



# BIOS → BOOTLOADER

The environment for the boot program is very simple: the CPU is in real mode and the general-purpose and segment registers are undefined, except SS, SP, CS, and DL. CS:IP always points to physical address `0x07C00`. What values CS and IP actually have is not well defined. Some BIOSes use a CS:IP of `0x0000:0x7C00` while others may use `0x07C0:0x0000`. Because boot programs are always loaded at this fixed address, there is no need for a boot program to be relocatable. DL may contain the drive number, as used with [INT 13h](https://en.wikipedia.org/wiki/INT_13H), of the boot device. SS:SP points to a valid stack that is presumably large enough to support hardware interrupts, but otherwise SS and SP are undefined. (A stack must be already set up in order for interrupts to be serviced, and interrupts must be enabled in order for the system timer-tick interrupt, which BIOS always uses at least to maintain the time-of-day count and which it initializes during POST, to be active and for the keyboard to work. The keyboard works even if the BIOS keyboard service is not called; keystrokes are received and placed in the 15-character type-ahead buffer maintained by BIOS.) The boot program must set up its own stack, because the size of the stack set up by BIOS is unknown and its location is likewise variable; although the boot program can investigate the default stack by examining SS:SP, it is easier and shorter to just unconditionally set up a new stack.



# BIOS Interrupt table

https://en.wikipedia.org/wiki/BIOS_interrupt_call



# 相关

https://en.wikipedia.org/wiki/BIOS

https://en.wikipedia.org/wiki/Power-on_self-test

https://wiki.osdev.org/BIOS

https://en.wikipedia.org/wiki/BIOS_interrupt_call