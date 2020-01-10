Notepad++ 是一款我很喜欢的文本编辑器，除了写代码和写文章外的一般文本编辑工作我都是使用它完成的，配置新电脑时自然不能少了它。我下载的是 Portable 便携版，不过也正因如此，原本在 Installer 中通过选项可以添加的 Edit with Notepad++ 右键菜单项也没有了（此操作需要写注册表）。

因为这个右键菜单还蛮好用的，所以我打算把它找回来。

### 方法一：使用官方提供的 NppShell.dll
此方法来自 Notepad++ 的官方 Wiki（见底部参考链接）。

添加右键菜单项需要修改注册表，除了手动修改注册表，Notepad++ 官方还提供了一个 DLL 文件用于注册以及卸载右键菜单。如果你使用的是安装版，那么在程序目录下应该会有一个 NppShell_06.dll 文件（不同版本下文件名中的数字可能会不同）；如果没有或者是便携版，那么请在这里下载该文件：

https://github.com/notepad-plus-plus/notepad-plus-plus/tree/master/PowerEditor/bin

上面的地址是 Notepad++ 官方 GitHub 仓库中提供的预编译 DLL，32 位系统的用户请下载 NppShell.dll，64 位系统的请下载 NppShell64.dll。不要吐槽为啥这文件四年没更新了，因为人家确实是四年没更新了，Wiki 原文中提供的链接还是八年前的呢（笑）。

下载后，打开一个具有管理员权限的 cmd.exe 或者 PowerShell，cd 到 Notepad++ 的安装目录（直接指定 DLL 的绝对路径是没用的，必须在程序目录下运行），并运行如下命令（文件名自行替换）：
```
regsvr32 /i NppShell_06.dll
```
(我这里没有NppShell64.dll 改成NppShell_06.dll也一样)

运行后会弹出一个对话框，直接点 OK 就可以了。

如果没给管理员权限，会报错「模块 NppShell64.dll 已加载，但对 DllRegisterServer 的调用失败」。

如果要删除右键菜单，请运行：

regsvr32 /u NppShell64.dll
如果你不会运行这些命令也没事，将以下内容使用记事本保存为 .bat 文件，放到 Notepad++ 的安装目录下，右键「以管理员身份运行」即可（此脚本修改自：Notepad++ 添加右键打开菜单 - 成功志）。

``` bat
@Echo Off
cd /d %~dp0
title Notepad++ 右键菜单添加/删除工具

SetLocal EnableDelayedExpansion
echo 1. 添加 Notepad++ 右键菜单
echo ------------------------
echo 2. 删除 Notepad++ 右键菜单
echo ------------------------

Set /p u=请输入数字并按 Enter 确定：

If "%u%"=="1" Goto regnpp
If "%u%"=="2" Goto unregnpp

:regnpp
regsvr32 /i NppShell64.dll
exit

:unregnpp
regsvr32 /u NppShell64.dll
exit
```
