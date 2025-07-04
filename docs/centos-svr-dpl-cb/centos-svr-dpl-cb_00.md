# 前言

在过去的十多年里，CentOS 项目通过重新品牌化和重新编译 Red Hat Enterprise Linux 源代码，为社区提供了一个免费的企业级操作系统。由于 CentOS 用户几乎完全依赖社区提供支持，因此当 Packt 联系我撰写关于该项目最新版本 CentOS 7 的书时，我非常愿意参与。我们选择的配方涵盖了从入门到管理许多常见 Web 服务的广泛主题，希望所有技能水平的管理员都能找到有趣的内容。

然而，写一本书是一项巨大的工作。因此，我要感谢 Packt 的工作人员、我的家人和朋友们的支持。狗需要遛弯，家庭事务需要参加，工作场所也会有突发事件。如果没有身边人的理解和鼓励，以及编辑团队的帮助，你就无法看到这本书。

# 本书内容

本书中提供的配方旨在通过提供逐步指导和讨论，使即使是最复杂的配置任务也变得简单。以下是你可以期待从每一章中获得的内容的简要概述。

第一章，*CentOS 入门*，包含了使用图形化、基于文本和 Kick-start 方法安装 CentOS 的配方。还讨论了如何为在 Docker 和 Amazon Web Services 上运行的项目设置 CentOS 平台。

第二章，*网络配置*，包含帮助你完成常见网络任务的配方，例如如何设置静态 IP 地址、为单个网络接口分配多个地址、使用相同地址绑定多个接口，以及如何使用 FirewallD 和 iptables 配置系统防火墙。它还提供了配置网络服务（如 DHCP、NFS 和 Samba）的配方。

第三章，*用户和权限管理*，展示了如何通过强制执行密码限制、调整新创建文件和目录的默认权限以及使用 sudo 避免传播 root 密码来增强系统的安全性。还讨论了如何与 SELinux 配合使用。

第四章，*软件安装管理*，提供了专注于软件仓库和安装软件的配方。你将学习如何注册 EPEL 和 Remi 仓库，如何优先选择软件包的安装源，以及如何自动更新软件。你还将学习如何从源代码编译和安装软件。

第五章，*管理文件系统与存储*，介绍了如何设置和使用 RAID 和 LVM 的操作步骤。这些服务利用系统的存储来保持可用性，增加可靠性，并保护数据免受硬盘故障的影响。

第六章，*允许远程访问*，旨在帮助你以安全的方式为 CentOS 系统提供远程访问。其操作指南涉及使用 SSH，配置 chroot 监狱，以及通过加密的 SSH 隧道进行 VNC 连接。

第七章，*与数据库工作*，汇集了提供必要步骤的操作指南，帮助你入门 MySQL、MongoDB 和 OpenLDAP 等数据库服务。你还将学习如何为这些服务提供备份和冗余。

第八章，*管理域名与 DNS*，带你进入 DNS 的世界。操作指南展示了如何设置一个解析 DNS 服务器，以减少由于域名查找引起的延迟，以及如何使用权威 DNS 服务器管理自己的域名。

第九章，*管理电子邮件*，将帮助你设置自己的邮件服务器。内容涵盖配置 Postfix 提供 SMTP 服务，配置 Dovecot 提供 IMAP 和 POP3 服务，并使用 TLS 对这些服务进行加密保护。你还将看到如何设置 SpamAssassin 来减少未经请求的大宗电子邮件。

第十章，*管理 Web 服务器*，包含了配置 Apache 来提供 Web 内容的操作指南。你将学习如何设置基于名称的虚拟主机，如何通过 HTTPS 提供服务器页面，以及如何进行 URL 重写。还将讨论如何设置 NGINX 作为负载均衡器。

第十一章，*防范威胁*，包含了帮助保护你在 CentOS 服务器上投资内容的操作指南。内容涉及日志记录、威胁监控、病毒和 Rootkit，以及网络备份。

第十二章，*虚拟化*，展示了如何将 CentOS 作为主机操作系统，为一个或多个虚拟化客户端提供服务。这使得你能够通过在同一物理系统上运行多个操作系统，更好地利用硬件资源。

# 本书所需内容

要跟随本书中的操作指南，首先，你需要一台能够运行 CentOS 7 的系统。最低要求（以及最大能力）已在 Red Hat Enterprise Linux 知识库中记录，网址为 [`access.redhat.com/articles/rhel-limits`](https://access.redhat.com/articles/rhel-limits)。简而言之，你需要一台具备以下条件的系统：

+   x86_64 处理器（RHEL/CentOS 7 不支持 x86）

+   1 GB 内存

+   8 GB 磁盘容量

除了安装 CentOS 的系统外，你还需要一份 CentOS 安装介质和一个正常工作的网络连接。你可以直接从[`www.centos.org/download/`](https://www.centos.org/download/)下载，或者使用 BitTorrent。

# 本书适用对象

本书适合具备基本 Unix/Linux 功能经验的 Linux 专业人士，可能曾经设置过服务器，想要提升自己在管理各种服务方面的知识。

# 部分

本书中，你会找到一些经常出现的标题（*准备工作*、*如何操作…*、*原理介绍…*、*更多内容…* 和 *参见*）。

为了提供明确的配方完成指导，我们使用以下这些部分：

## 准备工作

本节告诉你在配方中应期待什么，并描述了设置所需的任何软件或前期设置。

## 如何操作…

本节包含执行配方所需的步骤。

## 原理介绍…

本节通常包含对上一节内容的详细解释。

## 更多内容…

本节包含有关配方的附加信息，旨在使读者更全面地了解配方。

## 参见

本节提供有用的链接，帮助你获取配方中的其他相关信息。

# 约定

本书中，你将看到多种文本样式，用于区分不同类型的信息。以下是这些样式的一些示例及其含义的解释。

文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟网址、用户输入和 Twitter 账号将按如下方式显示：“仓库的配置文件位于`/etc/yum.repos.d`目录下。”

代码块设置如下：

```
    [sshd]
    enabled=true
    bantime=86400
    maxretry=5
```

所有命令行输入或输出如下所示：

```
 firewall-cmd --zone=public --permanent --add-service=dns

```

**新术语**和**重要词汇**以粗体显示。屏幕上显示的词汇，例如在菜单或对话框中，出现在文本中如下：“选择你希望的语言并点击**继续**。”

### 注意

警告或重要提示会以这种框体形式显示。

### 提示

提示和技巧以这种形式出现。

# 读者反馈

我们始终欢迎读者反馈。让我们知道你对这本书的看法——你喜欢或不喜欢的地方。读者反馈对我们非常重要，因为它帮助我们开发出你能真正受益的书籍。

若要向我们发送一般反馈，只需通过电子邮件 feedback@packtpub.com 联系我们，并在邮件主题中提及书名。

如果你在某个领域有专长，并且有兴趣为书籍写作或贡献内容，请参阅我们的作者指南：[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在你已经成为一本 Packt 书籍的骄傲拥有者，我们有许多帮助你从购买中获得最大收益的资源。

## 勘误

尽管我们已经尽最大努力确保内容的准确性，但错误还是会发生。如果你在我们的书中发现错误——可能是文本或代码中的错误——我们将非常感激你向我们报告。这样，你不仅能帮助其他读者避免困扰，还能帮助我们改进该书的后续版本。如果你发现任何勘误，请通过访问 [`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata) 提交，选择你的书籍，点击 **勘误提交表单** 链接，填写勘误的详细信息。一旦勘误得到确认，你的提交将被接受，勘误将上传至我们的网站或添加到该书的勘误列表中。

要查看之前提交的勘误，请访问 [`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索框中输入书名。所需的信息将在 **勘误** 部分显示。

## 盗版

互联网版权材料的盗版问题在所有媒体中持续存在。在 Packt，我们非常重视保护我们的版权和许可证。如果你在互联网上遇到任何形式的非法复制品，请立即提供相关地址或网站名称，以便我们采取措施解决此问题。

如有盗版材料的疑虑，请通过 copyright@packtpub.com 与我们联系，并提供相关链接。

我们感谢你帮助保护我们的作者和我们为你带来有价值内容的能力。

## 问题

如果你在本书的任何部分遇到问题，可以通过 questions@packtpub.com 联系我们，我们将尽力解决问题。
