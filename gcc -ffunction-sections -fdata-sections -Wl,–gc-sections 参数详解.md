**背景**

有时我们的程序会定义一些暂时使用不上的功能和函数，虽然我们不使用这些功能和函数，但它们往往会浪费我们的ROM和RAM的空间。这在使用静态库时，体现的更为严重。有时，我们只使用了静态库仅有的几个功能，但是系统默认会自动把整个静态库全部链接到可执行程序中，造成可执行程序的大小大大增加。

**参数详解**

为了解决前面分析的问题，我们引入了标题中的几个参数。GCC链接操作是以section作为最小的处理单元，只要一个section中的某个符号被引用，该section就会被加入到可执行程序中去。因此，GCC在编译时可以使用 -ffunction-sections 和 -fdata-sections 将每个函数或符号创建为一个sections，其中每个sections名与function或data名保持一致。而在链接阶段， -Wl,–-gc-sections 指示链接器去掉不用的section（其中-wl, 表示后面的参数 -gc-sections 传递给链接器），这样就能减少最终的可执行程序的大小了。

我们常常使用下面的配置启用这个功能：

```makefile
CFLAGS += -ffunction-sections -fdata-sections
LDFLAGS += -Wl,--gc-sections
```

　　

# 例子

main.c 文件如下：

```c
#include <stdio.h>
 
int fun_0(void)
{
    printf("%s: %d\n", __FUNCTION__, __LINE__);
    return 0;
}
 
int fun_1(void)
{
    printf("%s: %d\n", __FUNCTION__, __LINE__);
    return 0;
}
 
int fun_2(void)
{
    printf("%s: %d\n", __FUNCTION__, __LINE__);
    return 0;
}
 
int fun_3(void)
{
    printf("%s: %d\n", __FUNCTION__, __LINE__);
    return 0;
}
 
void main(void)
{
	fun_0();
    fun_3();
}
```

　　Makefile文件如下：

```makefile
main_sections:
	gcc -ffunction-sections -fdata-sections -c main.c
	gcc -Wl,--gc-sections -o $@ main.o
 
main_normal:
	gcc -c main.c
	gcc -o $@ main.o
 
clean:
	-rm -rf *.o main_sections main_normal
```

　　

# 验证

## 运行

```shell
$ make main_sections
gcc -ffunction-sections -fdata-sections -c main.c
gcc -Wl,--gc-sections -o main_sections main.o
$ make main_normal
gcc -c main.c
gcc -o main_normal main.o
```

　　

## 比较大小

```shell
$ ll main_*
-rwxrwxrwx 1 8592 Jan  9 17:04 main_normal*
-rwxrwxrwx 1 8336 Jan  9 16:56 main_sections*
```

　　

可以看见使用该功能的二进制文件要小于不使用该功能的二进制文件

## 分析sections

```shell
$ make clean
rm -rf *.o main_sections main_normal
$ make main_sections
gcc -ffunction-sections -fdata-sections -c main.c
gcc -Wl,--gc-sections -o main_sections main.o
$ readelf -t main.o
...
  [ 5] .text.fun_0
       PROGBITS               PROGBITS         0000000000000000  0000000000000048  0
       0000000000000028 0000000000000000  0                 1
       [0000000000000006]: ALLOC, EXEC
  [ 6] .rela.text.fun_0
       RELA                   RELA             0000000000000000  0000000000000540  23
       0000000000000048 0000000000000018  5                 8
       [0000000000000040]: INFO LINK
  [ 7] .text.fun_1
       PROGBITS               PROGBITS         0000000000000000  0000000000000070  0
       0000000000000028 0000000000000000  0                 1
       [0000000000000006]: ALLOC, EXEC
  [ 8] .rela.text.fun_1
       RELA                   RELA             0000000000000000  0000000000000588  23
       0000000000000048 0000000000000018  7                 8
       [0000000000000040]: INFO LINK
  [ 9] .text.fun_2
       PROGBITS               PROGBITS         0000000000000000  0000000000000098  0
       0000000000000028 0000000000000000  0                 1
       [0000000000000006]: ALLOC, EXEC
  [10] .rela.text.fun_2
       RELA                   RELA             0000000000000000  00000000000005d0  23
       0000000000000048 0000000000000018  9                 8
       [0000000000000040]: INFO LINK
  [11] .text.fun_3
       PROGBITS               PROGBITS         0000000000000000  00000000000000c0  0
       0000000000000028 0000000000000000  0                 1
       [0000000000000006]: ALLOC, EXEC
  [12] .rela.text.fun_3
       RELA                   RELA             0000000000000000  0000000000000618  23
       0000000000000048 0000000000000018  11                8
       [0000000000000040]: INFO LINK
```

　　

从object文件中可以发现，fun_0 ~ fun_3 每个函数都是一个独立的section. 
而如果使用 make main_normal 生成的object文件，则共享一个默认的sections（.text）。

## 分析elf文件

```shell
$ readelf -a main_normal | grep fun_
    48: 000000000000064a    40 FUNC    GLOBAL DEFAULT   14 fun_0
    51: 000000000000069a    40 FUNC    GLOBAL DEFAULT   14 fun_2
    61: 00000000000006c2    40 FUNC    GLOBAL DEFAULT   14 fun_3
    62: 0000000000000672    40 FUNC    GLOBAL DEFAULT   14 fun_1
$ readelf -a main_sections | grep fun_
    46: 000000000000064a    40 FUNC    GLOBAL DEFAULT   14 fun_0
    55: 0000000000000672    40 FUNC    GLOBAL DEFAULT   14 fun_3
```

　　可以看见，在最终的目标文件中，未使用的函数并未被链接进最终的目标文件。