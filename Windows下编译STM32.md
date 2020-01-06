### MinGW安装

下载地址：http://www.mingw.org/

下载 mingw-get-setup.exe 并安装 MinGW Installation Manager 。

在 MinGW Installation Manager 中选择 mingw32-base 和 mingw32-gcc-g++ 安装，右键选择“Mark for Installation”,之后选择"Installation -> Apply Changes”，等待下载完成。

如果部分包下载失败，可以再次尝试，已经下载的包不会重新下载。

安装后，将 MinGW\bin ， MinGW\include， MinGW\lib 加入到环境变量 PATH 。

运行 gcc -v 确认MinGW已经安装成功。

### GNU Arm Embedded Toolchain

下载地址：https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm

下载最新的Windows安装包，我下载的是 gcc-arm-none-eabi-9-2019-q4-major-win32-sha2.exe 。

按照默认选项安装，安装完成时，选择自动加入到环境变量 PATH 。

运行 arm-none-eabi-gcc -v 确认安装成功。

### 安装STM32CubeMX

下载地址：https://www.st.com/zh/development-tools/stm32cubemx.html，点击获取软件

全部按照默认选项安装。

### 生成工程

通过选择芯片新建工程。

双击选中的芯片

在 Project Manager 中输入工程名和路径，并选择开发要用的IDE，我这里用 gcc 编译，所以选择Makefile，然后点击 GENERATE CODE 。

生成好的工程中已经复制进去了库文件，可以把整个工程复制到其他机器编译。

### 编译

在工程根目录下打开命令行，运行 mingw32-make 编译。将会创建 build 文件夹，并在其中生成编译结果的 .hex 和 .bin 文件。
