
# <a href="http://www.zhangxinxu.com/wordpress/2017/07/js-text-string-download-as-html-json-file/">小tip:JS前端创建html或json文件并浏览器导出下载</a>
这篇文章发布于 2017年07月5日，星期三，02:03，归类于 canvas相关, js实例。 阅读 8310 次, 今日 19 次
by zhangxinxu from http://www.zhangxinxu.com/wordpress/?p=6252
本文可全文转载，但需得到原作者书面许可，同时保留原作者和出处，摘要引流则随意。

## 一、HTML与文件下载
如果希望在前端侧直接触发某些资源的下载，最方便快捷的方法就是使用HTML5原生的download属性，例如：

<a href="large.jpg" download>下载</a>
具体介绍可参考我之前的文章：“了解HTML/HTML5中的download属性”。

但显然，如果纯粹利用HTML属性来实现文件的下载（而不是浏览器打开或浏览），对于动态内容，就无能为力。

例如，我们对页面进行分享的时候，希望分享图片是页面内容的实时截图，此时，这个图片就是动态的，纯HTML显然是无法满足我们的需求的，借助JS和其它一些HTML5特性，例如，将页面元素转换到canvas上，然后再转成图片进行下载。

但本文要介绍的下载不是图片的下载，而是文本信息的下载，所需要使用的HTML特性不是canvas，而是其它。

## 二、借助HTML5 Blob实现文本信息文件下载
如果对Blob不了解，可以先看看我好些年之前写的“理解DOMString、Document、FormData、Blob、File、ArrayBuffer数据类型”一文。

原理其实很简单，我们可以将文本或者JS字符串信息借助Blob转换成二进制，然后，作为<a>元素的href属性，配合download属性，实现下载。

代码也比较简单，如下示意（兼容Chrome和Firefox）：

var funDownload = function (content, filename) {
    // 创建隐藏的可下载链接
    var eleLink = document.createElement('a');
    eleLink.download = filename;
    eleLink.style.display = 'none';
    // 字符内容转变成blob地址
    var blob = new Blob([content]);
    eleLink.href = URL.createObjectURL(blob);
    // 触发点击
    document.body.appendChild(eleLink);
    eleLink.click();
    // 然后移除
    document.body.removeChild(eleLink);
};
其中，content指需要下载的文本或字符串内容，filename指下载到系统中的文件名称。

万般言语不达意，一枚实例来走心。

您可以狠狠地点击这里：基于funDownload实现的html格式文件下载demo

点击“下载”按钮，会把文本域中的内容全部作为一个.html后缀文件下载下来，各流程效果如下面几张图：

下载按钮点击示意

出现下载确认框（根据浏览器的设置不同也可能直接下载），然后名称默认就是test.html。

默认就是test.html名称

然后对应保存目录就多了个类似下图的文件：

保存好的test.html文件截图示意

双击该test.html文件可以在浏览器中正常浏览，说明，保存信息无误。

test.html在浏览器中访问的效果

触发下载的JS代码就几行：

button.addEventListener('click', function () {
    funDownload(textarea.value, 'test.html');	
});
以上~

## 三、结束语
不止是.html文件，.txt, .json等只要内容是文本的文件，都是可以利用这种小技巧实现下载的。

在Chrome浏览器下，模拟点击创建的<a>元素即使不append到页面中，也是可以触发下载的，但是在Firefox浏览器中却不行，因此，上面的funDownload()方法有一个appendChild和removeChild的处理，就是为了兼容Firefox浏览器。

download属性从Edge13开始支持，理论上，edge也应该支持直接JS触发的浏览器文件下载，但我手头上并无相关浏览器，无法确定真实情况如何，欢迎有条件的小伙伴帮忙测下告知结果。

就这些吧，感谢阅读！

