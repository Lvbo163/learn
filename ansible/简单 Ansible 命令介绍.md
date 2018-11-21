# 简单 Ansible 命令介绍 
**作者： Shusain 译者： LCTT geekpi | 2018-04-08 15:18**

在这个 Ansible 教程中，我们将学习一些基本的 Ansible 命令的例子，我们将用它来管理基础设施。所以让我们先看看一个完整的 Ansible 命令的语法：
```ssh
$ ansible <group> -m <module> -a <arguments>
```
在这里，我们可以用单个主机或用 <group> 代替一组主机，<arguments> 是可选的参数。现在我们来看看一些 Ansible 的基本命令。

**检查主机的连通性**

检查主机连接的命令是：
```ssh
$ ansible <group> -m ping
```
**重启主机**
```ssh
$ ansible <group> -a "/sbin/reboot"
```
**检查主机的系统信息**

Ansible 收集所有连接到它主机的信息。要显示主机的信息，请运行：
```ssh
$ ansible <group> -m setup | less
```
其次，通过传递参数来从收集的信息中检查特定的信息：
```ssh
$ ansible <group> -m setup -a "filter=ansible_distribution"
```
**传输文件**

对于传输文件，我们使用模块 copy ，完整的命令是这样的：
```ssh
$ ansible <group> -m copy -a "src=/home/dan dest=/tmp/home"
```

## 管理用户
要管理已连接主机上的用户，我们使用一个名为 user 的模块，并如下使用它。

**创建新用户**
```ssh
$ ansible <group> -m user -a "name=testuser password=<encrypted password>"
```
**删除用户**
```ssh
$ ansible <group> -m user -a "name=testuser state=absent"
```
注意： 要创建加密密码，请使用 "mkpasswd -method=sha-512"。

## 更改权限和所有者
要改变已连接主机文件的所有者，我们使用名为 file 的模块，使用如下。

**更改文件权限**
```ssh
$ ansible <group> -m file -a "dest=/home/dan/file1.txt mode=777"
```
**更改文件的所有者**
```ssh
$ ansible <group> -m file -a "dest=/home/dan/file1.txt mode=777 owner=dan group=dan"
```

## 管理软件包
我们可以通过使用 yum 和 apt 模块来管理所有已连接主机的软件包，完整的命令如下：

**检查包是否已安装并更新**
```ssh
$ ansible <group> -m yum -a "name=ntp state=latest"
```
**检查包是否已安装，但不更新**
```ssh
$ ansible <group> -m yum -a "name=ntp state=present"
```
**检查包是否是特定的版本**
```ssh
$ ansible <group> -m yum -a "name= ntp-1.8 state=present"
```
**检查包是否没有安装**
```ssh
$ ansible <group> -m yum -a "name=ntp state=absent"
```

## 管理服务
要管理服务，我们使用模块 service ，完整命令如下：

**启动服务**
```ssh
$ansible <group> -m service -a "name=httpd state=started"
```
**停止服务**
```ssh
$ ansible <group> -m service -a "name=httpd state=stopped"
```
**重启服务**
```ssh
$ ansible <group> -m service -a "name=httpd state=restarted"
```
