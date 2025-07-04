# 前言

最近有许多技术进步，真正改变了我们的生活、工作和娱乐方式。电视、电脑和手机都深刻地影响了我们的生活。每一项技术最初通常由一些早期采用者开始，他们大多是拥有大量资源、能够负担得起新技术的个人。然而，很快就出现了一股运动，旨在让这些技术变得更加平价，让更多人能够使用。

最新的技术趋势是机器人技术。机器人的数量、种类和应用正在快速增长。这些机器人的最初开发出现在大学实验室或军事研究中心。然而，就像计算机的普及一样，已经有一个不断壮大的草根运动，许多自制开发者涌现出来，致力于让机器人融入我们的日常生活。

这一运动得益于低价硬件和免费的开源软件。同时，它也得到了一个开发者社区的支持，许多人愿意帮助他人入门或克服他们曾遇到的挑战。

本书是以自制运动的精神提供的。在书中，你将找到如何将一台便宜、小巧但多功能的树莓派 B 2 计算机与低价硬件和开源软件结合，打造一个可以行走、感知障碍物，甚至能“看到”周围环境的双足机器人。

然而，要小心——这种信息可能是危险的。没多久，你可能就会创造出下一代的思考、行走、感知机器，它们将成为机器人革命的核心。

# 本书内容

第一章，*配置和编程树莓派*，从讨论如何连接电源开始，并通过设置一个完整的系统，配置好并准备好连接任何令人惊叹的设备和软件能力，来开发先进的机器人应用。

第二章，*构建双足机器人*，展示了如何构建双足平台的机械部分，无论是使用 3D 打印、购买，还是自己制作双足和身体。

第三章，*双足机器人运动*，讲述了一旦你建立好平台后，你需要编程让它行走、挥手、装死或执行任何其他有趣的动作，以便你能够协调平台的运动。

第四章，*利用传感器避障*，向你展示如何添加红外传感器，这样你就可以避免撞到障碍物。

第五章，*双足机器人路径规划*，介绍了如何规划双足机器人的运动。当你移动时，你会希望能从 A 点到 B 点。

第六章, *为双足机器人添加视觉*，提供了如何连接网络摄像头、硬件和软件的详细信息，以便我们可以将其用于将视觉数据输入到我们的系统中。

第七章，*远程访问您的双足机器人*，介绍了如何将 Raspberry Pi 配置为无线接入点，从而使您能够远程控制您的双足机器人。

# 本书所需的资源

这是您所需资源的清单：

+   Raspbian

+   putty

+   Windows 版 Image Writer

+   libusb-1.0-0-dev

+   VncServer

# 本书的读者群体

本书适合那些有一定 Raspberry Pi 背景并希望用其进行机器人项目创建的读者。在创建一个可以走路、感知环境、规划动作、并自主执行运动和颜色跟踪的双足机器人时，我们假设您有一定的编程基础。

# 约定

在本书中，您会看到一些文本样式，用于区分不同类型的信息。以下是这些样式的一些示例及其含义说明。

文本中的代码词汇、数据库表名、文件夹名称、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名显示如下：“但是，您确实需要找到您的卡的`/dev`设备标签。”

任何命令行输入或输出均按以下方式书写：

```
sudo dd if=2015-01-31-raspbian.img  of=/dev/sdX

```

**新术语** 和 **重要词汇** 用粗体显示。您在屏幕上看到的词汇，例如在菜单或对话框中，文本中会以如下方式呈现：“点击**下一步**按钮将带您进入下一屏。”

### 注意

警告或重要提示会以框的形式出现，如下所示。

### 小贴士

提示和技巧如下所示。

# 读者反馈

我们始终欢迎读者的反馈。请告诉我们您对本书的看法——您喜欢什么或不喜欢什么。读者反馈对我们非常重要，它帮助我们开发出您真正能从中受益的书籍。

若您想提供一般性反馈，只需通过电子邮件`<feedback@packtpub.com>`联系我们，并在邮件主题中注明书籍名称。

如果您在某个主题领域有专业知识，并且有兴趣撰写或参与书籍创作，请查看我们的作者指南：[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您已经是一本 Packt 书籍的骄傲拥有者，我们提供了一些帮助您充分利用购买资源的方式。

## 下载本书的彩色图片

我们还为您提供了一份包含本书中使用的截图/图表的彩色图片的 PDF 文件。这些彩色图片将帮助您更好地理解输出的变化。您可以从[`www.packtpub.com/sites/default/files/downloads/Raspberry_Pi_Robotics_Essentials_Graphics.pdf`](https://www.packtpub.com/sites/default/files/downloads/Raspberry_Pi_Robotics_Essentials_Graphics.pdf)下载此文件。

## 勘误

尽管我们已尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在我们的书籍中发现错误——无论是文本错误还是代码错误——我们将不胜感激您能向我们报告。这样，您不仅能帮助其他读者避免困惑，还能帮助我们改进后续版本的内容。如果您发现任何勘误，请访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)提交，选择您的书籍，点击**勘误提交表单**链接，并填写勘误详情。一旦您的勘误得到验证，我们将接受您的提交，并将其上传到我们的网站，或者添加到该书籍的勘误列表中。

要查看之前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索框中输入书名。所需信息将显示在**勘误**部分。

## 盗版

互联网盗版问题在所有媒体中普遍存在。我们在 Packt 公司非常重视版权和许可证的保护。如果您在互联网上发现任何形式的我们作品的非法复制品，请立即提供网址或网站名称，以便我们采取措施。

如果您发现涉嫌盗版的资料，请通过`<copyright@packtpub.com>`与我们联系，并提供相关链接。

我们感谢您为保护我们的作者和我们提供有价值内容的能力所作的帮助。

## 问题

如果您在本书的任何方面遇到问题，您可以通过`<questions@packtpub.com>`与我们联系，我们将尽力解决问题。
