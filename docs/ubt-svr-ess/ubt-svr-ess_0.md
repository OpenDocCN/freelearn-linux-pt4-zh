# 序言

《Ubuntu 服务器基础》是一本实用的操作指南，提供了清晰的逐步过程来安装和管理 Ubuntu 服务器，帮助你充分发挥 Ubuntu 的真正力量，而无需成为该领域的专家。

本书节奏较快，适合那些希望了解最新版本 Ubuntu 服务器基础知识的管理员。本书的目的是引导读者，使其能够在办公环境中部署和配置 Ubuntu 服务器。首先，我们将从解释如何安装 Ubuntu 服务器开始。接着，我们将介绍随 Ubuntu 一起提供的命令行界面最有用的方面。与此同时，我们将探讨如何管理和配置 Ubuntu 服务器。关于这一主题，我们会有一章进行扩展。之后，我们将讨论如何在 Ubuntu 服务器上部署服务并进行安全配置。在结尾部分，我们将了解 Ubuntu 提供的虚拟化和云计算功能。最后，我们将探索一些与 Ubuntu 服务器相关的实用小贴士。

# 本书内容

第一章，《*Ubuntu 服务器安装*》作为 Ubuntu 服务器安装指南。

第二章，《*配置与管理 Ubuntu 服务器*》提供了管理 Ubuntu 服务器所需的知识和工具。

第三章，《*在 Ubuntu 上部署服务器*》让用户能够轻松设置和部署一系列常用服务，如电子邮件、网页、DNS 等。

第四章，《*Ubuntu 安全设置*》作为 Ubuntu 服务器的安全指南。

第五章，《*Ubuntu 服务器中的虚拟化与云计算*》提供了有关虚拟化和云计算所需的知识。

第六章，《*Ubuntu 服务器的技巧与窍门*》包含了每个 Ubuntu 管理员需要掌握的一些最实用的技巧和窍门。

# 本书所需工具

使用本书，你只需要基本的 Linux 操作系统知识和一杯咖啡。

# 本书适用对象

本书适用于熟悉 Linux 操作系统基础的系统管理员，且寻求一本快速入门 Ubuntu 的指南。熟悉旧版本 Ubuntu 的读者也会发现本书很有用。本书假设读者具有基本的 Linux 管理知识。通过本书，读者将能够很好地掌握最新版本 Ubuntu 的使用，并探索 Ubuntu 服务器管理的新特性。

# 术语规范

本书中，你会看到多种文本样式，用以区分不同种类的信息。以下是这些样式的几个示例及其含义的解释。

文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟网址、用户输入和 Twitter 用户名将按以下格式显示：“我们可以通过使用 `include` 指令包含其他上下文。”

代码块按以下方式设置：

```
zone "localhost" {
  type master;
  file "/etc/bind/db.local";
  allow-transfer { 192.168.1.2; };
  also-notify { 192.168.1.2; };
};
zone "127.in-addr.arpa" {
  type master;
  file "/etc/bind/db.127";
  allow-transfer { 192.168.1.2; };
  also-notify { 192.168.1.2; };
};
```

任何命令行输入或输出会按以下方式展示：

```
sudo ln -s /etc/phppgadmin/apache.conf /etc/apache2/sites-available/phppgadmin
sudo a2ensite phppgadmin
sudo apache2ctl graceful

```

**新术语** 和 **重要词汇** 会以粗体显示。你在屏幕上看到的词汇，比如菜单或对话框中的内容，会以这样的方式出现在文本中：“然后，进入机器的**设置**，选择**系统**选项卡。”

### 注意

警告或重要提示会以这样的框框显示。

### 提示

提示和技巧会以这样的方式展示。

# 读者反馈

我们始终欢迎读者的反馈。告诉我们你对这本书的看法——喜欢什么或不喜欢什么。读者反馈对我们非常重要，因为它帮助我们开发出你真正能从中获得最大价值的书籍。

要向我们发送一般反馈，只需通过电子邮件 `<feedback@packtpub.com>`，并在邮件主题中提到书籍标题。

如果你在某个主题上有专业知识，并且有兴趣撰写或参与书籍的贡献，请查看我们的作者指南，网址是 [www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在你已经是一本 Packt 书籍的骄傲拥有者，我们提供了许多帮助你充分利用购买的内容。

## 下载示例代码

你可以从你的账户中下载所有你购买的 Packt 出版社书籍的示例代码文件，访问 [`www.packtpub.com`](http://www.packtpub.com)。如果你是在其他地方购买的这本书，可以访问 [`www.packtpub.com/support`](http://www.packtpub.com/support) 并注册以便将文件通过电子邮件直接发送给你。

## 下载本书的彩色图片

我们还为你提供了一个 PDF 文件，里面包含了本书中使用的截图/图表的彩色图片。这些彩色图片将帮助你更好地理解输出结果的变化。你可以从 [`www.packtpub.com/sites/default/files/downloads/4800OS_ColoredImages.pdf`](http://www.packtpub.com/sites/default/files/downloads/4800OS_ColoredImages.pdf) 下载该文件。

## 勘误

虽然我们已尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在我们的书籍中发现错误——可能是文本或代码中的错误——我们将非常感激您能向我们报告。这样，您可以帮助其他读者避免困惑，并帮助我们改进后续版本的书籍。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)进行报告，选择您的书籍，点击**勘误提交表单**链接，并输入勘误的详细信息。一旦您的勘误被验证，您的提交将被接受，勘误将上传到我们的网站，或者添加到该书籍勘误部分的现有勘误列表中。

要查看以前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索框中输入书名。所需信息将显示在**勘误**部分。

## 盗版

互联网对受版权保护材料的盗版问题在所有媒体中持续存在。在 Packt，我们非常重视版权和许可证的保护。如果您在互联网上发现我们作品的任何非法复制，请立即提供其位置地址或网站名称，以便我们采取补救措施。

请通过`<copyright@packtpub.com>`联系我们，并提供涉嫌盗版材料的链接。

感谢您帮助保护我们的作者以及我们为您提供宝贵内容的能力。

## 问题

如果您对本书的任何方面有问题，可以通过`<questions@packtpub.com>`联系我们，我们将尽力解决问题。
