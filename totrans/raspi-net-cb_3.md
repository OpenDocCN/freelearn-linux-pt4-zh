# 第三章：维护

本章将涵盖以下主题：

+   更新操作系统（`apt-get`）

+   使用 `sources.list` 从 `wheezy` 升级到 `jessie` 的 Raspbian

+   搜索软件包（`apt-cache`）

+   安装软件包（`apt-get`）

+   包管理（`aptitude`）

+   阅读内置文档（`man`）

+   阅读内置文档（`info`）

# 介绍

本章中的配方是关于树莓派的基本维护。

前几个配方展示了如何更新 Raspbian 操作系统并安装新软件包，而最后两个配方则展示了如何访问树莓派上已内置的文档。

### 注意

本章中的配方适用于基于 Debian 的操作系统，如 PiNet、Raspbian（树莓派基金会推荐）和 Ubuntu MATE。

其他操作系统，如 Pidora 和 RISC OS，拥有自己独立的更新和安装机制。有关如何在这些操作系统上更新和安装软件的说明，请访问它们各自的官方网站。该配方末尾的*另见*部分提供了相关参考链接。

完成本章配方后，你将能够更新树莓派，并更好地理解内置文档。

# 更新操作系统（apt-get）

本配方演示了如何使用 `apt-get` 命令更新树莓派。

树莓派及其上运行的操作系统，如 Raspbian Linux 发行版，正在迅速发展。每周甚至每天都有新的更新和安全补丁可以下载和安装。

本配方使用名为 `apt-get` 的高级包管理工具命令行实用程序，将树莓派上已安装的软件更新到最新版本。此配方不安装新软件，只是升级已安装的软件。配方*搜索软件包*和*安装软件包*分别展示了如何搜索和安装新软件包。

完成本配方后，你将能够使用 `apt-get` 高级包管理工具更新 Raspbian 操作系统。

## 准备工作

材料：

+   你需要为已开机的树莓派进行*初始设置*或*基础网络配置*。你还需要以用户名为 `pi` 的用户身份登录（可以参考第一章中的配方，*安装与设置*，了解如何启动和登录，并参考第二章中的配方，*管理*，了解如何远程登录）。

+   你还需要一个网络连接。

树莓派需要连接到互联网，以便访问更新服务器，从中获取软件更新和安全修复。

如果树莓派已启用远程访问，则可以使用`SSH`或 PuTTY（参见第二章中的*远程访问*食谱）在远程完成此操作。

## 如何操作...

按照以下步骤更新树莓派的操作系统：

1.  直接或远程登录到树莓派。

1.  执行`apt-get update`命令来更新本地软件包数据库，如下所示：

    ```
    golden@raspberrypi ~ $ sudo apt-get update
    Get:1 http://mirrordirector.raspbian.org jessie InRelease [15.0 kB]            
    Get:2 http://archive.raspberrypi.org jessie InRelease [13.3 kB]                
    Get:3 http://mirrordirector.raspbian.org jessie/main armhf Packages [8,961 kB]
    Get:4 http://archive.raspberrypi.org jessie/main Sources [31.2 kB]
    Get:5 http://archive.raspberrypi.org jessie/ui Sources [5,197 B]              
    Get:6 http://archive.raspberrypi.org jessie/main armhf Packages [101 kB]     
    Get:7 http://archive.raspberrypi.org jessie/ui armhf Packages [7,639 B]       
    Ign http://archive.raspberrypi.org jessie/main Translation-en_GB                                                       
    Ign http://archive.raspberrypi.org jessie/main Translation-en                                                          
    Ign http://archive.raspberrypi.org jessie/ui Translation-en_GB                                                         
    Ign http://archive.raspberrypi.org jessie/ui Translation-en                                                            
    Get:8 http://mirrordirector.raspbian.org jessie/contrib armhf Packages [37.5 kB]                                       
    Get:9 http://mirrordirector.raspbian.org jessie/non-free armhf Packages [70.2 kB]                                      
    Get:10 http://mirrordirector.raspbian.org jessie/rpi armhf Packages [1,356 B]                                          
    Ign http://mirrordirector.raspbian.org jessie/contrib Translation-en_GB                                                
    Ign http://mirrordirector.raspbian.org jessie/contrib Translation-en                                                   
    Ign http://mirrordirector.raspbian.org jessie/main Translation-en_GB                                                   
    Ign http://mirrordirector.raspbian.org jessie/main Translation-en                                                      
    Ign http://mirrordirector.raspbian.org jessie/non-free Translation-en_GB                                               
    Ign http://mirrordirector.raspbian.org jessie/non-free Translation-en                                                  
    Ign http://mirrordirector.raspbian.org jessie/rpi Translation-en_GB                                                    
    Ign http://mirrordirector.raspbian.org jessie/rpi Translation-en                                                       
    Fetched 9,243 kB in 37s (245 kB/s)                                                                                     
    Reading package lists... Done

    pi@raspberrypi ~ $ 
    ```

1.  执行`apt-get –y dist-upgrade`命令来升级系统，如下所示：

    ```
    golden@raspberrypi ~ $ sudo apt-get dist-upgrade -y

    Reading package lists... Done
    Building dependency tree       
    Reading state information... Done
    Calculating upgrade... Done
    ```

1.  可升级的软件包列表的计算方式如下：

    ```
    The following packages will be upgraded:
      cups-bsd cups-client cups-common fuse libcups2 libcupsimage2 libfuse2
      libsqlite3-0 libssl1.0.0 openssl raspi-config
    11 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
    Need to get 3,877 kB of archives.
    After this operation, 455 kB disk space will be freed.
    ```

1.  每个软件包下载时，都会显示状态信息，如下所示：

    ```
    Get:1 http://mirrordirector.raspbian.org/raspbian/ jessie/main libssl1.0.0 armhf 1.0.1e-2+rvt+deb7u17 [1,053 kB]
    Get:2 http://archive.raspberrypi.org/debian/ jessie/main raspi-config all 20150131-4 [13.2 kB]
    …

    Get:11 http://mirrordirector.raspbian.org/raspbian/ jessie/main openssl armhf 1.0.1e-2+rvt+deb7u17 [702 kB]
    Fetched 3,877 kB in 3s (1,060 kB/s)
    ```

1.  在下载可升级软件包后，它们将被预配置并准备好安装：

    ```
    Preconfiguring packages ...
    (Reading database ... 77851 files and directories currently installed.)
    Preparing to replace libssl1.0.0:armhf 1.0.1e-2+rvt+deb7u16 (using .../libssl1.0.0_1.0.1e-2+rvt+deb7u17_armhf.deb) ...
    Unpacking replacement libssl1.0.0:armhf ...
    Preparing to replace libsqlite3-0:armhf 3.7.13-1+deb7u1 (using .../libsqlite3-0_3.7.13-1+deb7u2_armhf.deb) ...
    Unpacking replacement libsqlite3-0:armhf ... 
    …
    ```

1.  在软件包预配置并准备好安装后，它们将被安装：

    ```
    Processing triggers for man-db ...
    Processing triggers for initramfs-tools ...
    Setting up libssl1.0.0:armhf (1.0.1e-2+rvt+deb7u17) ...
    Setting up libsqlite3-0:armhf (3.7.13-1+deb7u2) ...
    Setting up libcups2:armhf (1.5.3-5+deb7u6) ...
    Setting up libcupsimage2:armhf (1.5.3-5+deb7u6) ...
    Setting up cups-common (1.5.3-5+deb7u6) ...
    Setting up cups-client (1.5.3-5+deb7u6) ...
    Setting up cups-bsd (1.5.3-5+deb7u6) ...
    Setting up libfuse2:armhf (2.9.0-2+deb7u2) ...
    Setting up fuse (2.9.0-2+deb7u2) ...
    udev active, skipping device node creation.
    update-initramfs: deferring update (trigger activated)
    Setting up openssl (1.0.1e-2+rvt+deb7u17) ...
    Setting up raspi-config (20150131-4) ...

    golden@raspberrypi ~ $ 
    ```

1.  当`apt-get dist-upgrade`命令完成时，树莓派上当前可升级的软件将被完全安装和配置，所有当前的安全修复也将被应用。

1.  为确保树莓派使用所有可升级的软件包，重启系统。使用`reboot`命令来重启树莓派，如下所示：

    ```
    pi@raspberrypi ~ $ sudo reboot

    Broadcast message from root@raspberrypi (pts/0) (Sun Jul  5 12:24:10 2015):
    The system is going down for reboot NOW!

    pi@raspberrypi ~ $ 

    Connection to 192.168.2.8 closed by remote host.
    Connection to 192.168.2.8 closed.

    golden-macbook:~ rick$ 
    ```

1.  当树莓派重启后，升级完成！

## 它是如何工作的...

登录到树莓派后，使用`apt-get`更新命令来更新本地软件包数据库，该数据库是当前可用软件包的本地副本。

在本地软件包数据库更新后，使用`apt-get dist-upgrade`命令来确定哪些软件包可以升级，下载可升级的软件包，并预配置和安装这些软件包。

当`apt-get dist-upgrade`命令完成时，树莓派会重启，以确保它使用所有新升级的软件包和安全修复。

## 还有更多内容…

Raspbian Linux 发行版，像大多数树莓派的操作系统发行版一样，是作为一系列软件包的集合进行组织的。每个软件包包含一个或多个应用程序及其配置文件和支持库。每个软件包还会标明其当前版本以及对其他软件包的依赖关系。**高级软件包工具**（`apt`）及其支持工具，如`apt-get`，用于管理 Raspbian Linux 发行版的软件包。

## 另见

+   **apt – 高级软件包工具** ([`manpages.debian.net/cgi-bin/man.cgi?query=apt`](http://manpages.debian.net/cgi-bin/man.cgi?query=apt)): 这是 Debian 对`apt`的手册页面。

+   **apt-get – APT 软件包管理** ([`manpages.debian.net/cgi-bin/man.cgi?query=apt-get`](http://manpages.debian.net/cgi-bin/man.cgi?query=apt-get)): 这是 Debian 对`apt-get`的手册页面。

# 使用 sources.list 将 Raspbian 从 wheezy 升级到 jessie

本食谱展示了如何将树莓派 2 从 Raspbian Linux 发行版的`jessie`版本升级到`stretch`版本。

目前，有三个版本的 Raspbian 操作系统可供树莓派 2 使用，分别是`wheezy`、`jessie`和`stretch`。这些发行版的源地址在[`mirrordirector.raspbian.org/raspbian`](http://mirrordirector.raspbian.org/raspbian)。

`wheezy`发行版基于 Debian 7，`jessie`基于 Debian 8，而`stretch`基于 Debian 9。目前的`stable`版本是`jessie`。默认的`wheezy`版本被认为是`oldstable`发行版。下一个计划发布的版本是`testing`，即`stretch`。

### 提示

如果你计划将操作系统从一个版本升级到另一个版本，包括从`jessie`到`stretch`，你应该在安装其他软件包之前进行升级，因为这样将大大简化整体升级过程。

树莓派基金会为`wheezy`和`jessie`版本的操作系统提供了默认镜像。本食谱中的说明适用于你想保持现有`wheezy`配置但尝试升级到`jessie`的情况。或者，当你想尝试`stretch`，即 Raspbian 操作系统的下一个版本时，也可以使用这些说明。

目前，Raspbian 没有升级操作系统版本的工具。升级一个版本到另一个版本（比如从`wheezy`到`jessie`，即从`old stable`到`stable`）的唯一方法是按照本食谱中的说明进行操作。

### 注意

这次升级将修改超过一千个软件包。升级需要至少 4 小时！

将操作系统从一个版本升级到另一个版本是有风险的。中断版本升级可能会使系统处于不稳定状态。然而，如果你想尝试最新的操作功能或升级已经配置好的操作系统，这个风险可能是值得的。

### 注意

小于 8 GB 的 SD 卡空间不足以完成升级。如果在升级之前已经安装了额外的包，那么升级会需要更长的时间。

完成本食谱后，你将把 Raspbian 从一个版本升级到另一个版本：从`wheezy`到`jessie`，或者从`jessie`到`stretch`。

## 准备工作

材料：

+   你需要一个已启动的树莓派，且该设备已进行*初始设置*或*基本网络配置*。同时，你还需要以用户名为`pi`的用户登录（可以参考第一章中的食谱，*安装与设置*，了解如何启动和登录，以及参考第二章中的食谱，*管理*，了解如何远程登录）。

+   还需要一个网络连接。

Raspberry Pi 需要连接互联网，才能访问更新服务器以拉取软件更新和安全修复。

如果 Raspberry Pi 已启用远程访问，可以使用 `ssh` 或 PuTTY 远程完成此操作（可以查看 第二章中的 *远程访问* 配方，*管理*）。

### 注意

本配方使用 `vi` 文本编辑器修改配置文件。也可以使用其他文本编辑器。Raspberry Pi 上可用的另一个编辑器是 `nano` 文本编辑器。

关于编辑器使用的文档可以在编辑器本身中找到（在 `vi` 中输入 **:help** 或在 `nano` 中按 **<ctrl-g>** 获取帮助）。

## 如何操作...

执行以下步骤，将 Raspbian Linux 从一个版本升级到另一个版本：

1.  直接或远程登录到 Raspberry Pi。

1.  使用 vi（或其他编辑器）修改 `/etc/apt/sources.list`。从 wheezy 升级到 jessie 时，将每行第三个字段的值从 wheezy 改为 jessie。 从 jessie 升级到 `stretch` 时，将第三个字段的值改为 `stretch`，如下所示：

    ```
    pi@raspberrypi:~$ sudo vi /etc/apt/sources.list

    deb http://mirrordirector.raspbian.org/raspbian/ stretch main contrib non-free rpi
    # Uncomment line below then 'apt-get update' to enable 'apt-get source'
    #deb-src http://archive.raspbian.org/raspbian/ stretch main contrib non-free rpi
    ~                                                                               
    ~                                                                               
    ~                                                                               
    "/etc/apt/sources.list" 3 lines, 234 characters
    ```

1.  使用 `vi`（或其他编辑器）修改 `/etc/apt/sources.list.d/raspi.list`。从 `wheezy` 升级到 `jessie` 时，将每行第三个字段的值从 `wheezy` 改为 `jessie`。从 `jessie` 升级到 `stretch` 时，*此次不要修改此文件*：

    ```
    pi@raspberrypi:~$ sudo vi /etc/apt/sources.list.d/raspi.list

    deb http://archive.raspberrypi.org/debian/ jessie main
    # Uncomment line below then 'apt-get update' to enable 'apt-get source'
    #deb-src http://archive.raspberrypi.org/debian/ jessie main
    ~                                                                               
    ~                                                                               
    ~                                                                               
    "/etc/apt/sources.list.d/raspi.list" 3 lines, 187 characters
    ```

1.  如果你从 `wheezy` 升级到 `jessie`，请使用 `rm` 命令删除 `collabora.list` 配置文件。如果你从 `jessie` 升级到 `stretch`，此文件不存在。因此，你需要 *跳过此步骤*：

    ```
    pi@raspberrypi ~ $ sudo rm /etc/apt/sources.list.d/collabora.list 

    pi@raspberrypi ~ $ 
    ```

1.  现在，按照前面的配方更新操作系统（*更新操作系统*）。

    ### 提示

    请注意，此更新可能会超过 4 小时！

## 它是如何工作的...

登录 Raspberry Pi 后，会更改 `apt` 的 `sources.list` 配置。

`/etc/apt/sources.list` 文件已被修改，使其引用下一个版本的 Debian。例如，如果前一个版本是 `wheezy`，则改为 `jessie`；如果前一个版本是 `jessie`，则改为 `stretch`。

`/etc/apt/sources.list.d/raspi.list` 文件只有在前一个版本为 `wheezy` 时才会被修改。即使你升级到 `stretch`，该文件应引用 Debian 版本，也就是 `jessie`。

### 注意

在写这本书时，如果你从 `jessie` 升级到 `stretch`，`raspi.list` 文件无需更改。如果你在 2016 年春季后使用此配方，你需要查看 Debian 仓库 [`archive.raspberrypi.org/debian/dists/`](http://archive.raspberrypi.org/debian/dists/) 以检查是否有 `stretch` 发行版文件夹。如果有该文件夹，更新此文件的第三个字段为 `stretch`。

从 `wheezy` 升级到 `jessie` 时，使用 `rm` 命令删除 `collabora.list` 配置文件。对于 `jessie` 版本的 Raspbian 操作系统，此文件不存在。因此，可以跳过此步骤。

更改源之后，使用之前的食谱（*更新操作系统*）来更新操作系统。

## 还有更多……

`apt-get update` 命令用于从 `/etc/apt/sources.list` 文件中配置的软件分发站点和位于 `/etc/apt/sources.list.d` 目录中的文件中获取当前的软件目录。

### sources.list 和 sources.list.d

位于 `/etc/apt` 目录中的 `sources.list` 文件指向来自 `raspbian.org` 的基础操作系统分发，而位于 `/etc/apt/sources.list.d` 目录中的文件（`raspi.list`）则指向附加的 Raspberry Pi 软件包位置。

在以下终端会话中，使用 `cat` 命令显示 `wheezy` 版本的 Raspbian Linux 分发的源文件内容：

```
pi@raspberrypi ~ $ cat /etc/apt/sources.list

deb http://mirrordirector.raspbian.org/raspbian/ wheezy main contrib non-free rpi
# Uncomment line below then 'apt-get update' to enable 'apt-get source'
#deb-src http://archive.raspbian.org/raspbian/ wheezy main contrib non-free rpi

pi@raspberrypi ~ $ ls /etc/apt/sources.list.d/

collabora.list  raspi.list

pi@raspberrypi ~ $ cat /etc/apt/sources.list.d/collabora.list 

deb http://raspberrypi.collabora.com wheezy rpi

pi@raspberrypi ~ $ cat /etc/apt/sources.list.d/raspi.list 

deb http://archive.raspberrypi.org/debian/ wheezy main
# Uncomment line below then 'apt-get update' to enable 'apt-get source'
#deb-src http://archive.raspberrypi.org/debian/ wheezy main

pi@raspberrypi ~ $ 
```

这些配置文件中的每个文件描述了一个或多个软件包仓库的位置。每个文件中优先列出最受推荐的仓库。`sources.list` 文件优于 `sources.list.d` 目录中的文件。

### sources.list 文件格式

这些文件的格式很简单。每一行描述了一个软件分发库中的软件包集合，提供集合的类型、集合名称以及集合包的位置。

每行中的第一项定义了集合的类型。对于基于 Debian 的分发，如 Raspbian Linux 分发，定义为 `deb` 或 `deb-src`。`deb-src` 目录包含源代码和已编译的二进制文件。

每行中的第二项是软件分发库的根位置。`sources.list` 文件指向位于 `mirrordirector.raspbian.org` 网站上的 `raspbian` 操作系统分发的根目录。`collabora.list` 文件指向由 Collabora 提供的位于 `raspberrypi.collabora.com` 网站上的软件包。此外，`raspi.list` 文件指向由 Raspberry Pi 基金会提供的附加 Raspberry Pi 特定软件包，这些软件包位于 `archive.raspberrypi.org` 网站上。

每行中的第三项是你想使用的 Debian 操作系统版本的名称。所有文件应指定相同的版本。在撰写本书时，`jessie` 被认为是当前稳定版的 Debian。一个新的版本，名为 `stretch`，将很快发布。

该行的其余部分是一个分发组件列表。基本发行版的配置文件命名为 `source.list`，它指向 Raspbian Linux 发行版，并定义了多个组件。每个二级配置文件，即 `collabra.list` 和 `raspi.list`，指向更小的 Raspberry Pi 特定包的发行版。每个二级配置文件只有一个组件，即 `collabra.list` 为 `rpi`，`raspi.list` 为 `main`。

Debian 操作系统的标准软件包集合如下：

+   `main`：这是由核心团队支持的基础发行版。

+   `contrib`：包含由外部人员贡献给 Debian 的包，这些包由核心团队以外的人员支持。

+   `non-free`：包含不属于开源或有某些版权限制的包，这些包也由核心团队以外的人员支持。

+   `rpi`：包含 Raspberry Pi 特定的包，这些包由 Raspberry Pi 社区支持。

+   `rpi2`：包含专为新的 Raspberry Pi 2 芯片组设计并由 Raspberry Pi 社区支持的 Raspberry Pi 包。

## 另见

+   **sources.list – APT 包资源列表** ([`manpages.debian.net/cgi-bin/man.cgi?query=sources.list`](http://manpages.debian.net/cgi-bin/man.cgi?query=sources.list)): Debian 的 `sources.list` 手册页面描述了配置文件的格式。

+   **vim – VI 改进版** ([`manpages.debian.net/cgi-bin/man.cgi?query=vi`](http://manpages.debian.net/cgi-bin/man.cgi?query=vi)): Debian 的 `vi` 手册页面描述了文本编辑器的基本使用方法。

+   **cat – 连接文件并打印到标准输出** ([`manpages.debian.net/cgi-bin/man.cgi?query=cat`](http://manpages.debian.net/cgi-bin/man.cgi?query=cat)): Debian 的 cat 手册页面描述了如何将文件打印到标准输出。

+   **Debian – 通用操作系统** ([`www.debian.org/`](https://www.debian.org/)): Raspbian Linux 操作系统发行版基于 Debian。

+   **Debian 发行版** ([`www.debian.org/releases/`](https://www.debian.org/releases/)): Debian 发行版的命名规则在这里进行了说明。

+   **Raspberry Pi 基金会论坛** ([`www.raspberrypi.org/forums/`](https://www.raspberrypi.org/forums/)): 你可以在 Raspberry Pi 基金会的论坛上报告 bug、错误或其他软件缺陷，或提出改进建议。

+   **Pidora 是为 Raspberry Pi 优化的 Fedora Remix 版本** ([`pidora.ca/`](http://pidora.ca/)): Pidora 不是基于 Debian 的操作系统。它基于 Fedora，这是 Red Hat Linux 企业版的开源版本。

+   **RISC OS – 一款快速且易于定制的 ARM 设备操作系统** ([`www.riscosopen.org/content/`](https://www.riscosopen.org/content/)): RISC OS 是为 ARM 设备设计的原始操作系统，诞生于 1980 年代。

+   **Collabora (**[`www.collabora.com/`](https://www.collabora.com/)): Collabora 支持多个开源项目，包括专门为 Raspberry Pi 做出的贡献。

# 搜索软件包（apt-cache）

这个教程展示了如何使用 `apt-cache` 搜索软件包。

Raspbian Linux 发行版的完整软件目录包含了大量预构建的软件包，这些软件包可以直接下载和安装。通过 `apt-cache` 命令，可以通过关键词搜索这个庞大的软件目录。

这个教程特别针对在 Raspbian Linux 发行版的所有软件包名称中搜索关键字“fortune”。如果你想搜索其他软件包，可以将“fortune”关键字替换为你感兴趣的关键字。

完成此教程后，你将能够通过使用 `apt-cache search` 命令搜索关键字来定位软件包。

## 准备工作

配料：

+   你需要为已开机的 Raspberry Pi 设置 *初始设置* 或 *基础网络设置*，并且需要以用户名为 `pi` 的用户身份登录（有关如何启动和登录的信息，请参见第一章的教程，*安装和设置*部分；有关如何远程登录的信息，请参见第二章的教程，*管理*部分）。

+   你还需要一个网络连接。

Raspberry Pi 需要连接到互联网，以便访问更新服务器，从而下载软件更新和安全修复。

如果 Raspberry Pi 启用了远程访问功能，可以通过 `ssh` 或 PuTTY 在远程完成这个教程（请参见第二章的*远程访问*教程，*管理*部分）。

## 如何操作...

执行以下步骤来搜索 Raspbian Linux 软件包：

1.  直接或远程登录到 Raspberry Pi。

1.  使用 `apt-get update` 命令更新软件目录，如本章第一个教程中所述（*更新操作系统*），如下所示：

    ```
    pi@raspberrypi ~ $ sudo apt-get update
    ```

1.  执行 `apt-cache search --names-only fortune` 命令。搜索缓存不需要超级用户权限：

    ```
    pi@raspberrypi ~ $ apt-cache search -–names-only fortune

    fortune-mod - provides fortune cookies on demand
    fortunes - Data files containing fortune cookies
    fortunes-min - Data files containing selected fortune cookies
    fortunes-off - Data files containing offensive fortune cookies
    fortune-zh - Chinese Data files for fortune
    fortunes-bg - Bulgarian data files for fortune
    fortunes-bofh-excuses - BOFH excuses for fortune
    fortunes-br - Data files with fortune cookies in Portuguese
    fortunes-cs - Czech and Slovak data files for fortune
    fortunes-de - German data files for fortune
    fortunes-debian-hints - Debian Hints for fortune
    fortunes-eo - Collection of esperanto fortunes.
    fortunes-eo-ascii - Collection of esperanto fortunes (ascii encoding).
    fortunes-eo-iso3 - Collection of esperanto fortunes (ISO3 encoding).
    fortunes-es - Spanish fortune database
    fortunes-es-off - Spanish fortune cookies (Offensive section)
    fortunes-fr - French fortunes cookies
    fortunes-ga - Irish (Gaelige) data files for fortune
    fortunes-it - Data files containing Italian fortune cookies
    fortunes-it-off - Data files containing Italian fortune cookies, offensive section
    fortunes-mario - Fortunes files from Mario
    fortunes-pl - Polish data files for fortune
    fortunes-ru - Russian data files for fortune
    libfortune-perl - Perl module to read fortune (strfile) databases

    golden@raspberrypi ~ $
    ```

1.  该命令显示了一个包含**fortune**名称的软件包列表。

## 工作原理...

登录到 Raspberry Pi 后，使用本章中的第一个教程更新 Raspbian Linux 软件目录，即 *更新操作系统*。

一旦软件目录更新完成，使用 `apt-cache search` 命令可以搜索包含 `fortune` 关键字的软件包名称（`--names-only`）。

搜索返回的一个软件包，**fortune-mod**，是一个命令行工具，可以按需提供幸运饼干。

以 **fortunes-** 开头的软件包包含了 `fortune` 命令行工具在显示幸运饼干时从中选择的命运集合。

## 还有更多…

在这个食谱中，我们在 Raspbian Linux 软件目录中搜索了包名包含 "fortune" 的软件包。由于指定了 `--names-only` 选项，搜索仅限于包名。当没有使用 `--names-only` 选项时，`apt-cache` 命令会在软件包的摘要中也搜索指定的关键词。

`apt-cache search` 命令接受多个关键词作为参数。如果给定多个关键词，结果会缩小到只包含所有关键词的软件包。以下是如何搜索德国语言的幸运语句的示例：

```
pi@raspberrypi:~$ apt-cache search fortunes german

fortunes-de - German data files for fortune

pi@raspberrypi:~$
```

安装 `fortune` 包的详细步骤请参见下一个食谱。

## 另见

+   **apt-cache – 查询 APT 缓存 (**[`manpages.debian.net/cgi-bin/man.cgi?query=apt-cache`](http://manpages.debian.net/cgi-bin/man.cgi?query=apt-cache)) `apt-cache` 命令可以从本地 Debian 软件目录的元数据中生成有趣的输出。Debian 的 `apt-cache` 手册页描述了该命令及其选项。

# 安装软件包（apt-get）

本食谱演示了如何使用 `apt-get install` 命令安装新的软件包。

除了更新软件包目录和升级已经安装的软件包外，`apt-get` 命令还可以用来安装新的软件包。

在这个食谱中，`fortune-mod` 软件包是通过 `apt-get install` 命令安装的。其他软件包也可以使用这个食谱安装；只需将 `fortune-mod` 包替换为你想要安装的软件包即可。

### 提示

在开始安装新的软件包之前，最好先使用 `apt-get update` 命令更新软件包目录，然后使用 `apt-get dist-upgrade` 命令升级现有的软件包（有关更多信息，请参考 *更新操作系统* 食谱）。

完成这个食谱后，你将能够使用 `apt-get install` 命令安装新的软件包。

## 准备工作

配料：

+   你需要为已开机的树莓派准备 *初始设置* 或 *基本网络设置*。同时，你还需要以用户名为 `pi` 的用户身份登录（查看第一章中的食谱，了解如何启动和登录，及第二章中的食谱，学习如何远程登录）。

+   还需要一个网络连接。

树莓派需要访问互联网，才能从更新服务器获取软件更新和安全修复。

如果树莓派启用了远程访问功能，可以通过 `ssh` 或 PuTTY 远程完成这个食谱（参见第二章中的 *远程访问* 食谱，*管理*）。

## 如何操作…

执行以下步骤以安装软件包：

1.  直接或远程登录到树莓派。

1.  使用 `apt-get update` 命令更新软件目录，正如本章第一条配方中所述（*更新操作系统*）：

    ```
    golden@raspberrypi ~ $ sudo apt-get update
    ```

1.  使用 `apt-get install –y` 命令安装 `fortune-mod` 软件包。该命令的 `–y` 选项会自动对所有安装问题回答“是”：

    ```
    golden@raspberrypi:~$ sudo apt-get install -y fortune-mod
    ```

1.  该命令计算包的依赖关系以及该包和其配置文件将使用的额外空间：

    ```
    Reading package lists... Done
    Building dependency tree       
    Reading state information... Done
    The following extra packages will be installed:
      fortunes-min librecode0
    Suggested packages:
      fortunes x11-utils
    The following NEW packages will be installed:
      fortune-mod fortunes-min librecode0
    0 upgraded, 3 newly installed, 0 to remove and 0 not upgraded.
    Need to get 599 kB of archives.
    After this operation, 1,588 kB of additional disk space will be used.
    ```

1.  该命令随后下载必要的软件包，即 **fortunes-mod**、**fortunes-min** 和 **librecode0**：

    ```
    Get:1 http://ftp.debian.org/debian/ jessie/main librecode0 armhf 3.6-21 [477 kB]
    Get:2 http://ftp.debian.org/debian/ jessie/main fortune-mod armhf 1:1.99.1-7 [47.4 kB]
    Get:3 http://ftp.debian.org/debian/ jessie/main fortunes-min all 1:1.99.1-7 [74.3 kB]
    Fetched 599 kB in 2s (252 kB/s)    
    ```

1.  下载的文件被解压并预配置：

    ```
    Selecting previously unselected package librecode0:armhf.
    (Reading database ... 19391 files and directories currently installed.)
    Preparing to unpack .../librecode0_3.6-21_armhf.deb ...
    Unpacking librecode0:armhf (3.6-21) ...
    Selecting previously unselected package fortune-mod.
    Preparing to unpack .../fortune-mod_1%3a1.99.1-7_armhf.deb ...
    Unpacking fortune-mod (1:1.99.1-7) ...
    Selecting previously unselected package fortunes-min.
    Preparing to unpack .../fortunes-min_1%3a1.99.1-7_all.deb ...
    Unpacking fortunes-min (1:1.99.1-7) ...
    ```

1.  然后，软件包及其依赖项被安装，并且命令执行完成：

    ```
    Processing triggers for man-db (2.7.0.2-5) ...
    Setting up librecode0:armhf (3.6-21) ...
    Setting up fortune-mod (1:1.99.1-7) ...
    Setting up fortunes-min (1:1.99.1-7) ...
    Processing triggers for libc-bin (2.19-18) ...

    golden@raspberrypi:~$ 
    ```

1.  最后，测试 `fortune` 命令，检查它是否正常工作：

    ```
    golden@raspberrypi:~$ fortune

    You single-handedly fought your way into this hopeless mess.

    golden@raspberrypi:~$ fortune

    Your best consolation is the hope that the things you failed to get weren't
    really worth having.

    golden@raspberrypi:~$ fortune

    Q:	How many hardware engineers does it take to change a light bulb?
    A:	None.  We'll fix it in software.

    Q:	How many system programmers does it take to change a light bulb?
    A:	None.  The application can work around it.

    Q:	How many software engineers does it take to change a light bulb?
    A:	None.  We'll document it in the manual.

    Q:	How many tech writers does it take to change a light bulb?
    A:	None.  The user can figure it out.

    golden@raspberrypi:~$ 
    ```

## 它是如何工作的……

登录到树莓派后，使用本章第三章中的第一条配方更新 Raspbian Linux 软件目录，*更新操作系统*。

一旦软件目录更新完成，就使用 `apt-get install` 命令安装 `fortune-mod` 软件包。

该命令首先计算包的依赖关系，确定在安装 **fortune-mod** 包之前需要安装哪些其他包。

`apt-get install` 命令继续下载 `fortunes-mod` 包以及它所依赖的两个包（`fortune-min` 和 `librecode0`）。

然后，命令解压并预配置这三个包。最后，`apt-get install` 命令安装软件包。安装完成后，测试 `fortune` 命令。

## 还有更多…

`apt-get install` 命令使用与 `apt-cache search` 命令相同的本地软件目录。这两个命令都依赖于 `apt-get update` 命令来下载更新的包信息。

## 另见

+   **fortune – 从文件中获取示例行** ([`manpages.debian.net/cgi-bin/man.cgi?query=fortune`](http://manpages.debian.net/cgi-bin/man.cgi?query=fortune)) Debian 手册页面列出了 `fortune` 命令的所有选项。

# 包管理（aptitude）

本配方使用 `aptitude` 前端查找并安装 `pianobar` 应用程序。

有许多 Advance Package Tool（`apt`）的前端，它们提供了丰富的用户界面。这些前端背后仍然调用 `apt-get` 命令和 `apt-cache` 命令，但它们将所有 `apt` 工具的功能整合到一个用户界面中。

### 提示

如果你的树莓派上没有安装 `aptitude` 应用程序，使用上一条配方中的说明（*安装软件包*）安装 `aptitude` 软件包及其依赖项。

完成本食谱后，你将能够使用`aptitude`应用程序查找并安装软件包。

## 准备工作

配料：

+   你需要为已开启的树莓派准备*初始设置*或*基础网络配置*。你还需要以用户名为`pi`的用户身份登录（请参阅第一章，*安装与设置*中的食谱，了解如何启动和登录，以及第二章，*管理*中的食谱，了解如何远程登录）

+   还需要网络连接。

树莓派需要访问互联网，以连接更新服务器，下载新的软件包、软件更新和安全修复。

如果树莓派启用了远程访问，则可以使用`ssh`或 PuTTY 远程完成此食谱（请参阅第二章，*管理*中的*远程访问*食谱）。

## 如何做...

执行以下步骤以使用`aptitude`管理软件包：

1.  登录到树莓派，可以是直接登录或远程登录。

1.  执行`aptitude`命令。该命令需要以管理员身份运行：

    ```
    golden@raspberrypi:~$ sudo aptitude
    ```

1.  显示`aptitude`应用程序的主屏幕：

    ```
    Actions  Undo  Package  Resolver  Search  Options  Views  Help
    C-T: Menu  ?: Help  q: Quit  u: Update  g: Download/Install/Remove Pkgs
    aptitude 0.6.11
    --- New Packages (1)                                                            
    --- Installed Packages (317)
    --- Not Installed Packages (49041)
    --- Virtual Packages (7075)
    --- Tasks (216)

    These packages have been added to Debian since the last time you cleared the   
    list of "new" packages (choose "Forget new packages" from the Actions menu to  
    empty this list).                                                              

    This group contains 1 package.
    ```

1.  按**u**更新本地软件包目录缓存。屏幕将显示正在下载的状态信息。

1.  输入**/pianobar**搜索`pianobar`软件包。然后，按**<enter>**键跳转到第一个找到的包：

    ```
    Actions  Undo  Package  Resolver  Search  Options  Views  Help
    C-T: Menu  ?: Help  q: Quit  u: Update  g: Download/Install/Remove Pkgs
    aptitude 0.6.11
    p     pianobar-dbg                          <none>         2014.06.08-1+b
    p     pianobooster-dbg                      <none>         0.6.4b-2
    p     pidgin-dbg                            <none>         2.10.11-1
    p     pidgin-microblog-dbg                  <none>         0.3.0-3
    p     pidgin-mra-dbg                        <none>         20100304-1
    p ┌───────────────────────────────────────────────────────────────────┐
    p │Search for:                                                        │
    p │pianobar                                                           │
    co│              [ Ok ]                             [ Cancel ]        │
    pi└─────────────────────────────────────────────────── ───────────────┘ 
    Pandora, supporting all important features the official Flash™ client has:     

    * Create, delete, rename stations and add more music                           
    * Rate and temporary ban tracks as well as move them to another station        
    * "Shared stations"                                                            
    ```

1.  按**n**直到选择**pianobar**软件包。每次按下‘**n**’键时，搜索将继续到下一个符合条件的`pianobar`软件包。搜索将循环经过最后一个符合`pianobar`的包并继续寻找第一个符合条件的包。

1.  按**+**选择要安装的软件包。字母“i”出现在软件包名称旁边，表示已选择该软件包进行安装：

    ```
    Actions  Undo  Package  Resolver  Search  Options  Views  Help
    C-T: Menu  ?: Help  q: Quit  u: Update  g: Download/Install/Remove Pkgs
    aptitude 0.6.11                 Will use 59.7 MB of disk space DL Size: 18.8 MB
    pi    pianobar                           +119 kB   <none>         2014.06.08-1+b
    p     picard                                       <none>         1.2-2+b8
    p     plait                                        <none>         1.6.2-1
    p     playmidi                                     <none>         2.4debian-10
    p     pmidi                                        <none>         1.6.0-5
    p     pms                                          <none>         0.42-1
    p     poc-streamer                                 <none>         0.4.2-3
    p     pocketsphinx                                 <none>         0.8-5
    p     pocketsphinx-hmm-en-hub4wsj                  <none>         0.8-5
    p     pocketsphinx-hmm-en-tidigits                 <none>         0.8-5
    console based player for Pandora radio 
    pianobar is a cross-platform console client for the personalized web radio     
    Pandora, supporting all important features the official Flash™ client has:     

    * Create, delete, rename stations and add more music                           
    * Rate and temporary ban tracks as well as move them to another station        
    * "Shared stations"                                                            
    ```

1.  按**g**预览所选软件包的安装。依赖包的列表将显示：

    ```
    Actions  Undo  Package  Resolver  Search  Options  Views  Help
    C-T: Menu  ?: Help  q: Quit  u: Update  g: Download/Install/Remove Pkgs
                    Packages                                Preview
    aptitude 0.6.11                 Will use 59.7 MB of disk space DL Size: 18.8 MB
    --\ Packages being automatically installed to satisfy dependencies (55)         
    piA  libao-common                        +49.2 kB  <none>         1.1.0-3       
    piA  libao4                              +113 kB   <none>         1.1.0-3       
    piA  libavcodec56                        +7,131 kB <none>         6:11.4-1~deb8u
    piA  libavfilter5                        +324 kB   <none>         6:11.4-1~deb8u
    piA  libavformat56                       +1,254 kB <none>         6:11.4-1~deb8u
    piA  libavresample2                      +131 kB   <none>         6:11.4-1~deb8u
    piA  libavutil54                         +226 kB   <none>         6:11.4-1~deb8u
    piA  libdrm-freedreno1                   +75.8 kB  <none>         2.4.58-2      
    piA  libdrm-nouveau2                     +84.0 kB  <none>         2.4.58-2      
    ```

    ```
    These packages are being installed because they are required by another package
    you have chosen for installation.                                              

    This group contains 55 packages.                                               

    If you select a package, an explanation of its current state will appear in this space.
    ```

1.  第二次按**g**键以完成安装。

1.  `aptitude`应用程序下载所选的软件包**pianobar**及其依赖项。然后，它清除屏幕并运行`apt-get install`命令以完成安装。安装完成后，按**<enter>**键返回`aptitude`：

    ```
    Setting up libao-common (1.1.0-3) ...
    Setting up libao4 (1.1.0-3) ...
    Setting up pianobar (2014.06.08-1+b1) ...
    Setting up va-driver-all:armhf (1.4.1-1) ...
    Processing triggers for libc-bin (2.19-18) ...
    Processing triggers for systemd (215-17+deb8u1) ...
    Press Return to continue.
    ```

1.  返回到`aptitude`后，按**q**然后按**y**退出。

## 它是如何工作的…

登录到树莓派后，启动`aptitude`应用程序。

启动`aptitude`应用程序时，应用程序会首先花一些时间进行初始化，然后显示应用程序的主屏幕。

主屏幕顶部有一个工具栏，包含菜单项列表、常用命令列表（包括**?: 帮助**和**q: 退出**—按'**?**'获取帮助，按'**q**'退出）以及应用程序的版本号，版本是**aptitude 0.6.11**。

该应用程序通过以下命令键进行导航和控制：

+   要获取应用程序的帮助，按**?**（问号）

+   使用箭头键浏览包。

+   按**u**（小写字母 u）来更新包缓存。

+   按**U**（大写字母 U）选择已更新的包。

+   按**/**（斜杠）搜索包。

+   要进入下一个找到的包，按**n**（小写字母 n）。

+   要返回到上一个找到的包，按**p**（小写字母 p）

+   通过按**+**（加号）选择单个包；按**-**（减号）取消选择高亮的包。

+   按**g**（小写字母 g）来安装选定的包。

+   按**<ctrl-T>**激活菜单。

+   按**q**退出应用程序。

应用程序初始化完成后，按**u**（小写字母 u）更新包缓存。当本地软件包缓存下载并更新时，会显示**Downloading**状态信息。

缓存更新完成后，输入**/pianobar**。按**/**进入搜索模式，**pianobar**是搜索字符串。在输入搜索字符串时，aptitude 会逐步搜索该字符串。按**<enter>**键完成搜索短语，并选择找到的第一个包。

如果没有选择**pianobar**包，按**n**进入下一个匹配搜索字符串的包，或按**p**返回上一个匹配搜索字符串的包。

当选择了**pianobar**包时，按**+**来选择该包。如果错误地选择了一个包，按**-**来取消选择该包。

选择**pianobar**包后，按**g**计算依赖包并准备安装选定的包及其依赖项。

再次按**g**来安装**pianobar**包及其依赖项。此时，**aptitude**应用程序会启动**apt-get install**命令，开始安装**pianobar**包及其依赖项。

屏幕上充满了来自`apt-get install`命令的安装信息（有关`apt-get install`命令的更多信息，请参见*安装包*部分）。安装完成后，系统会提示**按回车继续**，按**<enter>**键继续。

最后，按**q**退出**aptitude**应用程序，并按**y**确认退出。

### 提示

虽然安装后不一定需要重启，但通常重启是个好主意。可以使用以下命令来重启：

`sudo reboot`

## 还有更多内容...

`aptitude` 软件包管理器是一款将 `apt-cache` 和 `apt-get` 命令的功能与一个复杂的终端用户界面结合起来的软件应用程序，仍然可以通过 `ssh` 等命令远程使用。

`aptitude` 应用程序提供的基于终端的用户界面简化了搜索和安装软件包的过程。通过简单的键盘操作，可以找到、安装和更新软件包。

`aptitude` 应用程序的终端用户界面适用于交互式使用。然而，命令行界面更适合用于脚本和自动化。

### 命令行界面

`aptitude` 应用程序还提供一组命令，可以作为 `apt-cache` 和 `apt-get` 命令功能的替代前端（有关使用 `apt-cache` 和 `apt-get` 命令查找和安装软件包的说明，请参阅名为 *查找软件包* 和 *安装软件包* 的教程）。

`aptitude update` 命令可用于更新软件包目录。它提供与 `apt-get update` 命令相同的功能：

```
pi@raspberrypi ~ $ sudo aptitude update

Hit http://archive.raspberrypi.org jessie InRelease
Hit http://mirrordirector.raspbian.org stretch InRelease
Hit http://archive.raspberrypi.org jessie/main Sources

...
```

`aptitude full-upgrade` 命令可用于完全升级已安装的软件包。它提供与 `apt-get dist-upgrade` 命令类似的功能：

```
pi@raspberrypi ~ $ sudo aptitude full-upgrade

The following NEW packages will be installed:
  raspberrypi-sys-mods{a} 
The following packages will be upgraded:
  bluej dhcpcd5 epiphany-browser epiphany-browser-data fonts-opensymbol 
  gir1.2-gdkpixbuf-2.0 gir1.2-gtk-3.0 gstreamer1.0-omx krb5-locales 
  libfreetype6 libfreetype6-dev libgdk-pixbuf2.0-0 libgdk-pixbuf2.0-common 

...
```

可以使用 `aptitude search` 命令来搜索软件包，如下所示：

```
pi@raspberrypi ~ $ sudo aptitude search pianobar

p   pianobar                        - console based player for Pandora radio    
p   pianobar-dbg                    - console based player for Pandora radio - d

pi@raspberrypi ~ $ 
```

`aptitude install` 命令用于安装软件包。它与 `apt-get install` 命令的功能相同：

```
pi@raspberrypi ~ $ sudo aptitude install pianobar

The following NEW packages will be installed:
  libao-common{a} libao4{a} libavfilter5{a} libpiano0{a} pianobar 
0 packages upgraded, 5 newly installed, 0 to remove and 0 not upgraded.
Need to get 246 kB of archives. After unpacking 546 kB will be used.
Do you want to continue? [Y/n/?] 

...
```

有关 `aptitude` 软件包管理器命令行使用的更多信息，请参阅 `aptitude` 手册页（`man aptitude`）。

## 另见

+   **aptitude – 包管理的高级界面** ([`manpages.debian.net/cgi-bin/man.cgi?query=aptitude`](http://manpages.debian.net/cgi-bin/man.cgi?query=aptitude)) Debian 的 `aptitude` 手册页包含关于该命令及其选项的详细信息。

+   **pianobar – 个性化在线广播客户端** ([`6xq.net/projects/pianobar/`](http://6xq.net/projects/pianobar/)) `pianobar` 应用程序是一个开源的基于控制台的 Pandora 客户端。

# 阅读内置文档（man）

本教程展示了如何使用 `man` 命令来显示内置文档。

Raspbian Linux 发行版内置了大量本地文档。这些文档主要有三种来源：man 手册页、`Info` 文档和 `/usr/share/docs` 目录。

本教程使用 `man` 命令来显示 `fortune` 命令的手册页文档。

本章的下一个教程展示了如何使用 `info` 命令在存储为 info 页的文档中进行浏览。

### 注意

有关安装 `fortune` 命令的说明，请参阅本章前面讨论的名为 *安装一个软件包* 的教程。

完成此操作后，您将能够使用`man`命令阅读 Raspbian Linux 发行版的内置手册文档。

## 准备工作

配料：

+   您需要一个*初始设置*或*基本网络设置*的树莓派，已经开机，并且以`pi`用户名登录（参考第一章中的相关操作，了解如何启动和登录，以及第二章中的远程登录方法）。

+   网络连接是可选的要求。

本应用程序不需要树莓派连接到互联网。

如果树莓派启用了远程访问，可以使用`ssh`或 PuTTY（参见第二章中的*远程访问*章节，*管理*）完成此操作。

## 如何操作...

执行以下步骤使用`man`阅读内置文档：

1.  直接或远程登录到树莓派。

1.  执行`man`命令以阅读`fortune`命令的内置文档：

    ```
    pi@raspberrypi:~$ man fortune
    ```

1.  显示`fortune`命令的 man 页面如下：

    ```
    FORTUNE(6)                   UNIX Reference Manual                  FORTUNE(6)

    NAME
           fortune - print a random, hopefully interesting, adage

    SYNOPSIS
           fortune [-acefilosuw] [-n length] [ -m pattern] [[n%] file/dir/all]

    DESCRIPTION
           When  fortune  is run with no arguments it prints out a random epigram.
           Epigrams are divided into several categories, where  each  category  is
           sub-divided  into those which are potentially offensive and those which
           are not.

       Options
           The options are as follows:

           -a     Choose from all lists of maxims, both offensive and  not.   (See
                  the -o option for more information on offensive fortunes.)

           -c     Show the cookie file from which the fortune came.

           -e     Consider  all  fortune files to be of equal size (see discussion
                  below on multiple files).

     Manual page fortune(6) line 1 (press h for help or q to quit)
    ```

1.  按下**<空格>**以翻页。按**h**以获取用于阅读和搜索的关键命令列表。按**q**退出阅读 man 页面。

## 工作原理...

登录树莓派后，使用`man`命令显示`fortune`命令的内置文档。

## 还有更多...

大多数命令行实用程序都有一个 man 页面。配置文件和软件库也有 man 页面。所有 man 页面都可以通过`man`命令访问。

欲了解更多信息，请使用`man`命令阅读其自身的手册页，即使用`man man`命令。

可以使用`apropos`命令搜索 man 页面库。`apropos`命令会在 man 页面数据库中搜索关键字并返回一组 man 页面。`man`命令也有一个适用的形式，即`man –k`。

下面是一个使用`apropos`命令搜索关键字“music”的简短终端会话：

```
pi@raspberrypi:~$ apropos music

pianobar (1)         - console pandora.com music player

pi@raspberrypi:~$ 
```

### 提示

可使用`aptitude`软件包管理应用程序安装`pianobar`控制台音乐播放器（参见名为*软件包管理*的操作指南）。

## 另请参阅

+   **apropos - 搜索** **手册页名称和描述** ([`manpages.debian.net/cgi-bin/man.cgi?query=apropos`](http://manpages.debian.net/cgi-bin/man.cgi?query=apropos)) Debian `apropos`命令的手册页面包含有关该命令及其选项的信息。

+   **man – 在线参考手册的接口** ([`manpages.debian.net/cgi-bin/man.cgi?query=man`](http://manpages.debian.net/cgi-bin/man.cgi?query=man)) Debian `man`命令的手册页面可以在线或本地找到。

# 阅读内置文档（info）

这篇教程展示了如何使用`info`命令读取 Info 页。

Info 页是另一种文档来源，已经随基础的 Raspbian Linux 发行版一起安装。`info`命令用于显示作为 Info 页存储的文档。

完成此教程后，你将能够使用`info`命令读取作为 Info 页存储的内置文档。

## 准备就绪

材料：

+   你需要一个已开机的 Raspberry Pi，并且需要进行*初始设置*或*基础网络*配置。你还需要以用户名为`pi`的用户登录（请参考第一章中的教程，了解如何启动和登录，并参考第二章中的教程，了解如何远程登录）。

+   网络连接是可选要求。

这个应用程序不需要 Raspberry Pi 连接互联网。

如果 Raspberry Pi 已启用远程访问，可以通过`ssh`或 PuTTY 远程完成此教程（请参考第二章中的*远程访问*教程，*管理*部分）。

## 如何操作……

执行以下步骤以使用`info`读取内置文档：

1.  直接或通过远程方式登录 Raspberry Pi。

1.  执行`info`命令，如下所示：

    ```
    pi@raspberrypi:~$ info
    ```

1.  Info 页的主屏幕显示如下：

    ```
    File: dir,      Node: Top       This is the top of the INFO tree

      This (the Directory node) gives a menu of major topics.
      Typing "q" exits, "?" lists all Info commands, "d" returns here,
      "h" gives a primer for first-timers,
      "mEmacs<Return>" visits the Emacs manual, etc.

      In Emacs, you can click mouse button 2 on a menu item or cross reference
      to select it.

    * Menu:

    Basics
    * Common options: (coreutils)Common options.
    * Coreutils: (coreutils).       Core GNU (file, text, shell) utilities.

    -----Info: (dir)Top, 174 lines --Top--------------------------------------------
    ```

1.  按空格键逐页浏览文档。这是从上到下阅读文件的最简单方式。

1.  使用箭头键将光标移至菜单项旁边的*****符号，然后按**<enter>**键跳转到文件中的该位置。

1.  按**h**（小写字母 h）了解有关`info`命令的更多信息。按**H**（大写字母 H）开始关于如何使用`info`的教程。

1.  按**q**退出。

## 它是如何工作的……

登录到 Raspberry Pi 后，`info`应用程序会启动。

## 还有更多……

`info`命令读取特别格式化的文件和 Info 页，并在文本浏览器中显示它们。Info 页结构清晰，具有菜单层级，可以通过概念浏览文档。它们看起来非常像网页。输入`info`命令而不带参数时，会显示所有 Info 页的根目录：`info`。

### Coreutils - Raspbian Linux 中最常用的工具

Info 页文档中的`Coreutils`部分涵盖了 Raspbian Linux 发行版中最常用的命令行工具，包括所有应用程序都应理解的常见选项：

```
File: coreutils.info,  Node: Common options,  Next: Output of entire files,  Pr\
ev: Introduction,  Up: Top

2 Common options
****************

Certain options are available in all of these programs.  Rather than
writing identical descriptions for each of the programs, they are
described here.  (In fact, every GNU program accepts (or should accept)
these options.)

   Normally options and operands can appear in any order, and programs
act as if all the options appear before any operands.  For example,
'sort -r passwd -t :' acts like 'sort -r -t : passwd', since ':' is an
option-argument of '-t'.  However, if the 'POSIXLY_CORRECT' environment
variable is set, options must appear before operands, unless otherwise
specified for a particular command.

   A few programs can usefully have trailing operands with leading '-'.
With such a program, options must precede operands even if
'POSIXLY_CORRECT' is not set, and this fact is noted in the program
description.  For example, the 'env' command's options must appear
before its operands, since in some cases the operands specify a command

--zz-Info: (coreutils.info.gz)Common options, 73 lines --Top--------------------
```

在 Info 页中，有关于文件权限、访问模式及其他在管理 Raspberry Pi 时有用的概念的文档。与 man 页不同，Info 页文档不仅提供总结，还包含选项列表，解释命令背后的概念及其预期用途。

具有讽刺意味的是，`info`命令的手册页面在 Raspbian Linux 发行版中默认并不包含。相反，可以使用`man info`命令来阅读关于`info`命令的更多内容。

### 搜索信息

你可以使用`info`命令的`–k`选项来搜索信息页面。当你记不住命令的名称时，可以尝试搜索(`-k`) `man` 和 `info`，并使用诸如“permission”、“access”或“download”等关键词。

当你在信息页面中搜索“permissions”时，你会得到以下内容：

```
golden@raspberrypi:~$ info -k permissions

"(coreutils)chmod invocation" -- access permissions, changing
"(coreutils)Setting Permissions" -- adding permissions
"(coreutils)chmod invocation" -- changing access permissions
"(coreutils)Copying Permissions" -- copying existing permissions
"(coreutils)Umask and Protection" -- giving away permissions
"(coreutils)Setting Permissions" -- group, permissions for
"(coreutils)Multiple Changes" -- multiple changes to permissions
"(coreutils)Setting Permissions" -- other permissions
"(coreutils)Setting Permissions" -- owner of file, permissions for
"(coreutils)install invocation" -- permissions of installed files, setting
"(coreutils)chmod invocation" -- permissions, changing access
"(coreutils)Copying Permissions" -- permissions, copying existing
"(coreutils)touch invocation" -- permissions, for changing file timestamps
"(coreutils)What information is listed" -- permissions, output by 'ls'
"(coreutils)chmod invocation" -- recursively changing access permissions
"(coreutils)Setting Permissions" -- removing permissions
"(coreutils)Setting Permissions" -- setting permissions
"(coreutils)Setting Permissions" -- subtracting permissions
"(coreutils)chmod invocation" -- symbolic links, permissions of
"(gnupg1)GPG Esoteric Options" -- preserve-permissions
"(wget)FTP Options" -- file permissions

golden@raspberrypi:~$ 
```

你已经拥有了关于 Linux、Debian 以及安装在树莓派上的 Raspbian Linux 操作系统的详细信息。`info`命令让你可以访问存储为信息页面的资料。

## 另见

+   **info – 阅读信息文档** ([`manpages.debian.net/cgi-bin/man.cgi?query=info`](http://manpages.debian.net/cgi-bin/man.cgi?query=info))。该链接指向 Debian 的`info`命令手册，描述了该命令及其选项。
