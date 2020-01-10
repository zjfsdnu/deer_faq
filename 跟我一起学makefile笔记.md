# 2 介绍



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

|                             参数                             |                  描述                  |
| :----------------------------------------------------------: | :------------------------------------: |
|             -n, --just-print, --dry-run, --recon             |        不执行参数，只是打印命令        |
|                         -t, --touch                          | 把目标文件的时间更新，但不更改目标文件 |
|                        -q, --question                        |  如果目标不存在，会打印出一条出错信息  |
| -W <file>, --what-if=<file>, --assume-new=<file>, --new-file=<file> |                                        |
|                            -p -v                             |                                        |

### make 的参数

|                             参数                             |                             描述                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|                            -b, -m                            |                 忽略和其它版本 make 的兼容性                 |
|                      -B, --always-make                       |              认为所有的目标都需要更新（重编译）              |
|                 -C <dir>, --directory=<dir>                  |                   指定读取 makefile 的目录                   |
|                      -debug[=<options>]                      | 输出 make 的调试信息<br />a: 也就是 all，输出所有的调试信息<br />b: 也就是 basic，只输出简单的调试信息。即输出不需要重编译的目标<br /> |
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
| -W <file>, --what-if=<file>, --new-file=<file>, --assume-new=<file> |                                                              |
|                  --warn-undefined-variables                  |       只要 make 发现有未定义的变量，那么就输出警告信息       |



# 9 隐含规则

