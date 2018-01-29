
# 服务注册

## 定义服务

服务可以通过两种方式进行注册：提供服务定义文件或调用http api。通常服务定义最常用的服务注册方式。

首先，创建一个Consul配置文件夹。Consul会加载该目录下的所有配置文件。所以常用的方式是在unix系统中国建立一个目录类似 /etc/consul.d (.d 后缀暗示这个目录里面包含了一组配置文件)

其次，我们在目录中添加配置文件。假设我们的服务叫 web 运行在 80 端口上。 其服务定义如下:

```` json
{
  "service": {
    "name": "web",
    "port": 80,
    "tags": ["rails"]
  }
}
````

最后， 启动agent，并设置好配置文件目录

```` ssh
$ consul agent -dev -config-dir=/etc/consul.d
==> Starting Consul agent...
...
    [INFO] agent: Synced service 'web'
...
````

从输出可以看出，agent “synced”了这个web 服务。这说明agent从目录中加载在配置文件并且成功将服务注册到了Service catalog。

如果像注册多个服务，可以再目录下创建多个配置文件，也可以在一个配置文件中添加多个配置。

## 查询服务

服务注册后，可以通过 DNS或http api进行查询.

### DNS API

对于DNS API， 服务的DNS名称是 NAME.service.consul。By default, all DNS names are always in the consul namespace, though this is configurable. The service subdomain tells Consul we're querying services, and the NAME is the name of the service.

对于我们注册的服务，这些习俗（conventions）和设置生成一个完全合格的域名：web.service.consul

```` ssh
$ dig @127.0.0.1 -p 8600 web.service.consul
...

;; QUESTION SECTION:
;web.service.consul.        IN  A

;; ANSWER SECTION:
web.service.consul. 0   IN  A   172.20.20.11
````

从上面可以看出，返回了一条记录，该记录只有IP属性. 也可以获取完整的ip/port对。

```` ssh
$ dig @127.0.0.1 -p 8600 web.service.consul SRV
...

;; QUESTION SECTION:
;web.service.consul.        IN  SRV

;; ANSWER SECTION:
web.service.consul. 0   IN  SRV 1 1 80 Armons-MacBook-Air.node.dc1.consul.

;; ADDITIONAL SECTION:
Armons-MacBook-Air.node.dc1.consul. 0 IN A  172.20.20.11
````

也可以通过tag来对service进行过滤。查询的格式为 TAG.NAME.service.consul。

```` ssh
$ dig @127.0.0.1 -p 8600 rails.web.service.consul
...

;; QUESTION SECTION:
;rails.web.service.consul.      IN  A

;; ANSWER SECTION:
rails.web.service.consul.   0   IN  A   172.20.20.11
````

### HTTP API

```` ssh
$ curl http://localhost:8500/v1/catalog/service/web
[{"Node":"Armons-MacBook-Air","Address":"172.20.20.11","ServiceID":"web", \
    "ServiceName":"web","ServiceTags":["rails"],"ServicePort":80}]
````

上面的结构会返回所有Node中的web 服务。 如果我们想只看通过了健康检测的服务可以添加查询参数

```` ssh
$ curl 'http://localhost:8500/v1/health/service/web?passing'
[{"Node":"Armons-MacBook-Air","Address":"172.20.20.11","Service":{ \
    "ID":"web", "Service":"web", "Tags":["rails"],"Port":80}, "Checks": ...}]
````

## 服务更新

可以通过修改配置文件并发送一个SIGHUP 给agent来更新服务。这样服务不会导致服务查询的暂停，及没有服务down掉的时间。

同时，也可以通过 http api来动态的添加、删除和修改服务。











