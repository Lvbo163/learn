# [AWK 工作原理](http://www.runoob.com/w3cnote/awk-work-principle.html)

<!-- TOC -->
- [开始块（BEGIN）](#开始块begin)
- [主体块（BODY）](#主体块body)
- [结束块（END）](#结束块end)
- [实例](#实例)
<!-- /TOC -->

AWK 工作流程可分为三个部分：
- 读输入文件之前执行的代码段（由BEGIN关键字标识）。
- 主循环执行输入文件的代码段。
- 读输入文件之后的代码段（由END关键字标识）。

命令结构:
```sh
awk 'BEGIN{ commands } pattern{ commands } END{ commands }'
```
下面的流程图描述出了 AWK 的工作流程：

![工作原理](http://www.runoob.com/wp-content/uploads/2018/10/20170719154838100.png)

- 1、通过关键字 BEGIN 执行 BEGIN 块的内容，即 BEGIN 后花括号 {} 的内容。
- 2、完成 BEGIN 块的执行，开始执行body块。
- 3、读入有 \n 换行符分割的记录。
- 4、将记录按指定的域分隔符划分域，填充域，$0 则表示所有域(即一行内容)，$1 表示第一个域，$n 表示第 n 个域。
- 5、依次执行各 BODY 块，pattern 部分匹配该行内容成功后，才会执行 awk-commands 的内容。
- 6、循环读取并执行各行直到文件结束，完成body块执行。
- 7、开始 END 块执行，END 块可以输出最终结果。
# 开始块（BEGIN）
开始块的语法格式如下：
```sh
BEGIN {awk-commands}
```
开始块就是在程序启动的时候执行的代码部分，并且它在整个过程中只执行一次。

一般情况下，我们可以在开始块中初始化一些变量。

`BEGIN` 是 `AWK` 的关键字，因此它必须是大写的。

**注意**：开始块部分是可选的，你的程序可以没有开始块部分。

# 主体块（BODY）
主体部分的语法格式如下：
```sh
/pattern/ {awk-commands}
```
对于每一个输入的行都会执行一次主体部分的命令。

默认情况下，对于输入的每一行，AWK 都会执行命令。但是，我们可以将其限定在指定的模式中。

**注意**：在主体块部分没有关键字存在。

# 结束块（END）
结束块的语法格式如下：
```sh
END {awk-commands}
```
结束块是在程序结束时执行的代码。 END 也是 AWK 的关键字，它也必须大写。 与开始块相似，结束块也是可选的。

# 实例
先创建一个名为 marks.txt 的文件。其中包括序列号、学生名字、课程名称与所得分数。
```
1)    张三    语文    80
2)    李四    数学    90
3)    王五    英语    87
```
接下来，我们将使用 AWK 脚本来显示输出文件中的内容，同时输出表头信息。
```sh
$ awk 'BEGIN{printf "序号\t名字\t课程\t分数\n"} {print}' marks.txt
```
执行以上命令，输出结果如下：
```sh
序号    名字    课程    分数
1)    张三    语文    80
2)    李四    数学    90
3)    王五    英语    87
```
程序开始执行时，AWK 在开始块中输出表头信息。在主体块中，AWK 每读入一行就将读入的内容输出至标准输出流中，一直到整个文件被全部读入为止。
