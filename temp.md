

openflow&sdn

centos 7 网卡做bond：https://www.cnblogs.com/yxy-linux/p/8327818.html

linux ip命令创建vlan:https://blog.csdn.net/weixin_42441072/article/details/116585086

[state machines](https://en.wikipedia.org/wiki/State_machine)

wireshark&tcpdump

fsck

fdisk

smartctl

as -m32

ld -melf_i386

![image-20210129014641831](https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20210129014648.png)



![image-20210129175544564](https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20210129175544.png)

![image-20210201200302593](https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20210201200302.png)

```
ssh -Nf -R 3240:localhost:3240 root@192.168.15.24
```



big real mode：

进入保护模式，加载数据段寄存器，再退出保护模式，不再重载数据段寄存器。此时，cpu处理实模式，但却拥有32的数据寻址能力。

作用：既拥有32数据寻址能力，又可以使用实模式的中断程序(为加载os之前的bios中断程序)

```assembly
        push    ax
        in      al,     92h
        or      al,     00000010b
        out     92h,    al
        pop     ax

        cli

        db      0x66
        lgdt    [GdtPtr]

        mov     eax,    cr0
        or      eax,    1
        mov     cr0,    eax

        mov     ax,     SelectorData32
        mov     fs,     ax
        mov     eax,    cr0
        and     al,     11111110b
        mov     cr0,    eax

        sti
```



进入保护模式：

1.open A20

2.cti

3.load gdtr

4.置位cr0.PE

5.jmp far

6.重载数据段寄存器

7.sti(需要完成IDT初始化)

后续可选：

初始化idt，之后可sti

初始化ldt

开启分页：1load cr3 2置位cr0.PG



进入长模式

1.cti

2.load gdtr

3.若已开启分页，复位cr0.PG,关闭分页

4.置位cr4.PAE,开启PAE

5.load cr3

6.置位IA32_EREF.LME，开启IA-32E

7.置位cr0.PG，开启分页，此时cpu会检测目前处于长模式，自动置位IA32_EREF.LMA



后续可选:

初始化idt，之后可sti

初始化ldt















![image-20211007153524473](/Users/jiang/Library/Application Support/typora-user-images/image-20211007153524473.png)





![image-20211008100831214](/Users/jiang/Library/Application Support/typora-user-images/image-20211008100831214.png)

页表、page_init、一致性页表、allocate_page



;|----------------------|
;|      100000 ~ END    |
;|         KERNEL       |
;|----------------------|
;|      E0000 ~ 100000  |
;| Extended System BIOS |
;|----------------------|
;|      C0000 ~ Dffff   |
;|     Expansion Area   |
;|----------------------|
;|      A0000 ~ bffff   |
;|   Legacy Video Area  |
;|----------------------|
;|      9f000 ~ A0000   |
;|       BIOS reserve   |
;|----------------------|
;|      90000 ~ 9f000   |
;|       kernel tmpbuf  |
;|----------------------|
;|      10000 ~ 90000   |
;|         LOADER       |
;|----------------------|

;|      8000 ~ 10000    |
;|        VBE info      |
;|----------------------|
;|      7e00 ~ 8000     |
;|        mem info      |
;|----------------------|
;|      7c00 ~ 7e00     |
;|       MBR (BOOT)     |
;|----------------------|
;|      0000 ~ 7c00     |
;|       BIOS Code      |
;|----------------------|



![image-20220203233528429](https://raw.githubusercontent.com/Reventon1993/pictures/master/picgo/image-20220203233528429.png)

![image-20220203233609732](/Users/jiangchuan/Library/Application Support/typora-user-images/image-20220203233609732.png)





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
