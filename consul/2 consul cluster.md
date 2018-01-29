# Consul 组

前面已经介绍了怎么使用Consul。但是还没有展示Consul怎么扩展成一个可伸缩的产品级服务发现系统。本节中间创建第一个真正的有多个member的cluster。

当一个agent启动的时候，它是孤立的，并不知道其他的节点。要知道cluster中的其他member，agent必须加入一个已经存在的cluster。要加入一个cluster，agent需要知道一个存在的member。
加入后，这个agent会后这个member交谈并快速的发现cluster中的其他member。agent可以加入任何其他的agent，并不一定必须是server模式的agent。


## Start Agents

为了模拟，我们用Vagrant启动两个node。

```` ssh
$ vagrant up
$ vagrant ssh n1
vagrant@n1:~$ consul agent -server -bootstrap-expect=1 \
    -data-dir=/tmp/consul -node=agent-one -bind=172.20.20.10 \
    -enable-script-checks=true -config-dir=/etc/consul.d
...
````

启动node后，在启动consul agent。

cluster中的每一个节点都必须有一个唯一的名字。这个名字默认会使用node的hostname，但是我们可以通过 -node command-line option 手动的重写这个名字。

我们可以一个bind address。consul 会监听这个地址，并且同一个cluster中的所有其他node必须都能访问该地址。 虽然bind address 并不是必须的，但是最好还是设置一个。因为Consulh会默认的监听系统上所有的IPV4 interfaces，但是如果系统有多个IP的话，Consul启动就会失败。因为产品服务器通常有多个interfaces，所以指定一个bind address可以确保你不会bind到一个错误的interface上。

在当前cluster中我们将设置第一个节点为我们唯一的servcer节点，所以我们可以通过 server来进行指定

 -bootstrap-expect 给server节点指明了我们希望加入的其他服务节点的数量。这个标记用于延迟日志复制的启动，直到所有的server都成功的加入之后才启动。
 
  -enable-script-checks 设置为true使得健康检测能够执行外部脚本。
  
  -config-dir 指定服务和检测定义文件的位置。


在启动第二个节点

```` ssh
$ vagrant ssh n2
vagrant@n2:~$ consul agent -data-dir=/tmp/consul -node=agent-two \
    -bind=172.20.20.11 -enable-script-checks=true -config-dir=/etc/consul.d
...
````

现在我们有连个节点了：一个服务节点，一个客户端节点。但是这两个agent现在还相互不知道。可以通过 consul members 命令来确认。你会发现每个agent中只有一个member可见。








