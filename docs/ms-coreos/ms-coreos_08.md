第八章 容器编排

随着容器成为现代应用开发和部署的基础，部署数百个或数千个容器到单一数据中心集群或数据中心集群变得必要。集群可以是本地集群或云端集群。为了在大规模下部署和管理容器，必须有一个良好的容器编排系统。

本章将覆盖以下主题：

+   现代应用部署的基础

+   使用 Kubernetes、Docker Swarm 和 Mesos 进行容器编排及其核心概念、安装和部署

+   流行的编排解决方案比较

+   使用 Docker Compose 进行应用定义

+   打包的容器编排解决方案——AWS 容器服务、Google 容器引擎和 CoreOS Tectonic

现代应用部署

我们在第一章中介绍了微服务的基础，CoreOS 概述。在基于云的应用开发中，基础设施被视为“牲畜”而非“宠物”([`www.slideshare.net/randybias/pets-vs-cattle-the-elastic-cloud-story`](http://www.slideshare.net/randybias/pets-vs-cattle-the-elastic-cloud-story))。这意味着基础设施通常是商品化的硬件，可能会出现故障，高可用性需要在应用层或应用编排层进行处理。高可用性可以通过负载均衡器和编排系统的组合来实现，这些系统监控服务的健康状况，采取必要的行动，例如在服务崩溃时重新启动服务。容器具有隔离和封装的优良特性，使得独立的团队能够作为容器开发各自的组件。

企业可以采用按需增长模式，根据需要扩展容器。在大规模管理数百或数千个容器时，必须使用容器编排系统。以下是容器编排系统的一些特点：

+   它将不同的基础设施硬件视为一个集合，并将其表示为应用程序的单一资源

+   它根据用户约束调度容器，并以最有效的方式利用基础设施

+   它动态地扩展容器

+   它保持服务的高可用性

应用程序定义与容器编排之间有着密切的关系。应用程序定义通常是一个清单文件，描述了应用程序中的容器以及容器暴露的服务。容器编排是基于应用程序定义进行的。容器编排器在资源上运行，这些资源可以是虚拟机或裸机。通常，容器运行的节点安装有容器优化的操作系统，例如 CoreOS、DCOS 和 Atomic。以下图像展示了应用程序定义、容器编排和容器优化节点之间的关系，并给出了每个类别中一些解决方案的示例：

![](img/00458.jpg)

容器编排

容器编排的基本要求是高效地将 M 个容器部署到 N 个计算资源中。

以下是容器编排系统应该解决的一些问题：

+   它应该高效地调度容器，并为用户提供足够的控制，以便根据需要调整调度参数

+   它应该提供跨集群的容器网络

+   服务应该能够动态发现彼此

+   编排系统应该能够处理服务故障

我们将在接下来的章节中介绍 Kubernetes、Docker Swarm 和 Mesos。Fleet 是 CoreOS 内部用于容器编排的工具。Fleet 功能非常简约，适合在 CoreOS 中部署关键系统服务。对于非常小的部署，如果需要，也可以使用 Fleet 进行容器编排。

Kubernetes

Kubernetes 是一个开源的容器编排平台。最初由谷歌发起，现在多个供应商正在共同参与这个开源项目。谷歌一直使用容器来开发和部署应用程序，且在其内部数据中心有一个叫 Borg 的系统（[`research.google.com/pubs/pub43438.html`](http://research.google.com/pubs/pub43438.html)）用于集群管理。Kubernetes 借鉴了 Borg 的许多概念，并结合了现有的现代技术。Kubernetes 轻量级，能够在几乎所有环境中运行，目前在行业中有着广泛的应用。

Kubernetes 的概念

Kubernetes 有一些独特的概念，深入了解这些概念会对理解 Kubernetes 的架构非常有帮助。

Pods

Pods 是一组在同一个节点上调度并需要紧密协作的容器。Pod 中的所有容器共享 IPC 命名空间、网络命名空间、UTS 命名空间和 PID 命名空间。通过共享 IPC 命名空间，容器可以使用 IPC 机制相互通信。

通过共享网络命名空间，容器可以使用套接字相互通信，并且 Pod 中的所有容器共享一个 IP 地址。通过共享 UTS 命名空间，卷可以挂载到 Pod 上，所有容器都可以访问这些卷。以下是一些常见的应用部署模式与 Pods 结合使用的例子：

+   Sidecar 模式：一个例子是应用容器和日志容器，或者像 Git 同步器这样的应用同步容器。

+   Ambassador 模式：在这种模式下，应用容器和代理容器一起工作。当应用容器发生变化时，外部服务仍然可以像以前一样与代理容器通信。一个例子是一个带有 redis 代理的 redis 应用容器。

+   Adapter 模式：在这种模式下，存在一个应用容器和适配器容器，适配不同的环境。一个例子是一个日志容器，它作为适配器，随着不同云服务提供商的变化而变化，但适配器容器的接口保持不变。

Kubernetes 中最小的单元是 Pod，Kubernetes 负责调度 Pods。

以下是一个包含 NGINX 容器和 Git 助手容器的 Pod 定义示例：

`apiVersion: v1 kind: Pod metadata:   name: www spec:   containers:   - name: nginx     image: nginx   - name: git-monitor     image: kubernetes/git-monitor     env:     - name: GIT_REPO       value: http://github.com/some/repo.git`

网络

Kubernetes 采用每个 Pod 一个 IP 的方式。这种方式避免了容器共享主机 IP 地址时，使用 NAT 访问容器服务的痛苦。Pod 中的所有容器共享相同的 IP 地址。跨节点的 Pods 可以使用不同的技术进行通信，例如云提供商的云路由、Flannel、Weave、Calico 等。最终目标是将网络作为 Kubernetes 中的插件，用户可以根据需求选择插件。

服务

服务是 Kubernetes 提供的一种抽象，用于逻辑上组合提供相似功能的 Pods。通常，标签（Labels）作为选择器，用来从 Pods 创建服务。由于 Pods 是短暂的，Kubernetes 会创建一个服务对象，并为其分配一个永远保持不变的 IP 地址。Kubernetes 负责多个 Pod 的负载均衡。

以下是一个示例服务：

`{     "kind": "Service",     "apiVersion": "v1",     "metadata": {         "name": "my-service"     },     "spec": {         "selector": {             "app": "MyApp"         },         "ports": [             {                 "protocol": "TCP",                 "port": 80,                 "targetPort": 9376             }         ]     } }`

在前面的例子中，我们创建了一个`my-service`服务，将所有带有`Myapp`标签的 Pod 组合在一起。任何访问`my-service`服务的 IP 地址和端口号`80`的请求都会被负载均衡到所有带有`Myapp`标签的 Pods，并被重定向到端口号`9376`。

根据服务类型，需要在内部或外部发现服务。内部发现的示例是 Web 服务需要与数据库服务通信。外部发现的示例是将 Web 服务暴露给外部世界。

对于内部服务发现，Kubernetes 提供两种选项：

+   环境变量：创建新 Pod 时，可以从旧服务导入环境变量。这允许服务之间进行通信。此方法强制执行服务创建中的顺序。

+   DNS：每个服务都注册到 DNS 服务；通过此服务，新服务可以找到并与其他服务通信。Kubernetes 为此提供了`kube-dns`服务。

对于外部服务发现，Kubernetes 提供两种选项：

+   NodePort：通过此方法，Kubernetes 通过节点 IP 地址的特殊端口（30000-32767）公开服务。

+   负载均衡器：通过此方法，Kubernetes 与云提供商交互以创建负载均衡器，将流量重定向到 Pod。目前此方法在 GCE 上可用。

Kubernetes 架构

以下图表显示了 Kubernetes 架构的不同软件组件及其相互交互方式：

![](img/00460.jpg)

以下是关于 Kubernetes 架构的一些注释：

+   主节点托管 Kubernetes 控制服务。从节点运行 Pod 并由主节点管理。可以有多个主节点以实现冗余和扩展主服务。

+   主节点运行关键服务，例如调度器、复制控制器和 API 服务器。从节点运行关键服务，例如 Kubelet 和 Kube-proxy。

+   用户与 Kubernetes 的交互通过 Kubectl 进行，它使用标准的 Kubernetes 公开的 API。

+   Kubernetes 调度器负责根据 Pod 清单中指定的约束在节点上调度 Pod。

+   复制控制器是维护 Pod 的高可用性和根据复制控制器清单创建多个 Pod 实例所必需的。

+   主节点中的 API 服务器与每个从节点的 Kubelet 进行通信，以部署 Pod。

+   Kube-proxy 负责服务重定向和负载均衡流量到 Pod。

+   Etcd 用作所有节点之间通信的共享数据存储库。

+   DNS 用于服务发现。

Kubernetes 安装

Kubernetes 可以安装在裸金属、虚拟机或云提供商（如 AWS、GCE 和 Azure）上。我们可以根据这些系统中的任何操作系统来决定主机操作系统的选择。在本章中，所有示例都将使用 CoreOS 作为主机操作系统。由于 Kubernetes 由多个组件组成，如 API 服务器、调度器、复制控制器、kubectl 和 kubeproxy，这些组件分布在主节点和从节点之间，手动安装这些单独的组件会非常复杂。Kubernetes 及其用户提供了自动化某些节点设置和软件安装的脚本。截至 2015 年 10 月，Kubernetes 的最新稳定版本是 1.0.7，本章中的所有示例均基于 1.0.7 版本。

非 CoreOS Kubernetes 安装

对于非 CoreOS 基础的 Kubernetes 安装，过程很简单：

1.  从[`github.com/kubernetes/kubernetes/releases`](https://github.com/kubernetes/kubernetes/releases)找到所需的 Kubernetes 发布版本。

1.  下载适当版本的`kubernetes.tar.gz`并解压。

1.  将`KUBERNETES_PROVIDER`设置为以下之一（AWS、GCE、Vagrant 等）

1.  在`cluster`目录中更改集群大小和任何其他配置参数。

1.  运行`cluster/kube-up.sh`。

Kubectl 安装

Kubectl 是与 Kubernetes 交互的 CLI 客户端。Kubectl 默认未安装。Kubectl 可以安装在客户端机器或 Kubernetes 主节点上。

可以使用以下命令安装 kubectl。需要确保 kubectl 版本与 Kubernetes 版本匹配：

`ARCH=linux; wget https://storage.googleapis.com/kubernetes-release/release/v1.0.7/bin/$ARCH/amd64/kubectl`

如果在客户端机器上已安装 Kubectl，我们可以使用以下命令将请求代理到 Kubernetes 主节点：

`ssh -f -nNT -L 8080:127.0.0.1:8080 core@<control-external-ip>`

Vagrant 安装

我使用了[`github.com/pires/kubernetes-vagrant-coreos-cluster`](https://github.com/pires/kubernetes-vagrant-coreos-cluster)中的过程，在 Vagrant 环境中使用 CoreOS 创建了一个 Kubernetes 集群。我最初在 Windows 上尝试过此操作。由于我遇到了[`github.com/pires/kubernetes-vagrant-coreos-cluster/issues/158`](https://github.com/pires/kubernetes-vagrant-coreos-cluster/issues/158)中提到的问题，我转向了在 Ubuntu Linux 上运行的 Vagrant 环境。

以下是命令：

`Git clone https://github.com/pires/kubernetes-vagrant-coreos-cluster.git``Cd coreos-container-platform-as-a-service/vagrant``Vagrant up`

以下输出显示了两个运行中的 Kubernetes 节点：

![](img/00462.jpg)

GCE 安装

我使用了[`github.com/rimusz/coreos-multi-node-k8s-gce`](https://github.com/rimusz/coreos-multi-node-k8s-gce)中的过程，在 GCE 中使用 CoreOS 创建了一个 Kubernetes 集群。

以下是命令：

`git clone https://github.com/rimusz/coreos-multi-node-k8s-gce``cd coreos-multi-node-k8s-gce`

在 `settings` 文件中，修改 `project`、`zone`、`node count` 以及其他必要的配置。

按照相同顺序运行以下三个脚本：

`1-bootstrap_cluster.sh``2-get_k8s_fleet_etcd.sh``3-install_k8s_fleet_units.sh`

以下输出展示了由三个节点组成的集群：

![](img/00464.jpg)

以下输出展示了 Kubernetes 客户端和服务端版本：

![](img/00465.jpg)

以下输出展示了运行在主节点和从节点上的 Kubernetes 服务：

![](img/00468.jpg)

本示例中使用的脚本通过 Fleet 协调 Kubernetes 服务。如前面图示所示，API 服务器、控制器和调度器运行在主节点上，而 kubelet 和代理运行在从节点上。每个从节点上都有三个副本的 kubelet 和 kube-proxy。

AWS 安装

我使用了 [`coreos.com/kubernetes/docs/latest/kubernetes-on-aws.html`](https://coreos.com/kubernetes/docs/latest/kubernetes-on-aws.html) 中的流程在 AWS 上创建了运行 CoreOS 的 Kubernetes 集群。

第一部是安装 kube-aws 工具：

`Git clone https://github.com/coreos/coreos-kubernetes/releases/download/v0.1.0/kube-aws-linux-amd64.tar.gz`

解压并将 kube-aws 复制到可执行路径。确保 `~/.aws/credentials` 已更新为你的凭证。

创建一个默认的 `cluster.yaml` 文件：

`curl --silent --location https://raw.githubusercontent.com/coreos/coreos-kubernetes/master/multi-node/aws/cluster.yaml.example > cluster.yaml`

修改 `cluster.yaml` 文件，填写你的 `keyname`、`region` 和 `externaldnsname`；`externaldnsname` 只对外部访问重要。

要部署集群，我们可以执行以下操作：

`Kube-aws up`

以下输出展示了属于 Kubernetes 集群的两个节点：

![](img/00470.jpg)

Kubernetes 应用示例

以下图示展示了我们将用来说明前面章节中讨论的不同 Kubernetes 概念的留言本示例。此示例基于 [`kubernetes.io/v1.1/examples/guestbook/README.html`](http://kubernetes.io/v1.1/examples/guestbook/README.html) 的参考：

![](img/00472.jpg)

以下是关于这个留言本应用的一些说明：

+   该应用使用 php 前端与 redis 主从后端存储留言本数据库。

+   前端 RC 创建了三个 `kubernetes/example-guestbook-php-redis` 容器实例。

+   Redis-master RC 创建了一个 redis 容器实例。

+   Redis-slave RC 创建了两个 `kubernetes/redis-slave` 容器实例。

在本示例中，我使用了上一节中创建的集群，该集群在 AWS 上运行 Kubernetes，使用 CoreOS。集群中有一个主节点和两个从节点。

让我们来看看这些节点：

![](img/00474.jpg)

在本示例中，Kubernetes 集群使用 flannel 跨 pods 进行通信。以下输出展示了分配给集群中每个节点的 flannel 子网：

![](img/00477.jpg)

以下是启动应用程序所需的命令：

`kubectl create -f redis-master-controller.yaml``kubectl create --validate=false -f redis-master-service.yaml``kubectl create -f redis-slave-controller.yaml``kubectl create --validate=false -f redis-slave-service.yaml``kubectl create -f frontend-controller.yaml``kubectl create --validate=false -f frontend-service.yaml`

让我们查看 Pod 列表：

![](img/00478.jpg)

上述输出显示了三个 php 前端实例，一个 Redis 主节点实例和两个 Redis 从节点实例。

让我们查看 RC 列表：

![](img/00480.jpg)

上述输出显示了每个 Pod 的副本数。前端有三个副本，`redis-master`有一个副本，`redis-slave`有两个副本，正如我们所要求的那样。

让我们看一下服务列表：

![](img/00481.jpg)

在上述输出中，我们可以看到组成留言簿应用程序的三个服务。

对于内部服务发现，本示例使用了 `kube-dns`。以下输出显示了正在运行的 `kube-dns` RC：

![](img/00484.jpg)

对于外部服务发现，我修改了示例以使用 `NodePort` 机制，其中一个内部端口被暴露。以下是新的 `frontend-service.yaml` 文件：

`apiVersion: v1 kind: Service metadata:   name: frontend   labels:     name: frontend spec:   # 如果你的集群支持它，取消注释以下内容以自动创建   # 外部负载均衡 IP 用于前端服务。   type: NodePort   ports:     # 服务应该监听的端口   - port: 80   selector:     name: frontend`

以下是在启动 `NodePort` 类型的前端服务时的输出。输出显示该服务通过端口 `30193` 被暴露：

![](img/00486.jpg)

一旦我们通过 AWS 防火墙暴露端口 `30193`，就可以按以下方式访问留言簿应用程序：

![](img/00488.jpg)

让我们看看 `Node1` 中的应用程序容器：

![](img/00489.jpg)

让我们看看 `Node2` 中的应用程序容器：

![](img/00492.jpg)

上述输出包含了三个前端实例，一个 Redis 主节点实例和两个 Redis 从节点实例。

为了说明复制控制器如何维持 Pod 的副本数，我在一个节点中停止了留言簿前端 Docker 容器，如下图所示：

![](img/00493.jpg)

Kubernetes RC 检测到 Pod 未运行并重新启动了 Pod。这可以通过查看其中一个留言簿 Pod 的重启次数来看到，如下图所示：

![](img/00495.jpg)

为了进行一些基本的调试，我们可以登录到 Pod 或容器中。以下示例显示了如何进入 Pod：

![](img/00496.jpg)

上述输出显示了在留言簿 Pod 中的 IP 地址，这与分配给该节点的 Flannel 子网一致，正如本示例开头的 Flannel 输出所示。

另一个对调试有用的命令是 `kubectl logs`，如下所示：

![](img/00476.jpg)

Kubernetes 与 Rkt

默认情况下，Kubernetes 使用容器运行时 Docker。Kubernetes 的架构允许其他容器运行时，如 Rkt，与 Kubernetes 一起工作。目前正在进行积极的工作（[`github.com/kubernetes/kubernetes/tree/master/docs/getting-started-guides/rkt`](https://github.com/kubernetes/kubernetes/tree/master/docs/getting-started-guides/rkt)）以将 Kubernetes 与 Rkt 和 CoreOS 集成。

Kubernetes 1.1 更新

Kubernetes 于 2015 年 11 月发布了 1.1 版本（[`blog.kubernetes.io/2015/11/Kubernetes-1-1-Performance-upgrades-improved-tooling-and-a-growing-community.html`](http://blog.kubernetes.io/2015/11/Kubernetes-1-1-Performance-upgrades-improved-tooling-and-a-growing-community.html)）。1.1 版本的重要新增功能包括性能提升、自动扩展和用于批处理任务的作业对象。

Docker Swarm

Swarm 是 Docker 的原生编排解决方案。以下是 Docker Swarm 的一些特点：

+   与其单独管理每个 Docker 节点，不如将整个集群作为一个整体来管理。

+   Swarm 内置调度器，负责决定容器在集群中的位置。Swarm 使用用户特定的约束条件和亲和性（[`docs.docker.com/swarm/scheduler/filter/`](https://docs.docker.com/swarm/scheduler/filter/)）来决定容器的放置。约束条件可以是 CPU 和内存，而亲和性是将相关容器分组在一起的参数。Swarm 还提供将其调度器移除，并与其他调度器（如 Kubernetes）协作的选项。

以下图示展示了 Docker Swarm 架构：

![](img/00188.jpg)

以下是关于 Docker Swarm 架构的一些说明：

+   Swarm 主节点负责根据调度算法、约束条件和亲和性调度 Docker 容器。支持的算法有 spread、`binpack` 和 `random`。默认算法是 spread。可以并行运行多个 Swarm 主节点，以提供高可用性。Spread 调度算法用于均匀分配工作负载。Binpack 调度算法则是在将任务调度到其他节点之前，先充分利用每个节点。

+   Swarm 代理在每个节点上运行，并与 Swarm 主节点通信。

+   有多种方法可以让 Swarm 工作节点发现 Swarm 主节点。发现是必要的，因为 Swarm 主节点和代理程序运行在不同的节点上，并且 Swarm 代理不是由 Swarm 主节点启动的。Swarm 代理和 Swarm 主节点需要相互发现，才能了解它们属于同一个集群。可用的发现机制包括 Docker hub、Etcd、Consul 等。

+   Docker Swarm 与 Docker machine 集成，简化了 Docker 节点的创建。Docker Swarm 与 Docker compose 集成，用于多容器应用的编排。

+   随着 Docker 1.9 版本的发布，Docker Swarm 与多主机 Docker 网络集成，使得跨主机调度的容器能够相互通信。

Docker Swarm 安装

本示例的前提是安装 Docker 1.8.1 和 Docker-machine 0.5.0。我使用[`docs.docker.com/swarm/install-w-machine/`](https://docs.docker.com/swarm/install-w-machine/)中的步骤创建了一个带有两个 Docker Swarm 代理节点的单个 Docker Swarm 主节点。以下是步骤：

1.  创建发现令牌。

1.  使用创建的发现令牌创建 Swarm 主节点。

1.  使用创建的发现令牌创建两个 Swarm 代理节点。

通过将环境变量设置为`swarm-master`，如以下命令所示，我们可以使用常规的 Docker 命令控制 Docker Swarm 集群：

`eval $(docker-machine env --swarm swarm-master)`

我们来看一下 Swarm 集群中`docker info`的输出：

![](img/00056.jpg)

上述输出显示集群中有三个节点（一个主节点和两个代理节点），并且集群中正在运行四个容器。`swarm-master`节点有两个容器，而`swarm-agent`节点每个有一个容器。这些容器用于管理 Swarm 服务。应用容器只会在 Swarm 代理节点中调度。

我们来看一下主节点中的单个容器。这显示了主节点和代理服务正在运行：

![](img/00191.jpg)

我们来看一下运行在代理节点中的容器。这显示了正在运行的`swarm agent`：

![](img/00193.jpg)

Docker Swarm 示例

为了演示 Docker Swarm 容器编排，我们先启动四个`nginx`容器：

`docker run -d --name nginx1 nginx``docker run -d --name nginx2 nginx``docker run -d --name nginx3 nginx``docker run -d --name nginx4 nginx`

从以下输出中，我们可以看到四个容器均匀分布在`swarm-agent-00`和`swarm-agent-01`之间。这里使用了默认的`spread`调度策略：

![](img/00195.jpg)

以下输出显示了包括主节点和两个代理节点在内的整个集群的容器总数。总计八个容器，包括 Swarm 服务容器和 nginx 应用容器：

![](img/00252.jpg)

Mesos

Apache Mesos 是一个开源的集群软件。Mesosphere 的 DCOS 是 Apache Mesos 的商业版。Mesos 结合了集群操作系统（Clustering OS）和集群管理器（Cluster Manager）。集群操作系统负责将来自多个不同计算机的资源表示为一个统一的资源，以便可以在其上调度应用程序。集群管理器负责在集群中调度任务。相同的集群可以用于不同的工作负载，如 Hadoop 和 Spark。在 Mesos 中有两级调度。第一层调度负责在框架之间进行资源分配，框架则负责在该框架内调度任务。每个框架是一个应用程序类别，如 Hadoop、Spark 等。对于通用应用程序，最好的框架是 Marathon。Marathon 是一个分布式 INIT 和高可用（HA）系统，用于调度容器。Chronos 框架类似于 Cron 作业，适合运行需要定期执行的较短工作负载。Aurora 框架则为复杂任务提供了更加细粒度的控制。

以下图片展示了 Mesos 架构中的不同层次：

![](img/00198.jpg)

比较 Kubernetes、Docker Swarm 和 Mesos

尽管这些解决方案（Kubernetes、Docker Swarm 和 Mesos）都用于应用程序编排，但它们在方法和使用场景上存在很多差异。我尝试基于它们的最新版本总结这些差异。所有这些编排解决方案都在积极开发中，因此功能集可能会发生变化。此表格已更新至 2015 年 10 月：

| 特性 | Kubernetes | Docker Swarm | Mesos |
| --- | --- | --- | --- |
| 部署单元 | Pods | 容器 | 进程或容器 |
| 容器运行时 | Docker 和 Rkt。 | Docker。 | Docker；Mesos 正在讨论与 Rkt 的集成。 |
| 网络 | 每个容器都有一个 IP 地址，并且可以使用外部网络插件。 | 最初，通过一个公共代理 IP 地址进行端口转发。使用 Docker 1.9 后，采用了 Overlay 网络和每个容器的 IP 地址，并且可以使用外部网络插件。 | 最初，通过一个公共代理 IP 地址进行端口转发。目前，使用每个容器的 IP 地址，并与 Calico 集成。 |
| 工作负载 | 同质工作负载。通过命名空间，可以创建多个虚拟集群。 | 同质工作负载。 | 可以并行运行多个框架，如 Marathon、Aurora 和 Hadoop。 |
| 服务发现 | 可以使用基于环境变量的发现或 Kube-dns 进行动态发现。 | 静态，修改 `/etc/hosts`。未来计划使用 DNS 方法。 | 基于 DNS 的方法，用于动态发现服务。 |
| 高可用性 | 使用复制控制器，服务具有高度可用性。服务扩展可以轻松完成。 | 服务的高可用性尚未实现。 | 框架处理服务的高可用性。例如，Marathon 具有`Init.d`系统来运行容器。 |
| 成熟度 | 相对较新。首次生产版本发布于几个月前。 | 相对较新。首次生产版本发布于几个月前。 | 稳定性较好，广泛应用于大型生产环境中。 |
| 复杂性 | 简单。 | 简单。 | 设置稍微有些困难。 |
| 使用场景 | 更适合同质化工作负载。 | 提供 Docker 前端界面，使得 Docker 用户无需学习新的管理界面也能轻松使用。 | 适合异质化工作负载。 |

Kubernetes 可以作为框架在 Mesos 之上运行。在这种情况下，Mesos 为 Kubernetes 提供一级调度，而 Kubernetes 则调度和管理已调度的应用程序。这个项目（[`github.com/mesosphere/kubernetes-mesos`](https://github.com/mesosphere/kubernetes-mesos)）致力于在 Mesos 之上运行 Kubernetes。

目前正在进行的工作是将 Docker Swarm 与 Kubernetes 集成，以便 Kubernetes 可以作为集群的调度器和过程管理器运行，同时用户仍然可以使用 Docker Swarm 的 Docker 界面来管理容器。

应用程序定义

当一个应用程序由多个容器组成时，使用单一的 JSON 或 YAML 文件表示每个容器的属性及其依赖关系非常有用，这样可以整体实例化应用程序，而不是单独实例化应用程序中的每个容器。应用程序定义文件负责定义多容器应用程序。Docker-compose 定义了应用程序文件和运行时，根据应用程序文件实例化容器。

Docker-compose

Docker-compose 为你提供了应用程序定义格式，当我们运行该工具时，Docker-compose 负责解析应用程序定义文件并实例化容器，处理所有依赖关系。

Docker-compose 具有以下优点和使用场景：

+   它提供了一种简单的方法来指定包含多个容器及其约束和亲和性的应用程序清单

+   它与 Dockerfile、Docker Swarm 和多主机网络集成良好

+   相同的 compose 文件可以通过环境变量适应不同的环境

单节点应用程序

以下示例展示了如何构建一个包含 WordPress 和 MySQL 容器的多容器 WordPress 应用程序。

以下是定义容器及其属性的`docker-compose.yml`文件：

`wordpress:   image: wordpress   ports:    - "8080:80"   environment:     WORDPRESS_DB_HOST: "composeword_mysql_1:3306"     WORDPRESS_DB_PASSWORD: mysql mysql:   image: mysql   environment:     MYSQL_ROOT_PASSWORD: mysql`

以下命令展示了如何使用 `docker-compose` 启动应用：

`docker-compose –p composeword –f docker-compose.yml up -d`

以下是前面命令的输出：

![](img/00200.jpg)

容器名称以 `-p` 选项中指定的关键字为前缀。在上面的示例中，我们使用了 `composeword_mysql_1` 作为主机名，且 IP 地址是通过此容器动态生成并更新到 `/etc/hosts` 中。

以下输出显示了 Wordpress 应用运行的容器：

![](img/00202.jpg)

以下输出是 Wordpress 容器中 `/etc/hosts` 的输出，显示 MySQL 容器的 IP 地址已被动态更新：

![](img/00220.jpg)

一个多节点应用

我使用了 [`docs.docker.com/engine/userguide/networking/get-started-overlay/`](https://docs.docker.com/engine/userguide/networking/get-started-overlay/) 中的示例，通过 `docker-compose` 创建了一个跨多个节点的 Web 应用。在这个例子中，`docker-compose` 与 Docker Swarm 和 Docker 多主机网络集成。

这个示例的前提是需要有一个正常工作的 Docker Swarm 集群以及 Docker 版本 1.9+。

以下命令创建了多主机计数器应用。该应用有一个 Web 容器作为前端，一个 Mongo 容器作为后端。必须在 Swarm 集群中执行这些命令：

`docker-compose –p counter –x-networking up -d`

以下是前面命令的输出：

![](img/00204.jpg)

以下输出显示了作为本应用一部分创建的叠加网络 `counter`：

![](img/00206.jpg)

以下输出显示了 Swarm 集群中运行的容器：

![](img/00207.jpg)

以下输出显示了 Swarm 集群信息，共有五个容器——其中三个是 Swarm 服务容器，两个是前述应用容器：

![](img/00209.jpg)

以下输出显示了工作中的 Web 应用：

![](img/00211.jpg)

打包的容器编排解决方案

部署一个大规模分布式微服务应用需要许多组件，以下是一些重要的组件：

+   一个基础设施集群

+   一个容器优化操作系统

+   一个内置调度器、服务发现和网络功能的容器编排工具

+   存储集成

+   支持多租户能力并具备认证功能

+   一个可以简化管理的 API 层次结构

像 Amazon 和 Google 这样的云服务提供商已经有了管理虚拟机的生态系统，他们的做法是将容器和容器编排集成到他们的 IaaS 产品中，以便容器能够与他们的其他工具良好配合。AWS 容器服务和 Google 容器引擎属于这一类。CoreOS 的重点是开发一个安全的容器优化操作系统，并为分布式应用程序开发提供开源工具。CoreOS 意识到，将他们的产品与 Kubernetes 集成将为客户提供一个集成的解决方案，Tectonic 提供了这个集成解决方案。

还有一些其他项目，如 OpenStack Magnum（[`github.com/openstack/magnum`](https://github.com/openstack/magnum)）和 Cisco 的 Mantl（[`mantl.io/`](https://mantl.io/)），也属于此类托管容器编排服务。我们在本章没有涉及这些内容。

AWS 容器服务

AWS EC2 容器服务（ECS）是 AWS 提供的一种容器编排服务。以下是该服务的一些关键特性：

+   ECS 创建并管理容器启动的节点集群。用户只需指定集群的规模。

+   容器健康状况通过运行在节点上的容器代理进行监控。容器代理与主节点通信，主节点负责做出所有与服务相关的决策。这确保了容器的高可用性。

+   ECS 负责在集群中调度容器。调度器 API 实现为插件，这使得与其他调度器（如 Marathon 和 Kubernetes）的集成成为可能。

+   ECS 与其他 AWS 服务如 Cloudformation、ELB、日志记录、Volume 管理等良好集成。

安装 ECS 及示例

ECS 可以通过 AWS 控制台、AWS CLI 或 ECS CLI 进行控制。以下示例中，我使用了 ECS CLI，ECS CLI 可以通过此链接的过程安装（[`docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_installation.html`](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_installation.html)）。

我使用了以下示例（[`docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_tutorial.html`](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_tutorial.html)）来创建一个包含两个容器（WordPress 和 MySQL）的 WordPress 应用程序，使用 compose YML 文件。

以下是步骤：

1.  创建 ECS 集群。

1.  将应用程序作为服务部署到集群上。

1.  集群规模或服务规模可以根据需求动态调整。

以下命令显示 WordPress 应用程序的运行容器：

`ecs-cli ps`

以下是前述命令的输出：

![](img/00213.jpg)

我们可以使用 `ecs-cli` 命令来扩展应用程序。以下命令将每个容器的规模设置为二：

`ecs-cli compose --file hello-world.yaml scale 2`

以下输出显示了此时正在运行的容器。正如我们所看到的，容器已经扩展到两个：

![](img/00214.jpg)

注意

注意：MySQL 容器通常使用单一主节点和多个从节点进行扩展。

我们还可以登录到每个 AWS 节点并查看正在运行的容器。以下输出显示了其中一个 AWS 节点中的三个容器。两个是应用程序容器，第三个是 ECS 代理容器，它负责容器监控并与主节点通信：

![](img/00216.jpg)

注意

注意：要登录每个节点，我们需要使用`ec2-user`作为用户名，并使用创建集群时使用的私钥。

为了演示高可用性，我尝试停止容器或节点。由于容器代理监控每个节点中的容器，容器会被重新调度。

Google 容器引擎

Google 容器引擎是 Google 提供的集群管理和容器编排解决方案，构建在 Kubernetes 之上。与使用 Kubernetes 运行容器集群相比，GCE 有以下差异或优势：

+   节点集群由 Google 容器引擎自动创建。用户只需指定集群大小以及 CPU 和内存需求。

+   Kubernetes 由多个独立的服务组成，如 API 服务器、调度器和代理等，这些服务需要安装才能使 Kubernetes 系统正常工作。Google 容器引擎负责创建带有适当服务的 Kubernetes 主节点，并在代理节点中安装其他 Kubernetes 服务。

+   Google 容器引擎与其他 Google 服务（如 VPC 网络、日志记录、自动扩展、负载均衡等）集成得很好。

+   可以使用 Docker Hub、Google 容器注册表或本地注册表来存储容器镜像。

安装 GCE 和示例

[`cloud.google.com/container-engine/docs/before-you-begin`](https://cloud.google.com/container-engine/docs/before-you-begin)中的步骤可用于安装 gcloud 容器组件和 kubectl。容器也可以使用 GCE 控制台进行管理。

我使用[`cloud.google.com/container-engine/docs/tutorials/guestbook`](https://cloud.google.com/container-engine/docs/tutorials/guestbook)中的步骤创建了一个包含三个服务的留言簿应用程序。这个应用程序与之前指定的 Kubernetes 应用程序示例中的应用程序相同。

以下是步骤：

1.  创建一个具有所需集群大小的节点集群。这将自动创建一个 Kubernetes 主节点，并在节点上安装适当的代理服务。

1.  使用复制控制器和服务文件部署应用程序。

1.  集群可以根据需求动态调整大小。

以下是我创建的集群。该集群中有四个节点，数量由`NUM_NODES`指定：

![](img/00217.jpg)

以下命令显示了正在运行的服务，其中包括 frontend、redis-master 和 redis-slave。Kubernetes 服务也在主节点上运行：

`Kubectl get services`

以下是前一个命令的输出：

![](img/00342.jpg)

由于前端服务已与 GCE 负载均衡器集成，因此还有一个外部 IP 地址。通过外部 IP 地址可以访问 guestbook 服务。以下命令显示了与负载均衡器关联的端点列表：

`Kubectl describe services frontend`

以下是前一个命令的输出：

![](img/00222.jpg)

要调整集群大小，首先需要找到与集群关联的实例组并对其进行调整。以下命令显示了与 guestbook 关联的实例组：

![](img/00224.jpg)

使用实例组，我们可以按如下方式调整集群大小：

![](img/00225.jpg)

显示集群大小为四的初始输出是在调整集群大小后生成的。

我们可以登录到各个节点，并使用常规的 Docker 命令查看节点中启动的容器。在以下输出中，我们看到一个 `redis-slave` 实例和一个前端实例在该节点中运行。其他容器为 Kubernetes 基础设施容器：

![](img/00227.jpg)

CoreOS Tectonic

Tectonic 是 CoreOS 的商业产品，集成了 CoreOS 和 CoreOS 的开源组件（Etcd、Fleet、Flannel、Rkt 和 Dex）以及 Kubernetes。通过 Tectonic，CoreOS 将其其他商业产品如 CoreUpdate、Quay 仓库和 Enterprise CoreOS 集成到 Tectonic 中。

计划将 Kubernetes API 作为其在 Tectonic 中的原始形式公开。CoreOS 开源项目的开发将继续进行，并且最新的软件会更新到 Tectonic 中。

下图展示了 Tectonic 的不同组件：

![](img/00228.jpg)

Tectonic 为你提供分布式可信计算（DTM），在所有层级（包括硬件和软件）提供安全性。以下是一些独特的区别点：

+   在固件级别，可以嵌入客户密钥，这允许客户验证系统中运行的所有软件。

+   嵌入固件中的安全密钥可以验证引导加载程序以及 CoreOS。

+   诸如 Rkt 这样的容器可以通过其镜像签名进行验证。

+   可以使用嵌入在 CPU 主板中的 TPM 硬件模块，使日志防篡改。

总结

在本章中，我们讲解了容器编排的重要性，并深入介绍了流行的容器编排解决方案，如 Kubernetes、Docker Swarm 和 Mesos。许多公司提供集成的容器编排解决方案，我们也介绍了一些流行的解决方案，例如 AWS 容器服务、Google 容器引擎和 CoreOS Tectonic。对于本章介绍的所有技术，均提供了安装和示例，供您尝试使用。客户可以选择使用集成的容器编排解决方案，或者手动将编排解决方案集成到他们的基础设施中。影响选择的因素包括灵活性、与内部解决方案的集成以及成本。下一章我们将介绍 OpenStack 与容器和 CoreOS 的集成。

参考文献

+   Kubernetes 页面: [`kubernetes.io/`](http://kubernetes.io/)

+   Mesos: [`mesos.apache.org/`](http://mesos.apache.org/) 和 [`mesosphere.com/`](https://mesosphere.com/)

+   Docker Swarm: [`docs.docker.com/swarm/`](https://docs.docker.com/swarm/)

+   Kubernetes on CoreOS: [`coreos.com/kubernetes/docs/latest/`](https://coreos.com/kubernetes/docs/latest/)

+   Google 容器引擎: [`cloud.google.com/container-engine/`](https://cloud.google.com/container-engine/)

+   AWS ECS: [`aws.amazon.com/ecs/`](https://aws.amazon.com/ecs/)

+   Docker Compose: [`docs.docker.com/compose`](https://docs.docker.com/compose)

+   Docker machine: [`docs.docker.com/machine/`](https://docs.docker.com/machine/)

+   Tectonic: [`tectonic.com`](https://tectonic.com)

+   Tectonic 分布式可信计算: [`tectonic.com/blog/announcing-distributed-trusted-computing/`](https://tectonic.com/blog/announcing-distributed-trusted-computing/)

进一步阅读和教程

+   使用 Kubernetes 和 CoreOS 进行容器编排: [`www.youtube.com/watch?v=tA8XNVPZM2w`](https://www.youtube.com/watch?v=tA8XNVPZM2w)

+   比较编排解决方案: [`radar.oreilly.com/2015/10/swarm-v-fleet-v-kubernetes-v-mesos.html`](http://radar.oreilly.com/2015/10/swarm-v-fleet-v-kubernetes-v-mesos.html), [`www.slideshare.net/giganati/orchestration-tool-roundup-kubernetes-vs-docker-vs-heat-vs-terra-form-vs-tosca-1`](http://www.slideshare.net/giganati/orchestration-tool-roundup-kubernetes-vs-docker-vs-heat-vs-terra-form-vs-tosca-1), 和 [`www.openstack.org/summit/vancouver-2015/summit-videos/presentation/orchestration-tool-roundup-kubernetes-vs-heat-vs-fleet-vs-maestrong-vs-tosca`](https://www.openstack.org/summit/vancouver-2015/summit-videos/presentation/orchestration-tool-roundup-kubernetes-vs-heat-vs-fleet-vs-maestrong-vs-tosca)

+   Mesosphere 介绍: [`www.digitalocean.com/community/tutorials/an-introduction-to-mesosphere`](https://www.digitalocean.com/community/tutorials/an-introduction-to-mesosphere)

+   Docker 和 AWS ECS: 0[`medium.com/aws-activate-startup-blog/cluster-based-architectures-using-docker-and-amazon-ec2-container-service-f74fa86254bf#.afp7kixga`](https://medium.com/aws-activate-startup-blog/cluster-based-architectures-using-docker-and-amazon-ec2-container-service-f74fa86254bf#.afp7kixga)
