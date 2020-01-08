# 用rm递归删除子目录下所有.o后缀文件

``` shell
find . -name "*.o"  | xargs rm -f
```



可以通过管道命令来操作，先find出主目录 下想删除的文件，然后通过“xargs”这个构造参数列表并运行命令。

```shell
find named/ -name *.bak | xargs rm -f
```