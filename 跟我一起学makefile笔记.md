# 2 makefile介绍

我们的规则是：

1. 如果这个工程没有编译过，那么我们的所有 c 文件都要编译并被链接
2. 如果这个工程的某几个 c 文件被修改，那么我们只编译被修改的 c 文件，并链接目标程序
3. 如果这个工程的头文件被改变了，那么我们需要编译引用了这几个头文件的 c 文件，并链接目标程序  

### makefile 的规则

```makefile
target ... : prerequisites ...
	command
	...
	...
```

`target` ： 规则的目标。通常是最后需要生成的文件名或者为了实现这个目的而必需的中间过程文件名。可以是.o 文件、也可以是最后的可执行程序的文件名等。另外，目标也可以是一个 make执行的动作的名称，如目标“clean”，我们称这样的目标是“伪目标”  

`prerequisites`：规则的依赖。生成规则目标所需要的文件名列表。通常一个目标依赖于一个或者多个文件

`command`：规则的命令行。是规则所要执行的动作（任意的 shell 命令或者是可在 shell 下执行的程序）。它限定了 make 执行这条规则时所需要的动作  

一个规则可以有多个命令行，每一条命令占一行。
注意：每一个命令行必须以[Tab]字符开始， [Tab]字符告诉 make 此行是一个命令行。 make 按照命令完成相应的动作。这也是书写 Makefile 中容易产生，而且比较隐蔽的错误  

命令就是在任何一个目标的依赖文件发生变化后重建目标的动作描述。一个目标可以没有依赖而只有动作（指定的命令）。比如 Makefile 中的目标“clean”，此目标没有依赖，只有命令。它所定义的命令用来删除 make 过程产生的中间文件（进行清理工作）。
在 Makefile 中“规则”就是描述在什么情况下、如何重建规则的目标文件，通常规则中包括了
目标的依赖关系（目标的依赖文件）和重建目标的命令。 make 执行重建目标的命令，来创建或者
重建规则的目标（此目标文件也可以是触发这个规则的上一个规则中的依赖文件）。 **规则包含了文件之间的依赖关系和更新此规则目标所需要的命令**  

### 简单的示例

```makefile
edit : main.o kbd.o command.o display.o \
	insert.o search.o files.o utils.o
	cc -o edit main.o kbd.o command.o display.o \
		insert.o search.o files.o utils.o
main.o : main.c defs.h
	cc -c main.c
kbd.o : kbd.c defs.h command.h
	cc -c kbd.c
command.o : command.c defs.h command.h
	cc -c command.c
display.o : display.c defs.h buffer.h
	cc -c display.c
insert.o : insert.c defs.h buffer.h
	cc -c insert.c
search.o : search.c defs.h buffer.h
	cc -c search.c
files.o : files.c defs.h buffer.h command.h
	cc -c files.c
utils.o : utils.c defs.h
	cc -c utils.c
clean :
	rm edit main.o kbd.o command.o display.o \
		insert.o search.o files.o utils.o
```

可以将一个较长行使用反斜线（\）来分解为多行，这样可以使我们的 Makefile 书写清晰、容易阅读理解。但需要注意：反斜线之后不能有空格  

**命令行必需以 [Tab] 键开始，以和 Makefile其他行区别。就是说所有的命令行必需以 [Tab] 字符开始，但并不是所有的以 [Tab] 键出现行都是命令行。但 make 程序会把出现在第一条规则之后的所有以 [Tab] 字符开始的行都作为命令行来处理**

**Makefile 中把那些没有任何依赖只有执行动作的目标称为“伪目标”（phony targets）**   

### make 如何工作 

对于一个 Makefile 文件，“make”首先解析终极目标所在的规则（上节例子中的第一个规则），根据其依赖文件依次（按照依赖文件列表从左到右的顺序）寻找创建这些依赖文件的规则。首先为第一个依赖文件寻找创建规则，如果第一个依赖文件依赖于其它文件，则同样为这个依赖文件寻找创建规则……，直到为所有的依赖文件找到合适的创建规则。之后 make 从最后一个规则回退开始执行，最终完成终极目标的第一个依赖文件的创建和更新。之后对第二个、第三个、第四个……终极目标的依赖文件执行同样的过程

### 指定变量

```makefile
objects = main.o kbd.o command.o display.o \
	insert.o search.o files.o utils.o
edit : $(objects)
	cc -o edit $(objects)
……
……
clean :
	rm edit $(objects)
```

### 自动推导规则

```makefile
# sample Makefile
objects = main.o kbd.o command.o display.o \
	insert.o search.o files.o utils.o

edit : $(objects)
	cc -o edit $(objects)

main.o : defs.h
kbd.o : defs.h command.h
command.o : defs.h command.h
display.o : defs.h buffer.h
insert.o : defs.h buffer.h
search.o : defs.h buffer.h
files.o : defs.h buffer.h command.h
utils.o : defs.h

.PHONY : clean
clean :
	rm edit $(objects)
```

### 另类风格的 makefile

```makefile
#sample Makefile
objects = main.o kbd.o command.o display.o \
	insert.o search.o files.o utils.o

edit : $(objects)
	cc -o edit $(objects)

$(objects) : defs.h
kbd.o command.o files.o : command.h
display.o insert.o search.o files.o : buffer.h
```

书写规则建议的方式是： **单目标，多依赖**。就是说尽量要做到一个规则中只存在一个目标文件，可有多个依赖文件。尽量避免多目标，单依赖的方式。这样后期维护也会非常方便，而且 Makefile会更清晰、明了

### 清除工作目录过程文件

```makefile
.PHONY : clean
clean :
	-rm edit $(objects)
```

1. 通过“.PHONY”特殊目标将“clean”目标声明为伪目标。避免当磁盘上存在一个名为“clean”文件时，目标“clean”所在规则的命令无法执行
2.  在命令行之前使用“-”，意思是忽略命令“rm”的执行错误

### Makefile 的内容

在一个完整的 Makefile 中，包含了 5 个东西：显式规则、隐含规则、变量定义、指示符和注释。
关于“规则”、“变量”和“Makefile 指示符”将在后续的章节进行详细的讨论。本章讨论的是一些基本概念
• 显式规则：它描述了在何种情况下如何更新一个或者多个被称为目标的文件（Makefile 的目标文件）。书写 Makefile 时需要明确地给出目标文件、目标的依赖文件列表以及更新目标文件所需要的命令（有些规则没有命令，这样的规则只是纯粹的描述了文件之间的依赖关系）
• 隐含规则：它是 make 根据一类目标文件（典型的是根据文件名的后缀）而自动推导出来的规则。 make 根据目标文件的名，自动产生目标的依赖文件并使用默认的命令来对目标进行更新（建立一个规则）
• 变量定义：使用一个字符或字符串代表一段文本串，当定义了一个变量以后， Makefile 后续在需要使用此文本串的地方，通过引用这个变量来实现对文本串的使用。第一章的例子中，我们就定义了一个变量“objects”来表示一个.o 文件列表
• Makefile 指示符：指示符指明在 make 程序读取 makefile 文件过程中所要执行的一个动作。其中包括：
– 读取一个文件，读取给定文件名的文件，将其内容作为 makefile 文件的一部分
– 决定（通常是根据一个变量的得值）处理或者忽略 Makefile 中的某一特定部分
– 定义一个多行变量
• 注释： Makefile 中“#”字符后的内容被作为是注释内容（和 shell 脚本一样）处理。如果此行的第一个非空字符为“#”，那么此行为注释行。注释行的结尾如果存在反斜线（\），那么下一行也被作为注释行。一般在书写 Makefile 时推荐将注释作为一个独立的行，而不要和 Makefile的有效行放在一行中书写。当在 Makefile 中需要使用字符“#”时，可以使用反斜线加# `\#`来实现
**需要注意的地方：**
Makefile 中第一个规则之后的所有以 [Tab] 字符开始的的行， make 程序都会将其交给系统 shell程序去解释执行。因此，以 [Tab] 字符开始的注释行也会被交给 shell 来处理，此命令行是否需要被执行（shell 执行或者忽略）是由系统 shell 程序来判决的

### makefile 文件的命名

默认的情况下， make 会在工作目录（执行 make 的目录）下按照文件名顺序寻找 makefile 文件读取并执行，查找的文件名顺序为：`GNUmakefile`、`makefile`、`Makefile`
通常应该使用“`makefile`或者`Makefile`作为一个 makefile 的文件名（我们推荐使用`Makefile`，首字母大写而比较显著，一般在一个目录中和当前目录的一些重要文件（`README`, `Changelist`等）靠近，在寻找时会比较容易的发现它）  

当通过`-f`或者`–file`指定 make 读取 makefile 的文件时， make就不再自动查找这三个标准命名的 makefile 文件  

### 包含其它 makefile 文件

```makefile
include foo.make *.mk $(bar)
-include <filename>
sinclude <filename>
```

filename是 shell 所支持的文件名（可以使用通配符）。指示符“include”所在的行可以一个或者多个空格开始，切忌不能以 [Tab] 字符开始。指示符“include”和文件名之间、多个文件之间使用空格或者 [Tab] 键隔开。行尾的空白字符在处理时被忽略

### 变量 MAKEFILES

如果在当前环境定义了一个“MAKEFILES”环境变量， make 执行时首先将此变量的值作为需要读入的 Makefile 文件，多个文件之间使用空格分开  

1. 环境变量指定的 makefile 文件中的“目标”不会被作为 make 执行的“终极目标”  
2. 环境变量定义的包含文件是否存在不会导致 make 错误  
3. make 在执行时，首先读取的是环境变量“MAKEFILES”所指定的文件列表，之后才是工作目录下的 makefile 文件，“include”所指定的文件是在 make 发现此关键字的时、暂停正在读取的文件而转去读取“include”所指定的文件

实际应用中很少设置此变量  推荐的做法是：在需要包含其它 makefile 文件时使用指示符`include`来实现

### 总结

make 的执行过程如下：

1. 依次读取变量“MAKEFILES”定义的 makefile 文件列表。
2. 读取工作目录下的 makefile 文件（根据命名的查找顺序“GNUmakefile”，“makefile”，“Makefile”，首先找到那个就读取那个）。
3. 依次读取工作目录 makefile 文件中使用指示符“include”包含的文件。
4. 查找重建所有已读取的 makefile 文件的规则（如果存在一个目标是当前读取的某一个 makefile
文件，则执行此规则重建此 makefile 文件，完成以后从第一步开始重新执行）。
5. 初始化变量值并展开那些需要立即展开的变量和函数并根据预设条件确定执行分支。
6. 根据“终极目标”以及其他目标的依赖关系建立依赖关系链表。
7. 执行除“终极目标”以外的所有的目标的规则（规则中如果依赖文件中任一个文件的时间戳比目标文件新，则使用规则所定义的命令重建目标文件）。
8. 执行“终极目标”所在的规则。

执行一个规则的过程是这样的：对于一个存在的规则（明确规则和隐含规则）首先， make 程序将比较目标文件和所有的依赖文件的时间戳。如果目标的时间戳比所有依赖文件的时间戳更新（依赖文件在上一次执行 make 之后没有被修改），那么什么也不做。否则（依赖文件中的某一个或者全部在上一次执行 make 后已经被修改过），规则所定义的重建目标的命令将会被执行。这就是 make工作的基础，也是其执行规制所定义命令的依据



# 3 书写规则



### 自动生成依赖性



# 4 书写命令

命令以`tab`键开头

### 显示命令

前面加@字符 不会显示命令本身

`make -n（--just-print）`只显示命令 不执行

`make -s（--silent或--quiet）`不显示命令

### 命令执行

命令放在一行 分号分割 才能依赖

``` makefile
exec:
	cd /home/zjf; pwd
```

### 命令出错

命令退出码非0表示出错 终止执行当前规则

前面加-字符 忽略错误

```makefile
clean:
	-rm -f *.o
```

`make -i (--ignore-errors)` 忽略错误

以`.IGNORE`做为目标 忽略错误

`make -k（--keep-going）`出错终止该规则 继续执行其他规则

### 嵌套执行make

总控makefile

```makefile
subsystem:
	cd subdir && $(MAKE)
```

等价于

```makefile
subsystem:
	$(MAKE) -C subdir
```

```makefile
export v1 = value1
export v2 := value2
export v3 += value3
```

单独一个export表示传递所有变量

------

SHELL和MAKEFLAGS 总是传递

`-C` `-f` `-h` `-o` `-W` 参数不传递

------

`make -w` 输出当前工作目录

使用 `-C`参数时 `-w`自动打开（有`-s`或`--no-print-directory`时失效）

### 定义命令包

```makefile
define run-yacc
yacc $(firstword $^)
mv y.tab.c $@
endef
```

```makefile
foo.c: foo.y
	$(run-yacc)
```

# 5 使用变量

变量代表一个字符串 执行时会自动展开

大小写敏感

### 变量的基础

声明时赋初值 使用前面加`$`  然后用小括号或者大括号包起来保证安全

```makefile
objects = program.o foo.o utils.o
program: $(objects)
	cc -o program $(objects)

$(objects): defs.h
```

### 变量中的变量

可以使用后面定义的变量

 ```makefile
foo = $(bar)
bar = $(ugh)
ugh = Huh?

all:
	echo $(foo)
 ```

```makefile
A = $(B)
B = $(A)
```

会陷入无限的变量展开过程中 make会报错

```makefile
x := foo
y := $(x) bar
x := later
```

等价于

```makefile
y := foo bar
x := later
```

这种方法不能使用后面的变量

```makefile
ifeq (0, ${MAKELEVEL})
	cur-dir   := $(shell pwd)
	whoami    := $(shell whoami)
	host-type := $(shell arch)
	MAKE := ${MAKE} host-type=${host-type} whoami=${whoami}
endif
```

`MAKELEVEL`表示当前调用的层数

------

```makefile
nullstring :=
space := $(nullstring) # end of the line
```

`nullstring`是一个`Empty`变量 表示变量的值开始了 后面的#表示变量的终止

------

```makefile
FOO ?= bar
```

等价于

```makefile
ifeq ($(origin FOO), undefined)
	FOO = bar
endif
```

### 变量高级用法

##### 变量值的替换

格式为：

```makefile
$(var:a=b)
${var:a=b}
```

把变量`var`中所有以`a`结尾的子串替换成`b`子串

```makefile
foo := a.o b.o c.o
bar := $(foo:.o=.c)
```

##### 把变量的值再当成变量

```makefile
x = y
y = z
a := $($(x))
```

`$(a)`的值是`z`

```makefile
ifdef do_sort
	func := sort
else
	func := stip
endif

bar := a d b g q c
foo := $($(func) $(bar))
```

### 追加变量值

```makefile
objects = main.o foo.o bar.o utils.o
objects += another.o
```

### override指示符

覆盖命令行参数设置的变量

```makefile
override <variable>; = <value>;
override <variable>; := <value>;

override define foo
bar
endef
```

### 多行变量

命令要以`tab`键开头

```makefile
define two-lines
	echo foo
	echo $(bar)
endef
```

### 环境变量

环境变量会被makefile中的变量和命令行带入的变量覆盖（`make -e` 则不会被覆盖）

嵌套调用时上层变量以环境变量传到下一层 使用`export`关键字

### 目标变量

普通变量属于全局变量

`$<`属于规则型变量

局部变量

```makefile
prog: CFLAGS = -g
prog: prog.o foo.o bar.o
	$(CC) $(CFLAGS) prog.o foo.o bar.o
prog.o: prog.c
	$(CC) $(CFLAGS) prog.c
foo.o: foo.c
	$(CC) $(CFLAGS) foo.c
bar.o: bar.c
	$(CC) $(CFLAGS) bar.c
```

### 模式变量

```makefile
%.o: CFLAGS = -O
```



# 6 使用条件判断

### 示例

```makefile
libs_for_gcc = -lgpu
normal_libs = 

ifeq ($(CC), gcc)
	libs = $(libs_for_gcc)
else
	libs = $(normal_libs)
endif

foo: $(objects)
	$(CC) -o foo $(objects) $(libs)
```

### 语法

4个关键字

`ifeq` `ifneq` `ifdef` `ifndef`

```makefile
bar = 
foo = $(bar)
ifdef foo
	frobozz = yes
else
	frobozz = no
endif
```

```makefile
foo = 
ifdef foo
	frobozz = yes
else
	frobozz = no
endif
```

第一个`$(frobozz)`是yes 第二个是no

不要把自动化变量放入条件表达式中



# 7 使用函数

### 函数的调用语法

```makefile
$(<function> <arguments>)
${<function> <arguments>}
```

```makefile
comma := ,
empty :=
space := $(empty) $(empty)
foo := a b c
bar := $(subst $(space),$(comma),$(foo))
```

### 字符串处理函数

**字符串替换函数**

```makefile
$(subst <from>,<to>,<text>)
$(subst ee,EE,feet on the street)
```

**模式字符串替换函数**

```makefile
$(patsubst <pattern>,<replacement>,<text>)
$(patsubst %.c,%.o,x.c c.c bar.c)
```

**去空格函数**

```makefile
$(strip <string>)
$(strip a b c)
```

**查找字符串函数**

```makefile
$(findstring <find>,<in>)
$(findstring a,a b c)
$(findstring a,b c)
```

如果找到 返回<find> 否则返回空字符串

**过滤函数**

```makefile
$(filter <pattern...>,<text>)

sources := foo.c bar.c baz.s ugh.h
foo: $(sources)
	cc $(filter %.c %s,$(sources)) -o foo
```

以<pattern>模式过滤<text>字符串中的单词，保留符合模式的单词 可以有多个模式

**反过滤函数**

```makefile
$(filter-out <pattern...>,<text>)
```

**排序函数**

```makefile
$(sort <list>)
```

会去掉重复的单词

**取单词函数**

```makefile
$(word <n>,<text>)
$(word 2,foo bar baz)
```

取字符串<text>中第<n>个单词 从1开始

**取单词串函数**

```makefile
$(wordlist <ss>,<e>,<text>)
$(wordlist 2,3,foo bar baz)
```

从字符串<text>中取从<ss>开始到<e>的单词串 <ss>和<e>是一个数字

**单词个数统计函数**

```makefile
$(words <text>)
$(words foo bar baz)
$(word $(words <text>),<text>)
```

**首单词函数**

```makefile
$(firstword <text>)
$(firstword foo bar)
```

综合示例

```makefile
override CFLAGS += $(patsubst %,-I%,$(subst :, ,$(VPATH)))
```

### 文件名操作函数

**取目录函数**

```makefile
$(dir <names...>)
$(dir src/foo.c hacks) #返回src/ ./
```

**取文件函数**

```makefile
$(notdir <names...>)
$(notdir src/foo.c hacks) #返回foo.c hacks
```

**取后缀函数**

```makefile
$(suffix <names...>)
$(suffix src/foo.c src-1.0/bar.c hacks) #返回.c .c
```

**取前缀函数**

```makefile
$(basename <names...>)
$(basename src/foo.c src-1.0/bar.c hacks) #返回src/foo src-1.0/bar hacks
```

**加后缀函数**

```makefile
$(addsuffix <suffix>,<names...>)
$(addsuffix .c,foo bar)
```

**加前缀函数**

```makefile
$(addprefix <prefix>,<names...>)
$(addprefix src/,foo bar)
```

**连接函数**

```makefile
$(join <list1>,<list2>)
$(join aaa bbb, 111 222 333) #返回aaa111 bbb222 333
```

### foreach函数

```makefile
$(foreach <var>,<list>,<text>)

names := a b c d
files := $(foreach n,$(names),$(n).o)
#返回` a.o b.o c.o d.o`
```

参数 <var> 的作用域只在 foreach 函数当中

### if 函数

```makefile
$(if <condition>,<then-part>)
$(if <condition>,<then-part>,<else-part>)
```

如果 <condition> 为真（非空字符串），那个 <then-part> 会是整个函数
的返回值，如果 <condition> 为假（空字符串），那么 <else-part> 会是整个函数的返回值，此时如果<else-part> 没有被定义，那么，整个函数返回空字串

### call函数

用来创建新的参数化的函数

```makefile
$(call <expression>,<param1>,<param2>,...,<paramn>)

reverse = $(1) $(2)
foo = $(call reverse,a,b) #a b

reverse = $(2) $(1)
foo = $(call reverse,a,b) #b a
```

### origin 函数

```makefile
$(origin <variable>)

ifdef bletch
	ifeq "$(origin bletch)" "environment"
		bletch = barf, gag, etc.
	endif
endif
```

 origin 函数的返回值

|    返回值    |                 描述                 |
| :----------: | :----------------------------------: |
|  undefined   |            从来没有定义过            |
|   default    |           比如“CC”这个变量           |
| environment  | 是一个环境变量 并且-e 参数没有被打开 |
|     file     |         被定义在 Makefile 中         |
| command line |             被命令行定义             |
|   override   |      被 override 指示符重新定义      |
|  automatic   |     是一个命令运行中的自动化变量     |

### shell 函数

```makefile
contents := $(shell cat foo)
files := $(shell echo *.c)
```

### 控制 make 的函数

```makefile
$(error <text ...>) # 产生一个致命的错误
$(warning <text ...>) # 输出一段警告信息

ifdef ERROR_001
	$(error error is $(ERROR_001))
endif

ERR = $(error found an error!)

.PHONY: err

err: $(ERR)
```



# 8 make的运行

### make 的退出码

| 退出码 |                           描述                           |
| :----: | :------------------------------------------------------: |
|   0    |                         成功执行                         |
|   1    |                    运行时出现任何错误                    |
|   2    | 使用了 make 的“-q”选项，并且 make 使得一些目标不需要更新 |

### 指定 Makefile

```shell
make –f hchen.mk
make --file hchen.mk
```

### 指定目标

`MAKECMDGOALS` 存放你所指定的终极目标的列表

```makefile
sources = foo.c bar.c
ifneq ( $(MAKECMDGOALS),clean)
	include $(sources:.c=.d)
endif
```

```makefile
.PHONY: all
all: prog1 prog2 prog3 prog4
```

### 检查规则

|                 参数                 |                  描述                  |
| :----------------------------------: | :------------------------------------: |
| -n, --just-print, --dry-run, --recon |        不执行参数，只是打印命令        |
|             -t, --touch              | 把目标文件的时间更新，但不更改目标文件 |
|            -q, --question            |  如果目标不存在，会打印出一条出错信息  |

### make 的参数

|                             参数                             |                             描述                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|                            -b, -m                            |                 忽略和其它版本 make 的兼容性                 |
|                      -B, --always-make                       |              认为所有的目标都需要更新（重编译）              |
|                 -C <dir>, --directory=<dir>                  |                   指定读取 makefile 的目录                   |
|                      -debug[=<options>]                      | 输出 make 的调试信息<br />a: 也就是 all，输出所有的调试信息<br />b: 也就是 basic，只输出简单的调试信息。即输出不需要重编译的目标 |
|                              -d                              |                       相当于“–debug=a”                       |
|                 -e, --environment-overrides                  |        指明环境变量的值覆盖 makefile 中定义的变量的值        |
|         -f <file>, --file=<file>, --makefile=<file>          |                   指定需要执行的 makefile                    |
|                          -h, --help                          |                         显示帮助信息                         |
|                     -i , --ignore-errors                     |                    在执行时忽略所有的错误                    |
|                -I <dir>, --include-dir=<dir>                 |              指定一个被包含 makefile 的搜索目标              |
|                      -j [N], --jobs[=N]                      |                      同时运行命令的个数                      |
|                       -k, --keep-going                       |                       出错也不停止运行                       |
|    -l <load>, --load-average[=<load>], -max-load[=<load>]    |                   指定 make 运行命令的负载                   |
|             -n, --just-print, --dry-run, --recon             |            仅输出执行过程中的命令序列，但并不执行            |
|      -o <file>, --old-file=<file>, --assume-old=<file>       |   不重新生成的指定的 <file>，即使这个目标的依赖文件新于它    |
|                    -p, --print-data-base                     |       输出 makefile 中的所有数据，包括所有的规则和变量       |
|                        -q, --question                        |   不运行命令，也不输出。仅仅是检查所指定的目标是否需要更新   |
|                    -r, --no-builtin-rules                    |                  禁止 make 使用任何隐含规则                  |
|                  -R, --no-builtin-variabes                   |           禁止 make 使用任何作用于变量上的隐含规则           |
|                    -s, --silent, --quiet                     |                 在命令运行时不输出命令的输出                 |
|                 -S, --no-keep-going, --stop                  |                      取消“-k”选项的作用                      |
|                         -t, --touch                          | 相当于 UNIX 的 touch 命令，只是把目标的修改日期变成最新的，也就是阻止生成目标的命令运行 |
|                        -v, --version                         |         输出 make 程序的版本、版权等关于 make 的信息         |
|                    -w, --print-directory                     |            输出运行 makefile 之前和之后的当前目录            |
|                     --no-print-directory                     |                         禁止“-w”选项                         |
| -W <file>, --what-if=<file>, --new-file=<file>, --assume-new=<file> | 设定文件“FILE”的时间戳为当前时间，但不改变文件实际的最后修改时间 |
|                  --warn-undefined-variables                  |       只要 make 发现有未定义的变量，那么就输出警告信息       |



# 9 隐含规则

内嵌变量 `CFLAGS` 代表了 `gcc` 编译器编译源文件的编译选项

### 使用隐含规则

隐含规则所提供的依赖文件只是一个最基本的（通常它们之间的对应关系为：“EXENAME.o”对应“EXENAME.c”、“EXENAME”对应于“EXENAME.o”）

```makefile
foo : foo.o bar.o
	cc -o foo foo.o bar.o $(CFLAGS) $(LDFLAGS)
```

隐含规则是：

```makefile
foo.o : foo.c
	cc –c foo.c $(CFLAGS)
bar.o : bar.c
	cc –c bar.c $(CFLAGS)
```

```makefile
# 要使用pascal编译器必须明确指出编译命令
foo.o: foo.p
	pc $< -o $@
```

```makefile
# sample Makefile

CUR_DIR = $(shell pwd)
INCS := $(CUR_DIR)/include
CFLAGS := -Wall –I$(INCS)

EXEF := foo bar

.PHONY : all clean
all : $(EXEF)

foo : CFLAGS+=-O2
bar : CFLAGS+=-g

clean :
	$(RM) *.o *.d $(EXES)
```

### 隐含规则一览

make 的默认后缀列表为：

`.out、.a、.ln、.o、.c、.cc、.C、.p、.f、.F、.r、.y、.l、.s、.S、.mod、.sym、.def、.h、.info、.dvi、.tex、.texinfo、.texi、txinfo、.w、.ch、.web、.sh、.elc、el`

|          隐含规则          | 依赖目标                       | 命令                                                         |
| :------------------------: | :----------------------------- | ------------------------------------------------------------ |
|         编译C 程序         | .o<-.c                         | $(CC) -c $(CPPFLAGS) $(CFLAGS)                               |
|        编译C++ 程序        | .o<-.cc或.C                    | $(CXX) -c $(CPPFLAGS) $(CFLAGS)                              |
|      编译Pascal 程序       | .o<-.p                         | $(PC) -c $(PFLAGS)                                           |
|  编译Fortran/Ratfor 程序   | .o<-.r<br />.o<-.F<br />.o<-.f | $(FC) –c $(FFLAGS) $(RFLAGS)<br />$(FC) –c $(FFLAGS) $(CPPFLAGS)<br />$(FC) –c $(FFLAGS) |
| 预处理Fortran/Ratfor 程序  | .o<-.r<br />.o<-.F             | $(FC) –F $(FFLAGS) $(RFLAGS)<br />$(FC) –F $(FFLAGS) $(CPPFLAGS) |
|     编译Modula-2 程序      | .sym<-.def<br />.o<-.mod       | $(M2C) $(M2FLAGS) $(DEFFLAGS)<br />$(M2C) $(M2FLAGS) $(MODFLAGS) |
| 汇编和需要预处理的汇编程序 | .o<-.s<br />.s<-.S             | $(AS) $(ASFLAGS)<br />$(CPP) $(CPPFLAGS)                     |
|   链接单一的object 文件    | <-.o                           | $(CC) $(LDFLAGS) N.o $(LOADLIBES) $(LDLIBS)                  |
|        Yacc C 程序         | .c<-.y                         | $(YACC) $(YFLAGS)                                            |
|         Lex C 程序         | .c<-.l                         | $(LEX) $(LFLAGS)”                                            |

### 隐含规则使用的变量

**关于命令的变量**

| 变量名   | 对应命令 | 描述                                       |
| -------- | -------- | ------------------------------------------ |
| AR       | ar       | 函数库打包程序 可创建静态库.a 文档         |
| AS       | as       | 汇编程序                                   |
| CC       | cc       | C 编译程序                                 |
| CXX      | g++      | C++ 编译程序                               |
| CO       | co       | 从RCS 中提取文件的程序                     |
| CPP      | $(CC) -E | C 程序的预处理器（输出是标准输出设备）     |
| FC       | f77      | Fortran 和Ratfor 的编译器和预处理程序      |
| GET      | get      | 从SCCS 中提取文件程序                      |
| LEX      | lex      | 将Lex 语言转变为C 或Ratfor 的程序          |
| PC       | pc       | Pascal 语言编译程序                        |
| YACC     | yacc     | Yacc 文法分析器（针对于C 程序）            |
| YACCR    | yacc –r  | Yacc 文法分析器（针对于Ratfor 程序）       |
| MAKEINFO | makeinfo | 转换Texinfo 源文件（.texi）到Info 文件程序 |
| TEX      | tex      | 从TeX 源文件创建TeX DVI 文件的程序         |
| TEXI2DVI | texi2dvi | 从Texinfo 源文件创建TeX DVI 文件的程序     |
| WEAVE    | weave    | 转换Web 到TeX 的程序                       |
| CWEAVE   | cweave   | 转换C Web 到TeX 的程序                     |
| TANGLE   | tangle   | 转换Web 到Pascal 语言的程序                |
| CTANGLE  | ctangle  | 转换C Web 到C                              |
| RM       | rm -f    | 删除文件命令                               |

**关于命令参数的变量**

| 变量名   | 默认值 | 描述                                             |
| -------- | ------ | ------------------------------------------------ |
| ARFLAGS  | rv     | 函数库打包程序AR 命令的参数                      |
| ASFLAGS  |        | 汇编语言编译器参数（当明显地调用.s 或.S 文件时） |
| CFLAGS   |        | C 语言编译器参数                                 |
| CXXFLAGS |        | C++ 语言编译器参数                               |
| COFLAGS  |        | RCS 命令参数                                     |
| CPPFLAGS |        | C 预处理器参数（C 和Fortran 编译器也会用到）     |
| FFLAGS   |        | Fortran 语言编译器参数                           |
| GFLAGS   |        | SCCS “get”程序参数                               |
| LDFLAGS  |        | 链接器参数                                       |
| LFLAGS   |        | Lex 文法分析器参数                               |
| PFLAGS   |        | Pascal 语言编译器参数                            |
| RFLAGS   |        | Ratfor 程序的Fortran 编译器参数                  |
| YFLAGS   |        | Yacc 文法分析器参数                              |

### 隐含规则链

除非中间的目标不存在， 才会引发中间规则

只要目标成功产生，所产生的中间目标文件会被以 `rm -f` 删除

使用特殊目标 `.INTERMEDIATE`来指除将那些文件作为中间过程文件来处理

需要保留的文件作为特殊目标 `.SECONDARY` 的依赖文件罗列

需要保留所有.o 的中间过程文件，可以将.o 文件的模式（%.o）作为特殊目标 `.PRECIOUS` 的依赖

同一个隐含规则在一个“链”中只能出现一次

Make会优化一些特殊的隐含规则，而不生成中间文件

### 定义模式规则

变量和函数的展开发生在 make 载入 Makefile 时，而模式规则中的 % 则发生在运行时  

**模式规则介绍**

模式规则中，至少在规则的目标定义中要包含 `%` 

如果 % 定义在目标中，那么，目标中的 % 的值决定了依赖目标中的 % 的值  

**模式规则示例**

```makefile
%.o : %.c
	$(CC) -c $(CFLAGS) $(CPPFLAGS) $< -o $@

%.tab.c %.tab.h: %.y
	bison -d $<
```

**自动化变量**

| 自动化变量 | 含义                                           | 补充                                                         |
| ---------- | ---------------------------------------------- | ------------------------------------------------------------ |
| $@         | 规则中的目标文件集                             |                                                              |
| $%         | 仅当目标是函数库文件中，表示规则中的目标成员名 | 如果一个目标是 foo.a(bar.o)，那么， `$%` 就是 bar.o ， `$@` 就是 foo.a 。如果目标不是函数库文件（Unix 下是 .a ， Windows下是 .lib ），那么，其值为空 |
| $<         | 规则的第一个依赖文件名                         | 如果是一个目标文件使用隐含规则来重建，则它代表由隐含规则加入的第一个依赖文件 |
| $?         | 所有比目标文件更新的依赖文件列表，空格分割     | 如果目标是静态库文件名，代表的是库成员（.o 文件）            |
| $^         | 规则的所有依赖文件列表，使用空格分隔           | 变量`$^`会去掉重复的依赖文件                                 |
| $+         | 类似“$^”                                       | 但是它保留了依赖文件中重复出现的文件。主要用在程序链接时库的交叉引用场合 |
| $*         | 表示目标模式中 % 及其之前的部分                | 如果目标是 `dir/a.foo.b` ，并且目标的模式是`a.%.b` ，那么， `$*` 的值就是 `dir/a.foo` |

```makefile
lib : foo.o bar.o lose.o win.o
	ar r lib $?
```

**模式的匹配**

通常，模式规则中目标模式由前缀、后缀、模式字符“%”组成，这三个部分允许两个同时为空。实际文件名应该是以模式指定的前缀开始、后缀结束的任何文件名。文件名中除前缀和后缀以外的所有部分称之为“茎”（模式字符“%”可以代表若干字符。因此：模式“%.o”所匹配的文件“test.c”中“test”就是“茎”）。模式规则中依赖文件名的确定过程是：首先根据规则定义的目标模式匹配实际的目标文件，确定“茎”，之后使用“茎”替代规则依赖文件名中的模式字符“%”，生成依赖文件名。这样就产生了一个明确指定了目标和依赖文件的规则。例如模式规则：`%.o : %.c`，当“test.o”需要重建时将形成规则`test.o : test.c`

当目标模式中包含斜杠（目录部分）。在进行目标文件匹配时，文件名中包含的目录字符串在匹配之前被移除，只进行基本文件名的匹配；匹配成功后，再将目录部分加入到匹配之后的字符串之前形成“茎”。来看一个例子：例如目标模式为“e%t”，文件“src/eat”匹配这个模式，那么“茎”就是“src/a”；模式规则中依赖文件的产生：首先使用“茎”中的非目录部分（“a”）替代依赖文件中的模式字符“%”，之后再将目录部分（“src/”）加入到形成的依赖文件名之前构成依赖文件的全路径名。这里如果模式规则的依赖模式为`c%r`，则那么目标`src/eat`对应的依赖文件就为`src/car`  

**重载内建隐含规则**

```makefile
%.o : %.c
	$(CC) $(CFLAGS) –D__DEBUG__ $< -o $@

%.o : %.s # 取消了编译.s 文件的内嵌规则
```

### 老式风格的“后缀规则”

后缀规则是一种古老定义隐含规则的方式，在新版本的 make 中使用模式规则作为对它的替代，模式规则相比后缀规则更加清晰明了。在现在版本中保留它的原因是为了能够兼容旧的 makefile 文件。后缀规则有两种类型：“双后缀”和“单后缀”  

双后缀规则定义了一对后缀：目标文件的后缀和依赖目标（源文件）的后缀。如 `.c.o` 相当于 `%o :%c` 。单后缀规则只定义一个后缀，也就是源文件的后缀。如 `.c` 相当于 `% : %.c` 

后缀规则中所定义的后缀应该是 make 所认识的，如果一个后缀是 make 所认识的，那么这个规则
就是单后缀规则，而如果两个连在一起的后缀都被 make 所认识，那就是双后缀规则  

```makefile
.c.o:
	$(CC) -c $(CFLAGS) $(CPPFLAGS) -o $@ $
	
.SUFFIXES: # 删除默认的后缀
.SUFFIXES: .c .o .h # 定义自己的后缀
```

make 的参数 `-r` 或 `-no-builtin-rules` 也会使用得默认的后缀列表为空。而变量 SUFFIXE 被用来定义默认的后缀列表，你可以用 .SUFFIXES 来改变后缀列表  

### 隐含规则搜索算法

比如我们有一个目标叫 T。下面是搜索目标 T 的规则的算法。请注意，在下面，我们没有提到后缀规则，原因是，所有的后缀规则在 Makefile 被载入内存时，会被转换成模式规则。如果目标是`archive(member)` 的函数库文件模式，那么这个算法会被运行两次，第一次是找目标 T，如果没有找到的话，那么进入第二次，第二次会把`member` 当作 T 来搜索。

1. 把 T 的目录部分分离出来。叫 D，而剩余部分叫 N。（如：如果 T 是 `src/foo.o` ，那么， D 就是`src/` ， N 就是 `foo.o` ）
2. 创建所有匹配于 T 或是 N 的模式规则列表。
3. 如果在模式规则列表中有匹配所有文件的模式，如 % ，那么从列表中移除其它的模式。
4. 移除列表中没有命令的规则。
5. 对于第一个在列表中的模式规则：
      1. 推导其“茎” S， S 应该是 T 或是 N 匹配于模式中 % 非空的部分。
      2. 计算依赖文件。把依赖文件中的 % 都替换成“茎” S。如果目标模式中没有包含斜框字符，而把 D 加在第一个依赖文件的开头。
      3. 测试是否所有的依赖文件都存在或是理当存在。（如果有一个文件被定义成另外一个规则的目标文件，或者是一个显式规则的依赖文件，那么这个文件就叫“理当存在”）
      4. 如果所有的依赖文件存在或是理当存在，或是就没有依赖文件。那么这条规则将被采用，退出该算法。
6. 如果经过第 5 步，没有模式规则被找到，那么就做更进一步的搜索。对于存在于列表中的第一个模式规则：
      1. 如果规则是终止规则，那就忽略它，继续下一条模式规则。
      2. 计算依赖文件。（同第 5 步）
      3. 测试所有的依赖文件是否存在或是理当存在。
      4. 对于不存在的依赖文件，递归调用这个算法查找他是否可以被隐含规则找到。
      5. 如果所有的依赖文件存在或是理当存在，或是就根本没有依赖文件。那么这条规则被采用，退出该算法。
      6. 如果没有隐含规则可以使用，查看 `.DEFAULT` 规则，如果有，采用，把 `.DEFAULT` 的命令给 T使用。

一旦规则被找到，就会执行其相当的命令，而此时，我们的自动化变量的值才会生成。  



# 10 使用make 更新函数库文件

函数库文件也就是对Object 文件（程序编译的中间文件）的打包文件。在Unix 下，一般是由命令 `ar` 来完成打包工作

### 函数库文件的成员

```makefile
foolib(hack.o): hack.o
	ar cr foolib hack.o
	
foolib(hack.o kludge.o)

foolib(*.o)
```

### 函数库成员的隐含规则

```shell
make foo.a(bar.o)
# 等价于
cc -c bar.c -o bar.o
ar r foo.a bar.o
rm -f bar.o
```

`$%`

### 函数库文件的后缀规则

```makefile
.c.a:
    $(CC) $(CFLAGS) $(CPPFLAGS) -c $< -o $*.o
    $(AR) r $@ $*.o
    $(RM) $*.o
# 等价于
(%.o) : %.c
    $(CC) $(CFLAGS) $(CPPFLAGS) -c $< -o $*.o
    $(AR) r $@ $*.o
    $(RM) $*.o
```

### 注意事项

请小心使用make的并行机制（ `-j` 参数）







