# [kubelet运行机制分析](https://www.jianshu.com/p/639903727a60)

<!-- TOC -->
- [1. 节点管理](#1-节点管理)
- [2. Pod管理](#2-pod管理)
- [3. 容器健康检查](#3-容器健康检查)
- [4. cAdvisor资源监控](#4-cadvisor资源监控)
<!-- /TOC -->

在Kubernetes集群中，每个Node节点（又称Minion）上都会启动一个Kubelet服务进行。该进程用于处理Master节点下发到本节点的任务，管理Pod及Pod中的容器。每个Kubelet进程会在API Server上注册节点自身信息，定期向Master节点汇报节点资源的使用情况，并通过cAdvise监控容器和节点资源。

## 1. 节点管理
节点通过设置Kubelet的启动参数“--register-node”，来决定是否想API Server注册自己。如果该参数值为true，那么Kubelet将试着通过API Server注册自己。作为自注册，Kubelet启动时还包含下列参数：
- --api-servers Master节点API Server的位置；
- --kubeconfig 告诉Kubelet在哪儿可以找到用于访问API Server的证书；
- --cloud-provider 告诉Kubelet如何从云服务商（IAAS）哪里读取到和自己相关的元数据

当前每个Kubelet被授予创建和修改任何节点的权限。但是在实践中，它仅仅创建和修改自己。

Kubelet在启动时通过API Server注册节点信息，并定时向API Server发送节点新消息，API Server在接收到这些信息后，将这些信息写入etcd。通过Kubelet的启动参数“--node-status-update-frequency”设置Kubelet每隔多少时间向API Server报告节点状态，默认为10秒。

## 2. Pod管理
Kubelet通过以下几种方式获取自身Node上所要运行的Pod清单。

1. 文件：Kubelet启动参数“--config”指定的配置文件目录下的文件（默认目录为”/etc/kubernets/manifests/“）。通过--file-check-frequency设置检查该文件目录的时间间隔，默认为20秒。
2. HTTP端点（URL）：通过”--manifest-url“参数设置。通过--http-check-frequency设置检查该HTTP端点的数据时间间隔，默认为20秒。
3. API Server：Kubelet通过API Server监听etcd目录，同步Pod清单。

所有以非API Server方式创建的Pod都叫作Static Pod。Kubelet将Static Pod的状态汇报给API Server，API Server为该Static Pod创建一个Mirror Pod和其相匹配。Mirror Pod的状态将真实反映Static Pod的状态。当Static Pod被删除时，与之相对应的Mirror Pod也会被删除。

Kubelet通过API Server Client使用Watch加List的方式监听”/registry/node/$当前节点名称“和”/registry/pods“目录，将获取的信息同步到本地缓存中。

Kubelet监听etcd，所有针对Pod的操作将会被Kubelet监听到。如果发现有新的绑定到本节点的Pod，则按照Pod清单的要求创建该Pod。

如果发现本地的Pod被修改，则Kubelet会做出相应的修改，比如删除Pod中的某个容器时，则通过Docker Client删除该容器。

如果发现删除本节点的Pod，则删除相应的Pod，并通过Docker Client删除Pod中的容器。

Kubelet读取监听到的信息，如果是创建和修改Pod任务，则做如下处理：
1. 为该Pod创建一个数据目录。
2. 从API Server读取该Pod清单。
3. 为该Pod挂载外部卷（Extenal Volume）。
4. 下载Pod用到的Secret。
5. 检查已经运行在节点中的Pod，如果该Pod没有容器或Pause容器（”kubernetes/pause“镜像创建的容器）没有启动，则先停止Pod里所有容器的进程。如果在Pod中有需要删除的容器，则删除这些容器。
6. 用”kubernetes/pause“镜像为每个Pod创建一个容器。该Pause容器用于接管Pod中所有其他容器的网络。每创建一个新的Pod，Kubelet都会先创建一个Pause容器，然后创建其他容器。”kubernetes/pause“镜像大概为200KB，是一个非常小的容器镜像。
7. 为Pod中的每个容器做如下处理：
> 为容器计算一个hash值，然后用容器的名字去Docker查询对应容器的hash值。若查到容器，且两者hash值不同，则停止Docker中容器的进程，并停止与之关联的Pause容器进程；若两者相同，则不做任何处理；
>
>如果容器被终止了，且容器没有指定的restartPolicy（重启策略），则不做任何处理；
>
>调用Docker Client下载容器镜像，调用Docker Client运行容器。

## 3. 容器健康检查
Pod通过两类探针来检查容器的健康状态:
1. 一个是LivenessProbe探针，用于判断容器是否健康，告诉Kubelet一个容器什么时候处于不健康的状态。如果LivenessProbe探针探测到容器不健康，则Kubelet将删除该容器，并根据容器的重启策略做相应的处理。如果一个容器不包含LivenessProbe探针，那么Kubelet认为该容器的LivenessProbe探针返回的值永远是”Seccess“；
2. 另一类是ReadinessProbe探针，用于判断容器是否启动完成，且准备接受请求。如果ReadinessProbe探针检测到失败，则Pod的状态将被修改。Endpoint Controller将从Service的Endpoint中删除包含该容器所在Pod的IP地址的Endpoint条目。

Kubelet定期调用容器的LivenessProbe探针来诊断容器的健康状况。LivenessProbe包含以下三种实现方式：

1. ExecAction：在容器内部执行一个命令，如果该命令的退出状态吗为0，则表示容器健康。
2. TCPSocketAction：通过容器的IP地址和端口号执行TCP检查，如果端口能被访问，则表示容器健康。
3. HTTPGetAction：通过容器的IP地址和端口号及路径调用HTTP Get方法，如果响应的状态码大于等于200且小于等于400，则认为容器状态健康。

LivenessProbe探针包含在Pod定义的spec.containers.{某个容器}中。

## 4. cAdvisor资源监控
cAdvisor是一个开源的分析容器资源使用率和性能特性的代理工具。它是因为容器而产生的，因此自然支持Docker容器。在Kubernetes项目中，cAdvisor被集成到Kubernetes代码中。cAdvisor自动查找所有在其所在节点上的容器，自动采集CPU、内存、文件系统和网络使用的统计信息。cAdvisor通过它所在节点机的Root容器，采集并分析该节点机的全面的使用情况。

---
[原文地址](https://www.jianshu.com/p/639903727a60)
