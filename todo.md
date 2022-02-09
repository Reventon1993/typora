gas:帧指针rbp

调用约定：1、资源划分(栈平衡规则)：retn x

​                  2、参数传递

cdecl

内嵌汇编：

c扩展



fat12

簇

![企业微信截图_0a15337c-ddda-465b-adc4-b58907e78f95](https://raw.githubusercontent.com/Reventon1993/pictures/master/picgo/20220209161352.png)

![image-20211116133945854](https://raw.githubusercontent.com/Reventon1993/pictures/master/picgo/20220209161400.png)

![企业微信截图_22fca902-5378-42ff-b706-7672716c9f65](https://raw.githubusercontent.com/Reventon1993/pictures/master/picgo/20220209161409.png)

![企业微信截图_befdfb3f-ca7d-4de6-a658-f15e44dd2d31](https://raw.githubusercontent.com/Reventon1993/pictures/master/picgo/20220209161416.png)





lds



rdi、rsi、rdx、rcx、r8、r9



## 

## NGINX+STREAM



共享库：在装载而不是链接时重定位对共享库中的符号引用。这使得程序模块化。

pic：共享库本身并不解决共享代码的的问题。为了实现共享代码，曾经有一种静态共享库的实现。

​        而现在都是使用动态共享库，其中使用了pic

-shared 编译共享库

-shared -fPIC编译动态共享库

