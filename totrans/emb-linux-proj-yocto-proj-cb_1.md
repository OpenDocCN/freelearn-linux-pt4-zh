# 第一章：构建系统

在本章中，我们将涵盖以下教程：

+   设置主机系统

+   安装 Poky

+   创建构建目录

+   构建你的第一个镜像

+   解释 Freescale Yocto 生态系统

+   安装 Freescale 硬件的支持

+   构建 Wandboard 镜像

+   排查你的 Wandboard 第一次启动问题

+   配置网络启动以进行开发设置

+   共享下载

+   共享共享状态缓存

+   设置软件包源

+   使用构建历史

+   使用构建统计信息

+   调试构建系统

# 介绍

Yocto 项目（[`www.yoctoproject.org/`](http://www.yoctoproject.org/)）是一个嵌入式 Linux 发行版构建工具，利用了其他多个开源项目。

Yocto 项目提供了一个嵌入式 Linux 的参考构建系统，称为 **Poky**，它以 **BitBake** 和 **OpenEmbedded-Core**（**OE-Core**）项目为基础。Poky 的目的是构建嵌入式 Linux 产品所需的组件，具体包括：

+   启动加载程序镜像

+   一个 Linux 内核镜像

+   一个根文件系统镜像

+   用于应用程序开发的工具链和**软件开发工具包**（**SDK**）

通过这些，Yocto 项目满足了系统和应用程序开发人员的需求。当 Yocto 项目作为引导加载程序、Linux 内核和用户空间应用程序的集成环境时，我们称之为系统开发。

对于应用程序开发，Yocto 项目构建 SDK，使得应用程序可以独立于 Yocto 构建系统进行开发。

Yocto 项目每六个月发布一个新版本。本文写作时的最新版本是 Yocto 1.7.1 Dizzy，本书中的所有示例都指的是 1.7.1 版本。

一个 Yocto 发行版包括以下组件：

+   Poky，参考构建系统

+   一个构建设备；即一个准备好使用 Yocto 的主机系统的 VMware 镜像

+   适用于主机系统的 **应用程序开发工具包**（**ADT**）安装程序

+   以及针对不同支持的平台：

    +   预构建工具链

    +   预构建的打包二进制文件

    +   预构建镜像

Yocto 1.7.1 版本可从 [`downloads.yoctoproject.org/releases/yocto/yocto-1.7.1/`](http://downloads.yoctoproject.org/releases/yocto/yocto-1.7.1/) 下载。

# 设置主机系统

本教程将说明如何设置主机 Linux 系统以使用 Yocto 项目。

## 准备工作

开发嵌入式 Linux 系统的推荐方式是使用本地 Linux 工作站。虽然虚拟机可以用于演示和测试目的，但不推荐用于开发工作。

Yocto 从零开始构建之前提到的所有组件，包括交叉编译工具链和它所需的本地工具，因此 Yocto 的构建过程对处理能力以及硬盘空间和 I/O 都有较高要求。

尽管 Yocto 在配置较低的机器上也能正常工作，但对于专业开发人员的工作站，建议使用**对称多处理**（**SMP**）系统，配置应为 8 GB 或更大内存，并配备大容量、高速硬盘。构建服务器可以采用分布式编译，但这不在本书的范围内。由于构建过程中的不同瓶颈，似乎在超过 8 个 CPU 或大约 16 GB 内存时，性能提升并不显著。

第一次构建时，系统还会从互联网下载所有源代码，因此建议使用快速的互联网连接。

## 如何操作...

Yocto 支持多种发行版，每个 Yocto 版本都会记录支持的发行版列表。尽管强烈建议使用支持的 Linux 发行版，但 Yocto 也能在任何 Linux 系统上运行，只要它具备以下依赖项：

+   Git 1.7.8 或更高版本

+   Tar 1.24 或更高版本

+   Python 2.7.3 或更高版本（但不支持 Python 3）

Yocto 还提供了一种安装这些工具正确版本的方法，方法是下载 *buildtools-tarball* 或在支持的机器上构建一个。这使得几乎任何 Linux 发行版都可以运行 Yocto，同时确保未来能够复制你的 Yocto 构建系统。这对具有长期可用性要求的嵌入式产品非常重要。

本书将使用 Ubuntu 14.04 **长期稳定版**（**LTS**）Linux 发行版进行所有示例的操作。其他 Linux 发行版的安装说明可以在 *Yocto 项目开发手册* 的 *支持的 Linux 发行版* 部分找到，但所有示例仅在 Ubuntu 14.04 LTS 上进行过测试。

为了确保你安装了 Yocto 所需的包依赖，并能够跟随本书中的示例进行操作，请从你的 shell 中运行以下命令：

```
$ sudo apt-get install gawk wget git-core diffstat unzip texinfo gcc- multilib build-essential chrpath socat libsdl1.2-dev xterm make xsltproc docbook-utils fop dblatex xmlto autoconf automake libtool libglib2.0-dev python-gtk2 bsdmainutils screen

```

### 提示

**下载示例代码**

你可以从 [`www.packtpub.com`](http://www.packtpub.com) 账户下载所有已购买的 Packt 图书的示例代码文件。如果你在其他地方购买了本书，可以访问 [`www.packtpub.com/support`](http://www.packtpub.com/support) 并注册，文件将直接通过电子邮件发送给你。

本书中的示例代码可以通过多个 GitHub 仓库访问，地址是 [`github.com/yoctocookbook`](https://github.com/yoctocookbook)。按照 GitHub 上的说明获取源代码副本。

## 如何操作...

上述命令将使用 `apt-get`，即**高级包装工具**（**APT**）的命令行工具。它是**dpkg**包管理器的前端，包含在 Ubuntu 发行版中。它将安装所有必需的包及其依赖项，以支持 Yocto 项目的所有功能。

## 更多内容...

如果构建时间对你来说是一个重要因素，在准备磁盘时，有一些步骤可以进一步优化：

+   将`build`目录放置在独立的磁盘分区或快速外部驱动器上。

+   使用 ext4 文件系统，但配置其不在 Yocto 专用分区上使用日志记录。请注意，电源丧失可能会损坏你的构建数据。

+   挂载文件系统时，确保读取时不记录读写时间，禁用写入屏障，并使用以下挂载选项延迟提交文件系统更改：

    ```
    noatime,barrier=0,commit=6000.
    ```

+   不要在网络挂载的驱动器上进行构建。

这些更改减少了数据完整性保护，但由于`build`目录已分离到独立磁盘，失败只会影响临时构建数据，这些数据可以被删除并重新生成。

## 另见

+   完整的 Yocto 项目安装说明适用于 Ubuntu 和其他受支持的发行版，可以在*Yocto 项目参考手册*中找到，网址是[`www.yoctoproject.org/docs/1.7.1/ref-manual/ref-manual.html`](http://www.yoctoproject.org/docs/1.7.1/ref-manual/ref-manual.html)

# 安装 Poky

本食谱将解释如何使用 Poky 设置你的 Linux 主机系统，Poky 是 Yocto 项目的参考系统。

## 准备工作

Poky 使用 OpenEmbedded 构建系统，因此使用 BitBake 工具，BitBake 是一个用 Python 编写的任务调度器，源自 Gentoo 的 Portage 工具。你可以将 BitBake 视为 Yocto 中的 make 工具。它将解析配置和食谱元数据，调度任务列表，并执行其中的任务。

BitBake 也是 Yocto 的命令行接口。

Poky 和 BitBake 是 Yocto 使用的两个开源项目。Poky 项目由 Yocto 社区维护。你可以从其 Git 仓库下载 Poky，网址是[`git.yoctoproject.org/cgit/cgit.cgi/poky/`](http://git.yoctoproject.org/cgit/cgit.cgi/poky/)。

你可以通过访问开发邮件列表[`lists.yoctoproject.org/listinfo/poky`](https://lists.yoctoproject.org/listinfo/poky)来跟踪和参与开发讨论。

另一方面，BitBake 由 Yocto 和 OpenEmbedded 社区共同维护，因为这两个社区都使用该工具。你可以从其 Git 仓库下载 BitBake，网址是[`git.openembedded.org/bitbake/`](http://git.openembedded.org/bitbake/)。

你可以通过访问开发邮件列表[`lists.openembedded.org/mailman/listinfo/bitbake-devel`](http://lists.openembedded.org/mailman/listinfo/bitbake-devel)来跟踪和参与开发讨论。

Poky 构建系统仅支持以下架构的虚拟化 QEMU 机器：

+   ARM (qemuarm)

+   x86 (qemux86)

+   x86-64 (qemux86-64)

+   PowerPC (qemuppc)

+   MIPS (qemumips, qemumips64)

除了这些，Poky 还支持一些**板级支持包**（**BSPs**），这些 BSP 代表了刚刚列出的架构。以下是这些 BSP：

+   Texas Instruments Beaglebone (beaglebone)

+   Freescale MPC8315E-RDB (mpc8315e-rdb)

+   基于 Intel x86 的 PC 和设备（genericx86 和 genericx86-64）

+   Ubiquiti Networks EdgeRouter Lite (edgerouter)

如果要在不同的硬件上开发，你需要为 Poky 补充硬件特定的 Yocto 层。这部分内容将在后续介绍。

## 如何操作...

Poky 项目包含了一个稳定的 BitBake 版本，因此要开始使用 Yocto，我们只需要在我们的 Linux 主机系统中安装 Poky。

### 注意

请注意，你也可以通过发行版的包管理系统独立安装 BitBake。虽然可以这样做，但不推荐这么做，因为 BitBake 需要与 Yocto 中使用的元数据兼容。如果你通过发行版安装了 BitBake，请将其卸载。

当前的 Yocto 版本是 1.7.1，也叫 Dizzy，所以我们将把它安装到主机系统中。我们将使用 `/opt/yocto` 文件夹作为安装路径：

```
$ sudo install -o $(id -u) -g $(id -g) -d /opt/yocto
$ cd /opt/yocto
$ git clone --branch dizzy git://git.yoctoproject.org/poky

```

## 它是如何工作的...

上述说明将使用 Git（源代码管理系统命令行工具）来克隆包含 BitBake 的 Poky 仓库，并将其指向 Dizzy 稳定分支，将仓库克隆到当前路径下的新 `poky` 目录中。

## 还有更多...

Poky 包含三个元数据目录，分别是 `meta`、`meta-yocto` 和 `meta-yocto-bsp`，以及一个名为 `meta-skeleton` 的模板元数据层，可用作新层的基础。以下是 Poky 的三个元数据目录的解释：

+   `meta`：此目录包含 OpenEmbedded-Core 元数据，支持 ARM、x86、x86-64、PowerPC、MIPS 和 MIPS64 架构以及 QEMU 仿真硬件。你可以从其 Git 仓库下载：[`git.openembedded.org/openembedded-core/`](http://git.openembedded.org/openembedded-core/)。

    你可以访问开发邮件列表，关注和参与开发讨论，网址：[`lists.openembedded.org/mailman/listinfo/openembedded-core`](http://lists.openembedded.org/mailman/listinfo/openembedded-core)。

+   `meta-yocto`：此目录包含 Poky 的发行版特定元数据。

+   `meta-yocto-bsp`：此目录包含参考硬件板的元数据。

## 另见

+   有关 Git 这个分布式版本控制系统的文档可以在[`git-scm.com/doc`](http://git-scm.com/doc)找到。

# 创建构建目录

在构建第一个 Yocto 镜像之前，我们需要为它创建一个 `build` 目录。

构建过程，按照之前所述，在主机系统上可能需要长达一小时，并且需要大约 20 GB 的硬盘空间来构建仅限控制台的镜像。像`core-image-sato`这样的图形镜像，构建过程可能需要长达 4 小时，并占用大约 50 GB 的空间。

## 如何操作...

我们需要做的第一件事是为我们的项目创建一个 `build` 目录，构建输出将在该目录中生成。有时，`build` 目录也可能被称为项目目录，但 `build` 目录才是 Yocto 中的标准术语。

当你有多个项目时，`build`目录的结构没有绝对正确的方式，但一个好的做法是每个架构或机器类型都有一个`build`目录。它们可以共享一个公共的`downloads`文件夹，甚至一个共享的状态缓存（稍后会介绍），因此将它们分开不会影响构建性能，但会允许你同时开发多个项目。

要创建一个`build`目录，我们使用由 Poky 提供的`oe-init-build-env`脚本。需要在当前的 shell 中执行该脚本，它会设置好使用 OpenEmbedded/Yocto 构建系统的环境，包括将 BitBake 工具添加到你的路径中。你可以指定一个`build`目录来使用，否则它会默认使用`build`目录。我们在这个示例中使用`qemuarm`。

```
$ cd /opt/yocto/poky
$ source oe-init-build-env qemuarm

```

脚本会切换到指定的目录。

### 注意

由于`oe-init-build-env`只配置当前的 shell，因此每次开启新 shell 时都需要重新执行它。但是，如果你将脚本指向现有的`build`目录，它会设置好你的环境，但不会改变你已有的配置。

### 提示

BitBake 采用客户端/服务器的抽象设计，因此我们也可以启动一个内存驻留的服务器，并将客户端连接到它。通过这种设置，可以避免每次加载缓存和配置信息，从而节省一些开销。要运行一个始终可用的内存驻留 BitBake，你可以按如下方式使用`oe-init-build-env-memres`脚本：

```
$ source oe-init-build-env-memres 12345 qemuarm

```

这里的`12345`是将要使用的本地端口。

不要同时使用两种 BitBake 变体，因为这可能会导致问题。

然后，你可以通过执行以下命令终止内存驻留的 BitBake：

```
$ bitbake -m

```

## 它是如何工作的...

这两个脚本都调用了`poky`目录中的`scripts/oe-setup-builddir`脚本来创建`build`目录。

在创建时，`build`目录包含一个`conf`目录，其中有以下三个文件：

+   `bblayers.conf`：该文件列出了要考虑的元数据层，用于该项目。

+   `local.conf`：该文件包含项目特定的配置变量。你可以使用`site.conf`文件为不同的项目设置通用配置变量，但默认情况下不会创建该文件。

+   `templateconf.cfg`：该文件包含用于创建项目的模板配置文件所在的目录。默认情况下，它使用指向你 Poky 安装目录中的`templateconf`文件所指定的目录，默认路径为`meta-yocto/conf`。

### 注意

要从头开始进行构建，`build`目录只需要这些。

删除这些文件以外的所有内容将会重新创建你的构建环境。

```
$ cd /opt/yocto/poky/qemuarm
$ rm -Rf tmp sstate-cache

```

## 还有更多...

在创建`build`目录时，你可以通过`TEMPLATECONF`变量指定一个不同的模板配置文件，例如：

```
$ TEMPLATECONF=meta-custom/config source oe-init-build-env <build- dir>

```

`TEMPLATECONF`变量需要指向一个包含`local.conf`和`bblayer.conf`模板的目录，但文件名应为`local.conf.sample`和`bblayers.conf.sample`。

对于我们的目的，我们可以使用未修改的默认项目配置文件。

# 构建您的第一个镜像

在构建我们的第一个镜像之前，我们需要决定要构建哪种类型的镜像。本配方将介绍一些可用的 Yocto 镜像，并提供构建简单镜像的说明。

## 准备工作

Poky 包含一组默认的目标镜像。您可以通过执行以下命令列出它们：

```
$ cd /opt/yocto/poky
$ ls meta*/recipes*/images/*.bb

```

不同镜像的完整描述可以在*Yocto 项目参考手册*中找到。通常，这些默认镜像作为基础使用，并根据您自己的项目需求进行定制。最常用的基础默认镜像有：

+   `core-image-minimal`：这是一个最小的基于 BusyBox、sysvinit 和 udev 的仅控制台镜像

+   `core-image-full-cmdline`：这是一个基于 BusyBox 的仅控制台镜像，提供完整的硬件支持和更完整的 Linux 系统，包括 bash

+   `core-image-lsb`：这是一个仅控制台镜像，符合 Linux 标准基础（LSB）标准

+   `core-image-x11`：这是一个基于 X11 窗口系统的基础镜像，带有图形终端

+   `core-image-sato`：这是一个基于 X11 窗口系统的镜像，带有 SATO 主题和 GNOME 移动桌面环境

+   `core-image-weston`：这是一个基于 Wayland 协议和 Weston 参考合成器的镜像

您还会找到以下后缀的镜像：

+   `dev`：这些镜像适用于开发工作，因为它们包含头文件和库文件。

+   `sdk`：这些镜像包括一个完整的 SDK，可用于目标开发。

+   `initramfs`：这是一个可以用于基于 RAM 的根文件系统的镜像，且可以选择与 Linux 内核一起嵌入。

## 如何操作...

要构建镜像，我们需要配置要构建的`MACHINE`并将其名称传递给 BitBake。例如，对于`qemuarm`机器，我们将运行以下命令：

```
$ cd /opt/yocto/poky/qemuarm
$ MACHINE=qemuarm bitbake core-image-minimal

```

或者，我们可以通过以下命令将`MACHINE`变量导出到当前的 shell 环境：

```
$ export MACHINE=qemuarm

```

但是，首选且持久的做法是编辑`conf/local.conf`配置文件，将默认机器更改为`qemuarm`：

```
- #MACHINE ?= "qemuarm"
+ MACHINE ?= "qemuarm"
```

然后，您可以执行以下操作：

```
$ bitbake core-image-minimal

```

## 它是如何工作的...

当您将目标配方传递给 BitBake 时，它首先解析以下配置文件：

+   `conf/bblayers.conf`：此文件用于查找所有配置的层

+   `conf/layer.conf`：此文件用于每个配置的层

+   `meta/conf/bitbake.conf`：此文件用于其自身的配置

+   `conf/local.conf`：此文件用于用户对当前构建的任何其他配置

+   `conf/machine/<machine>.conf`：此文件是机器配置文件；在我们的例子中，这是`qemuarm.conf`

+   `conf/distro/<distro>.conf`：此文件是分发政策；默认情况下，这是`poky.conf`文件

然后，BitBake 会解析所提供的目标配方及其依赖项。最终结果是一组相互依赖的任务，BitBake 将按照顺序执行这些任务。

## 还有更多...

大多数开发者不会对保留每个包的整个构建输出感兴趣，因此建议你在`conf/local.conf`文件中进行如下配置，来移除它：

```
INHERIT += "rm_work"
```

但与此同时，为所有包配置意味着你将无法开发或调试它们。

你可以通过将包添加到`RM_WORK_EXCLUDE`变量中，来添加一个要从清理中排除的包列表。例如，如果你要进行 BSP 工作，一个好的设置可能是：

```
RM_WORK_EXCLUDE += "linux-yocto u-boot"
```

记住，你可以在自己的层中使用自定义模板`local.conf.sample`配置文件来保存这些配置，并将它们应用到所有项目中，以便可以在所有开发者之间共享。

一旦构建完成，你可以在`build`目录下的`tmp/deploy/images/qemuarm`目录中找到输出的镜像。

默认情况下，镜像不会从`deploy`目录中删除，但你可以通过在`conf/local.conf`文件中添加以下内容来配置你的项目删除先前构建的相同镜像版本：

```
RM_OLD_IMAGE = "1"
```

你可以通过执行以下命令在 QEMU 模拟器上测试运行你的镜像：

```
$ runqemu qemuarm core-image-minimal

```

Poky 的`script`目录中包含的`runqemu`脚本是一个启动 QEMU 机器模拟器的封装器，简化了其使用。

# 解释 Freescale Yocto 生态系统

正如我们所见，Poky 元数据以`meta`、`meta-yocto`和`meta-yocto-bsp`层开始，并可以通过使用更多层来扩展。

兼容 Yocto 项目的 OpenEmbedded 层的索引可在[`layers.openembedded.org/`](http://layers.openembedded.org/)上查阅。

嵌入式产品的开发通常从使用制造商参考板设计进行硬件评估开始。除非你正在使用 Poky 已经支持的某个参考板，否则你需要扩展 Poky 以支持你的硬件。

## 准备工作

第一步是选择你的设计将基于哪种基础硬件。我们将使用一款基于 Freescale i.MX6 **系统级芯片**（**SoC**）的板卡作为我们嵌入式产品设计的起点。

本食谱概述了 Yocto 项目对 Freescale 硬件的支持。

## 如何操作...

SoC 制造商（在本例中为 Freescale）提供了一系列参考设计板可供购买，以及基于 Yocto 的官方软件发布。同样，其他使用 Freescale SoC 的制造商也提供参考设计板及其自有的基于 Yocto 的软件发布。

选择适合的硬件来作为你的设计基础是嵌入式产品中最重要的设计决策之一。根据你的产品需求，你将决定：

+   使用生产就绪的板卡，例如**单板计算机**（**SBC**）

+   使用模块并围绕它设计自定义载板

+   直接使用 Freescale 的 SoC 并设计你自己的板卡

大多数情况下，生产就绪的板卡无法满足专业嵌入式系统的具体要求，使用 Freescale SoC 设计完整载板的过程也会非常耗时。因此，使用一个已经解决了大部分技术挑战设计问题的合适模块是一个常见选择。

需要考虑的一些重要特性包括：

+   工业温度范围

+   电源管理

+   长期可用性

+   预认证的无线和蓝牙（如果适用）

支持 Freescale 基板的 Yocto 社区层被称为 `meta-fsl-arm` 和 `meta-fsl-arm-extras`。`meta-fsl-arm` 层支持的板卡仅限于 Freescale 参考设计，如果你考虑围绕 Freescale 的 SoC 设计自己的载板，参考设计将是起点。其他厂商的板卡则由 `meta-fsl-arm-extras` 层维护。

还有其他嵌入式厂商使用 `meta-fsl-arm`，但他们没有将自己的板卡集成到 `meta-fsl-arm-extras` 社区层中。这些厂商将保留自己的 BSP 层，该层依赖于 `meta-fsl-arm`，并针对其硬件提供特定支持。一个例子是 Digi International 及其基于 i.MX6 SoC 的 ConnectCore 6 模块。

## 它的工作原理...

为了理解 Freescale Yocto 生态系统，我们需要从 Freescale 社区 BSP 开始，其中包括支持 Freescale 参考板的 `meta-fsl-arm` 层及其配套的 `meta-fsl-arm-extra` 层，后者支持其他厂商的板卡，以及它与 Freescale 为其参考设计提供的官方 Freescale Yocto 发布版之间的区别。

社区发布版和 Freescale Yocto 发布版之间有一些关键的区别：

+   Freescale 发布版由 Freescale 内部开发，未涉及社区参与，并用于在 Freescale 参考板上进行 BSP 验证。

+   Freescale 发布版经过内部 QA 和验证测试流程，并由 Freescale 支持进行维护。

+   Freescale 为特定平台发布的版本会达到一个成熟的阶段，之后不再继续开发。此时，所有的开发工作已被集成到社区层中，平台将由 Freescale BSP 社区继续维护。

+   Freescale Yocto 发布版与 Yocto 不兼容，而社区发布版则兼容。

Freescale 的工程团队与 Freescale BSP 社区紧密合作，确保他们在官方发布版中的所有开发工作都能以可靠且快速的方式集成到社区层中。

通常，最佳选择是使用 Freescale BSP 社区发布版，但仍然使用作为厂商稳定 BSP 发布版一部分的 U-Boot 和 Linux 内核版本。

这意味着你将从制造商那里获得最新的 Linux 内核和 U-Boot 更新，同时也从社区那里获得根文件系统的最新更新，延长产品的生命周期，并确保你能够获得最新的应用程序、bug 修复和安全更新。

这利用了制造商的质量控制过程，针对接近硬件的系统组件，并使得在获得来自社区的用户空间更新的同时，能够使用制造商的支持。Freescale BSP 社区也非常响应和活跃，因此通常可以与他们共同解决问题，以便惠及所有方面。

## 还有更多...

Freescale BSP 社区通过以下层扩展了 Poky：

+   `meta-fsl-arm`：这是支持 Freescale 参考设计的社区层。它依赖于 OpenEmbedded-Core。即使 Freescale 停止对其进行积极开发，该层中的机器仍会得到维护。你可以从其 Git 仓库下载`meta-fsl-arm`，网址为[`git.yoctoproject.org/cgit/cgit.cgi/meta-fsl-arm/`](http://git.yoctoproject.org/cgit/cgit.cgi/meta-fsl-arm/)。

    可以通过访问开发邮件列表[`lists.yoctoproject.org/listinfo/meta-freescale`](https://lists.yoctoproject.org/listinfo/meta-freescale)来跟踪和参与开发讨论。

    `meta-fsl-arm`层通过以下链接从 Freescale 的仓库获取 Linux 内核和 U-Boot 的源代码：

    +   **Freescale Linux 内核 Git** **仓库**：[`git.freescale.com/git/cgit.cgi/imx/linux-2.6-imx.git/`](http://git.freescale.com/git/cgit.cgi/imx/linux-2.6-imx.git/)

    +   **Freescale** **U-Boot Git 仓库**：[`git.freescale.com/git/cgit.cgi/imx/uboot-imx.git/`](http://git.freescale.com/git/cgit.cgi/imx/uboot-imx.git/)

    还有其他 Linux 内核和 U-Boot 版本可用，但建议保持使用制造商支持的版本。

    `meta-fsl-arm`层包含 Freescale 的专有二进制文件，以启用某些硬件功能——最显著的是其硬件图形、 multimedia 和加密功能。为了使用这些功能，最终用户需要接受 Freescale 的**最终用户许可协议**（**EULA**），该协议包含在`meta-fsl-arm`层中。要接受该许可，需将以下行添加到项目的`conf/local.conf`配置文件中：

    ```
    ACCEPT_FSL_EULA = "1"
    ```

+   `meta-fsl-arm-extra`：这个层增加了对其他社区维护的板子的支持；例如，Wandboard。要下载该层的内容，你可以访问[`github.com/Freescale/meta-fsl-arm-extra/`](https://github.com/Freescale/meta-fsl-arm-extra/)。

+   `meta-fsl-demos`：这个层为演示目标镜像添加了一个元数据层。要下载该层的内容，你可以访问[`github.com/Freescale/meta-fsl-demos`](https://github.com/Freescale/meta-fsl-demos)。

Freescale 在上述层的基础上，还使用了一个额外的层用于其官方软件发布：`meta-fsl-bsp-release`。

+   `meta-fsl-bsp-release`：这是一个 Freescale 维护的层，用于官方 Freescale 软件发布。它包含了对 `meta-fsl-arm` 和 `meta-fsl-demos` 的修改，并不属于社区发布的一部分。

## 另见

+   欲了解更多信息，请参考 FSL 社区 BSP 版本说明，访问 [`freescale.github.io/doc/release-notes/1.7/`](http://freescale.github.io/doc/release-notes/1.7/)

# 安装对 Freescale 硬件的支持

在本食谱中，我们将安装社区版 Freescale BSP Yocto 版本，该版本为我们的 Yocto 安装添加对 Freescale 硬件的支持。

## 准备就绪

由于层数较多，手动克隆每一层并将它们添加到项目的 `conf/bblayers.conf` 文件中非常繁琐。社区使用了 Google 为其社区 Android 开发的 `repo` 工具来简化 Yocto 的安装过程。

要在主机系统中安装 `repo`，请输入以下命令：

```
$ sudo curl http://commondatastorage.googleapis.com/git-repo- downloads/repo > /usr/local/sbin/repo
$ sudo chmod a+x /usr/local/sbin/repo

```

`repo` 工具是一个 Python 实用程序，它解析一个名为 `manifest` 的 XML 文件，文件中列出了 Git 仓库。然后使用 `repo` 工具来整体管理这些仓库。

## 它是如何做的...

例如，我们将使用 `repo` 下载前面食谱中列出的所有仓库到我们的主机系统。为此，我们将指向 Freescale 社区 BSP `manifest`，用于 Dizzy 发布版：

```
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
  <default sync-j="4" revision="master"/>
  <remote fetch="git://git.yoctoproject.org" name="yocto"/>
  <remote fetch="git://github.com/Freescale" name="freescale"/>
  <remote fetch="git://git.openembedded.org" name="oe"/>
  <project remote="yocto" revision="dizzy" name="poky" path="sources/poky"/>
  <project remote="yocto" revision="dizzy" name="meta-fsl-arm" path="sources/meta-fsl-arm"/>
  <project remote="oe" revision="dizzy" name="meta-openembedded" path="sources/meta-openembedded"/>
  <project remote="freescale" revision="dizzy" name="fsl- community-bsp-base" path="sources/base">
        <copyfile dest="README" src="img/README"/>
        <copyfile dest="setup-environment" src="img/setup- environment"/>
  </project>
  <project remote="freescale" revision="dizzy" name="meta-fsl-arm- extra" path="sources/meta-fsl-arm-extra"/>
  <project remote="freescale" revision="dizzy" name="meta-fsl- demos" path="sources/meta-fsl-demos"/>
  <project remote="freescale" revision="dizzy" name="Documentation" path="sources/Documentation"/>
</manifest>
```

`manifest` 文件显示所有将要安装的不同组件的安装路径和仓库源。

## 它是如何工作的...

`manifest` 文件是 Freescale 社区 BSP 版本所需的不同层的列表。我们现在可以使用 `repo` 来安装它。运行以下命令：

```
$ mkdir /opt/yocto/fsl-community-bsp
$ cd /opt/yocto/fsl-community-bsp
$ repo init -u https://github.com/Freescale/fsl-community-bsp- platform -b dizzy
$ repo sync

```

如果你有多核机器并希望进行多线程操作，可以选择性地传递 `-jN` 参数进行同步；例如，在一个 8 核的主机系统中，你可以传递 `repo sync -j8`。

## 还有更多...

要列出不同层支持的硬件板，我们可以运行：

```
$ ls sources/meta-fsl*/conf/machine/*.conf

```

要列出新引入的目标镜像，请使用以下命令：

```
$ ls sources/meta-fsl*/recipes*/images/*.bb

```

社区 Freescale BSP 发布引入了以下新目标镜像：

+   `fsl-image-mfgtool-initramfs`：这是一个小型的基于内存的 `initramfs` 镜像，用于与 Freescale 制造工具配合使用。

+   `fsl-image-multimedia`：这是一个仅限控制台的镜像，包含 `gstreamer` 多媒体框架，并通过帧缓冲区提供支持（如果适用）。

+   `fsl-image-multimedia-full`：这是 `fsl-image-multimedia` 的扩展，但扩展了 `gstreamer` 多媒体框架，包含所有可用的插件。

+   `fsl-image-machine-test`：这是 `fsl-image-multimedia-full` 的扩展，用于测试和基准测试。

+   `qte-in-use-image`：这是一个图形镜像，包含对通过帧缓冲区支持的 Qt4 的支持。

+   `qt-in-use-image`：这是一个图形镜像，包含对通过 X11 Windows 系统支持的 Qt4 的支持。

## 另见

+   使用 `repo` 工具的说明，包括在代理服务器下使用 `repo`，可以在 Android 文档中找到，网址为 [`source.android.com/source/downloading.html`](https://source.android.com/source/downloading.html)

# 构建 Wandboard 镜像

为支持的板（例如，`Wandboard Quad`）构建镜像的过程与我们之前描述的 QEMU 机器的过程相同，唯一的区别是使用了 `setup-environment` 脚本，它是 `oe-init-build-env` 的封装。

## 如何操作...

要为 `wandboard-quad` 机器构建镜像，使用以下命令：

```
$ cd /opt/yocto/fsl-community-bsp
$ mkdir -p wandboard-quad
$ MACHINE=wandboard-quad source setup-environment wandboard-quad
$ bitbake core-image-minimal

```

### 注意

当前版本的 `setup-environment` 脚本仅在 `build` 目录位于安装文件夹下时有效；在我们的例子中，即 `/opt/yocto/fsl-community-bsp`。

## 如何操作...

`setup-environment` 脚本将创建一个 `build` 目录，设置 `MACHINE` 变量，并提示你接受前面提到的 Freescale EULA。你的 `conf/local.conf` 配置文件将更新，包含指定的机器和 EULA 接受变量。

### 注意

请记住，如果你关闭了终端会话，你需要重新设置环境才能使用 BitBake。你可以安全地重新运行之前看到的 `setup-environment` 脚本，因为它不会修改现有的 `conf/local.conf` 文件。运行以下命令：

```
$ cd /opt/yocto/fsl-community-bsp/
$ source setup-environment wandboard-quad

```

生成的镜像 `core-image-minimal.sdcard`，它是在 `build` 目录下创建的，可以被编程到 microSD 卡中，然后插入到 Wandboard CPU 板的主槽中，并使用以下命令进行启动：

```
$ cd /opt/yocto/fsl-community-bsp/wandboard- quad/tmp/deploy/images/wandboard-quad/
$ sudo dd if=core-image-minimal.sdcard of=/dev/sdN bs=1M && sync

```

在这里，`/dev/sdN` 对应的是在你的主机系统中分配给 microSD 卡的设备节点。

### 提示

在运行 `dd` 命令时要小心，因为它可能会损坏你的机器。你必须确保 *sdN* 设备对应的是你的 microSD 卡，而不是开发机器上的驱动器。

## 另请参见

+   你可以在 Android 的文档中找到有关 `repo` 工具的更多信息，网址为 [`source.android.com/source/using-repo.html`](https://source.android.com/source/using-repo.html)

# 故障排除 Wandboard 的首次启动

如果你在启动镜像时遇到问题，按照这个方法进行故障排除。

## 准备工作

1.  在未插入 microSD 卡的情况下，将 microUSB 转 USB 数据线插入到 Wandboard 的 USB OTG 接口。检查你的 Linux 主机上的 `lsusb` 工具，查看 Wandboard 是否以以下方式显示：

    ```
    Bus 002 Device 006: ID 15a2:0054 Freescale Semiconductor, Inc. i.MX6Q SystemOnChip in RecoveryMode
    ```

    如果你没有看到此内容，尝试更换电源。电源应为 5V，10W。

1.  确保你使用 NULL 调制解调器串行电缆连接 Wandboard 目标板上的 RS232 接口和你的 Linux 主机的串口。然后，使用以下命令打开一个终端程序，如 minicom：

    ```
    $ minicom -D /dev/ttyS0 -b 115200

    ```

    ### 注意

    你需要将你的用户添加到 dialout 组，或者尝试以 sudo 身份运行命令。这应该会打开一个 115200 8N1 的串口连接。串口设备在你的 Linux 主机中可能有所不同。例如，USB 转串口适配器可能会被识别为`/dev/ttyUSB0`。同时，确保硬件和软件流控制都已禁用。

## 如何操作...

1.  将 microSD 卡插入模块插槽，而不是基板插槽，因为后者仅用于存储而不是启动，然后为其供电。你应该在 minicom 会话的输出中看到 U-Boot 横幅。

1.  如果没有，你可能会遇到串口通信问题。默认情况下，FSL 社区 BSP 映像中的以太网接口配置为通过 DHCP 请求地址，因此你可以用它来连接到目标。

    确保在测试网络中目标设备所在的网络上运行 DHCP 服务器。

    你可以使用像**Wireshark**这样的包嗅探器，在 Linux 主机上捕获网络跟踪并过滤类似`bootp`协议的包。至少，你应该看到目标设备的一些广播，如果你使用以太网集线器，你还应该看到 DHCP 回复。

    可选地，你可以登录到你的 DHCP 服务器并检查日志，看是否分配了新的 IP 地址。如果你看到 IP 地址被分配，你可能想考虑向你的 core-image-minimal 映像中添加 SSH 服务器，如**Dropbear**，这样你就可以与目标建立网络连接。你可以通过将以下行添加到`conf/local.conf`配置文件来完成：

    ```
    IMAGE_INSTALL_append = " dropbear"
    ```

    注意初始引号后的空格。

    构建并重新编程后，你可以从 Linux 主机启动一个 SSH 会话到 Wandboard，命令如下：

    ```
    $ ssh root@<ip_address>

    ```

    连接应该会自动登录，而无需密码提示。

1.  尝试从[`www.wandboard.org/index.php/downloads`](http://www.wandboard.org/index.php/downloads)编程默认的 microSD 卡映像，以确保硬件和你的设置有效。

1.  尝试重新编程你的 microSD 卡。确保你使用的是正确的映像（例如，不要混合双核和四核映像）。同时，尝试使用不同的卡和卡读卡器。

这些步骤将使你的 Wandboard 开始启动，你应该在串口连接中看到一些输出。

## 还有更多...

如果其他方法都失败了，你可以验证启动加载程序在 microSD 卡上的位置。你可以通过以下命令转储 microSD 卡前几个块的内容：

```
$ sudo dd if=/dev/sdN of=/tmp/sdcard.img count=10

```

你应该在偏移量 0x400 处看到一个 U-Boot 头部。这个偏移量是 i.MX6 启动 ROM 在从 microSD 接口引导时查找引导加载程序的位置。使用以下命令：

```
$ head /tmp/sdcard.img | hexdump
0000400 00d1 4020 0000 1780 0000 0000 f42c 177f

```

你可以通过转储构建的 U-Boot 映像来识别 U-Boot 头部。运行以下命令：

```
$ head u-boot-wandboard-quad.imx | hexdump
0000000 00d1 4020 0000 1780 0000 0000 f42c 177f

```

# 为开发环境配置网络启动

大多数专业的 i.MX6 开发板都具有内置的 **嵌入式 MMC** (**eMMC**) 闪存，这是启动固件的推荐方式。Wandboard 其实并不是一个为专业用途设计的产品，因此它没有此功能。但无论是 eMMC 还是 microSD 卡都不适合用于开发工作，因为任何系统更改都会涉及重新编程固件镜像。

## 准备就绪

开发工作的理想设置是，在您的主机系统中同时使用 TFTP 和 NFS 服务器，并且仅在 eMMC 或 microSD 卡中存储 U-Boot 启动加载程序。通过这种设置，启动加载程序将从 TFTP 服务器获取 Linux 内核，而内核将从 NFS 服务器挂载根文件系统。对内核或根文件系统的更改可以在不重新编程的情况下进行。只有启动加载程序的开发工作需要重新编程物理介质。

### 安装 TFTP 服务器

如果您尚未运行 TFTP 服务器，请按照以下步骤在您的 Ubuntu 14.04 主机上安装和配置 TFTP 服务器：

```
$ sudo apt-get install tftpd-hpa

```

`tftpd-hpa` 配置文件安装在 `/etc/default/tftpd-hpa` 中。默认情况下，它使用 `/var/lib/tftpboot` 作为根 `TFTP` 文件夹。使用以下命令更改文件夹权限，使其对所有用户可访问：

```
$ sudo chmod 1777 /var/lib/tftpboot

```

现在，按如下方式将 Linux 内核和设备树从您的 `build` 目录复制过来：

```
$ cd /opt/yocto/fsl-community-bsp/wandboard- quad/tmp/deploy/images/wandboard-quad/
$ cp zImage-wandboard-quad.bin zImage-imx6q-wandboard.dtb /var/lib/tftpboot

```

### 安装 NFS 服务器

如果您尚未运行 NFS 服务器，请按照以下步骤在您的 Ubuntu 14.04 主机上安装和配置 NFS 服务器：

```
$ sudo apt-get install nfs-kernel-server

```

我们将使用 `/nfsroot` 目录作为 NFS 服务器的根目录，因此我们将在此目录中“解压”目标的根文件系统，从我们的 Yocto `build` 目录中：

```
$ sudo mkdir /nfsroot
$ cd /nfsroot
$ sudo tar xvf /opt/yocto/fsl-community-bsp/wandboard- quad/tmp/deploy/images/wandboard-quad/core-image-minimal-wandboard- quad.tar.bz2

```

接下来，我们将配置 NFS 服务器来导出 `/nfsroot` 文件夹：

```
/etc/exports:
/nfsroot/ *(rw,no_root_squash,async,no_subtree_check)

```

然后，我们将重新启动 NFS 服务器，以使配置更改生效：

```
$ sudo service nfs-kernel-server restart

```

## 如何操作...

启动 Wandboard，并通过在串口控制台按任意键停止在 U-Boot 提示符处。然后执行以下步骤：

1.  通过 DHCP 获取 IP 地址：

    ```
    > dhcp

    ```

    另外，您可以使用以下命令配置静态 IP 地址：

    ```
    > setenv ipaddr <static_ip>

    ```

1.  配置您的主机系统的 IP 地址，TFTP 和 NFS 服务器已在此系统上设置：

    ```
    > setenv serverip <host_ip>

    ```

1.  配置根文件系统挂载：

    ```
    > setenv nfsroot /nfsroot

    ```

1.  配置 Linux 内核和设备树文件名：

    ```
    > setenv image zImage-wandboard-quad.bin
    > setenv fdt_file zImage-imx6q-wandboard.dtb

    ```

1.  如果您已经配置了静态 IP 地址，您需要通过运行以下命令禁用 DHCP 启动：

    ```
    > setenv ip_dyn no

    ```

1.  将 U-Boot 环境保存到 microSD 卡：

    ```
    > saveenv

    ```

1.  执行网络启动：

    ```
    > run netboot

    ```

Linux 内核和设备树将从 TFTP 服务器获取，而根文件系统将在从您的网络获取 DHCP 地址后由内核从 NFS 共享挂载（除非使用静态 IP 地址）。

您应该能够使用 root 用户登录，而无需密码提示。

# 共享下载

您通常会同时处理多个项目，可能针对不同的硬件平台或不同的目标镜像。在这种情况下，通过共享 `downloads` 来优化构建时间是非常重要的。

## 准备就绪

构建系统会在多个地方搜索下载的源代码：

+   它会尝试使用本地的`downloads`文件夹。

+   它会检查配置的预镜像，这些镜像通常是本地的，属于你的组织。

+   然后，它会尝试从在包配方中配置的上游源获取数据。

+   最后，它会检查配置的镜像。镜像是源代码的公共备用位置。

如果在这四个地方都没有找到包源，包构建将会失败并报错。当从上游获取失败并尝试镜像时，还会发出构建警告，以便可以查看上游问题。

Yocto 项目维护了一组镜像，以将构建系统与上游服务器的问题隔离开来。然而，在添加外部层时，你可能会添加一些不在 Yocto 项目镜像服务器或其他配置镜像中的包，因此建议你保持一个本地预镜像，以避免源代码不可用的问题。

新项目的默认 Poky 设置是将下载的包源存储在当前的`build`目录中。这是构建系统首先搜索源`downloads`的地方。这个设置可以在项目的`conf/local.conf`文件中通过`DL_DIR`配置变量来进行配置。

## 如何操作...

为了优化构建时间，建议在所有项目之间共享一个`downloads`目录。`meta-fsl-arm`层的`setup-environment`脚本会将默认的`DL_DIR`更改为`repo`工具创建的`fsl-community-bsp`目录。在这种设置下，`downloads`文件夹将在主机系统中的所有项目之间共享。配置如下：

```
DL_DIR ?= "${BSPDIR}/downloads/"
```

一种更具可扩展性的设置（例如，适用于远程分布的团队）是配置一个预镜像。例如，可以在你的`conf/local.conf`文件中添加以下内容：

```
INHERIT += "own-mirrors"
SOURCE_MIRROR_URL = "http://example.com/my-source-mirror"
```

一种常见的设置是让构建服务器提供它的`downloads`目录。构建服务器可以配置为准备 Git 目录的 tarball 文件，以避免从上游服务器执行 Git 操作。这个设置会影响构建性能，但通常在构建服务器中是可以接受的。添加以下内容：

```
BB_GENERATE_MIRROR_TARBALLS = "1"
```

这种设置的一个优势是，构建服务器的`downloads`文件夹也可以备份，以保证未来产品的源代码可用性。这在需要长期可用性的嵌入式产品中尤为重要。

为了测试这个设置，你可以检查是否仅通过使用预镜像就能进行构建，方法如下：

```
BB_FETCH_PREMIRRORONLY = "1"
```

这个设置可以通过在项目创建过程中使用`TEMPLATECONF`变量，将`conf/local.conf`文件中的设置分发到团队成员。

# 共享共享状态缓存

Yocto 项目从源代码开始构建一切。当你创建一个新项目时，只会创建配置文件。然后，构建过程会从头开始编译所有内容，包括交叉编译工具链和一些对构建重要的本地工具。

这个过程可能需要很长时间，Yocto 项目实现了一个共享状态缓存机制，用于增量构建，旨在仅为给定更改构建严格必要的组件。

为了使这一点生效，构建系统会计算给定任务的输入数据的校验和。如果输入数据发生变化，则需要重新构建该任务。简单来说，构建过程为每个任务生成一个可以进行校验和对比的运行脚本。它还会跟踪任务的输出，以便可以重用。

一个包食谱可以修改共享状态缓存到某个任务；例如，通过将其标记为 `nostamp` 来强制始终重新构建。有关共享状态缓存机制的更深入解释，请参见 *Yocto 项目参考手册*，网址为 [`www.yoctoproject.org/docs/1.7.1/ref-manual/ref-manual.html`](http://www.yoctoproject.org/docs/1.7.1/ref-manual/ref-manual.html)。

## 如何操作……

默认情况下，构建系统将在你的 `build` 目录中使用一个名为 `sstate-cache` 的共享状态缓存目录来存储缓存数据。你可以通过在 `conf/local.conf` 文件中配置 `SSTATE_DIR` 来更改这一点。缓存数据存储在以哈希值前两位字符命名的目录中。目录内的文件名包含完整的任务校验和，因此可以通过查看文件名来确定缓存的有效性。构建过程设置场景任务将评估缓存数据并在有效时使用它来加速构建。

当你想从一个干净的状态开始构建时，你需要删除 `sstate-cache` 目录和 `tmp` 目录。

你还可以通过在运行 BitBake 时使用 `--no-setscene` 参数，指示 BitBake 忽略共享状态缓存。

备份干净的共享状态缓存是一个好习惯（例如，从构建服务器备份），以防共享状态缓存损坏时可以使用。

## 还有更多内容……

共享共享状态缓存是可能的；然而，这需要小心处理。并非所有更改都会被共享状态缓存实现检测到，当发生这种情况时，部分或所有缓存需要失效。这可能会在共享状态缓存被共享时引发问题。

在这种情况下的建议取决于使用场景。致力于 Yocto 元数据的开发人员应将共享状态缓存保持为默认设置，并按项目分开存储。

然而，验证和测试工程师、内核和引导加载程序开发人员以及应用程序开发人员可能会从一个维护良好的共享状态缓存中受益。

要配置一个 NFS 共享驱动器供开发团队共享以加速构建，你可以将以下内容添加到你的 `conf/local.conf` 配置文件中：

```
SSTATE_MIRRORS ?= "\
     file://.* file:///nfs/local/mount/sstate/PATH"
```

这个例子中的 `PATH` 表达式将被构建系统替换为一个以哈希值的前两个字符命名的目录。

# 设置软件包源

嵌入式系统项目很少需要对 Yocto 构建系统进行更改。大部分时间和精力都花费在应用程序开发上，接下来是可能的内核和引导加载程序开发。

因此，整个系统的重建可能很少进行。一个新项目通常是从一个预构建的共享状态缓存中构建的，应用程序开发工作仅需对少数几个包执行完整或增量构建。

一旦软件包构建完成，它们需要安装到目标系统上进行测试。模拟机对于应用程序开发来说是可以的，但大多数与硬件相关的工作需要在嵌入式硬件上完成。

## 准备就绪

一种选择是手动将构建的二进制文件复制到目标的根文件系统中，可以将其复制到目标从主机系统挂载其根文件系统的 NFS 共享中（如之前在*为开发环境配置网络启动*配方中所述），或者使用 SCP、FTP，甚至 microSD 卡等其他方法。

这个方法也被像 Eclipse 这样的集成开发环境（IDE）用于调试你正在开发的应用程序。然而，当你需要安装多个软件包和依赖项时，这种方法的扩展性较差。

下一个选项是将打包后的二进制文件（即 RPM、deb 或 ipk 包）复制到目标的文件系统中，然后使用目标的包管理系统进行安装。要使此方法有效，目标的文件系统需要内建包管理工具。做到这一点很简单，只需将 `package-management` 功能添加到根文件系统中；例如，你可以将以下行添加到项目的 `conf/local.conf` 文件中：

```
EXTRA_IMAGE_FEATURES += "package-management"
```

所以对于 RPM 包，你将它复制到目标设备上，并使用 **rpm** 或 **smart** 工具来安装它。smart 包管理工具是 GPL 许可证的，可以与多种包格式一起使用。

然而，最优化的方法是将主机系统包的输出目录转换为一个包源。例如，如果你使用的是默认的 RPM 包格式，你可以将 `build` 目录中的 `tmp/deploy/rpm` 转换为目标可以用来更新的包源。

为了使这一切正常工作，你需要在你的计算机上配置一个 HTTP 服务器来提供软件包。

### 版本管理软件包

你还需要确保生成的包具有正确的版本，这意味着每次更改时都需要更新配方修订版本，**PR**。虽然可以手动完成此操作，但推荐的—也是如果你想使用软件包源时必须使用的—方法是使用 PR 服务器。

然而，PR 服务器默认是未启用的。没有 PR 服务器生成的软件包彼此之间是一致的，但对于已运行的系统没有更新保证。

最简单的 PR 服务器配置是将其本地运行在你的主机系统上。为此，你需要将以下内容添加到你的`conf/local.conf`文件中：

```
PRSERV_HOST = "localhost:0"
```

使用此设置，可以保证更新的一致性。

如果你想与其他开发者共享你的软件包源，或者你正在配置构建服务器或软件包服务器，可以通过运行以下命令来运行一个 PR 服务器的单实例：

```
$ bitbake-prserv --host <server_ip> --port <port> --start

```

然后，你将更新项目的构建配置，使用集中式 PR 服务器，并编辑`conf/local.conf`如下：

```
PRSERV_HOST = "<server_ip>:<port>"
```

此外，如果你之前使用了共享状态缓存，所有共享状态缓存的贡献者都需要使用相同的 PR 服务器。

一旦确保了软件包源的完整性，我们需要配置一个 HTTP 服务器来提供该源。

## 如何操作...

我们将在本示例中使用`lighttpd`，因为它轻量且易于配置。按照以下步骤操作：

1.  安装 Web 服务器：

    ```
    $ sudo apt-get install lighttpd

    ```

1.  默认情况下，`/etc/lighttpd/lighttpd.conf`配置文件中指定的文档根目录为`/var/www/`，因此我们只需要为我们的软件包源创建一个符号链接：

    ```
    $ sudo mkdir /var/www/wandboard-quad
    $ sudo ln -s /opt/yocto/fsl-community-bsp/wandboard- quad/tmp/deploy/rpm /var/www/wandboard-quad/rpm

    ```

    接下来，按如下方式重新加载配置：

    ```
    $ sudo service lighttpd reload

    ```

1.  刷新软件包索引。这需要手动执行，以便在每次构建后更新软件包源：

    ```
    $ bitbake package-index

    ```

1.  然后，我们需要使用新的软件包源配置目标文件系统：

    ```
    # smart channel --add all type=rpm-md \ baseurl=http://<server_ip>/wandboard-quad/rpm/all

    # smart channel --add wandboard_quad type=rpm-md \ baseurl=http://<server_ip>/wandboard-quad/rpm/wandboard_quad

    # smart channel --add cortexa9hf_vfp_neon type=rpm-md \ baseurl=http://<server_ip>/wandboard- quad/rpm/cortexa9hf_vfp_neon

    ```

1.  一旦设置完成，我们将能够通过以下方式查询和更新目标根文件系统中的软件包：

    ```
    # smart update
    # smart query <package_name>
    # smart install <package_name>

    ```

为了使此更改在目标的根文件系统中保持持久性，我们可以在编译时通过在`conf/local.conf`中使用`PACKAGE_FEED_URIS`变量来配置软件包源，如下所示：

```
PACKAGE_FEED_URIS = "http://<server_ip>/wandboard-quad"
```

## 另见

+   更多信息和智能工具的用户手册可以在[`labix.org/smart/`](https://labix.org/smart/)找到。

# 使用构建历史

在维护嵌入式产品的软件时，你需要一种方法来了解什么内容发生了变化，以及它将如何影响你的产品。

在 Yocto 系统上，你可能需要更新软件包修订版本（例如，修复安全漏洞），并且你需要确保这项更改的影响；例如，软件包依赖关系和根文件系统的更改。

构建历史让你能够做到这一点，我们将在本教程中详细探讨。

## 如何操作...

要启用构建历史，请将以下内容添加到你的`conf/local.conf`文件中：

```
INHERIT += "buildhistory"
```

以下配置启用了信息收集，包括依赖关系图：

```
BUILDHISTORY_COMMIT = "1"
```

上述代码行启用了将构建历史存储到本地 Git 仓库的功能。

Git 仓库的位置可以通过`BUILDHISTORY_DIR`变量来设置，默认情况下，该变量设置为`build`目录下的`buildhistory`目录。

默认情况下，`buildhistory`跟踪包、镜像和 SDK 的变化。通过`BUILDHISTORY_FEATURES`变量可以进行配置。例如，仅跟踪镜像的变化，可以在`conf/local.conf`中添加如下内容：

```
BUILDHISTORY_FEATURES = "image"
```

它还可以跟踪特定文件，并将其复制到`buildhistory`目录。默认情况下，这仅包括`/etc/passwd`和`/etc/groups`，但它也可以用于跟踪任何重要文件，如安全证书。需要在你的`conf/local.conf`文件中使用`BUILDHISTORY_IMAGE_FILES`变量添加这些文件，如下所示：

```
BUILDHISTORY_IMAGE_FILES += "/path/to/file"
```

构建历史会减慢构建速度，增加构建大小，并可能使 Git 目录变得无法管理。建议在构建服务器上启用它，用于软件发布，或在特定情况下启用，比如更新生产环境中的软件。

## 它是如何工作的...

启用时，它会以 Git 仓库的形式记录每个包和镜像的变化，且可以被探索和分析。

对于一个包，它记录以下信息：

+   包和配方修订版

+   依赖关系

+   包的大小

+   文件

对于一个镜像，它记录以下信息：

+   构建配置

+   依赖关系图

+   包含文件列表，其中包括所有权和权限

+   已安装包的列表

对于 SDK，它记录以下信息：

+   SDK 配置

+   主机和目标文件的列表，包括所有权和权限

+   依赖关系图

+   已安装包的列表

### 查看构建历史

使用构建历史检查 Git 目录有多种方式：

+   使用像 gitk 或 git log 这样的 Git 工具。

+   使用**buildhistory-diff**命令行工具，显示人类可读格式的差异。

+   使用基于 Django-1.4 的 Web 界面。每次构建后，你需要将构建历史数据导入到应用程序的数据库中。详细信息请参阅[`git.yoctoproject.org/cgit/cgit.cgi/buildhistory-web/tree/README`](http://git.yoctoproject.org/cgit/cgit.cgi/buildhistory-web/tree/README)。

## 还有更多...

为了维护构建历史，重要的是优化它并避免其随着时间增长。定期备份构建历史并清理旧数据，对于保持构建历史仓库的可管理大小至关重要。

一旦`buildhistory`目录已被备份，以下过程将修剪它并保留最新的历史：

1.  将你的仓库复制到临时的 RAM 文件系统（`tmpfs`）中以加速操作。使用`df -h`命令检查输出，查看哪些目录是`tmpfs`文件系统，并查看它们的可用空间，然后选择一个。例如，在 Ubuntu 中，`/run/shm`目录是可用的。

1.  为一个没有父节点的提交添加一个一个月前的合并点：

    ```
    $ git rev-parse "HEAD@{1 month ago}" > .git/info/grafts

    ```

1.  使合并点永久生效：

    ```
    $ git filter-branch

    ```

1.  克隆一个新仓库以清理剩余的 Git 对象：

    ```
    $ git clone file://${tmpfs}/buildhistory buildhistory.new

    ```

1.  用新的清理过的`buildhistory`目录替换旧的目录：

    ```
    $ rm -rf buildhistory
    $ mv buildhistory.new buildhistory

    ```

# 使用构建统计信息

构建系统可以按任务和镜像收集构建信息。数据可用于识别构建时间的优化领域和瓶颈，特别是当系统中添加新的配方时。本配方将解释构建统计信息的工作原理。

## 如何执行……

要启用统计数据收集，您的项目需要通过将 `buildstats` 类添加到 `conf/local.conf` 文件中的 `USER_CLASSES` 来继承它。默认情况下，`fsl-community-bsp` 构建项目已配置为启用它们。

```
USER_CLASSES ?= "buildstats"

```

您可以使用 `BUILDSTATS_BASE` 变量配置这些统计信息的位置，默认情况下，它被设置为 `buildstats` 文件夹，该文件夹位于 `build` 目录下的 `tmp` 目录中（`tmp/buildstats`）。

`buildstats` 文件夹包含每个镜像的文件夹，镜像下有一个 `timestamp` 文件夹，里面会有每个包的子目录，并且会包含一个 `build_stats` 文件，其中包含：

+   主机系统信息

+   根文件系统的位置和大小

+   构建时间

+   平均 CPU 使用率

+   磁盘统计信息

## 它是如何工作的……

数据的准确性取决于下载目录 `DL_DIR` 和共享状态缓存目录 `SSTATE_DIR` 是否存在于同一分区或卷中，因此如果您计划使用构建数据，可能需要相应地配置它们。

一个示例 `build-stats` 文件如下所示：

```
Host Info: Linux agonzal 3.13.0-35-generic #62-Ubuntu SMP Fri Aug 15 01:58:42 UTC 2014 x86_64 x86_64
Build Started: 1411486841.52
Uncompressed Rootfs size: 6.2M  /opt/yocto/fsl-community- bsp/wandboard-quad/tmp/work/wandboard_quad-poky-linux- gnueabi/core-image-minimal/1.0-r0/rootfs
Elapsed time: 2878.26 seconds
CPU usage: 51.5%
EndIOinProgress: 0
EndReadsComp: 0
EndReadsMerged: 55289561
EndSectRead: 65147300
EndSectWrite: 250044353
EndTimeIO: 14415452
EndTimeReads: 10338443
EndTimeWrite: 750935284
EndWTimeIO: 816314180
EndWritesComp: 0
StartIOinProgress: 0
StartReadsComp: 0
StartReadsMerged: 52319544
StartSectRead: 59228240
StartSectWrite: 207536552
StartTimeIO: 13116200
StartTimeReads: 8831854
StartTimeWrite: 3861639688
StartWTimeIO: 3921064032
StartWritesComp: 0
```

这些磁盘统计信息来自 Linux 内核磁盘 I/O 统计信息（[`www.kernel.org/doc/Documentation/iostats.txt`](https://www.kernel.org/doc/Documentation/iostats.txt)）。不同的元素在这里进行了说明：

+   `ReadsComp`：这是已完成的读取操作总数

+   `ReadsMerged`：这是合并的相邻读取的总数

+   `SectRead`：这是读取的扇区总数

+   `TimeReads`：这是读取操作花费的总毫秒数

+   `WritesComp`：这是已完成的写入操作的总数

+   `SectWrite`：这是已写入的扇区总数

+   `TimeWrite`：这是写入操作花费的总毫秒数

`IOinProgress`：这是读取 `/proc/diskstats` 时正在进行的 I/O 操作总数

+   `TimeIO`：这是执行 I/O 操作所花费的总毫秒数

+   `WTimeIO`：这是执行 I/O 时所花费的加权时间总数

在每个包内，我们会列出一些任务；例如，对于 `ncurses-5.9-r15.1`，我们有以下任务：

+   `do_compile`

+   `do_fetch`

+   `do_package`

+   `do_package_write_rpm`

+   `do_populate_lic`

+   `do_rm_work`

+   `do_configure`

+   `do_install`

+   `do_packagedata`

+   `do_patch`

+   `do_populate_sysroot`

+   `do_unpack`

每个任务都包含以下内容，格式与之前相同：

+   构建时间

+   CPU 使用率

+   磁盘统计信息

## 还有更多……

您还可以使用 Poky 源代码中包含的 `pybootchartgui.py` 工具获取数据的图形表示。从您的项目的 `build` 文件夹中，您可以执行以下命令，在 `/tmp` 中获取 `bootchart.png` 图形：

```
$ ../sources/poky/scripts/pybootchartgui/pybootchartgui.py tmp/buildstats/core-image-minimal-wandboard-quad/ -o /tmp

```

# 调试构建系统

在本章的最后一个食谱中，我们将探索可用于调试构建系统及其元数据问题的不同方法。

## 准备工作

让我们首先介绍一些调试会话中常见的用例。

### 查找食谱

检查当前层中是否支持某个特定包的一种好方法是如下搜索：

```
$ find -name "*busybox*"

```

这将递归地搜索所有层，查找 BusyBox 模式。你可以通过执行以下命令将搜索限制在食谱和附加文件中：

```
$ find -name "*busybox*.bb*"

```

### 转储 BitBake 的环境

在开发或调试包或镜像食谱时，通常会要求 BitBake 列出其环境，既可以是全局环境，也可以是特定目标的环境，无论是包还是镜像。

要转储全局环境并使用 `grep` 查找感兴趣的变量（例如，`DISTRO_FEATURES`），请使用以下命令：

```
$ bitbake -e | grep -w DISTRO_FEATURES

```

可选地，若要定位某个特定包食谱（如 BusyBox）的源目录，请使用以下命令：

```
$ bitbake -e busybox | grep ^S=

```

你还可以执行以下命令来定位某个包或镜像食谱的工作目录：

```
$ bitbake -e <target> | grep ^WORKDIR=

```

### 使用开发 shell

BitBake 提供了 `devshell` 任务来帮助开发者。它可以通过以下命令执行：

```
$ bitbake -c devshell <target>

```

它将解包并修补源代码，并在目标源目录中打开一个新的终端（它会自动检测你的终端类型，或者可以通过 `OE_TERMINAL` 设置），该目录的环境已正确设置。

### 注意

在图形环境中，devshell 会打开一个新的终端或控制台窗口，但如果我们在非图形环境中工作，如 telnet 或 SSH，你可能需要在 `conf/local.conf` 配置文件中指定 `screen` 作为终端，如下所示：

```
OE_TERMINAL = "screen"
```

在 devshell 中，你可以运行开发命令如 `configure` 和 `make`，或直接调用交叉编译器（使用已设置好的 `$CC` 环境变量）。

## 如何操作...

调试包构建错误的起点是 BitBake 在构建过程中打印的错误信息。通常这会指向我们未能成功构建的任务。

要列出给定食谱中所有可用的任务及其描述，请执行以下命令：

```
$ bitbake -c listtasks <target>

```

如果你需要重现错误，可以使用以下命令强制进行构建：

```
$ bitbake -f <target>

```

或者你可以要求 BitBake 强制只运行特定任务，使用以下命令：

```
$ bitbake -c compile -f <target>

```

### 任务日志和运行文件

为了调试构建错误，BitBake 会为每个 shell 任务创建两种有用的文件，并将它们存储在工作目录的 `temp` 文件夹中。以 BusyBox 为例，我们可以查看：

```
 /opt/yocto/fsl-community-bsp/wandboard-quad/tmp/work/cortexa9hf- vfp-neon-poky-linux-gnueabi/busybox/1.22.1-r32/temp
```

然后找到日志和运行文件的列表。文件名格式为：

`log.do_<task>.<pid>`

以及 `run.do_<task>.<pid>`。

幸运的是，我们还有符号链接，去掉了 `pid` 部分，链接到最新版本。

日志文件将包含任务的输出，通常这是我们调试问题所需的唯一信息。运行文件包含 BitBake 执行的实际代码，用来生成之前提到的日志。这仅在调试复杂构建问题时需要。

另一方面，Python 任务目前并不会像之前描述的那样写入文件，尽管未来计划这样做。Python 任务是在内部执行并将信息记录到终端。

### 为配方添加日志记录

BitBake 配方接受 bash 或 Python 代码。Python 日志记录通过 `bb` 类完成，并使用标准的 Python 日志库模块。它包含以下组件：

+   `bb.plain`：这个函数使用 `logger.plain`。它可以用于调试，但不应提交到源代码中。

+   `bb.note`：这个函数使用 `logger.info`。

+   `bb.warn`：这个函数使用 `logger.warn`。

+   `bb.error`：这个函数使用 `logger.error`。

+   `bb.fatal`：这个函数使用 `logger.critical` 并退出 BitBake。

+   `bb.debug`：此函数应该作为第一个参数传递日志级别，并使用 `logger.debug`。

为了在我们的配方中打印来自 bash 的调试输出，我们需要通过执行以下命令来使用 `logging` 类：

```
inherit logging
```

`logging` 类默认由所有包含 `base.bbclass` 的配方继承，因此我们通常不需要显式继承它。继承后，我们将可以访问以下 bash 函数，这些函数会将输出记录到之前描述过的工作目录内 `temp` 目录中的日志文件（而非控制台）：

+   `bbplain`：这个函数字面输出传递给它的内容。它可以用于调试，但不应提交到配方源代码中。

+   `bbnote`：这个函数会以 `NOTE` 前缀打印信息。

+   `bbwarn`：这个函数打印一个带有 `WARNING` 前缀的非致命警告。

+   `bberror`：这会打印一个带有`ERROR`前缀的非致命错误。

+   `bbfatal`：该函数会中止构建并打印错误消息，类似于 `bberror`。

+   `bbdebug`：这个函数根据作为第一个参数传递的日志级别打印调试消息。使用格式如下：

    ```
    bbdebug [123] "message"
    ```

    ### 提示

    这里提到的 bash 函数不会记录到控制台，只会记录到日志文件中。

### 查看依赖关系

你可以通过以下命令让 BitBake 打印当前及提供的包版本：

```
$ bitbake --show-versions

```

另一个常见的调试任务是删除不需要的依赖项。

要查看拉取的依赖项概览，可以通过运行以下命令来使用 BitBake 的详细输出：

```
$ bitbake -v <target>

```

要分析一个包所拉取的依赖项，我们可以让 BitBake 创建描述这些依赖关系的 DOT 文件，通过运行以下命令：

```
$ bitbake -g <target>

```

DOT 格式是一种图形描述的文本语言，可以被 **GraphViz** 开源包及其所有使用该包的工具理解。DOT 文件可以被可视化或进一步处理。

你可以从图中省略依赖项，以产生更易读的输出。例如，要省略来自 `glibc` 的依赖项，可以运行以下命令：

```
$ bitbake -g <target> -I glibc

```

一旦执行了前面的命令，我们将在当前目录中得到三个文件：

+   `package-depends.dot`：此文件显示了运行时目标之间的依赖关系

+   `pn-depends.dot`：此文件显示了食谱之间的依赖关系

+   `task-depends.dot`：此文件显示了任务之间的依赖关系

还有一个 `pn-buildlist` 文件，其中列出了给定目标将要构建的包。

要将 `.dot` 文件转换为 PostScript 文件（`.ps`），你可以执行：

```
$ dot -Tps filename.dot -o outfile.ps

```

然而，显示依赖数据的最有用方式是让 BitBake 通过依赖关系探索器以图形方式显示它，如下所示：

```
$ bitbake -g -u depexp <target>

```

结果可以在以下截图中看到：

![查看依赖关系](img/5186OS_01_01.jpg)

### 调试 BitBake

调试 BitBake 本身并不常见，但你可能会在 BitBake 中发现一个 bug，并希望在报告给 BitBake 社区之前自己探索它。对于这种情况，你可以使用 `-D` 标志要求 BitBake 输出三种不同级别的调试信息。要显示所有调试信息，请运行以下命令：

```
$ bitbake -DDD <target>

```

### 错误报告工具

有时候，你会在没有修改的 Yocto 食谱中发现构建错误。检查错误的第一个地方是社区本身，但在启动邮件客户端之前，先访问 [`errors.yoctoproject.org`](http://errors.yoctoproject.org)。

这是一个用户报告错误的中央数据库。在这里，你可以检查是否有其他人也遇到相同的问题。

你可以将自己的构建失败提交到数据库，帮助社区调试问题。为此，你可以使用 `report-error` 类。在你的 `conf/local.conf` 文件中添加以下内容：

```
INHERIT += "report-error"
```

默认情况下，错误信息存储在 `build` 目录下的 `tmp/log/error-report` 中，但你可以通过 `ERR_REPORT_DIR` 变量设置一个特定的位置。

当错误报告工具被激活时，构建错误将被捕获到 `error-report` 文件夹中的一个文件里。构建输出还会打印出一个命令，用于将错误日志发送到服务器：

```
$ send-error-report ${LOG_DIR}/error-report/error-report_${TSTAMP}

```

当执行此命令时，它将返回一个链接，指向上游错误。

你可以设置一个本地错误服务器，并通过传递服务器参数来使用它。错误服务器的代码和设置详细信息可以在 [`git.yoctoproject.org/cgit/cgit.cgi/error-report-web/tree/README`](http://git.yoctoproject.org/cgit/cgit.cgi/error-report-web/tree/README) 找到。

## 还有更多……

虽然你可以使用 Linux 工具来解析 Yocto 的元数据和构建输出，但 BitBake 缺少一个用于常见任务的命令行基础 UI。一个旨在提供此功能的项目是 `bb`，可以在 [`github.com/kergoth/bb`](https://github.com/kergoth/bb) 找到。

要使用它，你需要通过执行以下命令将仓库克隆到本地：

```
$ cd /opt/yocto/fsl-community-bsp/sources
$ git clone https://github.com/kergoth/bb.git

```

然后运行 `bb/bin/bb init` 命令，它会提示你将一个 bash 命令添加到 `~/.bash_profile` 文件中。

你可以选择执行该操作，或者在当前的 shell 中按如下方式执行：

```
$ eval "$(/opt/yocto/fsl-community-bsp/sources/bb/bin/bb init -)"

```

你需要像往常一样先设置好环境：

```
$ cd /opt/yocto/fsl-community-bsp
$ source setup-environment wandboard-quad

```

### 注意

有些命令只有在工作目录已填充时才能使用，因此如果你想使用 `bb`，可能需要删除 `rm_work` 类。

`bb` 工具可以简化的部分任务包括：

+   探索包的内容：

    ```
    $ bb contents <target>

    ```

+   在配方中搜索某个模式：

    ```
    $ bb search <pattern>

    ```

+   显示全局 BitBake 环境或某个特定包的环境，并使用 grep 查找特定变量：

    ```
    $ bb show -r <recipe> <variable>

    ```
