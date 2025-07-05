# 序言

Apache Mesos 抽象化了 CPU、内存、存储和其他计算资源，将它们从机器（物理或虚拟）中分离出来，从而使得可以轻松构建和高效运行容错且弹性的分布式系统。它提高了资源利用率，简化了系统管理，并支持多种分布式应用程序，能够通过其可插拔架构轻松部署。

本书将提供一个详细的逐步指南，使用所有标准的 DevOps 工具部署 Mesos 集群，帮助有效地迁移 Mesos 框架，并总体上揭示 Mesos 的概念。

本书将首先建立 Mesos 的存在理由，并有效地解释其架构。随后，书中将带领读者逐步进入 Mesos 的复杂世界，从简单的单机设置逐步过渡到高度复杂的多节点集群设置，并沿途引入新的概念。旅程的最后，读者将掌握管理当今现代数据中心需求复杂性的所有资源。

# 本书内容

第一章，*介绍 Mesos*，介绍了 Mesos，深入探讨了其架构，并介绍了一些重要的主题，如框架、资源分配和资源隔离。还讨论了 Mesos 采用的两级调度方法，提供了其 API 的详细概述，并给出了 Mesos 在生产环境中使用的一些示例。

第二章，*Mesos 内部机制*，全面概述了 Mesos 的特性，并引导读者了解几个关于高可用性、容错性、扩展性和效率的关键主题，如资源分配、资源预留和恢复等。

第三章，*开始使用 Mesos*，讲解了如何在公共云（AWS、GCE 和 Azure）和私有数据中心（本地）上手动设置并运行 Mesos 集群。还讨论了各种调试方法，并深入探讨了如何排除 Mesos 设置中的故障。

第四章，*服务调度与管理框架*，介绍了几个基于 Mesos 的调度和管理框架或应用程序，这些框架或应用程序对于长时间运行服务的轻松部署、发现、负载均衡和故障处理至关重要。

第五章，*Mesos 集群部署*，解释了如何使用系统管理员和 DevOps 工程师常用的标准部署和配置管理工具，轻松地设置和监控 Mesos 集群。还讨论了在部署 Mesos 集群时常见的一些问题及其解决方案。

第六章，*Mesos 框架*，详细介绍了 Mesos 框架的概念和特性。它还提供了 Mesos API 的详细概述，包括新的 HTTP 调度器 API，并提供了在 Mesos 上构建自定义框架的方案。

第七章，*Mesos 容器化器*，介绍了容器的概念，并简单讲解了 Docker，可能是目前最流行的容器技术。它还提供了 Mesos 中不同“容器化器”选项的详细概述，并介绍了诸如 Mesos 管理的容器网络和获取缓存等其他主题。最后，提供了一个关于如何在 Mesos 中部署容器化应用的示例，以帮助更好地理解。

第八章，*Mesos 大数据框架*，是一本指南，介绍如何在 Mesos 上部署重要的大数据处理框架，如 Hadoop、Spark、Storm 和 Samza。

第九章，*Mesos 大数据框架 2*，引导读者通过在 Mesos 上部署重要的大数据存储框架，如 Cassandra、Elasticsearch-Logstash-Kibana（ELK）堆栈和 Kafka。

# 本书所需

为了最大限度地发挥本书的作用，你需要具备基本的 Mesos 和集群管理知识，同时熟悉 Linux 操作系统。你还需要能够访问 AWS、GCE 和 Azure 等云服务，最好是在 Ubuntu 或 CentOS 操作系统上运行，具有 15GB 内存和四个核心的配置。

# 本书适合谁阅读

本书旨在服务那些熟悉 Linux 系统及其工具管理基础的 DevOps 工程师和系统管理员。

# 约定

在本书中，你会看到一些不同的文本样式，以区分不同种类的信息。以下是这些样式的示例以及它们的含义解释。

文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟网址、用户输入和 Twitter 用户名以如下方式显示：“为了简化，我们将直接运行`sleep`命令。”

代码块如下所示：

```
{
  "args": [
    "--zk=zk://Zookeeper.service.consul:2181/Mesos"
  ],  
  "container": {
    "type": "DOCKER",
    "Docker": {
      "network": "BRIDGE",
      "image": "{{ Mesos_consul_image }}:{{ Mesos_consul_image_tag }}"
    }   
  },  
  "id": "Mesos-consul",
  "instances": 1,
  "cpus": 0.1,
  "mem": 256
}
```

当我们希望特别引起你对某个代码块部分的注意时，相关行或项目会以粗体显示：

```
 # Tasks for Master, Slave, and ZooKeeper nodes

- name: Install mesos package
  apt: pkg={{item}} state=present update_cache=yes
  with_items:
    - mesos={{ mesos_pkg_version }}
  sudo: yes
```

任何命令行输入或输出都以如下方式编写：

```
# Update the packages.
$ sudo apt-get update
# Install the latest OpenJDK.
$ sudo apt-get install -y openjdk-7-jdk
# Install autotools (Only necessary if building from git repository).
$ sudo apt-get install -y autoconf libtool
# Install other Mesos dependencies.
$ sudo apt-get -y install build-essential python-dev python-boto libcurl4-nss-dev libsasl2-dev maven libapr1-dev libsvn-dev

```

**新术语**和**重要词汇**以粗体显示。你在屏幕上看到的词汇，例如菜单或对话框中的词语，会以这样的方式出现在文本中：“现在按下**ADD**按钮以添加特定端口。”

### 注意

警告或重要提示会以这样的框框显示。

### 提示

提示和技巧如下所示。

# 读者反馈

我们始终欢迎读者的反馈。让我们知道你对本书的看法——你喜欢或不喜欢什么。读者反馈对我们非常重要，它帮助我们开发出你真正能从中获益的书籍。

如果您想向我们提供一般反馈，请通过电子邮件 `<feedback@packtpub.com>` 与我们联系，并在邮件主题中注明书名。

如果您有某个领域的专业知识，并且对编写或为书籍贡献内容感兴趣，请查看我们的作者指南 [www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在，作为一位骄傲的 Packt 图书拥有者，我们为您提供了许多帮助您充分利用购买的资源。

## 下载示例代码

您可以从您的帐户在 [`www.packtpub.com`](http://www.packtpub.com) 下载本书的示例代码文件。如果您在其他地方购买了这本书，您可以访问 [`www.packtpub.com/support`](http://www.packtpub.com/support) 并注册，让文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标悬停在顶部的 **SUPPORT** 标签上。

1.  点击 **Code Downloads & Errata**。

1.  在 **Search** 框中输入书名。

1.  选择您想要下载代码文件的书籍。

1.  从下拉菜单中选择您购买这本书的地点。

1.  点击 **Code Download**。

下载文件后，请确保使用最新版本的以下软件解压或提取文件夹：

+   适用于 Windows 的 WinRAR / 7-Zip

+   适用于 Mac 的 Zipeg / iZip / UnRarX

+   适用于 Linux 的 7-Zip / PeaZip

本书的代码包也托管在 GitHub 上，地址为 [`github.com/PacktPublishing/Mastering-Mesos`](https://github.com/PacktPublishing/Mastering-Mesos)。我们还有来自丰富图书和视频目录的其他代码包，您可以在 [`github.com/PacktPublishing/`](https://github.com/PacktPublishing/) 查看！快来看看吧！

## 勘误

虽然我们已经尽力确保内容的准确性，但错误确实会发生。如果您在我们的书中发现错误——可能是文本或代码中的错误——我们将非常感激您能将其报告给我们。通过这样做，您可以帮助其他读者避免困惑，并帮助我们改进该书的后续版本。如果您发现任何勘误，请访问 [`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata) 提交，选择您的书籍，点击 **勘误提交表单** 链接，并输入勘误的详细信息。您的勘误一旦验证通过，您的提交将被接受，勘误将上传至我们的网站或添加至该书籍的勘误列表中。

要查看先前提交的勘误，请访问 [`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索框中输入书名。所需信息将显示在**勘误**部分。

## 盗版

网络上的版权盗版问题在所有媒体中都持续存在。在 Packt，我们非常重视版权和许可的保护。如果您在互联网上发现任何形式的非法复制我们的作品，请立即提供该位置地址或网站名称，以便我们采取补救措施。

请通过 `<copyright@packtpub.com>` 与我们联系，并提供涉嫌盗版内容的链接。

感谢您帮助我们保护作者权益，以及确保我们能够为您提供有价值的内容。

## 问题

如果您对本书的任何内容有疑问，您可以通过 `<questions@packtpub.com>` 与我们联系，我们将尽力解决问题。
