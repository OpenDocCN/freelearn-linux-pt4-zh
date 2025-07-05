# 第五章：故障排除用户、目录和文件

与之前讨论的主题不同，故障排除用户、目录和文件的过程可以看作是一个持续的过程，需要在服务器的生命周期中不断关注。它将成为日常事务，因此，我们将从用户管理的基本原则开始，目的是向你展示如何恢复默认的文件和文件夹权限，恢复丢失的文件，并带你走过许多相关主题，为你准备应对专业故障排除人员可能遇到的各种问题。

在本章中，我们将：

+   了解如何有效管理添加、删除、修改用户的过程，并使用`login.defs`实现系统范围的更改

+   了解如何使用`utmpdump`监控用户活动

+   了解如何重置 root 密码并启动基于 root 的日志记录，以实现更好的命令行安全审计

+   了解如何使用 Scalpel 恢复丢失的数据

+   了解如何恢复默认的权限和所有权

+   通过了解如何进行持续修复和检查碎片整理，进一步探索 XFS 文件系统

+   了解如何审计目录和文件

+   了解如何可视化目录和文件

# 用户

用户管理是与管理服务器相关的基础技能，在这方面，它无疑是解决任何系统问题时的一个里程碑。因此，考虑到这一点，我们将快速分析管理用户的过程，以消除任何困惑。

## 添加用户并强制密码更改

你可以使用以下命令添加新用户（并为他们创建主目录）：

```
# adduser <username>

```

你可以这样为新用户提供密码：

```
# passwd <username>

```

或者，如果你想强制重置密码，意味着用户必须重置其密码，那么以下命令就足够了：

```
# chage -d 0 <username>

```

此外，你可以通过输入以下命令为特定用户取消密码：

```
# usermod -p "" <username>

```

但是，如果你想授予新用户使用`sudo`的权限，那么请输入以下命令：

```
# gpasswd -a <username> wheel

```

最后，如果你想了解更多关于用户的信息，使用以下命令将显示他们当前的属性：

```
# id <username>

```

## 删除用户

删除用户账户的操作通常是直接的，但它可能涉及多个容易被忽视的步骤。因此，为了避免在大规模用户系统中出现任何未来的问题，在删除用户之前，应先按以下方式锁定账户：

```
# passwd -l <username>

```

然后，你需要使用`tar`备份用户的主目录，并通过输入以下命令来确认是否存在与该账户关联的活动进程：

```
# ps aux | grep -i <username>

```

完成这些后，你现在可以通过以下命令结束与该账户关联的任何活动进程：

```
# pkill -u <username>

```

或者，你可以像这样删除单独的进程 ID：

```
# kill -9 <pid>

```

通过使用`pkill`，您调用了`SIGTERM`命令，这将简化移除与该账户关联的任何活动进程的任务。因此，在这个阶段，您应该考虑移除任何文件、打印作业，并重新分配或删除与该账户关联的任何`cron`作业。

您可以通过键入以下命令来执行此操作：

```
# find / -user <username> -print

```

完成这些操作后，您可以安全地删除用户：

```
# userdel -r <username>

```

使用`-r`选项还会删除与该账户关联的主目录，但如果您希望删除用户、其主目录并移除任何`SELinux`映射，您应该使用：

```
# userdel -rZ <username>

```

然而，如果遇到任何困难，您可以始终以以下方式使用强制选项：

```
# userdel -rfZ <username>

```

最后，您需要考虑移除与该用户关联的任何 SSH 密钥。确保该账户没有启用`sudo`或`su`，然后逐一处理您的应用程序和服务（包括数据库、电子邮件、文件共享、htaccess、网页目录、CGI 文件等），同时为系统可能使用的任何公共账户重新分配新的设置。

## 修改用户

对于故障排除人员来说，用户管理的一个最有用的方面是能够修改现有的用户账户。可能有许多原因需要执行此任务，但最好的说明这种技能的方法是从修改以下文件中的默认`adduser`属性开始：

```
# nano /etc/default/useradd

```

从这里，您可以重新定义使用的 shell、主目录的默认位置以及是否设置默认的邮件队列。

例如，您可以使用此技术将主目录的默认位置从`/home`更改为`/home/<公司名>`。然而，如果您更倾向于手动操作（逐个案例处理），为了更改主目录的位置，您需要使用`usermod`命令结合`-d`选项（新目录的路径）和`-m`选项（移动当前主目录的内容），如下所示：

```
# usermod -m -d /path/to/new/home/directory <username>

```

在运行上述命令时，重要的是要意识到，如果该用户正在使用系统，控制台上将显示一个 PID，在进行任何修改之前，必须终止该进程。

最后，如果需要将现有用户转移到不同的组，可以通过调用`-g`选项来实现，如下所示：

```
# usermod -g <new_group_name> <username>

```

然而，完成这些操作后，就像删除用户一样，您必须手动更改任何`crontab`文件或任务的所有权，并通过对任何剩余（相关/现有）服务进行必要的修改来完成该过程。

## 了解 login.defs

在管理用户时，另一种替代方法或长期方法是考虑修改`/etc/login.defs`中找到的默认设置，这样您就可以更改删除命令的行为。

例如，假设您发现以下行被注释掉，如下所示：

```
#USERDEL_CMD     /usr/sbin/userdel_local
```

取消注释这一行，它将确保移除所有`at/cron/print`任务。此外，你还可以使用`login.defs`文件来确定分配给用户邮件目录、密码加密方式、密码过期周期、`userid`、`groupid`等的默认值。

# 使用 utmpdump 监控用户活动

跟踪用户活动是任何 Linux 管理员最基本的技能之一。在需要解决与用户管理相关的故障排除问题时，我们可以使用`utmpdump`。

用户历史通常存储在以下位置：

+   `/var/run/utmp`：这个二进制文件的目的是记录打开的会话。你可以使用`utmpdump /var/run/utmp`查看该文件的内容。

+   `/var/run/wtmp`：这个二进制文件的目的是记录连接历史。你可以使用`utmpdump /var/log/wtmp`查看该文件的内容。

+   `/var/log/btmp`。这个二进制文件的目的是记录失败的登录尝试。你可以使用`utmpdump /var/log/btmp`查看该文件的内容。

更进一步，你还可以通过输入以下命令来查看`/var/run/wtmp`中当前记录的会话历史：

```
# last

```

你可以通过输入以下命令查看`/var/run/btmp`中当前记录的会话历史：

```
# lastb

```

然而，鉴于对这些文件的简单查看对于我们的需求来说有些冗余，你可以使用以下命令查看这些文件的当前状态：

```
# stat /var/run/utmp
# stat /var/log/wtmp
# stat /var/log/btmp

```

这些命令的输出可能类似于以下内容：

```
Access: 2015-04-26 07:29:13.143818061 -0400
Modify: 2015-04-26 06:24:02.444728081 -0400
Change: 2015-04-26 06:24:02.444728081 -0400

```

由于二进制文件无法通过基本的阅读命令如`cat`、`less`和`more`进行查看，因此，除了简单依赖`last`、`who`、`lastb`等基本命令外，另一种方法是使用`utmpdump`命令，方式如下：

```
# utmpdump /path/to/binary

```

如前所述，如果你想读取`/var/run/utmp`，可以使用以下命令：

```
# utmpdump /var/run/utmp

```

而其余的文件可以通过以下方式访问：

```
# utmpdump /var/log/wtmp
# utmpdump /var/log/btmp

```

使用完这三个命令后，你会注意到输出格式是熟悉的，最明显的区别是`wtmp`的结果按逆序显示，而`utmp`和`btmp`则按时间顺序显示。

`utmpdump`的结果格式如下：

+   第一列显示会话标识符；值 7 通常与新登录事件关联，而值 8 则与登出事件关联。

+   第二列显示 PID。

+   第三列可以根据以下任一条件包含一个相对变量：

    +   `~~`，表示运行级别或系统重启变化

    +   `bw`，或称为启动等待进程

    +   一个数字或 TTY 值

    +   一个字符/数字，表示 PTY 值（伪终端）。

+   第四列有时可能为空，或者保留一个关联的用户名、运行级别或重启值。

+   第五列（如果该信息可用）将显示 TTY 或 PTY 值。

+   第六列将显示远程主机的身份。在大多数本地情况下，你最多只会看到一个运行级别信息，但对于远程访问，你会看到 IP 地址或名称。

+   第七列将显示远程主机的 IP 地址，或者如果是本地访问，则显示 0.0.0.0。

+   第八列（最后一列）将指示记录创建的时间和日期信息。

你还应该注意，如果没有进行 DNS 解析，第六列和第七列将显示相同的信息。

所以，考虑到前述信息，通过一点练习，并使用我们在前几章中发现的技巧，`utmpdump`可以用来执行广泛的查询，例如显示像这样的常规访问信息：

```
# utmpdump /var/log/wtmp

```

此外，你可以使用`grep`显示特定记录的详细信息。

例如，如果你想显示某个特定用户的`wtmp`记录，你可以输入：

```
# utmpdump /var/log/wtmp | grep <username>

```

进一步来说，你可以使用`grep`以以下方式识别来自特定 IP 地址的登录次数：

```
# utmpdump /var/log/wtmp | grep XXX.XXX.XXX.XXX

```

或者使用以下语法检查 root 访问系统的次数：

```
# utmpdump /var/log/wtmp | grep root

```

然后使用以下命令来监视失败登录尝试的次数：

```
# utmpdump /var/log/btmp

```

请记住，`btmp`的输出应该是最小化的，因为这个二进制文件将显示与使用错误密码或尝试使用未知用户名登录相关的各种问题。特别是当 tty1 被显示为正在使用时，后者尤其重要，因为这表示一个未知的人访问了你机器上的终端。从这个角度来看，注意到这样一个重要问题可能会激发你运行安全审计，检查访问权限和密钥，通过以下命令创建一个基本的基于文本的输出文件：

```
# utmpdump /var/log/btmp > btmp-YYYY-MM-DD.txt

```

# 重置 root 密码并增强日志记录

随着 CentOS 7 的发布，你可能会发现重置 root 密码的过程发生了变化。所以，如果你忘记了 root 密码，你需要按照这些重要步骤进行操作。

启动计算机，并在内核屏幕阶段按*E*键。在下一个屏幕上，滚动文本并查找以下行：

```
root=/dev/mapper/centos-root ro
```

现在，替换字母`ro`为以下内容：

```
rw init=/sysroot/bin/sh
```

它应该看起来像这样：

```
root=/dev/mapper/centos-root rw init=/sysroot/bin/sh
```

完成后，按*Control* + *X* 或 *Ctrl* + *X* 以使用 bash shell `/sysroot/bin/sh`进入单用户模式。

在单用户模式下，输入：

```
# chroot /sysroot

```

在井号（`#`）后，输入：

```
# passwd root

```

按照屏幕上的指示进行操作，并继续重置密码，但如果你确实需要更新`SELINUX`，在做任何操作之前，使用命令`touch /.autorelabel`。

当你准备好完成时，输入以下命令以按通常的方式访问机器：

```
# exit

```

现在，以通常的方式重新启动你的系统：

```
# reboot

```

做得好！你现在应该能够使用新的 root 密码完全访问系统。然而，如果你决定更新所有系统命令的日志记录，只需在你喜欢的文本编辑器中打开以下文件：

```
# nano /etc/bashrc

```

向下滚动到文件底部并添加以下行：

```
readonly PROMPT_COMMAND='history -a >(logger -t "$USER[$PWD] $SSH_CONNECTION")'
```

完成这一步后，你会发现所有基于 SSH 的命令行活动都被记录在`/var/log/messages`中，如下所示：

```
Jan 11 11:38:14 centurion1 journal: root[/root] 192.168.1.17 53421 192.168.1.183 22: last
Jan 11 11:38:26 centurion1 journal: root[/var/log] 192.168.1.17 53421 192.168.1.183 22: cd /var/log
Jan 11 11:38:32 centurion1 journal: root[/var/log] 192.168.1.17 53421 192.168.1.183 22: cat messages
Jan 11 11:38:49 centurion1 journal: root[/var/log] 192.168.1.17 53421 192.168.1.183 22: last

```

# 使用 Scalpel 恢复丢失或删除的文件

如果文件不小心从系统中删除，你可以使用一个名为 Scalpel 的小工具来恢复它。Scalpel 是 Foremost 的一个更快的替代工具，Foremost 最初由美国空军特种调查办公室和信息系统安全研究中心开发。如今，它是一个通常与数字取证调查和文件恢复相关的工具，你可以通过键入以下命令来安装它：

```
# yum install scalpel

```

你需要 EPEL 仓库来完成这个过程（这在前一章中讨论过），但是当你准备好时，只需更新以下配置文件，以确定你希望搜索哪些类型的文件：

```
# nano /etc/scalpel.conf

```

完成这一步后，你应该创建一个恢复目录，然后转到`/etc`目录以使用`scalpel.conf`，如图所示：

```
# cd /etc

```

你可以通过定制以下命令来扫描相关设备：

```
# scalpel /path/to/device -o /path/to/recovery/directory

```

上述命令的示例看起来像这样：

```
# scalpel /dev/sda1 -o /tmp/recovery-session1

```

Scalpel 将通过创建工作队列来开始，但请注意，整个操作需要一定的时间才能完成。简单来说，完成扫描所需的实际时间将取决于磁盘大小、删除文件的数量、机器的性能以及系统当前正在执行的其他活动。

你可以使用`ls`命令像这样查看结果：

```
# ls -la /path/to/recovery/directory

```

最后，在开始之前，你需要注意，每次运行 Scalpel 时都必须创建一个新的恢复目录（所以你可能需要考虑使用一个备用硬盘），因为结果将由一个单一的审计文件维护。

可以通过键入以下命令来查看这个特定文件：

```
# less /path/to/recovery/directory/audit.txt

```

记住，Scalpel 可以与各种文件系统格式或原始分区一起工作，从这个角度来看，它可以视为一个非常有用的故障排除工具。

你可以通过查看手册来了解更多关于 Scalpel 的内容，方法如下：

```
# man scalpel

```

# 恢复文件和目录权限

文件和目录权限非常重要，要查看特定目录中所有文件的当前状态，可以运行以下命令：

```
# ll

```

或者，你可以通过运行以下命令来定位特定目录：

```
# ll /path/to/directory

```

然而，在某个系统文件或文件夹的权限被误改的情况下，可以通过以下基于 RPM 的命令来修复这一灾难性情况：

```
# rpm --setugids PACKAGENAME
# rpm --setperms PACKAGENAME

```

另一方面，如果整个目录被错误地使用 `chown` 或 `chmod` 命令更新，那么以下命令会更加有用：

```
# for package in $(rpm -qa); do rpm --setugids $package; done
# for package in $(rpm -qa); do rpm --setperms $package; done

```

根据前面显示的命令，第一个命令将重置所有文件和文件夹的所有权值到默认状态，而第二个命令将重置相对的文件权限。因此，在运行这些命令后，你可能会看到以下消息：

```
chgrp: cannot access '/usr/share/man/zh_TW/man5x': No such file or directory
chown: cannot access '/usr/share/man/zh_TW/man6': No such file or directory
chgrp: cannot access '/usr/share/man/zh_TW/man6': No such file or directory
chown: cannot access '/usr/share/man/zh_TW/man6x': No such file or directory

```

不用担心！无论列出的是哪个文件或目录，这些通知都可以安全忽略。

# 使用和扩展 XFS 文件系统

XFS 最早于 1993 年由 Silicon Graphics 开发，其主要目的是不仅支持创建能够进行元数据日志记录的大型文件系统，还提供一种可以在挂载和活动状态下进行碎片整理和扩展的技术。作为故障排除者，这些信息对你可能没什么用，但你应该知道，最新版本的 CentOS 默认使用的文件系统就是 XFS。如果你没有对分区进行过大的自定义，那么你可能会发现 XFS 是你需要处理的文件系统。

你可以通过以下命令快速确认系统的结构：

```
# df -Th

```

上述命令（忽略磁盘大小和分区）可能会产生类似于以下输出的结果：

```
Filesystem              Type      Size  Used Avail Use% Mounted on
/dev/mapper/centos-root xfs        42G  1.5G   40G   4% /
devtmpfs                devtmpfs  913M     0  913M   0% /dev
tmpfs                   tmpfs     919M     0  919M   0% /dev/shm
tmpfs                   tmpfs     919M  8.4M  911M   1% /run
tmpfs                   tmpfs     919M     0  919M   0% /sys/fs/cgroup
/dev/sda1               xfs       494M  139M  356M  29% /boot
/dev/mapper/centos-home xfs        21G   33M   21G   1% /home

```

在标记为 `type` 的列下，如果出现 `xfs`，那就是我们要找的。如果发现服务器确实使用了 XFS 文件系统，那么可以使用以下命令安装 XFS 工具和实用程序文件 `xfsprogs.x86_64`：

```
# yum install xfsprogs

```

一般来说，你应该了解，如果服务器系统相对较小，XFS 可能会导致性能轻微下降。在这些情况下，ext4 在一些单线程和元数据密集型工作负载下通常会更快。此外，由于 XFS 不支持缩小，你应该知道，即使在未挂载时，该技术也不允许文件系统缩小。因此，当不需要大文件系统或大文件时，你可能会选择继续使用 ext4。

从更广泛的角度来看，你会感到宽慰，因为创建 XFS 所需的基本语法与其他文件系统相似：

```
# mkfs.xfs /dev/device

```

所以，没什么意外的，而且由于与其他文件系统的相似性，我假设你可以舒适地完成这整个过程。然而，在你开始之前，应该始终了解服务器的硬件配置，因为在开始操作之前，可能会有一些你需要注意的显著问题。

例如，假设服务器超过了 2 TB。所以，在完成初步的 `fdisk` 操作以构建文件系统布局后（挂载之前），你可能会决定对系统进行基准测试，因为每个优秀的故障排除者都知道 XFS 启用了写入屏障，以确保文件系统的完整性。

你可以通过输入以下命令来完成这个简单的操作：

```
# mount -o inode64 /dev/device /mount/point

```

默认情况下，写入屏障将有助于保护文件系统免受电源故障、重置和系统崩溃等问题的影响，但如果你的硬件具有良好的写入缓存，可能更明智的做法是禁用写入屏障，以减少对性能的影响。

在这方面，你可以通过以下方式挂载设备：

```
# mount -o nobarrier /dev/device /mount/point

```

完成后，你可以始终使用以下语法请求有关特定卷的更多信息：

```
# xfs_info /mount/point

```

正如我们所看到的，XFS 确实有许多优秀的功能和工具，但在排查服务器问题时，正是这些差异可能会成为问题的根源。

在这方面，正如我们现在看到的，XFS 应该与类似的基于 ext3 或 ext4 的系统有所不同对待。然而，如果你需要扩展文件系统，那么你会高兴地发现，XFS 配备了一种名为 `xfs_growfs` 的标准工具，可以通过以下方式使用：

```
# xfs_growfs -d /mount/point

```

假设你已经查看了 man 页，显然可以指出，你的语法会使用 `-d` 选项来将文件系统扩展到设备所支持的最大大小。

# 对 XFS 进行修复

XFS 是为了支持极大的文件系统而创建的。在高负载下，它表现得非常出色，并且能够扩展大文件，但也因此更容易受到损坏。正因为如此，我们现在考虑一组工具，帮助我们排查服务器问题并恢复文件系统。

这个被称为 `xfs_repair` 的工具，用于确认文件系统的一致性并修复发现的问题。这个过程不会恢复丢失的数据，但应该可以恢复相关设备上的文件系统。

`xfs_repair` 使用的基本语法如下：

```
# xfs_repair /mount/point

```

然而，为了避免任何错误信息，该过程需要你首先 `umount` 相关设备。整个过程如下所示：

```
# umount /mount/point
# xfs_repair /mount/point

```

结果输出将继续经过一系列阶段，并确认相关事件。完成后，只需按常规方式重新挂载设备以完成任务。然而，如果 `xfs_repair` 失败，请再次重复此过程，并对相关错误信息进行研究。

如果 `xfs_repair` 第三次未能修复一致性问题，依据错误信息，你可能需要考虑为服务器制定一个替代的救援计划，因为应该假设数据恢复只能通过备份来完成。

### 注意

话虽如此，你也可以考虑采取额外的步骤来恢复相关设备。

在当前阶段，你应该假设数据恢复只能通过备份进行，你的计划现在是基于仅恢复文件系统。然而，值得记住的是，你不应采取任何可能影响生产环境的操作。

通过备份和恢复文件系统上的文件，可能能够从磁盘恢复文件。要做到这一点，请以只读模式挂载文件系统，然后使用`xfsdump`进行备份。从此以后，你需要重新创建分区并使用`xfsrestore`恢复文件。有关详细信息，请查阅`man xfsdump`和`man xfsrestore`。

或者，如果日志恢复失败，可能可以通过以只读模式挂载文件系统，并使用`no recover`选项来恢复部分数据。这将避免运行日志恢复过程，但使用这种方法，文件系统可能不一致，并且无法保证所有数据都会恢复。

`xfs_repair`工具用于修复文件系统。它与文件系统的大小无关（无论大文件系统还是小文件系统都一样处理），但与其他修复工具不同，它不会在启动时运行，并且只会在挂载时启动日志记录，以确保文件系统的一致性。如果`xfs_repair`遇到损坏的日志文件，它将无法修复文件系统，因此如果发生这种情况，你需要清除相关日志，挂载然后卸载 XFS 文件系统，这可以通过添加`-L`选项来强制清零日志，如下所示：

```
# xfs_repair -L /mount/point

```

请记住，重置日志可能会导致文件系统处于不一致状态。这通常会导致数据丢失和/或数据损坏。因此，只应在仅打算恢复文件系统的情况下应用这些方法。记住，`xfs_repair`命令并非用于恢复该文件系统上的数据。

# 调查 XFS 上的碎片化

在文件系统运行缓慢的情况下，可能是碎片化影响了你的服务器。在这种情况下，如果你怀疑发生了或正在发生碎片化，可以在相关设备上运行以下命令：

```
# xfs_db -c frag -r /mount/point

```

使用此命令时，我们让`xfs_db`以只读模式（`-r`选项）打开文件系统，并传递一个命令（`-c`选项）来获取该设备的文件碎片数据（`frag`）。当我们使用`frag`命令时，它只会返回与文件数据相关的信息，而不会关注空闲空间的碎片化。因此，根据系统的具体情况，输出结果可能如下所示：

```
fragmentation factor 0.31%

```

在更严重的情况下，它可能会报告以下输出：

```
fragmentation factor 93.39%

```

通过将你的注意力引导到前面示例中的碎片因素（以百分比表示），你可能已经找到了至少一个需要故障排除的原因。修复这种情况只需要调用文件系统重组工具，也就是 `xfs_fsr`。我们只需要系统重组我们的分区或设备，以类似于 Microsoft Windows 桌面的方式优化磁盘使用。在这方面，使用 `xfs_fsr` 的最基本语法如下：

```
# xfs_fsr /path/to/device

```

而对于单个文件，你可以使用：

```
# xfs_fsr /path/to/file

```

然而，考虑到这些事件完成的时间可能相当长，这个命令的更简洁用法是指定一个要重组的文件系统列表（`-m`），一个以秒为单位计算的时间选项 `-t`，以及一个详细选项 `-v`，以清晰指示发生的情况，如下所示：

```
# xfs_fsr -m /etc/mtab -t 7200 -v

```

对应的输出将显示 inode 前后范围的数量。默认情况下，`xfs_fsr` 会在完成过程之前执行十次遍历，除非你决定使用选项 `-p` 来减少遍历次数，如下所示：

```
# xfs_fsr -m /etc/mtab -t 7200 -v -p 2

```

你应该知道，`xfs_fsr` 不应被用来碎片整理整个系统，因为这通常被认为是不必要的，它可能导致空闲空间碎片化，因此你可以分阶段完成此任务，并知道操作可以被干净地中断。这样将保持文件系统的一致性。如果你中断了过程（使用 *Ctrl* + *C*），`xfs_fsr` 会将碎片整理过程保存到以下位置：

```
# /var/tmp/.fsrlast_xfs

```

然而，在你深入了解之前，真正需要注意的问题是，应该小心谨慎地在实时系统上处理碎片问题，因为在高负载期间对设备或分区进行碎片整理将给你的服务器带来不必要的负担。因此，在这种情况下，最好的做法是在相关设备或分区未满载或在较轻工作负荷期间运行 `xfs_fsr`。

最后，在完成碎片整理过程后，你可以使用以下命令确认已完成的工作量：

```
# xfs_db -c frag -r /mount/point

```

因此，在完成这些简单操作后，或需要未来的（并可能是重复的）定时任务，你应该能立即注意到文件和文件夹移动及传输速度的改善。

# 审计目录和文件

处理故障排除的一个重要任务可以来自于对与文件读写操作相关的活动的理解。CentOS 7 提供了一个简单的实用工具。这个工具称为 `auditd`，它在启动过程中启动。事件会记录到一个关联的日志文件，位于 `/var/log/audit`，并且由于它在后台运行，你可以通过以下命令检查当前服务状态：

```
# systemctl status | grep audit

```

审计服务是可以自定义的，你可以通过访问以下文件，并使用你喜欢的文本编辑器，直接管理日志文件的大小、位置和相关属性：

```
# nano /etc/audit/auditd.conf

```

此外，如果你不希望丢失任何审计数据，你可以在无法执行审计时禁用机器。为此，打开配置文件 `auditd.conf`，并添加或修改以下几行：

```
max_log_file_action = keep_logs
space_left_action = email
action_mail_acct = root
admin_space_left_action = halt

```

这个操作很严重，在没有做好充分准备的情况下不建议贸然进行，但它会移除旋转日志文件的默认操作，并用发送电子邮件给 root 用户的指令来替代。

最后，如果你希望对每个进程利用审计服务标志，只需打开 `/etc/default/grub` 并将以下参数添加到内核行：

```
audit=1
```

记得使用以下命令重新生成 grub 并重启：

```
# grub2-mkconfig -o /boot/grub2/grub.cfg

```

这将确保在启动序列启动后，每个进程都设置一个可审计的标志，为了更简便，我们可以通过编辑以下文件来构建一套独特的规则：

```
# nano /etc/audit/rules.d/audit.rules

```

为了简化操作，最好的方法是找到服务器的 `stig.rules` 文件，路径为 `/usr/share/doc/audit-X.X.X/stig.rules`，并将其复制到 `/etc/audit/rules.d/audit.rules`。根据当前的包版本（以我为例），`stig.rules` 文件可以在 `/usr/share/doc/audit-2.3.3/stig.rules` 找到。因此，我运行了以下命令来创建默认规则集：

```
# cp /usr/share/doc/audit-2.3.3/stig.rules /etc/audit/rules.d/audit.rules

```

因此，在自定义规则并重启 `auditd` 服务后，你会发现可以使用以下语法发起查询：

```
# ausearch -f /path/to/directory/or/file
# ausearch -f /path/to/directory/or/file | less
# ausearch -f /path/to/directory/or/file -i | less

```

作为替代方案，你可以使用 `aureport` 以以下方式生成一系列审计：

要监控异常行为，可以使用：

```
# aureport --key --summary

```

要构建用户登录报告，可以使用：

```
# aureport -l -i -ts yesterday -te today

```

要审查访问违规行为，可以尝试：

```
# ausearch --key access --raw | aureport --file --summary

```

最后，要查看异常情况，可以使用：

```
# aureport --anomaly

```

当然，我们没有涵盖审计服务的每个方面，但前面的示例应该能帮助你入门。记住，所有示例都可以添加到 cron 作业中，如果你想了解更多内容，可以随时通过输入以下命令查看 `aureport` 手册：

```
# man ausearch
# man aureport

```

# 可视化目录和文件

良好的管理始于良好的家务处理，因此，维护关于服务器布局的详细记录通常被认为是任何 Linux 管理员的良好起点。这项任务不仅能让你随时了解系统整体所做的更改，还可以作为调试的有效方法。此外，因为你可能继承了这个系统，或者和其他管理员共享访问权限，因此考虑运行一次最新的更改清单可能是个好主意。

所有可以访问的目录、文件夹和文件都会以树状结构排列在一个特定的基于 Linux 的系统中。从根目录 (`/`) 开始，这个层次结构可能包含本地或远程文件、本地或远程文件系统，以及本地或远程块设备。

要查看此树，只需确保你已经安装了以下软件包：

```
# yum install tree

```

默认情况下，`tree` 命令将从当前位置开始索引，因此首先，只需将位置更改为启动目录，如下所示：

```
# cd /boot

```

现在，运行以下命令：

```
# tree

```

`tree` 命令在技术上被描述为 *递归目录列表命令*，它以树状格式显示服务器的内容。它高度可定制，因此如果你希望从当前位置指定一个特定目录，可以使用：

```
# tree /path/to/folder

```

你可能已经注意到，`tree` 命令默认不会显示隐藏文件。因此，为了查看所有文件（包括所有隐藏文件），请使用 `-a` 选项，如下所示：

```
# tree -a /path/to/folder

```

然而，如果你希望 `tree` 功能仅显示文件夹名称，你应该使用 `-d` 选项，如下所示：

```
# tree -d /path/to/folder

```

如果一切看起来有些平凡无奇，你可以通过 `-C` 选项为输出添加一些颜色，如下所示：

```
# tree -C /path/to/folder

```

最后，你可以将之前的选项组合起来，通过输入以下命令将输出打印到文本文件中：

```
# tree > /folder/name/filename.txt

```

例如，如果你想维护一个列出当前权限的文件清单，可以使用 `-p` 选项，如下所示：

```
# tree -p > /folder/name/filename.txt

```

或者，如果你更愿意显示嵌入了 HTML 代码的输出以便导出，可以尝试：

```
# tree -H /path/to/folder

```

因此，无论你是采用了新服务器，还是被访问并写入文件的用户数量所困扰，`tree` 功能提供了一个相对的解决方案，可以通过输入以下命令来保持对你的服务器或设备的可视化审计：

```
# tree -d /sys/devices

```

那为什么不把它与定时任务（cron job）结合使用呢？这样你就可以定期监控潜在问题的出现，甚至保持一个视觉记录，了解这些变化何时发生。从这个角度看，你可以断言 `tree` 软件包是一个非常有用的工具，要了解更多，你可以随时通过输入以下命令查看手册：

```
# man tree

```

# 总结

在本章中，我们讨论了与用户、目录和文件相关的多个主题，同时介绍了与 XFS 文件系统发布相关的一些主题。从强制更改密码到可视化目录结构，从恢复 root 密码到理解磁盘碎片整理的必要性，我们对 CentOS 7 故障排除的探索表明，解决系统基础问题所获得的知识与持续的人为问题密切相关。可以说，你永远无法预演一个灾难性场景，因为每个事件可能是独特的，可能仅适用于一个或多个系统，但正如我们所看到的，无论你是在监控用户、修改用户、恢复数据，还是在维护整个文件系统，通过遵循一些简单的步骤，许多与文件、目录和用户相关的问题都可以快速有效地解决；这也自然引导我们进入了共享资源故障排除的话题。

# 参考资料

+   Red Hat 客户门户: [`access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/`](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/)

+   Tree 项目主页: [`mama.indstate.edu/users/ice/tree/`](http://mama.indstate.edu/users/ice/tree/)

+   XFS 常见问题解答: [`xfs.org/index.php/XFS_FAQ`](http://xfs.org/index.php/XFS_FAQ)

+   XFS 用户指南: [`xfs.org/docs/xfsdocs-xml-dev/XFS_User_Guide//tmp/en-US/html/index.html`](http://xfs.org/docs/xfsdocs-xml-dev/XFS_User_Guide//tmp/en-US/html/index.html)

+   Red Hat XFS 指南: [`access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Storage_Administration_Guide/ch-xfs.html`](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Storage_Administration_Guide/ch-xfs.html)

+   XFS 维基页面: [`en.wikipedia.org/wiki/XFS`](http://en.wikipedia.org/wiki/XFS)
