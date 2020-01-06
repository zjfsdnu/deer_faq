### 下载安装MinGW
MinGW下载网页：http://sourceforge.net/projects/mingw/files/latest/download?source=files
下载后，运行程序：mingw-get-setup.exe，选择download latest repository catalogues. 选择编译器是勾选C Compiler 与C++ Compiler，点击next进行下载及安装。

### 设置环境变量
右击计算机->属性->高级系统设置->环境变量，在系统变量中找到PATH，将MinGW安装目录里的bin文件夹的地址添加到PATH里面。
打开MinGW的安装目录，打开bin文件夹，将mingw32-make.exe复制一份，重命名为make.exe。

### 测试GCC编译
创建一下test.c,用记事本打开该文件，将以下内容复制到文件中。
``` c
#include<stdio.h>
#include<stdlib.h>

int main(void){
    printf("Hello, world!\n");
    system("pause");
    return 0;
}
```

打开命令提示符，更改目录到test.c的位置，键入

gcc -o test.exe test.c

可生成test.exe可执行文件。

### 测试makefile
新建文件夹，在文件夹内创建max_num.c、max.h、max.c、makefile四个文件。

max_num.c内容如下：

``` c
#include <stdio.h>
#include <stdlib.h>
#include "max.h"
 
int main(void)
{
    printf("The bigger one of 3 and 5 is %d\n", max(3, 5));
    system("pause");
    return 0;
}
```

max.h内容如下：

``` c
int max(int a, int b);
```

max.c内容如下：

``` c
#include "max.h"
 
int max(int a, int b)
{
    return a > b ? a : b;
}
```

makefile内容如下：

``` makefile
max_num.exe: max_num.o max.o
	gcc -o max_num.exe max_num.o max.o
 
max_num.o: max_num.c max.h
	gcc -c max_num.c
 
max.o: max.c max.h
	gcc -c max.c
```


注意所有含有gcc的行前面是一个制表符，而非若干空格。否则可能会保存，无法编译。

打开命令提示符，更改目录到新建的文件夹，键入make，可生成指定的应运程序。

测试完成。

————————————————

版权声明：本文为CSDN博主「pdcxs007」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/pdcxs007/article/details/8582559
