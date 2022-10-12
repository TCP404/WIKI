# Hello Kubernetes
开源
用于管理云平台中多个主机上的容器化应用的编排部署、规划、更新等运维工作。

## K8S 架构
![k8s架构](https://blogpicure.oss-cn-shenzhen.aliyuncs.com/blog/illustration-pic/k8s/k8s%E6%9E%B6%E6%9E%84.jpg)

K8S 管理的是一个个集群，一个集群中含有 Master 节点和其他 Node 节点。

- Master 节点一般包括 4 个组件：ApiServer、Scheduler、Controller-Manager、ETCD。
    - **ApiServer**：所有服务访问的统一入口，上接其余组件、下连 ETCD，提供各类 API 处理、鉴权、Node 上的 kubelet 通信等。只有 ApiServer 会链接 ETCD；
    - **Controller-Manager**：控制器管理器，控制着各类 Controller，通过控制器模式，致力于将当前状态变为期望的状态；
    - **Scheduler**：负责接受任务，选择合适的节点分配任务；
    - **ETCD**：整个集群的数据库，存储 K8S 集群所有需要持久化的重要信息，可以不部署在 Master 节点，单独搭建。

- Node 节点一般包括 3 个组件：Docker、kubelet、kube-proxy。
    - **kubelet**：直接跟容器引擎交互，实现管理容器的生命周期；理解 k8s 的指令，翻译成容器的指令；
    - **kube-proxy**：负责网络打通、负载均衡，即将写入规则到 iptables，现在是写到 IPVS。
    - **Docker**：具体跑应用的容器引擎，不一定非得是 Docker，但一般都是 Docker；

- 其他插件：
    - **CoreDNS**：Core 公司的 DNS，可以为集群中的 SVC 创建一个域名和 IP 的对应解析，这样可以直接通过域名访问 SVC 了；
    - **Dashboard**：给 K8S 提供一个 B/S 结构的访问体系；
    - **Ingress Controller**：可以实现七层代理，官方只能实现四层代理；
    - **Federation**：提供一个可以跨集群中心、多 K8S 的统一管理功能；
    - **Prometheus**：监控 K8S 集群
    - **ELK**：集群日志统一分析介入平台

## Pod
Pod 分为自主 *自主式 Pod* 和 *控制器管理的 Pod*。自主式 Pod 如果挂了不会自己重启等等，无法自维护。控制器管理的 Pod 则可以通过控制器实现自重启等功能，维持集群的高可用。

Pod 是 K8S 中独有的概念，她是 K8S 管理的最小单位。

每一个 Pod 都包含一个 Pause 的容器，Pod 内其他容器都连接到这个 Pause 容器。由此 Pod 内其他容器实现了共享同一个网络环境、存储卷等。

即如果 Golang 要访问 NGINX 的 8080 端口，只需要访问 localhost:8080 即可，不需要做什么映射。

### 无状态
**副本控制器（RC，ReplicationController）** 用于确保容器应用的副本数始终保持在用户定义的副本数。如果有容器异常退出，会自动创建新的 Pod 来替代，而异常退出的容器会被回收。

**副本集合（RS，ReplicaSet）** 跟 RC 没有本质不同，而且 RS 还支持集合式的 Selector。但是 RS 不支持滚动更新。

**Deployment** 支持滚动更新，但不负责 Pod 创建，所以一般使用 Deployment 管理 RS，这样 RS 就能滚动更新了；即先创建新版本的容器，等新版本创建成功了，把流量切过来了，才把旧版本删除，这样能使得用户无感更新。 

**水平自动扩展（HPA，Horizontal Pod Autoscaling）** 仅适用于 Deployment 和 RS，支持根据 Pod 的 CPU 利用率、内存利用率、用户自定义的 metric 自动扩容缩容

### 有状态
**StatefulSet** 是为了解决有状态服务的问题，其应用场景包括：

    - 稳定的持久化存储，即 Pod 重新调度后还是能访问到相同的持久化数据，基于 PVC 实现
    - 稳定的网络标志，即 Pod 重新调度后其 PodName 和 HostName 不变，基于 Headless Service（即没有 Cluster IP 的 Service）来实现
    - 有序部署、有序扩展，即 Pod 是有顺序的，在部署或者扩展的时候要依据定义的顺序依次进行，基于 init containers 来实现
    - 有序收缩、有序删除

### 定时任务 Job、CronJob
Job 负责批处理任务，即仅执行一次的任务，她保证批处理任务的一个或多个 Pod 成功结束。

Cron Job 管理基于时间的 Job，即 **在给定时间点只运行一次的任务** 或 **周期性地在给定时间点运行**