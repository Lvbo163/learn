# KV Data

除了服务法相和集成健康监测，Consul提供了一个简易的方式来使用KV存储。KV可以被用于持有动态配置信息，协助服务协调，build leader election或其他任意你想做的事。

## Simple Usage

有两种方式可以和KV store进行交互：HttpApi和Consul KV CLI。因为CLI更简单，下面将使用CLI来举例。

首先，我们查询名为"redis/config/minconns"的key对应的值

````ssh
$ consul kv get redis/config/minconns
Error! No key exists at: redis/config/minconns
````

**设置**

可以看到，我们没有获取到结果，这是由于我们没有往KV store中添加任何数据。现在我们新增一些数据:

````ssh
$ consul kv put redis/config/minconns 1
Success! Data written to: redis/config/minconns

$ consul kv put redis/config/maxconns 25
Success! Data written to: redis/config/maxconns

$ consul kv put -flags=42 redis/config/users/admin abcd1234
Success! Data written to: redis/config/users/admin

````

对于"redis/config/users/admin"，我们添加了一个值为42的flag。所有的key都支持设置一个64-bit的整数flag值。这个值consul内部不会使用，但是客户端可以使用来添加一些有意义的元数据.

**查询**

````ssh
$ consul kv get redis/config/minconns
1
````

Consul 还保存了一些额外的元数据，我们可以通过 **-detailed** 标记来获取

````ssh
$ consul kv get -detailed redis/config/minconns
CreateIndex      207
Flags            0
Key              redis/config/minconns
LockIndex        0
ModifyIndex      207
Session          -
Value            1
````

**获取所有key**

````ssh
$ consul kv get -recurse
redis/config/maxconns:25
redis/config/minconns:1
redis/config/users/admin:abcd1234
````

**删除key**

````ssh
$ consul kv delete redis/config/minconns
Success! Deleted key: redis/config/minconns
````

**使用前缀进行删除**

````ssh
$ consul kv delete -recurse redis
Success! Deleted keys with prefix: redis
````

**update**

````ssh
$ consul kv put foo bar

$ consul kv get foo
bar

$ consul kv put foo zip

$ consul kv get foo
zip
````

**CAS**

Consul可以使用一个Check-And-Set操作提供一个原子key更新。要执行一个CAS操作，需要指明-cas标签

````ssh
$ consul kv put -cas -modify-index=123 foo bar
Success! Data written to: foo

$ consul kv put -cas -modify-index=123 foo bar
Error! Did not write to foo: CAS failed
````

在上例中，第一个CAS更新成功了，是由于index是123。第二个操作失败了是由于index不在是123了。

