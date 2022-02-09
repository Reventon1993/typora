# CPU



## cpu频率天花板

cpu频率越来越高，目前已达瓶颈:现有工艺决定了4GHZ天花板，目前已经接近。

cpu往多核发展(multicore)、多cpu发展(multiprocessor)。



## multiprocessor system

### SMP

Symmetrical Multi-Processing。



### NUMA

SMP中，每个cpu对每块内存的访问是"一致的"、"对称的"

NUMA，Non Uniform Memory Access，非对称内存访问。在这种架构上，cpu对内存的访问不是"对称的"

物理机(主板布线)被划分成几个区域:numa节点。在同一个numa节点的资源的之间的交互是很快的，远远快于跨numa节点的交互。

numa节点的资源主要包括cpu、内存、pci插槽。最常见的场景就是CPU访问内存。



### MPP



## multicore processor

### multicore

一个CPU由多个core组成

### Hyper-Threading

使用多线程后，一个core被划分成两个thread(线程)。没有开启的话一个core对应一个thread。

### 逻辑CPU

在多核、超线程技术出现后。线程就是cpu最小的独立的处理单元。所有线程也被成为逻辑CPU。



# motherboard&chipset

主板的作用是连接计算机的各个组件，让CPU能访问内存、外设。

CPU通过chipset(芯片组)和内存、外设交流。

## chipset

intel也是chipset的设计商，intel为他的cpu设计适配的chipset。所以cpu和主板有适配的关系。

## 南桥&北桥

南桥和北桥就是chipset。现在北桥已经集成在cpu中了。目前主板上只有南桥chipset。

## 外设接口

CPU通过外设接口(寄存器)、设备存储(比如显存)和外设交互。外设接口就存在于chipset内。

## 频率

外频：主板上的所有元器件之间的通信需要在一个频率上，主板上的晶振生成一个基础频率。所有元器件都统一

以这个频率相互通信，这个就是外频。

主频：每个元器件自己的运行的频率。

倍频：主频 = 外频 * 倍频。倍频是一个倍数。

超频：有的设备的倍频和主板的外频是可以用户自行调高的，高出预设值。这就是超频。

​           这需要元器件和主板的支持，同时也要关闭相应锁频的设置。

睿频：一种cpu的技术。CPU自动去超频，也叫正规超频。intel该技术的名称为Intel Tur[bo](https://product.pconline.com.cn/itbk/diy/vocality/1109/2522431.html) Boost Technology”，

​          简称“Turbo Boost”。这个特性需要在bios中开启。

内存的外频：内存的外频和主板的基础频率是可以不一样的。这个特性叫做内存外频异步工作

​                      外频/基础频率的值称之为内存ratio。常见的ratio有1.333333。

## Z390 chipset

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master/picgo/index.php" alt="img"  />

# GPU

## 显卡&GPU

显卡=VGA(video graphics array)

所谓的VGA口实际是d-sub 15接口，他是vga端子的一种(video graphics array connector),其他的vga端子还有dvi、hdmi接口。

VGA只用于图形显示，加上3D计算，才是GPU。

GPU使用上图的直连PCI插槽



# CMOS

Complementary Metal Oxide Semiconductor,互补金属氧化物半导体

是易失性存储，主板上的电池就是给其供电的。

BIOS会和cmos交互，比如读取系统时间、读取/设置引导顺序。



# BMC&IPMI

https://www.cnblogs.com/baby123/p/14107037.html

简单来说BMC是一个硬件芯片，提供IPMI接口

IPMI的IP通过BMC设置，BMC有个自己独立的网卡，IMPI的IP可以配在这个网卡上，也可以使用共享模式，ip配在其他的网卡上。

![企业微信截图_4c5812f7-b6fb-4004-85ec-369b11dfeddc](https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20210927134034.png)



# DISK

![image-20201224104906218](https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20201224104906.png)

# PCI