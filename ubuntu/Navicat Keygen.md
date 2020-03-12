参考：https://gitee.com/andisolo/navicat-keygen

http://ask.mykeji.net/ruanjiantuijian/258.html



## 前提条件

1. 请确保你安装了下面几个库：

   - `capstone`
   - `keystone`
   - `rapidjson`

   你可以通过下面的命令来安装它们：

   ```shell
   $ sudo apt-get install libcapstone-dev
   
   $ sudo apt-get install cmake
   $ git clone https://github.com/keystone-engine/keystone.git
   $ cd keystone
   $ mkdir build
   $ cd build
   $ ../make-share.sh
   $ sudo make install
   $ sudo ldconfig
   
   $ sudo apt-get install rapidjson-dev
   ```

2. 你的gcc支持C++17特性



## 破解

```
mkdir ~/Desktop/DoNavicat

cd ~/Desktop/DoNavicat

wget "http://cloud.mykeji.net/Linux_soft/navicat15/navicat15-premium-cs.AppImage"

mkdir bin

mkdir navicat15-premium-cs

sudo mount -o loop navicat15-premium-cs.AppImage navicat15-premium-cs

cp -r navicat15-premium-cs navicat15-premium-cs-patched

sudo umount navicat15-premium-cs

rm -rf navicat15-premium-cs

cd bin

wget "http://cloud.mykeji.net/Linux_soft/navicat15/navicat-patcher"

chmod +x navicat-patcher

./navicat-patcher ../navicat15-premium-cs-patched

cd ..

wget "http://cloud.mykeji.net/Linux_soft/navicat15/appimagetool-x86_64.AppImage"

chmod +x appimagetool-x86_64.AppImage

./appimagetool-x86_64.AppImage navicat15-premium-cs-patched navicat15-premium-cs-patched.AppImage

chmod +x navicat15-premium-cs-patched.AppImage

#运行此软件，也可以鼠标双击运行(在非root环境下运行)
./navicat15-premium-cs-patched.AppImage

cd bin

wget "http://cloud.mykeji.net/Linux_soft/navicat15/navicat-keygen"

chmod +x bin/navicat-keygen

./navicat-keygen --text ../RegPrivateKey.pem

选择类别、语言、主版本号（这里要选对 否则无法激活）

会生成一个序列号

输入用户名

输入组织名

断网

在navicat中找到注册 输入key生成的序列号

点击 激活

选择 手动激活

复制 请求码 到keygen，连按两次回车结束

最终你会得到一个base64编码的 激活码。

将之复制到 手动激活 的窗口，然后点击 激活

最后的清理：

rm ~/Desktop/DoNavicat/navicat15-premium-cs.AppImage

rm -rf ~/Desktop/DoNavicat/navicat15-premium-cs-patched

mv ~/Desktop/DoNavicat/navicat15-premium-cs-patched.AppImage ~/Desktop/DoNavicat/navicat15-premium-cs.AppImage
```