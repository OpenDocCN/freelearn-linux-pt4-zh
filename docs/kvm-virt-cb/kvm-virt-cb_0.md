# 前言

现代大多数云部署的基础是某种虚拟化技术，例如基于内核的虚拟机（KVM）。KVM 自 2.6.20 版本起就成为主流 Linux 内核的一部分，该版本发布于 2007 年 2 月，自那时起，KVM 得到了广泛的系统应用。

虚拟化不仅提供了充分利用服务器资源的方式，还允许更大的多租户支持，并能在同一系统上运行各种工作负载。

OpenStack 云操作系统将 KVM 作为其默认计算驱动程序，提供了一种集中管理虚拟机生命周期的方式：从创建、调整大小、迁移到暂停和终止。

本书介绍了 KVM 及其如何以最有效的方式构建和管理虚拟机。与 Docker 等容器化解决方案不同，KVM 旨在运行整个操作系统，而不是单个进程。尽管容器化有其优势，完全虚拟化通过在客户操作系统和主机之间加入虚拟机监控程序层提供了额外的安全性，并且通过运行不同的客户操作系统内核（客户实例的内核崩溃不会导致整个主机宕机）或完全不同的操作系统提供了额外的稳定性。

本书采取了一种直接且务实的逐步方法——你将学习如何创建自定义客户镜像，安装和配置 QEMU 和 libvirt，调整实例大小和迁移实例，部署监控，并使用 OpenStack 和 Python 为客户提供服务。

# 本书涵盖的内容

第一章，*QEMU 和 KVM 入门，* 提供了安装和配置 QEMU、创建和管理磁盘镜像以及使用 qemu-system 工具运行虚拟机的食谱。

第二章，*使用 libvirt 管理 KVM，* 涵盖了使用 libvirt 安装、配置和运行 KVM 实例所需的一切内容。你将学习所需的包和工具，以及如何使用 XML 定义文件配置虚拟机的不同方式。在本章结束时，你将拥有一个运行 KVM 实例的 Linux 系统。

第三章，*使用 libvirt 进行 KVM 网络配置，* 将介绍如何使用 Linux 桥接和 Open vSwitch 工作，并演示如何使用 NAT、桥接和 PCI 直通网络连接 KVM 实例。

第四章，*迁移 KVM 实例，* 将展示如何进行运行中的 KVM 虚拟机的离线和在线迁移的示例。

第五章，*KVM 虚拟机的监控和备份，* 将展示如何使用 Sensu 和 Uchiwa 部署完整的监控系统，并演示如何创建快照以用作备份。

第六章，*使用 OpenStack 部署 KVM 实例*，演示了如何使用 OpenStack 配置 KVM 实例。它首先介绍了构成 OpenStack 的各种组件，以及如何使用 LXC Nova 驱动程序自动配置虚拟机。

第七章，*使用 Python 构建和管理 KVM 实例*，将展示使用 Python libvirt 库来构建、启动和管理 KVM 实例生命周期的教程。我们还将看到如何构建一个简单的 RESTful API 来与 KVM 配合使用。

第八章，*KVM 性能调优*，展示了如何调整宿主操作系统，以优化 I/O、CPU、内存和网络利用率。所示的示例也可以根据 KVM 实例的工作负载在其中使用。

# 本书所需的工具

跟随本书教程并运行示例需要具备初级 Linux 和命令行知识。要完全理解并能够运行第七章，*使用 Python 构建和管理 KVM 实例*中的示例，需要一些 Python 经验。

本书中的大多数教程已经在支持虚拟化的裸机服务器和最新版本的 Ubuntu Linux 上进行过测试。

# 本书适合谁

本书适合任何对 KVM 虚拟化感兴趣的人——从希望深入了解如何在大规模生产环境中部署和管理 KVM 的 Linux 管理员，到需要快速轻松地在隔离的客户机中原型化代码的软件开发人员。对于那些想从头到尾阅读本书并尝试所有示例的人来说，DevOps 工程师可能是最合适的职位名称。

# 章节

本书中，你会发现几个经常出现的标题（准备工作、如何操作、如何工作、还有更多内容、另见）。

为了清晰地指导完成一个教程，我们使用如下的几个部分：

# 准备工作

本节告诉你可以期待本教程中的内容，并描述了如何设置任何软件或进行任何预备设置。

# 如何操作…

本节包含了完成本教程所需的步骤。

# 如何工作…

本节通常包含对上一节内容的详细解释。

# 还有更多内容…

本节包含有关该教程的附加信息，以帮助读者更好地理解教程内容。

# 另见

本节提供了有助于本教程的其他有用信息的链接。

# 约定

本书中，你将看到一些文本样式，用于区分不同类型的信息。以下是这些样式的一些示例，并附上它们的含义解释。

文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟网址、用户输入和 Twitter 账号都以如下方式显示：“使用 `qemu-img` 管理磁盘镜像。”

代码块如下所示：

```
import libvirt
from bottle import run, request, get, post, HTTPResponse
def libvirtConnect():
try:
conn = libvirt.open('qemu:///system')
except libvirt.libvirtError:
conn = None
return conn 

```

任何命令行输入或输出都以如下格式书写：

```
root@kvm:~# apt-get update 

```

新术语和重要词汇以粗体显示。你在屏幕上看到的词汇，如在菜单或对话框中的内容，都会以如下方式出现在文本中：“KVM 实例的 memory_check 现在已显示在 Uchiwa 仪表板中。”

警告或重要说明通常以框的形式显示，如下所示。

小贴士和技巧通常像这样显示。

# 读者反馈

我们始终欢迎读者反馈。让我们知道你对本书的看法——你喜欢或不喜欢什么。读者的反馈对我们非常重要，它帮助我们开发出你真正能从中受益的书籍。

要向我们提供一般反馈，请直接通过电子邮件发送至 `feedback@packtpub.com`，并在邮件主题中提及书名。

如果你在某个领域具有专业知识，并且有兴趣为书籍撰写或贡献内容，请参阅我们的作者指南，链接为 [www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在，你已经是一本 Packt 书籍的骄傲拥有者，我们有很多帮助你最大限度利用购买内容的资源。

# 下载示例代码

你可以从你的账户下载本书的示例代码文件，链接为 [`www.packtpub.com`](http://www.packtpub.com)。如果你是从其他地方购买的本书，可以访问 [`www.packtpub.com/support`](http://www.packtpub.com/support) 并注册，以便将文件直接发送到你的电子邮箱。

你可以通过以下步骤下载代码文件：

1.  使用你的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标指针悬停在页面顶部的 SUPPORT 标签上。

1.  点击 Code Downloads & Errata。

1.  在搜索框中输入书名。

1.  选择你想要下载代码文件的书籍。

1.  从下拉菜单中选择你购买这本书的地方。

1.  点击 Code Download。

你也可以通过点击本书网页上的 Code Files 按钮下载代码文件。可以通过在搜索框中输入书名来访问此页面。请注意，你需要登录 Packt 账户。

文件下载完成后，请确保使用最新版的解压软件解压文件夹：

+   Windows 下的 WinRAR / 7-Zip

+   Mac 下的 Zipeg / iZip / UnRarX

+   Linux 下的 7-Zip / PeaZip

本书的代码包也托管在 GitHub 上，链接为 [`github.com/PacktPublishing/KVM-Virtualization-Cookbook`](https://github.com/PacktPublishing/KVM-Virtualization-Cookbook)。我们还提供了来自我们丰富书籍和视频目录的其他代码包，链接为 **[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**。快来查看吧！

# 下载本书的彩色图片

我们还为您提供了一份包含本书中使用的截图/图表的彩色图片的 PDF 文件。这些彩色图片将帮助您更好地理解输出中的变化。您可以从[`www.packtpub.com/sites/default/files/downloads/KVMVirtualizationCookbook_ColorImages.pdf.`](https://www.packtpub.com/sites/default/files/downloads/KVMVirtualizationCookbook_ColorImages.pdf)下载该文件。

# 勘误

虽然我们已尽最大努力确保内容的准确性，但错误还是会发生。如果您在我们的书籍中发现错误——无论是文字错误还是代码错误——我们将非常感激您能向我们报告。通过这样做，您不仅可以帮助其他读者避免困惑，还可以帮助我们改进后续版本的内容。如果您发现任何勘误，请访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击“勘误提交表单”链接，并输入勘误的详细信息。一旦您的勘误得到验证，您的提交将被接受，勘误将上传到我们的网站或添加到该书籍的勘误部分的现有勘误列表中。

若要查看之前提交的勘误表，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索框中输入书名。所需的信息将在“勘误”部分显示。

# 盗版

网络上存在版权材料盗版问题，涉及所有媒体。在 Packt，我们非常重视保护我们的版权和许可证。如果您在互联网上遇到我们作品的任何非法复制品，请立即向我们提供位置地址或网站名称，以便我们采取措施。

请通过`copyright@packtpub.com`与我们联系，并提供涉嫌盗版材料的链接。

我们感谢您在保护我们的作者和确保我们能为您带来有价值的内容方面的帮助。

# 问题

如果您在本书的任何方面遇到问题，您可以通过`questions@packtpub.com`与我们联系，我们将尽力解决问题。
