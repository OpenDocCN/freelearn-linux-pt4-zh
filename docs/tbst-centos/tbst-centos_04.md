# 第四章。解决包管理和系统升级问题

Yellowdog Updater, Modified (Yum) 已经存在一段时间了。它易于使用，并在安装或升级 CentOS 软件包时减少了依赖管理的复杂性。更常被称为 Yum，本章将假设您已经精通其基本用法（包括安装软件包、更新软件包、删除软件包和搜索软件包），因为我们的目的是通过接近包管理和系统升级的主题来继续本书的总体前提。

在本章中，我们将：

+   学习如何使用 RPM 和 YUM 收集软件信息

+   学习如何使用 Yum 插件使故障排除过程更轻松

+   故障排除与包安装及更新相关的问题

+   发现如何扩展系统并安装额外的 Yum 仓库

+   学习如何使用 Yum 下载 RPM 软件包

+   学习如何恢复 RPM 数据库

+   讨论管理小系统升级的复杂性

# 收集软件信息

在开始解决 YUM 之前，我们将稍微偏离一下，把注意力转向收集必要的软件信息，以便更多地了解系统整体情况。

为此，我们将从运行以下命令开始：

```
# cat /etc/redhat-release; lscpu | grep -i arch; yum repolist all; ls -alsh /etc/yum.repos.d;

```

在这个阶段，我不会解释上面示例中显示的每个命令，但您会注意到输出内容冗长，并详细描述了 CentOS 的发布信息。显示的信息包括服务器的整体架构、域、时间和日期信息，以及 Yum 使用的仓库列表的范围、状态和权限。

到目前为止一切顺利，但是如果我需要了解服务器上已安装的基于 RPM 的软件包呢？与其浏览整个系列的配置文件（甚至是做一个有根据的猜测），为什么不简单地使用以下命令：

```
# rpm -qa | sort | less

```

如您所见，根据服务器上安装的内容，上述命令将输出大量（或者相对较少）基于 RPM 的软件包列表。如果这显得太详细，而您只是想快速了解安装历史，您可以输入以下命令来显示基于 RPM 的软件包总数：

```
# rpm -qa | wc -l

```

正如我们使用了`wc`命令，输出将被限制为表示所请求信息的数字值。对于您的确切需求来说，这可能过于简单或模糊，但它将为您提供关于所讨论的服务器当前使用情况的坚实基础（特别是如果此任务在特定时间段内重复执行）。

那么，为什么要止步于此？如果您感到更加好奇，那么您还可以通过输入以下命令来请求关于安装的 Yum 软件包的信息：

```
# yum list installed | sort | less

```

### 注意

在这个阶段，你可能会问，`yum list installed` 和之前提到的 `rpm –qa` 命令有什么区别？简单来说，前者是基于 Yum 的命令，它还会列出包依赖关系，因此你应该在输出中看到更多的细节。

再次说明，这个列表的内容可能会根据所涉及服务器的用途而有所不同，但对于那些喜欢良好管理的人来说，这个功能可以通过将输出打印到你选择的文件中来加以改进，命令如下：

```
# yum list installed | sort > /path/to/filename.txt

```

再次说明，在这个阶段并没有什么复杂的内容，但在我们完成这些初步步骤之前，需要意识到，我们刚刚揭开了一个以前未知的 CentOS 服务器的一些秘密。我们对系统的熟悉程度正在不断提高，随着我们活动的进行，你现在会对新环境感到更加舒适和自如。毕竟，当服务器宕机，其他人都在惊慌失措时，这里获得的信心将有助于你最终的成功。

# 使用 Yum 插件

Yum 是最广泛使用的包管理工具之一，但许多管理员并不知道它自带一个插件系统，可以用来扩展其功能。可以说，许多插件是默认安装的，但由于假设你对当前系统一无所知（这对于任何故障排除人员来说都是常见的情况），我们将从安装 `yum-skip-broken` 包开始。

那么，我们从输入以下命令开始：

```
# yum install yum-skip-broken

```

完成了这个步骤（并确认该套包现在已被系统识别），我们可以使用 `--skip-broken` 插件来处理任何想要更新或升级某个包，但由于报告依赖关系损坏而被拒绝的情况。

为此，可以使用以下命令的组合：

```
# yum update --skip-broken && yum upgrade --skip-broken

```

当然，你可以简化上述命令，以便符合自己的需求（例如将命令分成两行），但需要注意的是，这些情况下发生错误的原因通常与混合（不兼容的）第三方仓库有关。在这方面，为了实现长期解决方案，你需要减少使用的第三方仓库的数量。然而，在尝试这一操作之前，你应该回顾 Yum 过去的事务，以了解这会如何影响系统。

这可以通过输入以下命令来实现：

```
# yum history

```

上述命令现在将生成一个包含 Yum 所有历史操作的列表，从而提供一个非常有用的详细信息层级，当你尝试调试一个故障服务、修正包依赖或协助减少对第三方仓库的依赖时，这个信息会非常有用。此外，在这个命令生成的列表中，你还会注意到每个事务都有一个唯一的标识符。

唯一标识符可以用来获取有关特定事件的更多信息，输入以下命令：

```
# yum history info <unique_identifier>

```

或者，在不需要如此详细信息的情况下，你可以通过输入以下命令始终获得所有最近事件的摘要：

```
# yum history summary

```

话虽如此，既然我们谈论到了 Yum 插件，你还应该知道，`changelog`信息并不是直接可以通过 Yum 包管理器获得的。同样，这类信息可能很有用，但为了进一步了解，你需要通过输入以下命令安装`changelog`插件：

```
# yum install yum-plugin-changelog

```

完成此操作后，你就能够像这样查询 Yum 的`changelog`：

```
# yum changelog all <package_name>

```

由于此输出可能会让人感到有些不知所措，因此知道你可以通过简单地将`all`替换为表示要显示记录数量的数字值来限制显示的信息，这一点非常有帮助。所以，如果你想查看 Postfix 的`5`个最近更新，你可以输入：

```
# yum changelog 5 postfix

```

此外，安装了这个插件后，你可以通过输入以下命令来查看任何包安装之前的相关`changelog`信息：

```
# yum update --changelog

```

这是一个小巧但有趣的功能，可以通过输入以下命令来扩展显示所有最近的`changelog`信息：

```
# yum changelog all recent

```

或者，你可以通过输入以下命令查看所有过时的包：

```
# yum changelog all obsoletes

```

最后，在我们结束关于使用 Yum 插件的讨论之前，可能会有一种情况需要使用`yum-utils`，如果它当前没有在系统中安装，你可以通过输入以下命令来安装：

```
# yum install yum-utils

```

这一补充使得包管理变得更加轻松，考虑到需要解决的一系列问题，包括删除孤立包或重复包，以及包清理操作的执行。

例如，要删除孤立包，可以输入：

```
# package-cleanup --orphans

```

要删除重复项，可以使用：

```
# package-cleanup --dupes

```

为了方便删除旧内核，你可以输入：

```
# package-cleanup --oldkernels

```

参考上述示例，如果你的系统维护着多个旧的内核，可以通过运行命令`rpm -q kernel`来查询它们。正如你所看到的，这个简单的操作可以提供你所需的信息，从而决定是否可以释放磁盘空间。如果看到旧的内核，你可以按照常规方式将其删除。然而，我并不建议你非得这么做，我揭示这个功能的目的是证明`yum-utils`有许多有趣的功能，值得探索，因为它是 Yum 中一个被低估的方面，在故障排除包管理时能发挥重要作用。

# 修复 Yum 操作

现在，需要意识到的是，在许多情况下，Yum 出错的普遍原因可能与网络环境、磁盘空间、混合仓库或系统的 DNS 设置有关。然而，也有一些情况，常见的操作错误需要通过刷新 Yum 本身来解决，可以使用以下命令中的一个或多个：

要清除旧的包信息，您应使用：

```
# yum clean headers

```

要清除所有缓存的包，请使用以下命令：

```
# yum clean packages

```

要清除所有缓存的基于 XML 的数据，请使用以下命令：

```
# yum clean metadata

```

然而，如果您希望完全刷新 Yum 缓存（包括所有头信息、元数据、下载的包等），您可以随时使用以下命令完成此过程：

```
# yum clean all

```

# 安装额外的 Yum 仓库

安装额外的仓库不一定被视为故障排除任务，但它确实有助于缓解许多与包依赖性相关的问题，并尽量保持您的系统在 Dev/Ops 环境中具有相关性。

在接下来的文本中，我已包含一些最流行的仓库的安装说明。然而，您应该意识到，这些仓库的位置在 CentOS 的不同版本中会有所不同，并且这些链接会随着时间的推移而更新。更多信息可以在本章末尾找到，供日后参考。

## EPEL

**企业 Linux 附加软件包**（**EPEL**）仓库提供了官方 CentOS Linux 仓库中未包含的有用软件包，其中一些将在本书的后续章节中介绍。它也是许多其他第三方仓库的依赖项。

对于 CentOS 7，您应按照此过程安装 EPEL 仓库：

```
# cd /root
# wget https://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-5.noarch.rpm
# yum install epel-release-7-5.noarch.rpm

```

完成相关步骤后，只需确认是否继续安装，然后输入以下命令以确保其已启用：

```
# yum repolist all

```

EPEL 通常被认为是一个基础仓库，许多其他仓库也常常依赖它。鉴于我们希望保持系统的更新，我已包含了安装 Remi 和 IUS 仓库的程序。然而，带着强烈的警告，我建议只使用其中一个，以避免在同一系统上使用它们可能引发的冲突。

## Remi

Remi 仓库依赖于 EPEL 仓库，并向 CentOS 核心 Linux 仓库提供软件的新版本；因此，安装了此仓库后，您可以预期在下次运行 `yum update` 时，CentOS 系统会进行多次更新。

对于 CentOS 7，您应按照此过程安装 Remi 仓库：

```
# cd /root
# wget http://rpms.famillecollet.com/enterprise/remi-release-7.rpm
# rpm -Uvh remi-release-7*.rpm

```

默认情况下，Remi 仓库是禁用的；要启用它，我们需要更新配置文件并将其标记为活动状态。

要执行此操作，请在您喜欢的文本编辑器中打开以下文件：

```
# nano /etc/yum.repos.d/remi.repo

```

现在，改变：

```
enabled = 0
```

阅读：

```
enabled = 1
```

此外，如果您想使用 PHP 5.5 库，只需在同一文件 `remi.repo` 中取消注释 `[remi-php55]` 引用。完成后，保存并关闭文件，然后输入以下命令以确保其已启用：

```
# yum repolist all

```

## IUS 仓库

作为 Remi 的替代方案，IUS 仓库也向核心 CentOS Linux 仓库提供更新的版本，但开发人员强调，IUS 通常使用不同的包名，以避免软件版本更新时可能出现的冲突。这个简单的包名方法在 CentOS 社区中得到了广泛支持，因为这种控制级别在关键任务环境中非常有用。IUS 仓库依赖于 EPEL 仓库，但再次强调，我建议不要将这个仓库与其他来源混合使用。

对于 CentOS 7，你应该按照以下步骤安装 IUS 仓库：

```
# cd /root
# wget http://dl.iuscommunity.org/pub/ius/stable/CentOS/7/x86_64/ius-release-1.0-13.ius.centos7.noarch.rpm
# rpm -Uvh ius-release*.rpm

```

此外，鉴于 IUS 仓库的工作方式独特，你应该知道有一个叫做 `yum-plugin-replace` 的包，它用于帮助从默认包升级到 IUS `packageXY` 风格的包。

再次，我在本章结尾提供了一个关于如何使用这个工具的链接（预计这些指令可能会随时间改变），但目前，你可以通过输入以下命令开始：

```
# yum install yum-plugin-replace

```

总体而言，无论你是使用 Remi 还是 IUS，请记住，黄金法则是不混合使用它们，但无论你选择哪个仓库，预期它们都会对你有帮助。

# 使用 Yum 下载 RPM 包

如果有需要下载一个包但不安装它的情况，可以通过 Yum 来实现。然而，在开始这个过程之前，你需要确保你的系统已经安装了以下工具：

```
# yum install yum-plugin-downloadonly

```

在大多数情况下，它可能已经被安装（因为它是之前提到的 `yum-utils` 包的一部分），但完成这个任务后（或者确认它已经存在），你可以通过自定义以下命令，将所需的包下载到你选择的目录：

```
# yum install <package_name> --downloadonly --downloaddir=/path/to/folder

```

例如，为了理解之前的命令，你可以通过输入以下命令，将 Samba 及其所有依赖项下载到用户目录中：

```
# yum install samba --downloadonly --downloaddir=/home/username

```

或者，你也可以调用以下变体，尽管这个版本的命令假设下载的文件会存储在你当前的目录：

```
# yumdownloader <package_name>

```

完成这一步后，接下来的步骤是提取使用 `yumdownloader` 下载的包内容，以便通过以下命令访问相关的 RPM：

```
# rpm2cpio <package_name> | cpio -idmv

```

`cpio` 命令通常用于备份，其功能是便于将文件进出 `cpio` 压缩包。它的工作方式类似于 `tar`，但与 `tar` 不同，`cpio` 可以与 `find` 命令结合使用。

例如，要进行备份，你可以通过以下方式访问相关目录：

```
# cd /path/to/directory

```

列出目录内容，确保所有内容都在：

```
# ls -altr

```

现在，以以下方式运行备份目录的命令：

```
# find . -print | cpio -ocv > /path/to/backup-name.cpio

```

所以，通过几步简单的操作，我们已经学会了如何通过 Yum 下载一个软件包，并使用`cpio`命令提取相关的包数据。这个过程在排查特定服务或应用程序的依赖问题时可能会非常有用。这可能是你第一次接触`cpio`命令，也许你不会感到惊讶，因为我们仅仅触及了它的一小部分功能。因此，我鼓励你深入学习它，因为你会发现它在未来某些时候会非常有用。

关于`cpio`命令的更多信息可以在本章末尾找到，但作为参考，你可以通过输入以下命令开始你的探索：

```
$ man cpio

```

# 诊断损坏的 RPM 数据库

RPM 是一个包管理工具，它将软件包的信息存储在位于`/var/lib/rpm`的数据库中，但有时会观察到该数据库可能会失败。如果发生这种情况，`rpm`命令将无法使用，在这种情况下，系统可能会开始出现与任何基于 Yum 或 RPM 的进程相关的故障迹象。

例如，在进行典型的 Yum 更新过程时，你可能会看到以下消息：

```
"error: cannot open Packages database in /var/lib/rpm"

```

在此，我不得不强调良好备份策略的重要性，但作为故障排查人员，这可能超出你的控制范围。因此，在这种情况下，可以通过完成以下步骤来诊断恢复 RPM 数据库的基本过程：

```
# cd /var/lib/rpm
# rm -rf __db*
# rpm -v --rebuilddb

```

完成上述步骤后，你应该能够运行各种健康检查，以确认是否没有段错误，可以通过以下命令之一或多个进行检查：

```
# db_verify Packages
# db_stat -CA
# rpm -qa
# rpm -Va
# yum update

```

例如，运行命令`db_verify Packages`后，你应该看到以下类型的输出：

```
BDB5105 Verification of Packages succeeded.

```

希望这个过程能够解决问题。然而，需要认识到，这个过程并不是绝对的，它可能会失败。换个角度来看，为了避免这种情况，定期备份`/var/lib/rpm`应视为所有 CentOS/RHEL 系统的常规操作。如果包验证过程失败，你需要考虑从最近的备份中进行完全恢复。

# 小版本升级

对于 CentOS 7 用户，完成小版本升级的简便方法只是使用 Yum，但和往常一样，在继续操作之前，你应该进行完整备份。

你应该备份的文件类型因系统而异，但通常包括配置文件、重要的系统文件、用户数据、数据库、网站版本控制和应用程序文件。此外，如果你使用的是专有软件，在进行更新之前，你应与原开发者确认任何升级的可行性。

因此，在考虑了所有这些措施之后，如果可能的话，我建议您在操作前备份整个系统，您可以放心，备份中会有所有内容。

要开始进行小幅升级，您可以通过输入以下命令查看当前 CentOS 版本信息：

```
# cat /etc/redhat-release

```

然后，您可以通过以下命令查看 Linux 信息：

```
# uname -mrs

```

现在，在您调用 Yum 获取更新包列表之前，通常最好先输入以下命令：

```
# yum clean all

```

然后，按照以下说明清除 Yum：

```
# yum check-update

```

现在，您将看到所选系统的更新列表。这些待更新的内容将以常见的格式显示，详细列出包名和版本信息。

如果您希望更新系统，请使用以下命令：

```
# yum update

```

任何更新所需的时间可能会有所不同，因此您可能需要耐心等待。然而，在成功完成此步骤后，您可能想考虑重新启动系统。如果更新需要在启动阶段对内核进行任何更改，重启系统尤为有利。

要执行此操作，请输入：

```
# reboot

```

最后，在重新启动完成后，您可以使用以下命令验证系统更新：

```
# uname -a

```

现在，运行以下一个或多个命令以确保所有服务和应用程序都正常运行：

```
# tail -f /var/log/messages
# netstat -tulpn
# ps aux | less

```

# 摘要

在本章中，我们迅速了解了包管理故障排除的细节。虽然还有很多内容可以进一步学习，但作为系统管理员，您不仅发现了一系列可以用来解决 Yum 相关常见问题的工具，还讨论了通过各种插件扩展 Yum、安装第三方仓库、启用 Yum 下载 RPM 包以及恢复 RPM 数据库的能力。

如您所见，在我们继续讨论用户、目录和文件之前，通过一些 lateral thinking（横向思维）和一点练习，您现在应该能够排除大量问题，并将几乎所有的包管理问题抛诸脑后。

# 参考资料

+   CentOS 升级工具：[`wiki.centos.org/TipsAndTricks/CentOSUpgradeTool`](http://wiki.centos.org/TipsAndTricks/CentOSUpgradeTool)

+   如何从 Enterprise Linux 6 升级到 Red Hat Enterprise Linux 7？：[`access.redhat.com/solutions/637583`](https://access.redhat.com/solutions/637583)

+   CPIO 的维基百科页面：[`en.wikipedia.org/wiki/Cpio`](http://en.wikipedia.org/wiki/Cpio)

+   `cpio` 命令：[`www.gnu.org/software/cpio/manual/cpio.html`](http://www.gnu.org/software/cpio/manual/cpio.html)

+   EPEL/epel7beta-faq：[`fedoraproject.org/wiki/EPEL/epel7beta-faq`](https://fedoraproject.org/wiki/EPEL/epel7beta-faq)

+   EPEL/epel7：[`fedoraproject.org/wiki/EPEL/epel7`](https://fedoraproject.org/wiki/EPEL/epel7)

+   /pub/epel/7/x86_64/e 的索引: [`dl.fedoraproject.org/pub/epel/7/x86_64/e/`](http://dl.fedoraproject.org/pub/epel/7/x86_64/e/)

+   Les RPM de Remi - 博客: [`blog.famillecollet.com/pages/Config-en`](http://blog.famillecollet.com/pages/Config-en)

+   Remi 索引: [`rpms.famillecollet.com/enterprise/`](http://rpms.famillecollet.com/enterprise/)

+   IUS 社区项目: [`iuscommunity.org/pages/IUSClientUsageGuide.html`](https://iuscommunity.org/pages/IUSClientUsageGuide.html)

+   /pub/ius/stable/CentOS 的索引: [`dl.iuscommunity.org/pub/ius/stable/CentOS/`](http://dl.iuscommunity.org/pub/ius/stable/CentOS/)

+   IUS 包替换指南: [`iuscommunity.org/pages/IUSClientUsageGuide.html`](https://iuscommunity.org/pages/IUSClientUsageGuide.html)

+   ATrpms 仓库: [`atrpms.net/about/`](http://atrpms.net/about/)

+   Nux 仓库: [`li.nux.ro/repos.html`](http://li.nux.ro/repos.html)

+   RPM 首页: [`rpm.org`](http://rpm.org)

+   RPM 恢复诊断: [`www.rpm.org/wiki/Docs/RpmRecovery`](http://www.rpm.org/wiki/Docs/RpmRecovery)
