# Shell脚本报错unary operator expected
在匹配字符串时用了类似这样的语句
```sh
if[ $timeofday = "yes"]; then

  echo "Good morning"

  exit 0
```

报错的原因是：

如果变量`timeofday`的值为空,那么就if语句就变成了`if [  ="yes" ]`, 显然 [ 和 "OK" 不相等并且缺少了 [ 符号，所以报了这样的错误。 这不是一个合法的条件。

为了避免出现这种情况，我们必须给变量加上引号`if [ "$timeofdat"="yes" ]`，这样即使是空变量也提供了合法的测试条件，`if [  " "="yes"  ]`

或者用下面的方法也能避免这种错误：
```sh
if [ "$STATUS"x == "OK"x ]; then     
    echo "OK"
fi
```
当然，x也可以是其他字符.

或者
```sh
if [[ $STATUS = "OK" ]]; then     
    echo "OK"
fi
```
