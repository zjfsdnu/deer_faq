### 语法

####  dir

> $(dir <names...>)

- 名称：取目录函数——dir。
- 功能：从文件名序列 <names> 中取出目录部分。目录部分是指最后一个反斜杠（/ ）之前的部分。
如果没有反斜杠，那么返回 ./ 。
- 返回：返回文件名序列 <names> 的目录部分。
- 示例：$(dir src/foo.c hacks) 返回值是 src/ ./ 。


#### notdir

> $(notdir <names...>)

- 名称：取文件函数——notdir。
- 功能：从文件名序列 <names> 中取出非目录部分。非目录部分是指最後一个反斜杠（/ ）之后的部
分。
- 返回：返回文件名序列 <names> 的非目录部分。
- 示例: $(notdir src/foo.c hacks) 返回值是 foo.c hacks 。


#### addprefix

> $(addprefix <prefix>,<names...>)

- 名称：加前缀函数——addprefix。
- 功能：把前缀 <prefix> 加到 <names> 中的每个单词前面。
- 返回：返回加过前缀的文件名序列。
- 示例：$(addprefix src/,foo bar) 返回值是 src/foo src/bar 。


####  sort

> $(sort <list>)

- 名称：排序函数
- 功能：给字符串 <list> 中的单词排序（升序）。
- 返回：返回排序后的字符串。
- 示例：$(sort foo bar lose) 返回 bar foo lose 。
- 备注：sort 函数会去掉 <list> 中相同的单词。


####  文件搜寻

在一些大的工程中，有大量的源文件，我们通常的做法是把这许多的源文件分类，并存放在不同的
目录中。所以，当 make 需要去找寻文件的依赖关系时，你可以在文件前加上路径，但最好的方法是把一
个路径告诉 make，让 make 在自动去找。


Makefile 文件中的特殊变量 VPATH 就是完成这个功能的，如果没有指明这个变量，make 只会在当
前的目录中去找寻依赖文件和目标文件。如果定义了这个变量，那么，make 就会在当前目录找不到的情
况下，到所指定的目录中去找寻文件了。

> VPATH = src:../headers

上面的定义指定两个目录，“src”和“../headers”，make 会按照这个顺序进行搜索。目录由“冒号”
分隔。（当然，当前目录永远是最高优先搜索的地方）

另一个设置文件搜索路径的方法是使用 make 的“vpath”关键字（注意，它是全小写的），这不是变
量，这是一个 make 的关键字，这和上面提到的那个 VPATH 变量很类似，但是它更为灵活。它可以指
定不同的文件在不同的搜索目录中。这是一个很灵活的功能。它的使用方法有三种：

```
vpath <pattern> <directories> 为符合模式 <pattern> 的文件指定搜索目录 <directories>。
vpath <pattern> 清除符合模式 <pattern> 的文件的搜索目录。
vpath 清除所有已被设置好了的文件搜索目录。
```

vpath 使用方法中的 <pattern> 需要包含 % 字符。% 的意思是匹配零或若干字符，（需引用 % ，使
用 \ ）例如，%.h 表示所有以 .h 结尾的文件。<pattern> 指定了要搜索的文件集，而 <directories> 则
指定了 < pattern> 的文件集的搜索的目录。例如：
``` makefile
vpath %.h ../headers
```
该语句表示，要求 make 在“../headers”目录下搜索所有以 .h 结尾的文件。（如果某文件在当前目
录没有找到的话）

我们可以连续地使用 vpath 语句，以指定不同搜索策略。如果连续的 vpath 语句中出现了相同的
<pattern> ，或是被重复了的 <pattern>，那么，make 会按照 vpath 语句的先后顺序来执行搜索。如：
``` makefile
vpath %.c foo
vpath % blish
vpath %.c bar
```
其表示 .c 结尾的文件，先在“foo”目录，然后是“blish”，最后是“bar”目录。
``` makefile
vpath %.c foo:bar
vpath % blish
```
而上面的语句则表示 .c 结尾的文件，先在“foo”目录，然后是“bar”目录，最后才是“blish”目录。




