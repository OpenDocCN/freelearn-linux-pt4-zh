# 前言

本书将提供一个通过 CentOS 7 学习 Linux 必备技能的介绍。它描述了 Linux 系统的组织结构，并介绍了关键的命令行概念，读者可以自行练习。它引导读者执行基本的系统管理任务和日常操作。

如今，Linux 无处不在。它是大多数技术创新的核心，驱动着从最小的智能设备到全球最大的超级计算机等各种设备。Linux 内核开发至今仍是全球最大的合作项目。读者将学习 Linux 和开放源代码技术在现代计算环境中的基础知识。本书将以简明且实用的方式向用户介绍 CentOS 7。书中大多数命令行都配有图示，以帮助更好地理解。

阅读完本书后，你将对使用命令行操作 Linux 有扎实的理解。你将学习如何管理运行 CentOS 7 或类似操作系统（如 RHEL 7、Scientific Linux 和 Oracle Linux）的系统的核心系统管理技能。阅读完本书后，你将能够执行安装、建立网络连接、管理用户和进程、修改文件权限、使用命令行管理文本文件，并实现基本的安全管理。

# 本书适合谁阅读

本书适合任何希望在其环境中将 Linux 用作服务器或桌面计算机的个人。无论你是开发人员、新手系统管理员，还是没有 Linux 管理背景的技术爱好者，本书将帮助你以 CentOS 7 为基础，开始你的 Linux 学习之旅。

尽管本书是为初学者编写的 Linux 使用指南，但有经验的 Linux 用户也能从每一章中获得一些收获。你不需要任何 Linux 命令行的工作经验就能阅读本书。大多数 Linux 新手发现使用命令行很困难，偶尔也会对选择从哪个 Linux 发行版开始感到困惑。你将使用 CentOS 7 学习 Linux，它是基于 RHEL 7 的最受欢迎和最稳定的 Linux 发行版之一。

本书的一些关键特点如下：

+   阅读本书前无需任何 Linux 环境经验

+   读者将会熟悉一个流行且稳定的 Red Hat 企业版 Linux 发行版

+   内容简明扼要，全面覆盖重要工具

本书以一种易于理解的方式编写，任何对操作系统有基本了解的计算机用户都可以开始使用它。唯一的前提是你有一台足够好的硬件，可以安装 CentOS 7 并练习书中的命令。

# 如何充分利用本书

一如既往，我们已尽最大努力确保本书内容符合用户需求。本书中涉及的所有命令行均基于 CentOS 7。您可以使用任何小版本的 CentOS 7，从 CentOS 7.1 到 CentOS 7.6。本书唯一的要求是使用 CentOS 7 操作系统。然而，对于初学者，建议在任何桌面虚拟化应用程序中安装并练习 CentOS 7，例如 VirtualBox 和 VMWare Workstation。

对于希望使用虚拟环境的 Windows 和 macOS 用户，他们可以使用 VMWare 或 VirtualBox 设置 CentOS 7 并执行给定的命令行示例。对于 Linux 新手，本书第一章 *CentOS 7 入门* 中详细介绍了 CentOS 7 的安装方法。

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码字、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 账户名。例如：“例如，`mkdir` 使用中的 `DIRECTORY..` 表示我们需要插入我们希望与 `mkdir` 命令一起使用的目录名。”

任何命令行输入或输出如下所示：

```
$ mkdir -p -v demo/linux/centos
```

**粗体**：表示新术语、重要单词或屏幕上显示的单词。例如，菜单或对话框中的单词会像这样出现在文本中。举个例子：“之后，‘开始安装’按钮将被启用。”

警告或重要说明以这种方式显示。

小贴士和技巧以这种方式出现。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：如果您对本书的任何部分有疑问，请在邮件主题中提及书名，并通过 `customercare@packtpub.com` 给我们发送邮件。

**勘误**：虽然我们已尽一切努力确保内容的准确性，但错误仍然会发生。如果您在本书中发现错误，我们将非常感激您能向我们报告。请访问 [www.packt.com/submit-errata](http://www.packt.com/submit-errata)，选择您的书籍，点击“勘误提交表单”链接，并填写详细信息。

**盗版**：如果您在互联网上发现我们作品的任何非法复制品，我们将非常感激您提供该位置地址或网站名称。请通过 `copyright@packt.com` 与我们联系，并附上相关材料的链接。

**如果您有兴趣成为作者**：如果您在某个主题方面有专长，并且有兴趣写作或参与书籍创作，请访问 [authors.packtpub.com](http://authors.packtpub.com/)。

# 评价

请留下评论。读完并使用本书后，为什么不在您购买本书的网站上留下评论呢？潜在读者可以查看并参考您公正的意见做出购买决策，我们 Packt 可以了解您对我们产品的看法，而我们的作者也能看到您对他们书籍的反馈。谢谢！

如需了解更多关于 Packt 的信息，请访问 [packt.com](http://www.packt.com/)。
