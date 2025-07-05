# 第六章. Ubuntu 服务器的技巧和窍门

这是本书的最后一章。在本章中，我们将查看一些每个 Ubuntu 服务器管理员在管理基于 Ubuntu 的服务器时，简化工作所需的最佳技巧和窍门。

本章分为以下三个部分：

+   通用技巧 – 一套与 CLI、配置和脚本相关的通用技巧

+   故障排除技巧 – 有助于故障排除任务的实用技巧

+   实用工具和工具集 – 一套对系统管理员极有价值的复杂程序

# 通用技巧

本节将探讨一些有用的 Ubuntu 服务器技巧和窍门。这些是与**命令行界面**（**CLI**）、软件包管理和配置定制相关的通用技巧。

## Ubuntu 服务器命令行界面技巧和窍门

以下是一些有用的 CLI 命令：

1.  通过使用以下命令，获取安装软件包的原因，即检查给定软件包的反向依赖关系：

    ```
    apt-cache rdepends --installed [package_name]
    apt-cache rdepends --installed mysql-server

    ```

1.  执行以下命令列出所有用户及其最后登录时间：

    ```
    lastlog

    ```

1.  要更改服务器时区，你需要运行以下代码：

    ```
    sudo dpkg-reconfigure tzdata

    ```

    此外，要检查`timezone`是否已更改，你需要运行以下命令：

    ```
    less /etc/timezone

    ```

    另外，不要忘记重启`cron`，以便它能够识别`timezone`的更改，如下所示：

    ```
    sudo service cron restart

    ```

1.  如果你需要手动同步服务器时间，请使用以下命令：

    ```
    sudo ntpdate ntp.ubuntu.com

    ```

1.  如果你想安装时间同步服务，请执行以下命令：

    ```
    sudo apt-get install ntp

    ```

1.  要更新硬件时钟，请运行以下命令：

    ```
    hwclock --set --date '`date`'

    ```

1.  要列出服务器上所有已挂载和未挂载的分区，请使用以下命令：

    ```
    sudo fdisk -l | less

    ```

1.  如果你只想列出已挂载的分区及每个分区的空闲空间，请使用以下命令：

    ```
    df -h

    ```

1.  使用以下代码查看特定文件或文件夹的大小（在查看文件夹大小时非常重要）：

    ```
    du -h myfile
    du -sh myfolder

    ```

1.  随着文件的增大，你可以使用以下代码查看追加的数据（这对于监控和故障排除非常有用）：

    ```
    tail -f filename
    less +F filename

    ```

1.  通过使用以下命令获取系统环境变量：

    ```
    printenv

    ```

1.  要更改服务器主机名，请执行以下代码：

    ```
    sudo sh -c "echo new_host_name > /etc/hostname"
    sudo /etc/init.d/hostname restart

    ```

1.  要查看有关 Linux 发行版的信息，你需要执行以下命令：

    ```
    lsb_release -a

    ```

1.  要查看操作系统的安装日期，请运行以下命令：

    ```
    ls -ld /var/log/installer

    ```

## 如何防止服务器守护进程在安装过程中启动

在某些情况下，我们不希望在通过`apt-get`或`dpkg`安装更新时，服务器或守护进程作为后安装脚本的一部分启动。为达到这个目标，你需要在`/usr/sbin/`目录下创建一个名为`policy-rc.d`的文件，并将以下内容放入其中：

```
#!/bin/sh
exit 101

```

然后，使用以下命令赋予其*755*权限：

```
chmod 755 /usr/sbin/policy-rc.d

```

完成后，别忘了使用以下命令删除这个文件：

```
rm -f /usr/sbin/policy-rc.d

```

## 如何移动或复制目录

`cp`（复制）命令仅适用于文件。要复制目录，您需要使用`-r`递归标志，如下所示：

```
cp -r input_directry destination_directory

```

`mv`（移动）命令的工作方式相同。您需要使用`-r`递归标志来移动目录，如下所示：

```
mv -r input_directry destination_directory

```

## 系统资源限制

Ubuntu Server 的默认行为是不会对用户进程施加资源限制。这意味着用户可以自由使用机器上的所有可用内存，或启动一个无限循环的进程，从而使系统在几秒钟内变得无法使用。解决这个问题的最佳方法是通过以下命令编辑`/etc/security/limits.conf`文件来限制某些用户资源：

```
sudoedit /etc/security/limits.conf

```

文件的内容解释得很清楚。此外，对于该文件的配置，有一个很好的`man`页面：

```
man limits.conf

```

对于磁盘配额，您需要安装配额包，正如本书前面的章节所讨论的那样。

## 一遍又一遍地运行命令

有时我们需要一遍又一遍地运行相同的命令，要么监控一个操作的进度或复制一个文件，要么排查服务器的状态。虽然您可以一遍又一遍地运行相同的命令，但更好的方法是使用`watch`命令。`watch`程序将命令作为参数，然后每两秒运行该命令一次，将其输出显示在屏幕上。因此，例如，如果我们想监控`ubuntu.iso`的大小，可以输入`watch "ls -l ubuntu.iso"`。`watch`命令接受一些参数，例如`-n`参数，可以调整它在重新运行程序之前等待多少秒。

# 故障排除提示

本节中，您将找到与任务故障排除相关的高级技巧。这些对于大多数 Ubuntu Server 管理员来说是必备的。

## 在 Ubuntu Server 上自定义日志轮转

故障排除问题的最佳起点是阅读相关的服务日志。日志不会无限期保存，它们会存储在系统上，并遵循特定的策略。经过一段时间后，最旧的日志会被删除。因此，我们只保留一些最新的日志文件。这就叫做**日志轮转**；在 Ubuntu Server 上，它在两个层面上完成。通用策略配置在`/etc/logrotate.conf`文件中，而某些服务的特定策略配置在`/etc/logrotate.d/`目录下的特定文件中。

这个特定文件，名为`/etc/logrotate.d/*`，没有注释，而`/etc/logrotate.conf`通用文件注释丰富且非常清晰，语法与特定文件相同。`man`页面也有很好的文档，可以通过执行以下命令查看：

```
man logrotate

```

这些文件中默认存在的一些参数如下：

+   **日志轮转的周期性**，可以设置日志轮转的周期为每日、每周、每月等。

+   **每个日志文件的生命周期**，即日志文件在系统中保存的时间（直到被删除），以周为单位，并与 `rotate` 参数相关联。

+   **压缩日志文件的选项** 旨在通过使用 `compress` 参数节省空间。

以下是一些可以手动添加的其他参数：

+   `delaycompress` 参数将上一日志文件的压缩推迟到下一个轮换周期，从而允许用户轻松查看上周的日志，而无需解压缩。

+   `dateex` 指令将默认的回溯文件名称从默认行为（以数字命名文件：`logname.0`、`logname.1`、`logname.2` 等）更改为包含 *YYYYMMDD* 日期模式，而不是 0、1、2 等。

## 主要系统日志文件

系统日志文件是每项故障排除任务的支柱。仔细查看这些文件通常可以让系统管理员提前发现系统中出现的问题，并在问题扩大之前解决大多数问题。

通常，日志文件存储在 `/var/log` 目录下；对于已经运行一段时间的服务器，该目录中会有大量的旧日志文件，并且大多数文件都使用 `gzip` 压缩。

以下是一些值得注意的日志文件：

+   一般系统日志：`/var/log/syslog`

+   系统认证日志：`/var/log/auth.log`

+   系统邮件日志：`/var/log/mail.log`

+   一般日志信息：`/var/log/messages`

+   内核环形缓冲区信息，通常在系统启动时：`/var/log/dmesg`

## 检查打开的文件

在故障排除过程中，有时我们需要检查特定的文件（检查哪个进程正在运行并且正在打开它），或者检查特定的文件系统（哪些文件已打开并且是用于哪些操作）。这些信息由 `lsof` 命令提供。在 `lsof` 输出中，你会看到所有进程的列表，包括它们在文件系统上打开的文件、进程 ID、用户以及分配给它们的资源大小。

## 从 /proc 获取信息

排查 Ubuntu 服务器问题的一个良好起点是获取有关进程、硬件设备、内核子系统和其他 Linux 属性的信息。这可以通过 `/proc` 文件来实现。

从 `/proc` 目录中的文件获取信息可以通过使用 `cat` 命令简单完成。在 `/proc` 下，你会看到每个正在运行的进程都有一个独立的目录（每个进程的名称为其 PID），该目录包含有关该进程的信息。你还会找到包含计算机 CPU、软件版本、磁盘分区、内存使用情况等各种测量数据的 `/proc` 文件。

下面是一些示例，展示了你可以从 Ubuntu 系统的 `/proc` 目录中获取的信息：

+   要显示启动提示中使用的选项，请运行 `cat /proc/cmdline`

+   要显示与服务器处理器相关的信息，请运行`cat /proc/cpuinfo`

+   要显示现有的字符设备和块设备，请运行`cat /proc/devices`

+   要显示磁盘、分区及统计信息，请运行`cat /proc/diskstats`

+   要列出当前内核中的文件系统类型，请运行`cat /proc/filesystems`

+   要显示物理内存地址，请运行`cat /proc/iomem`

+   要显示虚拟内存地址，请运行`cat /proc/ioports`

+   要显示 1 分钟、5 分钟和 15 分钟的负载平均值，以及正在运行的进程/总进程和最高的**进程识别号**（**PID**），请运行`cat /proc/loadavg`

+   要显示可用的 RAM 和交换内存，请运行`cat /proc/meminfo`

+   要显示加载的模块、内存大小、已加载的实例、依赖关系加载状态和内核内存，请运行`cat /proc/modules`

+   要显示挂载的本地/远程文件系统信息，请运行`cat /proc/mounts`

+   要显示挂载的本地磁盘分区，请运行`cat /proc/partitions`

+   要显示 RAID 状态（使用 RAID 软件时），请运行`cat /proc/mdstat`

+   要显示系统启动以来的内核统计信息，请运行`cat /proc/stat`

+   要显示交换空间的信息，请运行`cat /proc/swaps`

+   要显示内核版本及相关编译器，请运行`cat /proc/version`

## 在 Ubuntu 服务器上恢复 root 密码

有时，服务器管理员会忘记 root 密码。在 Ubuntu 服务器上，可以通过在称为**单用户模式**的特定模式下启动系统来恢复 root 密码，这也叫做**运行级别 1**或**恢复模式**。

所以，第一步是通过使用 GRUB 菜单进入该模式。通常，在 Ubuntu 服务器上，你会自动找到此选项（它在 GRUB 条目名称后面的小括号内带有`Recovery Mode`标签）。如果不是这种情况，使用 GRUB 手动编辑启动时提议的菜单条目，在你想要的引导条目的末尾添加单词“single”。

内核应该像往常一样启动，但你会看到一个 root 提示符（`sh#`），而不是普通的提示符（`sh$`）。

到此为止，我们已经获得了对文件系统的 root 访问权限。我们可以最终更改密码。现在你拥有 root 权限，修改密码时不会要求你输入旧密码。因此，在运行以下命令时，你将直接被要求输入新密码及其确认：

```
# passwd

```

最后，你可以重新启动机器并再次获得 root 权限。

### 提示

默认情况下，恢复模式下的文件系统是只读的。需要重新挂载为读/写模式，才能更改 root 密码或进行任何需要写入磁盘的操作。要以读/写访问模式挂载，请输入以下命令：

```
mount -o remount,rw /

```

注意空格：如果忽略空格或添加多余空格，将会导致错误。

如果你的`/home`、`/boot`、`/tmp`或任何其他挂载点在单独的分区上，你可以使用以下命令挂载它们：

```
mount --all

```

然而，必须在第一次重新挂载步骤后立即执行此操作，以确保 `/etc/mtab` 可写。

# 实用工具和工具

在本节中，我们将看看一些工具和实用程序，帮助 Ubuntu 服务器管理员以最佳方式管理系统，以便系统的可用性达到最大化。

## NetHogs，网络监控工具

**NetHogs** 是一个网络监控工具，帮助你识别消耗带宽的程序。它类似于 `top` 命令，但用于网络。当你在排查网络问题时，例如 **是什么占用了互联网连接？**，这个工具非常有用。

启动 `nethogs` 需要使用 `sudo` 命令以 root 权限运行，因为它依赖 `libpcap` 来嗅探网络，像大多数网络监控工具一样。

安装 `nethogs` 很简单，只需运行以下命令：

```
sudo apt-get install nethogs

```

安装后，可以通过以下命令开始使用：

```
sudo nethogs network_interface_card (by default without using the NIC it will use eth0), for example: sudo nethogs eth1

```

程序运行后，你可以使用 *M* 键在以下两种模式之间切换：

+   实时使用情况

+   总使用量（不同模式基于大小单位，如 KB、MB 等）

## vnStat，网络监控工具

而 `nethogs`，我们在上一节中看到的工具，允许你通过在给定日期/时间收集统计信息来实时监控网络带宽使用情况，但如果你希望监控一小时/一天/一周/月的带宽使用情况，它就帮不上忙了。这个棘手的功能由另一个名为 `vnStat` 的网络监控工具提供。`vnstat` 守护进程是一组监控网络带宽使用的程序。

要在 Ubuntu 服务器上安装 `vnStat`，可以使用 `apt-get`，如下所示：

```
sudo apt-get install vnstat

```

`vnStat` 的配置通过编辑 `/etc/vnstat.conf` 文件完成；只需不要忘记重启 `itwith`，如下所示：

```
sudo /etc/init.d/vnstat restart

```

要启动 `vnStat` 守护进程，请执行 `vnstatd` 命令

在使用此工具之前，你需要等待一段时间以收集数据，然后运行 `vnstat` 命令。

这个工具带有许多有用的选项。你可以通过使用 `man` 页面来发现它们。以下是最有用的几个：

+   `vnstat -h`：用于查看每小时使用情况的摘要

+   `vnstat -d`：用于查看每日使用情况的摘要

+   `vnstat -w`：用于查看每周使用情况的摘要

+   `vnstat -m`：用于查看每月使用情况的摘要

+   `vnstat -t`：显示所有时间内流量前 10 的天数。

+   `vnstat -tr 10`：显示 `X` 秒内的使用情况（默认值为 5）

+   `vnstat -l`：显示实时带宽使用情况

`vnStat` 是一个非常有用的小工具，如果你想获得按小时/日/周/月计算的带宽使用概况，并且几乎不需要占用任何资源来实现这一点。

## 使用 multitail 监视多个文件

在许多情况下，系统管理员需要同时查看多个文件。`multitail`是一个非常方便的 Linux 工具，提供了这一功能。`multitail`工具让您能够在同一个 shell 中同时查看多个文件。此外，`multitail`还允许您运行多个命令并查看它们的输出，这简化了系统管理员的工作，特别是在一些高级故障排除任务中。

要安装此工具，您只需运行以下命令：

```
sudo apt-get install multitail

```

有很多方法可以使用这个命令。您可以参考`man`页面，那里有不同的示例，解释如何更好地自定义输出。

## 程序 cockpit——Ubuntu 服务器的远程管理器

`cockpit`命令是一个交互式的服务器管理界面。它易于使用且非常轻量级，允许 Ubuntu Server 管理员在一个用户友好的界面中执行许多任务，如存储管理、日志检查、服务管理（启动和停止）等。

要在 Ubuntu Server 上安装`cockpit`，您需要运行以下命令：

```
sudo add-apt-repository ppa:jpsutton/cockpit
sudo apt-get update
sudo apt-get install cockpit

```

要启动`cockpit`服务，请使用以下命令：

```
sudo systemctl start cockpit

```

如果您希望在重启服务器时自动启动`cockpit`服务，请使用以下命令：

```
sudo systemctl enable cockpit.socket

```

最后，要访问`cockpit`图形界面，请使用以下 URL：

```
https://<server_ip>:9090

```

## Webmin：著名的系统管理工具

**Webmin**是最著名的 Unix 管理工具之一。它是一个基于 Web 的界面，提供了大量功能，轻松管理服务器资源，包括：设置用户帐户；管理服务，如 Apache、FTP、DNS 等；共享文件管理；以及其他许多功能。总之，它是一个用户友好的界面。

安装`webmin`之前，首先需要一个完全正常运行的 LAMP 服务器。在本书的第三章《在 Ubuntu 上部署服务器》中，我们讨论了如何完成这一点。

之后，您需要从 APT 仓库安装`webmin`。首先，您需要将`webmin`官方仓库添加到服务器的源列表文件中。这可以通过使用`sudo vi /etc/apt/sources.list`命令编辑`/etc/apt/sources.list`文件来完成。然后，向`sources.list`文件中添加以下行：

```
deb http://download.webmin.com/download/repository sarge contrib
deb http://webmin.mirror.somersettechsolutions.co.uk/repository sarge contrib

```

完成此操作后，您需要下载**GNU 隐私保护**（**GPG**）密钥并通过以下命令将其添加到 APT 密钥中：

```
wget http://www.webmin.com/jcameron-key.asc
sudo apt-key add jcameron-key.asc

```

接下来，通过以下命令更新源列表：

```
sudo apt-get update

```

最后，通过以下命令安装`webmin`：

```
sudo apt-get install webmin

```

要访问`webmin`，您需要访问`https://<server_ip>:10000`。

## 使用 uvtool 程序并扩展 Cloud 镜像的使用

`uvtool` 程序由 Canonical 提供，供那些希望在云基础设施外部使用已经准备好的 Cloud 镜像来创建虚拟机的 Ubuntu 管理员使用，无需完整安装云服务器。这大大简化了使用 Cloud 镜像生成虚拟机的任务。

安装 `uvtool` 很简单，只需使用以下命令：

```
sudo apt-get -y install uvtool

```

### 注意

有关使用 `uvtool` 查找和同步 Cloud 镜像、创建和管理虚拟机以及该强大工具提供的其他功能的详细信息，可以参考官方的丰富文档，网址为 [`help.ubuntu.com/lts/serverguide/cloud-images-and-uvtool.html`](https://help.ubuntu.com/lts/serverguide/cloud-images-and-uvtool.html)。

# 总结

在本书的最后一章，我们回顾了 Ubuntu Server 管理员需要掌握的一些最佳技巧和方法，帮助他们以简便、高效、强大的方式管理 Ubuntu 服务器。到此为止，你已经掌握了必要的知识和技能，可以在信息通信技术（ICT）领域最优秀的操作系统之一上翱翔。
