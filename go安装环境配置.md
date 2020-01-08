## 用法

虽然下面的内容主要是讲解如何设置 GOPROXY，但是我们也推荐你在使用 Go 模块时将 GO111MODULE 设置为 on 而不是 auto。

### Go 1.13 及以上（推荐）
打开你的终端并执行：

> $ go env -w GOPROXY=https://goproxy.cn,direct

完成。

### macOS 或 Linux
打开你的终端并执行：

> $ export GOPROXY=https://goproxy.cn

或者

> $ echo "export GOPROXY=https://goproxy.cn" >> ~/.profile && source ~/.profile

完成。

### Windows
打开你的 PowerShell 并执行：

> C:\> $env:GOPROXY = "https://goproxy.cn"

或者

1. 打开“开始”并搜索“env”
2. 选择“编辑系统环境变量”
3. 点击“环境变量…”按钮
4. 在“<你的用户名> 的用户变量”章节下（上半部分）
5. 点击“新建…”按钮
6. 选择“变量名”输入框并输入“GOPROXY”
7. 选择“变量值”输入框并输入“https://goproxy.cn”
8. 点击“确定”按钮

完成。
