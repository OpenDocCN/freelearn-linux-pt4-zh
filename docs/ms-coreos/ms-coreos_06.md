第六章 CoreOS 存储管理

存储是分布式基础设施中的关键组件。容器技术的初步关注点是无状态容器，存储由传统技术如 NAS 和 SAN 管理。无状态容器通常是像 NGINX 和 Node.js 这样的 Web 应用程序，不需要持久化数据。近年来，越来越多的关注点转向有状态容器，并且许多新技术正在开发中以实现有状态容器。有状态容器是如 SQL 和 Redis 这样的数据库，数据需要持久化。CoreOS 和 Docker 与不同的存储技术集成得很好，并且在这个领域有很多积极的工作正在进行中。

本章将涵盖 CoreOS 存储的以下三个方面：

+   CoreOS 基础文件系统和分区表

+   容器文件系统，由 Union 文件系统和写时复制（CoW）存储驱动组成

+   用于共享数据持久化的容器数据卷，可以是本地的、分布式的或共享的外部存储

本章将涵盖以下主题：

+   CoreOS 文件系统及将 AWS EBS 和 NFS 存储挂载到 CoreOS 文件系统

+   用于存储容器镜像的 Docker 容器文件系统，包含存储驱动程序和 Union 文件系统。

+   Docker 数据卷

+   使用 Flocker、GlusterFS 和 Ceph 实现容器数据持久化

存储概念

以下是我们在本章及以后将使用的一些存储术语及其基本定义：

+   本地存储：这是附加到本地主机的存储。一个例子是带有 ZFS 的本地硬盘。

+   网络存储：这是通过网络访问的常见存储。可以是 SAN 或像 Ceph 和 GlusterFS 这样的集群存储。

+   云存储：由云提供商提供的存储，例如 AWS EBS、OpenStack Cinder 和 Google 云存储。

+   块存储：需要低延迟，通常用于操作系统相关的文件系统。一些例子包括 AWS EBS 和 OpenStack Cinder。

+   对象存储：用于不可变存储项，其中延迟不是大问题。一些例子包括 AWS S3 和 OpenStack Swift。

+   NFS：这是一种分布式文件系统。可以在任何集群存储之上运行。

CoreOS 文件系统

我们在第三章中详细介绍了 CoreOS 的分区表，CoreOS 自动更新。以下截图展示了 AWS CoreOS 集群中的默认分区：

![](img/00390.jpg)

默认情况下，CoreOS 使用根分区作为容器文件系统。在前面的表格中，`/dev/xvda9`将用于存储容器镜像。

以下输出显示 Docker 在 AWS 上运行的 CoreOS 节点中使用 Ext4 文件系统和 Overlay 存储驱动：

![](img/00440.jpg)

为了获得额外的存储，可以在 CoreOS 中挂载外部存储。

挂载 AWS EBS 卷

Amazon Elastic Block Store (EBS) 为您提供持久的块级存储卷，可以与 AWS 云中的 Amazon EC2 实例一起使用。以下示例展示了如何向运行在 AWS 中的 CoreOS 节点添加一个额外的 EBS 卷，并将其用于容器文件系统。

将以下 `cloud-config` 文件重命名为 `cloud-config-mntdocker.yml`：

`#cloud-config coreos:   etcd2:     name: etcdserver     initial-cluster: etcdserver=http://$private_ipv4:2380     advertise-client-urls: http://$private_ipv4:2379,http://$private_ipv4:4001     initial-advertise-peer-urls: http://$private_ipv4:2380     listen-client-urls: http://0.0.0.0:2379,http://0.0.0.0:4001     listen-peer-urls: http://$private_ipv4:2380,http://$private_ipv4:7001   units:     - name: etcd2.service       command: start     - name: fleet.service       command: start     - name: format-ephemeral.service       command: start       content: |         [Unit]         Description=格式化临时驱动器         After=dev-xvdf.device         Requires=dev-xvdf.device         [Service]         Type=oneshot         RemainAfterExit=yes         ExecStart=/usr/sbin/wipefs -f /dev/xvdf         ExecStart=/usr/sbin/mkfs.btrfs -f /dev/xvdf     - name: var-lib-docker.mount       command: start       content: |         [Unit]         Description=将临时存储挂载到/var/lib/docker         Requires=format-ephemeral.service         After=format-ephemeral.service         Before=docker.service         [Mount]         What=/dev/xvdf         Where=/var/lib/docker         Type=btrfs`

以下是有关前面提到的 `cloud-config` 单元文件的一些详细信息：

+   Format-ephemeral 服务负责将文件系统格式化为 `btrfs`

+   Mount 服务负责在 `docker.service` 启动之前将新卷挂载到 `/var/lib/docker`

我们可以使用以下命令启动带有额外 EBS 卷的 CoreOS 节点，并使用前述 `cloud-config`：

`aws ec2 run-instances --image-id ami-85ada4b5 --count 1 --instance-type t2.micro --key-name "smakam-oregon" --security-groups "coreos-test" --user-data file://cloud-config-mntdocker.yaml --block-device-mappings "[{\"DeviceName\":\"/dev/sdf\",\"Ebs\":{\"DeleteOnTermination\":false,\"VolumeSize\":8,\"VolumeType\":\"gp2\"}}]"`

前述命令创建了一个单节点 CoreOS 集群，且附加了一个 8 GB 的额外卷。该新卷以 `btrfs` 文件系统挂载为 `/var/lib/docker`。`/dev/sdf` 目录被挂载到 CoreOS 系统中的 `/dev/xvdf`，因此挂载文件使用的是 `/dev/xvdf`。

以下是节点中带有前述 `cloud-config` 配置的分区表：

![](img/00461.jpg)

如我们所见，`/var/lib/docker` 被挂载到一个新的 8 GB 分区上。

以下输出展示了 docker 文件系统正在使用我们请求的 `btrfs` 存储驱动：

![](img/00399.jpg)

挂载 NFS 存储

我们可以使用 NFS 在 CoreOS 节点上挂载一个卷。NFS 提供了一种共享存储机制，集群中的所有 CoreOS 节点都可以看到相同的数据。这种方法可以用于容器数据持久化，当容器在节点之间移动时。在以下示例中，我们在 Linux 服务器上运行 NFS 服务器，并在 Vagrant 环境中运行的 CoreOS 节点上挂载该卷。

以下是设置 CoreOS 节点上 NFS 挂载的步骤：

1.  启动 NFS 服务器并导出要共享的目录。

1.  设置 CoreOS 的`cloud-config`以启动`rpc-statd.service`。挂载服务还需要在`cloud-config`中启动，以便将必要的 NFS 目录挂载到本地目录。

设置 NFS 服务器

启动 NFS 服务器。我已将我的 Ubuntu 14.04 机器设置为 NFS 服务器。以下是我为设置 NFS 服务器所执行的步骤：

1.  安装 NFS 服务器：

    `sudo apt-get install nfs-kernel-server`

1.  创建具有适当所有者的 NFS 目录：

    `sudo mkdir /var/nfs``sudo chown core /var/nfs （我已经创建了一个 core 用户）`

1.  将`NFS`目录导出到必要的节点。在我的例子中，`172.17.8.[101-103]`是 CoreOS 集群的 IP 地址。使用以下命令创建`/etc/exports`：

    `/var/nfs    172.17.8.101(rw,sync,no_root_squash,no_subtree_check)``/var/nfs    172.17.8.102(rw,sync,no_root_squash,no_subtree_check)``/var/nfs    172.17.8.103(rw,sync,no_root_squash,no_subtree_check)`

1.  启动 NFS 服务器：

    `sudo exportfs -a``sudo service nfs-kernel-server start`

    注意

    注意：NFS 对用户 ID（UID）和组 ID（GID）的检查非常敏感，除非正确设置，否则客户端机器无法进行写入访问。客户端用户的 UID 和 GID 必须与服务器中目录的 UID 和 GID 匹配。另一个选择是设置`no_root_squash`选项（如前面示例所示），以便客户端的 root 用户可以像服务器中的 UserID 一样进行修改。

如下命令所示，我们可以看到在完成必要的配置后导出的目录：

![](img/00005.jpg)

将 CoreOS 节点设置为 NFS 客户端

以下`cloud-config`可以用于在 CoreOS 集群的所有节点中挂载远程的`/var/nfs`到`/mnt/data`：

`#cloud-config  write-files:   - path: /etc/conf.d/nfs     permissions: '0644'     content: |       OPTS_RPC_MOUNTD=""  coreos:   etcd2:     discovery: <yourtoken>     advertise-client-urls: http://$public_ipv4:2379     initial-advertise-peer-urls: http://$private_ipv4:2380     listen-client-urls: http://0.0.0.0:2379,http://0.0.0.0:4001     listen-peer-urls: http://$private_ipv4:2380,http://$private_ipv4:7001   fleet:     public-ip: $public_ipv4   flannel:     interface: $public_ipv4   units:     - name: etcd2.service       command: start     - name: fleet.service       command: start     - name: rpc-statd.service       command: start       enable: true     - name: mnt-data.mount       command: start       content: |         [Mount]         What=172.17.8.110:/var/nfs         Where=/mnt/data         Type=nfs         Options=vers=3,sec=sys,noauto`

在前面的配置中，`cloud-config`中，`rpc-statd.service` 是 NFS 客户端服务所必需的，`mnt-data.mount` 是将 NFS 卷挂载到`/mnt/data`本地目录所必需的。

以下输出来自已经执行 NFS 挂载的 CoreOS 节点之一。正如我们所看到的，NFS 挂载成功：

![](img/00023.jpg)

在此步骤之后，集群中的任何 CoreOS 节点都可以从`/mnt/data`进行读写。

容器文件系统

容器使用 CoW 文件系统存储容器镜像。以下是 CoW 文件系统的一些特性：

+   多个用户/进程可以共享相同的数据，就像它们拥有自己的数据副本一样。

+   如果数据由某个进程或用户更改，该进程/用户将仅在此时创建数据的新副本。

+   多个运行中的容器共享同一组文件，直到对文件进行更改。这使得容器启动非常快速。

这些特性使得容器文件系统操作非常快速。Docker 支持多种能够进行 CoW 的存储驱动程序。每个操作系统选择一个默认的存储驱动程序。Docker 提供了一个选项，可以更改存储驱动程序。要更改存储驱动程序，我们需要在`/etc/default/docker`中指定存储驱动程序，并重新启动 Docker 守护进程：

`DOCKER_OPTS="--storage-driver=<driver>"`

主要支持的存储驱动程序包括`aufs`、`devicemapper`、`btrfs`和`overlay`。在更改存储驱动程序之前，我们需要确保存储驱动程序被 Docker 所安装的操作系统支持。

存储驱动程序

存储驱动程序负责管理文件系统。以下表格列出了 Docker 支持的主要存储驱动程序之间的差异：

| 属性 | AUFS | 设备映射器 | BTRTS | OverlayFS | ZFS |
| --- | --- | --- | --- | --- | --- |
| 文件/块 | 基于文件 | 基于块 | 基于文件 | 基于文件 | 基于文件 |
| Linux 内核支持 | 不在主内核中 | 主内核中存在 | 主内核中存在 | 主内核中存在 > 3.18 | 不在主内核中 |
| 操作系统 | Ubuntu 默认 | Red Hat |   | Red Hat | Solaris |
| 性能 | 不适合写入大文件；适用于 PaaS 场景 | 首次写入较慢 | 更新大量小文件可能导致低性能 | 比 AUFS 更好 | 占用大量内存 |

存储驱动程序需要根据工作负载类型、主 Linux 内核的可用性需求以及对特定存储驱动程序的熟悉程度来选择。

以下输出显示了在 Ubuntu 系统上运行的 Docker 使用的默认 AUFS 存储驱动程序：

![](img/00408.jpg)

以下输出显示了 Docker 在 CoreOS 节点上使用 Overlay 驱动程序。CoreOS 曾经使用 `btrfs`，但由于 `btrfs` 的稳定性问题，它们最近转向了 Overlay 驱动程序。

![](img/00046.jpg)

`/var/lib/docker` 目录是容器元数据和卷数据存储的位置。以下重要信息存储在这里：

+   容器：容器元数据

+   卷：主机卷

+   存储驱动程序，如 aufs 和 device mapper：这些将包含差异和层

以下截图显示了在运行 Docker 的 Ubuntu 系统中的目录输出：

![](img/00062.jpg)

Docker 和 Union 文件系统

Docker 镜像利用 Union 文件系统创建一个由多个层组成的镜像。Union 文件系统使用了写时复制（CoW）技术。每一层就像是镜像的快照，记录了特定的变化。以下示例展示了一个 Ubuntu Docker 镜像的镜像层：

![](img/00417.jpg)

每一层显示了在基础层上执行的操作，以获得这层新内容。

为了说明层级结构，我们以这个基础的 Ubuntu 镜像为例，并使用以下 Dockerfile 创建一个新的容器镜像：

`FROM ubuntu:14.04` `MAINTAINER Sreenivas Makam <smxxxx@yahoo.com>` `# 安装 apache2` `RUN apt-get install -y apache2` `EXPOSE 80` `ENTRYPOINT ["/usr/sbin/apache2ctl"]` `CMD ["-D", "FOREGROUND"]`

构建一个新的 Docker 镜像：

`docker build -t="smakam/apachetest"`

让我们来看一下这个新截图的层：

![](img/00419.jpg)

前四个层是由 Dockerfile 创建的，而最后四个层是 Ubuntu 14.04 基础镜像的一部分。如果系统中已经有了 Ubuntu 14.04 镜像并尝试下载 `smakam/apachetest`，那么只会下载前四个层，因为其他层已经存在于主机中并且可以重用。这个层重用机制使得从 Docker Hub 下载 Docker 镜像更快，并且能够有效地存储 Docker 镜像在容器文件系统中。

容器数据

容器数据不是容器文件系统的一部分，而是存储在容器运行的主机文件系统中。容器数据可以用来存储需要频繁操作的数据，例如数据库。容器数据通常需要在多个容器之间共享。

Docker 卷

在容器中所做的更改会作为联合文件系统的一部分存储。如果我们希望将一些数据保存在容器范围之外，可以使用卷。卷存储在主机文件系统中，并在容器中挂载。当容器的更改被提交时，卷不会被提交，因为它们位于容器文件系统之外。卷可以用来与主机文件系统共享源代码、保持数据库等持久化数据、在容器之间共享数据，并充当容器的草稿板。对于需要频繁读写操作的应用（如数据库），卷比联合文件系统提供更好的性能。使用卷并不能保证容器数据的持久性。使用仅数据容器是一种保持持久性并在容器之间共享数据的方法。还有其他方法，比如使用共享和分布式存储来跨主机持久化容器数据。

容器卷

以下示例启动带有`/data`卷的 Redis 容器：

`docker run -d --name redis -v /data redis`

如果我们运行 Docker 以检查 Redis 容器，我们可以获得该容器挂载的卷的详细信息，如下截图所示：

![](img/00122.jpg)

`Source`目录是主机机器上的目录，`Destination`是容器中的目录。

带有主机挂载目录的卷

以下是通过挂载主机目录使用卷进行代码共享的示例：

`docker run -d --name nginxpersist -v /home/core/local:/usr/share/nginx/html -p ${COREOS_PUBLIC_IPV4}:8080:80 nginx`

如果我们执行`docker inspect nginxpersist`，我们可以看到主机目录和容器挂载目录：

![](img/00123.jpg)

在主机机器中，可以在`/home/core/local`位置进行代码开发，主机中的任何代码更改会自动反映在容器中。

由于主机目录在不同主机上可能不同，这使得容器无法移植，并且 Dockerfile 不支持主机挂载选项。

仅数据容器

Docker 支持功能强大的仅数据容器。多个容器可以继承来自仅数据容器的卷。与常规基于主机的卷挂载相比，使用仅数据容器的优势在于我们无需担心主机文件权限。另一个优势是仅数据容器可以在主机之间移动，并且一些最近的 Docker 卷插件能够在容器跨主机移动时处理卷数据的迁移。

以下示例展示了容器死亡并重启时，如何持久化卷。

我们创建一个名为`redisvolume`的卷容器，用于`redis`，并在`redis1`容器中使用这个卷。`hellocounter`容器统计网页点击次数，并使用`redis`容器来保持计数的持久性：

`docker run -d --name redisvolume -v /data redis``docker run -d --name redis1 --volumes-from redisvolume redis``docker run -d --name hello1 --link redis1:redis -p 5000:5000 smakam/hellocounter python app.py`

让我们查看正在运行的容器：

![](img/00126.jpg)

让我们通过 curl 多次访问 hellocounter 容器，如下图所示：

![](img/00129.jpg)

现在，让我们停止这个容器并使用以下命令重启另一个容器。新的 redis 容器 `redis2` 仍然使用相同的 `redisvolume` 容器：

`docker stop redis1 hello1``docker rm redis1 hello1``docker run -d --name redis2 --volumes-from redisvolume redis``docker run -d --name hello2 --link redis2:redis -p 5001:5000 smakam/hellocounter python app.py`

如果我们尝试通过端口 `5001` 访问 hellocounter 容器，我们会看到计数器从 `6` 开始，因为即使我们已经停止了该容器并重新启动了一个新的 redis 容器，之前的值 `5` 仍然保存在数据库中：

![](img/00380.jpg)

数据容器也可以用于在容器之间共享数据。一个示例用例是，网页容器写入日志文件，而日志处理容器处理该日志文件并将其导出到中央服务器。网页和日志容器都可以挂载同一个卷，一个容器写入该卷，另一个容器从该卷读取数据。

要备份我们创建的 `redisvolume` 容器数据，可以使用以下命令：

`docker run --volumes-from redisvolume -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar /data`

这将从 `redisvolume` 中获取 `/data`，并使用 Ubuntu 容器将内容备份到当前主机目录中的 `backup.tar`。

删除卷

作为删除容器的一部分，如果我们使用 `docker rm –v` 选项，卷将被自动删除。如果我们忘记使用 `-v` 选项，卷将会悬挂。这有一个缺点，即为卷分配的主机空间将被浪费且无法删除。

Docker 1.7 之前版本还没有原生的解决方案来处理悬挂的卷。有一些实验性的容器可用来清理悬挂的卷。我使用这个测试容器 `martin/docker-cleanup-volumes` 来清理悬挂的卷。首先，我们可以使用 `dry-run` 选项来确定悬挂的卷。以下是一个示例，显示了四个悬挂的卷和一个正在使用的卷：

![](img/00131.jpg)

如果我们去掉 `dry-run` 选项，悬挂的卷将被删除，如下图所示：

![](img/00134.jpg)

Docker 卷插件

类似于 Docker 的网络插件，卷插件扩展了 Docker 容器的存储功能。卷插件提供了高级存储功能，例如跨节点的卷持久化。下图展示了卷插件架构，其中卷驱动程序暴露了一组标准的 API，插件可以实现这些 API。GlusterFS、Flocker、Ceph 和其他几家公司提供 Docker 卷插件。与 Docker 网络插件不同，Docker 没有原生的卷插件，而是依赖外部供应商的插件：

![](img/00137.jpg)

Flocker

Docker 数据卷绑定到创建容器的单一节点。当容器在节点之间迁移时，数据卷不会随之迁移。Flocker 解决了将数据卷与容器一起迁移的问题。下图展示了 Flocker 架构中的所有重要组件：

![](img/00139.jpg)

以下是 Flocker 实现的一些内部细节：

+   Flocker 代理在每个节点上运行，负责与 Docker 守护进程和 Flocker 控制服务进行通信。

+   Flocker 控制服务负责管理卷以及 Flocker 集群。

+   当前支持的后端存储包括 Amazon AWS EBS、Rackspace 块存储和 EMC ScaleIO。使用 ZFS 的本地存储目前处于实验性阶段。

+   REST API 和 Flocker CLI 都用于管理卷和 Docker 容器。

+   Docker 可以使用 Flocker 作为数据卷插件来管理卷。

+   Flocker 插件将负责管理数据卷，包括在容器跨主机迁移时迁移与容器关联的卷。

+   Flocker 将使用容器网络技术在主机之间进行通信——这可以是原生 Docker 网络，或者是 Docker 网络插件，如 Weave。

在接下来的三个示例中，我们将演示 Flocker 如何在不同环境下实现容器数据持久化。

使用 AWS EBS 作为后端的 Flocker 卷迁移

本示例将演示使用 AWS EBS 作为存储后端的数据持久化。在本示例中，我们将在 AWS 云中创建三个 Linux 节点。一个节点将作为 Flocker 主节点，运行控制服务，另外两个节点将运行 Flocker 代理，运行容器并挂载 EBS 存储。通过这些节点，我们将创建一个有状态容器，并演示容器迁移时的容器数据持久化。该示例将使用一个 hellocounter 容器，后端为 redis 容器，并演示当 redis 计数器跨主机迁移时的数据持久化。下图展示了主节点和代理节点如何与 EBS 后端关联：

![](img/00141.jpg)

我按照 Flocker 网页上提到的过程操作，参考了[`docs.clusterhq.com/en/1.4.0/labs/installer.html`](https://docs.clusterhq.com/en/1.4.0/labs/installer.html)。

以下是使用 AWS EBS 设置 Flocker 卷迁移的步骤总结：

1.  创建运行 Docker 容器和 Flocker 服务的虚拟机需要拥有 AWS 账户。

1.  对于执行前端 Flocker 命令，我们需要一台 Linux 主机。在我的例子中，是一台 Ubuntu 14.04 虚拟机。

1.  使用 Flocker 脚本在 Linux 主机上安装 Flocker 前端工具。

1.  使用 Flocker 脚本在控制节点上安装 Flocker 控制服务，并在从节点上安装 Flocker 代理。

1.  此时，我们可以在从节点上创建带有数据卷的容器，并迁移容器，同时保持数据卷持久化。

以下是安装 Flocker 前端工具和 Flocker 控制服务与代理后的相关输出。

这是 Flocker 前端工具的版本：

![](img/00144.jpg)

Flocker 节点列表显示了将运行 Flocker 代理的两个 AWS 节点：

![](img/00147.jpg)

以下输出显示了 Flocker 卷列表。最初没有任何卷：

![](img/00148.jpg)

让我们查看主节点中的主要进程。以下截图展示了控制服务正在运行：

![](img/00150.jpg)

让我们查看从节点中的主要进程。在这里，我们可以看到 Flocker 代理和 Flocker Docker 插件正在运行：

![](img/00153.jpg)

让我们在特定的从节点上创建一个 hellocounter 容器，后端是 redis 容器，更新数据库中的计数器，然后移动容器，演示数据卷在容器移动时如何保持持久化。

让我们首先设置一些快捷方式：

`NODE1="52.10.201.177" (此公共 IP 地址对应 flocker list-nodes 输出中显示的私有地址)``NODE2="52.25.14.152"``KEY="keylocation"`

让我们在 `node1` 上启动 `hellocontainer` 和 `redis` 容器：

`ssh -i $KEY root@$NODE1 docker run -d -v demo:/data --volume-driver=flocker --name=redis redis:latest``ssh -i $KEY root@$NODE1 docker run -d -e USE_REDIS_HOST=redis --link redis:redis -p 80:5000 --name=hellocounter smakam/hellocounter`

现在，让我们查看此时创建并附加的卷。100 GB 的 EBS 卷已附加到从节点 `node1`：

![](img/00156.jpg)

从以下输出中，我们可以看到两个容器正在 `node1` 上运行：

![](img/00158.jpg)

现在，我们来创建一些数据库条目。当前计数器的值为 `6`，如下图所示：

![](img/00160.jpg)

现在，让我们移除 `NODE1` 上的容器，并在 `NODE2` 上创建 `hellocounter` 和 `redis` 容器：

`ssh -i $KEY root@$NODE1 docker stop hellocounter``ssh -i $KEY root@$NODE1 docker stop redis``ssh -i $KEY root@$NODE1 docker rm -f hellocounter``ssh -i $KEY root@$NODE1 docker rm -f redis``ssh -i $KEY root@$NODE2 docker run -d -v demo:/data --volume-driver=flocker --name=redis redis:latest``ssh -i $KEY root@$NODE2 docker run -d -e USE_REDIS_HOST=redis --link redis:redis -p 80:5000 --name=hellocounter smakam/hellocounter`

正如我们所看到的，卷已迁移到第二个从节点：

![](img/00163.jpg)

让我们查看 `node2` 中的容器：

![](img/00166.jpg)

现在，让我们检查数据是否持久化：

![](img/00441.jpg)

从前面的输出可以看到，计数器值从先前的`6`开始，并递增到`7`，这表明当 redis 容器跨节点移动时，redis 数据库是持久化的。

使用 ZFS 后端的 Flocker 卷迁移

本示例将展示使用 ZFS 作为存储后端和 Vagrant Ubuntu 集群的数据显示持久化。ZFS 是一个开源文件系统，重点关注数据完整性、复制和性能。我按照[`docs.clusterhq.com/en/1.4.0/using/tutorial/vagrant-setup.html`](https://docs.clusterhq.com/en/1.4.0/using/tutorial/vagrant-setup.html)的流程，设置了一个两节点的 Vagrant Ubuntu Flocker 集群，并参照[`docs.clusterhq.com/en/1.4.0/using/tutorial/volumes.html`](https://docs.clusterhq.com/en/1.4.0/using/tutorial/volumes.html)尝试了一个示例应用程序，该应用程序支持容器迁移及相关的卷迁移。示例应用程序使用 MongoDB 容器进行数据存储，并演示了数据持久化。

以下是使用 ZFS 后端设置 Flocker 卷迁移的步骤总结：

1.  在客户端机器中安装 Flocker 客户端工具和`mongodb`客户端。我的情况下，这是一个 Ubuntu 14.04 虚拟机。

1.  创建一个两节点 Vagrant Ubuntu 集群。在集群设置过程中，Flocker 服务将在每个节点上启动，包括控制服务和代理服务。

1.  启动 flocker-deploy 脚本，启动`mongodb`容器在`node1`上运行。

1.  启动`mongodb`客户端并在`node1`中写入一些条目。

1.  启动 flocker-deploy 脚本，将`mongodb`容器从`node1`迁移到`node2`。

1.  启动`mongodb`客户端到`node2`，检查数据是否被保留。

启动两节点 Vagrant 集群后，让我们检查相关的 Flocker 服务。

`Node1`上运行着 Flocker 控制服务和代理服务，如下图所示：

![](img/00170.jpg)

`Node2`仅运行 Flocker 代理服务，并由`Node1`管理，如下图所示：

![](img/00173.jpg)

让我们查看 Flocker 节点列表；这显示了两个节点：

![](img/00176.jpg)

按照以下步骤在`node1`上部署`mongodb`容器：

![](img/00221.jpg)

让我们查看卷列表。如我们所见，卷附加到`node1`：

![](img/00179.jpg)

以下输出显示了`node1`中的容器：

![](img/00182.jpg)

让我们往`mongodb`中添加一些数据：

![](img/00185.jpg)

现在，让我们将容器重新部署到`node2`：

![](img/00127.jpg)

让我们查看卷的输出。如我们所见，卷已移动到`node2`：

![](img/00135.jpg)

从以下输出可以看到，`mongodb`内容，`数据`，得到了保留：

![](img/00140.jpg)

使用 AWS EBS 后端的 CoreOS 上的 Flocker

Flocker 最近已与 CoreOS 进行实验性集成，使用 AWS EBS 后端存储。我按照 https://github.com/clusterhq/flocker-coreos 和[`clusterhq.com/2015/09/01/flocker-runs-on-coreos/`](https://clusterhq.com/2015/09/01/flocker-runs-on-coreos/)上的程序进行此示例。过程中，我遇到了一些问题，无法使 Flocker 工具的版本 1.4.0 与 CoreOS 节点配合使用。工具的 1.3.0 版本（[`docs.clusterhq.com/en/1.3.0/labs/installer.html`](https://docs.clusterhq.com/en/1.3.0/labs/installer.html)）运行正常。

在这个例子中，我们演示了如何使用 Flocker 插件在 CoreOS 集群中通过 Docker 实现容器数据持久化。

以下是设置 AWS 上运行的 CoreOS 集群中的 Flocker 卷迁移的步骤总结：

1.  使用 Flocker 指定的模板和新创建的发现令牌，通过 AWS Cloudformation 创建 CoreOS 集群。

1.  创建包含节点 IP 和访问详情的`cluster.yml`文件。

1.  启动 Flocker 脚本来配置 CoreOS 节点，包含 Flocker 控制服务和 Flocker 代理。Flocker 脚本还会处理将 CoreOS 节点中默认的 Docker 二进制文件替换为支持 Volume 插件的 Docker 二进制文件。

1.  检查容器迁移是否能正常工作并保持数据持久化。

我使用了以下 Cloudformation 脚本，通过 Flocker 的模板文件创建 CoreOS 集群：

`aws cloudformation create-stack --stack-name coreos-test1 --template-body file://coreos-stable-flocker-hvm.template --capabilities CAPABILITY_IAM --tags Key=Name,Value=CoreOS --parameters ParameterKey=DiscoveryURL,ParameterValue="your token" ParameterKey=KeyPair,ParameterValue="your keypair"`

以下是包含三台节点的 CoreOS 集群的详细信息：

![](img/00145.jpg)

以下是已安装的旧版本和新版本 Docker。Docker 版本 1.8.3 支持 Volume 插件：

![](img/00149.jpg)

以下是节点中的 CoreOS 版本：

![](img/00154.jpg)

以下输出显示了 Flocker 节点列表，列出了三台运行 Flocker 的 CoreOS 节点：

![](img/00159.jpg)

我尝试了与上一部分中提到的相同的`hellocounter`示例，卷会在节点之间自动移动。以下输出显示了最初附加到`node1`的卷，后来作为容器迁移的一部分，移动到`node2`。

这是附加到`node1`的卷：

![](img/00164.jpg)

这是附加到`node2`的卷：

![](img/00169.jpg)

根据 Flocker 文档，他们计划在某个时刻支持 CoreOS 上的 ZFS 后端，以允许我们使用本地存储而不是 AWS EBS。目前还不确定 CoreOS 是否会原生支持 ZFS。

Flocker 最近的新增功能

Flocker 在 2015 年 11 月新增了以下功能：

+   Flocker 卷中心（[`clusterhq.com/volumehub/`](https://clusterhq.com/volumehub/)）管理来自中心位置的所有 Flocker 卷。

+   Flocker dvol ([`clusterhq.com/dvol/`](https://clusterhq.com/dvol/)) 为你提供类似 Git 的数据卷功能。这可以帮助管理数据库，如代码库。

GlusterFS

GlusterFS 是一个分布式文件系统，存储分布在多个节点上，并作为一个单一的单元呈现。GlusterFS 是一个开源项目，可以在任何类型的存储硬件上运行。Red Hat 已收购 Gluster，而 Gluster 是 GlusterFS 的发源地。以下是 GlusterFS 的一些特性：

+   多台服务器及其关联的存储通过对等关系（peering）加入 GlusterFS 集群。

+   GlusterFS 可以在普通存储以及 SAN 上运行。

+   通过避免使用中央元数据服务器并采用分布式哈希算法，GlusterFS 集群具备可扩展性，能够扩展为非常大的集群。

+   从 GlusterFS 的角度来看，砖块（Bricks）是存储的最小组件。一个砖块由带有基础文件系统的存储磁盘上的挂载点组成。砖块绑定到单个服务器上。一个服务器可以拥有多个砖块。

+   卷（Volumes）由多个砖块组成。卷作为挂载目录挂载到客户端设备上。

+   主要的卷类型有分布式（distributed）、复制（replicated）和条带化（striped）。分布式卷类型允许将文件分布到多个砖块上。复制卷类型允许文件有多个副本，从冗余角度来看非常有用。条带化卷类型允许将一个大文件分割成多个较小的文件并将其分布到各个砖块上。

+   GlusterFS 支持多种访问方法来访问 GlusterFS 卷，包括基于 FUSE 的本地访问、SMB、NFS 和 REST。

下图展示了 GlusterFS 的不同层次结构：

![](img/00174.jpg)

设置 GlusterFS 集群

在以下示例中，我已经设置了一个由两台节点组成的 GlusterFS 3.5 集群，每台服务器都运行 Ubuntu 14.04 虚拟机。我还将 GlusterFS 服务器节点作为 GlusterFS 客户端使用。

以下是设置 GlusterFS 集群的步骤概览：

1.  在两台节点上安装 GlusterFS 服务器，并在集群中的一个节点上安装客户端软件。

1.  GlusterFS 节点必须能够相互通信。我们可以设置 DNS 或者使用静态的 `/etc/hosts` 方法让节点之间进行通信。

1.  如有需要，关闭防火墙，以便服务器能够相互通信。

1.  设置 GlusterFS 服务器对等。

1.  创建砖块。

1.  在创建的砖块基础上创建卷。

1.  在客户端机器上，将卷挂载到挂载点，并开始使用 GlusterFS。

以下命令需要在每台服务器上执行。这将安装 GlusterFS 服务器组件。此操作需要在两台节点上执行：

`sudo apt-get install software-properties-common``sudo add-apt-repository ppa:gluster/glusterfs-3.5``sudo apt-get update``sudo apt-get install glusterfs-server`

以下命令将安装 GlusterFS 客户端。仅在 `node1` 中需要此操作：

`sudo apt-get install glusterfs-client`

设置 `/etc/hosts` 以允许节点相互通信：

`192.168.56.102  gluster1``192.168.56.101  gluster2`

禁用防火墙：

`sudo iptables -I INPUT -p all -s 192.168.56.102 -j ACCEPT``sudo iptables -I INPUT -p all -s 192.168.56.101 -j ACCEPT`

创建复制卷并启动它：

`sudo gluster volume create volume1 replica 2 transport tcp gluster1:/gluster-storage gluster2:/gluster-storage force (/gluster-storage 是每个节点中的砖块)``sudo gluster volume start volume1`

在每个节点中设置服务器探针。以下命令适用于 `Node1`：

`sudo gluster peer probe gluster2`

以下命令适用于 `Node2`：

`sudo gluster peer probe gluster1`

对 GlusterFS 卷进行客户端挂载。这是在 `Node1` 中需要的操作：

`sudo mkdir /storage-pool``sudo mount -t glusterfs gluster2:/volume1 /storage-pool`

现在，让我们查看 `Node1` 中的 GlusterFS 集群状态和创建的卷：

![](img/00178.jpg)

现在，让我们看一下 `Node2`：

![](img/00183.jpg)

看一下卷的详细信息。正如我们所见，`volume1` 被设置为复制卷类型，且在 `gluster1` 和 `gluster2` 上有两个砖块：

![](img/00074.jpg)

以下输出显示了 `df -k` 输出中的客户端挂载点：

![](img/00078.jpg)

此时，我们可以从客户端挂载点 `/storage-pool` 读写内容。

为 CoreOS 集群设置 GlusterFS

通过设置 CoreOS 节点使用 GlusterFS 文件系统，容器卷可以使用 GlusterFS 存储卷相关数据。这使得容器能够在节点之间迁移并保持卷的持久性。此时，CoreOS 不支持本地 GlusterFS 客户端。使用 GlusterFS 的一种方法是通过 NFS 导出 GlusterFS 卷，并从 CoreOS 节点进行 NFS 挂载。

继续使用上一节中创建的 GlusterFS 集群，我们可以按照以下步骤在 GlusterFS 集群中启用 NFS：

`sudo gluster volume set volume1 nfs.disable off`

用于挂载 NFS 存储部分的 CoreOS `cloud-config` 也可以在这里使用。以下是挂载特定部分，我们在 CoreOS 节点的 `/mnt/data` 中挂载了 GlusterFS 卷 `172.17.8.111:/volume1`：

`    - name: mnt-data.mount       command: start       content: |         [Mount]         What=172.17.8.111:/volume1         Where=/mnt/data         Type=nfs         Options=vers=3,sec=sys,noauto`

我在 GlusterFS 卷 `/volume1` 中创建了一些文件，并能够从 CoreOS 节点进行读写。以下输出显示了 CoreOS 节点中的 `/mnt/data` 内容：

![](img/00082.jpg)

使用 Docker 卷插件访问 GlusterFS

使用 GlusterFS 卷插件（[`github.com/calavera/docker-volume-glusterfs`](https://github.com/calavera/docker-volume-glusterfs)）为 Docker，我们可以使用常规的 Docker 卷 CLI 创建和管理卷。

在以下示例中，我们将安装 GlusterFS Docker 卷插件，并创建一个持久化的`hellocounter`应用程序。我使用的是运行 GlusterFS 卷的相同 Ubuntu 14.04 虚拟机来运行 Docker。

以下是设置 Docker 卷插件所需的步骤：

+   Docker 实验性版本支持 GlusterFS 卷插件，因此需要下载实验性 Docker 版本。

+   GlusterFS Docker 卷插件需要下载并启动。需要安装 GO（[`golang.org/doc/install`](https://golang.org/doc/install)）以获取卷插件。

+   使用 Docker 和 GlusterFS Docker 卷插件。为此，必须停止并重新启动 Docker 服务。

以下是在两个节点上运行的 Docker 实验性发布版本：

![](img/00087.jpg)

下载并启动 GlusterFS 卷插件：

`go get github.com/calavera/docker-volume-glusterfs``sudo docker-volume-glusterfs -servers gluster1:gluster2 &`

使用 GlusterFS 卷驱动启动`redis`容器，如下所示：

`docker run -d -v volume1:/data --volume-driver=glusterfs --name=redis redis:latest`

启动`hellocounter`容器并将其链接到`redis`容器：

`docker run -d -e USE_REDIS_HOST=redis --link redis:redis -p 80:5000 --name=hellocounter smakam/hellocounter`

通过访问计数器几次来更新它，如下图所示：

![](img/00089.jpg)

现在，停止`node1`中的容器，并在`node2`中启动它们。我们来看看`node2`中正在运行的容器：

![](img/00092.jpg)

如果我们现在访问`hellocounter`容器，我们可以看到计数器从`3`开始，因为先前的计数值已经被持久化：

![](img/00094.jpg)

Ceph

Ceph 为你提供类似于 GlusterFS 的分布式存储，并且是一个开源项目。Ceph 最初由 Inktank 开发，后来被 Red Hat 收购。以下是 Ceph 的一些特性：

+   Ceph 使用可靠的自适应分布式对象存储（RADOS）作为存储机制。其他存储访问机制如文件和块存储是基于 RADOS 实现的。

+   Ceph 和 GlusterFS 似乎具有类似的特性。根据 Red Hat 的说法，Ceph 更适合 OpenStack 集成，而 GlusterFS 则用于大数据分析，二者会有一些重叠。

+   Ceph 有两个关键组件。它们分别是 Monitor 和 OSD。Monitor 存储集群地图，而对象存储守护进程（OSD）是形成存储集群的单个存储节点。存储客户端和 OSD 都使用 CRUSH 算法将数据分布到集群中。

与 GlusterFS 相比，设置 Ceph 似乎稍微复杂一些，并且目前有活跃的工作正在进行，以使 Ceph 组件能够作为 Docker 容器运行，并且将 Ceph 与 CoreOS 集成。此外，还有工作在进行中，涉及 Ceph Docker 卷插件。

NFS

NFS 是一种分布式文件系统，它允许客户端计算机像本地存储一样访问网络存储。我们可以通过共享的 NFS 存储实现容器数据持久化。

使用 NFS 实现容器数据持久化。

本节将介绍一个使用 NFS 进行数据持久化的 Web 应用示例。以下是该应用的一些细节：

+   `hellocounter.service`单元启动一个容器，用来跟踪应用程序的网页访问次数。

+   `Hellocounter.service`使用`redis.service`容器来跟踪访问计数。

+   Redis 容器使用 NFS 存储来保存数据。

+   当数据库容器因某种原因停止时，Fleet 会在集群中的另一个节点重启该容器，由于该服务使用 NFS 存储，计数值会被持久化。

以下图示显示了本节中使用的示例：

![](img/00097.jpg)

以下是前提条件和所需的步骤：

1.  启动 NFS 服务器和一个三节点的 CoreOS 集群，将 NFS 数据挂载到如“挂载 NFS 存储”部分所述。

1.  使用 Fleet 启动`hellocounter.service`和`redis.service`，通过 X-fleet 属性控制容器的调度。`hellocounter.service`在所有节点上启动；`redis.service`在其中一个节点上启动。

`Hellocounter@.service`的代码如下：

`[Unit] Description=hello counter with redis backend  [Service] Restart=always RestartSec=15 ExecStartPre=-/usr/bin/docker kill %p%i ExecStartPre=-/usr/bin/docker rm %p%i ExecStartPre=/usr/bin/docker pull smakam/hellocounter  ExecStart=/usr/bin/docker run --name %p%i -e SERVICE_NAME=redis -p 5000:5000 smakam/hellocounter python app.py  ExecStop=/usr/bin/docker stop %p%i  [X-Fleet] X-Conflicts=%p@*.service`

`Redis.service`的代码如下：

`[Unit] Description=app-redis  [Service] Restart=always RestartSec=5 ExecStartPre=-/usr/bin/docker kill %p ExecStartPre=-/usr/bin/docker rm %p ExecStartPre=/usr/bin/docker pull redis ExecStart=/usr/bin/docker run --name redis -v /mnt/data/hellodata:/data redis  ExecStop=/usr/bin/docker stop %p  [X-Fleet] Conflicts=redis.service`

我们启动三个`hellocounter@.service`实例和一个`redis.service`实例。下图显示了在 CoreOS 集群中运行的三个`hellocounter`服务实例和一个`redis`服务实例。

![](img/00099.jpg)

从前面的截图可以看到，`hellocounter@2.service`和`redis.service`都在同一个节点 node3 上。

让我们尝试从`node3`访问网页服务几次，来检查计数值：

![](img/00102.jpg)

当前计数器的值为`6`，并存储在 NFS 中。

现在，让我们重启`node3`。从以下输出中可以看到，只剩下两台机器：

![](img/00104.jpg)

让我们看看正在运行的服务。从以下输出可以看出，`redis.service`已从`node3`移至`node2`：

![](img/00108.jpg)

现在，让我们检查一下 `node2` 上的 Web 访问计数。正如我们从以下输出中看到的，计数从 `7` 开始，因为之前的计数在 `node3` 上设置为 `6`。这证明了容器数据已被持久化：

![](img/00110.jpg)

注意

注意：此示例不具备实际应用性，因为有多个独立运行的 Web 服务器实例。在一个更实际的示例中，负载均衡器将作为前端。此示例的目的是仅为了说明如何使用 NFS 实现容器数据持久化。

Docker 1.9 更新

Docker 1.9 添加了命名卷，这使得卷成为 Docker 中的一级公民。Docker 卷可以使用 `docker volume` 进行管理。

以下截图展示了 `docker volume` 的选项：

![](img/00114.jpg)

命名卷取代了之前用于跨容器共享卷的数据-only 容器。

以下一组命令展示了使用命名卷替代数据-only 容器的相同示例：

`docker volume create --name redisvolume` `docker run -d --name redis1 -v redisvolume:/data redis` `docker run -d --name hello1 --link redis1:redis -p 5000:5000 smakam/hellocounter python app.py`

在前面的示例中，我们创建了一个名为 `redisvolume` 的命名卷，该卷用于 `redis1` 容器。`hellocounter` 应用程序链接到 `redis1` 容器。

以下截图展示了 `redis1` 卷的信息：

![](img/00116.jpg)

使用命名卷的另一个优势是，我们不再需要担心之前存在的悬空卷问题。

摘要

在本章中，我们介绍了 CoreOS 系统中存储容器镜像和容器数据的不同存储选项。通过实际示例展示了 Flocker、GlusterFS、NFS 和 Docker 卷等技术以及它们与容器和 CoreOS 的集成。容器存储技术仍在发展中，并且需要一些时间才能成熟。业界普遍趋势是从昂贵的 SAN 技术转向本地和分布式存储。在下一章中，我们将讨论容器运行时 Docker 和 Rkt，以及它们如何与 CoreOS 集成。

参考文献

+   GlusterFS: [`gluster.readthedocs.org/`](http://gluster.readthedocs.org/)

+   GlusterFS Docker 卷插件: [`github.com/calavera/docker-volume-glusterfs`](https://github.com/calavera/docker-volume-glusterfs)

+   Flocker: [`docs.clusterhq.com`](https://docs.clusterhq.com)

+   Docker 卷插件: [`github.com/docker/docker/blob/master/docs/extend/plugins_volume.md`](https://github.com/docker/docker/blob/master/docs/extend/plugins_volume.md)

+   Docker 存储驱动: [`docs.docker.com/engine/userguide/storagedriver/imagesandcontainers/`](https://docs.docker.com/engine/userguide/storagedriver/imagesandcontainers/)

+   Docker 卷: [`docs.docker.com/userguide/dockervolumes/`](https://docs.docker.com/userguide/dockervolumes/)

+   在 CoreOS 上挂载存储: [`coreos.com/os/docs/latest/mounting-storage.html`](https://coreos.com/os/docs/latest/mounting-storage.html)

+   容器文件系统: [`jpetazzo.github.io/assets/2015-03-03-not-so-deep-dive-into-docker-storage-drivers.html#1`](http://jpetazzo.github.io/assets/2015-03-03-not-so-deep-dive-into-docker-storage-drivers.html#1)

+   Ceph: [`docs.ceph.com/`](http://docs.ceph.com/)

+   Ceph Docker: [`github.com/ceph/ceph-docker`](https://github.com/ceph/ceph-docker)

进一步阅读和教程

+   CoreOS 集群中的持久数据存储: [`gist.github.com/Luzifer/c184b6b04d83e6d6fbe1`](https://gist.github.com/Luzifer/c184b6b04d83e6d6fbe1)

+   创建一个 GlusterFS 集群: [`www.digitalocean.com/community/tutorials/how-to-create-a-redundant-storage-pool-using-glusterfs-on-ubuntu-servers`](https://www.digitalocean.com/community/tutorials/how-to-create-a-redundant-storage-pool-using-glusterfs-on-ubuntu-servers)

+   GlusterFS 概述: [`www.youtube.com/watch?v=kvr6p9gSOX0`](https://www.youtube.com/watch?v=kvr6p9gSOX0)

+   在 CoreOS 上使用 Flocker 的有状态容器: [`www.slideshare.net/ClusterHQ/stateful-containers-flocker-on-coreos-54492047`](http://www.slideshare.net/ClusterHQ/stateful-containers-flocker-on-coreos-54492047)

+   Docker 存储网络研讨会: [`blog.docker.com/2015/12/persistent-storage-docker/`](https://blog.docker.com/2015/12/persistent-storage-docker/)

+   Contiv 卷插件: [`github.com/contiv/volplugin`](https://github.com/contiv/volplugin)

+   Ceph RADOS: [`ceph.com/papers/weil-rados-pdsw07.pdf`](http://ceph.com/papers/weil-rados-pdsw07.pdf)
