# yes --重复输出字符串，自动回答命令行提示

# 用途说明

yes命令用于重复输出字符串（outputa string repeatedly until killed）。这个命令可以帮你自动回答命令行提示，例如，进入一个含有多个文件的目录，执行 "yes | rm -i *"，所有的 rm:remove regular empty file `xxx'? 提示都会被自动回答 y。这在编写脚本程序的时候会很用处。yes命令还有另外一个用途，可以用来生成大的文本文件。

# 常用参数

yes命令不指定参数时，不断的输出y；指定字符串参数时，就不断的输出该字符串。要终止输出，必须杀掉该进程，比如按Ctrl+C，或killall yes。（Repeatedly output a line with all specifiedSTRING(s), or ‘y’.）比如：要不断输出n时，输入yes n。

**使用示例**

示例一 删除文件时自动回答y
```sh
[root@web ~]# ls -l *.txt

-rw-r--r-- 1 rootroot     7 11-28 11:54 1.txt

-rw-r--r-- 1 rootroot 10217 07-06 13:10 data.txt

[root@web ~]# yes | rm -i *.txt
rm：是否删除一般文件  "1.txt" | rm -i.txt”? rm：是否删除 一般文件“data.txt”? 

[root@web ~]# yes| rm -i *.txt     

rm: lstat “*.txt”失败: 没有那个文件或目录

[root@web ~]# ls -l *.txt     

ls: *.txt: 没有那个文件或目录

[root@web ~]#
```
 

示例二 生成大的文本文件

下面的脚本把yes命令输出的内容保存到文件中，然后1秒钟之后停止输出。在这台测试机器上，生成了一个93M的文件。

```sh
#!/bin/sh  
  
yes hello >hello.txt &  
PID=$!  
  
sleep 1  
kill $PID  
  
ls -l hello.txt  
```

```
[root@web ~]# cat yes.sh

#!/bin/sh

yes hello>hello.txt &

PID=$!

sleep 1

kill $PID

ls -l hello.txt

[root@web ~]# ./yes.sh

-rw-r--r-- 1 rootroot 93003776 11-28 14:02 hello.txt

./yes.sh:line 9:  5771 已终止                 yes hello > hello.txt

[root@web ~]# ./yes.sh

-rw-r--r-- 1 rootroot 95346688 11-28 14:09 hello.txt

./yes.sh:line 9:  7072 已终止                 yes hello > hello.txt

[root@web ~]# ./yes.sh

-rw-r--r-- 1 rootroot 0 11-28 14:09 hello.txt

[root@web ~]# ./yes.sh

-rw-r--r-- 1 rootroot 0 11-28 14:09 hello.txt

[root@web ~]# ./yes.sh

-rw-r--r-- 1 rootroot 0 11-28 14:09 hello.txt

[root@web ~]# ./yes.sh

-rw-r--r-- 1 rootroot 94040064 11-28 14:10 hello.txt

[root@web ~]# ./yes.sh

-rw-r--r-- 1 rootroot 0 11-28 14:10 hello.txt

[root@web ~]# ./yes.sh

-rw-r--r-- 1 rootroot 0 11-28 14:10 hello.txt

[root@web ~]#

``` 
