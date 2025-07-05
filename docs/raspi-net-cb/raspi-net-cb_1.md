# 第一章 安装与设置

在本章中，我们将涵盖以下主题：

+   准备初次启动

+   下载新 SD 卡

+   使用 NOOBS 启动

+   Mac OS X 磁盘工具 – diskutil 和 dd

+   Windows 图像写入工具 – Win32DiskImager.exe

+   转换并复制到 Linux – dd

+   第一次启动 Raspbian Linux

+   关闭树莓派

# 介绍

本章介绍了树莓派。它首先列出了除了树莓派本身外，你还需要的组件，比如电源。

本章的核心教程描述了如何下载、安装和配置一些常见的树莓派操作系统。

最后两个教程描述了官方 Raspbian Linux 发行版的初次启动以及如何安全关机。

完成本章后，你将能够为你的树莓派下载、安装并配置操作系统，并首次启动树莓派。

# 准备初次启动

本教程解释了在首次开机前，除了树莓派外，还需要哪些组件才能启动。

树莓派基金会自 2012 年 6 月首次发布以来，发布了多个版本的树莓派，包括：树莓派 B（2012 年 4 月）、树莓派 A（2013 年 2 月）、树莓派计算模块（2014 年 4 月）、树莓派 B+（2014 年 7 月）以及树莓派 2 型号 B（2015 年 2 月）。

原始的树莓派型号 B 内存仅为 512MB，采用单核处理器和两个 USB 端口。而当前型号树莓派 2 型号 B 则配备了 1GB 内存、四核处理器和四个 USB 端口。

### 注意

本书中的示例以树莓派 2 型号 B 为例。

树莓派出厂时没有外壳和电源适配器，也没有键盘或显示器。根据你打算如何使用树莓派，你可能需要额外的组件。本书大多数教程仅需要电源、SD 卡和网线。

根据你打算如何使用树莓派，你可能希望连接其他外设。如果你打算将树莓派当作桌面电脑使用，则需要 HDMI 线、USB 键盘和 USB 鼠标。本教程列出了多个树莓派项目及其所需的外设。

完成本教程后，你将准备好进行树莓派的初次启动。

## 准备工作

为了开始本教程，你需要熟悉一些前提知识。

### 基本组件

以下是一些基本组件：

+   树莓派

+   一张 SD 卡

+   一个 5V 的 Micro USB 电源

树莓派通过一个 5V 的 Micro USB 电源供电，并且需要一张 SD 卡作为操作系统存储。尽管启动树莓派时不需要额外的组件，但本书中的许多网络解决方案会要求使用额外的硬件。

一张单一的 4GB SD Class 10 卡拥有足够的空间和速度来存储基础操作系统以及许多有用的应用程序。因为 SD 卡是树莓派存储操作系统的地方，所以操作系统的速度取决于 SD 卡的速度。Class 10 卡的性能要优于 Class 4 或 Class 6 卡。嵌入式 Linux Wiki 维护了一张 SD 卡兼容性表格，可以通过访问[`elinux.org/RPi_SD_cards`](http://elinux.org/RPi_SD_cards)来查看。

除了 SD 卡，树莓派还需要额外的组件来支持许多应用程序。对于本书中的大多数食谱，您只需要一个网络连接。对于某些应用，您可能还需要一个显示器以及键盘和鼠标。

以下是一些网络应用的示例及其所需的组件。

### 基本网络

基本网络指的是拥有网络连接。对于最简单的网络解决方案，树莓派所需的唯一附加组件就是网络连接——无论是直接的 TCP 电缆连接，还是无线网络 USB 加密狗。一旦网络配置完成，并且可以远程登录树莓派，就可以远程访问、更新和管理树莓派。

### 媒体中心

作为媒体中心的一部分，我们需要一个 HDMI 电视或显示器。

对于最简单的网络媒体解决方案，除了基本的网络组件，树莓派所需的唯一附加组件是 HDMI 连接。音频和视频都可以通过树莓派的 HDMI 连接进行流媒体播放。此外，SD 卡上有足够的空间来存储一小部分音乐和视频文件，以及操作系统。

### 台式计算机

如果您使用的是台式计算机，以下设备是必需的：

+   一台 HDMI 电视或显示器

+   一个 USB 键盘

+   一个 USB 鼠标

树莓派 2 拥有四个 USB 端口，能够提供足够的电力支持低功耗设备，如 USB 键盘或 USB 鼠标。它配备四核处理器，足以浏览网页、发送电子邮件、编辑文档或图像。由于运行 Linux 操作系统，树莓派还可以运行数百种教育、科学和商业程序。简而言之，树莓派可以运行许多有用的开源桌面应用程序。

### 网络集线器

对于网络集线器，我们需要以下设备：

+   一个带电 USB 集线器

+   一个 USB LAN 适配器

+   一个 USB WLAN 适配器（Wi-Fi 加密狗）

+   一个 USB 硬盘

+   一个 USB 打印机

当使用树莓派作为防火墙或无线接入点时，需要额外的 LAN 或 WLAN 网络适配器。如果网络适配器通过 USB 连接供电，还需要一个额外的带电 USB 连接器以确保适配器的可靠运行。

### 游戏主机

+   一个带电 USB 集线器

+   USB 游戏控制器

Raspberry Pi 是一个出色的游戏平台，如果你希望创建游戏，或玩单人控制台游戏或多人网络游戏。许多较老的基于文本的游戏可以仅通过键盘或远程登录在 Raspberry Pi 上运行。然而，也可以连接 USB 游戏控制器以进一步丰富多媒体动作游戏的游戏体验。

### 初始设置

+   Raspberry Pi

+   一张 SD 卡

+   一款 5V 微型 USB 电源适配器

+   一条网络连接

+   一台 HDMI 电视或显示器

+   一只 USB 键盘

+   一只 USB 鼠标

电源适配器、预格式化的 SD 卡、显示器、键盘和鼠标是初始设置所需的最基本组件。当通过 HDMI 连接电视时，电视将同时输出音频和视频。

## 如何操作…

执行以下步骤以启动 Raspberry Pi：

1.  下载最新的磁盘映像。

1.  将磁盘映像写入 SD 卡。

1.  将格式化的 SD 卡插入 Raspberry Pi。

1.  将显示器连接到 HDMI 接口。

1.  将 USB 板和 USB 鼠标连接到 USB 端口。

1.  将 5V 微型 USB 电源连接到 Raspberry Pi，设备便会启动。

1.  最后，关闭 Raspberry Pi。

## 它是如何工作的…

在你能够启动 Raspberry Pi 之前，首先需要一张带有可启动磁盘映像的 SD 卡。官方的 Raspbian Linux 映像可以从 [`www.raspberrypi.org/downloads`](http://www.raspberrypi.org/downloads) 下载。

下载磁盘映像后，需要将其写入 SD 卡（请参阅*设置新 SD 卡*的操作步骤）。

在 SD 卡准备好并插入 Raspberry Pi 后，可以连接显示器、键盘和鼠标。然后它就可以启动了（请参阅“启动 Raspbian Linux”的操作步骤）。

最后连接电源！Raspberry Pi 没有开关。当电源连接时，Raspberry Pi 会立即启动。因此，在连接电源之前，确保所有电缆都已连接，并且 SD 卡已插入。

当需要关闭 Raspberry Pi 时，必须先关闭操作系统，这与启动相反（请参阅*关闭 Raspberry Pi*的操作步骤）。

## 还有更多…

Raspberry Pi 2 是一款低成本的单板计算机（仅售 $35）。它是裸板出售的，使用之前需要电源、一个预格式化的 SD 卡来存储操作系统、键盘和显示器。然而，它确实拥有多种标准 I/O 接口和板载组件，可以使其连接到各种设备。

### 接口

Raspberry Pi 的标准连接器和接口如下：

+   **电源**（5V，800 mA（4.0 W））：Raspberry Pi 配有一个 Micro USB 电源连接口，应直接连接到电源适配器，而不是电脑的 USB 端口或 USB 集线器。

+   **SD 卡**：树莓派设计为从预格式化的 SD 卡启动（建议使用 4GB 或更大容量的 SD 卡；Class 10 SD 卡提供最佳性能）。

+   **GPIO**：用于模拟和数字输入/输出连接，用于扩展和实验。

+   **音频输出**（3.5 毫米接口——立体声）：树莓派没有音频输入接口。不过，可以添加 USB 麦克风或声卡。音频输出支持 I2S 协议，可以连接数字音频设备。

+   **LED 指示灯**：这些指示灯用于显示磁盘、电源和网络流量的状态。当这些 LED 灯闪烁时，树莓派正在积极处理数据。关机后，等待 LED 灯停止闪烁再拔掉树莓派。

+   **USB 2.0**（四个端口）：这些端口的电力有限。通过 USB 连接到树莓派的设备应当具有自己的电源供应，或者通过带电 USB 集线器连接。

+   **网络**（10/100 有线以太网 RJ45）：请注意，板载网络与连接的 USB 设备竞争带宽。

+   **HDMI**（版本 1.3 和 1.4）：可以用于视频和音频输出。支持分辨率从 640x350 到 1920x1200，包括 PAL 和 NTSC 标准。

### 板载组件

树莓派的主要板载组件如下：

+   **SoC**：即系统单芯片。我们需要的是 Broadcom BCM2836 媒体处理器。其特点如下：

    +   **CPU**：900MHz 四核 ARM Cortex-A7

    +   **GPU**：24 GFLOPS 计算能力

    +   **内存**：1GB SDRAM

+   **LAN9512**

    +   10/100 MB 以太网（自动 MDIX）

    +   4x USB 2.0

### 推荐附件

除了电源外，建议配备以下附件：

+   **机箱**：这是树莓派的保护外壳。

+   **带电 USB 集线器**：该集线器有自己的电源供应，独立于树莓派电源。它提供足够的电力支持连接的设备。

### 电源问题

很难确定树莓派实际需要多少电力，因为所需电量取决于树莓派的工作负载和连接的外设。然而，已经有报告提到电力不足导致的一些问题。当树莓派的电源能够提供至少 5V、800mA 的电流，并且 USB 设备通过带电 USB 集线器间接连接时，这些问题会减少或消失。

#### 症状

以下是一些症状：

+   屏幕右上角出现彩虹方块

+   网络连接不稳定

+   启动桌面 GUI 后，键盘无法使用

+   间歇性 SD 卡错误

#### 原因

以下是导致问题的原因：

+   电源供应额定值低于 800mA

+   复杂键盘或带有内置 USB 集线器的键盘，例如 Apple Macintosh 键盘

+   USB 硬盘或超大 U 盘直接连接到树莓派，而不是通过一个带电的 USB 集线器间接连接

#### 解决方案

以下是解决方案：

+   使用至少 700mA、5V 的稳压电源

+   仅将简单的 USB 设备直接连接到 Raspberry Pi

+   将 USB 设备连接到带电的 USB 集线器，并仅将集线器直接连接到 Raspberry Pi

## 另见

+   **Wikipedia—Raspberry Pi** ([`en.wikipedia.org/wiki/Raspberry_Pi`](http://en.wikipedia.org/wiki/Raspberry_Pi)): 这篇关于 Raspberry Pi 的维基百科文章包括了所有 Raspberry Pi 型号的对比、每个 Raspberry Pi 组件的详细信息，以及 Raspberry Pi 的扩展历史。

+   **MagPi** ([`www.raspberrypi.org/magpi`](http://www.raspberrypi.org/magpi)): MagPi 是 Raspberry Pi 的官方杂志，每月期刊可以在线查看。

+   **Raspberry Pi 官方网站** ([`www.raspberrypi.org`](http://www.raspberrypi.org)): Raspberry Pi 的官方网站提供了历史、新闻以及 Raspberry Pi 的文档，还包含了快速入门指南、论坛、维基和下载区域。

+   **R-Pi Hub— eLinux.org** ([`elinux.org/R-Pi_Hub`](http://elinux.org/R-Pi_Hub)): R-Pi Hub 是一个为 Raspberry Pi 用户提供的嵌入式 Linux 社区维基页面。该页面包含了购买指南、初学者指南、验证过的外设列表以及比官方网站上更多的 Raspberry Pi 发行版列表。它拥有丰富且井然有序、更新及时的信息。

+   **Raspberry Pi 硬件历史** ([`elinux.org/RPi_HardwareHistory`](http://elinux.org/RPi_HardwareHistory)): 嵌入式 Linux 社区已经记录了 Raspberry Pi 的硬件历史，包括每个版本的详细规格和图像。

# 下载新的 SD 卡

以下步骤说明了如何使用 `Win32DiskImager.exe`、`dd` 和 `diskutil` 从下载的磁盘镜像创建可启动的 SD 卡。

Raspberry Pi 没有预装操作系统。在 Raspberry Pi 启动之前，需要一张安装了操作系统的 SD 卡。可以购买预装操作系统的 SD 卡。然而，下载并安装操作系统镜像并不困难。

完成此步骤后，你将知道如何下载 Raspberry Pi 操作系统。接下来的步骤将展示如何将操作系统写入 SD 卡。

## 如何操作……

执行以下步骤将镜像写入 SD 卡：

1.  下载 Raspberry Pi 镜像。

1.  将镜像写入 SD 卡。

## 它是如何工作的……

最简单的入门方式是从 Raspberry Pi 基金会官网下载 **NOOBS**（**New Out Of Box Software**）发行版，可以通过访问 [`www.raspberrypi.org/downloads`](http://www.raspberrypi.org/downloads) 查看。该发行版的文件可以直接复制到格式化的 SD 卡中。无需额外的磁盘工具即可创建可启动镜像（参考 *Booting with NOOBS* 步骤）。

NOOBS 附带了树莓派基金会推荐的操作系统分发版——Raspbian Linux。在下载页面，你还可以找到 Raspbian Linux 镜像的链接。还有许多其他第三方操作系统的链接。想要获取更多树莓派镜像，请访问嵌入式 Linux 社区的 wiki 页面 ([`elinux.org/RPi_Distributions`](http://elinux.org/RPi_Distributions))。

与 NOOBS 不同，一旦下载了这些单独的操作系统镜像，你需要使用特定的磁盘工具将其写入 SD 卡。

如果你使用的是 Mac OS 操作系统，使用 `diskutil` 和 `dd` 将操作系统镜像写入 SD 卡（请参阅 Mac OS 磁盘工具配方）。如果你是在 Windows 计算机上写入 SD 卡，请使用 `Win32DiskImager.exe`（请参阅 *Windows 图像写入工具* 配方）。如果你使用的是 Linux 操作系统来将镜像写入 SD 卡，请使用 `dd` 命令行工具（请参阅 *Linux 转换与复制* 配方）。

## 另见

+   树莓派网站—下载页面 ([`www.raspberrypi.org/downloads`](http://www.raspberrypi.org/downloads)): 树莓派网站的下载页面是你可以找到推荐版本的树莓派操作系统分发版的地方。当前，树莓派基金会提供了以下操作系统分发版的链接：

    +   NOOBS 和 NOOBS Lite

    +   Raspbian（Jessie 和 Wheezy）

    +   Ubuntu Mate（Linux 桌面版）

    +   Snappy Ubuntu Core（开发者版分发）

    +   Windows 10 IoT Core（开发者版分发）

    +   开源媒体中心（OSMC）

    +   开源嵌入式 Linux 娱乐中心（OpenELEC）

    +   PINET（课堂版分发）

    +   RISC OS（非 Linux 分发版）

    Raspbian Linux 分发版是树莓派基金会推荐的操作系统分发版，也是本书中使用的操作系统分发版。

+   **树莓派的嵌入式 Linux 分发版** ([`elinux.org/RPi_Distributions`](http://elinux.org/RPi_Distributions)): 嵌入式 Linux 社区维护了一个关于树莓派操作系统分发版的优秀 wiki 页面。该页面提供了一个对比表格，并链接到可下载的镜像文件。许多这些分发版是针对特定用途而定制的，如渗透测试；作为家庭影院、防火墙或廉价桌面 PC 使用；或者用于软件开发。

+   **Windows 10 IoT—下载** ([`ms-iot.github.io/content/Downloads.htm`](https://ms-iot.github.io/content/Downloads.htm)): 用于 IoT 的 Windows 以及你开发 Windows IoT 设备（如树莓派）所需的其他工具可以在这个网站上找到。

    截至本修订版本，Windows 10 IoT 分发版没有用户界面。它被标记为 **Windows 10 IoT Core Insider Preview**。与 IoT Core 交互所需的工具可以从 Windows 10 IoT 的下载页面获取。

# 使用 NOOBS 启动

本步骤介绍了如何使用树莓派基金会的 NOOBS 工具安装树莓派操作系统。

NOOBS 不是一个操作系统发行版，而是一个用来安装操作系统的工具。通过使用 NOOBS，你可以为树莓派选择操作系统。

这是开始使用树莓派的最简单方式。无需特殊的磁盘工具，因此，只要你的计算机有 SD 卡写入功能，都可以使用这个方法。

完成这个过程后，你将能够使用 NOOBS 为树莓派选择操作系统。

## 准备工作

配料：

+   一台带有 SD 卡写入功能的计算机

+   为树莓派进行初始设置（参见*准备首次启动*的步骤）

+   格式化后的 SD 卡—4 GB 或更大（class 10 的速度最好）

+   NOOBS ZIP 文件

在 SD 卡上安装 NOOBS 并不依赖于特定的操作系统。

从树莓派官网（[`www.raspberrypi.org/downloads/noobs/`](https://www.raspberrypi.org/downloads/noobs/)）下载 NOOBS ZIP 文件（`NOOBS_v1_4_2.zip`）。

## 如何操作...

复制 NOOBS 到 SD 卡的步骤如下：

1.  将格式化后的 SD 卡插入计算机。

1.  将 NOOBS ZIP 文件解压到 SD 卡中。

1.  弹出 SD 卡。

1.  将 SD 卡插入树莓派并开启电源。

1.  选择并安装操作系统。

## 工作原理...

NOOBS 的安装过程与操作系统无关。NOOBS ZIP 文件中的文件仅需解压到一个全新格式化的 SD 卡上。

文件解压到 SD 卡后，SD 卡可以安全地从计算机中弹出并插入树莓派。

在将 SD 卡牢固地插入树莓派并连接所有其他组件（HDMI 显示器、网络连接、USB 键盘和 USB 鼠标）后，你可以连接电源并启动树莓派。确保电源在最后连接，否则树莓派将无法正常启动。

当树莓派完成 NOOBS 启动后，你将看到一系列操作系统供选择。按空格键或用鼠标点击列表顶部的**Raspbian [推荐]**。点击**安装 (i)**或按*I*键来安装 Raspbian Linux 操作系统。

### 注意

Raspbian Linux 是本书中使用的操作系统。

NOOBS 会提取 Raspbian Linux 操作系统并重新启动树莓派。当 NOOBS 正在提取操作系统时，屏幕上会显示一些关于如何使用树莓派的提示，包括默认的用户名和密码（默认用户名是`pi`，默认密码是`raspberry`）。

树莓派重启后，你将能够使用`raspi-config`命令来完成安装（参见*首次启动 Raspbian Linux*的步骤）。

## 还有更多...

NOOBS 是开始使用树莓派的最简单方法。它是一个安装工具，而不是完整的操作系统。

在本食谱中，你使用 NOOBS 安装了 Raspbian Linux 操作系统。NOOBS 还可以用于安装其他多个操作系统，包括 Arch、OpenELEC、Pidora 和 RaspBMC。

通过使用 NOOBS，Raspbian Linux 可以配置为直接启动到一个名为**Scratch**的易用编程环境。NOOBS 还具有内置配置编辑器，专家可以使用它来对启动配置进行额外的调整。

## 另见

+   NOOBS（即新盒子软件）可在[`github.com/raspberrypi/noobs`](https://github.com/raspberrypi/noobs)下载

NOOBS 旨在使选择和安装树莓派操作系统变得更加容易，无需担心手动为 SD 卡创建镜像。

# Mac OS X 磁盘工具 – diskutil 和 dd

本食谱解释了如何使用 Mac OS X 计算机上的`diskutil`和`dd`磁盘工具，在 SD 卡上安装树莓派操作系统镜像。

你应该已经下载了一个树莓派的磁盘镜像，并且准备好将镜像写入 SD 卡。

完成这个步骤后，你将能够从 Mac OS X 计算机将 SD 卡写入。

## 准备工作

材料：

+   一台运行 Mac OS X 的计算机和一台 SD 卡写入器

+   一个 4GB 或更大的 SD 卡（Class 10 卡性能最好）

+   一个树莓派操作系统镜像文件

`diskutil`和`dd`磁盘工具命令在 Mac OS X 操作系统中默认安装。`diskutil`命令用于管理磁盘设备，`dd`命令用于将数据复制到磁盘设备或从磁盘设备复制数据。

`dd`命令需要管理员权限。使用`sudo`命令可以临时给予用户管理员权限。

## 如何操作...

在 Linux 电脑上，将磁盘镜像写入 SD 卡需要执行以下步骤：

1.  打开终端。

1.  使用以下命令确定 SD 卡驱动器的名称：

    ```
    diskutil list

    ```

1.  使用以下命令卸载已挂载的 SD 卡：

    ```
    diskutil unmountdisk /dev/disk2

    ```

1.  使用`dd`将磁盘镜像复制到 SD 卡（这需要`sudo`），如下所示：

    ```
    sudo dd bs=1M if=raspbian.img of=/dev/rdisk2

    ```

### 注意

小心选择磁盘！确保不要擦除错误的磁盘！

下面是一个 Terminal 会话示例，展示如何使用`diskutil`和`dd`命令来发现 SD 卡的磁盘驱动器名称，卸载 SD 卡，并将树莓派操作系统镜像写入 SD 卡：

```
macosx:~ $ diskutil list

/dev/disk0
   #:                   TYPE NAME              SIZEIDENTIFIER
   0:  GUID_partition_scheme*500.3 GB   disk0
   1:                    EFI EFI209.7 MB   disk0s1
   2:      Apple_CoreStorage499.4 GB   disk0s2
   3:             Apple_Boot Recovery HD       650.0 MB   disk0s3
/dev/disk1
   #:                   TYPE NAME              SIZE       IDENTIFIER
   0:              Apple_HFS Macintosh HD     *499.1 GB   disk1
/dev/disk2
   #:                   TYPE NAME              SIZE       IDENTIFIER
   0: FDisk_partition_scheme NO_NAME *4.0 GB   disk2
   1:             DOS_FAT_32 NO_NAME           4.0 GB     disk2s1

macosx:~ $ diskutilumountdisk /dev/disk2

macosx:~ $ cd Downloads

macosx:Downloads $ dd bs=1M if=raspbian.img of=/dev/rdisk2
```

## 它是如何工作的...

`diskutil`命令用于查找 SD 卡的名称并卸载磁盘。

该命令首先与`list`子命令一起使用，以显示每个已挂载磁盘驱动器的信息。

插入 SD 卡后，SD 卡会出现在此列表中，路径为`/dev/disk2`。

现在我们知道 SD 卡磁盘设备的路径是`/dev/disk2`，可以使用`unmountdisk`子命令来卸载 SD 卡。

最后，使用`dd`命令将 Raspberry Pi 磁盘镜像写入 SD 卡：

+   每个写入的磁盘块为 1MB（`bs=1M`）

+   输入文件（`if`）是`raspbian.img`

+   输出文件（`of`）是 SD 卡磁盘设备（`/dev/rdisk2`）

注意，输出文件被命名为`/dev/rdisk2`，而不是`/dev/disk2`。额外的`r`要求 Mac OS X 在写入磁盘时使用原始模式。原始模式比默认模式更快，如果你需要写入的话。

## 还有更多内容…

`diskutil`命令工具是一个功能丰富的工具，用于在 Mac OS X 上修改、验证和修复磁盘。有关`diskutil`命令的更多信息可以通过使用内置的 man 页面（`man diskutil`）查看。

在前面的示例中，镜像文件在复制到 SD 卡之前，需要先卸载磁盘（`diskutil unmountdisk`）。在格式化或覆盖磁盘之前，卸载磁盘非常重要。

当使用`dd`命令复制镜像时，

+   `if=` 指定输入文件（`raspbian.img`）

+   `of=` 指定输出文件（`/dev/rdisk2`）

+   `bs=` 指定写入磁盘的块大小

`dd`工具也可以用作备份工具。只需交换输入文件（`if=`）和输出文件（`of=`）即可。

使用以下命令从前面的示例中的磁盘创建备份：

```
dd bs=1M if=/dev/rdisk2 of=backup-2015-06-20.img

```

在`dd`命令运行时，按下*Ctrl* + *T*会使命令报告其进度。

有关`dd`命令的更多信息可以在其 man 页面（`man dd`）中找到。

## 另见

+   **磁盘工具** ([`en.wikipedia.org/wiki/Disk_Utility`](http://en.wikipedia.org/wiki/Disk_Utility)): `diskutil`命令可以用来卸载系统中的磁盘。这篇维基百科文章详细解释了`diskutil`命令的所有功能。

+   **diskutil – 修改、验证和修复本地磁盘** ([`developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man8/diskutil.8.html`](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man8/diskutil.8.html)): `diskutil`命令是 Mac OS X 操作系统的一部分。Apple 的 man 页面描述了`diskutil`命令及其选项。

+   **dd – 转换和复制文件** ([`developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man1/dd.1.html`](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man1/dd.1.html)): `dd`命令可以用来将镜像文件复制到磁盘或从磁盘复制。Apple 的 man 页面为`dd`命令提供了详细的信息和选项。

# Image Writer for Windows – Win32DiskImager.exe

本教程展示了如何使用开源的 Windows 图像写入工具`Win32DiskImager.exe`将 Raspberry Pi 操作系统镜像安装到 SD 卡上。

你应该已经下载了 Raspberry Pi 磁盘镜像，并准备好使用 Windows PC 将镜像写入 SD 卡。

要完成此教程，你还需要连接互联网来下载 Windows 版的 Image Writer。

完成此步骤后，您将能够从 Windows 计算机将树莓派映像写入 SD 卡。

## 准备开始

以下是所需的组件：

+   一台运行 Windows 的计算机，带有 SD 卡读卡器

+   4GB 或更大的 SD 卡（class 10 具有最佳性能）

+   一个树莓派操作系统的映像文件

+   预编译的`Win32DiskImager`二进制文件

预编译的`Win32DiskImager`二进制文件作为 ZIP 文件分发，可以从[`launchpad.net/win32-image-writer`](https://launchpad.net/win32-image-writer)下载。

## 如何操作...

以下步骤是将磁盘映像写入 SD 卡到 Windows 计算机所需的步骤：

1.  从[`launchpad.net/win32-image-writer`](https://launchpad.net/win32-image-writer)下载`Win32DiskImager`的 ZIP 文件。

1.  将 ZIP 文件解压到硬盘上的文件夹中，例如`C:\Win32DiskImager`。

1.  从安装文件夹运行`Win32DiskImager.exe`。

1.  选择下载的树莓派磁盘映像作为源映像文件，并选择 SD 卡写入设备作为目标设备。

1.  点击**写入**按钮将映像文件复制到 SD 卡中。

将映像写入磁盘大约需要 5 分钟，适用于 2GB 的映像文件。映像写入 SD 卡后，可以弹出 SD 卡并用来启动树莓派。

## 它是如何工作的...

首先，您需要下载并安装 Windows 版本的 Image Writer（`Win32DiskImager`）。`Win32DiskImager`是一个独立的可执行文件，可以安装到计算机上的任何文件夹中。

双击扩展后的`Win32DiskImager`可执行文件启动应用程序。

一旦应用程序启动，选择下载的树莓派磁盘映像作为源映像文件，然后选择 SD 卡写入设备的位置作为目标设备。当您点击**写入**按钮时，`Win32DiskImager`会将树莓派磁盘映像写入 SD 卡。

## 还有更多...

`Win32DiskImager`也是一个优秀的备份工具！在启动并配置好树莓派后，可以进行备份，以防 SD 卡损坏或丢失时保存映像。

要创建备份，请执行以下步骤：

1.  运行`Win32DiskImager.exe`。

1.  选择 SD 卡作为源设备，选择一个新的映像文件作为目标。

1.  点击**读取**按钮将 SD 卡中的内容读取到新的映像文件中。

从 SD 卡备份树莓派磁盘映像所需的步骤与写入映像的步骤类似。唯一的区别是在备份过程中，SD 卡是源文件，磁盘上的新映像文件是目标。

每次更新树莓派操作系统、应用程序软件或配置后，都应创建新的备份。

## 另请参见

+   **Windows 图像写入工具** ([`launchpad.net/win32-image-writer`](https://launchpad.net/win32-image-writer)): 该工具最初是为了读取和写入特定 Linux 发行版的磁盘镜像而编写的。然而，现在它已被泛化，成为许多开发项目中流行的工具，如 Raspberry Pi。Windows 图像写入工具的主页提供了该磁盘镜像工具的详细信息。

# 转换并复制到 Linux – dd

本教程说明了如何使用 `dd` 标准 Linux 工具将操作系统镜像安装到 SD 卡上。

你应该已经下载了 Raspberry Pi 的磁盘镜像，并且现在准备使用 Linux 电脑将该镜像写入 SD 卡。

大多数版本的 Linux 和 Mac OS 都已安装 `dd` 命令。这个强大的复制命令版本（`cp`）可以用来将数据块写入设备，如 SD 卡。

完成此教程后，你将能够从 Linux 计算机写入 SD 卡。

## 准备工作

这里是所需的材料：

+   一台运行 Linux 的计算机和一个 SD 卡读写器

+   容量为 4 GB 或更大的 SD 卡（Class 10 具有最佳性能）

+   一个 Raspberry Pi 操作系统镜像文件

`dd` 工具通常默认安装在大多数 Linux 发行版中。如果没有安装，可以使用适当的 Linux 安装工具进行安装。

本示例中的所有命令都作为特权用户（root）执行。

## 操作步骤...

在 Linux 计算机上将磁盘镜像写入 SD 卡的步骤如下：

1.  使用 `df` 命令来确定 SD 驱动器的名称。

    ```
    df
    ```

1.  使用 `umount` 卸载已挂载的磁盘分区，如下所示：

    ```
    umount /dev/mmcblk0p1

    ```

1.  使用 `dd` 将磁盘镜像复制到 SD 卡，如下所示：

    ```
    dd bs=1M if=rasbian.img of=/dev/mmcblk0

    ```

下面是一个终端会话的示例，显示了使用 `df` 命令来查找 SD 卡驱动器的名称，使用 `umount` 命令卸载 SD 卡，以及使用 `dd` 命令将 Raspberry Pi 镜像写入 SD 卡：

```
user@host ~ $ df –vh

Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1       9.1G  7.3G  1.4G  85% /
udev            992M     0  992M   0% /dev
/dev/sda3       9.1G  4.9G  3.8G  57% /sys1
/dev/sda4       9.1G  4.2G  4.4G  49% /sys2
/dev/mmcblk0p   3.8G  1.2G  2.6G  32% /media/A181-918F

user@host ~ $ umount /dev/mmcblk0p1

user@host ~ $ dd bs=1M if=raspbian.img of=/dev/mmcblk0
```

## 它是如何工作的...

使用 `df` 命令发现 SD 驱动器的名称。

`df` 命令显示每个已挂载磁盘驱动器的可用空间。插入 SD 卡后，SD 卡的主分区（`p1`）将出现在此列表中，显示为 `/dev/mmcblk0p1`。因此，SD 卡磁盘设备是 `/dev/mmcblk0`（注意，`p1` 丢失了）。

现在我们知道 SD 卡磁盘设备是 `/dev/mmcblk0`，通过使用 `umount` 命令卸载 SD 卡（注意 `umount` 中没有 `n`）。

最后，使用 `dd` 命令将 Raspberry Pi 镜像写入 SD 卡：

+   每个写入的磁盘块为 1 MB（`bs=1M`）

+   输入文件（`if`）是 `raspbian.img`

+   输出文件（`of`）是 SD 卡磁盘设备（`/dev/mmcblk0`）

在 `dd` 命令运行时按 *Ctrl* + *T*，将使命令报告进度。

## 还有更多...

`dd`工具是大多数 Linux 发行版中常见的核心 Gnu 工具之一。它是一个低级工具，简单地将数据块从一个文件复制到另一个文件。

前面的示例展示了如何使用`df`命令来确定 SD 卡磁盘驱动器的名称。SD 磁盘的第一个分区`/dev/mmcblk0p1`被挂载在/`media/A1B1-918F`。磁盘镜像覆盖整个磁盘，而不仅仅是一个分区。因此，前述示例中的磁盘驱动器的正确名称是`/dev/mmcblk0`（注意没有`p1`）。

在将镜像复制到前述示例中的 SD 卡之前，需要卸载磁盘分区。格式化或覆盖磁盘之前，最好先卸载所有磁盘分区。

当使用`dd`命令复制镜像时，

+   `if=`指定输入文件（`raspbian.img`）

+   `of=`指定输出文件`(/dev/mmcblk0)`

+   `bs=`指定写入磁盘的块大小

`dd`工具也可以作为备份工具使用，只需交换输入文件（`if=`）和输出文件（`of=`）。

使用以下命令创建备份，使用前述示例中的磁盘：

```
dd bs=1M if=/dev/mmcblk0 of=backup-2015-06-20.img

```

## 另请参见

+   **dd (Unix)**（[`en.wikipedia.org/wiki/Dd_(Unix)`](http://en.wikipedia.org/wiki/Dd_(Unix))）：这篇维基百科文章解释了`dd`命令的原始应用。

+   **dd – 转换并复制文件**（[`manpages.debian.net/cgi-bin/man.cgi?query=dd`](http://manpages.debian.net/cgi-bin/man.cgi?query=dd)）：Debian 的`dd`命令手册页面描述了该命令及其选项。

+   **dd（gnu - coreutils）**（[`www.gnu.org/software/coreutils/manual/html_node/dd-invocation.html`](http://www.gnu.org/software/coreutils/manual/html_node/dd-invocation.html)）：GNU 操作系统手册中的`dd`参考文档，提供了详细描述。

# 第一次启动 Raspbian Linux

本步骤解释了如何启动官方的 Raspbian Jessie Linux 发行版，并使用`raspi-config`命令远程完成树莓派的安装。

当树莓派第一次启动时，它会自动进入**图形用户界面**（**GUI**）模式——树莓派桌面。树莓派还会在首次启动时启动一个安全外壳服务器。因此，可以在没有显示器连接到树莓派的情况下完成安装。

在此步骤中，`raspi-config`命令通过远程外壳（SSH 或 PuTTY）运行，以完成树莓派的安装。有关`raspi-config`的使用详细信息，请参见第二章，*管理*。

### 注意

树莓派的安装也可以通过 GUI 使用**树莓派配置**工具完成，该工具可以在**首选项**菜单中找到。

一旦此步骤完成，您将第一次启动树莓派。

## 准备工作

这里是所需材料：

+   树莓派的基本网络设置（请参见*准备初次启动*步骤）

+   格式化为 Raspbian Linux 镜像的 SD 卡

+   网络连接

对于此操作，SD 卡应已经格式化为 Raspbian Jessie 磁盘镜像，或者使用 NOOBS 选择了 Raspbian 操作系统，并且树莓派应已连接到本地网络，另一台计算机用于远程连接树莓派。

## 如何操作...

执行以下步骤以首次启动树莓派：

1.  将 SD 卡插入树莓派并接入电源。树莓派应开始启动。

1.  在短暂的初始启动后，树莓派将通过`raspberrypi.local`主机名在本地网络上宣布自己。

1.  使用安全外壳登录树莓派。用户名为`pi`的用户的默认密码为 raspberry（第二章, *管理*，中有两个远程访问的操作步骤）。

    ```
    golden-macbook:~ rick$ ssh pi@raspberrypi.local

    The authenticity of host 'raspberrypi.local (fe80::ba27:ebff:fe57:796d%en5)' can't be established.
    RSA key fingerprint is da:2a:c1:d4:93:f8:02:2c:36:71:ae:6b:9e:83:a6:d4.

    Are you sure you want to continue connecting (yes/no)? yes

    pi@raspberrypi.local's password: 

    The programs included with the Debian GNU/Linux system are free software;
    the exact distribution terms for each program are described in the
    individual files in /usr/share/doc/*/copyright.

    Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
    permitted by applicable law.
    Last login: Thu Sep 24 15:33:00 2015

    pi@raspberrypi ~ $ 
    ```

1.  使用`raspi-config`命令更新操作系统。此命令需要管理员权限，执行时需要加上`sudo`前缀。有关*执行带有权限的命令*的更多信息，请参见第二章, *管理*。

    ```
    pi@raspberrypi ~ $ sudo raspi-config
    ```

1.  请注意，`raspi-config`主菜单有多个选项用于更新树莓派操作系统：

    ```
    ┌─────────┤ Raspberry Pi Software Configuration Tool (raspi-config) ├──────────┐
    │                                                                              │
    │    1 Expand Filesystem              Ensures that all of the SD card s        │
    │    2 Change User Password           Change password for the default u        │
    │    3 Boot Options                   Choose whether to boot into a des        │
    │    4 Internationalisation Options   Set up language and regional sett        │
    │    5 Enable Camera                  Enable this Pi to work with the R        │
    │    6 Add to Rastrack                Add this Pi to the online Raspber        │
    │    7 Overclock                      Configure overclocking for your P        │
    │    8 Advanced Options               Configure advanced settings              │
    │    9 About raspi-config             Information about this configurat        │
    │                                                                              │
    │                                                                              │
    │                                                                              │
    │                     <Select>                     <Finish>                    │
    │                                                                              │
    └──────────────────────────────────────────────────────────────────────────────┘

    ```

1.  以下是主要的配置选项：

    +   **扩展文件系统**：将根分区调整大小以填充 SD 卡（如果你使用了 NOOBS，则无需此操作）

    +   **更改用户密码**：更改默认密码（这应该是你做的第一件事）。

    +   **启动选项**：选择以文本模式或桌面 GUI 模式启动树莓派（本书仅使用文本模式）。

    +   **国际化选项**：更改显示语言和默认键盘布局（默认语言为英语（英国），默认键盘布局为英国）

    +   **启用相机**：启用树莓派相机的使用

    +   **添加到 Rastrack**：将树莓派添加到树莓派基金会的使用统计中

    +   **超频**：将树莓派置于 Turbo 模式（最新型号，树莓派 2，仅有一个速度选项）

    +   **高级选项**：为高级用户提供其他配置选项（如过扫描、SSH、内存分配和音频）

    +   **关于 raspi-config**：提供有关`raspi-config`的信息

1.  选择**1 扩展文件系统**，将 SD 卡上的文件系统扩展，以便使用 SD 卡上的所有可用空间。第二章, *管理*，中有关于*扩展文件系统大小*的详细说明。

1.  选择**2 更改用户密码**以更改默认密码。

1.  选择**完成**以完成配置并重启系统。

## 工作原理...

启动时，树莓派会将其主机名（`raspberrypi.local`）注册到本地 **多播域名服务器**（**mDNS**）。大多数家庭网关和局域网都包括一个 mDNS，为动态连接到网络的移动设备和计算机提供域名注册服务。

一旦树莓派启动并注册其主机名，可以使用安全外壳客户端通过 `raspberrypi.local` 主机名连接到树莓派，用户名为 `pi`，密码为 `raspberry`。第二章，*管理*，有两个远程访问食谱，一个用于 Windows（PuTTY），一个用于 Mac OS X 和 Linux（SSH）。

Raspbian Linux 操作系统发行版包含 `raspi-config` 工具。此配置工具应在首次启动操作系统时运行，用于扩展文件系统并更改默认密码。

`raspi-config` 命令具有管理员权限，需要在前面加上 `sudo` 前缀才能运行。有关 *以管理员权限执行命令* 的更多信息，请参见 第二章，*管理*。

当 `raspi-config` 主菜单出现时，你可以使用键盘的箭头键、*Tab* 键、空格键或 *Return* 键来导航菜单。

第二章，*管理*，有一些使用 `raspi-config` 命令配置树莓派的食谱。现在，只需使用 **扩展文件系统** 和 **更改用户密码** 菜单项，以及 *扩展文件系统大小* 和 *更改登录密码* 食谱。

### 注意

如果你使用了 NOOBS，则不需要扩展文件系统，因为 NOOBS 已经扩展了文件系统。

从主菜单选择 **Finish** 将使树莓派重新启动。

一旦重新启动，树莓派就可以使用了！

## 另见

+   **多播 DNS** ([`en.wikipedia.org/wiki/Multicast_DNS`](https://en.wikipedia.org/wiki/Multicast_DNS))：这篇维基百科文章描述了 mDNS 如何在小型局域网中解析主机名为 IP 地址。

# 关闭树莓派

本食谱展示了如何安全地关闭树莓派。

在关闭树莓派之前，首先关闭操作系统非常重要，这样树莓派上的所有应用程序和服务才有机会完成任何正在进行的磁盘写入操作，并为下次启动做准备。

外部设备，如硬盘，也需要时间来关闭并刷新它们的缓冲区。`shutdown` 命令还为连接到树莓派的设备提供了一个清理并为下次启动做好准备的机会。

完成此步骤后，你将能够安全地关闭树莓派。

## 准备就绪

这是我的材料：

+   树莓派的初始设置（参考 *准备启动初始化* 食谱）

+   使用官方 Raspbian Linux 镜像格式化的 SD 卡

在执行此操作之前，树莓派应该已经开机并完成启动。

## 如何操作...

执行以下步骤以关闭树莓派：

1.  如果你尚未登录，请以`pi`用户身份登录到树莓派（默认密码为`raspberry`）：

    ```
    Raspbian GNU/Linux 7raspberrypi tty1

    Raspberrypi login: pi
    Password:

    Last Login: Sun Jun 21 19:45:35 UTC 2015 on tty1
    Linux raspberrypi 3.18.11-v7+ #781SMP PREEEMPT Tue Apr 21 18:07:59 BST 2015armv7l

    The programs included with the Debian GNU/Linux system are free software;The exact distribution terms for each program are described in the individual files in /user/share/doc/*/copyright.

    Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent permitted by applicable law.

    pi@raspberrypi ~ $ 
    ```

1.  关闭并停止（`-h`）操作系统。此命令需要管理员权限。使用`sudo`前缀以管理员身份运行关机命令，如下所示：

    ```
    pi@raspberrypi ~ $ sudo shutdown –h now

    Broadcast message from root@raspberrypi (pts/0) (Sun Jun 21 19:53:03 2015):
    The system is going down for system halt NOW!

    ```

1.  在执行`shutdown`命令后，树莓派将开始其关机过程，显示应用程序、设备和服务的消息，展示它们在清理并准备下次启动时的状态。

1.  一旦操作系统关闭，树莓派将停止，只有一个红色 LED 灯会亮起（只要 LED 灯还在闪烁，树莓派仍在忙于关机）。

1.  电源现在可以从树莓派上拔下。

## 它是如何工作的...

如果你尚未登录到树莓派，在关机之前，你需要先登录到树莓派。

默认用户是`pi`。你应该在第一次启动时已经更改了默认用户的密码（参见*首次启动 Raspbian Linux*的操作）。如果你没有更改密码，默认密码是`raspberry`。

登录后，执行带有`–h`选项的`shutdown`命令，这将告诉树莓派在操作系统关闭后停止。

`shutdown`命令是特权命令。因此，`sudo`命令用于临时授予权限。有关*以特权身份执行命令*的更多信息，请参见第二章，*管理*部分。

## 还有更多...

`shutdown`命令还可以用来重启系统。只需使用`–r`重启选项代替`–h`关机选项。

当你以`pi`用户身份登录时，可以通过以下命令重启系统：

```
pi@raspberrypi ~ $ sudo shutdown –r now

```

`shutdown`命令的同义词包括`poweroff`和`reboot`。

为了关闭系统，你也可以使用以下命令代替`shutdown –h`：

```
pi@raspberrypi ~ $ sudo poweroff

```

除了使用`shutdown –r`，你还可以使用以下命令：

```
pi@raspberrypi ~ $ sudo reboot

```

有关这些命令的更多信息，请查阅它们的手册页面。

## 另请参见

+   **halt, reboot, poweroff – 停止系统** ([`manpages.debian.net/cgi-bin/man.cgi?query=halt`](http://manpages.debian.net/cgi-bin/man.cgi?query=halt)): `shutdown`命令有替代选项。Debian 的 halt、poweroff 和 reboot 手册页面详细描述了这些命令。

+   **shutdown – 关闭系统** ([`manpages.debian.net/cgi-bin/man.cgi?query=shutdown`](http://manpages.debian.net/cgi-bin/man.cgi?query=shutdown)): `shutdown`命令可以用来停止系统（`-h`）或重启系统（`-r`）。Debian 的`shutdown`手册页面详细描述了该命令及其所有选项。
