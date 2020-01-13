1. 安装vmware， 略

2. 下载xp

http://msdn.itellyou.cn/

用迅雷下载Windows XP Professional with Service Pack 3 (x86) - CD VL (Chinese-Simplified)

下载下来后的名字是zh-hans_windows_xp_professional_with_service_pack_3_x86_cd_x14-80404.iso

3. 安装。

安装到虚拟机， 提示输入密钥时，直接忽略然后下一步，直到安装完毕。

4. 激活

不要使用小马激活，有木马，而且也不能成功激活。 网上的密钥也都失效了，这个时候有一个简单的windows xp 激活方法 ,就是修改注册表。

（1）点击“开始”——运行——输入regedit ——回车 。此时进入注册表编辑器。

（2）找到主键 Hkey_Local_Machine\Software\Microsoft\WindowsNT\CurrentVersion\WPAEvents\

（3）删除子键 lastWPAEventLoged  （右击，删除）

（4）（右击，“修改”）修改子键 OOBETimer 键值为：ff d5 71 d6 8b 6a 8d 6f d5 33 93 fd （不能复制，打字把原来的值一个一个替换，小心点不要出现错误）

（5）右击注册表中的“WPAEvents”→“权限”→“高级”→“所有者”→你的用户名→→“确定”

（6）回到“安全”→“高级”→选择列表中的“system”→“编辑” ，把“拒绝”列下的方框全部打勾即可 （勾“完全控制”即可），确定，退出。

（7）此时可以发现windows xp 已被激活，要不要重启随你，最好重启一下。

（8）小贴士：在第一次激活后，要先备份好Windows下面的System32子目录中的wpa.dbl文件。以后重新安装Windows XP后，只需要复制该文件 到Windows下System32子目录中即可激活。
