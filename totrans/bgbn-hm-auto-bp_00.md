# 序言

BeagleBone Black 是一款嵌入式系统，能够运行嵌入式 GNU/Linux 发行版以及像 Debian 或 Ubuntu 这样的普通（和强大的）发行版，并且用户可以通过两个专用扩展连接器连接多个外部外设。

因为它具有强大的分发能力，并且是一款易于扩展的嵌入式板，BeagleBone Black 系统是一款先进的设备，允许用户构建强大而多功能的监控和控制应用程序。

本书展示了几个家庭自动化原型，包括硬件和软件，以便向您解释如何使用连接了多个设备的 BeagleBone Black 来控制您的家。

每个原型都在其各自的硬件和软件章节中讨论，解释所有连接和管理多个外设所需的软件包。然后，详细解释了将所有内容粘合在一起的代码，直至每个项目的最终测试。

本书所用的硬件设备已经选择，以涵盖我们在使用 BeagleBone Black 开发板时可能遇到的所有连接类型，因此您会找到 I2C、SPI、USB、1-Wire、串行、数字和模拟设备。

本书选择的编程语言是根据找到解决当前问题的最快最简单方法的规则进行选择的；特别是，您可以在 Bash、C、PHP、Python、HTML 甚至 JavaScript 的示例代码中找到例子。

### 注意

警告！本书中所有项目均为原型，不能用作最终应用。

本书的作者和 Packt Publishing 都不建议或支持单独使用或作为任何应用程序组件的这些产品，除非进行必要的修改将这些原型转换为最终产品。

本书的作者和 Packt Publishing 不会对这些原型的未经授权使用负责。用户可以自行承担使用这些设备的硬件和软件的风险！

在我们需要使用守护程序或内核模块的章节中，或者我们需要重新编译整个内核的章节中，我已经添加了关于读者应该做什么以及他们可以在哪里获取关于所使用工具更多信息的简短描述；然而，一些管理 GNU/Linux 系统、内核模块或内核本身的基本技能是必需的（读者可以查看由本书作者编写的书籍《BeagleBone Essentials》，Packt Publishing，以获取有关这些主题的更多信息）。

# 本书内容概述

第一章, *危险气体传感器*，将展示如何使用 BeagleBone Black 监控房间内的危险气体，如一氧化碳、甲烷、液化石油气等，并在危险情况下启用声光报警系统。此外，通过使用 GSM 模块，用户可以向预定的电话号码发送短信，提醒例如亲戚等。

第二章, *超声波停车助手*，将展示如何使用 BeagleBone Black 实现一个停车助手。我们将使用超声波传感器检测汽车与车库墙壁之间的距离，并使用 LED 灯向驾驶员反馈汽车的位置，避免碰撞。

第三章, *水族箱监控器*，将展示如何制作一个水族箱监控器，借此我们可以通过 PC、智能手机或平板电脑上的网页面板记录所有环境数据，并控制我们心爱的鱼类的生活。

第四章, *Google Docs 气象站*，将介绍一个简单的气象站，该气象站也可以作为物联网设备使用。这一次，我们的 BeagleBone Black 将收集环境数据并将其发送到远程数据库（Google Docs 电子表格），以便重新处理并呈现在共享环境中。

第五章, *WhatsApp 洗衣房监控器*，将展示一个洗衣房监控器的实现，该监控器配有多个传感器，能够在特定事件发生时直接通过 WhatsApp 帐户提醒用户。

第六章, *婴儿房警卫*，将展示如何实现一个能够监控婴儿房的婴儿房警卫，检测婴儿是否在哭泣，或者婴儿在睡觉时是否有呼吸。此外，作为一项特殊功能，该系统还能够使用无接触温度传感器测量婴儿的体温。

第七章, *Facebook 植物监控器*，将展示如何实现一个植物监控器，该监控器能够测量光照、土壤湿度和土壤温度（包括土壤内外），并通过网络摄像头在特定时间间隔拍摄一些照片，然后将其发布到 Facebook 时间线上。

第八章, *入侵检测系统*，将展示如何利用 BeagleBone Black 和两个（或更多）网络摄像头实现一个低成本、合理质量的入侵检测系统。该系统将能够通过发送带有入侵者照片的电子邮件来提醒用户。

第九章，*带智能卡和 RFID 的 Twitter 访问控制系统*，将展示如何使用智能卡读卡器以及两种 RFID 读卡器（低频和超高频），以展示不同的方法来实现一个最简单的身份识别系统，用于访问控制，并向 Twitter 账户发送警报信息。

第十章，*使用电视遥控器的灯光管理器*，将展示如何通过电视遥控器或任何红外设备来管理连接到 BeagleBone Black 的简单开关设备。

第十一章，*使用 Z-Wave 的无线家庭控制器*，将展示如何通过连接到 BeagleBone Black 的 Z-Wave 控制器以及两个 Z-Wave 设备（一个墙壁插座和一个多功能传感器设备）来实现一个小型的无线家庭控制器。

# 本书所需内容

为了从本书中获得最大收益，您需要具备以下先决条件。

## 软件先决条件

关于软件，读者应对非图形文本编辑器（如 vi、emacs 或 nano）有一些基本了解。即使读者能够将 LCD 显示器、键盘和鼠标直接连接到 BeagleBone Black，并使用图形界面，本书仍假设读者能够通过仅支持文本的编辑器对文本文件进行一些修改。

主机计算机，即读者用来交叉编译代码和/或管理 BeagleBone Black 的计算机，假定运行的是基于 GNU/Linux 的发行版。我的主机 PC 运行的是 Ubuntu 14.04 LTS，但读者也可以使用基于 Debian 的系统，稍作修改后即可使用，或者使用其他 GNU/Linux 发行版，但需要进行一些努力，主要是为了安装交叉编译工具。由于我们不应使用低技术系统来为高技术系统开发代码，因此本书**不**涉及 Windows、Mac OS 或类似系统！

了解 C 编译器如何工作以及如何管理 `Makefile` 可能会有所帮助，但不用担心，所有示例都从最基础开始，即使是没有经验的开发者也能完成任务。

本书将介绍一些内核编程技巧，但不应将其视为 *内核编程课程*。对于这种主题，您需要一本专门的书籍！然而，每个示例都有很好的文档说明，读者将找到若干建议的资源。

关于内核，我想说明一下，默认情况下我使用的是板载内核，即版本 3.8.13。然而，在某些章节中，我使用了自编译的内核，版本 3.13.11；在这种情况下，我会提供一个小教程，教读者如何完成这项工作。

如果你使用的是较新的内核版本，可能会遇到一些小问题，但你应该能够毫无问题地移植我所做的工作。如果你使用的是非常新的内核，请注意，cape 管理器文件 `/sys/devices/bone_capemgr.9/slots` 已经被移到 `/sys/devices/platform/bone_capemgr/slots`，因此你需要相应地更改所有相关命令。

最后需要说明的是，我假设读者知道如何将 BeagleBone Black 板连接到互联网，以便下载包或通用文件。

## 硬件先决条件

本书中的所有代码都是为 BeagleBone Black 板的 *C* 版本开发的，但读者也可以使用较旧的版本，且不会遇到任何问题；事实上，代码是可移植的，应该也能在其他系统上运行！

关于本书中使用的计算机外设，我在每章中都有注明硬件的来源和购买地点，当然，读者也可以选择上网寻找更好、更便宜的购买选择。还会有一条说明，告诉读者哪里可以找到数据手册。

读者在将本书中介绍的硬件连接到 BeagleBone Black 时应该不会遇到任何困难，因为连接非常少且文档齐全。读者无需具备特殊的硬件技能（只需知道如何使用烙铁）；然而，具备一定的电子学知识会有所帮助。

在阅读过程中，我将提到 BeagleBone Black 的引脚，尤其是扩展连接器上的引脚。所有使用的引脚都会有说明，但是如果需要，你可以在 [`elinux.org/Beagleboard:Cape_Expansion_Headers`](http://elinux.org/Beagleboard:Cape_Expansion_Headers) 上找到 BeagleBone Black 连接器的完整描述。

# 本书的目标读者

如果你是一位开发者，想要学习如何使用嵌入式机器学习功能，并获取访问 GNU/Linux 设备驱动程序的权限以收集外设数据或控制设备，那么这本书就是为你准备的。

如果你希望通过实施不同种类的智能家居设备来管理你的家庭，这些设备可以与智能手机、平板电脑或 PC 进行交互，或者如果你只是独立工作，具备一定的硬件或电气工程经验，那么本书适合你。你还需要了解 C、Bash、Python、PHP 和 JavaScript 编程基础，尤其是在 UNIX 环境下的编程。

# 约定

本书中有许多不同的文本样式，用以区分不同类型的信息。以下是一些样式示例及其含义的解释。

## 代码和命令行

文本中的代码词、数据库表名、文件夹名称、文件名、文件扩展名、路径名、虚拟网址、用户输入以及 Twitter 用户名均如下所示：“为了获取之前的内核消息，我们可以使用 `dmesg` 和 `tail -f /var/log/kern.log` 命令。”

代码块如下所示：

```
CREATE TABLE status (
   n VARCHAR(64) NOT NULL,
   v VARCHAR(64) NOT NULL,
   PRIMARY KEY (n)
) ENGINE=MEMORY;
```

所有在 BeagleBone Black 上给出的命令行输入或输出如下所示：

```
root@beaglebone:~# make CFLAGS="-Wall -O2" helloworldcc -Wall -O2 helloworld.c -o helloworld

```

任何在我的主机计算机上作为*非特权用户*给出的命令行输入或输出都会写成如下：

```
$ tail -f /var/log/kern.log

```

当我需要以特权用户（root）的身份在我的主机计算机上给出命令时，命令行的输入或输出将写成如下：

```
# /etc/init.d/apache2 restart

```

读者应该注意，所有特权命令也可以通过正常用户使用`sudo`命令来执行，格式如下：

```
$ sudo <command>

```

因此，前面的命令可以由普通用户执行，如下所示：

```
$ sudo /etc/init.d/apache2 restart

```

## 内核和日志信息

在多个 GNU/Linux 发行版中，内核信息具有以下形式：

```
Oct 27 10:41:56 hulk kernel: [46692.664196] usb 2-1.1: new high-speed USB device number 12 using ehci-pci

```

这对于本书来说是一个相当长的行，因此从行的开头直到实际信息开始的部分都会被省略。所以，在前面的示例中，这行的输出将被报告如下：

```
usb 2-1.1: new high-speed USB device number 12 using ehci-pci

```

长输出和终端中的重复或不太重要的行会被替换为三个点（...），如下所示：

```
output begin
output line 1
output line 2
...
output line 10
output end

```

## 文件修改

当读者需要修改文本文件时，我将使用统一的上下文`diff`格式，因为这是表示文本修改的非常高效和紧凑的方式。该格式可以通过使用`diff`命令并添加`-u`选项参数来获得。

作为一个简单的例子，假设我们考虑`file1.old`中的以下文本：

```
This is first line
This is the second line
This is the third line
...
...
This is the last line
```

假设我们需要修改第三行，如下片段所示：

```
This is first line
This is the second line
This is the new third line modified by me
...
...
This is the last line
```

读者可以轻松理解，每次对一个简单的修改都报告整个文件是相当晦涩且浪费空间的；然而，通过使用统一的上下文`diff`格式，前面的修改可以写成如下：

```
$ diff -u file1.old file1.new
--- file1.old 2015-03-23 14:49:04.354377460 +0100
+++ file1.new 2015-03-23 14:51:57.450373836 +0100
@@ -1,6 +1,6 @@
 This is first line
 This is the second line
-This is the third line
+This is the new third line modified by me
 ...
 ...
 This is the last line
```

现在，修改非常清晰，并且以紧凑的形式写出！它以一个两行的标题开始，原文件前面是`---`，新文件前面是`+++`，然后是一个或多个变化块，包含文件中的行差异。前面的示例只有一个变化块，其中未改变的行前面有一个空格字符，而要添加的行前面有一个`+`字符，要删除的行前面有一个`-`字符。

## 串行和网络连接

在本书中，我将主要使用两种不同的方式与 BeagleBone Black 开发板进行交互：串行控制台和 SSH 终端。前者可以通过连接器 J1 直接访问（在本书中从未使用）或通过与用于供电的同一 USB 连接进行模拟访问，而后者则可以通过上面的 USB 连接或通过以太网连接来使用。

串行控制台主要用于从命令行管理系统。它在很大程度上用于监控系统，特别是用于控制内核消息。

SSH 终端与串行控制台非常相似，尽管它并不完全相同（例如，内核信息不会自动出现在终端上）；然而，它可以像串行控制台一样使用，执行命令并从命令行编辑文件。

在接下来的章节中，我将通过串行控制台或 SSH 连接不加区分地使用终端，给出实现本书中所有原型所需的大多数命令和配置设置。

要从主机 PC 访问 USB 仿真串行控制台，可以使用以下`minicon`命令：

```
$ minicom -o -D /dev/ttyACM0 

```

请注意，在某些系统上，你可能需要 root 权限才能访问`/dev/ttyACM0`设备（在这种情况下，你可以使用`sudo`命令来覆盖它）。

如上所述，要访问 SSH 终端，你可以通过与串行控制台相同的 USB 电缆使用仿真以太网连接。实际上，如果你的主机 PC 配置正确，当你插入 USB 电缆为 BeagleBone Black 开发板供电时，过一会儿，你应该会看到一个新连接，IP 地址为`192.168.7.1`。然后，你可以使用这个新连接通过以下命令访问 BeagleBone Black：

```
$ ssh root@192.168.7.2 

```

最后一个可用的通信通道是以太网连接。它主要用于从主机 PC 或互联网下载文件，可以通过将以太网电缆连接到 BeagleBone Black 的以太网端口并根据你的局域网设置配置端口来建立连接。

然而，需要特别指出的是，你还可以通过之前提到的仿真以太网连接来连接互联网。实际上，通过在主机 PC（显然是基于 GNU/Linux 的）上使用以下命令，你将能够将其用作路由器，让你的 BeagleBone Black 开发板像连接到真实以太网端口一样上网：

```
# iptables --table nat --append POSTROUTING --out- interface eth1 -j MASQUERADE
# iptables --append FORWARD --in-interface eth4 -j ACCEPT
# echo 1 >> /proc/sys/net/ipv4/ip_forward

```

然后，在 BeagleBone Black 上，我们应该通过 USB 电缆使用以下命令设置网关：

```
root@beaglebone:~# route add default gw 192.168.7.1

```

请注意，`eth1`设备是我主机系统首选的互联网连接，而`eth4`设备是 BeagleBone Black 在我主机系统上显示的设备，因此你需要相应修改命令以满足你的需求。

## 其他约定

**新术语**和**重要单词**以粗体显示。在屏幕上看到的单词，例如在菜单或对话框中，文本中会这样显示：“点击**下一步**按钮将你带到下一个屏幕。”

### 注意

警告或重要说明会出现在像这样的框中。

### 提示

提示和技巧会以这种方式显示。

# 读者反馈

我们欢迎读者的反馈。请告诉我们你对本书的看法——你喜欢或不喜欢什么。读者反馈对我们非常重要，因为它帮助我们开发出你真正能从中受益的书籍。

要向我们发送一般反馈，请简单地发送电子邮件至`<feedback@packtpub.com>`，并在邮件主题中提及书名。

如果您在某个领域有专业知识，并且有兴趣编写或为书籍做贡献，请查看我们的作者指南：[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

既然您已经成为 Packt 书籍的骄傲拥有者，我们提供了许多帮助您充分利用购买内容的资源。

## 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)的账户中下载所有您购买的 Packt 出版书籍的示例代码文件。如果您在其他地方购买了本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)，并注册以便直接通过电子邮件收到文件。

本书的示例代码也可以从作者的 GitHub 库中下载，地址为[`github.com/giometti/beaglebone_home_automation_blueprints`](https://github.com/giometti/beaglebone_home_automation_blueprints)。

只需使用以下命令一次性获取它：

```
$ git clone https://github.com/giometti/beaglebone_home_automation_blueprints

```

示例按章节名称分组，因此您可以在阅读书籍时轻松找到代码。

## 下载本书的彩色图像

我们还为您提供了一份 PDF 文件，其中包含本书中使用的截图/图表的彩色图像。这些彩色图像将帮助您更好地理解输出的变化。您可以从[`www.packtpub.com/sites/default/files/downloads/BeagleBoneHomeAutomationBlueprints_ColoredImages.pdf`](http://www.packtpub.com/sites/default/files/downloads/BeagleBoneHomeAutomationBlueprints_ColoredImages.pdf)下载此文件。

## 勘误

虽然我们已尽一切努力确保内容的准确性，但错误难免发生。如果您在我们的书籍中发现错误——可能是文本或代码中的错误——我们将非常感激您向我们报告。通过这样做，您可以帮助其他读者避免困扰，并帮助我们改进后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)报告，选择您的书籍，点击**Errata Submission Form**链接，并输入您的勘误详情。您的勘误一旦验证后，将被接受，并上传到我们的网站或添加到该书籍标题下的现有勘误列表中。

要查看先前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索框中输入书名。所需信息将显示在**Errata**部分。

## 盗版

网络上的版权盗版问题在所有媒体中普遍存在。Packt 公司非常重视我们版权和许可证的保护。如果您在互联网上遇到我们作品的非法复制品，无论形式如何，请立即向我们提供位置地址或网站名称，以便我们采取相应的补救措施。

如发现涉嫌盗版的资料，请通过`<copyright@packtpub.com>`与我们联系，并提供相关链接。

感谢您在保护我们作者权益和帮助我们提供有价值内容方面的支持。

## 问题

如果您对本书的任何方面有问题，可以通过`<questions@packtpub.com>`与我们联系，我们将尽力解决问题。
