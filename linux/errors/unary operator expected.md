# Shell脚本报错unary operator expected
在匹配字符串时用了类似这样的语句
```sh
if[ $timeofday = "yes"]; then

  echo "Good morning"

  exit 0
```

报错的原因是：

如果变量`timeofday`的值为空,那么就if语句就变成了`if [  ="yes" ]`,这不是一个合法的条件。

为了避免出现这种情况，我们必须给变量加上引号`if [ "$timeofdat"="yes" ]`，这样即使是空变量也提供了合法的测试条件，`if [  " "="yes"  ]`
