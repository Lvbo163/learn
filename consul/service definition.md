[原文地址](https://www.consul.io/docs/agent/services.html)

# Consul 中服务配置定义

## 单个服务配置文件定义

```` json
{
  "service": {
    "name": "redis",
    "tags": ["primary"],
    "address": "",
    "port": 8000,
    "enable_tag_override": false,
    "checks": [
      {
        "script": "/usr/local/bin/check_redis.py",
        "interval": "10s"
      }
    ]
  }
}
````
## 字段说明

服务定义必须包含 name 属性，可选的配置可以有 id, tags, address, port, check, and enable_tag_override.

### name & id

当没有提供id属性配置的时候，id将被设置为name. consul 要求在同一个node中所有服务都必须有一个唯一的ID， 所以如果 name 属性可能冲突的话，应该提供一个id属性.

### enableTagOverride

对于 0.9.3 及以前的版本，需要使用 enableTagOverride字段。1.0版本后，consul 同时支持enable_tag_override 和enableTagOverride. 但是和enableTagOverride 是被废弃了的（deprecated），而且在1.1版本中会被删除。

### tags

tag 属性是一组对consul不明确的值。他们可能被用于区分主从节点，不同的版本或其他服务等级的标签。

### address

address 属性用于指定一个服务的具体ip地址。如果没有提供该属性的话，服务的ip会默认的使用当前agent的ip。
The port field can be used as well to make a service-oriented architecture simpler to configure; this way, the address and port of a service can be discovered.

Services may also contain a token field to provide an ACL token. This token is used for any interaction with the catalog for the service, including anti-entropy syncs and deregistration.

### health check

服务可以有一个关联的服务检测。这是一个强大的特性因为他使web balancer 可以优雅的移除失败的节点。健康检测也被集成到了DNS 接口中。 
如果一个服务失败了，DSN 接口将从Service query 中移除该节点。
健康检测必须是script, HTTP, TCP or TTL type。 
如果是script， 那么必须提供脚本和间隔时间。
如果是HTTP类型，那么必须提供http和间隔时间。
如果是TCP类型，那么必须提供tcp和间隔时间。
如果是TTL类型，那么只需要提供TTL。

健康检测的名字是自动生成的（service:<service-id>）。如果注册了多个健康检测， ID将按service:<service-id>:<num> 这种格式生成。 其中 num 是从1开始递增的数字
