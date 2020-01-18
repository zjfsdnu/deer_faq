# 1 关于Chocolatey

[Chocolatey](https://chocolatey.org/)是Windows平台上的包管理器，通过它可以集中安装、管理、更新各种各样的软件。

是和apt-get，brew差不都的一个东西。

特别适合管理一些小众、轻量的开源软件。

可以**一条命令更新全部软件**，特别适合治疗自己的更新强迫症（尤其是遇上一些不能自动检查更新的软件时）。

除了直接自动化从程序官网拽安装包，自动化安装外。官方的源里面，还有一些绿化的软件、净化软件可以开袋即食。

总体而言，如果不想特殊设置的话，Chocolatey整体的操作与使用还是比较亲民的。我这样的小白就可以按照官网说明直接用~~，而且路人看着会觉得特别厉害~~。

编不出其它话了，让我们直接开始安装吧！

# 2 Chocolatey的安装

考虑到这篇文章的信息会过时，安装的详细信息请依照[官方指示](https://chocolatey.org/install)。

推荐大家使用依靠管理员权限安装的方法，很方便快捷。

右键你任务栏开始菜单的Windows徽标，看看你是哪种命令行：

### 2.1 cmd (管理员)

直接贴上：

> @”%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe” -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command “iex ((New-Object System.Net.WebClient).DownloadString(‘https://chocolatey.org/install.ps1’))” && SET “PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin”

然后回车，等待完成就行了。

### 2.2 Windows PowerShell (管理员)

直接贴上：

> Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString(‘https://chocolatey.org/install.ps1’))

然后回车，等待完成就行了。

这个操作会改变你PowerShell安装软件的安全性程度。不过我开着杀软BDF我不怕（逃）

# 3 软件推荐

如果你想安装一个软件，直接在命令行（管理员）上敲上：

> choco install 软件的包名称

回车，就成了。

不过，软件的包名称和软件名称可能不一样，推荐直接去[官方的软件列表](https://chocolatey.org/packages)搜到名字之后再用。

如果你想试试Chocolatey的图形界面，可以安装：

> choco install chocolateygui

（但是其实挺鸡肋的）

类似的，想卸载某个东西的话，直接

> choco uninstall 软件的包名称

就行了。

### 3.1 浏览器相关

你可以通过Chocolatey安装无捆绑的**Flash插件**。但是要注意对应的插件类型；不同浏览器，如Firefox和Chrome，用的插件类型就不同。

> choco install flashplayerplugin

或是

> choco install flashplayerppapi

你的浏览器也可以通过Chocolatey管理。当然我常用的是Firefox，会自动更新。Chocolatey更适合更新**浏览器备胎**，于是我们

> choco install googlechrome

### 3.2 文件管理

首推**Listary**，像Spotlight一样可以找文件、开程序、自定义快捷操作。而且我的个人体验比Wox+Everything的组合更好。

> choco install listary

当然，**Wox**也是有的。

> choco install wox

压缩文件管理上，当然有大名鼎鼎的**7-Zip**。喜欢更好看一点的图形界面的话，可以试试**Peazip**:

> choco install 7zip
> choco install peazip

二者都有绿色版可以提供下载与安装使用。

文件多了之后，为了方便整理和查找大文件，推荐**Treesize Free**。用树或者色块大小显示文件夹在硬盘中的占用，Spacesniffer的替代品（比它快很多）。

> choco install treesizefree

当然“洁癖”的人群也可以安装著名的清理软件**CCleaner**：

> choco install ccleaner

如果你电脑不小心中了什么删不掉的流氓软件，可以试试辣鸡软件删除器**AdwCleaner**：

> choco install adwcleaner

顺便安装上可以解决ftp需求的**Filezilla**：

> choco install filezilla

想用移动设备播放电脑上电影的话，也可以用它的**Server**：

> choco install filezilla.server

### 3.3 媒体播放

视频播放，当然用**mpv**！

> choco install mpv

就是这个[磨人的小妖精](https://imhy.zbyzbyzby.com/wordpress/?p=1167)一直让我离不开包管理软件，哼。

看图可以尝试**Irfanview**，我也不清楚这软件有啥特殊的优点，总之比自带的图库反应快一些。

> choco install irfanview

简单的视频剪辑，推荐开源软件**Shotcut**：

> choco install shotcut

不喜欢啊逗比自己的软件更新方式的，可以用Chocolatey安装它的新版**Adobe Reader DC**：

> choco install adobereader-update

看文章时，遇上生词可以用轻量翻译软件**QTranslate**：

> choco install qtranslate

# 4 更新所有软件

> 软件的精髓就是看它版本号不断更新。
>
> —— 我

可以直接在命令行中用

> choco upgrade all

更新所有软件。

整个自动化的过程**非常令人愉悦**！

（不过貌似要提前设置一个自动同意条款，看说明设置一次就行了）

你可以在桌面上建一个文本文档，里面写上

> @echo off
> choco upgrade all
> pause

保存，更改扩展名为bat，然后以管理员权限运行。

你甚至可以自己弄一个**计划任务**，定时运行更新程序。

# 5 目前Chocolatey的缺陷

还记得我最开始是为了mpv而接触的Chocolatey，然后在我顺利的安装完mpv之后，我的第一反应：

**我的mpv呢(っ °Д °;)っ**

**我安装在哪里了(っ °Д °;)っ**

**我怎么感觉什么都没有发生(っ °Д °;)っ**

也就是说，Chocolatey的安装默认是自动化的。如果软件安装包没有默认创建快捷方式，你是第一时间反应不过来软件安装在了哪里。

当然，后面我都是通过Listary找到的软件安装目录。

官网虽然提供说明，可以自定义安装的路径；但这个过程对于不同的安装方式（msi、exe等）并不通用，实际操作性并不强。

所以Chocolatey很适合安装小的、轻量级的软件。想将软件安装在非默认地址，还是建议直接去官网下载安装包吧。

另外，我记得Chocolatey在国内的网络**连接很不稳定，网速很慢**。所以不推荐没有科学上网方式的童鞋在国内使用。



------

原文链接：https://imhy.zbyzbyzby.com/wordpress/?p=1395