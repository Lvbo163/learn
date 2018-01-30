# 健康检测

本节将依赖与[上一节 Consul cluster](https://github.com/aimuke/learn/blob/master/consul/2%20consul%20cluster.md)的环境。你现在应该已经有一个有两个node的cluster在运行。

## 检测定义 Defining Checks

和service类似，check可以通过check配置文件和调用Http API进行注册。

要启动脚本检测（script check），在Consul 0.9.0及其后续版本中agent配置文件中必须设置enable_script_checks为true。

在第二个node的Consul 配置文件目录中定义两个文件：

````ssh
vagrant@n2:~$ echo '{"check": {"name": "ping",
  "args": ["ping", "-c1", "google.com"], "interval": "30s"}}' \
  >/etc/consul.d/ping.json

vagrant@n2:~$ echo '{"service": {"name": "web", "tags": ["rails"], "port": 80,
  "check": {"args": ["curl", "localhost"], "interval": "10s"}}}' \
  >/etc/consul.d/web.json
````

第一个文件添加了一个叫做 ping的host-level检测。这个检测将每30s雨欣一次，调用 "ping -c1 google.com"。在一个基于脚本（script-based）的健康检测中，健康检测将使用启动Consul进程的用户用于来运行。
如果这个命令以一个非0（non-zero）的值退出，那么这个node将被标记为不健康的（unhealthy）。这是所有基于脚本的检测的协议。

第二个命令修改了叫做web的service。为他添加了一个健康检测。这个检测将通过curl每10秒发送一个请求来验证这个服务时可用的。和host-level的健康检测一样，如果这个脚本返回一个非0的退出码，这个服务将被标记为不健康。

现在重新启动第二个agent，用consul reload重新加载或者发送一个SIGHUP信号。你将看到如下的日志信息：

````ssh
==> Starting Consul agent...
...
    [INFO] agent: Synced service 'web'
    [INFO] agent: Synced check 'service:web'
    [INFO] agent: Synced check 'ping'
    [WARN] Check 'service:web' is now critical
````

第一行显示consul同步了一个新的定义。最后一行显示我们为web服务添加的健康检测是critical。这是因为我们实际上没有运行一个web服务，所以这个curl测试失败了。

## 检测健康状态 

现在我么您已经添加了一些简单的检测，我们可以通过http api来监控他们。首先，我们可以使用下面的命令来查找一下失败的检测（注意，这个命令可以在其他节点上运行）

````ssh
vagrant@n1:~$ curl http://localhost:8500/v1/health/state/critical
[{"Node":"agent-two","CheckID":"service:web","Name":"Service 'web' check","Status":"critical","Notes":"","ServiceID":"web","ServiceName":"web","ServiceTags":["rails"]}]
````

我们可以尝试使用DNS去查询web服务。 由于这个服务是不健康的，Consul不会返回任何结果。

````ssh
dig @127.0.0.1 -p 8600 web.service.consul
...

;; QUESTION SECTION:
;web.service.consul.        IN  A
````









