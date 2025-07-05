# 第一章

前言

像 Amazon 和 Google 这样的公共云服务提供商已经彻底改变了云技术，使得终端用户更容易使用。CoreOS 的目标是创建一个安全可靠的集群操作系统，并简化使用容器开发分布式应用程序的过程。CoreOS 的理念是将内核保持在最低限度，同时保持其安全性、可靠性和更新。除了少数关键服务，内核之上的所有内容都作为容器运行。容器革命性地改变了应用开发和部署的方式。尽管容器技术已存在多年，但 Docker 使得容器技术变得更加易于使用和分发，进而引发了这一技术的革命。

通过将其核心技术的关键组件开源，CoreOS 在 CoreOS 社区以及更广泛的 Linux 和云计算社区中赢得了大量追随者。通过 Tectonic，CoreOS 将所有开源技术与 Kubernetes 合并，推出了商业版解决方案。集群操作系统、容器和分布式应用程序部署都是相对较新的领域，这些领域正不断发展。CoreOS 处于这一技术的关键位置，这将激励你进一步了解它。

本书将介绍 CoreOS 内部机制以及围绕在 CoreOS 集群中部署基于容器的微服务的相关技术。

本书首先概述了 CoreOS 和分布式应用开发及其相关技术。你将首先学习如何在不同环境中安装 CoreOS，并使用 CoreOS 自动更新服务。接下来，书中将介绍管理和互联服务的核心 CoreOS 服务，以及网络和存储方面的考虑。之后，将介绍容器生态系统，包括 Docker 和 Rkt，并讲解容器编排技术。本书还涵盖了如何将 OpenStack、AWS 容器服务和 Google 容器引擎等流行的编排解决方案与 CoreOS 和 Docker 集成。最后，本书介绍了容器、CoreOS 以及分布式应用开发和部署的故障排除和生产环境考虑因素。

本书内容概述

第一章，CoreOS 概述，为您提供微服务、分布式应用开发概念、市场上现有容器操作系统的对比、容器基础、Docker 和 CoreOS 的概述。

第二章，搭建 CoreOS 实验环境，介绍如何在 Vagrant、Amazon AWS、Google GCE 和裸机上搭建 CoreOS 开发环境。

第三章，CoreOS 自动更新，涵盖了 CoreOS 发布周期、CoreOS 自动更新以及在集群中管理 CoreOS 更新的选项。

第四章，CoreOS 主要服务 —— Etcd、Systemd 和 Fleet，讲解了 CoreOS 关键服务 Etcd、Systemd 和 Fleet 的内部机制。对于每个服务，我们将介绍其安装、配置和应用。

第五章，CoreOS 网络和 Flannel 内部，讲解了容器网络的基础知识，重点是 CoreOS 如何通过 Flannel 实现容器网络。本章还涉及 Docker 网络和其他相关的容器网络技术。

第六章，CoreOS 存储管理，介绍了 CoreOS 基础文件系统和分区表、容器文件系统和容器数据卷。详细讲解了使用 GlusterFS 和 Flocker 实现容器数据持久化。

第七章，CoreOS 与容器的集成 —— Docker 和 Rkt，重点介绍了容器标准、高级 Docker 主题以及 Rkt 容器运行时的基础知识。本章的重点是 Docker 和 Rkt 如何与 CoreOS 集成。

第八章，容器编排，深入探讨了 Kubernetes、Docker 和 Swarm 的内部机制，并对市场上现有的编排解决方案进行了比较。还涵盖了 AWS 容器服务、Google 容器引擎和 Tectonic 等商业解决方案。

第九章，OpenStack 与容器和 CoreOS 的集成，提供了 OpenStack 概述以及 OpenStack 与容器和 CoreOS 集成的内容。还将介绍 OpenStack Magnum 和 Kuryr 项目的详细信息。

第十章，CoreOS 和容器 —— 故障排除与调试，讲解了 CoreOS 工具箱、Docker 远程 API 和日志记录。还将通过实践示例介绍容器监控工具（如 Sysdig 和 Cadvisor）以及容器日志记录工具（如 Logentries）。

第十一章，CoreOS 和容器 —— 生产环境考量，讨论了 CoreOS 和容器在生产环境中的应用考量，如服务发现、部署模式、CI/CD、自动化和安全性。同时还涵盖了 CoreOS 和 Docker 的发展路线图。

本书所需内容

+   本地机器：Linux、Windows 或 Mac。

+   虚拟化软件：Vagrant 和 Virtualbox，用于在本地机器上运行虚拟机。

+   云账户：Google 云和 AWS 账户，用于在云中运行虚拟机。

+   开源软件：CoreOS、Kubernetes、Docker、Weave、Calico、Flocker、GlusterFS、OpenStack、Sysdig、cadvisor、Ansible 和 LogEntries。这些开源软件在特定章节中有使用。

本书适用对象

如果您正在寻找部署 CoreOS 集群的方法，或者您已经拥有一个 CoreOS 集群并希望管理它以提高性能、安全性和扩展性，那么这本书非常适合您。本书为开发人员提供了足够的技术输入，帮助他们使用容器部署分布式应用程序，并为管理员提供了管理分布式 CoreOS 基础设施的指南。要跟进动手操作，您需要拥有 Google 和 AWS 云账户，并且能够在您的计算机上运行 CoreOS 虚拟机。

需要对公有云和私有云、容器、Docker、Linux 和 CoreOS 有基本了解。

约定

本书中，您将看到多种文本样式，用于区分不同类型的信息。以下是这些样式的示例及其含义解释。

文本中的代码词、数据库表名、文件夹名称、文件名、文件扩展名、路径名、虚拟网址、用户输入和 Twitter 用户名如下所示：“服务类型是最常见的类型，用于定义一个带有其依赖项的`service`。”

代码块如下所示：

`ExecStart=/usr/bin/docker run --name hello busybox /bin/sh -c "while true; do echo Hello World; sleep 1; done" ExecStop=/usr/bin/docker stop hello [X-Fleet] Global=true`

任何命令行输入或输出都如下所示：

`core@core-01 /etc/systemd/system/multi-user.target.wants $ ls -la``lrwxrwxrwx 1 root root   34 Aug 12 13:25 hello1.service -> /etc/systemd/system/hello1.service`

新术语和重要词汇以粗体显示。您在屏幕上看到的词汇，例如在菜单或对话框中，会以如下方式出现在文本中：“为了实现这一点，我们需要进入 AWS 控制台中的每个实例，选择`Networking` | `change source/dest check` | `disable`。”

注意

警告或重要说明会以框的形式出现。

小贴士

小贴士和技巧如下所示。

读者反馈

我们始终欢迎读者反馈。让我们知道您对这本书的看法——您喜欢或不喜欢的部分。读者的反馈对我们来说非常重要，因为它帮助我们开发出您真正能够从中受益的书籍。

若要向我们提供一般反馈，只需通过电子邮件发送 `<``feedback@packtpub.com``>`，并在邮件主题中提及书名。

如果您在某个领域有专长，并且有兴趣写作或为书籍做贡献，请查看我们的作者指南，网址：[www.packtpub.com/authors](http://www.packtpub.com/authors)。

客户支持

现在您是 Packt 书籍的骄傲拥有者，我们为您准备了许多资源，以帮助您充分利用购买的书籍。

下载示例代码

您可以从您的账户中下载所有已购买的 Packt Publishing 书籍的示例代码文件，访问 [`www.packtpub.com`](http://www.packtpub.com)。如果您是在其他地方购买的本书，可以访问 [`www.packtpub.com/support`](http://www.packtpub.com/support)，注册后文件会直接通过电子邮件发送给您。

勘误

尽管我们已尽力确保内容的准确性，但错误仍然会发生。如果您在我们的书中发现错误——可能是文本或代码中的错误——我们将非常感激您能向我们报告。这样，您可以帮助其他读者避免困扰，并帮助我们改进该书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)进行报告，选择您的书籍，点击“勘误提交表格”链接，并输入勘误的详细信息。一旦您的勘误被验证，您的提交将被接受，并且勘误将上传到我们的网站，或者添加到该书的勘误部分的现有勘误列表中。

若要查看之前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索框中输入书籍名称。所需的信息将出现在“勘误”部分。

盗版

网络上版权材料的盗版问题是一个持续存在的难题，涉及所有媒体。在 Packt，我们非常重视版权和许可证的保护。如果您在互联网上遇到我们作品的任何非法复制品，请立即向我们提供相关位置地址或网站名称，以便我们采取相应的措施。

请通过`<``copyright@packtpub.com``>`与我们联系，并提供涉嫌盗版内容的链接。

感谢您的帮助，帮助我们保护作者的权益，并使我们能够为您提供有价值的内容。

问题

如果您对本书的任何方面有问题，请通过`<``questions@packtpub.com``>`与我们联系，我们将尽力解决问题。
