通过官方PPA安装Ubuntu make

sudo add-apt-repository ppa:ubuntu-desktop/ubuntu-make

sudo apt-get update

sudo apt-get install ubuntu-make

通过umake命令安装VSCode

sudo umake ide visual-studio-code

之后会确认安装路径，直接回车就可以；再会确认安装，输入a。

作者：Twoold
链接：https://www.jianshu.com/p/7ef9369f612d
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

### 安装后找不到vscode

到/home/zjf/.local/share/umake/bin
执行./visual-studio-code
然后右键鼠标Lock to Launcher
