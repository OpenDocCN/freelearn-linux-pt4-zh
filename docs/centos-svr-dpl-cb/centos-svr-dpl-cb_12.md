# 第十二章：虚拟化

本章包含以下配方：

+   创建新的虚拟机

+   克隆虚拟机

+   向虚拟机添加存储

+   连接 USB 外部设备到来宾系统

+   配置来宾的网络接口

# 简介

本章的配方重点介绍如何在 CentOS 系统上使用虚拟化运行第二个操作系统作为来宾操作系统。你将学习如何设置虚拟机来安装来宾操作系统，如何通过克隆正确创建虚拟机的副本，并添加额外的存储资源。你还将学习如何共享对连接到宿主系统的 USB 外设的访问权限，并配置来宾的虚拟网络接口来访问网络。

# 创建新的虚拟机

本章节将教你如何安装 KVM 虚拟化软件并创建一个新的虚拟机。虚拟化使我们能够通过在同一物理系统上运行多个操作系统来利用可用的硬件资源。主操作系统是裸金属安装的，称为宿主操作系统（Host OS）。然后，安装特殊软件，使宿主操作系统能够提供硬件资源的仿真或直接访问。这些资源被分配为虚拟机，并且可以在宿主操作系统上安装并运行多个来宾操作系统，每个操作系统在其自己的虚拟机中运行。

## 准备工作

本配方需要一个具有工作网络连接和已安装图形用户界面的 CentOS 系统（请参阅 第一章中的 *安装 GNOME 桌面* 和 *安装 KDE Plasma 桌面* 配方）。还需要管理员权限，可以通过登录 `root` 账户或使用 `sudo` 获得。

## 如何操作...

按照以下步骤安装来宾操作系统：

1.  使用软件包组安装必要的虚拟化包：

    ```
    yum groupinstall "Virtualization Platform"   
           "Virtualization Client" "Virtualization Tools"

    ```

1.  启动虚拟机管理器应用程序：

    ```
     virt-manager

    ```

1.  通过从 **文件** 菜单选择 **新建虚拟机** 来创建新的虚拟机。这将打开 **新虚拟机** 向导。

1.  选择所需的安装方法并点击 **下一步**。对于本配方，我们将选择 **本地安装媒体** 选项：![如何操作...](img/image_12_001.jpg)

    新虚拟机向导收集创建新机器所需的详细信息

1.  选择媒体源。如果媒体是 CD 或 DVD，选择 **使用 CDROM 或 DVD** 选项。如果媒体是 ISO 文件，选择 **使用 ISO 镜像** 选项并指定镜像文件的路径。然后，点击 **下一步**：![如何操作...](img/image_12_002.jpg)

    新机器将使用 ISO 文件作为其安装媒体

1.  设置你想要分配给虚拟机的 RAM 大小和 CPU 数量，然后点击 **下一步**：![如何操作...](img/image_12_003.jpg)

    1 GB 的 RAM 和 1 个 CPU 分配给虚拟机

1.  指定分配给机器的存储容量，然后点击**下一步**:![如何操作...](img/image_12_004.jpg)

    该机器设置了 8 GB 的存储空间

1.  提供一个名称以标识虚拟机，然后点击**完成**:![如何操作...](img/image_12_005.jpg)

    向导已准备好创建虚拟机并启动安装介质

1.  虚拟机将自动启动并从指定的安装介质启动。现在，您可以像在物理系统上一样在机器上继续安装您的客户操作系统:![如何操作...](img/image_12_006.jpg)

    可以像在物理系统上安装操作系统一样，在虚拟机上安装操作系统

## 它是如何工作的...

必要的软件通过安装三个软件包组来完成；`虚拟化平台`组安装基础虚拟化库，`虚拟化客户端`包安装用于创建和管理虚拟机的客户端程序，`虚拟化工具`包安装用于维护机器的工具：

```
yum groupinstall "Virtualization Platform" 
"Virtualization Client"  "Virtualization Tools"

```

安装软件后，我们使用虚拟机管理器创建了一台机器。虚拟机定义了一个虚拟系统，指定了可供客户使用的资源以及客户如何访问这些资源。在 GNOME 桌面环境下，管理器可以从**系统工具**类别的**应用程序**菜单启动。在 KDE 中，可以通过 Kickoff 应用程序启动器在**应用程序**|**系统工具**下找到。管理器也可以通过命令行使用`virt-manager`启动：

```
virt-manager

```

### 注意

也可以通过命令行创建新的虚拟机，使用`virt-install`并指定资源分配作为参数。这对于希望通过脚本化方式快速创建新客户机的用户特别有用。

管理器的新虚拟机使得创建新的虚拟机定义变得容易，它会提示我们输入必要的资源分配。例如，我们需要提供内存大小、CPU 数量以及可供客户使用的存储空间。提供完这些值后，管理器会创建并启动虚拟机，从指定的安装介质启动并安装客户操作系统。从那时起，安装操作系统就像在物理系统上安装一样。

要启动虚拟机，从可用的虚拟机列表中选择目标虚拟机，使其高亮显示，然后点击管理工具栏上的播放箭头图标。或者，右键点击列表条目，并从上下文菜单中选择**运行**。这样虚拟机将启动，状态变为*运行中*。完成后，可以通过点击工具栏上的电源图标或在上下文菜单中选择**关机**选项来关闭虚拟机。虚拟机状态将变为*已关闭*。要在虚拟机运行时与其交互，可以双击条目，或者高亮条目后点击工具栏上的打开图标。

### 注意

如果客户机的显示内容过大，无法完全显示，窗口的侧边和底部会出现滚动条。调整显示以适应窗口可以改善体验。要调整显示设置，可以在**查看**菜单中选择**显示**。

## 另见

请参考以下资源获取更多关于虚拟机操作的信息：

+   `virt-install` 手册页面 (`man 1 virt-install`)

+   KVM 官方网站 ([`www.linux-kvm.org/page/Main_Page`](http://www.linux-kvm.org/page/Main_Page))

+   RHEL 7 虚拟化部署与管理指南 ([`access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Virtualization_Deployment_and_Administration_Guide/`](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Virtualization_Deployment_and_Administration_Guide/))

+   KVM 最佳实践 ([`www.ibm.com/support/knowledgecenter/linuxonibm/liaat/liaatbpkickoff.htm`](http://www.ibm.com/support/knowledgecenter/linuxonibm/liaat/liaatbpkickoff.htm))

# 克隆虚拟机

由于虚拟机本质上只是数据文件，因此可以轻松复制和共享。这很有用，因为你可以根据自己的需求设置一个金牌服务器，然后制作多个副本用于不同的目的。然而，使用`cp`命令并不是实现克隆的正确方式。本教程将展示如何通过克隆这一过程来正确地复制一台虚拟机。

## 准备工作

本教程要求先按照前一个教程中的说明设置好虚拟机。虽然克隆过程本身不需要管理员权限，但根据文件存放位置，访问虚拟机文件时可能需要管理员权限。默认情况下，文件存放在`/var/lib/libvirt/images`，这需要管理员访问权限。

## 如何操作...

按照以下步骤克隆虚拟机：

1.  请确保你想要克隆的虚拟机没有在运行。

1.  在虚拟机管理器中，右键点击列表中的目标虚拟机，并从上下文菜单中选择**克隆**。这将打开**克隆虚拟机**对话框：![如何操作...](img/image_12_007.jpg)

    **克隆虚拟机**对话框使克隆虚拟机镜像变得容易

1.  为新镜像指定一个唯一名称，然后点击**克隆**按钮。这将创建一个虚拟机及其选定存储的独立副本。

## 它是如何工作的...

本教程使用虚拟机管理器创建了一个机器副本，称为克隆。机器应以这种方式克隆，而不是仅仅复制底层文件，因为克隆过程还会更新应在机器之间唯一的各种标识符，例如网络接口的 MAC 地址。

### 注意

`virt-clone`命令可用于通过命令行克隆客户机。有关更多信息，请通过`man 1 virt-clone`查看该程序的手册页。

如果您希望在启动克隆机器之前更新其各个方面，可以使用诸如`virt-sysprep`和`virt-configure`之类的工具。这些程序将在一个 chroot 环境中挂载机器的磁盘镜像，执行所请求的修改，然后卸载镜像。`virt-sysprep`通过`libguestfs-tools-c`安装：

```
 yum install libguestfs-tools-c 

```

要查看`virt-sysprep`可以执行的可用维护操作列表，可以使用`--list-operations`命令启动程序。每个选项将显示并附带简短的描述，说明它的功能。要执行某项操作，请使用`--operation`参数，后跟一个或多个操作标签，标签之间用逗号分隔。例如，以下命令会清除系统中所有账户的 bash 历史记录，并删除可能位于`/tmp`中的任何文件。`-a`参数提供机器磁盘镜像的路径：

```
virt-sysprep -a /var/lib/virt/images/Ubuntu-clone.qcow2 
--operations bash-history,tmp-files

```

根据原始机器镜像的用途，您可能还会发现以下清理操作有用：

+   `ca-certificates`：此命令删除任何 CA 证书

+   `logfiles`：此命令用于删除日志文件

+   `ssh-hostkeys`：此命令用于删除 SSH 主机密钥

+   `ssh-userdir`：此命令用于删除用户的`.ssh`目录

+   `user-account`：此命令删除所有用户账户，除 root 外

`virt-sysprep`和`virt-customize`的功能有所重叠；然而，`virt-customize`执行更多的一般性定制操作，而`virt-sysprep`则侧重于清理镜像。`virt-customize`可以执行诸如移动和设置系统主机名、重置密码、安装和卸载包等操作。

要重置系统的主机名，请使用`--hostname`参数并提供所需的名称：

```
virt-customize -a /var/lib/virt/images/Ubuntu-clone.qcow2 
--hostname ubuntu2

```

`--install`和`--uninstall`参数用于添加或删除包，并指定一个或多个包名称，包名称之间用逗号分隔：

```
virt-customize -a /var/lib/virt/images/Ubuntu-clone.qcow2 
--install build-essential

```

以下是一些您可能会在`virt-customize`中找到有用的参数：

+   `--chmod`：此命令用于更改文件权限

+   `--copy`：此命令用于创建文件或目录的副本

+   `--delete`：此命令用于删除文件或目录

+   `--mkdir`：此命令用于创建一个新的目录

+   `--move`：此命令将文件或目录移动到新位置

+   `--password`：此命令用于更新用户的密码

+   `--run-command`：此命令在镜像上运行命令

## 另请参见

请参考以下资源，获取有关克隆和定制虚拟机的更多信息：

+   `virt-clone`手册页（`man 1 virt-clone`）

+   `virt-configure`手册页（`man 1 virt-configure`）

+   `virt-sysprep`手册页（`man 1 virt-sysprep`）

+   如何克隆 KVM 虚拟机并重置虚拟机（[`www.unixarena.com/2015/12/how-to-clone-a-kvm-virtual-machines-and-reset-the-vm.html`](http://www.unixarena.com/2015/12/how-to-clone-a-kvm-virtual-machines-and-reset-the-vm.html)）

# 向虚拟机添加存储

即使你不是数据囤积者，可能也会遇到需要为来宾系统添加额外存储的情况。别担心！这很简单！本配方教你如何添加和修改附加到机器上的虚拟硬件。

## 准备工作

该配方需要按照前面的配方描述设置虚拟机。

## 如何操作...

按照以下步骤为虚拟机添加存储：

1.  确保你想要修改的虚拟机没有在运行。

1.  通过双击可用机器列表中的目标条目来打开虚拟机。

1.  可以点击菜单栏中的灯泡图标或从**视图**中选择**详情**来显示虚拟机的硬件详细信息：![如何操作...](img/image_12_008.jpg)

    机器的虚拟硬件会显示出来，资源可以添加、修改或移除。

1.  点击窗口左下角的**添加硬件**按钮，打开**添加新虚拟硬件**窗口。

1.  从可用资源列表中选择**存储**。指定要分配给新磁盘的存储空间，并点击**完成**：![如何操作...](img/image_12_009.jpg)

    向机器添加了一个虚拟的 8GB 存储驱动器

1.  通过点击菜单栏中的计算机图标或从**视图**中选择**控制台**来离开硬件视图。

## 如何运作...

本配方展示了如何配置与机器相关的虚拟硬件定义。为了增加给来宾操作系统可用的存储空间，我们导航到该视图并添加了一个新的虚拟硬盘。可以通过界面创建存储设备，如配方所示，或者选择一个现有的硬盘文件并将其附加到系统中。

### 注意

如果你要创建一个新磁盘，你需要对存储进行分区、格式化和挂载，这样它才能使用。你可能会发现第五章，*管理文件系统和存储*对你有所帮助。

其他硬件也可以通过硬件视图进行管理。最显著的是，你可以添加和配置新的网络接口，并分配额外的 RAM 和 CPU 资源。增加 RAM/CPU 的目的是在系统上运行资源密集型进程——最好先分配少量资源，然后在需要时增加资源。

另一项有用的配置是更改显示服务器。默认情况下，显示配置为使用 SPICE 协议，这是一种比 VNC 更强大的协议。SPICE 服务器内置于虚拟化平台中，因此即使客户机仅运行控制台显示，你也可以使用 SPICE 客户端连接虚拟机并访问其显示（参阅[`www.spice-space.org/`](https://www.spice-space.org/)以获取 SPICE 客户端）。如果你想改用 VNC 连接，请在硬件列表中选择**显示 SPICE**项，并将其**类型**设置为**VNC 服务器**。将**地址**值更改为**所有接口**，以接受来自本地主机以外的连接，指定连接密码，然后点击**应用**按钮。

硬件列表中的显示标签将更改为**显示 VNC**：

![它是如何工作的...](img/image_12_010.jpg)

用户可以使用 SPICE 或 VNC 客户端连接到虚拟系统的显示

## 另请参见

请参考以下资源，了解更多关于虚拟硬件操作的信息：

+   RHEL 7 虚拟化部署与管理指南：存储池   ([`access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Virtualization_Deployment_and_Administration_Guide/chap-Storage_pools.html`](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Virtualization_Deployment_and_Administration_Guide/chap-Storage_pools.html))

+   存储管理 ([`libvirt.org/storage.html`](http://libvirt.org/storage.html))

# 将 USB 外设连接到客户系统

本配方教你如何将连接到主机系统的 USB 设备共享给虚拟机。这意味着你可以从客户操作系统中使用 USB 打印机、网络摄像头和存储设备。

## 准备工作

本配方要求按照之前的配方设置虚拟机。

## 如何操作...

按照以下步骤将 USB 外设连接到客户系统：

1.  确保你想要修改的虚拟机未运行。

1.  将 USB 设备连接到物理系统。

1.  通过双击可用机器列表中的目标条目打开虚拟机。

1.  通过点击菜单栏中的灯泡图标或从**视图**中选择**详细信息**来显示虚拟机的硬件详细信息。

1.  点击**添加硬件**按钮以打开**添加新虚拟硬件**窗口。

1.  从资源列表中选择**USB HOST 设备**。

1.  选择所需的 USB 设备，然后点击**完成**按钮：![如何操作...](img/image_12_011.jpg)

    附加到主机系统的 USB 设备可以分配给虚拟机

1.  通过点击菜单栏中的计算机图标或从**视图**中选择**控制台**来离开硬件视图。

1.  启动虚拟机并验证 USB 设备是否可用。

## 它是如何工作的...

连接到主机系统的 USB 设备可以通过硬件详情分配给虚拟机。我们选择了**USB 主机设备**类别，显示了所有当前已注册在主机上的设备，我们可以从中进行选择。使用 USB 设备时有几个需要注意的事项。首先，只支持 USB 1.1 协议。对于大多数外设（如网络摄像头、打印机和 USB 麦克风），传输速度并不是问题。然而，如果你打算连接一个 USB 存储设备并传输大量数据，则可能会成为一个问题。其次，设备必须在启动虚拟机之前插入并由主机可访问。这是因为运行在主机上的虚拟化平台负责提供对虚拟机的访问。

### 注意

本教程展示了如何将连接到主机系统的 USB 设备分配给虚拟机。如果你是通过 SPICE 客户端远程访问虚拟机，你可以将 USB 设备插入本地计算机并通过 USB 重定向将其重定向到远程虚拟机。更多信息请参见 RHEL 7 虚拟化部署与管理指南。

## 另见

参阅以下资源以获取有关共享 USB 设备的更多信息：

+   RHEL 7 虚拟化部署与管理指南：USB 设备 ([`access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Virtualization_Deployment_and_Administration_Guide/sect-Guest_virtual_machine_device_configuration-USB_devices.html`](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Virtualization_Deployment_and_Administration_Guide/sect-Guest_virtual_machine_device_configuration-USB_devices.html))

+   使用 libvirt 和 KVM 进行 USB 直通 ([`david.wragg.org/blog/2009/03/usb-pass-through-with-libvirt-and-kvm.html`](https://david.wragg.org/blog/2009/03/usb-pass-through-with-libvirt-and-kvm.html))

# 配置虚拟机的网络接口

本教程教你如何配置虚拟网络接口的行为。通过更改接口的行为，你可以为虚拟机提供直接访问或过滤访问网络的方式，甚至可以设置一个仅对主机系统和其他虚拟机可见的本地网络。

## 准备工作

本教程需要一个具有有效网络连接的 CentOS 系统，还需要按照前面教程的描述设置好虚拟机。

## 如何操作...

按照以下步骤配置虚拟机的网络接口：

1.  确保你要修改的虚拟机未在运行。

1.  双击列表中所需的虚拟机条目以打开虚拟机。

1.  通过点击菜单栏中的灯泡图标或从**视图**中选择**详细信息**来查看虚拟机的硬件详情。

1.  指定所需的**网络源**（NAT 或主机设备）。

1.  如果选择主机设备，请指定所需的模式（桥接、VEPA、私有或直通）：![如何操作...](img/image_12_012.jpg)

    虚拟网络接口可以配置为以不同方式处理来宾的流量。

1.  点击**应用**按钮保存配置。

1.  通过点击菜单栏中的计算机图标或从**视图**中选择**控制台**，离开硬件视图。

1.  启动虚拟机，并根据需要进行来宾网络配置。

## 它是如何工作的...

管理来宾的网络连接性是指定虚拟机网络适配器行为的问题。为了正确操作，我们首先需要了解可用选项的行为。

第一个选项是**网络地址转换**（**NAT**），这是新虚拟机的默认设置。虚拟化平台为来宾提供一个虚拟网络接口，并处理其所有流量。平台通过主机的物理接口转发流量，表现得像是来宾与主机之间的路由器。

第二个选项是将虚拟接口直接绑定到主机的物理接口。共有四种共享模式，具体如下：

+   **桥接**：虚拟化平台连接来宾和主机的接口，使来宾可以直接访问互联网。来宾需要获取自己的 IP 地址，并可以完全访问网络。

+   **VEPA**：此选项适用于支持 VEPA 的网络设备（必须满足特殊硬件要求）。

+   **私有**：平台创建一个私有网络，路由数据包使得同一主机上的虚拟机能够相互通信以及与外部网络通信，但来自网络的连接无法到达虚拟机。

+   **直通**：主机的接口直接共享（必须满足额外的技术要求）。

由于主题的技术性，文档和术语相当专业。而且，许多非网络专家的人在决定正确的配置时常常遇到困难。根据我的经验，非网络专业人员使用虚拟化的两种常见场景是：本地虚拟化用于提供备用环境，以及虚拟化用于提供多个服务器系统。如果你将虚拟机作为典型的桌面系统使用，用户需要通过互联网访问电子邮件和浏览网页，则使用 NAT 网络，并配置来宾使用 DHCP。如果你将这些机器用作服务器，则在桥接模式下共享主机适配器，并为来宾配置静态 IP 地址。

## 另请参见

请参考以下资源，获取更多关于配置虚拟网络接口的信息：

+   libvirt 虚拟化 API: 网络配置 ([`wiki.libvirt.org/page/Networking`](http://wiki.libvirt.org/page/Networking))

+   RHEL 7 虚拟化部署与管理指南：网络配置 ([`access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Virtualization_Deployment_and_Administration_Guide/chap-Network_configuration.html`](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Virtualization_Deployment_and_Administration_Guide/chap-Network_configuration.html))
