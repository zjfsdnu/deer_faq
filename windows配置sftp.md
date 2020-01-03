使用微软刚出的https://github.com/PowerShell/Win32-OpenSSH/wiki/Install-Win32-OpenSSH

https://github.com/PowerShell/Win32-OpenSSH/releases

\### 账号登陆

ssh-keygen -t rsa 生成密钥对 复制C:\Users\Administrator\.ssh\id_rsa 登录时使用此私钥

如果使用账号的话 使用windows账号

\### 修改配置导致mac登录失败

/Users/zhangjinfu/.ssh/known_hosts

删除对应的内容即可

\### 修改默认目录

[How to Change Default SFTP location in Open SSH windows](https://github.com/PowerShell/Win32-OpenSSH/issues/730)

另一种方案是修改注册表

\### 替代软件

[SFTP Servers](https://www.sftp.net/servers)

[buru](https://buruserver.com/download/free)