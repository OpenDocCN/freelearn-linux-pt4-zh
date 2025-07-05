# 第十三章：定时器的时间

在深入了解守护进程和 SSH 隧道之后，处理`cronjob`可能看起来像是一项简单的任务，但让我们稍作思考：我们的日常生活中有多少次需要安排一个任务在特定的时间范围内执行，也许是在深夜或者我们度假时？有多少次我们需要一个任务每天在准确的时间执行，每一天都不例外？我们真的想熬夜，或者放弃度假吗？更重要的是，我们能确保自己每天都能按时执行任务吗？简单来说，我们不能。因此，一个安排任务并在需要时执行的方法看似简单，但正是它让系统更易管理，并且为我们节省了大量麻烦。

我们有许多工具和分叉项目可用，但我们将重点关注其中的几个。最著名的老工具是**at**和**cron**。它们做的事情差不多，但实现方式不同，目标也有所不同。

# 一次性执行

有时候，我们需要在特定时间执行一个任务，并且不需要重复操作，仅仅是一次性执行。在这种情况下，我们可以使用一个简单的工具，叫做`at`，它有一个伴侣工具叫做 batch。它的作用是什么？它简单地从输入或文件中读取要执行的内容和时间，然后用`/bin/sh`调用我们想要执行的命令。不过有一个小小的区别：batch 会在系统负载降到 1.5 以下，或`atd`运行时指定的任何负载水平时执行任务，而不是在特定的时间执行。

所以，我们介绍了`atd`；这是什么？这是一个守护进程，用来执行由`at`工具定义并放入其队列的单次任务，因此它是一个通常以专用守护进程用户身份运行的守护进程：

```
root:# ps -fC atd UID PID PPID C STIME TTY TIME CMD daemon 722 1 0 Apr25 ? 00:00:00 /usr/sbin/atd -f

```

所以，这是一个作为系统服务启动的守护进程，但我们有一些其他选项；我们可以传递它来修改它如何处理计划任务。让我们看看有哪些可以使用的选项：

+   `-l`：定义超过该负载因子的调度任务将不会被执行。如果没有设置限制，默认为 1.5；但是，对于拥有 x 个 CPU 的系统，合适的限制值可能会高于 x-1。

+   `-b`：此选项设置两个连续任务之间的最小间隔（单位为秒）。默认为`60`。

+   `-d`：这里遇到问题了吗？此选项将启用调试功能，将错误信息从`syslog`重定向到`stderr`，通常是终端。此选项与`-f`选项一起使用。

+   `-f`：适合调试，此选项强制`atd`在前台运行。

+   `-s`：强制`at`和`batch`队列只执行一次。此选项用于与旧版`at`的向后兼容；它的运行方式类似我们以前运行的`atrun`命令。

`atd`运行过程涉及的文件很少：

+   `/var/spool/cron/atjobs`: 这是 `at` 创建的任务存储目录，`atd` 将从中读取队列。它必须由进程的同一所有者（在我们这个例子中是守护进程）拥有，并具有严格的 700 访问权限。

+   `/var/spool/cron/atspool`: 这是临时保存任务输出的目录。它由一个守护进程用户拥有，访问模式为 700。

+   `/etc/at.allow`, `/etc/at.deny`: 在这个文件中，我们可以设置哪些账户可以提交任务给 `atd`，哪些账户被禁止。

使用 `atd` 时有一个限制：如果它的排队目录通过 NFS 挂载，则无论你使用何种挂载选项，它都会完全无法工作。

我们看到了一些有趣的文件：

```
/etc/at.allow /etc/at.deny

```

可以提交任务给 `atd` 守护进程的管理员可以使用非常简单的语法：这是一个简单的账户名称列表，每行一个，不带空格。文件解析有优先顺序：`/etc/at.allow` 是第一个被读取的。如果其中找到任何账户名，这些账户将是唯一允许提交任务的账户。

如果 `/etc/at.allow` 文件不存在，那么会解析 `/etc/at.deny`，其中找到的每个账户名都将被禁止向 `atd` 提交任务。如果文件存在但为空，则表示所有账户都可以提交任务给 `atd`。

如果 `/etc/at.allow` 或 `/etc/at.deny` 都不存在，这意味着只有超级用户可以提交任务给 `atd`。

在系统中，我们使用沙箱。我们有 `/etc/at.deny` 和其中的内容如下：

```
root:# cat /etc/at.deny alias backup bin daemon ftp games gnats guest irc lp mail man nobody operator proxy qmaild qmaill qmailp qmailq qmailr qmails sync sys www-data

```

所以对于我们刚才看到的规则，如果 `/etc/at.allow` 文件不存在，所有列在 `/etc/at.deny` 中的账户都被禁止提交任务给 `atd`。这也很合理，因为我们可以看到这些账户与服务相关，或者是没有身份的用户，它们不应该有提交任务的需求。

这就是关于 `at` 服务部分的所有内容；接下来我们来看一下可以用来安排任务的工具。在客户端，我们可以使用以下工具来提交任务给 `atd`。

`at` 和 `batch` 命令从标准输入或指定文件读取，任务将在稍后的时间执行，使用 `/bin/sh`。

+   `at`: 这是我们使用的主要工具，它的功能是提交将在特定时间执行的任务。时间指定非常智能且灵活。

+   `HH:MM`: 这指定了当天运行 `at` 调用命令的时间。如果时间已经过去，该命令将计划在第二天同一时刻执行。

+   `midnight`: 这个任务意味着在午夜执行：

```
root:# at midnight
warning: commands will be executed using /bin/sh
at> ls
at> <EOT>
job 6 at Fri Apr 28 00:00:00 2017

```

提交任务时，只需在新的一行上按 *Ctrl*+*D*：

+   `noon`: 这个任务意味着在中午执行

+   `teatime`: 这个任务将在下午 4 点执行

+   `AM-PM`: 我们可以添加后缀 AM 或 PM，将任务安排在早晨或下午的特定时间执行：

```
root:# echo "systemctl restart ssh" | at 04:43 AM warning: commands will be executed using /bin/sh job 7 at Thu Apr 27 04:43:00 2017 root:# echo "systemctl restart ssh" | at 04:43 PM warning: commands will be executed using /bin/sh job 8 at Thu Apr 27 16:43:00 2017 

```

+   `date month_name [year]`：我们还可以设置任务在某个特定的时间，具体到某天、某月，甚至可以选择年份；但是我们需要记住，无论选择哪种格式，日期必须跟随时间规格，而不能位于其前：

```
root:# echo "systemctl restart ssh" | at 11:45 Aug 28 warning: commands will be executed using /bin/sh job 12 at Mon Aug 28 11:45:00 2017 root:# echo "systemctl restart ssh" | at 11:45 Aug 28 2018 warning: commands will be executed using /bin/sh job 13 at Tue Aug 28 11:45:00 2018

```

+   `MMDD[CC]YY`：日期格式为月、日、可选世纪和年份，中间无空格：

```
root:# echo "systemctl restart ssh" | at 11:45 070527 warning: commands will be executed using /bin/sh job 14 at Mon Jul 5 11:45:00 2027

```

+   `MM/DD/[CC]YY`：日期格式为月/日/可选世纪/年份，用斜杠分隔：

```
root:# echo "systemctl restart ssh" | at 07:23 PM 08222017 warning: commands will be executed using /bin/sh job 15 at Tue Aug 22 19:23:00 2017

```

+   `DD.MM.[CC]YY`：日期格式为日.月.可选世纪.年份，使用点分隔：

```
root:# echo "systemctl restart ssh" | at 18:05 15.09.29 warning: commands will be executed using /bin/sh job 17 at Sat Sep 15 18:05:00 2029

```

+   `[CC]YY-MM-DD`：日期格式为可选世纪年-月-日，用破折号分隔：

```
root:# echo "systemctl restart ssh" | at 18:15 17-09-15 warning: commands will be executed using /bin/sh job 18 at Fri Sep 15 18:15:00 2017 

```

+   `now + minutes | hours | days | weeks`：我们也可以设置任务从当前系统时间起，按分钟、小时、天数或周数计算的执行时间：

```
root:# date ; echo "systemctl restart ssh" | at now + 1 minutes Thu Apr 27 05:22:11 EDT 2017 warning: commands will be executed using /bin/sh job 20 at Thu Apr 27 05:23:00 2017 root:# date ; echo "systemctl restart ssh" | at now + 1 days Thu Apr 27 05:22:59 EDT 2017 warning: commands will be executed using /bin/sh job 21 at Fri Apr 28 05:22:00 2017 root:# date ; echo "systemctl restart ssh" | at now + 2 weeks Thu Apr 27 05:23:05 EDT 2017 warning: commands will be executed using /bin/sh job 22 at Thu May 11 05:23:00 2017

```

+   `today`：我们可以设置任务在今天的某个特定时间运行；如果没有定义时间，它将会立即执行：

```
root:# date ; echo "systemctl restart ssh" | at today Thu Apr 27 05:25:06 EDT 2017 warning: commands will be executed using /bin/sh job 23 at Thu Apr 27 05:25:00 2017 root:# date ; echo "systemctl restart ssh" | at 14:28 today Thu Apr 27 05:25:19 EDT 2017 warning: commands will be executed using /bin/sh job 24 at Thu Apr 27 14:28:00 2017

```

+   `tomorrow`：我们可以设置任务在第二天的特定时间运行；如果没有定义时间，它将会在创建任务时的同一时间执行，但会是第二天：

```
root:# date ; echo "systemctl restart ssh" | at tomorrow Thu Apr 27 05:27:34 EDT 2017 warning: commands will be executed using /bin/sh job 25 at Fri Apr 28 05:27:00 2017 root:# date ; echo "systemctl restart ssh" | at 14:28 tomorrow Thu Apr 27 05:27:41 EDT 2017 warning: commands will be executed using /bin/sh job 26 at Fri Apr 28 14:28:00 2017

```

时间规格的完整参考资料可见于`/usr/share/doc/at/timespec`。

正如我们从示例中看到的，命令可以从`stdin`或文件中读取，如果使用`-f`选项并指定文件名：

```
root:# echo "systemctl restart ssh" > /root/atjob1 root:# at -f /root/atjob1 14:28 tomorrow warning: commands will be executed using /bin/sh job 27 at Fri Apr 28 14:28:00 2017 

```

一旦任务被放入队列，它会保留一些来自创建时的状态：

除了`BASH_VERSINFO`、`DISPLAY`、`EUID`、`GROUPS`、`SHELLOPTS`、`TERM`、`UID`、`_`和`umask`外，其他环境变量都会保留自调用时的状态。

```
the working directory. the umask.

```

将会导出到任务中的环境库种类取决于未来的开发；例如，如果我们想为某个程序源代码调度编译任务，我们必须从任务本身设置`LD_LIBRARY_PATH`或`LD_PRELOAD`等库，因为这些库不会被继承。

一旦任务执行完毕，其结果将显示在`stdout`和`stderr`中，并通过`/usr/sbin/sendmail`发送邮件给用户；但是，如果任务是通过`su`和`at`执行，并且保留了原始用户 ID，那么结果会发送给最初登录到`su`的用户。

让我们运行一个简单的命令来生成一些输出，然后确保邮件被发送：

```
root:# echo "df" | at -m now

```

现在，我们检查一下根用户的默认别名邮箱（查看`/etc/aliases`以了解邮件会发送给谁）：

```
From root@debian Thu Apr 27 07:56:28 2017 Return-path: <root@debian> Envelope-to: root@debian Delivery-date: Thu, 27 Apr 2017 07:56:28 -0400 Received: from root by spoton with local (Exim 4.84_2)
 (envelope-from <root@debian>) id 1d3i28-0002vS-Ep for root@debian; Thu, 27 Apr 2017 07:56:28 -0400 Subject: Output from your job 37 To: root@debian Message-Id: <E1d3i28-0002vS-Ep@spoton> From: root <root@debian> Date: Thu, 27 Apr 2017 07:56:28 -0400 Filesystem 1K-blocks Used Available Use% Mounted on /dev/sda1 117913932 12476420 99424792 12% / udev 10240 0 10240 0% /dev tmpfs 779256 9192 770064 2% /run tmpfs 1948140 0 1948140 0% /dev/shm tmpfs 5120 4 5116 1% /run/lock tmpfs 1948140 0 1948140 0% /sys/fs/cgroup tmpfs 389628 0 389628 0% /run/user/0

```

现在我们已经了解了如何调度一个任务，接下来看看它支持哪些选项：

+   `-q`: 强制 `at` 使用指定的队列放置作业。队列用单个字符指定，从 a 到 z 和从 A 到 Z；默认队列分别以 *a 代表 at* 和 *b 代表 batch* 命名。具有更高字母的队列具有增加的 niceness，而具有名称 `=` 的特殊队列专用于实际运行的作业。如果将作业提交到具有大写字母的队列，则会被视为提交到 batch；因此，一旦达到时间规范，只有在系统的负载平均值低于阈值时，作业才会执行。

+   `-V`: 只是将实用程序的版本号打印到 `stderr` 并成功退出。

+   `-m`: 一旦作业完成，向用户发送电子邮件，即使作业本身没有输出。

+   `-M`: 永远不向用户发送任何电子邮件。

+   `-f` 文件名: 我们已经看到了这个选项；它强制 `at` 从指定文件中读取要在作业内运行的命令，而不是从 `stdin` 中读取。

+   `-t [[CC]YY]MMDDhhmm[.ss]`: 定义要运行命名为 `at/` 的作业的时间。

+   `-l`: 实际上是 `atq` 的别名。

+   `-r`: 实际上是 `atrm` 的别名。

+   `-d`: 实际上是 `atrm` 的别名。

+   `-b`: 实际上是 `batch` 的别名。

+   `-v`: 在读取作业之前显示将执行作业的时间：

```
root:# at -vf /root/atjob1 14:28 tomorrow Fri Apr 28 14:28:00 2017 warning: commands will be executed using /bin/sh job 31 at Fri Apr 28 14:28:00 2017 

```

+   `-c`: 在 `stdout` 上显示指定的作业：

```
root:# at -c 21 #!/bin/sh # atrun uid=0 gid=0 # mail root 0 umask 22 XDG_SESSION_ID=68; export XDG_SESSION_ID SSH_CLIENT=192.168.0.10\ 32994\ 9999; export SSH_CLIENT SSH_TTY=/dev/pts/0; export SSH_TTY USER=root; export USER MAIL=/var/mail/root; export MAIL PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin; export PATH PWD=/root; export PWD LANG=en_US.UTF-8; export LANG SHLVL=1; export SHLVL HOME=/root; export HOME LOGNAME=root; export LOGNAME SSH_CONNECTION=192.168.0.10\ 32994\ 192.168.0.5\ 9999; export SSH_CONNECTION XDG_RUNTIME_DIR=/run/user/0; export XDG_RUNTIME_DIR cd /root || {
 echo 'Execution directory inaccessible' >&2 exit 1 } systemctl restart ssh

```

+   `batch`: 当系统平均负载低于特定阈值时运行作业，默认阈值为 1.5。但是有一个重要的注意事项：为了正常工作，`batch` 依赖于在 `/proc` 上挂载的 Linux `proc` 文件系统：

```
root:# echo "systemctl restart ssh" | batch warning: commands will be executed using /bin/sh job 33 at Thu Apr 27 07:08:00 2017

```

如果用户在调用 `at` 时未登录，或者当名为 `/var/run/utmp` 的文件不可读时，作业结束时的电子邮件将发送到环境变量 LOGNAME 的值作为帐户找到的值。如果此变量不可用，则当前用户 id 将接收该电子邮件。

+   `atq`: 显示用户的待处理作业列表。如果由超级用户运行，则显示所有帐户的所有计划作业列表。列表的格式在这里：

```
job_id, date, hour, queue, account name

```

在这里，非特权用户看到的是：

```
root:# atq 3 Thu Apr 27 08:00:00 2017 a zarrelli

```

这就是 `root` 在同一系统上的同时看到的内容：

```
root:# atq 2 Wed Apr 26 14:00:00 2017 a root 3 Thu Apr 27 08:00:00 2017 a zarrelli

```

从列表中可以看出，名称为 `a` 和 `indeed` 的队列被指定为单个字符，从 a 到 z 和从 A 到 Z，而默认队列的名称分别为 *a 代表 at* 和 *b 代表 batch*。具有更高字母的队列具有增加的 niceness，而名为 `=` 的特殊队列专门用于实际运行的作业。如果 `atq` 被给定特定队列作为参数，则只显示该队列中的作业。让我们看看 `atq` 在没有参数的情况下能显示什么：

```
root:# atq 5 Wed Apr 26 05:46:00 2017 b root 2 Wed Apr 26 14:00:00 2017 a root 3 Thu Apr 27 08:00:00 2017 a zarrelli

```

现在，让我们将列表限制为仅显示 batch 队列：

```
root:# atq -q b 5 Wed Apr 26 05:46:00 2017 b root

```

因此，`atq` 支持以下选项：

+   `-q`: 它是 `atq` 接受的唯一选项，与 `V` 一起，限制其输出为指定队列的内容

+   `-V`: 只是将实用程序的版本号打印到 `stderr` 并成功退出

+   `atrm`：删除由任务 ID 标识的任务：

```
root:# atq 9 Wed Jan 31 11:45:00 2018 a root 3 Thu Apr 27 08:00:00 2017 a zarrelli 13 Tue Aug 28 11:45:00 2018 a root 29 Fri Apr 28 14:28:00 2017 a root

```

所以，我们有四个任务，其 ID 分别为`3`、`9`、`13`和`29`；让我们将它们删除：

```
root:# atrm 3 9 13 29 And check the queue again: root:# atq root:#

```

没有其他内容了，我们完成了。最后需要注意的是，`atrm`只接受一个选项，那就是`-V`。

到目前为止，我们看到的适用于一次性任务，因为`at`不会允许我们设置某些周期性任务；因此，如果我们想在一段时间内执行一些周期性任务，我们需要依赖另一种工具：著名的 cron 服务。

# cron 调度器

服务器上最常用的工具之一实际上是调度器，即使我们没有意识到自己依赖它有多深。我们系统中一些无需人工干预的服务都是由调度器处理的，它负责在几天、几周甚至几个月的特定时间运行它们。所有这些看似简单、重复的任务，虽然对我们环境的正常运行至关重要，却在幕后默默进行，完成我们不愿意做的事情，例如，在需要时轮换所有系统日志。如果我们每天都得做这些任务，在疯狂的时间点，为所有需要维护的服务？不，我们有更重要的事情要做。cron 调度器没有更重要的事情要做；它的目标是每分钟唤醒一次，检查是否有任务需要执行，这使它成为执行繁琐重复任务的最佳人选，也许是每天或每周在同一时间执行。因此，既然本书的目的之一是帮助我们从系统中获得最大效益，我们将深入了解这个谦逊的助手，学习如何配置和管理它，从而帮助我们完成必要但不那么有趣的系统管理任务。

与`at`一样，crontab 依赖三个不同的组件：一个名为`cron`的工具；一组配置文件，其中最著名的是`/etc/crontab`；还有一个名为 crontab 的客户端/编辑器。看起来有点混乱吗？让我们按顺序进行，并首先了解一下 cron 服务。

# cron

该服务每分钟运行一次，检查存储在其 crontab 文件中的任务，并查看是否需要在当前分钟执行。如果找到任务，则执行；否则，cron 会在下分钟重新运行，依此类推。任务执行后，命令输出将通过邮件发送给 crontab 所有者，或者发送给 crontab 中的 MAILTO 环境变量指定的用户（如果有）。特别地，每分钟，cron 不仅会读取 crontab 文件，还会检查其 spool 目录的修改文件，或者检查 `/etc/crontab` 是否已更改，如果有变化，它会分析所有 crontab 文件的修改时间并重新加载它们，以便获取任务规范的任何更改。因此，如果我们做了更改，也无需担心重新启动 cron。它会自行管理这些更改，不过 cron 也能够应对时钟变动：如果时间回退少于 3 小时，已运行的任务不会重新执行。然后，如果时间向前调整不到 3 小时，*跳过的* 任务将在时钟到达新时间时立即执行。这只会影响那些设置了特定执行时间的任务。所以，使用 `@hourly` 等关键字的任务，以及在小时或分钟规格中使用通配符 `*` 的任务将不会受到影响。如果时钟变化超过 3 小时，所有任务将按照新的时间设置执行。

每个发行版可以实现不同类型的 cron 服务，并且配置文件的位置可能略有不同。如果不确定，`man cron` 和 `man crontab` 可以显示该服务支持的功能以及相关文件所在的路径。

然后，还有一个惊喜：没有 cron。好吧，我们可以笑一笑，因为其实我们称之为 cron 的是该服务的守护进程部分，但提供该服务的实际程序可以根据我们使用的发行版而有所不同。我们有多种调度器可以选择。以下是其中的一些：

+   `vixie-cron`：这是所有现代 cron 的始祖：Paul Vixie 编写的著名 cron，编码于 1987 年。

+   `bcron`：它是一个安全性更强的 cron 替代品。

+   `cronie`：它是基于 Fedora 的 vixie-cron 版本。

+   `dcron`：Dillon 的 cron 是一个精简版的 cron；它既安全又简单。

+   `fcron`：它可以作为经典 vixie cron 的不错替代品，并且它是为非持续运行的系统设计的。因此，它有一些有趣的功能，比如能够在启动时调度任务。

这些只是一些 cron 实现，我们相信某个地方可能还有更多的实现，可能是某个分支或者是某个解决特定需求的原创版本。在本书中，我们将引用安装在 Debian 系统上的 vixie-cron。

我们与 cron 的交互选项不多，但让我们来看一下它支持什么：

+   `-f`：不进行守护进程化，并将保持在前台。这对于调试其运行状态非常有用。

+   `-l`：启用符合 Linux 标准基准的脚本名称，用于 `/etc/cron.d` 中的文件（[`lanana.org/lsbreg/cron/index.html`](http://lanana.org/lsbreg/cron/index.html)）。只有该目录中的文件会受到影响；`/etc/cron.hourly`、`/etc/cron.daily`、`/etc/cron/weekly` 和 `/etc/cron.monthly` 中的文件不会受到影响。

+   `-n`：在任务执行后的电子邮件主题中包含完全限定的域名；否则，只会使用主机名。

+   `-l`：设置日志级别。错误总是会被记录；但是，不同的级别解锁了额外的信息，这些信息通过系统日志设施记录，通常是在 *cron* 设施下的 syslog。可以将单个级别的值相加，得到的结果将启用收集多种信息：

    +   `1`：记录所有 cron 任务的开始

    +   `2`：记录所有 cron 任务的结束状态

    +   `4`：记录所有失败任务的结束状态，因此所有退出状态不为 `0` 的任务都会被记录

    +   `8`：记录所有 cron 任务的进程标识号

    +   `15`：将收集所有在前面级别中捕获的信息（8+4+2+1）

    +   默认的日志级别是 1，但如果我们希望完全禁用日志记录，可以指定 0

使用 cron 时需要注意几点：

cron 守护进程在处理任务时设置了一些环境变量，示例如下：

+   `SHELL`：设置为 `/bin/sh`

+   `LOGNAME`：从与 crontab 所有者相关的 `/etc/passwd` 行内容中设置

+   `HOME`：从与 crontab 所有者相关的 `/etc/passwd` 行内容中设置

+   `PATH`：设置为 `/usr/bin:/bin`

如果用户需要设置其他环境变量，最简单的方式是将它们设置在 vixie-cron 的 crontab 定义中；其他实现（如 cronie）不允许这样做，因此你可以在调用属于该任务的脚本或程序之前，在 crontab 行条目中预先添加这些环境变量：

```
# m h dom mon dow command export HTTP_PROXY=http://192.168.0.1:8080; env >> /var/log/proxy

```

如果我们查看 syslog 文件，我们可以看到 crontab 被安装：

```
Apr 28 16:57:35 moveaway crontab[27929]: (root) REPLACE (root) Apr 28 16:57:35 moveaway crontab[27929]: (root) END EDIT (root) Apr 28 16:58:01 moveaway cron[607]: (root) RELOAD (crontabs/root) Apr 28 16:58:01 moveaway CRON[27977]: (root) CMD (export HTTP_PROXY=http://192.168.0.1:8080; env >> /var/log/proxy)

```

因此，如果我们没有犯任何错误，我们应该看到 `/var/log/proxy` 文件被创建并更新，其中包含 `env` 命令被调用时的环境内容：

```
root:# cat /var/log/proxy LANGUAGE=en_GB:en HOME=/root LOGNAME=root PATH=/usr/bin:/bin LANG=en_GB.UTF-8 SHELL=/bin/sh PWD=/root HTTP_PROXY=http://192.168.0.1:8080 LANGUAGE=en_GB:en HOME=/root LOGNAME=root PATH=/usr/bin:/bin LANG=en_GB.UTF-8 SHELL=/bin/sh PWD=/root

```

`HTTP_PROXY` 环境变量为任务设置，我们将看到这些行逐渐增长到文件中，所以稍后我们将看到如何删除该 crontab 或单个任务。

读取一个环境变量，而不是设置它，这个变量叫做 **MAILTO**。如果定义了它，任务的输出将发送到指定的名称；如果为空，则不会发送邮件。MAILTO 也可以设置为将电子邮件发送给由逗号分隔的用户列表。如果没有设置，MAILTO 将被设置为将任务结果发送给 crontab 所有者。

一些 cron 实现支持 PAM，因此如果它们恰好为用户设置了一个新的 cron 作业，并且我们遇到了一些授权问题，那么我们需要查看以下三个文件：

+   `/etc/cron.allow`

+   `/etc/cron.deny`

+   `/etc/pam.d/cron`

`/etc/at.allow` 是第一个被读取的文件（如果存在）。如果其中找到任何账户名，则这些账户将是唯一被允许使用 crontab 工具提交作业的账户。如果 `/etc/cron.allow` 不存在，那么会解析 `/etc/cron.deny`，其中找到的每个账户名都会被禁止提交作业到 cron。如果文件存在，但为空，则只有超级用户或 `/etc/cron.allow` 中列出的用户才能提交作业。如果 `/etc/cron.allow` 和 `/etc/cron.deny` 都不可用，任何用户都可以提交作业到 cron。

我们已经讨论了 `crontab` 工具。那么它是什么呢？它实际上是让我们编写 crontab 文件的程序，crontab 文件告诉 cron 执行哪些作业，何时执行，以及由谁执行。事实上，每个用户可以拥有自己的 `crontab`，该文件存储在 `/var/spool/cron/crontabs` 中；但我们不应手动编辑这些文件。我们必须依赖 `crontab` 工具，它将允许我们以正确的方式编辑并安装 crontab 文件。那么，如何使用 `crontab` 呢？假设我们已经有了一个包含作业规格的文件，我们稍后会看到如何编写它；我们需要执行的唯一命令是：

```
crontab filename

```

这将从指定的文件为当前用户安装一个新的 crontab，但我们也可以通过命令行手动输入作业详情，使用 `crontab`。

从前面的命令行可以推测，如果没有传递用户名，`crontab` 将作用于调用它的用户的 cron 作业。因此，如果我们想列出或修改用户的 `crontab`，只要我们具有足够的权限（即超级用户权限），我们可以执行以下命令：

```
root:# crontab -u zarrelli -l no crontab for zarrelli

```

如果我们想编辑并安装一个新的 cron 作业，我们可以使用以下语法：

```
crontab -u user -e

```

查看以下示例：

```
root:# crontab -e -u zarrelli no crontab for zarrelli - using an empty one Select an editor. To change later, run 'select-editor'. 1\. /bin/nano <---- easiest 2\. /usr/bin/mcedit 3\. /usr/bin/vim.gtk 4\. /usr/bin/vim.tiny Choose 1-4 [1]: 

```

由于是第一次调用 `crontab` 工具，系统可能会要求选择默认编辑器；如果没有设置，则会实例化 visual 或 editor 环境变量。如果都没有设置，则使用 `/usr/bin/editor`。

在所示的示例中，Debian 发行版触发了 `/etc/alternatives` 的配置，该配置提供了指向默认编辑器的链接。

进入编辑器后，每个 cron 作业必须单独指定在一行上，例如 `* * * * * ps`。

我们稍后会看到这一序列的含义；目前，我们只需要退出编辑器并保存内容：

```
crontab: installing new crontab

```

完成后，`crontab` 会通知我们 crontab 已经安装，显示在 `stdout` 上也许会很有意思：

```
crontab -u zarrelli -l

```

如下所示的示例：

```
root:# crontab -u zarrelli -l # Edit this file to introduce tasks to be run by cron. # # Each task to run has to be defined through a single line # indicating with different fields when the task will be run # and what command to run for the task # # To define the time you can provide concrete values for # minute (m), hour (h), day of month (dom), month (mon), # and day of week (dow) or use '*' in these fields (for 'any').# # Notice that tasks will be started based on the cron's system # daemon's notion of time and timezones. # # Output of the crontab jobs (including errors) is sent through # email to the user the crontab file belongs to (unless redirected). # # For example, you can run a backup of all your user accounts # at 5 a.m every week with: # 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/ # # For more information see the manual pages of crontab(5) and cron(8) # # m h dom mon dow command * * * * * ps

```

如果我们想删除 crontab，可以使用一个便捷选项，将其完全移除：

```
crontab -u zarrelli -r 

```

但要小心——如果你想删除某些任务但保留其他任务，最好的解决方案是再次编辑 `crontab`，删除我们不想要的任务对应的行，然后保存。新的 `crontab` 将包含剩余的任务，并会完全替换旧的 `crontab`。

如果我们不确定是否删除 crontab，可以设置一个指向 `crontab -i -r` 的别名，使用 `-r` 并在删除 crontab 前提示用户确认：

```
root:# crontab -ir crontab: really delete root's crontab? (y/n)

```

我们不会修改一些特定发行版的配置，比如通过 `/etc/crontab` 文件提供的对 `/etc/cron.hourly`、`/etc/cron.daily`、`/etc/cron.weekly` 和 `/etc/cron.monthly` 的支持，否则我们将不得不深入分析所有主流发行版中所有可能的 cron 实现的细节和配置。

我们这里感兴趣的是理解处理 cron 的基本概念，这样无论我们遇到什么不同的实现方式，都能够应对并理解其独特性。现在还有几个有趣的点需要看：一个是我们用来定义任务的语法，另一个是对 anacron 的快速了解：一个我们经常听说的工具。

即使一个 `crontab` 文件乍一看可能有点难懂，但理解字符序列的含义并不难：第一个字段是任务必须执行的分钟；第二个字段是小时；第三个是日期；第四个是月份；第五个是星期几；第六个是要执行的命令。所以，我们几行前创建的 `crontab` 可以按以下表格的方式理解：

| **字段** | ***** | ***** | ***** | ***** | ***** | **ps** |
| --- | --- | --- | --- | --- | --- | --- |
| 分钟 | X |  |  |  |  |  |
| 小时 |  | X |  |  |  |  |
| 日期 |  |  | X |  |  |  |
| 月份 |  |  |  | X |  |  |
| 星期几 |  |  |  |  | X |  |
| 命令 |  |  |  |  |  | X |

字段的含义

好的，现在我们知道这些字段的含义了，那么那些星号是什么，它们在每个字段中可以写什么呢？另一个表格会让这一切更容易理解：

| **字段** | **允许的值** | **元字符** |
| --- | --- | --- |
| 分钟 | 0-59 | * , - / |
| 小时 | 0-23 | * , - / |
| 日期 | 1-31 | * , - / |
| 月份 | 1-12 或 Jan-Dec | * , - / |
| 星期几 | 0-7 或 Sun-Sat | * , - / |

每个字段中可以使用的值

我们快完成了。还有几件事情需要学习，之后我们就能完全理解一行 crontab；但首先，那些元字符是什么，它们意味着什么？

请记住，星期天可以在星期字段中同时用 `0` 和 `7` 来表示。

+   `*`：代表*每个*，可以用于所有字段。所以，在分钟字段中，它表示 cron 每分钟执行一次任务；在小时字段中，每小时执行一次，以此类推。

+   `,`：逗号定义一个列表。例如，在月份的日期字段中，`1`、`5`和`15`会指示 cron 在每月的第一天、第五天和第十五天运行任务。

+   `-`：连字符定义一个包含范围。例如，在星期几的字段中，`4-7`将强制 cron 在星期四到星期天执行任务。

+   `/`：正斜杠用于定义步长，它可以与范围一起使用，这样它会跳过范围内的数字值。例如，分钟字段中的`1-59/2`将给出每小时的所有奇数分钟，因为它从 1 开始，然后等待 2 分钟再执行下一个任务；也可以通过列表来指定：`1`、`3`、`5`、`7`、`9`、`11`、`13`、`15`、`17`、`19`、`21`、`23`、`25`、`27`、`29`、`31`、`33`、`35`、`37`、`39`、`41`、`43`、`45`、`47`、`49`、`51`、`53`、`55`、`57`、`59`。

如我们所见，步长非常实用。正斜杠也可以与星号结合使用：在小时字段中，`*/2`表示*每隔 2 小时*，在月份字段中表示*每隔两个月*，等等。

某些 cron 实现支持一个额外的字段用于*年份*，其值从`1970`到`2099`，并支持`*`和`-`元字符。

在分析 crontab 行之前，我们必须先了解一组特殊标记。我们可以使用一些带有`@`前缀的特殊关键字来定义重复的任务，如下表所示：

| **关键字** | **执行** | **与...相同** |
| --- | --- | --- |
| `@yearly` | 每年执行一次，在 1 月 1 日的午夜时分 | `0 0 1 1 *` |
| `@annually` | 等同于`@yearly` | `0 0 1 1 *` |
| `@monthly` | 每月执行一次，在每月的第一天的午夜时分 | `0 0 1 * *` |
| `@weekly` | 每周执行一次，在周六与周日的午夜之间 | `0 0 * * 0` |
| `@daily` | 每天执行一次，在午夜时分 | `0 0 * * *` |
| `@midnight` | 等同于`@daily` | `0 0 * * *` |
| `@hourly` | 每小时执行一次，在整点时刻 | `0 * * * *` |
| `@reboot` | 在 cron 守护进程启动时执行 | `无` |

具有特殊含义的关键字

`This`替换了前五个字段，可以在没有特殊小时或日期约束的情况下使用它，这样可以在一个通用的时间范围内执行任务。

在继续之前，有几点需要注意：

+   每一行定义 cron 任务的行必须以换行符结尾。

+   命令字段中的百分号`%`会被转换为换行符，所有在第一个`%`之后的数据将被发送到要执行的命令的`stdin`。可以通过将百分号写成`%%`来转义，这样它就不会被解释为换行符。

+   在 crontab 文件中，空行、前导空格和制表符不会被解析。如果一行以`#`开头，它将被视为注释并且不会被解析。

+   在 crontab 文件中，我们可以设置一些变量或定义 cron 任务；不允许做其他事情。

+   变量可以定义为`VARIABLE_NAME = value`。

+   等号周围的空格是可选的。

+   变量不会进行扩展或替换。所以，`VARIABLE_NAME=$LOGNAME`不会实例化`VARIABLE_NAME`，因为该值字符串甚至没有解析进行扩展或替换。

+   空值必须用引号括起来，如果值中包含空格，必须使用引号以保留它们。

+   不支持每个用户的时区。系统默认时区将被使用。

所以，考虑到这一点，让我们看一下 cron 作业规范：

| 35 | 1-24/4 | * | * | Mon,Thu | /opt/scripts/script.sh |
| --- | --- | --- | --- | --- | --- |
| 分钟 | 小时 | 月中的天数 | 月份 | 星期几 | 命令 |
| 在每个 4 小时的第 35 分钟，1 到 24 小时之间，星期一和星期四执行 |

一个有用的网格来理解作业规范行

通过表格形式，作业规范更容易理解，如果我们使用一些关键字，它会变得更加简单：

```
@weekly /opt/scripts/script.sh

```

就这些，定义被缩短为两个简单的字段。但对字段的调整可能会导致一些棘手的情况：

```
1-59/2 * * * * /opt/scripts/script.sh

```

这是什么作用？它会每个奇数分钟执行一次脚本，而不是偶数分钟，奇数分钟。

但如果系统重启或是桌面计算机，可能会关闭好几天或几周怎么办？使用 cron 会导致某些作业根本不执行或被跳过，这不是理想的行为。这个时候 anacron 就派上用场了：即使我们在预定时间之后打开桌面，它也会运行作业。怎么做到的呢？很简单，这个工具会记录所有作业及其执行时间，使用的是一系列带有时间戳的文件，保存在`/var/spool/anacron`中。

让我们有条理地继续，看看驱动 anacron 的文件，文件名是`/etc/anacron`。

为了更好地理解它，我们必须查看其内容：

```
root:# cat /etc/anacrontab # /etc/anacrontab: configuration file for anacron # See anacron(8) and anacrontab(5) for details. SHELL=/bin/sh PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin HOME=/root LOGNAME=root # These replace cron's entries 1 5 cron.daily run-parts --report /etc/cron.daily 7 10 cron.weekly run-parts --report /etc/cron.weekly @monthly 15 cron.monthly run-parts --report /etc/cron.monthly

```

正如我们所看到的，我们可以在 crontab 中使用变量，但作业定义的格式略有不同，并且可以采用以下之一的语法：

```
period delay job_name command @period_identifier delay job_name command

```

+   `period`：以天为单位，指定作业运行的频率。例如，10 表示每 10 天运行一次。

+   `@period`：允许使用一些关键字来指定频率：`@daily`，`@weekly`，和`@monthly`分别表示每天、每周和每月执行一次。

+   `delay`：以分钟为单位，定义 anacron 在达到阈值后执行计划作业的延迟时间。

+   `job-name`：我们可以为作业提供任何标识符，但不允许使用斜杠。anacron 将使用它作为作业时间戳文件的名称。

+   `command`：可以是任何命令。

正如我们所看到的，示例中的标准 anacron 文件将运行 run-parts 实用程序，并传入目录作为参数。而 run-parts 的确切任务是执行它在给定目录中找到的脚本。因此，解读 anacron 行应该变得简单了，不是吗？编辑它也很简单：我们可以手动进行，无需特殊的工具或程序。

anacron 的工作方式非常直接有效：它检查在 `anacrontab` 文件中指定的每个任务，并查看它是否在字段中指定的最近 x 时间内执行过。如果任务未执行并且达到阈值，它将在等待任务定义第二个字段中设置的延迟后执行任务。任务执行后，anacron 会在与任务相关的时间戳文件中记录日期；下次，它只需要读取该文件就能知道接下来该做什么：

```
root:# cat /var/spool/anacron/cron.weekly 20170424

```

一旦所有计划的任务都已执行完毕，anacron 会退出；但是，如果我们发送一个 SIGUSR1 信号给 anacron 来终止它，它将等待任何正在运行的任务完成，然后干净地退出。

但 anacron 还有一个任务，在任务执行完毕后会执行：如果任务产生了任何输出，它会发送邮件给运行 anacron 的用户，通常是 root，或者是 anacrontab 中 MAILTO 环境变量指定的用户。如果实例化了 `LOGNAME` 变量，它会作为邮件的发送者。

最后，让我们看看 anacron 支持哪些选项：

+   `-f`：强制 anacron 执行所有已定义的任务，而不考虑时间戳。

+   `-u`：仅更新任务的时间戳为当前日期，而不执行任务。

+   `-s`：串行执行任务；即在开始下一个任务之前，anacron 会等待当前任务完成。

+   `-n`：忽略每个任务的延迟规格，并在达到阈值时立即执行任务。意味着启用 -s 选项。

+   `-d`：通常 anacron 会在后台运行，但此选项会强制其在前台运行。它对于调试非常有用，因为 anacron 会将运行时消息发送到 stderr 和 syslog。输出的邮件仍然会发送给接收者。

+   `-q`：不将消息打印到 stderr，并且意味着启用 `-d` 选项。

+   `-t` 文件：从指定的文件读取任务定义，而不是默认的 anacrontab。

+   `-T`：测试 anacrontab 的有效性。如果有任何错误，将显示错误信息，anacron 会返回值 `1`；否则，返回值为 `0`。

+   `-S` 目录：使用指定的目录来存储时间戳文件。

+   `-V`：打印 anacron 版本并退出。

+   `-h`：打印使用帮助并退出。

# 概要

我们现在有两种方法来执行任务：一种是守护进程，它在后台分叉并一直工作，执行一些 hopefully 重要的任务；另一种是调度器，非常适合那些必须按周期执行的任务。这些工具确实能帮助我们保持一切井然有序，并通过简化服务器或甚至桌面维护，执行复杂且乏味的任务。但 Bash 不仅仅由命令、脚本、任务和服务组成：它是我们每天工作的家。它是我们的游乐场，我们的工作台，需要熟悉的地方。所以，下一章将讨论一些工具、配置和建议，帮助我们将 Bash 打造成一个舒适的数字生活空间。
