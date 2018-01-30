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

**-node command-line option** cluster中的每一个节点都必须有一个唯一的名字。这个名字默认会使用node的hostname，但是我们可以通过 -node command-line option 手动的重写这个名字。

**bind address** 我们可以一个bind address。consul 会监听这个地址，并且同一个cluster中的所有其他node必须都能访问该地址。 虽然bind address 并不是必须的，但是最好还是设置一个。因为Consulh会默认的监听系统上所有的IPV4 interfaces，但是如果系统有多个IP的话，Consul启动就会失败。因为产品服务器通常有多个interfaces，所以指定一个bind address可以确保你不会bind到一个错误的interface上。

**server** 在当前cluster中我们将设置第一个节点为我们唯一的servcer节点，所以我们可以通过 server来进行指定

**-bootstrap-expect** 给server节点指明了我们希望加入的其他服务节点的数量。这个标记用于延迟日志复制的启动，直到所有的server都成功的加入之后才启动。
 
**-enable-script-checks** 设置为true使得健康检测能够执行外部脚本。
  
**-config-dir** 指定服务和检测定义文件的位置。


在启动第二个节点

```` ssh
$ vagrant ssh n2
vagrant@n2:~$ consul agent -data-dir=/tmp/consul -node=agent-two \
    -bind=172.20.20.11 -enable-script-checks=true -config-dir=/etc/consul.d
...
````

现在我们有连个节点了：一个服务节点，一个客户端节点。但是这两个agent现在还相互不知道。可以通过 consul members 命令来确认。你会发现每个agent中只有一个member可见。


## 加入Cluster

现在我们在一个新的terminal上面通过下面的命令告诉第一个agent去加入第二个agent

```` ssh
$ vagrant ssh n1
...
vagrant@n1:~$ consul join 172.20.20.11
Successfully joined cluster by contacting 1 nodes.
````

分别通过两个agent的日志，可以看到他们都收到了join信息。此时在两个agent上运行consul members，可以看到两个agent都相互知道了彼此。

```` ssh
vagrant@n2:~$ consul members
Node       Address            Status  Type    Build  Protocol
agent-two  172.20.20.11:8301  alive   client  0.5.0  2
agent-one  172.20.20.10:8301  alive   server  0.5.0  2
````

## 启动时自动加入cluster

理想情况下，数据中心中无论何时一个新的node创建，它都应该无需人工干预的自动加入cluster。Consul通过一个给定的key/value tag在AWS、Google Cloud或Azure中启用服务自动发现来促进自动加入。为了使用这些集成功能，需要在Consul的配置文件中添加 retry_join_ec2, retry_join_gce or the retry_join_azure 嵌入对象。这将使得一个新的node不需硬编码配置就可以加入一个cluster。同样的，也可以通过在启动时通过 -join flag or start_join setting 设置其他已知agent的硬编码地址。

## 查询Nodes

可以通过DNS或HTTP APi来查询服务。对于DNS API 名字格式为 NAME.node.consul 或 NAME.node.DATACENTER.consul. 如果省略了datacenter参数的话，Consul将只搜索local datacenter。

比如，通过agent-one，我们可以查询agent-two的地址

```` ssh
vagrant@n1:~$ dig @127.0.0.1 -p 8600 agent-two.node.consul
...

;; QUESTION SECTION:
;agent-two.node.consul. IN  A

;; ANSWER SECTION:
agent-two.node.consul.  0 IN    A   172.20.20.11
````

## 退出Cluster

有两种方式可以退出Cluster：优雅的退出（Ctrl +　Ｃ）或强制退出。优雅的退出将是node转换到 left 状态。否则的话，其他节点将检测到它作为 failed。这种区别将在后续进行详细介绍。

















