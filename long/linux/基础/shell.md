# shell VS terminal VS ......
## 概念

 - **terminal**: 计算机与人交互的输入输出设备(终端)。
 - **tty**: 早期终端大都是打印机(teletype/tty)。所以会用tty指代终端。**早期串口(serial port)大都是连接终端，所以现在的串口设备在unix下汇编抽象为设备，设备路径/dev/ttyS***。
 - **console**:大型机的管理员操作的界面，后与terminal概念逐渐模糊，现在可以认为和terminal为同义词。
 - **terminal emulator**:现在已经没有专门的终端设备了，iterm、xshell等软件提供模拟的terminal。这些软件被称为 terminal。
   **emulator**(终端仿真器/终端模拟器)。
 - **virtual console**：这是一个明确所指的概念：linux在启动时会创建几个终端(终端模拟器的一种)，可以是使用Ctrl+Alt+
   F1-F7进入，这些终端被称为virtual console(虚拟控制台)。
 - **CLI**: command line interface,一个很宽泛的概念，可以指代上面的所有概念。
 - **shell**: bash、ksh、zsh等程序(这些程序提供相对应的shell脚本语言)，以terminal作为标准输入输出。


__可以简单地把上述所有概念归结成两种：terminal(中断)和shell__

## 实现

 - virtual console



```c
                   +-----------------------------------------+
                   |          Kernel                         |
                   |                           +--------+    |       +----------------+ 
 +----------+      |   +-------------------+   |  tty1  |<---------->| User processes |
 | Keyboard |--------->|                   |   +--------+    |       +----------------+
 +----------+      |   | Terminal Emulator |<->|  tty2  |<---------->| User processes |
 | Monitor  |<---------|                   |   +--------+    |       +----------------+
 +----------+      |   +-------------------+   |  tty3  |<---------->| User processes |
                   |                           +--------+    |       +----------------+
                   |                                         |
                   +-----------------------------------------+
```

 - ssh

```c
+----------+       +------------+
| Keyboard |------>|            |
+----------+       |  Terminal  |
| Monitor  |<------|            |
+----------+       +------------+
                         |
                         |  ssh protocol
                         |
                         ↓
                   +------------+
                   |            |
                   | ssh server |--------------------------+
                   |            |           fork           |
                   +------------+                          |
                       |   ↑                               |
                       |   |                               |
                 write |   | read                          |
                       |   |                               |
                 +-----|---|-------------------+           |
                 |     |   |                   |           ↓
                 |     ↓   |      +-------+    |       +-------+
                 |   +--------+   | pts/0 |<---------->| shell |
                 |   |        |   +-------+    |       +-------+
                 |   |  ptmx  |<->| pts/1 |<---------->| shell |
                 |   |        |   +-------+    |       +-------+
                 |   +--------+   | pts/2 |<---------->| shell |
                 |                +-------+    |       +-------+
                 |    Kernel                   |
                 +-----------------------------+
```

 - terminal emulator

```c
+----------+       +------------+
| Keyboard |------>|            |
+----------+       |  Terminal  |--------------------------+
| Monitor  |<------|            |           fork           |
+----------+       +------------+                          |
                       |   ↑                               |
                       |   |                               |
                 write |   | read                          |
                       |   |                               |
                 +-----|---|-------------------+           |
                 |     |   |                   |           ↓
                 |     ↓   |      +-------+    |       +-------+
                 |   +--------+   | pts/0 |<---------->| shell |
                 |   |        |   +-------+    |       +-------+
                 |   |  ptmx  |<->| pts/1 |<---------->| shell |
                 |   |        |   +-------+    |       +-------+
                 |   +--------+   | pts/2 |<---------->| shell |
                 |                +-------+    |       +-------+
                 |    Kernel                   |
                 +-----------------------------+
```
##  参考
https://segmentfault.com/a/1190000016129862  
https://segmentfault.com/a/1190000009082089  
https://www.linuxidc.com/Linux/2017-10/147315.htm  
https://blog.csdn.net/ltx06/article/details/52170852  

# subshell VS child shell
## 概念
两者都被翻译成子shell,其实两者的概念是不同的。
subshell是shell的一个专有概念，child shell是一个通用的概念，表示fork出来的shell进程。
## 使用场景
child shell:
    在shell中使用bash命令即会创建斌进入child shell

```shell    
[root@67483a771b65 /]# bash
```
下面是父子shell的关系
```shell
[root@67483a771b65 /]# pstree -p 175
bash(175)───bash(249)
```

subshell:
**目前没有完全清楚subshell的机制**
创建的方式是把命令用()括起来，()里面的命令即运行在subshell中。当()中有多条命令时，pstree会和正常情况不同。单条命令虽然pstree一样但还是有不同的(BASH_SUBSHELL值不一样，这里是最迷惑的地方)
```shell    
[root@67483a771b65 /]# sleep 10
[root@67483a771b65 /]# pstree -p 175
bash(175)───sleep(264)
[root@67483a771b65 /]# (sleep 10)
[root@67483a771b65 /]# pstree -p 175
bash(175)───sleep(270)

[root@67483a771b65 /]# (echo $BASH_SUBSHELL)
1  
[root@67483a771b65 /]# echo $BASH_SUBSHELL
0

[root@67483a771b65 /]# (sleep 10;echo $BASH_SUBSHELL)
[root@67483a771b65 /]# pstree -p 175 
bash(175)───bash(267)───sleep(268)
[root@67483a771b65 /]# sleep 10;echo $BASH_SUBSHELL;
[root@67483a771b65 /]# pstree -p 175
bash(175)───sleep(272)
```

## BASH_SUBSHELL&SHLVL
  SHLVL表示child shell的嵌套层数
  BASH_SUBSHELL表示subshell的嵌套层数

```shell
[root@67483a771b65 /]# echo $SHLVL
1
[root@67483a771b65 /]# (echo $SHLVL)
1
[root@67483a771b65 /]# echo $BASH_SUBSHELL
0
[root@67483a771b65 /]# (echo $BASH_SUBSHELL)
1
[root@67483a771b65 /]# ((echo $BASH_SUBSHELL);)
2
[root@67483a771b65 /]#
[root@67483a771b65 /]# bash
[root@67483a771b65 /]# echo $SHLVL
2
[root@67483a771b65 /]# (echo $SHLVL)
2
[root@67483a771b65 /]# echo $BASH_SUBSHELL
0
[root@67483a771b65 /]# (echo $BASH_SUBSHELL)
1
[root@67483a771b65 /]# ((echo $BASH_SUBSHELL);)
2
[root@67483a771b65 /]#
```
## 区别&总结
**目前没有完全清楚subshell的机制,并不清楚subshell的使用场景**
两者最大的区别在于:**subshell会继承所有的环境变量(包括局部环境变量)，而child shell只会继承全局变量**

```shell
[root@67483a771b65 /]# abc=111
[root@67483a771b65 /]# (echo $abc)
111
[root@67483a771b65 /]# bash
[root@67483a771b65 /]# echo $abc
```

## 参考
https://www.cnblogs.com/ziyunfei/p/4803832.html

# 环境变量
## 全局变量&局部变量
环境变量分成两种：全局环境变量和局部环境变量
全局变量:env/printenv命令显示全局变量
局部变量:set命令显示所有变量，没有专门显示局部变量的命令
&nbsp;
**export命令是局部变量提升为全局变量**

    [root@67483a771b65 /]# printenv |grep aaa
    [root@67483a771b65 /]# aaa=111
    [root@67483a771b65 /]# printenv |grep aaa
    [root@67483a771b65 /]# export aaa
    [root@67483a771b65 /]# printenv |grep aaa
    aaa=111

**child shell只会继承全局变量，subshell会继承所有变量**

## 系统环境变量

 - 登录时交互shell(虚拟终端,终端仿真器)

 首先，会运行/etc/profile文件，centos系统下文件有如下内容：

```bash
...

for i in /etc/profile.d/*.sh
do
    if [ -r "$i" ]; then
        if [ "${-#*i}" != "$-" ]; then
            . "$i"
        else
            . "$i" >/dev/null
        fi
    fi
done

...
```
profile脚本会执行读取/etc/profile.d路径下的.sh脚本

其次，还会依次寻找如下文件(只执行第一个找到的)：             
\$HOME/.bash_profile 
\$HOME/.bash_login 
\$HOME/.profile 
\centos中,存在root/.bash_profile文件，内容如下：

```shell
# .bash_profile

# Get the aliases and functions
if [ -f ~/.bashrc ]; then
        . ~/.bashrc
fi

# User specific environment and startup programs

PATH=$PATH:/sbin:/usr/sbin:/usr/local/sbin:/usr/bin

export PATH     
```

.bash_profile会去执行~/.bashrc文件

 - 非登录式交互shell

 在shell中运行bash命令会进入非登录式交互shell
只会运行~/.bashrc文件

 - 非交互式shell

 运行shell脚本时会进入非交互式shell
会运行$BASH_ENV文件
