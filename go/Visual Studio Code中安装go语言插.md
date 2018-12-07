[原文](https://blog.csdn.net/mscf/article/details/82696416)
# Visual Studio Code中安装go语言插件
2018年09月14日 01:50:41 薛定谔之猫

在vscode中安装go语言插件的过程中，提示工具不完整，之后点击全部安装按钮但是出错，通过搜索得到一些方法，但还是行不通。主要存在两个问题，首先golang.org被拦在墙外，借了梯子后依然无法成功安装，分析得出是在请求golang.org获取工具代码时出现了重定向，go命令并没有检测出这一变化，导致包括微软妹子写的工具也因依赖关系安装失败。

解决的办法是顺藤摸瓜，通过请求安装失败提示的URL，重定向到github站点，根据go源代码中import的包路径，在用户目录下创建对应的目录，然后把整个仓库clone到本地，之后就能成功安装了。

最后显示的提示的信息如下，精诚所至，金石为开：
```ssh
All tools successfully installed. You're ready to Go :).
```
在GOPATH中执行如下命令：
```ssh
mkdir src\golang.org\x
```
之后将两个仓库克隆至x目录下：

```ssh
git clone https://github.com/golang/lint.git
git clone https://github.com/golang/tools.git
```
打开vscode，编辑或保存一个.go文件触发插件对缺少工具的提示，之后自动安装就能成功了。

也可以根据插件安装过程中使用的go命令获取远程代码安装，或者进入本地代码目录直接安装本地克隆好的代码，不过都没有插件自动安装的方便，被墙的情况下可以根据依赖的提示逐步采用手动的方法。
