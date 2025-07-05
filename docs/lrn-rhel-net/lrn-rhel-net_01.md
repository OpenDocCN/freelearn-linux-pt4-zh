# 第一章：介绍企业级 Linux 7

欢迎来到企业级 Linux 版本 7 的世界。这在 2014 年 6 月 9 日首次向我们介绍。Red Hat 从 2013 年 12 月 11 日的 RHEL 7 beta 版开始了它的旅程。随后在 4 月 23 日发布了下一个候选版本。最后，正如预期的那样，黄金版在 2014 年 6 月上市。在撰写本书时，我们的示范将使用 beta 版本的 Update 1。

本章将帮助您理解为什么企业级 Linux 与其他最前沿的发行版不同。它还将帮助您了解 Red Hat、CentOS 和 Fedora 之间的关系。我们还希望本章能够帮助您深入了解如何在您选择的硬件平台上使用 RHEL 7。本章的主题如下：

+   Red Hat Enterprise Linux

+   CentOS

+   Fedora

+   确定您的发行版和版本

# Red Hat Enterprise Linux

当我们谈到 Linux 时，很多时候，Red Hat 将是主要考虑的对象；尤其是在企业级别工作时，Red Hat 几乎肯定会成为我们的一部分。可靠性、可预测性和稳定性是与这家非常赚钱和成功的组织密切相关的词汇。为了让大家了解他们的最近成功，该公司在 2010 年纳斯达克（RHT）的股价不到 30 美元。然而，到了 2014 年底，他们的价值围绕着 60 美元左右。

企业级 Linux 不太可能处于最前沿。作为企业发行版，它必须是可支持和可靠的。随着 RHEL 7 的发布，我们看到 Linux 内核的第三个版本在 RHEL 中首次使用。Linux 内核版本 3 于 2011 年 7 月 22 日首次亮相。因此，我们可以说企业级 Linux 可能落后于最新和最伟大的版本约 3-4 年。

在许多方面，可靠性比版本 3 将提供的新内核功能更为重要。这些新功能通常与硬件相关，但并不重要，因为企业级硬件必须在关键任务环境中采取类似的谨慎方法。我们发现企业级硬件必须是可靠的，这也许导致缺乏未经测试的新功能。新硬件和驱动程序的开发可以在小型企业和家庭用户中进行测试。这些测试者可以经历折磨，而开发工作可以为我们的关键任务服务器改进。蓝筹企业公司要求的支持水平超出了在支持论坛中发布技术查询并希望有人看到并回应的范围。几乎可以肯定，任何金融机构都必须能够证明其对系统的支持水平。这可以通过提供支持协议或合同以及相关的服务水平协议（SLA）来轻松实现。为此，Red Hat 并非免费，但收费是为了提供支持，而非为了发行目的。最基本的支持水平每年大约为 350 美元（美国 dollars）。

Red Hat 于 2002 年推出了**企业 Linux**，并发布了 RHEL 版本 2.1。最初，支持期为 10 年，但随着 RHEL 7 的发布，已延长至 13 年。这意味着 RHEL 7 的支持期可以延长至 2027 年 6 月 30 日。目前的 RHEL 7.1 beta 版本使用的是 Linux 内核 3.10.0-210，相较于 7.0 版本的 3.10.0-123，我们可以看到内核版本的微小增长，显示出在推出任何版本的 RHEL 时的谨慎态度。在写作时，维护者提供的最新 Linux 内核版本是 3.18.1（来自[`www.kernel.org`](https://www.kernel.org)）。

Red Hat 产品可以从[`access.redhat.com/downloads`](https://access.redhat.com/downloads)下载。你需要创建一个账户才能开始评估并下载 RHEL。

# CentOS

**CentOS**（**社区企业操作系统**）多年来作为 Red Hat 重建版本被广泛使用，并且完全免费。这是指 Red Hat 的标识和品牌从系统中移除，然后作为“CentOS”重新分发。这听起来并没有想象中那么糟糕。Red Hat 使用开源代码，且重新分发完全符合**GPL**（**GNU 通用公共许可证**）协议的要求。你失去的是支持。因此，你可能会发现 CentOS 更多地用于较小的商业操作和学术界（在这些地方，外部支持并不是那么重要）。CentOS 的支持只能通过公共论坛获得。当然，这意味着没有可保证的服务水平。

CentOS 于 2004 年开始运营，现在已进入第二个十年。它推出的免费产品在市场上复制了与其 Red Hat 兄弟相同的可靠性和可预测性。Red Hat 与 CentOS 之间的关系在 2014 年 1 月变得更加正式。CentOS 的治理小组现在包括 Red Hat 董事会成员。

CentOS 不像 Red Hat 那样发布测试版本。这意味着 CentOS 稳定版本中最新的版本是 7.0。它将使用与 RHEL 7.0 发行版相同的内核和版本 3.10.0-123。CentOS 与 Red Hat 的高度相似，通常意味着 CentOS 成为那些希望学习 Red Hat 并可能获得认证的用户的完美学习平台。这无疑是一个非常可行的选项，同样适用于学习本书内容。尽管我们将使用 RHEL 7.1 测试版，但如果你想使用 CentOS，它应该非常相似，且大部分与 CentOS 7 兼容。

由于 CentOS 不提供订阅支持，这反过来会影响产品生命周期。为了获得 RHEL 7 提供的 13 年完整支持，RHEL 客户将需要购买额外的支持来覆盖最后 3 年的服务。这意味着 CentOS 有仓库可以分发更新，持续更新 10 年，这导致 CentOS 7 可以持续更新到 2024 年 6 月。从这个角度看，CentOS 7 的支持时间相当不错，且完全没有财务成本。

你可以直接从 [`centos.org/download/`](http://centos.org/download/) 下载 CentOS 的最新版本，无需创建账户。

# Fedora

我们可以说 Fedora 是 Red Hat 的家庭版。虽然我们将其标记为家庭版，Fedora 也有服务器版，而你可以自由选择如何和在哪里使用 Fedora。它对新款笔记本和最新硬件的支持将大大增强，这使得它通常成为家庭用户和爱好者的首选。当前版本是 Fedora 21，几乎采用了所有最新的内核，版本为 3.17.4-301。

使用 Fedora 的另一个好处，即使不是用于生产环境，也能让你熟悉一些技术。这些技术最终会在某个时刻变得企业级可用。通过这种方式，你会在产品开发的过程中学习。例如，RHEL 7 基于 Fedora 19 和 20。如果你是 Fedora 的忠实支持者，你将已经熟悉 GRUB2、BTRFS、docker 和 systemd（这些都首次出现在 RHEL 7 中）。

Fedora 的支持是由社区驱动的，软件更新大约会在初始产品发布后的 13 个月内提供。例如，Fedora 21 会在 Fedora 23 发布后再支持 1 个月。发布周期大约为每 6 个月一次，这使得我们大致可以得到 13 个月的支持生命周期。这也是为什么 Fedora（以及类似的 Fedora 发行版）通常无法进入企业级类别的原因，因为更新生命周期如此之短。

对于学习和家庭使用，这是一个非常棒的发行版。你可以选择下载工作站版、服务器版或云版本，网址是 [`getfedora.org/`](https://getfedora.org/)。

从流行度的角度来看，Fedora 绝对算得上是其中之一。过去十二个月中，Fedora 下载页面的点击次数使其成为第四大最受欢迎的发行版。为了支持这些数据并查看我们从哪里获取这些信息，你可以访问 [`www.distrowatch.com`](http://www.distrowatch.com)。

# 确定你的发行版和版本

如果你是从头开始安装，那么我们希望你能确定自己到底在安装什么。如果你不能确定，那我们在安装之前就需要解决一些问题。然而，你可能会遇到一台预装的机器或一台你可以访问的实验室机器。进行任何故障排查的明显第一步是确定我们将要使用的操作系统和补丁级别。接下来我们将介绍多种方法，帮助你确定你将使用的 Linux 发行版。

## /etc/system-release 文件

`/etc/system-release` 文件在我们讨论过的所有 Red Hat 变种中都是一致的。可以通过 `cat` 命令（即连接命令）简单地读取它。实际上，在所有三种系统上，这个文件都是一个符号链接，指向以下列表中的相关文件：

```
/etc/redhat-release
/etc/centos-release
/etc/fedora-release

```

然而，阅读链接中的文件是有意义的，因为 `/etc/system-release` 文件在所有这些发行版中都会存在，并指向正确的操作系统文件。在示范的 RHEL 7.1 系统上运行以下命令会显示如下内容：

```
$ cat /etc/system-release

Red Hat Enterprise Linux Server release 7.1 Beta (Maipo)

```

## /etc/issue 文件

第二种方法可以是从物理机器上的标准终端读取登录横幅。如果设备上没有运行图形界面，物理终端可以是 `tty1` 到 `tty6`。然而，如果你在桌面或服务器上运行图形界面，通常 `tty2` 是第一个命令行终端。你可以通过 *CTRL* + *ALT* + *F2* 键序列从图形界面访问此终端。在登录提示符之前，将显示 `/etc/issue` 的内容。`/etc/issue` 内容需要通过 `/sbin/agetty` TTY 程序来读取。我们可以连接这个文件，但它没有什么用，因为它包含了特殊的转义字符，这些字符由 **agetty** 扩展。以纯文本查看该文件时，看到如下命令：

```
$ cat /etc/issue

\S
Kernel \r on an \m

```

`\S` 命令将显示操作系统，`\r` 命令将显示内核版本，`\m` 命令将显示机器类型。在 RHEL 7.1 系统上，我们将使用这个文件，在终端登录时显示如下内容：

```
Red Hat Enterprise Linux Server 7.1 (Maipo)
Kernel 3.10.0-210.el7.x86_64 on an x86_64

```

## 使用 lsb_release

如果有一种方法可以显示你正在使用的操作系统的详细信息，为什么不可以有三种方法呢！像 Linux 一般，我们可以通过多种方式解决这个问题。第三种方法是使用 `lsb_release` 命令。该命令通常不会作为默认安装的一部分，因此需要添加到你的系统中（如果尚未完成此操作）。

安装此软件可以通过 `yum` 实现，但需要以 `root` 用户（管理员）身份运行。因此，可以使用 `su -` 切换到 root 账户，或者如果你的账户已设置为管理员，则使用 `sudo`，如下面的代码所示：

```
$ sudo yum install redhat-lsb-core

```

### 提示

如果你是 Linux 管理的新手，那么下一章将从如何在 RHEL 中获取和管理管理员权限的快速教程开始。

尽管软件包名称中有 `redhat` 元素，但此命令也可以在 CentOS 上使用（如果你使用的是 CentOS 来学习企业级 Linux 7）。安装该软件包后，我们将使用 `lsb_release` 命令来识别操作系统。在本书中使用的系统上，我们可以查看以下输出：

```
$ lsb_release -a
LSB Version:    :core-4.1-amd64:core-4.1-noarch
Distributor ID: RedHatEnterpriseServer
Description:    Red Hat Enterprise Linux Server release 7.1 Beta (Maipo)
Release:        7.1
Codename:       Maipo

```

如果你使用的是 Fedora，可以通过以下命令安装该软件包：

```
$ sudo yum install redhat-lsb

```

输出结果类似，但与 Fedora 版本相关，以下是来自 Fedora 21 服务器的输出：

```
$ lsb_release -a
LSB Version:    :core-4.1-amd64:core-4.1-noarch:cxx-4.1-amd64:cxx-4.1-noarch:desktop-4.1-amd64:desktop-4.1-noarch:languages-4.1-amd64:languages-4.1-noarch:printing-4.1-amd64:printing-4.1-noarch
Distributor ID: Fedora
Description:    Fedora release 21 (Twenty One)
Release:        21
Codename:       TwentyOne

```

## 确定内核版本

我们已经看到，在登录终端时，Linux 内核版本可能会通过 **/etc/issue** 文件显示。然而，我们也可以通过 `uname -r` 命令轻松地显示当前内核的版本。内核是操作系统的核心，由 Linux 基金会作为开源软件进行维护。该命令可以作为普通用户运行。在 RHEL 7.1 系统上，它显示以下信息：

```
$ uname -r
3.10.0-210.el7.x86_64

```

同样，了解 Linux 内核的版本是建立系统故障排除和进行支持请求的一个很好的起点。

# 概述

到现在为止，我希望你对如何跟随本书学习 Red Hat Enterprise Linux 7 网络有了更好的理解，无论是在 RHEL、CentOS 还是 Fedora 上。你现在应该能够区分每个发行版的优势，并识别你将要使用的版本。

在下一章中，我们将开始研究如何在 RHEL 7 上配置网络。此外，我们还将探讨如何使用 `su` 或 `sudo` 获得管理员权限及其好处。这对于那些 Linux 管理新手以及不熟悉作为管理员运行任务的用户特别有用。
