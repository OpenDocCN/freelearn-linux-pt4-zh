# 序言

VMware Horizon View，作为 VMware 数字工作空间的一部分，是提供集中式虚拟桌面和应用程序的平台，这些桌面和应用程序托管在数据中心运行的虚拟化管理程序上的服务器上。最终用户通过终端设备（如 Windows 笔记本电脑、Apple Mac 或平板设备）远程连接到他们的虚拟桌面和应用程序。

这项技术最早由 VMware 于 2002 年推出，并发展成熟，成为今天我们所知的主流技术——**虚拟桌面基础设施**（**VDI**）。VDI 通过解放用户不必待在办公室的限制，使他们能够选择适合自己的设备进行工作，进而提高生产力，最终让你的业务变得更加灵活。

从 IT 管理员的角度来看，它使你能够集中管理桌面环境，从管理桌面镜像到轻松添加和移除用户权限，所有操作都可以通过单一的管理控制台进行管理。

VMware Horizon 7 和 Horizon View 7.7 版本是 VMware 最新的虚拟桌面解决方案，旨在通过 VMware 的**软件定义数据中心**（**SDDC**）产品组合中的市场领先虚拟化功能和技术，集中化并虚拟化你的桌面环境。

Horizon View 7 基于这一技术平台，今天远远超越了仅仅提供 VDI 的范畴，提供了丰富的用户体验，支持 BYOD（自带设备）、灵活工作、增强的安全性、应用交付和端到端管理。提供最终用户体验需要与其他基础设施型项目不同的方法，做到这一点是项目成功的关键，本书将向你展示如何实现成功。

# 本书适合的人群

如果你是桌面管理员或参与部署虚拟桌面和/或应用交付解决方案的项目团队成员，或者正在利用一些最新的 Horizon 功能，那么本书将是你完美的伙伴，帮助你部署一个解决方案，集中管理和虚拟化你的桌面环境，使用 Horizon 7。

你需要具备使用 Microsoft Windows 桌面和服务器操作系统进行桌面管理的经验，以及对 Windows 应用程序、Active Directory、SQL 和 VMware vSphere 基础设施（ESXi 和 vCenter Server）技术的了解。

# 本书内容

第一章，*介绍 VDI 和 VMware Horizon 7*，涵盖了 VDI 的介绍，解释了它是什么，以及它与其他 VDI 类型技术的比较。然后我们将简要回顾 VMware VDI 的历史，并提供最新解决方案的概述。

第二章，*理解 Horizon 7 架构与组件*，介绍了构成核心 VMware Horizon 解决方案的架构组件，重点讲解 Horizon View 的虚拟桌面元素及虚拟桌面机器的经纪功能。

第三章，*设计与部署考虑事项*，向你介绍在进行 VMware Horizon 项目时需要考虑的设计与部署技巧。我们将讨论如何验证技术，了解它如何在企业内部运行，评估用户现有工作负载的方法，以及如何使用

这些信息将帮助你设计你的 VMware Horizon 解决方案。

第四章，*安装和配置 Horizon 7 – 第一部分*，介绍了 Horizon View 核心组件的安装过程，这部分涉及连接服务器和 View Composer 的安装。

第五章，*安装与配置 Horizon 7 – 第二部分*，通过安装安全服务器、复制服务器、注册服务器，以及云端架构功能，完成 Horizon 环境的安装。安装完成后，我们将开始配置 Horizon View 安装的基本元素。

第六章，*使用 SSL 证书保障 Horizon View 安全*，讲述了 VMware Horizon View 的安全通信，特别是如何为终端用户客户端提供安全通信，以及数据中心内不同的基础设施组件。本章的前半部分将概述 SSL 证书是什么，然后介绍如何创建和发布证书，接着配置 Horizon View 使用证书。在本章的后半部分，我们将探讨如何配置 VMware True SSO 功能。

第七章，*构建和优化虚拟桌面操作系统*，介绍了在构建 Horizon View 基础设施及其组件之后，如何创建和配置虚拟桌面机器，并在其上构建桌面操作系统，配置使其在虚拟环境中以最佳性能运行。

第八章，*配置和管理桌面池 – 第一部分*，讲述了 Horizon View 如何利用桌面池的概念，为特定的使用场景创建虚拟桌面机器的集合，并将这些集合分配给最终用户。在本章中，我们将讨论配置不同类型桌面池的过程。

第九章，*配置和管理桌面池 – 第二部分*，完成了桌面池管理的过程，重点介绍了链接克隆、手动桌面池以及一些日常管理任务。

第十章，*优化终端用户体验*，介绍了在构建最佳用户体验时的关键任务之一，即开始优化终端用户与虚拟桌面机器会话的性能和体验。在本章中，我们将探讨调优技巧以及可以应用的预构建组策略对象，以创造这种体验。

第十一章，*使用 Horizon 7 发布应用程序*，深入探讨了 Horizon 高级版的关键功能，并讲解了 Horizon View 如何直接在 Horizon View 客户端中发布应用程序，而无需启动完整的虚拟桌面机器。本章将带您走过安装和配置过程，以便将我们第一组 Horizon View 发布的应用程序提供给终端用户。

第十二章，*Horizon 客户端选项*，讲述了如何使用 Horizon 客户端接收并显示终端用户设备上的虚拟桌面和应用程序。在本章中，我们将探讨 Horizon 客户端的硬件和软件选项，并讨论各种选项以及为什么选择某种方法而不是其他方法。

第十三章，*升级到新的 Horizon 版本*，介绍了升级前需要考虑的所有事项，并将带领您完成升级过程。本章旨在帮助当前运行早期版本 Horizon View 的用户升级到最新版本。

第十四章，*JMP 和 VMware Horizon 7 部署注意事项*，介绍了 Horizon 7 中新增加的即时管理平台功能，并讲解了如何将 UEM、App Volumes 和 Instant Clones 结合使用以按需交付桌面。本章讨论了架构，并引导您完成安装和配置过程。

第十五章，*故障排除*，介绍了在 Horizon View 中采用的一些故障排除技术和方法，而不是列举问题和故障。

第十六章，*Horizon 7 中的新特性*，讨论了 Horizon 7 最新版本中的最新功能。

[第十七章](https://www.packtpub.com/sites/default/files/downloads/Managing_the_End_User_Environment_in_Horizon.pdf)，*管理 Horizon 中的最终用户环境*，介绍了 Horizon View Persona 管理，讲解了它是什么以及为什么你要部署它。接下来，我们将探讨它如何通过标准的活动目录组策略驱动，最后深入了解可用的策略。本章的第二部分将介绍 VMware UEM，以及如何开始使用它。

本章内容请参考：[`www.packtpub.com/sites/default/files/downloads/Managing_the_End_User_Environment_in_Horizon.pdf`](https://www.packtpub.com/sites/default/files/downloads/Managing_the_End_User_Environment_in_Horizon.pdf)。

[第十八章](https://www.packtpub.com/sites/default/files/downloads/Delivering_Published_Desktops_with_Horizon_7.pdf)，*使用 Horizon 7 交付已发布的桌面*，介绍了 View 的发布功能的另一半，探讨了 Horizon View 如何从 Microsoft RDSH 基础设施中交付基于会话的桌面。

本章内容请参考：[`www.packtpub.com/sites/default/files/downloads/Delivering_Published_Desktops_with_Horizon_7.pdf`](https://www.packtpub.com/sites/default/files/downloads/Delivering_Published_Desktops_with_Horizon_7.pdf)。

# 为了充分利用本书

为了充分利用本书，你应该具备一定的桌面管理员经验，拥有与构建和设计基于 Microsoft Windows 的桌面环境相关的技能和知识。你还应熟悉 VMware vSphere 平台（ESXi 和 vCenter Server），并且能够自如地构建和配置虚拟机，以及为虚拟基础设施配置存储和网络。

在本书中，你将有机会按照逐步的实用指南，在示例实验室环境中部署 Horizon View。如果你想通过实际示例进行操作，你将需要以下软件：

+   VMware Horizon 7 或版本 7.6、7.7、7.8

+   vSphere for Desktop（ESXi 和 vCenter Server 6.5）

你可以从以下链接下载 Horizon 7 的试用版（你需要一个 VMware 账户）：

[`my.vmware.com/en/web/vmware/evalcenter?p=horizon-7`](https://my.vmware.com/en/web/vmware/evalcenter?p=horizon-7)

你还需要以下软件来构建虚拟机并部署应用程序：

+   Microsoft Windows Server 2016 64 位

+   Microsoft Windows 7 Professional 32 位或 64 位

+   Microsoft Windows 10

+   Microsoft SQL Express 2012

+   Microsoft Office 2016

# 下载彩色图像

我们还提供了一份包含本书中截图/图表彩色图像的 PDF 文件。你可以在这里下载：[`www.packtpub.com/sites/default/files/downloads/9781789802375_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/9781789802375_ColorImages.pdf)。

# 使用的约定

本书中使用了多种文本约定。

`CodeInText`：表示文本中的代码词汇、数据库表名、文件夹名称、文件名、文件扩展名、路径名、虚拟网址、用户输入和 Twitter 账号。例如：“此第二实例的连接服务器将安装在虚拟机上，虚拟机的主机名为 `hzn7-cs2.pvolab.com`，该虚拟机在本章开始时已创建。”

任何命令行输入或输出如下所示：

```
net stop certsvc
net start certsvc
```

**粗体**：表示新术语、重要单词或屏幕上看到的词汇。例如，菜单或对话框中的词汇会像这样出现在文本中。这里是一个例子：“在‘选择数据库所有者’框中，点击‘确定’以接受数据库所有者。”

警告或重要提示会以这种方式显示。

小贴士和技巧会以这种方式显示。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：如果您对本书的任何内容有疑问，请在邮件主题中注明书名，并通过电子邮件联系我们 `customercare@packtpub.com`。

**勘误表**：虽然我们已经尽力确保内容的准确性，但错误仍然可能发生。如果您在本书中发现错误，我们将非常感激您向我们报告。请访问 [www.packt.com/submit-errata](http://www.packt.com/submit-errata)，选择您的书籍，点击“勘误提交表格”链接，并填写相关细节。

**盗版**：如果您在互联网上发现我们的作品的非法复制品，无论形式如何，我们将非常感激您提供相关的地址或网站名称。请通过电子邮件将该链接发送至 `copyright@packt.com`。

**如果您有兴趣成为作者**：如果您在某个领域拥有专长，并且有兴趣撰写或为书籍贡献内容，请访问 [authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。在您阅读并使用本书后，不妨在您购买该书的网站上留下评论。潜在读者可以参考您的公正意见做出购买决策，我们 Packt 也能了解您对我们产品的看法，而我们的作者也可以看到您对其书籍的反馈。谢谢！

如需了解更多 Packt 的信息，请访问 [packt.com](http://www.packt.com/)。
