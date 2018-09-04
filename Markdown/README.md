# Markdown 语法的简要规则

## 目录

- [目录](#目录)
- [标题](#标题)
- [列表](#列表)
   - [有序列表](#有序列表)
   - [无序列表](#无序列表)
   - [待办列表](#待办列表)
- [引用](#引用)
- [图片与链接](#图片与链接)
   - [链接](#链接)
   - [图片](#图片)
- [锚点](#锚点)
- [粗体与斜体](#粗体与斜体)
- [表格](#表格)
- [代码框](#代码框)
- [分割线](#分割线)
- [图表](#图表)
   - [流程图](#流程图)
   - [时序图](#时序图)
   - [甘特图](#甘特图)


## 目录

[如何为 Markdown 文件自动生成目录？](https://www.jianshu.com/p/4721ddd27027)

<h2 name="标题">标题</h2>

标题是每篇文章都需要也是最常用的格式，在 Markdown 中，如果一段文字被定义为标题，只要在这段文字前加 # 号即可。以此类推，总共六级标题，建议在井号后加一个空格，这是最标准的 Markdown 语法。
# 一级标题

## 二级标题

### 三级标题
````
# 一级标题

## 二级标题

### 三级标题
````

## 列表

熟悉 HTML 的同学肯定知道有序列表与无序列表的区别，在 Markdown 下，列表的显示只需要在文字前加上 - 或 * 即可变为无序列表，有序列表则直接在文字前加1. 2. 3. 符号要和文字之间加上一个字符的空格。

### 无序列表
- 无序列表1
- 无序列表2
- 无序列表3

* 无序列表1
* 无序列表2
* 无序列表3

````
-无序列表1
-无序列表2
-无序列表3

*无序列表1
*无序列表2
*无序列表3
````

### 有序列表

1. 有序列表1
2. 有序列表2
3. 有序列表3

````
1. 有序列表1
2. 有序列表2
3. 有序列表3
````

### 待办列表

表示列表是否勾选状态（注意：[ ] 前后都要有空格）

- [ ] 不勾选
- [x] 勾选

```
- [ ] 不勾选
- [x] 勾选
```

## 引用

如果你需要引用一小段别处的句子，那么就要用引用的格式。

只需要在文本前加入 > 这种尖括号（大于号）即可
> 引用内容
````
> 引用内容
````

> 这是第一级引用。
>
> > 这是第二级引用。
>
> 现在回到第一级引用。

```
> 这是第一级引用。
>
> > 这是第二级引用。
>
> 现在回到第一级引用。
```

> ## 这是一个标题。
> 1. 这是第一行列表项。
> 2. 这是第二行列表项。
>
> 给出一些例子代码：
>
> return shell_exec(`echo $input | $markdown_script`);

```
> ## 这是一个标题。
> 1. 这是第一行列表项。
> 2. 这是第二行列表项。
>
> 给出一些例子代码：
>
> return shell_exec(`echo $input | $markdown_script`);
```

## 图片与链接

### 链接

Markdown支持两种形式的链接语法： **行内** 和 **参考** 两种形式.

**行内形式**是直接在后面用括号直接接上链接：

````
This is an [example link](http://example.com/).
````

**参考形式**的链接让你可以为链接定一个名称，之后你可以在文件的其他地方定义该链接的内容：

````
I get 10 times more traffic from [Google][1] than from
[Yahoo][2] or [MSN][3].

[1]: http://google.com/ "Google"
[2]: http://search.yahoo.com/ "Yahoo Search"
[3]: http://search.msn.com/ "MSN Search"
````

链接内容定义的形式为：

方括号（前面可以选择性地加上至多三个空格来缩进），里面输入链接文字

接着一个冒号

接着一个以上的空格或制表符

接着链接的网址

选择性地接着 title 内容，可以用单引号、双引号或是括弧包着

下面这三种链接的定义都是相同：

````
[foo]: http://example.com/  "Optional Title Here"
[foo]: http://example.com/  'Optional Title Here'
[foo]: http://example.com/  (Optional Title Here)
````

请注意：有一个已知的问题是 Markdown.pl 1.0.1 会忽略单引号包起来的链接 title。

链接网址也可以用方括号包起来：

````
[id]: <http://example.com/>  "Optional Title Here"
````

你也可以把 title 属性放到下一行，也可以加一些缩进，若网址太长的话，这样会比较好看：

````
[id]: http://example.com/longish/path/to/resource/here
    "Optional Title Here"
````

网址定义只有在产生链接的时候用到，并不会直接出现在文件之中。

链接辨别标签可以有字母、数字、空白和标点符号，但是并不区分大小写，因此下面两个链接是一样的：

````
[link text][a]
[link text][A]
````

隐式链接标记功能让你可以省略指定链接标记，这种情形下，链接标记会视为等同于链接文字，要用隐式链接标记只要在链接文字后面加上一个空的方括号，如果你要让 "Google" 链接到 google.com，你可以简化成：

````
[Google][]
````

然后定义链接内容：

````
[Google]: http://google.com/
````

由于链接文字可能包含空白，所以这种简化型的标记内也许包含多个单词：

````
Visit [Daring Fireball][] for more information.

[Daring Fireball]: http://daringfireball.net/
````

链接的定义可以放在文件中的任何一个地方，我比较偏好直接放在链接出现段落的后面，你也可以把它放在文件最后面，就像是注解一样。

### 图片

插入链接与插入图片的语法很像，区别在一个 !号

图片为：![](){ImgCap}{/ImgCap}

链接为：[]()

[Baidu](http://baidu.com)
![图片不存在时显示的文字](./img.png)
````
[Baidu](http://baidu.com)
![图片不存在时显示的文字](./img.png)
````

<h2 name="锚点">锚点</h2>

设置锚点链接目标：

````
<h6 id="anchorbyid">说明：</h6>
<h2 name="anchorbyname"> 锚点使用</h2>
````

添加锚点：

````
// 行内模式
[锚点](#anchorbyid "anchor alt text")

// 引用模式
[锚点][anchorbyname]
[anchorbyname]:#anchorbyname "anchor alt text"
````
> 锚点引用锚点定义可以使用name或id，具体参见html语法。锚点命名中应该避免使用空格符号

<h2 name="粗体与斜体">粗体与斜体</h2>

Markdown 的粗体和斜体也非常简单，用两个 * 包含一段文本就是粗体的语法，用一个 * 包含一段文本就是斜体的语法。

**这里是粗体** 
*这里是斜体*

````
**这里是粗体** 
*这里是斜体*
````

<h2 name="表格">表格</h2>

表格是我觉得 Markdown 比较累人的地方，例子如下：

| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |

````
| Tables        | Are           | Cool  |
| ------------- |:-------------:| -----:|
| col 3 is      | right-aligned | $1600 |
| col 2 is      | centered      |   $12 |
| zebra stripes | are neat      |    $1 |
````

<h2 name="代码框">代码框</h2>

如果你是个程序猿，需要在文章里优雅的引用代码框，在 Markdown下实现也非常简单，只需要用两个 ` 把中间的代码包裹起来。

````javascript

function test() {
  // some code
}

````

<h2 name="分割线">分割线</h2>

分割线的语法只需要三个 * 号，例如：

***
````
***
````

## 图表

Markdown 编辑器已支持绘制流程图、时序图和甘特图。通过 mermaid 实现图形的插入，点击查看 更多语法详情。

### 流程图
```graph
graph TD;
    A-->B;
    A-->C;
    B-->D;
    C-->E;
    E-->F;
    D-->F;
    F-->G;
```
效果图如下：

graph TD;
    A-->B;
    A-->C;
    B-->D;
    C-->E;
    E-->F;
    D-->F;
    F-->G;

### 时序图
```
sequenceDiagram
    participant Alice
    participant Bob
    Alice->John: Hello John, how are you?
    loop Healthcheck
        John->John: Fight against hypochondria
    end
    Note right of John: Rational thoughts 
prevail...
    John-->Alice: Great!
    John->Bob: How about you?
    Bob-->John: Jolly good!
```

sequenceDiagram
    participant Alice
    participant Bob
    Alice->John: Hello John, how are you?
    loop Healthcheck
        John->John: Fight against hypochondria
    end
    Note right of John: Rational thoughts 
prevail...
    John-->Alice: Great!
    John->Bob: How about you?
    Bob-->John: Jolly good!

### 甘特图
```
gantt
        dateFormat  YYYY-MM-DD
        title Adding GANTT diagram functionality to mermaid
        section A section
        Completed task            :done,    des1, 2014-01-06,2014-01-08
        Active task               :active,  des2, 2014-01-09, 3d
        Future task               :         des3, after des2, 5d
        Future task2               :         des4, after des3, 5d
        section Critical tasks
        Completed task in the critical line :crit, done, 2014-01-06,24h
        Implement parser and jison          :crit, done, after des1, 2d
        Create tests for parser             :crit, active, 3d
        Future task in critical line        :crit, 5d
        Create tests for renderer           :2d
        Add to mermaid                      :1d
```

gantt
        dateFormat  YYYY-MM-DD
        title Adding GANTT diagram functionality to mermaid
        section A section
        Completed task            :done,    des1, 2014-01-06,2014-01-08
        Active task               :active,  des2, 2014-01-09, 3d
        Future task               :         des3, after des2, 5d
        Future task2               :         des4, after des3, 5d
        section Critical tasks
        Completed task in the critical line :crit, done, 2014-01-06,24h
        Implement parser and jison          :crit, done, after des1, 2d
        Create tests for parser             :crit, active, 3d
        Future task in critical line        :crit, 5d
        Create tests for renderer           :2d
        Add to mermaid                      :1d
