CPU架构的发展

![image-20210717164258567](https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20210717164258.png)



多个CPU的的架构

- SMP：Symmetrical Multi-Processing(对称多处理)，每个CPU地位一样，共享内存和总线结构
- NUMA：Non-Uniform Memory Access(非统一内存访问)。计算机硬件(主板上的插槽)被划分成几个区域(主要是cpu和内存，好像也有pci设备？)。CPU访问同一个区域的内存会很快，不同区域的会很慢
- MPP：暂不清楚

       Non-Uniform  Memory Access (NUMA) refers to multiprocessor systems whose memory is divided into multiple memory nodes.  The access time
       of a memory node depends on the relative locations of the accessing CPU and the accessed node.  (This contrasts with a symmetric multi‐
       processor  system,  where  the  access time for all of the memory is the same for all CPUs.)  Normally, each CPU on a NUMA system has a
       local memory node whose contents can be accessed faster than the memory in the node local to another CPU or the memory on a bus  shared
       by all CPUs.

多核CPU的相关概念：

多核CPU:Multi-core processor

核：core

线程：thread

超线程：一个core对应两个线程

每个线程是一个独立的逻辑处理单元，所以也称之为逻辑CPU



缓存

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20210720160511.png" alt="image-20210720160511867" style="zoom: 50%;" />

intel缓存架构

![image-20210717184007547](https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20210717184007.png)

L3可以进行绑核操作:

绑核是拆分组里的行

具体查看intel rdt技术：

https://software.intel.com/content/www/cn/zh/develop/articles/intel-resource-director-technology-rdt-reference-manual.html



各种频率

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



linux相关指令

![](https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20210717180334.png)

![image-20210717181020899](https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20210717181021.png)











