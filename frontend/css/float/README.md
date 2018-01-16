[文章来源](http://www.cnblogs.com/iyangyuan/archive/2013/03/27/2983813.html)

# CSS浮动(float,clear)通俗讲解



首先要知道，div是块级元素，在页面中独占一行，自上而下排列，也就是传说中的流。如下图：

![正常文本流](./images/common.png)
 
可以看出，即使div1的宽度很小，页面中一行可以容下div1和div2，div2也不会排在div1后边，因为div元素是独占一行的。

注意，以上这些理论，是指标准流中的div。

小菜认为，无论多么复杂的布局，其基本出发点均是：“**如何在一行显示多个div元素**”。

显然标准流已经无法满足需求，这就要用到浮动。      

**浮动可以理解为让某个div元素脱离标准流，漂浮在标准流之上，和标准流不是一个层次。**

例如，假设上图中的div2浮动，那么它将脱离标准流，但div1、div3、div4仍然在标准流当中，所以div3会自动向上移动，占据div2的位置，重新组成一个流。如图：

![dvi2浮动](./images/div2float.png)

从图中可以看出，由于对div2设置浮动，因此它不再属于标准流，div3自动上移顶替div2的位置，div1、div3、div4依次排列，成为一个新的流。又因为浮动是漂浮在标准流之上的，因此div2挡住了一部分div3，div3看起来变“矮”了。

这里div2用的是左浮动(float:left;)，可以理解为漂浮起来后靠左排列，右浮动(float:right;)当然就是靠右排列。这里的靠左、靠右是说页面的左、右边缘。

如果我们把div2采用右浮动，会是如下效果：

![dvi2浮动](./images/div2floatright.png)

此时div2靠页面右边缘排列，不再遮挡div3，读者可以清晰的看到上面所讲的div1、div3、div4组成的流。

目前为止我们只浮动了一个div元素，多个呢？

下面我们把div2和div3都加上左浮动，效果如图：

![dvi23浮动](./images/div2div3float.png)

同理，由于div2、div3浮动，它们不再属于标准流，因此div4会自动上移，与div1组成一个“新”标准流，而浮动是漂浮在标准流之上，因此div2又挡住了div4。

咳咳，到重点了，当同时对div2、div3设置浮动之后，div3会跟随在div2之后，不知道读者有没有发现，一直到现在，div2在每个例子中都是浮动的，但并没有跟随到div1之后。因此，我们可以得出一个重要结论：

**假如某个div元素A是浮动的，如果A元素上一个元素也是浮动的，那么A元素会跟随在上一个元素的后边(如果一行放不下这两个元素，那么A元素会被挤到下一行)；如果A元素上一个元素是标准流中的元素，那么A的相对垂直位置不会改变，也就是说A的顶部总是和上一个元素的底部对齐。**

**div的顺序是HTML代码中div的顺序决定的。**

**靠近页面边缘的一端是前，远离页面边缘的一端是后。**

![元素前后](./images/earlier.png)

为了帮助读者理解，再举几个例子。

假如我们把div2、div3、div4都设置成左浮动，效果如下：

![dvi234浮动](./images/div2div3div4float.png)

根据上边的结论，跟着小菜理解一遍：先从div4开始分析，它发现上边的元素div3是浮动的，所以div4会跟随在div3之后；div3发现上边的元素div2也是浮动的，所以div3会跟随在div2之后；而div2发现上边的元素div1是标准流中的元素，因此div2的相对垂直位置不变，顶部仍然和div1元素的底部对齐。由于是左浮动，左边靠近页面边缘，所以左边是前，因此div2在最左边。

假如把div2、div3、div4都设置成右浮动，效果如下：

![dvi234浮动](./images/div234floatright.png)

道理和左浮动基本一样，只不过需要注意一下前后对应关系。由于是右浮动，因此右边靠近页面边缘，所以右边是前，因此div2在最右边。

假如我们把div2、div4左浮动，效果图如下：

![dvi24浮动](./images/div24float.png)

依然是根据结论，div2、div4浮动，脱离了标准流，因此div3将会自动上移，与div1组成标准流。div2发现上一个元素div1是标准流中的元素，因此div2相对垂直位置不变，与div1底部对齐。div4发现上一个元素div3是标准流中的元素，因此div4的顶部和div3的底部对齐，并且总是成立的，因为从图中可以看出，div3上移后，div4也跟着上移，div4总是保证自己的顶部和上一个元素div3(标准流中的元素)的底部对齐。

至此，恭喜读者已经掌握了添加浮动，但还有清除浮动，有上边的基础清除浮动非常容易理解。

经过上边的学习，可以看出：元素浮动之前，也就是在标准流中，是竖向排列的，而浮动之后可以理解为横向排列。

清除浮动可以理解为打破横向排列。

清除浮动的关键字是clear，

# [官方定义如下](https://www.w3.org/wiki/CSS/Properties/clear)

> The clear property indicates which sides of an element's box(es) may not be adjacent to an earlier floating box.
>
> clear 属性指明了元素框的哪一边不能与前面的浮动框相邻。

语法：

> clear : none | left | right | both

取值：

````
none: No constraint on the box's position with respect to floats.

left: Requires that the top border edge of the box be below the bottom outer edge of any left-floating boxes that resulted from elements earlier in the source document.
  要求当前元素的上边必须在当前元素前的所有左侧浮动元素的下边之下。

right: Requires that the top border edge of the box be below the bottom outer edge of any right-floating boxes that resulted from elements earlier in the source document.
  要求当前元素的上边必须在当前元素前的所有右侧浮动元素的下边之下。

both: Requires that the top border edge of the box be below the bottom outer edge of any right-floating and left-floating boxes that resulted from elements earlier in the source document.
  要求当前元素的上边必须在当前元素之前的所有浮动元素的下边之下。
  
inherit: Takes the same specified value as the property for the element's parent.

````

定义非常容易理解，但是读者实际使用时可能会发现不是这么回事。要注意一下:该属性定义只影响定义该属性的元素, 并不影响与之关联的元素(如: div 上面定义了clear：left，是指div自己的顶部必须在之前所有左侧浮动元素的底部下方。并不是说其之前的左侧浮动元素发生了变化。)

中文的文档中经常将此处解释为left 代表左侧不能有浮动， right表示右侧不能有浮动，这个描述是不对的。
根据上边的基础，假如页面中只有两个元素div1、div2，它们都是左浮动，场景如下：

![dvi12浮动](./images/div12float.png)

此时div1、div2都浮动，根据规则，div2会跟随在div1后边，但我们仍然希望div2能排列在div1下边，就像div1没有浮动，div2左浮动那样。

这时候就要用到清除浮动（clear），如果单纯根据官方定义，读者可能会尝试这样写：在div1的CSS样式中添加clear:right;，理解为不允许div1的右边有浮动元素，由于div2是浮动元素，因此会自动下移一行来满足规则。

其实这种理解是不正确的，这样做没有任何效果。看小菜定论：

对于CSS的清除浮动(clear)，一定要牢记：**这个规则只能影响使用清除的元素本身，不能影响其他元素。**

怎么理解呢？就拿上边的例子来说，我们是想让div2移动，但我们却是在div1元素的CSS样式中使用了清除浮动，试图通过清除div1右边的浮动元素(clear:right;)来强迫div2下移，这是不可行的，因为这个清除浮动是在div1中调用的，它只能影响div1，不能影响div2。

根据小菜定论，要想让div2下移，就必须在div2的CSS样式中使用浮动。本例中div2的左边有浮动元素div1，因此只要在div2的CSS样式中使用clear:left;来指定div2元素左边不允许出现浮动元素，这样div2就被迫下移一行。

![dvi12浮动](./images/div1clear.png)

那么假如页面中只有两个元素div1、div2，它们都是右浮动呢？读者此时应该已经能自己推测场景，如下：

![dvi12浮动](./images/div12floatright.png)

此时如果要让div2下移到div1下边，要如何做呢？

同样根据小菜定论，我们希望移动的是div2，就必须在div2的CSS样式中调用浮动，因为浮动只能影响调用它的元素。

可以看出div2的右边有一个浮动元素div1，那么我们可以在div2的CSS样式中使用clear:right;来指定div2的右边不允许出现浮动元素，这样div2就被迫下移一行，排到div1下边。

![dvi12浮动](./images/div2floatclear.png)

至此，读者已经掌握了CSS+DIV浮动定位基本原理，足以应付常见的布局。

其实，万变不离其宗，只要读者用心体会，再复杂的布局都可以通过小菜总结的规律搞定。
