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



### 自动生成依赖性

在 Makefile 中，我们的依赖关系可能会需要包含一系列的头文件，比如，如果我们的 main.c 中有一 句 *#include "defs.h"* ，那么我们的依赖关系应该是： 

> main.o : main.c defs.h 

但是，如果是一个比较大型的工程，你必需清楚哪些 C 文件包含了哪些头文件，并且，你在加入或 删除头文件时，也需要小心地修改 Makefile，这是一个很没有维护性的工作。为了避免这种繁重而又容 易出错的事情，我们可以使用 C/C++ 编译的一个功能。大多数的 C/C++ 编译器都支持一个“-M”的 选项，即自动找寻源文件中包含的头文件，并生成一个依赖关系。例如，如果我们执行下面的命令: 

> cc -M main.c 

其输出是：

```makefile
main.o : main.c defs.h
```

于是由编译器自动生成的依赖关系，这样一来，你就不必再手动书写若干文件的依赖关系，而由编 译器自动生成了。需要提醒一句的是，如果你使用 GNU 的 C/C++ 编译器，你得用 -MM 参数，不然，-M 参数会把一些标准库的头文件也包含进来。 gcc -M main.c 的输出是:

```makefile
main.o: main.c defs.h /usr/include/stdio.h /usr/include/features.h \
/usr/include/sys/cdefs.h /usr/include/gnu/stubs.h \
/usr/lib/gcc-lib/i486-suse-linux/2.95.3/include/stddef.h \
/usr/include/bits/types.h /usr/include/bits/pthreadtypes.h \
/usr/include/bits/sched.h /usr/include/libio.h \
/usr/include/_G_config.h /usr/include/wchar.h \
/usr/include/bits/wchar.h /usr/include/gconv.h \
/usr/lib/gcc-lib/i486-suse-linux/2.95.3/include/stdarg.h \
/usr/include/bits/stdio_lim.h
```

gcc -MM main.c 的输出则是:

``` makefile
main.o: main.c defs.h
```

那么，编译器的这个功能如何与我们的 Makefile 联系在一起呢。因为这样一来，我们的 Makefile 也 要根据这些源文件重新生成，让 Makefile 自已依赖于源文件？这个功能并不现实，不过我们可以有其它 手段来迂回地实现这一功能。GNU 组织建议把编译器为每一个源文件的自动生成的依赖关系放到一个 文件中，为每一个 name.c 的文件都生成一个 name.d 的 Makefile 文件，.d 文件中就存放对应 .c 文件 的依赖关系。 

于是，我们可以写出 .c 文件和 .d 文件的依赖关系，并让 make 自动更新或生成 .d 文件，并把其 包含在我们的主 Makefile 中，这样，我们就可以自动化地生成每个文件的依赖关系了。 

这里，我们给出了一个模式规则来产生 .d 文件：

```makefile
%.d: %.c
@set -e; rm -f $@; \
$(CC) -M $(CPPFLAGS) $< > $@.$$$$; \
sed 's,\($*\)\.o[ :]*,\1.o $@ : ,g' < $@.$$$$ > $@; \
rm -f $@.$$$$

```

这个规则的意思是，所有的 .d 文件依赖于 .c 文件，rm -f $@ 的意思是删除所有的目标，也就是 .d 文件，第二行的意思是，为每个依赖文件 $< ，也就是 .c 文件生成依赖文件，$@ 表示模式 %.d 文件， 如果有一个 C 文件是 name.c，那么 % 就是 name ，$$$$ 意为一个随机编号，第二行生成的文件有可能 是“name.d.12345”，第三行使用 sed 命令做了一个替换，关于 sed 命令的用法请参看相关的使用文档。 第四行就是删除临时文件。

总而言之，这个模式要做的事就是在编译器生成的依赖关系中加入 .d 文件的依赖，即把依赖关系：

```makefile
main.o : main.c defs.h
```

转成：

```makefile
main.o main.d : main.c defs.h
```

于是，我们的 .d 文件也会自动更新了，并会自动生成了，当然，你还可以在这个 .d 文件中加入的 不只是依赖关系，包括生成的命令也可一并加入，让每个 .d 文件都包含一个完赖的规则。一旦我们完成 这个工作，接下来，我们就要把这些自动生成的规则放进我们的主 Makefile 中。我们可以使用 Makefile 的“include”命令，来引入别的 Makefile 文件，例如：

```makefile
sources = foo.c bar.c
include $(sources:.c=.d)
```

上述语句中的 $(sources:.c=.d) 中的 .c=.d 的意思是做一个替换，把变量 $(sources) 所有 .c 的字串都替换成 .d 。当然，你得注意次序， 因为 include 是按次序来载入文件，最先载入的 .d 文件中的目标会成为默认目标。



在项目根目录下执行：

```shell
make -n
```

（-n 表示只显示 不执行）输出：

```shell
mkdir build

arm-none-eabi-gcc 						-c -mcpu=cortex-m3 -mthumb   -DUSE_HAL_DRIVER -DSTM32F103xE -IInc -IDrivers/STM32F1xx_HAL_Driver/Inc -IDrivers/STM32F1xx_HAL_Driver/Inc/Legacy -IDrivers/CMSIS/Device/ST/STM32F1xx/Include -IDrivers/CMSIS/Include -Og -Wall -fdata-sections -ffunction-sections -g -gdwarf-2 -MMD -MP -MF"build/main.d" -Wa,-a,-ad,-alms=build/main.lst Src/main.c -o build/main.o
arm-none-eabi-gcc 						-c -mcpu=cortex-m3 -mthumb   -DUSE_HAL_DRIVER -DSTM32F103xE -IInc -IDrivers/STM32F1xx_HAL_Driver/Inc -IDrivers/STM32F1xx_HAL_Driver/Inc/Legacy -IDrivers/CMSIS/Device/ST/STM32F1xx/Include -IDrivers/CMSIS/Include -Og -Wall -fdata-sections -ffunction-sections -g -gdwarf-2 -MMD -MP -MF"build/stm32f1xx_it.d" -Wa,-a,-ad,-alms=build/stm32f1xx_it.lst Src/stm32f1xx_it.c -o build/stm32f1xx_it.o
arm-none-eabi-gcc 						-c -mcpu=cortex-m3 -mthumb   -DUSE_HAL_DRIVER -DSTM32F103xE -IInc -IDrivers/STM32F1xx_HAL_Driver/Inc -IDrivers/STM32F1xx_HAL_Driver/Inc/Legacy -IDrivers/CMSIS/Device/ST/STM32F1xx/Include -IDrivers/CMSIS/Include -Og -Wall -fdata-sections -ffunction-sections -g -gdwarf-2 -MMD -MP -MF"build/stm32f1xx_hal_msp.d" -Wa,-a,-ad,-alms=build/stm32f1xx_hal_msp.lst Src/stm32f1xx_hal_msp.c -o build/stm32f1xx_hal_msp.o
arm-none-eabi-gcc 						-c -mcpu=cortex-m3 -mthumb   -DUSE_HAL_DRIVER -DSTM32F103xE -IInc -IDrivers/STM32F1xx_HAL_Driver/Inc -IDrivers/STM32F1xx_HAL_Driver/Inc/Legacy -IDrivers/CMSIS/Device/ST/STM32F1xx/Include -IDrivers/CMSIS/Include -Og -Wall -fdata-sections -ffunction-sections -g -gdwarf-2 -MMD -MP -MF"build/stm32f1xx_hal_gpio_ex.d" -Wa,-a,-ad,-alms=build/stm32f1xx_hal_gpio_ex.lst Drivers/STM32F1xx_HAL_Driver/Src/stm32f1xx_hal_gpio_ex.c -o build/stm32f1xx_hal_gpio_ex.o
arm-none-eabi-gcc 						-c -mcpu=cortex-m3 -mthumb   -DUSE_HAL_DRIVER -DSTM32F103xE -IInc -IDrivers/STM32F1xx_HAL_Driver/Inc -IDrivers/STM32F1xx_HAL_Driver/Inc/Legacy -IDrivers/CMSIS/Device/ST/STM32F1xx/Include -IDrivers/CMSIS/Include -Og -Wall -fdata-sections -ffunction-sections -g -gdwarf-2 -MMD -MP -MF"build/stm32f1xx_hal_tim.d" -Wa,-a,-ad,-alms=build/stm32f1xx_hal_tim.lst Drivers/STM32F1xx_HAL_Driver/Src/stm32f1xx_hal_tim.c -o build/stm32f1xx_hal_tim.o
arm-none-eabi-gcc 						-c -mcpu=cortex-m3 -mthumb   -DUSE_HAL_DRIVER -DSTM32F103xE -IInc -IDrivers/STM32F1xx_HAL_Driver/Inc -IDrivers/STM32F1xx_HAL_Driver/Inc/Legacy -IDrivers/CMSIS/Device/ST/STM32F1xx/Include -IDrivers/CMSIS/Include -Og -Wall -fdata-sections -ffunction-sections -g -gdwarf-2 -MMD -MP -MF"build/stm32f1xx_hal_tim_ex.d" -Wa,-a,-ad,-alms=build/stm32f1xx_hal_tim_ex.lst Drivers/STM32F1xx_HAL_Driver/Src/stm32f1xx_hal_tim_ex.c -o build/stm32f1xx_hal_tim_ex.o
arm-none-eabi-gcc 						-c -mcpu=cortex-m3 -mthumb   -DUSE_HAL_DRIVER -DSTM32F103xE -IInc -IDrivers/STM32F1xx_HAL_Driver/Inc -IDrivers/STM32F1xx_HAL_Driver/Inc/Legacy -IDrivers/CMSIS/Device/ST/STM32F1xx/Include -IDrivers/CMSIS/Include -Og -Wall -fdata-sections -ffunction-sections -g -gdwarf-2 -MMD -MP -MF"build/stm32f1xx_hal.d" -Wa,-a,-ad,-alms=build/stm32f1xx_hal.lst Drivers/STM32F1xx_HAL_Driver/Src/stm32f1xx_hal.c -o build/stm32f1xx_hal.o
arm-none-eabi-gcc 						-c -mcpu=cortex-m3 -mthumb   -DUSE_HAL_DRIVER -DSTM32F103xE -IInc -IDrivers/STM32F1xx_HAL_Driver/Inc -IDrivers/STM32F1xx_HAL_Driver/Inc/Legacy -IDrivers/CMSIS/Device/ST/STM32F1xx/Include -IDrivers/CMSIS/Include -Og -Wall -fdata-sections -ffunction-sections -g -gdwarf-2 -MMD -MP -MF"build/stm32f1xx_hal_rcc.d" -Wa,-a,-ad,-alms=build/stm32f1xx_hal_rcc.lst Drivers/STM32F1xx_HAL_Driver/Src/stm32f1xx_hal_rcc.c -o build/stm32f1xx_hal_rcc.o
arm-none-eabi-gcc 						-c -mcpu=cortex-m3 -mthumb   -DUSE_HAL_DRIVER -DSTM32F103xE -IInc -IDrivers/STM32F1xx_HAL_Driver/Inc -IDrivers/STM32F1xx_HAL_Driver/Inc/Legacy -IDrivers/CMSIS/Device/ST/STM32F1xx/Include -IDrivers/CMSIS/Include -Og -Wall -fdata-sections -ffunction-sections -g -gdwarf-2 -MMD -MP -MF"build/stm32f1xx_hal_rcc_ex.d" -Wa,-a,-ad,-alms=build/stm32f1xx_hal_rcc_ex.lst Drivers/STM32F1xx_HAL_Driver/Src/stm32f1xx_hal_rcc_ex.c -o build/stm32f1xx_hal_rcc_ex.o
arm-none-eabi-gcc 						-c -mcpu=cortex-m3 -mthumb   -DUSE_HAL_DRIVER -DSTM32F103xE -IInc -IDrivers/STM32F1xx_HAL_Driver/Inc -IDrivers/STM32F1xx_HAL_Driver/Inc/Legacy -IDrivers/CMSIS/Device/ST/STM32F1xx/Include -IDrivers/CMSIS/Include -Og -Wall -fdata-sections -ffunction-sections -g -gdwarf-2 -MMD -MP -MF"build/stm32f1xx_hal_gpio.d" -Wa,-a,-ad,-alms=build/stm32f1xx_hal_gpio.lst Drivers/STM32F1xx_HAL_Driver/Src/stm32f1xx_hal_gpio.c -o build/stm32f1xx_hal_gpio.o
arm-none-eabi-gcc 						-c -mcpu=cortex-m3 -mthumb   -DUSE_HAL_DRIVER -DSTM32F103xE -IInc -IDrivers/STM32F1xx_HAL_Driver/Inc -IDrivers/STM32F1xx_HAL_Driver/Inc/Legacy -IDrivers/CMSIS/Device/ST/STM32F1xx/Include -IDrivers/CMSIS/Include -Og -Wall -fdata-sections -ffunction-sections -g -gdwarf-2 -MMD -MP -MF"build/stm32f1xx_hal_dma.d" -Wa,-a,-ad,-alms=build/stm32f1xx_hal_dma.lst Drivers/STM32F1xx_HAL_Driver/Src/stm32f1xx_hal_dma.c -o build/stm32f1xx_hal_dma.o
arm-none-eabi-gcc 						-c -mcpu=cortex-m3 -mthumb   -DUSE_HAL_DRIVER -DSTM32F103xE -IInc -IDrivers/STM32F1xx_HAL_Driver/Inc -IDrivers/STM32F1xx_HAL_Driver/Inc/Legacy -IDrivers/CMSIS/Device/ST/STM32F1xx/Include -IDrivers/CMSIS/Include -Og -Wall -fdata-sections -ffunction-sections -g -gdwarf-2 -MMD -MP -MF"build/stm32f1xx_hal_cortex.d" -Wa,-a,-ad,-alms=build/stm32f1xx_hal_cortex.lst Drivers/STM32F1xx_HAL_Driver/Src/stm32f1xx_hal_cortex.c -o build/stm32f1xx_hal_cortex.o
arm-none-eabi-gcc 						-c -mcpu=cortex-m3 -mthumb   -DUSE_HAL_DRIVER -DSTM32F103xE -IInc -IDrivers/STM32F1xx_HAL_Driver/Inc -IDrivers/STM32F1xx_HAL_Driver/Inc/Legacy -IDrivers/CMSIS/Device/ST/STM32F1xx/Include -IDrivers/CMSIS/Include -Og -Wall -fdata-sections -ffunction-sections -g -gdwarf-2 -MMD -MP -MF"build/stm32f1xx_hal_pwr.d" -Wa,-a,-ad,-alms=build/stm32f1xx_hal_pwr.lst Drivers/STM32F1xx_HAL_Driver/Src/stm32f1xx_hal_pwr.c -o build/stm32f1xx_hal_pwr.o
arm-none-eabi-gcc 						-c -mcpu=cortex-m3 -mthumb   -DUSE_HAL_DRIVER -DSTM32F103xE -IInc -IDrivers/STM32F1xx_HAL_Driver/Inc -IDrivers/STM32F1xx_HAL_Driver/Inc/Legacy -IDrivers/CMSIS/Device/ST/STM32F1xx/Include -IDrivers/CMSIS/Include -Og -Wall -fdata-sections -ffunction-sections -g -gdwarf-2 -MMD -MP -MF"build/stm32f1xx_hal_flash.d" -Wa,-a,-ad,-alms=build/stm32f1xx_hal_flash.lst Drivers/STM32F1xx_HAL_Driver/Src/stm32f1xx_hal_flash.c -o build/stm32f1xx_hal_flash.o
arm-none-eabi-gcc 						-c -mcpu=cortex-m3 -mthumb   -DUSE_HAL_DRIVER -DSTM32F103xE -IInc -IDrivers/STM32F1xx_HAL_Driver/Inc -IDrivers/STM32F1xx_HAL_Driver/Inc/Legacy -IDrivers/CMSIS/Device/ST/STM32F1xx/Include -IDrivers/CMSIS/Include -Og -Wall -fdata-sections -ffunction-sections -g -gdwarf-2 -MMD -MP -MF"build/stm32f1xx_hal_flash_ex.d" -Wa,-a,-ad,-alms=build/stm32f1xx_hal_flash_ex.lst Drivers/STM32F1xx_HAL_Driver/Src/stm32f1xx_hal_flash_ex.c -o build/stm32f1xx_hal_flash_ex.o
arm-none-eabi-gcc 						-c -mcpu=cortex-m3 -mthumb   -DUSE_HAL_DRIVER -DSTM32F103xE -IInc -IDrivers/STM32F1xx_HAL_Driver/Inc -IDrivers/STM32F1xx_HAL_Driver/Inc/Legacy -IDrivers/CMSIS/Device/ST/STM32F1xx/Include -IDrivers/CMSIS/Include -Og -Wall -fdata-sections -ffunction-sections -g -gdwarf-2 -MMD -MP -MF"build/stm32f1xx_hal_exti.d" -Wa,-a,-ad,-alms=build/stm32f1xx_hal_exti.lst Drivers/STM32F1xx_HAL_Driver/Src/stm32f1xx_hal_exti.c -o build/stm32f1xx_hal_exti.o
arm-none-eabi-gcc 						-c -mcpu=cortex-m3 -mthumb   -DUSE_HAL_DRIVER -DSTM32F103xE -IInc -IDrivers/STM32F1xx_HAL_Driver/Inc -IDrivers/STM32F1xx_HAL_Driver/Inc/Legacy -IDrivers/CMSIS/Device/ST/STM32F1xx/Include -IDrivers/CMSIS/Include -Og -Wall -fdata-sections -ffunction-sections -g -gdwarf-2 -MMD -MP -MF"build/system_stm32f1xx.d" -Wa,-a,-ad,-alms=build/system_stm32f1xx.lst Src/system_stm32f1xx.c -o build/system_stm32f1xx.o
arm-none-eabi-gcc -x assembler-with-cpp -c -mcpu=cortex-m3 -mthumb   -DUSE_HAL_DRIVER -DSTM32F103xE -IInc -IDrivers/STM32F1xx_HAL_Driver/Inc -IDrivers/STM32F1xx_HAL_Driver/Inc/Legacy -IDrivers/CMSIS/Device/ST/STM32F1xx/Include -IDrivers/CMSIS/Include -Og -Wall -fdata-sections -ffunction-sections -g -gdwarf-2 -MMD -MP -MF"build/startup_stm32f103xe.d" startup_stm32f103xe.s -o build/startup_stm32f103xe.o

arm-none-eabi-gcc build/main.o build/stm32f1xx_it.o build/stm32f1xx_hal_msp.o build/stm32f1xx_hal_gpio_ex.o build/stm32f1xx_hal_tim.o build/stm32f1xx_hal_tim_ex.o build/stm32f1xx_hal.o build/stm32f1xx_hal_rcc.o build/stm32f1xx_hal_rcc_ex.o build/stm32f1xx_hal_gpio.o build/stm32f1xx_hal_dma.o build/stm32f1xx_hal_cortex.o build/stm32f1xx_hal_pwr.o build/stm32f1xx_hal_flash.o build/stm32f1xx_hal_flash_ex.o build/stm32f1xx_hal_exti.o build/system_stm32f1xx.o build/startup_stm32f103xe.o -mcpu=cortex-m3 -mthumb   -specs=nano.specs -TSTM32F103ZETx_FLASH.ld  -lc -lm -lnosys  -Wl,-Map=build/f103test.map,--cref -Wl,--gc-sections -o build/f103test.elf

arm-none-eabi-size build/f103test.elf
   text    data     bss     dec     hex filename
   2908      20    1572    4500    1194 build/f103test.elf

arm-none-eabi-objcopy -O ihex build/f103test.elf build/f103test.hex
arm-none-eabi-objcopy -O binary -S build/f103test.elf build/f103test.bin
```

大概流程是：

1. 通过*arm-none-eabi-gcc -c* 将17个.c文件编译为.o中间文件
2. 通过*arm-none-eabi-gcc -x assembler-with-cpp -c* 将 startup_stm32f103xe.s文件编译为.o中间文件
3. 通过*arm-none-eabi-gcc* 将这18个.o文件编译为.elf文件
4. 通过*arm-none-eabi-size* 查看elf文件的结构和大小
5. 通过*arm-none-eabi-objcopy -O ihex* 将elf文件转为.hex 16进制字符串文件
6. 通过*arm-none-eabi-objcopy -O binary* 将elf文件转为.bin 二进制文件

拿出第一条来分析一下
```shell
arm-none-eabi-gcc -c -mcpu=cortex-m3 -mthumb   -DUSE_HAL_DRIVER -DSTM32F103xE -IInc -IDrivers/STM32F1xx_HAL_Driver/Inc -IDrivers/STM32F1xx_HAL_Driver/Inc/Legacy -IDrivers/CMSIS/Device/ST/STM32F1xx/Include -IDrivers/CMSIS/Include -ID
rivers/CMSIS/Include -Og -Wall -fdata-sections -ffunction-sections -g -gdwarf-2 -MMD -MP -MF"build/main.d" -Wa,-a,-ad,-alms=build/main.lst Src/main.c -o build/main.o
```
`-c`表示编译为.o文件 不连接

`-mcpu=cortex-m3 -mthumb`

`-DUSE_HAL_DRIVER -DSTM32F103xE` 这里定义了两个宏变量

在 `Drivers\CMSIS\Device\ST\STM32F1xx\Include\stm32f1xx.h` 头文件里有判断
```c
#if defined (USE_HAL_DRIVER)
 #include "stm32f1xx_hal.h"
#endif /* USE_HAL_DRIVER */

#if defined(STM32F100xB)
  #include "stm32f100xb.h"
#elif defined(STM32F100xE)
  #include "stm32f100xe.h"
#elif defined(STM32F101x6)
  #include "stm32f101x6.h"
#elif defined(STM32F101xB)
  #include "stm32f101xb.h"
#elif defined(STM32F101xE)
  #include "stm32f101xe.h"
#elif defined(STM32F101xG)
  #include "stm32f101xg.h"
#elif defined(STM32F102x6)
  #include "stm32f102x6.h"
#elif defined(STM32F102xB)
  #include "stm32f102xb.h"
#elif defined(STM32F103x6)
  #include "stm32f103x6.h"
#elif defined(STM32F103xB)
  #include "stm32f103xb.h"
#elif defined(STM32F103xE)
  #include "stm32f103xe.h"
#elif defined(STM32F103xG)
  #include "stm32f103xg.h"
#elif defined(STM32F105xC)
  #include "stm32f105xc.h"
#elif defined(STM32F107xC)
  #include "stm32f107xc.h"
#else
 #error "Please select first the target STM32F1xx device used in your application (in stm32f1xx.h file)"
#endif
```

`-IInc -IDrivers/STM32F1xx_HAL_Driver/Inc -IDrivers/STM32F1xx_HAL_Driver/Inc/Legacy -IDrivers/CMSIS/Device/ST/STM32F1xx/Include -IDrivers/CMSIS/Include`

这里加了5个包含头文件的目录

`-Og` 优化调试体验。 -Og 启用不会影响调试的优化

`-Wall` 编译后显示所有警告

`-fdata-sections -ffunction-sections` 为每个function和data item分配独立的section
参考 [**gcc -ffunction-sections -fdata-sections -Wl,–gc-sections 参数详解**](https://github.com/zjfsdnu/deer_faq/blob/master/gcc -ffunction-sections -fdata-sections -Wl%2C–gc-sections 参数详解.md)

`-g -gdwarf-2` 

`-MMD -MP -MF"build/main.d"` 

`-Wa,-a,-ad,-alms=build/main.lst`

