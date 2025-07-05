# 第四章. 文件共享

本章内容包括：

+   挂载 USB 硬盘（`pmount`）

+   访问其他计算机的文件（`smbclient`）

+   从其他计算机共享文件夹（`mount.cifs`）

+   在启动时自动挂载 USB 硬盘（`/etc/fstab`）

+   在启动时自动挂载共享文件夹

+   创建文件服务器（Samba）

+   分享附加的 USB 硬盘（Samba）

# 简介

本章中的树莓派配方用于与同一局域网中的其他计算机共享文件。

本章从挂载连接到树莓派的 USB 硬盘的配方开始。接下来的配方展示了如何使用 SMB（CIFS）协议与其他计算机交换文件。然后，有两个配方展示了如何在启动时自动挂载磁盘。本章以几个配方结束，介绍如何将树莓派设置为文件服务器。

完成本章的配方后，你将能够自动挂载本地 USB 硬盘，并使用 SMB（CIFS）协议在树莓派和其他计算机之间交换文件。

# 挂载 USB 硬盘（pmount）

本配方安装并应用`pmount`命令，该命令以与桌面文件管理器相同的方式挂载直接连接到树莓派的 USB 硬盘。

如果通过 USB 端口将磁盘连接到树莓派，使用树莓派的**图形用户界面**（**GUI**）时，桌面文件管理器会请求用户许可来挂载磁盘。然而，当远程使用树莓派（或树莓派以文本模式启动时），没有图形界面来询问是否挂载文件。

本配方不依赖树莓派的 GUI 或桌面文件管理器来挂载磁盘。相反，使用命令行工具`pmount`以与 GUI 相同的方式挂载 USB 驱动器——在 `/media` 目录中。

完成本配方后，你将能够使用大型 USB 存储设备和小型 USB 闪存驱动器与其他计算机交换文件，而无需依赖树莓派的图形用户界面或桌面文件管理器。

## 准备工作

这里是本配方使用的配料：

+   树莓派的初始设置或基本网络设置，已开启电源。你还需要以 `pi` 用户登录（有关如何启动和登录的信息，请参见第一章，*安装与设置*，以及第二章，*管理*中的远程登录配方）。

+   一个带电的 USB 集线器（推荐）。

+   至少需要一个 USB 硬盘驱动器。

本配方不需要桌面 GUI，可以在基于文本的控制台或 LXTerminal 中运行。

如果树莓派的安全外壳服务器正在运行，则可以通过安全外壳客户端远程完成此配方（参见第二章中的*远程访问（SSH）*配方，*管理*）。

本配方中的示例使用了两个 USB 驱动器：一个 32 GB 的闪存驱动器和一个 500 GB 的磁盘。

### 注意

小心！树莓派有电力限制。

树莓派没有足够的内部电力来可靠地驱动直接连接到树莓派 USB 端口的大型 USB 磁盘。

为确保树莓派持续良好运作，任何需要电力的设备（包括 USB 磁盘）都应该通过带电源的 USB 集线器间接连接到树莓派。

带电源的 USB 集线器提供足够的电力来运行多个大型 USB 驱动器，而不需要额外从树莓派获取电力。

如果你看到图形用户界面右上角有一个大而五颜六色的方块闪烁，那是一个警告，表示你的树莓派电力不足。

## 怎么做...

挂载 USB 磁盘的步骤如下：

1.  直接或远程登录到树莓派。

1.  使用`apt-get`来安装`pmount`命令。

    ```
    pi@raspberrypi ~ $ sudo apt-get install pmount
    [sudo] password for pi: 
    ```

1.  `apt-get install`命令下载并安装`pmount`。

    ```
    Reading package lists... Done
    Building dependency tree       
    Reading state information... Done
    Suggested packages:
      cryptsetup
    The following NEW packages will be installed:
      pmount
    0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded.
    Need to get 98.0 kB of archives.
    After this operation, 727 kB of additional disk space will be used.
    Get:1 http://mirrordirector.raspbian.org/raspbian/ wheezy/main pmount armhf 0.9.23-2 [98.0 kB]
    Fetched 98.0 kB in 1s (93.3 kB/s)
    Selecting previously unselected package pmount.
    (Reading database ... 89035 files and directories currently installed.)
    Unpacking pmount (from .../pmount_0.9.23-2_armhf.deb) ...
    Processing triggers for man-db ...
    Setting up pmount (0.9.23-2) ...

    pi@raspberrypi ~ $ 
    ```

1.  将一个或多个 USB 磁盘连接到树莓派。示例中包括两个磁盘：一个 32 GB 的闪存驱动器和一个 500 GB 的磁盘。

1.  使用`fdisk –l`命令列出当前连接到树莓派的磁盘。

    ```
    pi@raspberrypi ~ $ sudo apt-get install pmount
    ```

1.  `fdisk`命令显示有关连接到树莓派的磁盘的信息。

    ```
    Disk /dev/mmcblk0: 31.9 GB, 31914983424 bytes
    4 heads, 16 sectors/track, 973968 cylinders, total 62333952 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x0009bf4f

            Device Boot      Start         End      Blocks   Id  System
    /dev/mmcblk0p1            8192      122879       57344    c  W95 FAT32 (LBA)
    /dev/mmcblk0p2          122880    62333951    31105536   83  Linux

    Disk /dev/sda: 128 MB, 128974848 bytes
    2 heads, 63 sectors/track, 1999 cylinders, total 251904 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x002d7aa3

       Device Boot      Start         End      Blocks   Id  System
    /dev/sda1   *          32      251903      125936    6  FAT16

    Disk /dev/sdb: 500.1 GB, 500107862016 bytes
    255 heads, 63 sectors/track, 60801 cylinders, total 976773168 sectors
    Units = sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x002317d5

       Device Boot      Start         End      Blocks   Id  System
    /dev/sdb1            2048   976773167   488385560    7  HPFS/NTFS/exFAT
    ```

1.  上述命令输出显示了三个磁盘：

    +   `/dev/mmcblk0`：31.9 GB 的 SD 卡（启动磁盘）有两个分区：`/dev/mmcblk0p1`和`/dev/mmcblk0p2`

    +   `/dev/sda`：128 MB 的闪存驱动器有一个分区

    +   `/dev/sdb`：500.1 GB 的磁盘也有一个分区

1.  使用新安装的`pmount`命令来挂载闪存驱动器的主分区(`/dev/sda1`)和 USB 磁盘的主分区(`/dev/sdb1`)。无需使用`sudo`命令。

    ```
    pi@raspberrypi ~ $ ls -l /media
    total 0

    pi@raspberrypi ~ $ pmount /dev/sda1
    pi@raspberrypi ~ $ pmount /dev/sdb1

    pi@raspberrypi ~ $ ls -l /media
    total 20
    drwx------ 4 pi pi 16384 Dec 31  1969 sda1
    drwxr-xr-x 6 pi pi  4096 May 30 12:23 sdb1

    pi@raspberrypi ~ $
    ```

1.  上述命令输出显示了在使用`pmount`命令挂载两个磁盘之前和之后`/media`目录的内容。执行`pmount`命令后，每个磁盘都出现在`/media`目录中。

1.  `df –h`命令也可以用来显示当前挂载的磁盘及其挂载点。在以下命令输出中，两个磁盘`/dev/sda1`和`/dev/sdb1`分别挂载在`/media/sda1`和`/media/sdb1`：

    ```
    pi@raspberrypi ~ $ df –h
    Filesystem      Size  Used Avail Use% Mounted on
    rootfs           30G  4.2G   24G  15% /
    /dev/root        30G  4.2G   24G  15% /
    devtmpfs        484M     0  484M   0% /dev
    tmpfs            98M  252K   98M   1% /run
    tmpfs           5.0M     0  5.0M   0% /run/lock
    tmpfs           195M     0  195M   0% /run/shm
    /dev/mmcblk0p1   56M   19M   37M  34% /boot
    /dev/sdb1       459G  172G  264G  40% /media/sdb1
    /dev/sda1       123M   71M   53M  58% /media/sda1

    pi@raspberrypi ~ $
    ```

## 工作原理...

登录到树莓派后，安装`pmount`命令，连接两个磁盘驱动器，使用命令`fdisk –l`显示连接到树莓派的磁盘的分区表。在示例中，显示了三个磁盘：

+   `/dev/mmcblk0`是启动磁盘

+   `/dev/sda`是 128 MB 的闪存驱动器

+   `/dev/sdb`是 500 GB 磁盘

闪存驱动器有一个分区(`/dev/sda1`)，磁盘也有一个分区(`/dev/sdb1`)。文件存储在磁盘分区中。

`pmount`命令用于挂载磁盘分区。磁盘分区挂载在`/media`目录下。使用`ls –l`命令在挂载前后显示磁盘分区挂载的位置。

`/media`目录中的挂载点被赋予了与已挂载磁盘分区相对应的名称。闪存驱动器的磁盘分区`/dev/sda1`被挂载在`/media/sda1`，而 500 GB 磁盘的磁盘分区`/dev/sdb1`则被挂载在`/media/sdb1`。

最后，使用`df -h`命令展示了另一种查看磁盘分区已挂载位置的方法。`df`命令还显示了每个已挂载磁盘上可用的空闲磁盘空间。`-h`选项告诉`df`以字节为单位显示每个磁盘的大小，而不是以磁盘块为单位。500 GB 磁盘已使用 40%，还有 264 GB 可用。闪存驱动器已使用 58%，只剩下 53 MB 可用。

## 还有更多…

让我们更多地了解一下磁盘。

### 设备文件

当每个磁盘或磁盘分区被挂载时，会为其创建一个设备文件并存储在`/dev`目录下。这些设备文件的名称会随着磁盘的挂载顺序依次分配。

USB 驱动器的分配名称都以`sd`开头。第一个 USB 磁盘驱动器被分配了设备文件`/dev/sda`，第二个则是`/dev/sdb`，依此类推。

SD 卡驱动器的分配名称以`mmcblk`开头，并为每个唯一的设备添加一个数字，从`0`开始。树莓派中的唯一 SD 卡驱动器（启动磁盘）的设备文件位于`/dev/mmcblk0`。

对于在磁盘分区表中发现的每个磁盘分区，也会创建一个设备文件。每个磁盘分区的设备文件与磁盘的设备文件名称相同，并附加一个数字，表示其在分区表中的位置。

在这个例子中，32 GB SD 卡被分配了设备文件`/dev/mmcblk0`。它有两个分区：`/dev/mmcblk0p1`和`/dev/mmcblk0p2`。如果 SD 卡还有另一个分区，那么第三个分区的设备文件将是`/dev/mmcblk0p3`。

当磁盘具有分区表时，每个分区需要单独挂载。使用`pmount`命令时，应使用分区的设备文件，而不是磁盘的设备文件。在示例中，500 GB 磁盘有一个分区，可以使用命令`pmount /dev/sdb1`来挂载它。

当磁盘没有分区表时，使用`pmount`命令时应使用分配给磁盘驱动器的设备文件。

### 挂载点

当磁盘分区被挂载时，默认情况下会在`/media`目录下为每个分区创建一个挂载点。默认挂载点的名称与用于挂载磁盘分区的设备文件名称相同。

128 MB 闪存驱动器的唯一分区被分配了设备文件`/dev/sda1`，因此默认的挂载点是`/media/sda1`。500 GB 磁盘也只有一个分区`/dev/sdb1`，被挂载在`/media/sdb1`。

若要在`/media`目录下创建一个与默认名称不同的挂载点，您需要将所需的挂载点作为第二个参数传递给`pmount`命令，如以下示例所示：

```
pi@raspberrypi ~ $ pmount /dev/sdb1 mydisk

pi@raspberrypi ~ $ ls -l /media
total 20
drwx------ 4 pi pi 16384 Dec 31  1969 sda1
drwxr-xr-x 6 pi pi  4096 May 30 12:23 mydisk

pi@raspberrypi ~ $
```

### 卸载磁盘

使用 `pumount` 命令卸载磁盘（命令名中没有 `n`）。磁盘卸载后，它们的挂载点会从 `/media` 目录中删除。

```
pi@raspberrypi ~ $ ls -l /media
total 20
drwx------ 4 pi pi 16384 Dec 31  1969 sda1
drwxr-xr-x 6 pi pi  4096 May 30 12:23 mydisk

pi@raspberrypi ~ $ pumount /dev/sda1

pi@raspberrypi ~ $ ls -l /media
total 4
drwxr-xr-x 6 pi pi 4096 May 30 12:23 mydisk

pi@raspberrypi ~ $ pumount /media/mydisk

pi@raspberrypi ~ $ ls -l /media
total 0

pi@raspberrypi ~ $
```

如前面的示例所示，`pumount` 命令接受磁盘设备（`/dev/sda1`）或挂载点（`/media/mydisk`）作为参数。

### plugdev 组

使用 `pmount` 命令不需要超级用户权限，但仍然需要一定的权限。执行 `pmount` 命令的权限仅限于 `plugdev` 系统组的成员。不在 `plugdev` 组中的用户无法执行 `pmount` 命令。

```
Here is an example of how to add a user to the plugdev group:
pi@raspberrypi ~ $ groups golden
golden : golden sudo

pi@raspberrypi ~ $ adduser golden plugdev

pi@raspberrypi ~ $ groups golden
golden : golden sudo plugdev

pi@raspberrypi ~ $ 
```

### 注意

在 Raspbian 上，默认用户 `pi` 已经是 `plugdev` 组的成员。

### 其他挂载命令

系统命令 `mount` 需要超级用户权限。在启动过程中，`mount` 命令会自动挂载像树莓派启动所用的 SD 卡这样的磁盘。下一个食谱将展示如何配置树莓派，使其在启动时自动挂载附加的驱动器。

包含在树莓派桌面中的文件管理器 PCManFM 也会在 `/media` 目录中挂载磁盘。然而，PCManFM 文件管理器使用磁盘的标签作为挂载点的名称，而不是磁盘的设备名称。

### 磁盘性能

Class 10 SD 卡的传输速率为每秒 10 MB，而外部硬盘可以以每秒 300 MB 的速度向计算机传输数据。硬盘的速度几乎比 SD 卡快了 30 倍！

尽管 SD 卡的容量越来越大，但它们并不是高性能的最佳选择。即便是超高速 SD 卡，30 MB/sec 的速度也比硬盘慢 10 倍。本书中的大多数食谱如果依赖的数据存储在外部硬盘上，会表现得更好。

### 树莓派的电力是有限的

没有独立电源的外部磁盘和闪存驱动器通过树莓派的 USB 连接来获取电力。没有自供电的设备所需的电力由树莓派提供。树莓派的电力是有限的；如果设备没有独立电源，它无法可靠地支持较大的 USB 设备。

外部 USB 设备所需的电力可能会轻易超过树莓派提供的电力。如果从树莓派提取的电力过多，其他 USB 设备，包括网络接口（内部使用 USB 总线）可能无法正常工作，甚至可能完全失效。

为了获得最佳的性能和可靠性，建议将所有 USB 设备（包括外部硬盘驱动器和闪存驱动器）通过一个有电源的 USB 集线器*间接*连接到树莓派，而不是直接连接到树莓派。

一个带电源的 USB 集线器是所有连接到它的 USB 设备的电源来源。只有数据会传输到树莓派，而带电源的 USB 集线器不会从树莓派获取电力。相反，它保护树莓派免受其他设备的电力消耗影响。通过带电源的 USB 集线器*间接*连接 USB 设备，可以确保树莓派以最佳性能和可靠性运行。

## 另见

+   **硬盘驱动器** ([`en.wikipedia.org/wiki/Hard_disk_drive`](http://en.wikipedia.org/wiki/Hard_disk_drive)): 这篇维基百科文章提供了关于磁盘驱动器的更多信息。

+   **USB 闪存驱动器** ([`en.wikipedia.org/wiki/USB_flash_drive`](http://en.wikipedia.org/wiki/USB_flash_drive)): 这篇维基百科文章提供了关于 USB 闪存驱动器的详细信息。

+   **SD 卡** ([`en.wikipedia.org/wiki/Secure_Digital`](http://en.wikipedia.org/wiki/Secure_Digital)): 这篇维基百科文章介绍了**安全数字**（**SD**）卡格式。

+   **mount (Unix)** ([`en.wikipedia.org/wiki/Mount_(Unix)`](http://en.wikipedia.org/wiki/Mount_(Unix))): 这篇维基百科文章介绍了`mount`命令。

+   **df – 报告文件系统磁盘空间使用情况** ([`manpages.debian.net/cgi-bin/man.cgi?query=df`](http://manpages.debian.net/cgi-bin/man.cgi?query=df)): Debian 的`df`手册页提供了关于该命令及其选项的更多信息。

+   **fdisk – Linux 分区表操作工具** ([`manpages.debian.net/cgi-bin/man.cgi?query=fdisk`](http://manpages.debian.net/cgi-bin/man.cgi?query=fdisk)): Debian 的`fdisk`手册页提供了关于该命令的更多信息。

+   **pmount – 以普通用户身份挂载任意热插拔设备** ([`manpages.debian.net/cgi-bin/man.cgi?query=pmount`](http://manpages.debian.net/cgi-bin/man.cgi?query=pmount)): Debian 的`pmount`手册页提供了关于该命令的更多信息。

+   **pumount – 卸载任意热插拔设备** ([`manpages.debian.net/cgi-bin/man.cgi?query=pumount`](http://manpages.debian.net/cgi-bin/man.cgi?query=pumount)): Debian 的`pumount`手册页提供了关于该命令的更多信息。

+   **udev – Linux 动态设备管理** ([`manpages.debian.net/cgi-bin/man.cgi?query=udev`](http://manpages.debian.net/cgi-bin/man.cgi?query=udev)): Debian 的`udev`手册页更详细地解释了设备如何挂载。

# 访问另一台计算机的文件（smbclient）

本教程的目标是通过`smbclient`命令在树莓派和其他计算机之间传输文件。

本教程展示了如何列出可用的网络共享文件夹、复制单个文件，并使用`smbclient`命令将整个文件夹的文件从共享文件夹的计算机复制到树莓派。

完成本教程后，你将能够列出本地网络上可用的文件共享，并使用 SMB（CIFS）协议直接在另一台计算机和树莓派之间传输文件。

## 准备中...

以下是本教程的材料：

+   树莓派已经开机并完成初始设置或基础网络设置。你也已经以用户`pi`登录（请参阅第一章中的配方，*安装与设置*，了解如何启动和登录，以及第二章中的配方，*管理*，了解如何远程登录）。

+   一个网络连接。

+   一台与树莓派连接到相同网络的客户端 PC。

本配方不需要桌面图形界面，可以在基于文本的控制台或 LXTerminal 中运行。

如果树莓派的安全外壳服务器正在运行，则可以使用安全外壳客户端远程完成此配方（请参阅第二章中的配方*远程访问（SSH）*，*管理*）。

本配方中的示例将树莓派连接到本地网络计算机 golden-macbook 和家庭文件服务器`terragolden`。使用 SMB（CIFS）协议的其他计算机配置将类似。

## 如何操作……

访问另一台计算机文件的步骤如下：

1.  直接或通过远程登录到树莓派。

1.  使用命令`apt-get install`下载并安装`smbclient`软件包。

    ```
    pi@raspberrypi ~ $ sudo apt-get install smbclient
    ```

1.  使用命令`smbclient`列出家庭文件服务器`terragolden`上的服务（`-L`）。无需密码（`-N`）即可进行游客访问。

    ```
    pi@raspberrypi ~ $ smbclient -N -L terragolden
    ```

1.  显示来自家庭文件服务器`terragolden`的 SMB（CIFS）服务列表。

    ```
    Anonymous login successful
    Domain=[GOLDEN] OS=[Unix] Server=[Samba 3.2.5]

        Sharename       Type      Comment
        ---------       ----      -------
        IPC$            IPC       IPC Service (Golden Family Media Drive)
        backups         Disk      
        livemusic       Disk      
        photos          Disk      
        public          Disk      

    Anonymous login successful
    Domain=[GOLDEN] OS=[Unix] Server=[Samba 3.2.5]

        Server               Comment
        ---------            -------
        TERRAGOLDEN          Golden Family Media Drive

        Workgroup            Master
        ---------            -------
        GOLDEN               

    pi@raspberrypi ~ $ 
    ```

1.  使用命令`smbclient`列出家庭计算机 golden-macbook 上的服务（`-L`）。访问需要有效的用户名（`-U`）和密码。

    ```
    pi@raspberrypi ~ $ smbclient –U golden -L golden-macbook –-signing=off

    Enter golden's password: 
    ```

1.  显示来自 golden-macbook 的 SMB（CIFS）服务列表。

    ```
    Domain=[GOLDEN-MACBOOK] OS=[Darwin] Server=[@(#)PROGRAM:smbd  PROJECT:smbx-276.92.2]

        Sharename       Type      Comment
        ---------       ----      -------
        IPC$            IPC       
        Macintosh HD    Disk      
        xfer            Disk      
        golden          Disk      

    pi@raspberrypi ~ $ 
    ```

1.  现在，使用`mkdir`命令在树莓派上创建一个新目录（`xfer`），使用`cd`命令切换到新目录，然后使用`ls`命令显示新创建的目录（`~/xfer`）中没有文件。

    ```
    pi@raspberrypi ~ $ mkdir xfer

    pi@raspberrypi ~ $ cd xfer

    pi@raspberrypi ~/xfer $ ls -l
    total 0

    pi@raspberrypi ~/xfer $ 
    ```

1.  使用`smbclient`命令与家庭文件服务器`terragolden`进行文件共享`livemusic`的对话。

    ```
    pi@raspberrypi ~/xfer $ smbclient -N //teragolden/livemusic
    smb: \> 
    ```

1.  显示命令提示符`smb: \>`，表示`smbclient`命令已准备好与`terragolden`进行对话。

1.  使用`ls`命令列出`terragolden`上`livemusic`共享的文件和文件夹。

    ```
    smb: \> ls
      .                                   D        0  Fri Oct 28 15:45:43 2011
      ..                                  D        0  Tue Oct  9 22:03:55 2012
      catalog.txt                         A     5119  Mon Jul 24 18:04:30 2017
      dso040424                           D        0  Wed Mar  9 19:46:22 2011
      dtb2005-03-04                       D        0  Wed Mar  9 19:43:35 2011
      franti2010-06-11.flac16             D        0  Fri Mar 25 08:43:13 2011
      garaj2005-04-10_16bit_64kb_mp3      D        0  Wed Mar  9 19:50:28 2011
      JackJohnson2010-07-17               D        0  Mon Mar 14 14:04:25 2011
      JasonMraz-FarmAid                   D        0  Thu Apr 28 16:03:48 2011
      JasonMrazGrandRex_vbr_mp3           D        0  Thu Apr 28 16:04:59 2011
      jgb2008-07-19_64kb_mp3              D        0  Wed Mar  9 19:41:06 2011
      jj2010-12-13.aud.flac16             D        0  Wed Mar  2 11:46:06 2011
      MF2010-10-06_Nak30016BitFlac        D        0  Fri Mar 25 10:31:05 2011
      mFranti2006-11-13                   D        0  Wed Mar  9 19:55:00 2011
      mfs2005-01-15                       D        0  Wed Mar  9 19:48:50 2011
      particle2010-04-15                  D        0  Fri Mar 25 09:13:14 2011
      particle2010-10-30                  D        0  Fri Mar 25 08:45:00 2011
      skb2005-10-01matrix_64kb_mp3        D        0  Mon Mar 14 14:12:01 2011
      Soulive2011-03-08                   D        0  Fri Mar 25 08:33:21 2011
      Soulive2011-03-10                   D        0  Fri Mar 25 08:34:11 2011
      tenaciousd2005-09-22.flac16_64kb    D        0  Wed Mar  9 20:32:06 2011
      um2007-04-21                        D        0  Wed Mar  9 19:47:59 2011
      ymsb2001-02-02                      D        0  Wed Mar  9 09:23:13 2011
      ymsb2007-06-22                      D        0  Wed Mar  9 19:50:18 2011
      .DS_Store                          AH     6148  Fri Oct 28 15:45:43 2011
      ._.DS_Store                        AH     4096  Fri Oct 28 15:45:43 2011

            59356 blocks of size 16777216\. 4490 blocks available

    smb: \>
    ```

1.  使用`get`命令将`catalog.txt`文件从`terragolden`下载到树莓派。

    ```
    smb: \> get catalog.txt
    getting file \catalog.txt of size 5119 as catalog.txt (208.3 KiloBytes/sec) (average 208.3 KiloBytes/sec)

    smb: \> 
    ```

1.  现在，使用`mget`命令从`terragolden`下载整个目录`particle2010-04-15`到树莓派。`mget`命令首先要求设置`tarmode`、`recurse`和`prompt`选项。

    ```
    smb: \> tarmode
    tarmode is now full, system, hidden, noreset, verbose
    smb: \> recurse
    smb: \> prompt
    smb: \> mget particle2010-04-15
    getting file \particle2010-04-15\particle2010-04-15.ffp of size 594 as particle2010-04-15.ffp (11.8 KiloBytes/sec) (average 11.8 KiloBytes/sec)
    getting file \particle2010-04-15\particle2010-04-15t01.flac of size 70373980 as particle2010-04-15t01.flac (8086.2 KiloBytes/sec) (average 8039.9 KiloBytes/sec)
    getting file \particle2010-04-15\particle2010-04-15t02.flac of size 58391723 as particle2010-04-15t02.flac (8070.1 KiloBytes/sec) (average 8053.6 KiloBytes/sec)
    getting file \particle2010-04-15\particle2010-04-15t03.flac of size 100964559 as particle2010-04-15t03.flac (4672.9 KiloBytes/sec) (average 6110.7 KiloBytes/sec)
    getting file \particle2010-04-15\particle2010-04-15t04.flac of size 82562641 as particle2010-04-15t04.flac (8648.2 KiloBytes/sec) (average 6624.5 KiloBytes/sec)
    getting file \particle2010-04-15\particle2010-04-15t05.flac of size 115649970 as particle2010-04-15t05.flac (5859.4 KiloBytes/sec) (average 6398.7 KiloBytes/sec)
    getting file \particle2010-04-15\particle2010-04-15t06.flac of size 57681372 as particle2010-04-15t06.flac (4870.3 KiloBytes/sec) (average 6168.8 KiloBytes/sec)
    getting file \particle2010-04-15\particle2010-04-15t07.flac of size 46807245 as particle2010-04-15t07.flac (8770.2 KiloBytes/sec) (average 6333.9 KiloBytes/sec)
    getting file \particle2010-04-15\text.txt of size 491 as text.txt (10.0 KiloBytes/sec) (average 6346.5 KiloBytes/sec)

    smb: \> 
    ```

1.  使用`quit`命令结束与家庭文件服务器`terragolden`的 SMB（CIFS）对话，并返回到树莓派命令提示符。

    ```
    smb: \> quit

    pi@raspberrypi ~/xfer $ 
    ```

1.  使用`ls –l`命令显示树莓派当前目录的内容。

    ```
    pi@raspberrypi ~/xfer $ ls –l

    total 20
    -rw------- 1 pi pi 13564 Jul 19 13:52 catalog.txt
    drwxr-xr-x 2 pi pi  4096 Jul 19 13:45 particle2010-04-15

    pi@raspberrypi ~/xfer $
    ```

## 它是如何工作的……

登录树莓派后，使用`apt-get install`命令安装软件分发包`smbclient`。有关`apt-get`命令如何工作的更多信息，请参见第三章，*维护*。

### 注意

你的软件分发包可能已经安装了`smbclient`。

`smbclient`软件分发包包含命令行应用程序`smbclient`。该应用程序用于通过 SMB 协议与其他计算机进行通信。

在软件分发包安装完成后，使用`smbclient`命令列出名为`terragolden`的主文件服务器上的可用服务。`–N`选项告诉命令不需要远程计算机的密码，而`–L`选项则告诉命令仅列出可用的共享和打印机。

为了对比，`smbclient`命令还用于列出另一台连接到本地网络的计算机（golden-macbook）上的可用服务。这次，访问该计算机需要提供用户名（`-U golden`）和密码（没有`–N`选项），并且`smbclient`命令还需要使用`--signing=off`选项，以关闭该计算机不支持的安全功能。

在使用`smbclient`命令与`terragolden`开始会话之前，首先使用`mkdir`命令创建一个传输目录（`xfer`）以接收传输的文件，并通过`ls –l`命令显示其（空的）内容。`–l`选项告诉`ls`命令使用长格式列出文件，其中包含文件总数（`0`）。

命令`smbclient –N //terragolden/livemusic`用于启动树莓派与远程计算机`terragolden`之间的会话，使用的是名为`livemusic`的文件共享。命令提示符会变为`smb: \>`，表示会话已开始。

`smbclient`的`ls`命令用于列出主文件服务器上的文件。也可以使用`dir`命令，返回的结果与`ls`命令相同。

命令`get catalog.txt`用于将文件`catalog.txt`从文件服务器传输到树莓派。`smbclient`命令报告该文件的大小为 5119 字节，并以每秒 208.3 KB 的速率完成传输。

在`catalog.txt`文件传输完成后，设置三个选项以准备进行多文件传输。`tarmode`选项被重置为默认设置，确保所有文件都能传输。`recurse`选项被切换，以便即将执行的`mget`命令也能复制子目录和文件。而`prompt`选项被切换，以便即将执行的`mget`命令在传输每个文件之前不会提示进行验证。

设置好传输选项后，使用`mget`命令将`particle2010-04-15`目录中的文件传输到树莓派。每当传输一个文件时，会显示该文件的大小及其传输速率。

所有文件传输完成后，`quit` 命令用于告诉 `smbclient` 应用程序与远程计算机 `terragolden` 的会话已经结束。

命令提示符返回到 `pi@raspberrypi ~ $`，显示会话已完成。

## 还有更多内容...

`smbclient` 应用程序拥有一套丰富的命令，用于通过 SMB 协议与远程计算机进行通信。

### help

可以使用 `help` 命令来列出从 `smb: \>` 提示符处可用的命令。

```
smb: \> help
?              allinfo        altname        archive        blocksize      
cancel         case_sensitive cd             chmod          chown          
close          del            dir            du             echo           
exit           get            getfacl        geteas         hardlink       
help           history        iosize         lcd            link           
lock           lowercase      ls             l              mask           
md             mget           mkdir          more           mput           
newer          open           posix          posix_encrypt  posix_open     
posix_mkdir    posix_rmdir    posix_unlink   print          prompt         
put            pwd            q              queue          quit           
readlink       rd             recurse        reget          rename         
reput          rm             rmdir          showacls       setea          
setmode        stat           symlink        tar            tarmode        
translate      unlock         volume         vuid           wdel           
logon          listconnect    showconnect    ..             !              

smb: \> help help
HELP help:
    [command] give help on a command

smb: \> 
```

每个 `smbclient` 应用程序的命令都有自己的帮助页面，可以使用 `help` 命令显示。键入 `help` 后跟命令名称，以显示关于该命令的更多信息。

### 更改远程目录

`smbclient` 中的 `cd` 命令可以用来像在 Raspberry Pi 上使用 `cd` 更改目录一样，在远程计算机上更改目录。注意，当前的远程目录路径会显示在 `smbclient` 命令提示符中。

```
smb: \> ls golden/stuff/astrology
  .                                   D        0  Wed Jun 29 22:20:25 2011
  ..                                  D        0  Wed Oct 26 13:49:49 2011
  astrolog data                      DR        0  Mon Dec 12 07:13:40 2005
  ephall-astrolog                    DR        0  Mon Jan 10 08:01:24 2005
  ephcom-1.0                         DR        0  Thu Apr  7 00:39:03 2005
  experiments                        DR        0  Mon Jan 10 21:45:51 2005
  jpl                                DR        0  Fri Mar  4 03:48:43 2005
  SwissEphemeris                     DR        0  Thu Apr  7 06:03:14 2005
  zodiac                             DR        0  Thu Apr  7 00:34:28 2005

        59356 blocks of size 16777216\. 4490 blocks available

smb: \ > cd golden/stuff/astrology/zodiac

smb: \golden\stuff\astrology\zodiac\> ls
  .                                  DR        0  Thu Apr  7 00:34:28 2005
  ..                                  D        0  Wed Jun 29 22:20:25 2011
  .classpath                        AHR      236  Thu Apr  7 00:33:05 2005
  .project                          AHR      382  Thu Apr  7 00:33:05 2005
  classes                            DR        0  Wed Apr 20 10:14:10 2005
  src                                DR        0  Thu Apr  7 00:34:28 2005

        59356 blocks of size 16777216\. 4490 blocks available

smb: \golden\stuff\astrology\zodiac\> 
```

### 获取单个文件

如果已经知道远程计算机中文件的位置，可以通过一个 `smbclient` 命令直接获取该文件。

```
pi@raspberrypi ~/xfer $ smbclient –N //terragolden –c 'get /livemusic/Seeed/03-seed-aufstehen.mp3'

getting file \livemusic\Seeed\03-seed-aufstehen.mp3 of size 650317 as \livemusic\Seeed\03-seed-aufstehen.mp3 (10414.8 KiloBytes/sec) (average 10414.8 KiloBytes/sec)

pi@raspberrypi ~/xfer $
```

### / 与 \

注意，SMB (CIFS) 协议使用 `\` 作为路径分隔符，而不是 `/`。

路径分隔符是 Windows 操作系统与基于 Linux 的操作系统（如 Raspberry Pi 使用的 Raspbian Linux）之间的主要区别之一。

在 `smbclient` 应用程序中，两个字符都可以使用，只要一致使用。不过，使用 `/` 更加方便，因为它是 Raspberry Pi 上的路径分隔符。

## 另请参见

+   **cd – 更改工作目录** ([`manpages.debian.net/cgi-bin/man.cgi?query=cd`](http://manpages.debian.net/cgi-bin/man.cgi?query=cd)): Debian 的 `cd` 手册页面提供了更多关于该命令及其选项的信息。

+   **smbclient – 用于访问服务器上的 SMB/CIFS 资源的客户端** ([`manpages.debian.net/cgi-bin/man.cgi?query=smbclient`](http://manpages.debian.net/cgi-bin/man.cgi?query=smbclient)): Debian 的 `smbclient` 手册页面提供了更多关于该命令的信息。

# 从其他计算机共享文件夹（mount.cifs）

这个食谱展示了如何挂载从另一台计算机共享的文件夹。

最近的 Linux 内核，包括 Raspberry Pi 上使用的 Raspbian Linux 内核，内置支持使用 SMB (CIFS) 协议挂载共享文件夹。这是 Windows 计算机和家庭文件共享设备常用的文件共享协议。在 OS X 上共享文件时也使用该协议。

这是在 Raspberry Pi 上通过命令行挂载共享文件夹的最简单方法。

完成这个食谱后，你将能够使用 SMB (CIFS) 协议与同一网络上的其他计算机共享文件。

### 注意

SMB 和 CIFS 是同一种文件共享协议的同义词。

## 准备工作

这是这个食谱所需的配料：

+   对于已开机的树莓派，需进行初步设置或基本网络设置。你还需要以`pi`用户身份登录（请参阅第一章中的配方，*安装与设置*，了解如何启动和登录，以及第二章中的配方，*管理*，了解如何远程登录）。

+   一个带电源的 USB 集线器（推荐）。

+   至少需要一个 USB 磁盘驱动器。

此配方不需要桌面 GUI，可以从基于文本的控制台或 LXTerminal 中运行。

如果树莓派的安全外壳（SSH）服务器正在运行，可以通过安全外壳客户端远程完成此操作（请参阅第二章中的*远程访问（SSH）*配方，*管理*）。

本配方中的示例将树莓派连接到名为`terragolden`的家庭文件服务器，该服务器共享一个名为`livemusic`的文件夹，并配置为访客访问。

## 如何操作...

从其他计算机共享文件夹的步骤如下：

1.  登录到树莓派，直接登录或远程登录均可。

1.  使用`mkdir`命令为共享文件夹创建一个挂载点：

    ```
    pi@raspberrypi ~ $ sudo mkdir /media/livemusic
    ```

1.  使用`mount`命令和`guest`及`uid=pi`选项（`-o`）将远程文件共享`//terragolden/livemusic`挂载到本地目录`/media/livemusic`。

    ```
    pi@raspberrypi ~ $ sudo mount -o guest,uid=pi //terragolden/livemusic /media/livemusic 
    ```

1.  使用`ls –l`命令列出`/media/livemusic`文件夹中的文件。

    ```
    pi@raspberrypi ~ $ ls -l /media/livemusic
    total 0
    drwx------ 2 pi nogroup 0 Mar  9  2011 dso040424
    drwx------ 2 pi nogroup 0 Mar  9  2011 dtb2005-03-04
    drwxrwxrwx 2 pi nogroup 0 Mar 25  2011 franti2010-06-11.flac16
    drwx------ 2 pi nogroup 0 Mar  9  2011 garaj2005-04-10_16bit_64kb_mp3
    drwx------ 2 pi nogroup 0 Mar 14  2011 JackJohnson2010-07-17
    drwx------ 2 pi nogroup 0 Apr 28  2011 JasonMraz-FarmAid
    drwx------ 2 pi nogroup 0 Apr 28  2011 JasonMrazGrandRex_vbr_mp3
    drwx------ 2 pi nogroup 0 Mar  9  2011 JGB2007-06-15
    drwx------ 2 pi nogroup 0 Mar  9  2011 jgb2008-07-19_64kb_mp3
    drwx------ 2 pi nogroup 0 Mar  2  2011 jj2010-12-13.aud.flac16
    drwxrwxrwx 2 pi nogroup 0 Mar 25  2011 MF2010-10-06_Nak30016BitFlac
    drwx------ 2 pi nogroup 0 Mar  9  2011 mFranti2006-11-13
    drwx------ 2 pi nogroup 0 Mar  9  2011 mfs2005-01-15
    drwxrwxrwx 2 pi nogroup 0 Mar 25  2011 particle2010-04-15
    drwxrwxrwx 2 pi nogroup 0 Mar 25  2011 particle2010-10-30
    drwx------ 2 pi nogroup 0 Mar 14  2011 skb2005-10-01matrix_64kb_mp3
    drwx------ 2 pi nogroup 0 Mar 25  2011 Soulive2011-03-08
    drwx------ 2 pi nogroup 0 Mar 25  2011 Soulive2011-03-10
    drwx------ 2 pi nogroup 0 Mar  9  2011 tenaciousd2005-09-22.flac16_64kb
    drwx------ 2 pi nogroup 0 Mar  9  2011 um2007-04-21
    drwx------ 5 pi nogroup 0 Mar  9  2011 ymsb2001-02-02
    drwx------ 2 pi nogroup 0 Mar  9  2011 ymsb2007-06-22

    pi@raspberrypi ~ $ 
    ```

## 它是如何工作的...

在登录到树莓派后，使用`mkdir`命令在`/media`目录中为共享文件夹创建一个挂载点。挂载点是一个空目录（`/media/livemusic`），作为文件系统中的占位符。

然后，从家庭文件服务器`terragolden`共享的文件`//terragolden/livemusic`被挂载到新创建的挂载点（`/media/livemusic`）。`mount`命令的选项（`–o guest,uid=pi`）告诉命令以无需用户名或密码（`guest`）的方式访问共享，并且挂载的文件将属于用户`pi`（`uid=pi`）。

最后，使用`ls -l`命令列出挂载点（`/media/livemusic`）中的文件，以验证它们是否与家庭文件服务器上的文件完全相同。

## 还有更多...

让我们看看与共享文件夹相关的其他方面。

### 受保护的共享需要用户名和密码

如果文件共享受密码保护，则`mount`命令中的选项需要扩展，添加`username=USER`和`password=PASS`选项。`USER`和`PASS`应分别替换为有权访问该共享的用户的用户名和密码。

```
pi@raspberrypi ~/xfer $ sudo mount -o uid=pi,user=golden //golden-macbook/xfer /media/xfer
Password:

pi@raspberrypi /media/xfer $ ls -l /media/golden-macbook/
total 8200
drwxr-xr-x 2 pi root       0 Jul 17 16:58 Applications
drwxr-xr-x 2 pi root       0 Jul  9 09:33 bin
drwxr-xr-x 2 pi root       0 Aug 24  2013 cores
drwxr-xr-x 2 pi root       0 Jul 14 18:06 dev
-rwxr-xr-x 1 pi root       0 Mar 11 09:23 etc
-rwxr-xr-x 1 pi root       0 Mar 11 09:26 HGN.flag
drwxr-xr-x 2 pi root       0 Jul 14 18:06 home
drwxr-xr-x 2 pi root       0 Mar 27 10:35 Library
-rwxr-xr-x 1 pi root 8394688 Mar 18 16:20 mach_kernel
drwxr-xr-x 2 pi root       0 Jul 14 18:06 net
drwxr-xr-x 2 pi root       0 Aug 24  2013 Network
drwxr-xr-x 2 pi root       0 Mar 18 08:53 opt
drwxr-xr-x 2 pi root       0 Mar 11 09:35 private
drwxr-xr-x 2 pi root       0 Jul  9 09:33 sbin
drwxr-xr-x 2 pi root       0 Mar 11 09:30 System
-rwxr-xr-x 1 pi root       0 Mar 11 09:23 tmp
drwxr-xr-x 2 pi root       0 Mar 27 14:48 Users
drwxr-xr-x 2 pi root       0 Mar 11 09:23 usr
-rwxr-xr-x 1 pi root       0 Mar 11 09:23 var
drwxr-xr-x 2 pi root       0 Jul 19 11:18 Volumes

pi@raspberrypi /media/xfer $ 
```

### 卸载磁盘

使用`umount`命令卸载磁盘（命令名称中没有`n`）。

```
pi@raspberrypi /media/xfer $ sudo umount /media/golden-macbook
pi@raspberrypi /media/xfer $ ls -l /media/golden-macbook
total 0

pi@raspberrypi /media/xfer $ 
```

在磁盘卸载后，挂载点会保留在`/media`目录下；然而，它们会再次变为空目录。`umount`命令需要特权（使用`sudo`）。

## 另见

+   **文件系统** ([`en.wikipedia.org/wiki/Filesystem`](http://en.wikipedia.org/wiki/Filesystem))：维基百科关于文件系统的文章更详细地解释了它们的区别。

+   **服务器消息块** ([`en.wikipedia.org/wiki/Server_Message_Block`](http://en.wikipedia.org/wiki/Server_Message_Block))：维基百科关于**服务器消息块**（**SMB**）协议的文章讨论了该协议的使用及其历史。

+   **统一命名约定** ([`en.wikipedia.org/wiki/Uniform_Naming_Convention`](http://en.wikipedia.org/wiki/Uniform_Naming_Convention))：维基百科关于**统一命名约定**（**UNC**）协议的文章展示了它如何支持 SMB（CIFS）协议。

+   **mount – 挂载文件系统** ([`manpages.debian.net/cgi-bin/man.cgi?query=mount`](http://manpages.debian.net/cgi-bin/man.cgi?query=mount))：Debian 的`mount`手册页面解释了该命令及其选项。

+   **mount.cifs – 使用 CIFS 文件系统挂载** ([`manpages.debian.net/cgi-bin/man.cgi?query=mount.cifs`](http://manpages.debian.net/cgi-bin/man.cgi?query=mount.cifs))：Debian 的`mount.cifs`手册页面更详细地解释了该命令。

+   **umount – 卸载文件系统** ([`manpages.debian.net/cgi-bin/man.cgi?query=umount`](http://manpages.debian.net/cgi-bin/man.cgi?query=umount))：Debian 的`umount`手册页面展示了该命令及其选项。

# 在启动时自动挂载 USB 磁盘（/etc/fstab）

本食谱展示了如何配置树莓派，以便它在启动过程中自动挂载附加的 USB 磁盘驱动器。

本食谱的目标是在启动时使用与`pmount`命令相同的配置挂载示例磁盘。

本食谱的前几个步骤使用`pmount`和`mount`命令来发现两个示例磁盘驱动器的正确文件系统表配置参数。发现正确的配置参数后，文件系统表配置文件(`/etc/fstab`)将被更新，以便在树莓派启动时挂载这些磁盘。

本食谱不提供每个配置参数的详细含义，而是重用了`pmount`命令中使用的相同配置。有关配置参数的更多细节可以在`mount`命令的手册页中找到。

完成本食谱后，树莓派将在启动时使用与`pmount`工具一致的配置挂载 USB 磁盘驱动器。

## 准备工作

以下是本食谱的配料：

+   树莓派的初步设置或基本网络设置已经完成，并且已登录为用户`pi`（有关如何启动和登录的信息，请参见第一章，《安装与设置》；有关如何远程登录的信息，请参见第二章，《管理》）。

+   一个已供电的 USB 集线器（推荐）。

+   至少需要一个 USB 磁盘驱动器。

本配方不需要桌面 GUI，可以在基于文本的控制台或在 LXTerminal 中运行。

如果 Raspberry Pi 的安全外壳服务器正在运行，则可以使用安全外壳客户端远程完成此配方（参见第二章，*管理*中的配方*远程访问（SSH）*）。

本配方中的示例使用了两块 USB 驱动器：一块 128 MB 的闪存驱动器和一块 500 GB 的磁盘。

`pmount`命令应该已经安装（参见配方*挂载 USB 磁盘*）。

### 注意

小心！一个损坏的`/etc/fstab`可能会导致系统无法启动。在重启之前，请确保已经测试过配置！

## 如何操作...

自动挂载 USB 磁盘的步骤如下：

1.  直接或远程登录到 Raspberry Pi。

1.  使用`pmount`命令将 128 MB 闪存驱动器的主分区`/dev/sda1`挂载到`/media`目录中的挂载点`thumbdrive`。

    ```
    pi@raspberrypi /media/xfer $ pmount /dev/sda1 thumbdrive
    ```

1.  使用`pmount`命令将 500 GB 磁盘的主分区`/dev/sdb1`挂载到`/media`目录中的挂载点`bigdisk`。

    ```
    pi@raspberrypi /media/xfer $ pmount /dev/sdb1 bigdisk
    ```

1.  使用`ls -l`命令显示磁盘是否已挂载。

    ```
    pi@raspberrypi ~ $ ls -l /media
    total 20
    drwxr-xr-x 6 pi pi  4096 May 30 12:23 bigdisk
    drwx------ 4 pi pi 16384 Dec 31  1969 thumbdrive

    pi@raspberrypi ~ $ 
    ```

1.  使用`mount`命令显示挂载这两个磁盘的配置。

    ```
    pi@raspberrypi ~ $ mount
    ```

1.  `mount`命令显示文件系统表，包括每个磁盘（或磁盘分区）所需的挂载参数。

    ```
    /dev/root on / type ext4 (rw,noatime,data=ordered)
    devtmpfs on /dev type devtmpfs (rw,relatime,size=494800k,nr_inodes=123700,mode=755)
    tmpfs on /run type tmpfs (rw,nosuid,noexec,relatime,size=99820k,mode=755)
    tmpfs on /run/lock type tmpfs (rw,nosuid,nodev,noexec,relatime,size=5120k)
    proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
    sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime)
    tmpfs on /run/shm type tmpfs (rw,nosuid,nodev,noexec,relatime,size=199620k)
    devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000)
    /dev/mmcblk0p1 on /boot type vfat (rw,relatime,fmask=0022,dmask=0022,codepage=437,iocharset=ascii,shortname=mixed,errors=remount-ro)
    /dev/sda1 on /media/thumbdrive type vfat (rw,nosuid,nodev,noexec,relatime,uid=1001,gid=1004,fmask=0177,dmask=0077,codepage=437,iocharset=iso8859-1,shortname=mixed,quiet,utf8,errors=remount-ro)
    /dev/sdb1 on /media/bigdisk type ext4 (rw,nosuid,nodev,noexec,relatime,errors=remount-ro,data=ordered)

    pi@raspberrypi ~ $ 
    ```

1.  在显示磁盘配置参数后，使用`pumount`命令卸载这两个磁盘。

    ```
    pi@raspberrypi ~ $ pumount /media/thumbdrive
    pi@raspberrypi ~ $ pumount /media/bigdisk

    pi@raspberrypi ~ $ ls -l /media
    total 0
    ```

1.  使用`cp`命令复制当前的文件系统配置。只有特权用户才能在`/etc`目录中创建（或复制到）文件（因此请使用`sudo`）。

    ```
    pi@raspberrypi ~ $ sudo cp /etc/fstab /etc/fstab.orig
    ```

1.  使用`vi`命令编辑文件系统配置表`fstab`。

    ```
    pi@raspberrypi ~ $ sudo vi /etc/fstab
    ```

1.  vi 编辑器显示`/etc/fstab`配置文件的内容。有关如何使用编辑器的说明，请参见 vi 手册页面。在第二章，*管理*中查阅“*阅读内置文档*”的配方。

    ```
    proc            /proc           proc    defaults          0       0
    /dev/mmcblk0p1  /boot           vfat    defaults          0       2
    /dev/mmcblk0p2  /               ext4    defaults,noatime  0       1
    # a swapfile is not a swap partition, so no using swapon|off from here on, use  dphys-swapfile swap[on|off]  for that
    ~                                                                               
    ~                                                                               
    ~                                                                               
    "/etc/fstab" 4 lines, 322 characters
    ```

1.  将每个磁盘的配置参数添加到配置文件的底部。每个磁盘的参数可以从`mount`命令的输出中复制。

    ### 注意

    每个磁盘配置应为一行，字段之间用空格或制表符分隔。示例不适合页面宽度。

    ```
    proc            /proc           proc    defaults          0       0
    /dev/mmcblk0p1  /boot           vfat    defaults          0       2
    /dev/mmcblk0p2  /               ext4    defaults,noatime  0       1
    # a swapfile is not a swap partition, so no using swapon|off from here on, use  dphys-swapfile swap[on|off]  for that
    /dev/sda1       /media/thumbdrive  vfat  rw,nosuid,nodev,noexec,relatime,uid=1001,gid=1004,fmask=0177,dmask=0077,codepage=437,iocharset=iso8859-1,shortname=mixed,quiet,utf8,errors=remount-ro 0 2
    /dev/sdb1       /media/bigdisk  ext4    rw,nosuid,nodev,noexec,relatime,errors=remount-ro,data=ordered 0 2
    ~                                                                               
    ~
    "/etc/fstab" 6 lines, 605 characters
    ```

1.  输入每个磁盘的配置参数后，保存文件并退出编辑器。

    ```
    pi@raspberrypi ~ $ 
    ```

1.  在磁盘可以挂载之前，需要先为其创建挂载点。使用`mkdir`命令创建挂载点。

    ```
    pi@raspberrypi ~ $ sudo mkdir /etc/thumbdrive
    pi@raspberrypi ~ $ sudo mkdir /etc/bigdisk

    pi@raspberrypi ~ $ ls –l /media
    total 8
    drwxr-xr-x 2 root root 4096 Jul 19 19:14 bigdisk
    drwxr-xr-x 2 root root 4096 Jul 19 19:15 thumbdrive

    pi@raspberrypi ~ $ ls -l /media/thumbrioe/
    total 0

    pi@raspberrypi ~ $ ls -l /media/bigdisk/
    total 0
    ```

1.  使用`mount -a`命令挂载所有已配置的磁盘。

    ```
    pi@raspberrypi ~ $ sudo mount -a

    pi@raspberrypi ~ $ ls -l /media
    total 20
    drwxr-xr-x 6 pi staff   4096 May 30 12:23 bigdisk
    drwx------ 4 pi pi     16384 Dec 31  1969 thumbdrive

    pi@raspberrypi ~ $ ls -l /media/bigdisk/
    total 24
    drwxrwsr-x 11 pi pi  4096 Apr 26  2012 archive
    drwxrws--- 20 pi pi  4096 Jan 23  2010 Live
    drwx------  2 pi pi 16384 Oct  4  2010 lost+found

    pi@raspberrypi ~ $ ls -l /media//thumbdrive/
    total 71278
    -rw------- 1 pi pi 72988392 Mar  7 00:34 docker.tgz

    pi@raspberrypi ~ $ 
    ```

1.  使用`reboot`命令重启系统，然后重新登录，使用`ls -l`命令显示两个挂载点`/media/thumdrive`和`/media/bigdisk`的内容。

    ```
    pi@raspberrypi ~ $ sudo reboot

    Broadcast message from root@raspberrypi (pts/2) (Sun Jul 19 18:40:55 2015):
    The system is going down for reboot NOW!
    pi@raspberrypi ~ $ Connection to 192.168.2.7 closed by remote host.
    Connection to 192.168.2.7 closed.

    golden-macbook:~ golden$ ssh pi@raspberrypi.local
    Linux raspberrypi 3.18.11-v7+ #781 SMP PREEMPT Tue Apr 21 18:07:59 BST 2015 armv7l

    The programs included with the Debian GNU/Linux system are free software;
    the exact distribution terms for each program are described in the
    individual files in /usr/share/doc/*/copyright.

    Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
    permitted by applicable law.
    Last login: Sun Jul 19 18:44:50 2015 from 192.168.2.1

    pi@raspberrypi ~ $ ls -l /media
    total 20
    drwxr-xr-x 6 pi staff   4096 May 30 12:23 bigdisk
    drwx------ 4 pi pi     16384 Dec 31  1969 thumbdrive

    pi@raspberrypi ~ $ ls -l /media/bigdisk
    total 24
    drwxrwsr-x 11 pi staff  4096 Apr 26  2012 archive
    drwxrws--- 20 pi staff  4096 Jan 23  2010 Live
    drwx------  2 pi staff 16384 Oct  4  2010 lost+found

    pi@raspberrypi ~ $ ls -l /media/thumbdrive
    total 71278
    -rw------- 1 pi pi 72988392 Mar  7 00:34 docker.tgz

    pi@raspberrypi ~ $ 
    ```

1.  现在磁盘在启动时会自动挂载！

## 它是如何工作的...

登录树莓派后，使用个人挂载命令`pmount`挂载两个示例磁盘（有关`pmount`命令的更多信息，请参见食谱*挂载 USB 磁盘*）。

128 MB 闪存驱动器（`/dev/sda1`）挂载在`/media/thumbdrive`。

500 GB 磁盘（`/dev/sdb1`）挂载在`/media/bigdisk`。

系统的`mount`命令用于显示附加到树莓派的所有磁盘的文件系统表配置参数，包括`bigdisk`和`thumbdrive`。

配置参数显示后，使用`pumount`命令卸载两个 USB 磁盘（`/dev/sda1`和`/dev/sdb1`）。`/media`目录再次变为空。

在对文件系统配置进行任何更改之前，使用`cp`命令创建`/etc/fstab`的备份副本。

然后使用 vi 编辑器将每个磁盘的配置参数添加到文件系统表配置文件（`/etc/fstab`）的底部。

在磁盘可以挂载之前，必须首先在文件系统中创建挂载点。使用`mkdir`命令为每个 USB 磁盘在`/media/thumbdrive`和`/media/bigdisk`下创建挂载点。

一旦挂载点创建完成，可以使用`mount all disks`命令，`mount –a`，来挂载示例磁盘。

在磁盘挂载后，可以使用`ls –l`命令显示磁盘的内容。

一切看起来都很正常，因此使用`reboot`命令重启树莓派。

当树莓派重启完成后，用户可以使用`ssh`命令重新登录到树莓派。

最后，再次使用`ls –l`命令列出两个磁盘挂载点`/media/thumbdrive`和`/media/bigdisk`的内容。

磁盘在启动过程中已成功挂载！

## 还有更多内容…

我们将看看如何从错误中恢复。

### 错误恢复

如果`mount`命令显示错误，请再次编辑`/etc/fstab`文件，检查是否有拼写错误和多余的空格。修正任何错误并重试。

如果仍然存在错误，请使用以下命令将配置文件替换为其原始版本：

```
pi@raspberrypi ~ $ sudo cp –f /etc/fstab.orig /etc/fstab
```

### fstab 文件格式

`fstab`配置文件的每一行都用于配置某个磁盘（或磁盘分区）的挂载位置。每行有六个字段，用空格（或制表符）分隔：

+   第一个字段是设备文件。设备文件会自动分配给每个磁盘，并且可以使用`fdisk –l`命令发现（请参阅之前的食谱）。128 MB 的`thumbdrive`有一个分区，其可挂载的设备文件是`/dev/sda1`。500 GB 磁盘也有一个分区，其可挂载的设备文件是`/dev/sdb1`。

+   第二个字段是挂载点。挂载点可以位于文件系统中的任何位置；然而，当前的惯例是将它们放在`/media`目录下。128 MB 闪存驱动器的挂载点是`/media/thumbdrive`，500 GB 磁盘的挂载点是`/media/bigdisk`。

+   第三个字段是设备的文件系统类型。128 MB 的闪存驱动器使用`vfat`文件系统，而 500 GB 的硬盘使用`ext4`文件系统。

+   第四个字段是一个以逗号分隔的配置参数列表。这些参数应该从`mount`命令输出中的括号列表复制过来。此字段中不应有空格，也不应包含括号。

+   第五个字段是转储标志。通常为零（`0`），表示文件系统不需要转储。两个磁盘在此字段中都是`0`。

+   第六个字段选择磁盘将被挂载的启动阶段。作为根文件系统（`/`）挂载的磁盘在第一阶段挂载；所有其他磁盘应该在第二阶段挂载。示例中的磁盘不会作为根文件系统挂载，因此它们在此字段中都有`2`。

对于 128 MB 的闪存驱动器，配置参数如下：

+   **设备文件**: `/dev/sda1`

+   **挂载点**: `/media/thumbdrive`

+   **文件系统类型**: `vfat`

+   **挂载选项**: `rw,nosuid,nodev,noexec,relatime,uid=1001,gid=1004,fmask=0177,dmask=0077,codepage=437,iocharset=iso8859-1,shortname=mixed,quiet,utf8,errors=remount-ro`

+   **转储选项**: （`0`）

+   **启动阶段**: （`2`）

对于 500 GB 的 USB 硬盘，配置参数如下：

+   **设备文件**: `/dev/sdb1`

+   **挂载点**: `/media/bigdisk`

+   **文件系统类型**: `ext4`

+   **挂载选项**: `rw,nosuid,nodev,noexec,relatime,errors=remount-ro,data=ordered`

+   **转储选项**: （`0`）

+   **启动阶段**: （`2`）

## 另请参阅

+   **fstab – 静态文件系统信息** ([`manpages.debian.net/cgi-bin/man.cgi?query=fstab`](http://manpages.debian.net/cgi-bin/man.cgi?query=fstab)): Debian 手册页面为`fstab`命令提供了关于命令及其选项的详细信息。

# 在启动时自动挂载共享文件夹

本配方的目标是在启动时挂载来自另一台计算机的共享文件夹。

上一个配方展示了如何配置`/etc/fstab`以挂载 USB 磁盘。这个配方展示了如何使用类似的配置，在启动时自动挂载 Windows 共享（或使用 SMB（CIFS）协议共享的任何其他文件集）。

在家里或办公室，常见的本地网络中会有某种形式的**网络附加存储**（**NAS**）作为共享文件夹，通过 SMB（CIFS）协议供网络用户使用。这个配方展示了如何配置树莓派，使其在启动时自动挂载共享文件夹。

完成此配方后，每次树莓派启动时，来自另一台计算机的共享文件夹将自动附加到根文件系统。

## 准备工作

以下是所需的组件：

+   已开机的 Raspberry Pi 进行的初步设置或基本网络设置。你还以 `pi` 用户身份登录（有关如何启动和登录的说明，请参见 第一章，*安装与设置*；有关如何远程登录的说明，请参见 第二章，*管理*）。

+   一个网络连接。

+   一台与 Raspberry Pi 处于同一网络的客户端计算机。

本食谱不需要桌面 GUI，可以从基于文本的控制台或 LXTerminal 中运行。

如果 Raspberry Pi 的 Secure Shell 服务器正在运行，可以使用 Secure Shell 客户端远程完成此食谱（参见 第二章，*管理* 中的食谱 *远程访问（SSH）*）。

本食谱中的示例将 Raspberry Pi 连接到一个名为 `terragolden` 的家庭文件共享设备。这个文件服务器有多个共享文件夹，包括备份。共享文件夹已配置为访客访问，因此不需要用户名或密码。

使用 SMB（CIFS）协议的其他计算机配置将类似。

## 如何操作...

自动挂载共享文件夹的步骤如下：

1.  直接或远程登录到 Raspberry Pi。

1.  使用 `cp` 命令将 `/etc/fstab` 配置文件备份为名为 `/etc/fstab.orig` 的副本。文件只能由具有特权的用户在 `/etc` 目录中创建（或复制到该目录）（因此使用 `sudo`）。

    ```
    pi@raspberrypi ~ $ sudo cp /etc/fstab /etc/fstab.orig
    ```

1.  执行以下命令：

    ```
    nano /etc/fstab
    ```

    编辑文件系统配置表 `fstab`。`fstab` 文件只能由特权用户修改（使用 `sudo`）。

1.  使用 `vi` 命令编辑文件系统配置表 `fstab`。

    ```
    pi@raspberrypi ~ $ sudo vi /etc/fstab
    ```

1.  vi 编辑器显示 `/etc/fstab` 配置文件的内容。使用编辑器的说明可以在 vi 手册中找到。请参阅 第二章，*管理* 中的食谱 *读取内置文档*。

    ```
    proc            /proc           proc    defaults          0       0
    /dev/mmcblk0p1  /boot           vfat    defaults          0       2
    /dev/mmcblk0p2  /               ext4    defaults,noatime  0       1
    # a swapfile is not a swap partition, so no using swapon|off from here on, use  dphys-swapfile swap[on|off]  for that
    //terragolden/backups /media/backups cifs guest,uid=pi 0 2
    ~                                                                                                                                                 
    ~                                                                               
    "/etc/fstab" 4 lines, 322 characters
    ```

1.  对于示例中的家庭文件服务器，配置如下：

    +   网络位置为 `//terragolden/backups`

    +   挂载点为 `/media/backups`

    +   文件系统类型为 `cifs`

    +   挂载选项为 `guest,uid=pi`

    +   dump 选项为 `0`

    +   启动阶段为 `2`

1.  输入文件共享的配置参数后，保存文件并退出编辑器。

    ```
    pi@raspberrypi ~ $ 
    ```

1.  在文件共享被挂载之前，需要创建挂载点。使用 `mkdir` 命令在 `/media/backups` 创建挂载点。

    ```
    pi@raspberrypi ~ $ sudo mkdir /etc/backups

    pi@raspberrypi ~ $ ls –l /media
    total 4
    drwxr-xr-x 2 root root 4096 Jul 19 19:15 backups

    pi@raspberrypi ~ $ ls -l /media/thumbdrive/
    total 0

    pi@raspberrypi ~ $ ls -l /media/bigdisk/
    total 0
    ```

1.  使用 `mount –a` 命令挂载所有已配置的磁盘。

    ```
    pi@raspberrypi ~ $ sudo mount -a

    pi@raspberrypi ~ $ ls -l /media
    total 0
    drwxrwxrwx 14 pi 254 0 Jul 24  2017 backups

    pi@raspberrypi ~ $ ls -l /media/backups
    total 3072
    drwxrwxrwx 44 pi nogroup      0 Oct 28  2011 0 - live music
    drwxr-xr-x  9 pi nogroup      0 Oct 26  2011 1 - family video
    drwxr-xr-x  4 pi nogroup      0 Oct  1  2011 3 - golden studios
    drwxrwxrwx  5 pi nogroup      0 Mar 29  2012 8 - software
    drwxrwxrwx  2 pi nogroup      0 May 19  2011 9 - virtual machines
    drwxrwxrwx  8 pi nogroup      0 May 11  2011 Alex
    drwxrwxrwx 21 pi nogroup      0 Jan 20  2012 Barbara
    drwxrwxrwx  4 pi nogroup      0 Feb 19 19:53 Michael
    drwxrwxrwx  2 pi nogroup      0 May 19  2011 Patrick
    drwxr-xr-x 30 pi nogroup      0 Apr  2  2012 Richard
    drwxr-xr-x  4 pi nogroup      0 Dec 10  2011 xfer

    pi@raspberrypi ~ $ 
    ```

1.  使用 `reboot` 命令重启系统。

    ```
    pi@raspberrypi ~ $ sudo reboot
    Broadcast message from root@raspberrypi (pts/2) (Sun Jul 19 18:40:55 2015):
    The system is going down for reboot NOW!
    ```

1.  重新登录 Raspberry Pi，然后使用 `ls –l` 命令显示文件共享 /media/backups 的内容。

    ```
    pi@raspberrypi ~ $ ls -l /media/backups
    total 3072
    drwxrwxrwx 44 pi nogroup      0 Oct 28  2011 0 - live music
    drwxr-xr-x  9 pi nogroup      0 Oct 26  2011 1 - family video
    drwxr-xr-x  4 pi nogroup      0 Oct  1  2011 3 - golden studios
    drwxrwxrwx  5 pi nogroup      0 Mar 29  2012 8 - software
    drwxrwxrwx  2 pi nogroup      0 May 19  2011 9 - virtual machines
    drwxrwxrwx  8 pi nogroup      0 May 11  2011 Alex
    drwxrwxrwx 21 pi nogroup      0 Jan 20  2012 Barbara
    drwxrwxrwx  4 pi nogroup      0 Feb 19 19:53 Michael
    drwxrwxrwx  2 pi nogroup      0 May 19  2011 Patrick
    drwxr-xr-x 30 pi nogroup      0 Apr  2  2012 Richard
    drwxr-xr-x  4 pi nogroup      0 Dec 10  2011 xfer

    pi@raspberrypi ~ $ 
    ```

1.  远程文件共享现在已在启动时自动挂载！

## 它是如何工作的...

登录 Raspberry Pi 后，对文件系统表配置文件 `/etc/fstab` 进行备份。配置文件被复制到 `/etc/fstab.orig`。

使用 vi 编辑器将共享文件夹的配置参数添加到配置文件的底部。

示例中使用的配置如下：

+   第一个字段是网络位置：`//terragolden/xfer`。第二个字段是挂载点：`/media/backups`。

+   第三个字段是文件系统的类型：`cifs`。这是文件服务器用来共享文件夹的协议。

+   第四个字段是一个以逗号分隔的选项列表：`guest,uid=pi`。其他选项包括`user=`和`password=`，用于指定共享文件夹受保护时的用户名和密码。

+   第五个字段是一个转储标志。它被设置为零（`0`），表示共享文件夹在启动时不需要转储。

+   第六个字段选择了磁盘挂载的启动阶段。作为根文件系统（`/`）挂载的磁盘在第一阶段挂载；所有其他磁盘应在第二阶段（`2`）挂载。

一旦文件系统表配置文件（`/etc/fstab`）被更新，配置会在重启系统前进行测试。使用`mount all disks`命令，即`mount -a`，来挂载示例磁盘。

如果没有错误，系统可以重启。

如果挂载命令显示错误，请再次编辑`/etc/fstab`文件，检查是否有拼写错误或多余的空格。修正错误后重新尝试。

如果仍然存在错误，请使用以下命令将配置文件替换为原始版本：

```
pi@raspberrypi ~ $ sudo cp –f /etc/fstab.orig /etc/fstab
```

## 另请参见

+   **网络附加存储** ([`en.wikipedia.org/wiki/Network_Attached_Storage`](http://en.wikipedia.org/wiki/Network_Attached_Storage))：这是关于**网络附加存储**（**NAS**）的一个有趣的维基百科文章。

# 创建文件服务器（Samba）

本步骤展示了如何将树莓派配置为局域网上的文件服务器。

树莓派，配备附加的文件存储，作为文件服务器运行良好。这样的文件服务器可以用作共享文件和文档的中心位置、存储其他计算机的备份文件，并存储大容量媒体文件，如照片、音乐和视频文件。

本步骤安装并配置了 samba 和 samba-common-bin。Samba 软件包包含一个用于设置*共享驱动器*或*共享文件夹*的 SMB（CIFS）协议服务器，这是现代计算机使用的协议。samba-common-bin 软件包包含一组用于管理共享文件访问的小工具。

本步骤包括为登录用户设置文件共享密码，并为用户主目录中的文件提供读/写权限。然而，它并没有设置新的文件共享，也没有显示如何共享 USB 磁盘。下一个步骤将展示如何操作。

完成此步骤后，局域网上的其他计算机可以与树莓派交换文件。

## 准备工作

以下是此步骤所需的配置信息：

+   为已开机的 Raspberry Pi 完成初始设置或基本的网络设置。你还需要作为用户 `pi` 登录（请参阅 第一章，*安装与设置* 中的配方，了解如何启动并登录，以及 第二章，*管理* 中的配方，了解如何远程登录）。

+   一个网络连接。

+   一台与 Raspberry Pi 连接到同一网络的客户端 PC。

本配方不需要桌面图形界面，可以在基于文本的控制台或 LXTerminal 中运行。

如果 Raspberry Pi 的安全外壳服务器正在运行，则可以使用安全外壳客户端远程完成此配方（请参见 第二章，*管理* 中的 *远程访问（SSH）* 配方）。

## 如何操作...

创建文件服务器的步骤如下：

1.  直接或远程登录到 Raspberry Pi。

1.  使用 `apt-get install` 命令安装软件包 samba 和 samba-common-bin。

    ```
    pi@raspberrypi ~ $ sudo apt-get install –y samba samba-common-bin

    Reading package lists... Done
    Building dependency tree       
    Reading state information... Done
    The following extra packages will be installed:
      tdb-tools
    Suggested packages:
      openbsd-inetd inet-superserver smbldap-tools ldb-tools ctdb
    The following NEW packages will be installed:
      samba samba-common-bin tdb-tools
    0 upgraded, 3 newly installed, 0 to remove and 0 not upgraded.
    Need to get 6,119 kB of archives.
    After this operation, 36.1 MB of additional disk space will be used.
    Get:1 http://mirrordirector.raspbian.org/raspbian/ wheezy/main samba armhf 2:3.6.6-6+deb7u5 [3,356 kB]
    Get:2 http://mirrordirector.raspbian.org/raspbian/ wheezy/main samba-common-bin armhf 2:3.6.6-6+deb7u5 [2,737 kB]
    Get:3 http://mirrordirector.raspbian.org/raspbian/ wheezy/main tdb-tools armhf 1.2.10-2 [25.9 kB]
    Fetched 6,119 kB in 2s (2,236 kB/s)
    Preconfiguring packages ...
    Selecting previously unselected package samba.
    (Reading database ... 89348 files and directories currently installed.)
    Unpacking samba (from .../samba_2%3a3.6.6-6+deb7u5_armhf.deb) ...
    Selecting previously unselected package samba-common-bin.
    Unpacking samba-common-bin (from .../samba-common-bin_2%3a3.6.6-6+deb7u5_armhf.deb) ...
    Selecting previously unselected package tdb-tools.
    Unpacking tdb-tools (from .../tdb-tools_1.2.10-2_armhf.deb) ...
    Processing triggers for man-db ...
    Setting up samba (2:3.6.6-6+deb7u5) ...
    Generating /etc/default/samba...
    Adding group `sambashare' (GID 110) ...
    Done.
    update-alternatives: using /usr/bin/smbstatus.samba3 to provide /usr/bin/smbstatus (smbstatus) in auto mode
    [ ok ] Starting Samba daemons: nmbd smbd.
    Setting up samba-common-bin (2:3.6.6-6+deb7u5) ...
    update-alternatives: using /usr/bin/nmblookup.samba3 to provide /usr/bin/nmblookup (nmblookup) in auto mode
    update-alternatives: using /usr/bin/net.samba3 to provide /usr/bin/net (net) in auto mode
    update-alternatives: using /usr/bin/testparm.samba3 to provide /usr/bin/testparm (testparm) in auto mode
    Setting up tdb-tools (1.2.10-2) ...
    update-alternatives: using /usr/bin/tdbbackup.tdbtools to provide /usr/bin/tdbbackup (tdbbackup) in auto mode

    pi@raspberrypi ~ $ 
    ```

1.  使用 `vi` 命令编辑 Samba 配置文件 (`/etc/samba/smb.conf`)。

    ```
    pi@raspberrypi ~ $ sudo vi /etc/samba/smb.conf
    ```

1.  vi 编辑器显示 `/etc/samba/smb.conf` 文件的内容。使用编辑器的说明可以在 vi 手册页中找到；请参见 第二章，*管理* 中的 *阅读内建文档*。

    ```
    #
    # Sample configuration file for the Samba suite for Debian GNU/Linux.
    #
    #
    # This is the main Samba configuration file. You should read the
    # smb.conf(5) manual page in order to understand the options listed
    # here. Samba has a huge number of configurable options most of which
    # are not shown in this example
    #
    "/etc/samba/smb.conf" 333 lines, 12173 characters
    ```

1.  更改 `= user` 的安全行，取消该行的注释。（移除行首的井号 `#`。）

    ```
    # "security = user" is always a good idea. This will require a Unix account
    # in this server for every user accessing the server. See
    # /usr/share/doc/samba-doc/htmldocs/Samba3-HOWTO/ServerType.html
    # in the samba-doc package for details.
    security = user
    ```

1.  将 `read only = yes` 行更改为 `read only = no`。

    ```
    [homes]
       comment = Home Directories
       browseable = no

    # By default, the home directories are exported read-only. Change the
    # next parameter to 'no' if you want to be able to write to them.
       read only = no
    ```

1.  保存（`:w`）并退出（`:q`）vi 编辑器。

1.  使用 `/etc/init.d/samba` 初始化脚本，告诉 Samba 服务器重新加载其配置文件。

    ```
    pi@raspberrypi ~ $ sudo /etc/init.d/samba reload
    [ ok ] Reloading /etc/samba/smb.conf: smbd only.
    ```

1.  使用 `smbpasswd –a` 命令为已登录的用户 `pi` 创建一个 SMB（CIFS）密码。输入密码（两次）。

    ```
    pi@raspberrypi ~ $ sudo smbpasswd -a pi
    New SMB password:
    Retype new SMB password:
    Added user pi.

    pi@raspberrypi ~ $ 
    ```

1.  现在，Raspberry Pi 作为文件服务器可以访问！

1.  在 Windows 计算机上，使用 **映射网络驱动器** 将 Raspberry Pi 挂载为网络磁盘。![如何操作...](img/B04547_04_01.jpg)

    上面的截图展示了在 Windows 7 上将网络驱动器映射到 Raspberry Pi。

1.  输入 UNC 地址 `\\raspberrypi\pi` 作为网络文件夹。选择一个合适的驱动器字母。以下示例使用 `R:` 驱动器。选择 **使用不同的凭据连接**。点击 **完成**。![如何操作...](img/B04547_04_02.jpg)

    上面的截图完成了将网络驱动器映射到 Raspberry Pi。

1.  使用新配置的 SMB（CIFS）密码登录（见步骤 7）。![如何操作...](img/B04547_04_03.jpg)

    在截图中，显示了一个对话框，提示使用 SMB（CIFS）用户名和密码登录到 Raspberry Pi。

1.  现在，Raspberry Pi 作为 Windows 共享可以访问！此时，仅用户 `pi` 的主目录可以访问。下一步的操作是配置 USB 磁盘作为共享驱动器使用。

## 如何操作...

登录树莓派后，配方通过 `apt-get install` 命令安装两个软件包：samba 和 samba-common-bin。

samba 包包含 **服务器消息块**（**SMB**）协议的实现（也称为 **公共互联网文件系统**，**CIFS**）。用于共享文件和打印机时，Microsoft Windows 计算机使用 SMB（CIFS）协议。

安装完软件包后，Samba 配置文件 `/etc/samba/smb.conf` 会被更新。该文件被更新以启用用户安全（`security = user`）并允许将文件写入用户的主目录（`readonly = no`）。

配置文件更新后，Samba 服务器会使用服务器的初始化文件 `/etc/init.d/samba` 重新加载其配置。

此时，树莓派应该能被使用 SMB 协议的本地网络中的其他计算机看到。然而，授权用户的密码尚未配置。

`smbpasswd` 命令用于将用户 `pi` 添加（`-a`）到授权与树莓派使用 SMB 协议共享文件的用户列表中。文件共享的密码与用于直接或远程登录树莓派的登录密码是分开管理的。

在为用户 `pi` 添加密码后，树莓派应该可以从本地网络中任何配置为 SMB 协议的机器访问。

配方的最后步骤配置了从 Windows 7 PC 使用映射的网络驱动器访问树莓派。文件共享的 UNC 名称 `\\raspberrypi\pi` 也可以直接在 Windows 资源管理器中访问共享。

## 还有更多...

这是一个非常简单的文件共享配置。它为登录到树莓派的用户启用文件共享。然而，它仅允许共享用户主目录中的文件。下一个配方描述了如何添加新的文件共享。

除了 SMB 协议服务器 `smbd`，Samba 软件包还包含一个 NetBIOS 名称服务器 `nmbd`。NetBIOS 名称服务器为使用 SMB 协议的计算机提供命名服务。`nmbd` 服务器将树莓派的配置名称 `raspberrypi` 广播到本地网络中的其他计算机。

除了文件共享，Samba 服务器还可以用作 **主域控制器**（**PDC**）——一个用于为局域网上所有计算机提供登录和安全性的中央网络服务器。关于将 samba 包用作 PDC 的更多信息可以在以下链接中找到。

## 另见

+   **Samba（软件）** ([`en.wikipedia.org/wiki/Samba_(software)`](http://en.wikipedia.org/wiki/Samba_(software)))：关于 Samba 软件套件的 Wikipedia 文章。

+   **nmbd – NetBIOS 通过 IP 命名服务** ([`manpages.debian.net/cgi-bin/man.cgi?query=nmbd`](http://manpages.debian.net/cgi-bin/man.cgi?query=nmbd))：`nmbd` 的 Debian 手册页面。

+   **samba – 用于 Unix 的 Windows SMB/CIFS 文件服务器** ([`manpages.debian.net/cgi-bin/man.cgi?query=samba`](http://manpages.debian.net/cgi-bin/man.cgi?query=samba)): Debian 的 samba 手册页。

+   **smb.conf – Samba 套件的配置文件** ([`manpages.debian.net/cgi-bin/man.cgi?query=smb.conf`](http://manpages.debian.net/cgi-bin/man.cgi?query=smb.conf)): Debian 的 `smb.conf` 手册页。

+   **smbd – 提供 SMB/CIFS 服务给客户端的服务器** ([`manpages.debian.net/cgi-bin/man.cgi?query=smbd`](http://manpages.debian.net/cgi-bin/man.cgi?query=smbd)): Debian 的 `smbd` 手册页。

+   **smbpasswd – 更改用户的 SMB 密码** ([`manpages.debian.net/cgi-bin/man.cgi?query=smbpasswd`](http://manpages.debian.net/cgi-bin/man.cgi?query=smbpasswd)): Debian 的 `smbpasswd` 手册页。

+   **System initialization** ([`www.debian.org/doc/manuals/debian-reference/ch04.en.html`](http://www.debian.org/doc/manuals/debian-reference/ch04.en.html)): *Debian 参考手册*中关于系统初始化的文章。

+   **Samba.org** ([`www.samba.org`](http://www.samba.org)): Samba 软件官网。

# 共享连接的 USB 硬盘（Samba）

本食谱扩展了默认的 Samba 配置，以包括共享 USB 硬盘。

上一个食谱展示了如何安装 Samba 并设置基本的文件共享。然而，之前使用的 Samba 配置仅共享用户的主目录中的文件。其他目录尚未通过 SMB（CIFS）协议开放。

本食谱扩展了之前食谱中使用的 Samba 配置，加入了一个新的文件共享定义，指向挂载在 Raspberry Pi 上的 USB 硬盘。

完成此食谱后，Raspberry Pi 可以作为文件服务器，供任何计算机通过 SMB（CIFS）协议共享文件（例如 Windows、Mac 和 Linux 计算机）。

## 准备工作

配料：

+   Raspberry Pi 的初始设置或基本网络设置，设备已经开机，并且你已作为用户 `pi` 登录（有关如何启动和登录的详细步骤，请参见 第一章 *安装与设置*，以及 第二章 *管理* 中的远程登录教程）。

+   一个网络连接（可选）。

+   一台与 Raspberry Pi 连接到同一网络的客户端 PC（可选）。

+   一个有电的 USB 集线器（推荐）。

+   至少一个 USB 硬盘驱动器（本示例使用 500 GB 的 USB 硬盘）。

本食谱不需要桌面 GUI，可以从基于文本的控制台或 LXTerminal 中运行。

如果 Raspberry Pi 的安全外壳（Secure Shell）服务器正在运行，并且它有网络连接，可以通过安全外壳客户端远程完成此食谱（见前文）。

本食谱中的示例使用了一个挂载在 `/media/bigdisk` 的 USB 硬盘（请参见本章开头的食谱 *挂载 USB 硬盘*）。

本配方中的示例还包括从本地网络中的另一台计算机（golden-macbook）测试访问新的文件共享。

树莓派应该已经安装了 Samba（请参见前一个配方）。

## 如何操作...

分享附加的 USB 磁盘的步骤如下：

1.  直接或远程登录到树莓派。

1.  使用`ls –l`命令列出已经挂载在`/media`目录下的磁盘驱动器。

    ```
    pi@raspberrypi ~ $ ls –l /media
    total 4
    drwxr-xr-x  6 pi staff 4096 May 30 12:23 bigdisk

    pi@raspberrypi ~ $ 
    ```

1.  使用 vi 编辑器编辑 Samba 配置文件`/etc/samba/smb.conf`。

    ```
    pi@raspberrypi ~ $ vi /etc/samba/smb.conf
    ```

1.  vi 编辑器显示配置文件的内容。关于如何使用编辑器的说明可以在 vi 手册页中找到；请参阅第二章中的配方*阅读内建文档*，*管理*。

    ```
    #
    # Sample configuration file for the Samba suite for Debian GNU/Linux.
    #
    #
    # This is the main Samba configuration file. You should read the
    # smb.conf(5) manual page in order to understand the options listed
    "/etc/samba/smb.conf" 333 lines, 12173 characters
    ```

1.  将新的共享配置添加到文件的底部：

    ```
    [bigdisk]
     comment = A really big disk!
     path = /media/bigdisk
     valid users = pi
     admin users = pi
     read only = No

    ```

    ```
    #======================= Share Definitions =======================

    [bigdisk]
       comment = A really big disk!
       path = /media/bigdisk
       valid users = pi
       admin users = pi
       read only = no

    [homes]
       comment = Home Directories
       browseable = no

    "/etc/samba/smb.conf" 340 lines, 12294 characters
    ```

1.  保存并退出 vi 编辑器。

1.  使用`/etc/init.d/samba reload`命令通知 Samba 服务器重新加载其配置文件。

    ```
    pi@raspberrypi ~ $ sudo /etc/init.d/samba reload
    [ ok ] Reloading /etc/samba/smb.conf: smbd only.

    pi@raspberrypi ~ $
    ```

1.  现在，可以从本地网络访问文件共享`\\raspberrypi\bigdisk`！请以用户`pi`身份连接。

    ```
    golden-macbook:~ golden$ smbutil view //pi@raspberrypi
    Password for 192.168.2.7: 
    Share                                     Type    Comments
    -------------------------------
    print$                                    Disk    Printer Drivers
    pi                                        Disk    Home Directories
    IPC$                                      Pipe    IPC Service (raspberrypi server)
    bigdisk                                   Disk    A really big disk!

    4 shares listed

    golden-macbook:~ golden$ 
    ```

## 它是如何工作的...

登录到树莓派后，配方列出了已经挂载在`/media`目录下的磁盘。这是树莓派桌面（GUI）在自动挂载 USB 磁盘时使用的目录，也是`pmount`命令（以及配方在启动时自动挂载 USB 磁盘）使用的目录。

然后，使用 vi 编辑器修改 Samba 配置文件`smb.conf`。新共享`[bigdisk]`的配置被添加到文件中。

+   `[bigdisk]`在`config`文件中启动一个新部分，并设置共享的名称为`bigdisk`。

+   `Comment =` 一个非常大的磁盘！定义了在其他计算机浏览共享时显示的注释。

+   `Path = /media/bigdisk`是树莓派上文件的位置。

+   `valid users = pi`声明只有用户`pi`可以访问这些文件。

+   `admin users = pi`授予用户`pi`对文件的管理员访问权限。

+   `read only = no`允许向共享中写入和删除文件。

配置保存并退出 vi 编辑器后，使用命令`/etc/init.d/samba reload`通知 Samba 服务器重新加载其配置。

一旦配置文件重新加载，Samba 服务器就准备好通过文件共享`bigdata`在 UNC 地址`\\raspberrypi\bigdata`上共享`/media/bigdata`目录中的文件。

最后，在另一台计算机（golden-macbook）上使用`smbutil`命令验证新的 bigdisk 共享是否可以从本地网络访问。

## 还有更多...

新共享中的文件是受保护的，要求其他计算机上的用户通过`pi`用户的 Samba 密码连接到共享。

### 注意

请记住：用户的 Samba 密码与登录密码是分开维护的。

使用`smbpasswd`命令创建和管理 Samba 密码（请参见前一个配方，*为示例创建文件服务器*）。
