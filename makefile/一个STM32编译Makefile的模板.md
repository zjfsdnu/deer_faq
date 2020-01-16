``` makefile
# STM32F103 Makefile模板
# 参考来源：
# https://github.com/andytt/stm32_template
# https://github.com/latelee/Makefile_templet
# 使用：
# make         #默认编译：调试版本，不输出详细编译过程
# make debug=n # release版本，不输出详细编译过程 
# make V=1     # 调试版本，输出详细编译过程
# make debug=n V=1 # release版本，输出详细编译过程 
#
# 为加快编译，make可加“-j”选项。
# 本Makefile不需要手动添加头文件/源码目录，会自动查找。
# 目的是为了工程的方便而牺牲一点编译时间。
# 可能需要修改的地方使用“!!!===”标示出来
# TODO：为尽量减少二进制文件体积，需要将不调用的函数剔除，
# 但又要保存源码文件（防止后面需要用到）
# 
# log:
# 2018.12.10: 首版完成
################################################

#!!!=== 交叉编译器
CROSS_COMPILE = arm-none-eabi-
CC  = $(CROSS_COMPILE)gcc
AS  = $(CROSS_COMPILE)gcc -x assembler-with-cpp
CP  = $(CROSS_COMPILE)objcopy
AR  = $(CROSS_COMPILE)ar
SIZE  = $(CROSS_COMPILE)size
HEX = $(CP) -O ihex
BIN = $(CP) -O binary -S

MKDIR_P ?= mkdir -p

#!!!=== 目标文件名，注：下面会生成$(target).hex等文件
target = target

#!!!=== 是否调试版本，如是，设置为y，反之，为n
debug  = n

#!!!=== 编译目录
BUILD_DIR = build

#!!!=== 头文件目录，在当前目录下搜索所有目录，将其当成头文件目录
INC := $(shell find ./ -type d)
INCDIRS := $(addprefix -I, $(INC))

######################################
# C源码文件
#注：find会递归查找项目目录所有.c文件，如c文件不必要，则要删除，否则可能会编译出错
C_SOURCES =  $(shell find ./ -name '*.c')

#!!!=== 启动汇编文件
ASM_SOURCES = $(shell find ./ -name '*.s') #app/startup_stm32f103xe.s
# TODO：其它目录在此列出

# float-abi 如不支持，则不填写
FLOAT-ABI = 
FPU = 

# 目标芯片特有编译指令
MCU = -mcpu=cortex-m3 -mthumb $(FPU) $(FLOAT-ABI)

# c编译标志
CFLAGS = $(MCU) $(DEFS) $(INCDIRS) -std=c99 -Wall -Wfatal-errors \
         -MMD -fdata-sections -ffunction-sections
ASFLAGS = $(CFLAGS) $(AS_DEFS)

# debug或release版本选择
ifeq ($(debug), y)
    CFLAGS += -g -gdwarf-2
else
    CFLAGS += -O2 -s # 或者-Og
endif

# AS宏定义
AS_DEFS = 

#!!!=== C宏定义
# STM32F103必须定义USE_STDPERIPH_DRIVER和STM32F10X_HD
# USE_FREERTOS 使用freertos系统
# USE_UCOSII   使用ucos-ii系统
# OS_SUPPORT   使用了OS（在GUI源码中使用到此宏）
DEFS_STR += STM32F10X_HD USE_STDPERIPH_DRIVER OS_SUPPORT USE_FREERTOS
DEFS     := $(addprefix -D, $(DEFS_STR))

#!!!=== 链接脚本文件
LDSCRIPT = $(shell find ./ -name '*.ld') #app/STM32F103XE_FLASH.ld

#!!!=== 静态库名称
LIBS = -lc -lm -lnosys 
LIBS += $(shell find ./ -name '*.a') # STemWin526_CM3_OS_GCC.a
# 其它库目录
LIBDIR = 
# 链接标志
#  添加-specs=rdimon.specs会造成close/seek与nosys库冲突
# nano库实现相应C库的功能，但体积会更小
LDFLAGS = $(MCU) -T$(LDSCRIPT) $(LIBDIR) $(LIBS) \
          -Wl,-Map=$(BUILD_DIR)/$(target).map,--cref -Wl,--gc-sections \
          -specs=nosys.specs -specs=nano.specs 

ifeq ($(V),1)
Q=
NQ=true
else
Q=@
NQ=echo
endif

# default action: build all
all: $(BUILD_DIR)/$(target).out $(BUILD_DIR)/$(target).hex $(BUILD_DIR)/$(target).bin

#######################################
## 目标文件规则（由.c .s产生.o的规则）
OBJECTS = $(addprefix $(BUILD_DIR)/,$(notdir $(C_SOURCES:.c=.o)))
vpath %.c $(sort $(dir $(C_SOURCES)))
OBJECTS += $(addprefix $(BUILD_DIR)/,$(notdir $(ASM_SOURCES:.s=.o)))
vpath %.s $(sort $(dir $(ASM_SOURCES)))

DEPS := $(OBJECTS:.o=.d)

# 编译.c .s文件
$(BUILD_DIR)/%.o: %.c Makefile | $(BUILD_DIR)
	@$(NQ) "Compiling: " $(basename $(notdir $@)).c
	$(Q)$(CC) -c $(CFLAGS) -Wa,-a,-ad,-alms=$(BUILD_DIR)/$(notdir $(<:.c=.lst)) $< -o $@

$(BUILD_DIR)/%.o: %.s Makefile | $(BUILD_DIR)
	@$(NQ) "Assembling: " $(basename $(notdir $@)).s
	$(Q)$(AS) -c $(CFLAGS) $< -o $@

# 生成out hex bin文件
$(BUILD_DIR)/$(target).out: $(OBJECTS) Makefile
	@$(NQ) "linking..."
	@$(NQ) "Creating file..." $(notdir $@)
	$(Q)$(CC) $(OBJECTS) $(LDFLAGS) -o $@
	$(Q)$(SIZE) $@

$(BUILD_DIR)/%.hex: $(BUILD_DIR)/%.out | $(BUILD_DIR)
	@$(NQ) "Creating hex file..." $(notdir $@)
	$(Q)$(HEX) $< $@
	$(Q)mv $@ .
	
$(BUILD_DIR)/%.bin: $(BUILD_DIR)/%.out | $(BUILD_DIR)
	@$(NQ) "Generating bin file..." $(notdir $@)
	$(Q)$(BIN) $< $@	
	#$(Q)mv $@ .

$(BUILD_DIR):
	$(Q)$(MKDIR_P) $@		

## 清理文件
clean:
	@$(NQ) "Cleaning..."
	$(Q)@-rm -fR .dep $(BUILD_DIR)
	$(Q)@find . -iname '*.o' -o -iname '*.bak' -o -iname '*.d' | xargs rm -f

## 烧录命令
flash:
	st-flash write $(BUILD_DIR)/$(target).bin 0x8000000
## 擦除命令
erase:
	st-flash erase

.PHONY: all clean flash erase

## 依赖文件
-include $(DEPS)
```
————————————————

版权声明：本文为CSDN博主「李迟」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。

原文链接：https://blog.csdn.net/subfate/article/details/86268016
