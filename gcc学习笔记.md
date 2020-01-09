## GNU编译器套件

GCC = GNU Compiler Collection

## GCC的组成部分以及使用到的软件



|   程序    |                             描述                             |
| :-------: | :----------------------------------------------------------: |
| addr2line | 给出一个可执行文件的内部地址，addr2line 使用文件中的调试信息将地址翻泽成源代码文件名和行号。该程序是 binutils 包的一部分 |
|    ar     | 可通过从文档中增加、删除和析取文件来维护库文件。通常使用该工具是为了创建和管理连接程序使用的目标库文档。该程序是 binutils 包的一部分 |
|    as     | GNU 汇编器。实际上它是一族汇编器，因为它可以被编译或能够在各种不同平台上工作。 该程序是 binutils 包的一部分 |
|    c++    | gcc 的一个版本，默认语言设置为 [C++](http://c.biancheng.net/cplus/)，而且在连接的时候自动包含标准 C++ 库。这和 g++ 一样 |
|  c++filt  | 程序接受被 C++ 编泽程序转换过的名字（不是被重载的），而且将该名字翻译成初始形式。 该程序是 binutils 包的一部分 |
|    cc1    |                       实际的C编译程序                        |
|  cc1plus  |                     实际的 C++ 编译程序                      |
| collect2  | 在不使用 GNU 连接程序的系统上，有必要运行 collect2 来产生特定的全局初始化代码（例如 C++ 的构造函数和析构函数） |
|  crt0.o   | 这个初始化和结束代码是为每个系统定制的，而且也被编译进该文件，该文件然后会被连接到每个可执行文件中来执行必要的启动和终止程序 |
|    g++    | gcc 的一个版本，默认语言设置为 [C++](http://c.biancheng.net/cplus/)，而且在连接的时候自动包含标准 C++ 库。这和 c++ 一样 |
|    gcc    |    该驱动程序等同于执行编译程序和连接程序以产生需要的输出    |
|   gcov    | gprof 使用的配置工具，用来确定程序运行的时候哪一部分耗时最大 |
|    gdb    |          GNU 调试器，可用于检查程序运行时的值和行为          |
|   gprof   | 该程序会监督编译程序的执行过程，并报告程序中各个函数的运行时间，可以根据所提供的配置文件来优化程序。该程序是 binutils 包的一部分 |
|    ld     | GNU 连接程序。该程序将目标文件的集合组合成可执行程序。该程序是 binutils 包的一部分 |
| libstdc++ |   运行时库，包括定义为标准语言一部分的所有的 C++ 类和函数    |
|   make    | 一个工具程序，它会读 makefile 脚本来确定程序中的哪个部分需要编译和连接，然后发布必要的命令。它读出的脚本（叫做 makefile 或 Makefile）定义了文件关系和依赖关系 |
|    nm     |    列出目标文件中定义的符号。该程序是 binutils 包的一部分    |
|  objcopy  | 将目标文件从一种二进制格式复制和翻译到另外一种。该程序是 binutils 包的一部分 |
|  objdump  | 显示一个或多个目标文件中保存的多种不同信息。该程序是 binutils 包的一部分 |
|  ranlib   | 创建和添加到 ar 文档的索引。该索引被 Id 使用来定位库中的模块。该程序是 binutils 包的一部分 |
|  readelf  | 从 ELF 格式的目标文件显示信息。该程序是 binutils 包的一部分  |
|   size    | 列出目标文件中每个部分的名字和尺寸。该程序是 binutils 包的一部分 |
|  strings  | 浏览所有类型的文件，析取出用于显示的字符串。该程序是 binutils 包的一部分 |
|   strip   | 从目标文件或文档库中去掉符号表，以及其他调试所需的信息。该程序是 binutils 包的一部分 |
|  windres  |    Window 资源文件编译程序。该程序是 binutils 包的一部分     |



## 检测是否已经安装GCC编译器

> cc --verion

或者

> gcc --version

#### 安装GCC

http://sourceforge.net/projects/mingw

http://sourceforge.net/projects/mingw-w64

http://gcc.gnu.org/install/binaries.html

http://win-builds.org/doku.php



## GCC编译C语言程序

```c
#include <stdio.h>

int main()
{
    puts("C语言中文网");
    return 0;
}
```

```shell
$gcc main.c
```

linux下默认生成 a.out文件

```shell
$gcc main.c -o main
```



## GCC分步骤编译C语言程序

1. #### 编译（Compile）

   ```shell
   gcc -c main.c
   ```

   对于微软编译器（内嵌在 Visual [C++](http://c.biancheng.net/cplus/) 或者 Visual Studio 中），目标文件的后缀为`.obj`；对于 [GCC](http://c.biancheng.net/gcc/) 编译器，目标文件的后缀为`.o`

   一个源文件会生成一个目标文件，多个源文件会生成多个目标文件，源文件数目和目标文件数目是一样的。通常情况下，默认的目标文件名字和源文件名字是一样的

   ```shell
   gcc -c main.c -o a.o
   ```

2. #### 链接（Link）

   ```shell
   gcc main.o
   ```

   

## GCC编译流程

GCC 编译器在编译一个C语言程序时需要经过以下 4 步：

1. 将C语言源程序预处理，生成`.i`文件
2. 预处理后的.i文件编译成为汇编语言，生成`.s`文件
3. 将汇编语言文件经过汇编，生成目标文件`.o`文件  
4. 将各个模块的`.o`文件链接起来生成一个可执行程序文件



## GCC常用选项

#### -S

将C语言源文件编译为[汇编语言](http://c.biancheng.net/asm/)，但是并不汇编该程序。使用该选项，我们可以查看C语言代码对应的汇编代码

#### -E 选项

`-E`选项将C语言源文件进行预处理，但是并不编译该程序

#### -I 选项

由于指定包含的头文件的目录，这一点对于大型的代码组织来说是很有用的

#### -g 选项

`-g`选项可生成能被 gdb 调试器所使用的调试信息。只有使用了该选项后生成的可执行文件，才带有程序中引用的符号表。这时 gdb 调试程序才能对该可执行程序进行调试

#### -save-temps

当使用该选项时，GCC 会正常地编译和链接，但是会把预处理器输出、汇编语言和对象文件全部存储在当前目录下。使用 -save-temps 选项所生成的中间文件，与对应的源文件具有相同的文件名，但文件扩展名分别为.i、.s和.o，分别表示为预处理输出、汇编语言输出和对象文件



## GCC -E选项：生成预处理文件

预处理器操作：宏展开、文件包含、删除部分代码等

默认情况下，预处理器的输出会被导入到标准输出流（也就是显示器），可以利用`-o`选项把它导入到某个输出文件：

```shell
$gcc -E ciicle.c -o circle.i
```

使用`-C`选项可以阻止预处理器删除源文件和头文件中的注释：

```shell
$gcc -E -C circle.c -o circle.i
```

下面是 GCC 预处理器阶段常用的选项：

#### -Dname[=definition]

在处理源文件之前，先定义宏 name。宏 name 必须是在源文件和头文件中都没有被定义过的。将该选项搭配源代码中的`#ifdef name`命令使用，可以实现条件式编译。如果没有指定一个替换的值，该宏被定义为值 1。

#### -Uname

如果在命令行或 GCC 默认设置中定义过宏 name，则“取消”name 的定义。`-D`和`-U`选项会依据在命令行中出现的先后顺序进行处理。

#### -Idirectory[:directory[...]]

当通过 #include 命令把所需的头文件包括进源代码中时，除系统标准 include 目录之外，指定其他的目录对这些头文件进行搜索。

#### -iquote directory[:directory[...]]

这是在最近 GCC 版本中新增的选项，它为在 #include 命令中采用引号而非尖括号指定的头文件指定搜索目录。

#### -isystem directory[:directory[...]]

该选项在标准系统 include 目录以外为系统头文件指定搜索目录，且它指定的目录优先于标准系统 include 目录被搜索。在目录说明开头位置的等号，被视作系统根目录的占位符，可以使用`--sysroot`或`-isysroot`选项来修改它。

#### -isysroot directory

该选项指定搜索头文件时的系统根目录。例如，如果编译器通常在 /usr/include 目录及其子目录下搜索系统头文件，则该选项将引导到 directory/usr/include 及其子目录下进行搜索。

> `--sysroot`选项，采用一个连字符替代 i，它为链接库搜索而不是头文件搜索指定系统根目录以外的目录。如果 isysroot 不可用，则 sysroot 既为头文件又为链接库搜索指定目录。

#### -I-

在较新版本的 GCC 中，该选项被`-iquote`替代。在旧版本中，该选项用于将命令行的所有`-Idirectory`选项分割为两组。所有在`-I-`左边加上`-I`选项的目录，被视为等同于采用`-iquote`选项；这指的是，它们只对 #include 命令中采用引号的头文件名进行搜索。

所有在`-I-`右边加上`-I`选项的目录，将对所有 #include 命令中的头文件名进行搜索，无论文件名是在引号还是尖括号中。

而且，如果命令行中出现了`-I-`，那么包括源文件本身的目录不再自动作为搜索头文件的目录。



## GCC -S选项：生成汇编文件

如果想把C语言变量的名称作为汇编语言语句中的注释，可以加上`-fverbose-asm`选项：

```shell
$ gcc -S -fverbose-asm circle.c
```



## GCC -l选项：手动添加链接库

链接器把多个二进制的目标文件（object file）链接成一个单独的可执行文件。在链接过程中，它必须把符号（变量名、函数名等一些列标识符）用对应的数据的内存地址（变量地址、函数地址等）替代，以完成程序中多个模块的外部引用。

而且，链接器也必须将程序中所用到的所有C标准库函数加入其中。对于链接器而言，链接库不过是一个具有许多目标文件的集合，它们在一个文件中以方便处理。

> 当把程序链接到一个链接库时，只会链接程序所用到的函数的目标文件。在已编译的目标文件之外，如果创建自己的链接库，可以使用 ar 命令。

标准库的大部分函数通常放在文件 libc.a 中（文件名后缀`.a`代表“achieve”，译为“获取”），或者放在用于共享的动态链接文件 libc.so 中（文件名后缀`.so`代表“share object”，译为“共享对象”）。这些链接库一般位于 /lib/ 或 /usr/lib/，或者位于 GCC 默认搜索的其他目录。

当使用 GCC 编译和链接程序时，GCC 默认会链接 libc.a 或者 libc.so，但是对于其他的库（例如非标准库、第三方库等），就需要手动添加。

令人惊讶的是，标准头文件 <math.h> 对应的数学库默认也不会被链接，如果没有手动将它添加进来，就会发生函数未定义错误。

GCC 的`-l`选项可以让我们手动添加链接库。下面我们编写一个数学程序 main.c，并使用到了 [cos](http://c.biancheng.net/ref/cos.html)() 函数，它位于 <math.h> 头文件。

```c
#include <stdio.h>
#include <math.h>
#define PI 3.14159265

int main()
{
    double param, result;
    param = 60.0;
    result = cos(param * PI / 180.0);
    printf("The cosine of %f degrees is %f.\n", param, result);
    return 0;
}
```

为了编译这个 main.c，必须使用`-l`选项，以链接数学库：

```shell
$ gcc main.c -o main.out -lm
```

数学库的文件名是 libm.a。前缀`lib`和后缀`.a`是标准的，`m`是基本名称，GCC 会在`-l`选项后紧跟着的基本名称的基础上自动添加这些前缀、后缀，本例中，基本名称为 m。

> 在支持动态链接的系统上，GCC 自动使用在 Darwin 上的共享链接库 libm.so 或 libm.dylib。



#### 链接其它目录中的库

通常，GCC 会自动在标准库目录中搜索文件，例如 /usr/lib，如果想链接其它目录中的库，就得特别指明。有三种方式可以链接在 GCC 搜索路径以外的链接库，下面我们分别讲解。

1) 把链接库作为一般的目标文件，为 GCC 指定该链接库的完整路径与文件名。

例如，如果链接库名为 libm.a，并且位于 /usr/lib 目录，那么下面的命令会让 GCC 编译 main.c，然后将 libm.a 链接到 main.o：

```shell
$ gcc main.c -o main.out /usr/lib/libm.a
```

2) 使用`-L`选项，为 GCC 增加另一个搜索链接库的目录：

```shell
$ gcc main.c -o main.out -L/usr/lib -lm
```

可以使用多个`-L`选项，或者在一个`-L`选项内使用冒号分割的路径列表。

3) 把包括所需链接库的目录加到环境变量 LIBRARYPATH 中。



## GCC生成动态链接库（.so文件）：-shared和-fPIC选项

[Linux](http://c.biancheng.net/linux_tutorial/)下动态链接库（shared object file，共享对象文件）的文件后缀为`.so`，它是一种特殊的目标文件（object file），可以在程序运行时被加载（链接）进来。使用动态链接库的优点是：程序的可执行文件更小，便于程序的模块化以及更新，同时，有效内存的使用效率更高。

#### [GCC](http://c.biancheng.net/gcc/) 生成动态链接库

如果想创建一个动态链接库，可以使用 GCC 的`-shared`选项。输入文件可以是源文件、汇编文件或者目标文件。

另外还得结合`-fPIC`选项。-fPIC 选项作用于编译阶段，告诉编译器产生与位置无关代码（Position-Independent Code）；这样一来，产生的代码中就没有绝对地址了，全部使用相对地址，所以代码可以被加载器加载到内存的任意位置，都可以正确的执行。这正是共享库所要求的，共享库被加载时，在内存的位置不是固定的。

例如，从源文件生成动态链接库：

```shell
$ gcc -fPIC -shared func.c -o libfunc.so
```

从目标文件生成动态链接库：

```shell
$ gcc -fPIC -c func.c -o func.o
$ gcc -shared func.o -o libfunc.so
```

-fPIC 选项作用于编译阶段，在生成目标文件时就得使用该选项，以生成位置无关的代码。

#### GCC 将动态链接库链接到可执行文件

如果希望将一个动态链接库链接到可执行文件，那么需要在命令行中列出动态链接库的名称，具体方式和普通的源文件、目标文件一样。请看下面的例子：

```shell
$ gcc main.c libfunc.so -o app.out
```

将 main.c 和 libfunc.so 一起编译成 app.out，当 app.out 运行时，会动态地加载链接库 libfunc.so。

当然，必须要确保程序在运行时可以找到这个动态链接库。你可以将链接库放到标准目录下，例如 /usr/lib，或者设置一个合适的环境变量，例如 LIBRARY_PATH。不同系统，具有不同的加载链接库的方法。