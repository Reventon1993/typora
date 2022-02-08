# 简介

**Make** is a [build automation](https://en.wikipedia.org/wiki/Build_automation) tool that automatically [builds](https://en.wikipedia.org/wiki/Software_build) [executable programs](https://en.wikipedia.org/wiki/Executable_program) and [libraries](https://en.wikipedia.org/wiki/Library_(software)) from [source code](https://en.wikipedia.org/wiki/Source_code) by reading [files](https://en.wikipedia.org/wiki/File_(computing)) called [Makefiles](https://en.wikipedia.org/wiki/Makefile) which specify how to derive the target program.

# 目的/任务

**make的唯一目的：生成“终极目标”**

# 执行过程

## 两个阶段

- 生成"makefile"：解析众多makefile文件，生成一个"makefile"(最终生成的总的makefile)。内建所有的变量、

​      明确规则、隐含规则，建立所有目标和依赖的关系(链表)。    

- 生成终极目标：根据阶段1的关系链表决定为了生成终极目标需要执行哪些规则(规则的指令)。

## 具体过程

1. 依次读取变量“MAKEFILES”定义的makefile文件列表
2. 读取工作目录下的makefile文件（根据命名的查找顺序“GNUmakefile”，“makefile”，“Makefile”，首先找到那个就读取那个） 
3. 依次读取工作目录makefile文件中使用指示符“include”包含的文件
4. 查找重建所有已读取的makefile文件的规则（如果存在一个目标是当前读取的某一个makefile文件，则执行此规则重建此makefile文件，完成以后从第一步开始重新执行） 
5. 初始化变量值并展开那些需要立即展开的变量和函数并根据预设条件确定执行分支
6. 根据“终极目标”以及其他目标的依赖关系建立依赖关系链表
7. 执行除“终极目标”以外的所有的目标的规则（规则中如果依赖文件中任一个文件的时间戳比目标文件新，则使用规则所定义的命令重建目标文件） 
8. 执行“终极目标”所在的规则 

# 生成"Makefile"

## Makefile入口文件

在运行make指令时，需要一个makefile文件，我称之为"makefile入口文件"

"入口makefile文件"寻找规则&优先级：

**--f/--file参数指定>GNUmakefile(cwd)>makefile(cwd)>Makefile(cwd)**

推荐使用Makefile的命名规则，不推荐GNUmakefile(只有gnu make能识别)



## include

在makefile文件中可以使用**include**命令来加载其他的makefile文件

include的处理过程和C语言中中的include类似。在阶段1中，对include文件的处理就是进行简单的内容替换。

使用相对路径指定include的makefile文件时，makefile文件(上层文件夹)的寻找规则&优先级：

 **pwd>-I/--include-dir参数指定>/usr/gnu/include>/usr/local/include>/usr/include** 



```
include /root/Makefile    #绝对路径

include another_makefile  #相对路径

include file1 file2       #include多个文件

include makefile*         #使用通配符
```



## -include

如果都没有找到makefile文件会报错并结束，使用-include不会报错。

**这种 “-” 的用法也可应用在规则的指令中。**

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20210109191817.png" alt="pic" style="zoom:50%;" />



## 内置变量MAKEFILE_LIST

可以使用内置变量MAKEFILE_LIST查看阶段1完成之后所有解析的makefile文件

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20210109191918.png" alt="1594457518154_4" style="zoom: 67%;" />

## 环境变量MAKEFILES

参考的书籍中说：阶段1中还会去加载环境变量MAKEFILES中指定的makefile文件(不能作为入口文件)，

但在我环境中并没有验证成功:环境变量MAKEFILES并没有起作用。



## makefile文件的重建

Makefile可由其他文件生成，比如RCS或SCCS文件。

todo

# 生成终极目标

## Makefile内容的组成

- **规则**
- **变量**
- **函数**
- **指示符/关键字**
- **注释**
- **隐含规则**
- **关键字**



## 注释

和c语言一样，# 为注释符

## 规则

Makefile中最基本的语句

### 形式

```
目标:/:: 依赖

[Tab]命令

[Tab]命令

[Tab]......
```

**目标是必须的，依赖和命令可以都没有**

### 目标

目标分成单目标、多目标、特殊目标&伪目标(特殊的单目标)

#### 多目标

**多目标规则本质上就是多个单目标规则。多目标规则形式有时会比较复杂，难以理解。**

```
obj1 obj2: dependencies  #两个目标
    commands

等价于两个单目标规则
obj1: dependencies  
    commands

obj2: dependencies  
    commands

obj* : dependencies      #支持通配符
    commands
```

#### 特殊目标&伪目标

在Makefile中，有一些名字，当它们作为规则的目标时，具有特殊含义。它们是一些特殊的目标，GNU make所支持的特殊的目标有： 

- .PHONY：该目标中的依赖都被称为**伪目标**
- .SUFFIXES
- .DEFAULT
- .PRECIOUS
- .INTERMEDIATE
- .SECONDARY
- .DELETE_ON_ERROR
- .IGNORE
- .LOW_RESOLUTION_TIME
- .SILENT
- .EXPORT_ALL_VARIABLES
- .NOTPARALLEL

```
.PHONY: clean            //特殊目标,该规则的依赖都是伪目标

clean:               //伪目标。在存在clean时make clean依旧会执行
    rm *.o
```

**makefile处理伪目标的逻辑和常规的目标不同，具体看下面**



#### 终极目标

终极目标由make命令指定，比如make install, install就是终极目标。如果命令没有指定，那么“makefile”中的第一个目标(不能是特殊目标&伪目标)会成为终极目标。

多目标为第一个目标的场景:

<img src="/Users/jiang/Desktop/pictures/1594460004462_5.png" alt="1594460004462_5" style="zoom:67%;" />

### 依赖

**依赖划分：1常规依赖、order-only依赖**

​                  **2非库依赖、库依赖**

```
obj1: dep1 dep2 //两个常规依赖

obj1: dep1 dep2 | dep3 dep4 //1、2位常规依赖，3、4位order-only依赖
obj1: | dep3 dep4

obj1: dep1 -lcurses   //依赖libcurses库，类似于gcc命令中的依赖库参数。

obj2: | -lcurese //libcurses库也是order-only依赖。未验证这种写法(为验证上述两种依赖的划分是否为正交)

foo.o bar.o : %.o: %.c  //静态模式，通常用于多目标。下面会叙述
```

#### 寻找路径

**VPATH变量**

VPATH = /foo /list  指定两个查找路径

**vpath关键字**

vpath %.c /foo     //所有c文件还需要查找 /foo目录

vpath %.h /lish

vpath %.b /bar

**库依赖查找逻辑**

查看下面的伪代码逻辑



### 命令

**命令一定要以Tab开头,所有Tab开头的都被视为命令**

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20210110215434.png" alt="1594461695379_6" style="zoom: 67%;" />

**#abc也被视为了指令**

![image-20210110215654387](https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20210110215654.png)



指令通常就是shell指令。

可以在指令前加入@前缀，表示不打印

<img src="/Users/jiang/Desktop/pictures/1594461899732_7.png" alt="1594461899732_7" style="zoom:67%;" />

也可以在指令前加入-前缀，忽略指令执行错误继续执行

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20210110215834.png" alt="1594448627502_2" style="zoom: 50%;" />

#### 执行逻辑

**针对单目标规则，以python形式的伪代码来展示执行逻辑**

**多目标的处理实际上就是一条条单目标规则逻辑的处理**

```python
def process_rule(object, dependencies, commands):  //处理目标的逻辑
    if is_phony_object(object):    #是否为伪目标
        for c in commands:
            deal_with_cmd(commands)
        return
    
    flag = 0
    
    found = search_obj(object) //查找目标是否存在
    if not found:
        flag = 1
    for d in dependencies:
        #查找依赖的生成规则，如果没找到且目标不存在报错终止
        try:
            sub_object, sub_dependencies, sub_commands = search_and_get_rule(d)
        except Notfound:
            if not search_obj(d):
                raise error
        result = process_rule(sub_object, sub_dependencies, sub_commands)
        #found的情况下已经flag为1，只需判断not found的场景
        #not found场景下，order-only本身会执行，单结不影响目标本身
        if not found and and not_order_only(d) and result == 1 or newer(d):
            flag = 1
        
    if flag==1:   
        for c in commands:
            deal_with_cmd(commands)
        return 1
            
    return 0

def search_obj(object): #查找目标/依赖的逻辑
    if ls_original_lib(object): #是否为-lcurses形式的库依赖
        dyna = trans_to_dyna(object) //cureses.so,动态库
        static = trans_to_static(object) //cureses.a,静态库
        found = search_obj(dyna) //优先搜索动态库cureses.so
        if found:
            return found
        found = search_obj(static)//动态库搜不到，再搜静态库
        return found
    
    cwd = getcwd()
    found = search_in_path(cwd, object)
    if found:
        return found:
    for path in get_VPATH():      #VPATH变量的路径
        found = search_in_path(path, object)
        if found:
            return found
    for path in get_vpath():  #vpath关键字的路径
        found = search_in_path(path, object)
        if found:
            return found
    if is_lib(object):   #是否为静态库/动态库。多一个查找路径 
        for path in get_lib_default_path():  #["/lib"、"/usr/lib"、"/usr/local/lib"]
            found = search_in_path(path, object)  
            if found:
                return found
    return 0


def search_and_get_rule(object):  #处理同一目标有多个规则时的逻辑。
    rules = serach_all_rules(object)
    if both_single_and_double_rule(rules): #同时存在单冒号和双冒号规则
        raise
    if all_single(rules):
        process_single_rules(rules) 
    else: #all_double
        process_double_rules(rules)
        

多个单引号规则(process_single_rules)：对多个规则合并，依赖叠加，命令使用最后一个规则的命令
多个双引号规则(process_double_rules)：每个rule单独不相互影响的执行，执行规则：rule没有依赖时，一定会执行，
                                      除此之外和普通的单个rule执行的规则一样


def deal_with_cmd(commands):
    for line in commands:                    #命令的每一逻辑行，"\" 为物理行的换行符
        bash_exec(line)                      #开启一个bash进程处理</b>
        
```

### 静态模式

静态模式规则的形式

```
TARGETS ...: TARGET-PATTERN: PREREQ-PATTERNS ...
    COMMANDS
```

"TAGET-PATTERN"和"PREREQ-PATTERNS"说明了如何为每一个目标文件生成依赖文件。从目标模式（TAGET-PATTERN）的目标名字中抽取一部分字符串（称为“茎”）。使用“茎”替代依赖模式（PREREQ-PATTERNS）中的相应部分来产生对应目标的依赖文件

```
objects = foo.o bar.o
$(objects): %.o: %.c
    $(CC) -c $(CFLAGS) $< -o $@
    
相当于如下两条规则：
foo.o : foo.c
    $(CC) -c $(CFLAGS) foo.c -o foo.o
bar.o : bar.c
    $(CC) -c $(CFLAGS) bar.c -o bar.o
```

### 其他

![image-20210110215614280](/Users/jiang/Library/Application Support/typora-user-images/image-20210110215614280.png)

## 隐含规则&自动推导规则

makefile最初设计目的就是实现大型C语言项目的编译与安装。所以他对C语言项目有很好的支持。

makefile存在这隐含规则，当目标不存在规则时，他会尝试用隐含规则去生成。

![1594480781193_8](/Users/jiang/Desktop/pictures/1594480781193_8.png)

默认规则包括:

.c/.o 文件用cc编译生成 目标文件

./c 文件生成用cc编译生成 ./o文件

 

makefile还能对明确的规则进行自动推到，自动加上c文件编译过程中的依赖和命令:

![1594481118372_9](https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20210110224627.png)

### gcc -M / gcc -MM

<img src="https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20210110224821.png" alt="image-20210110224821399" style="zoom: 50%;" />

## 变量

### **引用**

$(var) 或者 ${var}

对于单字符串的内置变量可以不用加括号的引用，比如：$@

### **赋值**

有四种赋值形式

- **var := value**       相当于c语言中的赋值方式，采用直接展开赋值方式

- **var = value**        这种赋值采用的是递归赋值方式

- **var ?= value**      如果var没有赋值，那么var = value,否则什么也不做

- **var += value**      和c语言中的用法一样，但他也采用递推赋值方式

#### **直接展开赋值vs递归赋值**

直接展开赋值就是我们C语言中变量赋值的方式

递归赋值

![1594482172823_10](https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20210110221610.png)

当以一个变量赋值一个变量时，右值变量(a)可以在左值变量(b)之后定义。这时候，

会递归地展开右值变量。这种赋值方式会导致死循环：

![1594482356329_11](https://raw.githubusercontent.com/Reventon1993/pictures/master//picgo/20210110221652.png)

### **嵌套变量**

形式：$($(x)) ，和bash中的嵌套变量使用方式一样

### TODO

内置变量/隐含变量/系统环境变量

目标指定变量/模式指定变量

**override**

## 函数

### **定义**

```
define fun1
        @echo "My name is $(0), param is $(1)"
endef
```

### **调用**

$(FUNCTION ARGUMENTS)  或者 ${FUNCTION ARGUMENTS}

比如：$(subst ee,EE,feet on the street) 

### **内置函数**

大量内置函数是Makefile不易理解的原因之一。

## 关键字/指示符

**vpath**

**define**

**ifeq**

**ifneq**

**ifdef**

**ifndef**

# TODO

**autoconfig/automake**

**cmake**

