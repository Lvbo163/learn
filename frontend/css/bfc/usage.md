[文章来源](https://www.cnblogs.com/lhb25/p/inside-block-formatting-ontext.html)

# BFC 的定义

首先 BFC 这个词语的英文的全称为 block formmating context。直译为”块级格式化上下文”。它是一个独立的渲染区域，只有Block-level box参与， 它规定了内部的Block-level Box如何布局，并且与这个区域外部毫不相干。

# BFC布局规则

1. 内部的Box会在垂直方向，一个接一个地放置。

2. Box垂直方向的距离由margin决定。属于同一个BFC的两个相邻Box的margin会发生重叠.

3. 每个元素的margin box的左边， 与包含块border box的左边相接触(对于从左往右的格式化，否则相反)。即使存在浮动也是如此。

4. BFC的区域不会与float box重叠。

5. BFC就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面的元素。反之也如此。

6. 计算BFC的高度时，浮动元素也参与计算

# 生成BFC的条件

- 根元素

- float属性不为none

- position为absolute或fixed

- display为inline-block, table-cell, table-caption, flex, inline-flex

- overflow不为visible

# BFC的作用

## 1. 自适应两栏布局

````HTML
<style>
    body {
        width: 300px;
        position: relative;
    }
 
    .aside {
        width: 100px;
        height: 150px;
        float: left;
        background: #f66;
    }
 
    .main {
        height: 200px;
        background: #fcc;
    }
</style>
<body>
    <div class="aside"></div>
    <div class="main"></div>
</body>
````

此时，效果图如下：

![image1.png](./images/two-column1.png)

根据BFC布局规则第3条：

> 每个元素的margin box的左边， 与包含块border box的左边相接触(对于从左往右的格式化，否则相反)。即使存在浮动也是如此。

因此，虽然存在浮动的元素aslide，但main的左边依然会与包含块的左边相接触。

可以看出，此时 div.main 与 div.aside 重合了。

根据BFC布局规则第四条：

> BFC的区域不会与float box重叠。

我们可以通过通过触发main生成BFC， 来实现自适应两栏布局。

当将div.main的CSS 改成这样：

````html
.main {
    overflow: hidden;
}
````

当触发main生成BFC后，这个新的BFC不会与浮动的aside重叠。因此会根据包含块的宽度，和aside的宽度，自动变窄。效果如下：

![image2.png](./images/two-column2.png)

## 2. 清除浮动

````HTML
<style>
    .par {
        border: 5px solid #fcc;
        width: 300px;
    }
 
    .child {
        border: 5px solid #f66;
        width:100px;
        height: 100px;
        float: left;
    }
</style>
<body>
    <div class="par">
        <div class="child"></div>
        <div class="child"></div>
    </div>
</body>
````
效果如图:

![image3.png](./images/clear-float1.png)

从图中可以看出来，当子元素都浮动之后，父元素的高度会塌陷。一般这时候选择给父元素设置一个高度。

根据BFC布局规则第六条：

> 计算BFC的高度时，浮动元素也参与计算

但是将div.parent元素的CSS样式改一下：

````css
.par {
    overflow: hidden;
}
````

此时效果如图所示：

![image4.png](./images/clear-float2.png)

## 3. 防止垂直 margin 重叠

````HTML
<style>
    p {
        color: #f55;
        background: #fcc;
        width: 200px;
        line-height: 100px;
        text-align:center;
        margin: 100px;
    }
</style>
<body>
    <p>Haha</p>
    <p>Hehe</p>
</body>
````

则此时，根据margin-collapse的理论，P之间会发生margin合并，即二者之间的margin值为100px，而不是200px。

如图所示：

![image5.png](./images/margin-collapse1.png)

那么，如果说，我们想要达到P之间的margin值为200px，即二者之间不发生margin合并的话。将P设置到不同的BFC中去。则此时二者之间的margin就不会发生合并。

根据BFC布局规则第二条：

> Box垂直方向的距离由margin决定。属于同一个BFC的两个相邻Box的margin会发生重叠

````HTML
<style>
    .wrap {
        overflow: hidden;
    }
    p {
        color: #f55;
        background: #fcc;
        width: 200px;
        line-height: 100px;
        text-align:center;
        margin: 100px;
    }
</style>
<body>
    <p>Haha</p>
    <div class="wrap">
        <p>Hehe</p>
    </div>
</body>
````

此时的效果如下图所示：

![image6.png](./images/margin-collapse2.png)

则此时P的垂直间距才为200px。

## 4. 总结

其实以上的几个例子都体现了BFC布局规则第五条：

> BFC就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面的元素。反之也如此。

因为BFC内部的元素和外部的元素绝对不会互相影响，因此， 当BFC外部存在浮动时，它不应该影响BFC内部Box的布局，BFC会通过变窄，而不与浮动有重叠。
同样的，当BFC内部有浮动时，为了不影响外部元素的布局，BFC计算高度时会包括浮动的高度。避免margin重叠也是这样的一个道理
