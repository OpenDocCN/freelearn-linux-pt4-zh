# 前言

随着越来越多的应用程序向云端迁移并使用服务器虚拟化，快速部署用户应用和服务并确保其可靠性，借助合适的服务器部署服务并保证 SLA，变得尤为必要。特别是当这些服务具有动态特性时，情况更加复杂，这要求这些服务能够在一组节点上自动化配置和自动扩展。用户应用的编排不仅限于在正确的服务器或虚拟机上部署服务，更要扩展到提供跨这些服务的网络连接，从而实现**基础设施即服务**（**IaaS**）。计算、网络和存储是云服务商在提供 IaaS 时必须管理的三大资源。目前，已有多种机制以更抽象的方式处理这些需求。有多个云编排框架可以管理计算、存储和网络资源。OpenStack、Cloud Stack 和 VMware vSphere 是一些能够编排这些资源池并提供 IaaS 的云平台。

虚拟机（VM）提供的服务器虚拟化需要在每个虚拟机上运行独立的操作系统实例，这会带来一定的开销。这降低了服务器上能够运行的虚拟机实例的数量，进而对运营成本产生重大影响。随着 Linux 命名空间、容器技术（如 docker 和 rkt）的流行，可以通过将应用服务部署在容器中而非虚拟机中，引入服务器虚拟化的另一层级。然而，容器或 Docker 在集群中部署服务、服务发现、服务参数管理、容器间的网络通信等方面需要一个编排和集群框架，CoreOS 就是为此目的而开发的。

CoreOS 是一个轻量级的云服务编排操作系统，基于 Google Chrome 开发。CoreOS 主要用于在一个节点集群上编排应用程序/服务。CoreOS 扩展了 Linux 提供的现有服务，使其能够在分布式集群上运行，而不仅限于单一节点。

# 本书涵盖的内容

第一章，*CoreOS，另一个 Linux 发行版？*，解释了容器、Docker 和 CoreOS 的高级架构基础。

第二章，*设置 CoreOS 环境*，将教你如何使用 Vagrant 和 VirtualBox 设置并运行 CoreOS 单机环境。它还包括如何创建和运行 docker 镜像，并熟悉重要的配置文件及其内容。

第三章, *创建 CoreOS 集群并管理集群*，教你如何使用多台机器设置 CoreOS 集群。你还将学习如何发现机器以及如何在这些机器上调度服务。此外，你将了解如何使用 Fleet 启动和停止服务。

第四章, *使用用户定义约束管理服务*，介绍了服务约束的概念，帮助在适当的成员上部署服务。

第五章, *在集群中发现正在运行的服务*，解释了在集群中发现服务的需求和机制。你还将学习到两种广泛用于服务发现的重要工具：etcdctl 和 curl。

第六章, *服务链和跨服务的网络通信*，解释了容器通信的重要性以及 CoreOS 和 Docker 提供的各种可能性来实现这种通信。

第七章, *使用 OVS 创建虚拟租户网络和服务链*，解释了 OVS 在容器通信中的重要性以及 OVS 提供的各种优势。本章详细介绍了如何通过 OVS 将不同客户/租户在 CoreOS 集群中部署的服务进行链接/连接。

第八章, *下一步是什么?*，涉及一些高级的 Docker 和 CoreOS 话题，并讨论了 CoreOS 的未来发展。

# 本书所需的内容

安装并启动一个示例 CoreOS 集群所需的软件如下。

+   Git - [`git-scm.com/download`](https://git-scm.com/download)

+   VirtualBox - [`www.virtualbox.org/wiki/Downloads`](https://www.virtualbox.org/wiki/Downloads)

+   Vagrant - [`www.vagrantup.com/downloads.html`](http://www.vagrantup.com/downloads.html)

+   VMware vSphere 客户端 - [`vsphereclient.vmware.com/vsphereclient/1/9/9/3/0/7/2/VMware-viclient-all-5.5.0-1993072.exe`](http://vsphereclient.vmware.com/vsphereclient/1/9/9/3/0/7/2/VMware-viclient-all-5.5.0-1993072.exe)

+   VMware 的 CoreOS 镜像 - [`stable.release.core-os.net/amd64-usr/current/coreos_production_vmware_ova.ova`](http://stable.release.core-os.net/amd64-usr/current/coreos_production_vmware_ova.ova)

+   Docker - [`docs.docker.com/engine/installation/`](https://docs.docker.com/engine/installation/)

# 本书适合的人群

本书适合云计算或企业管理员以及应用程序开发人员，尤其是那些希望学习 CoreOS 并在一组云服务器集群上部署云应用程序或微服务的人员。它还面向具有基本网络经验的管理员。您不需要具备 CoreOS 的任何知识。

# 约定

本书中，您将看到许多不同的文本样式，用于区分不同类型的信息。以下是一些样式及其含义的示例。

文本中的代码词汇、数据库表名、文件夹名称、文件名、文件扩展名、路径名、虚拟网址、用户输入和 Twitter 用户名以以下方式显示：“创建一个名为`coreos-vagrant`的目录，命令为`git clone`。”

一段代码如下所示：

```
[Unit]
Description=Example
After=docker.service
Requires=docker.service

[Service]
TimeoutStartSec=0
ExecStartPre=-/usr/bin/docker kill busybox1
ExecStartPre=-/usr/bin/docker rm busybox1
ExecStartPre=/usr/bin/docker pull busybox
ExecStart=/usr/bin/docker run --name busybox1 busybox /bin/sh -c "while true; do echo Hello World; sleep 1; done"
ExecStop=/usr/bin/docker stop busybox1
```

任何命令行输入或输出如下所示：

```
weave status
...
 Service: dns
 Domain: weave.local.
 Upstream: 10.0.2.3
 TTL: 1
 Entries: 2

 Service: proxy
 Address: unix:///var/run/weave/weave.sock

```

**新术语**和**重要词汇**以粗体显示。您在屏幕上看到的词语，例如菜单或对话框中的词语，以以下方式出现在文本中：“点击**Datastore ISO File**并从数据存储中选择上传的`iso`文件。”

### 注意

警告或重要提示会以如下框中的形式出现。

### 提示

提示和技巧以这种方式显示。

# 读者反馈

我们始终欢迎读者的反馈。让我们知道您对本书的看法——您喜欢什么或不喜欢什么。读者反馈对我们非常重要，因为它帮助我们开发出您真正能够从中受益的书籍。

要向我们发送一般反馈，只需发送电子邮件至`<feedback@packtpub.com>`，并在邮件主题中提及本书标题。

如果您对某个领域有专业知识并且有兴趣撰写或参与书籍的编写，请参阅我们的作者指南：[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在，您是《Packt》书籍的骄傲拥有者，我们为您提供了一些帮助您最大化利用购买的内容。

## 下载本书的彩色图像

我们还为您提供了一份 PDF 文件，其中包含本书中使用的截图/图表的彩色图像。彩色图像将帮助您更好地理解输出中的变化。您可以从[`www.packtpub.com/sites/default/files/downloads/LearningCoreOS_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/LearningCoreOS_ColorImages.pdf)下载此文件。

## 勘误表

尽管我们已经尽力确保内容的准确性，但错误仍然可能发生。如果您在我们的书籍中发现任何错误——可能是文本或代码中的错误——我们将非常感激您能报告给我们。通过这样做，您不仅能帮助其他读者避免困惑，还能帮助我们改进后续版本。如果您发现任何勘误，请访问 [`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata) 提交，选择您的书籍，点击 **勘误提交表格** 链接，输入勘误的详细信息。一旦您的勘误被验证，您的提交将被接受，勘误将被上传到我们的网站，或添加到该书籍的勘误列表中。

要查看之前提交的勘误，请访问 [`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索框中输入书名。所需的信息将在 **勘误** 部分显示。

## 盗版

互联网盗版问题在所有媒体中都存在。我们在 Packt 非常重视保护我们的版权和许可。如果您在互联网上发现任何形式的非法复制品，请立即提供地址或网站名称，以便我们采取措施。

如果您发现涉嫌盗版的材料，请通过 `<copyright@packtpub.com>` 与我们联系，并提供相关链接。

我们感谢您在保护我们的作者和我们提供有价值内容方面的帮助。

## 问题

如果您在本书的任何方面遇到问题，可以通过 `<questions@packtpub.com>` 联系我们，我们将尽力解决问题。
