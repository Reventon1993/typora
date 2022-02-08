# 前世今生

- 1965前后，贝尔实验室、麻省理工学院、通用发起了Multics计划。Multics是一个“大型”的操作系统(Mulitcs本身就有复杂多数的意思) 。

- 1969前后，贝尔实验室认为Multics不能成功退出了计划 。

- Ken Thompson是贝尔实验室中的一员，在Multics计划中，他受到启发。计划开发一个满足自己需求的小型操作系统。最终他用汇编语言开发出了Unics系统 

- Thompson和Ritchie用C语言重写了Unics,正式改名Unix，Unix版权属于AT&T(贝尔实验室属于AT&T) 。

- 柏克莱大学的Bill Joy修改了Unix内核代码以适应自己的机器，修改后的系统为BSD(后来移植到X86的叫FreeBSD)，这时出现了一些基于Unix开发的系统：System V等等 。Unix一开始是跑在服务器级别的机器上的，System V是第一个支持PC机的(适用于X86) 

- 出于商业的考虑，AT&T收回了版权 。

- 版权收回后，影响最大是使用和学习源码的学生(当时unix只流行在高校中。为了教学，Andre Tanenbaum(谭宁邦)教授在不看Unix代码的情况下，编写了Unix Like的Minix系统 

- 1984年Richard Mathew Stallman开启了GNU计划。计划目的是开发一个自由的GNU操作系统。GUN计划到现在也没有完成。

- 编写OS太难了，GNU计划开始编写运行在Unix上的程序为了支持GNU，Stallman成立了自由软件基金协会(FSF,Free Software Foundation)。GNU中GNU C(GCC)、GNU C Library(glibc)、EMACS、BASH shell是最出名的软件。

- 为了避免GNU所开发的自由软件被他人利用成为专利软件，Stallman与律师草拟了公共许可证(General Public LIcense,GPL),人们成他为”Copyleft“

- Linus_Torvalds改写了Minix，在GNU开始了Linux项目。为了能够兼容Unix,linux遵循POSIX规范，1991年，Linux首次发布。

- 1994年，XFree86(GUI)发布，并被整合到Linux中



# Linux distribution

GNU中的Linux项目仅仅是kernel。

Linux distribution = kernel + tools + software + docs + ...

简单来说Linux distribution是一个整合的linux作为内核的系统

包管理的方式是各大linux发行版本中的一个重要的不同，可以根据包管理方式把所有distributions划分成两大类

| 发行者   |           RPM            |    DPKG     |  其他  |
| -------- | :----------------------: | :---------: | :----: |
| 商业公司 |   RHEL(Red Hat)、SuSE    |   Ubuntu    |        |
| 社区组织 | Fedora、Centos、OpenSuSE | Debian、B2D | Gentoo |



# Linux内核版本

```shel
v3.10.0
主版本号+次版本号+发行版本号
```



