# 第七章：故障排除安全问题

在本章中，我们将讨论 CentOS 7 的安全问题。然而，我们不会采用传统的服务器加固方法，而是通过以另一种方式回顾 SSH、`SELinux`、HIDS 和 Firewalld，鼓励你更全面地探索系统。

在本章中，我们将：

+   了解如何使用 `aureport` 生成审计报告并使用 `setroubleshoot` 审计 `SELinux`

+   学习如何添加和管理 `SSH` 横幅，并使用 `FIGlet` 创建自定义横幅

+   了解更多关于调整 `SSH` 服务的基本知识

+   学习如何安装 `Tripwire` 并为系统提供入侵检测系统

+   了解更多关于 Firewalld、区域管理以及如何添加/删除接口、端口和伪装基础设施的内容

+   学习如何移除 Firewalld 并返回到 iptables

# 使用 aureport 和 setroubleshoot 审计 SELinux

禁用 `SELinux` 是一个相当常见的操作。通常在使用托管控制面板时，或者当某些特定应用程序似乎因 `SELinux` 启用而无法运行时，禁用 `SELinux` 是一个常见的做法。在这种情况下，禁用 `SELinux` 是一种经过验证的技术，能够为系统管理员节省大量时间。对于许多人来说，这是一个自动反应，而其他人则认为与 `SELinux` 相关的工具更适合用于桌面、工作站、带有 GUI 的服务器或受控网络环境。然而，事实很简单，禁用 `SELinux` 会移除一个关键的安全组件，从而使系统暴露。我同意，`SELinux` 是一个复杂的系统，对于那些希望享受它所提供保护的人来说，通过像这样运行 `aureport` 选项，生活可以变得更简单：

```
# aureport --avc | tail -n 10

```

这将提供一个 `avc` 消息的列表，其输出可能类似于以下内容：

```
AVC Report
========================================================
# date time comm subj syscall class permission obj event
========================================================
1\. 04/18/2015 13:50:53 ? system_u:system_r:init_t:s0 0 (null) (null) (null) unset 384
2\. 04/18/2015 13:55:49 ? system_u:system_r:init_t:s0 0 (null) (null) (null) unset 789

```

`aureport` 工具旨在创建基于列的报告，显示审计日志文件中记录的事件，进一步来说，你还可以使用相同的工具创建一个可执行文件的列表，具体变化如下：

```
# aureport -x

```

其输出结果根据你系统的不同，可能会类似于以下内容：

```
5988\. 05/03/2015 12:40:01 /usr/sbin/crond cron ? 0 773
5989\. 05/03/2015 12:40:01 /usr/sbin/crond cron ? 0 774
5990\. 05/03/2015 12:50:01 /usr/sbin/crond cron ? -1 775
5991\. 05/03/2015 12:50:01 /usr/sbin/crond cron ? -1 776
5992\. 05/03/2015 12:50:01 /usr/sbin/crond cron ? 0 778
5993\. 05/03/2015 12:50:01 /usr/sbin/crond cron ? 0 779
5994\. 05/03/2015 12:50:01 /usr/sbin/crond cron ? 0 780
5995\. 05/03/2015 12:50:01 /usr/sbin/crond cron ? 0 781
5996\. 05/03/2015 13:00:01 /usr/sbin/crond cron ? -1 782
5997\. 05/03/2015 13:00:01 /usr/sbin/crond cron ? -1 783
5998\. 05/03/2015 13:00:01 /usr/sbin/crond cron ? 0 785
5999\. 05/03/2015 13:00:01 /usr/sbin/crond cron ? 0 786
6000\. 05/03/2015 13:00:01 /usr/sbin/crond cron ? 0 787
6001\. 05/03/2015 13:00:01 /usr/sbin/crond cron ? 0 788
6002\. 05/03/2015 13:01:01 /usr/sbin/crond cron ? -1 789
6003\. 05/03/2015 13:01:01 /usr/sbin/crond cron ? -1 790
6004\. 05/03/2015 13:01:01 /usr/sbin/crond cron ? 0 792
6005\. 05/03/2015 13:01:01 /usr/sbin/crond cron ? 0 793
6006\. 05/03/2015 13:01:01 /usr/sbin/crond cron ? 0 794
6007\. 05/03/2015 13:01:01 /usr/sbin/crond cron ? 0 795

```

其他人可能希望使用这个工具来生成完整的身份验证报告，方法如下：

```
# aureport -au -i

```

其输出结果类似于以下内容：

```
Authentication Report
============================================
# date time acct host term exe success event
============================================
1\. 04/18/2015 12:40:57 root 192.168.1.17 ssh /usr/sbin/sshd yes 343
2\. 04/18/2015 12:40:57 root 192.168.1.17 ssh /usr/sbin/sshd yes 346
3\. 04/18/2015 19:28:26 root 192.168.1.17 ssh /usr/sbin/sshd yes 1099
4\. 04/18/2015 19:28:26 root 192.168.1.17 ssh /usr/sbin/sshd yes 1102
5\. 04/19/2015 04:57:06 root 192.168.1.17 ssh /usr/sbin/sshd yes 345

```

要生成失败身份验证事件的摘要报告，请使用以下命令：

```
# aureport -au --summary -i --failed

```

你可以使用以下语法创建一个成功身份验证事件的相对摘要报告：

```
# aureport -au --summary -i --success

```

因此，鉴于您可以获得的报告深度，当您处理运行`SELinux`的系统时，作为故障排除者，您的首要任务是考虑在审计系统时`aureport`的好处。作为补充，您还需要考虑一个名为`setroubleshoot`的工具。

`setroubleshoot`工具可以通过以下语法安装：

```
# yum install setroubleshoot setools

```

完成此操作后，您现在已经为系统配备了一个工具，它将主动从位于`/var/log/audit/audit.log`的日志文件中返回公告，并将其转化为更“人性化”的形式。这个工具叫做`sealert`，其目的是发布关于与`SELinux`相关的任何问题的报告和解决方案。

通过调用以下命令，可以启动一个进程：

```
# sealert -a /var/log/audit/audit.log

```

然而，如果您预计会返回大量数据，那么以下变体可能更适合您的需求：

```
# sealert -a /var/log/audit/audit.log | less

```

然而，在我们结束关于审计`SELinux`的讨论之前，对于那些运行无头服务器并希望接收电子邮件警报的用户，可能需要进行最终的配置更改。

为此，我们将打开我们喜欢的文本编辑器中的以下文件：

```
# nano /etc/setroubleshoot/setroubleshoot.conf

```

向下滚动该文件，找到`[email]`部分，并通过替换相关文本来添加您的电子邮件地址：

```
from_address = SELinux_Troubleshoot
```

现在，通过自定义以下命令，创建相关的收件人列表：

```
echo "email@domain.com" >> /var/lib/setroubleshoot/email_alert_recipients

```

`setroubleshoot`命令可能并不是适合每个人和每个环境的完美解决方案，但使用此包的效果是，无论您是运行无头服务器、带 GUI 的服务器，还是桌面工作站，`SELinux Alert`都是一种解决方案，使您能够继续使用并享受`SELinux`的好处，而不会牺牲您的安全性。

鉴于此主题的重要性，关于`SELinux`和`setroubleshoot`的进一步阅读可以在本章末尾找到。

# SSH 横幅

使用 SSH 横幅并不完全是纯粹的故障排除（是的，我们确实涉及了硬化的主题）。然而，由于通常认为所有服务器应该携带某种形式的法律横幅、通知或安全警告，这些内容应在 SSH 认证过程开始和结束之前展示给用户，因此这是我们应当探索的一个领域。故障排除者并不负责构建系统，但他们负责修复系统，因此这是您需要了解的内容。而且，作为进入 SSH 世界的一个入口，学习如何开发您自己的（并且独特的）SSH 登录横幅将是一个不错的起点。

要在 SSH 认证之前显示一个横幅，您需要在您喜欢的编辑器中打开以下文件：

```
# nano /etc/issue.net

```

现在，按照需要添加所需的消息、通知或安全警告，但请记住，尽量保持简短和简洁。

例如，您可能想说：

```
Warning! You are entering a secure area. This service is restricted to authorized users only. All activities on this system are logged. Any unauthorized access will be fully investigated and reported to the appropriate law enforcement agencies.
```

完成此操作并保存文件后，你应该像这样在你最喜欢的文本编辑器中打开 SSH 的主配置文件：

```
# nano /etc/ssh/sshd_config

```

然后向下滚动，直到找到以下这一行：

```
#Banner
```

取消注释并将正确的路径添加到 `issue.net`，如下所示：

```
Banner /etc/issue.net
```

现在保存并关闭文件，然后按照以下方式重启 SSH 服务：

```
# systemctl restart sshd

```

此时，你应该随时通过输入以下命令检查 SSH 服务的状态：

```
# systemctl status sshd

```

你可以通过以下方式限定 SSH 使用的横幅设置：

```
# grep -i banner /etc/ssh/sshd_config

```

但是，假设你想提供一条独特的消息，将纯文本转换为一个大型 ASCII 横幅。

为此，我们需要安装一个名为 `FIGlet` 的小工具：

```
# yum install figlet

```

要使用 `FIGlet`，你只需使用以下语法：

```
# figlet "My Message Here"

```

然而，出于 SSH 横幅的目的，我们希望创建一个存储在本地文件中的消息，如下所示：

```
# figlet "My Message Here" > /path/to/banner.txt

```

完成此操作后，只需返回以下文件：

```
# nano /etc/ssh/sshd_config

```

现在找到你之前取消注释的这一行：

```
Banner /etc/issue.net
```

用 `FIGlet` 创建的新横幅文件的目标路径替换成以下路径：

```
Banner /path/to/banner.txt
```

最后，保存并关闭文件，然后像这样重启 SSH 服务：

```
# systemctl restart sshd

```

你可以说到此为止，但如果你希望提供一个额外的登录后消息，可以通过编辑以下文件来实现：

```
# nano /etc/motd

```

同样，简单地在保存并关闭文件之前添加所需的消息。完成这些步骤后，下次用户完成 SSH 身份验证时，他们将不仅看到服务器消息，还会看到一个附加的消息，从而为你提供充足的机会，在需要时向系统用户提供适当的指示和报告。

请记住，登录横幅对于两个主要原因非常有用。它们不仅在用户访问系统之前提供一条小消息，还可以警告未授权访问，同时向系统管理员提供重要信息，而无需他们主动请求。

# 调整 SSH

SSH 是与系统通信的最终方式。它是系统生命线中的一个重要服务，并且它维护着一个系统范围的配置文件，使得系统管理员能够修改守护进程的操作。

SSH 访问通常使用以下语法：

```
# ssh username@ipaddress

```

然而，如果系统特别慢，排查问题的第一步是使用替代的调试模式，如下所示：

```
# ssh username@ipaddress -vvv

```

既然如此，让我们更仔细地查看这个文件，以帮助你排查 `sshd` 守护进程的整体问题。

我们将开始打开以下文件，在我们最喜欢的文本编辑器中：

```
# nano /etc/ssh/sshd_config

```

当处理字典攻击、扫描器或机器人时，作为一种良好的实践，你可以通过简单地将 `#Port 22` 替换为完全不同的值，例如 `Port 2222` 来更改 SSH 端口。

你还可以通过更新以下值来限制 root 登录（这总是被推荐的）：

```
PermitRootLogin no
```

要禁用隧道式明文密码，您应该取消注释以下行：

```
PermitEmptyPasswords no
```

SSH 日志有时可能会很难解读，因此，在我们结束对 SSH 的简要回顾之前，如果系统正处于一个难以诊断的阶段，通常可以通过简单地取消注释并更新日志记录参数来解决，方法如下：

```
#LogLevel INFO
```

修改为：

```
LogLevel VERBOSE
```

否则，如有必要，可以通过以下方式启用更高级别的日志记录：

```
LogLevel DEBUG
```

现在，最后的修改可能无法防止攻击，但通过要求 SSH 通过正向和反向 DNS 查找远程主机名，将在系统日志文件中生成适当的警告。为此，只需找到以下行：

```
#UseDNS yes
```

更新此行使其读取：

```
UseDNS yes
```

然而，如果 SSH 仍然有些迟缓，那么确保 SSH 不需要进行反向 DNS 查找可以大大改善这个情况。为此，只需将前面一行更改为：

```
UseDNS no
```

此外，也有可能由于使用了`GSSAPI`认证而引发问题。这种情况并不常见，因为这是 SSH 的一项功能，当需要`GSSAPI`服务器验证相关用户凭据时才会调用。为了避免这种情况，您应添加或编辑以下行，使其读取：

```
GSSAPIAuthentication no
```

此外，您还可能希望考虑超时的问题。这个常见问题可以通过配置正确的`ServerAliveInterval`、`ServerAliveCountMax`和`TCPKeepAlive`值来解决。我在这里给出一个简单的建议，但您应该记得确保这些值适合您的需求。

例如，以下规则表示每 60 秒会发送一个数据包：

```
ServerAliveInterval 15
ServerAliveCountMax 3
TCPKeepAlive yes
```

调整以下值可以提供更稳定的连接：

```
ClientAliveInterval 30
ClientAliveCountMax 5
```

最后，为了让 SSH 服务更安全一些，滚动到主配置文件的底部，添加以下行以维护一个允许进行 SSH 认证的用户名列表：

```
AllowUsers <username1> <username2>
```

或者，您也可以通过这种方式简化身份管理过程，而不是按用户逐个启用访问权限：

```
AllowGroup <groupname>
```

您的工作几乎完成，但话说回来，根据您故障排除 SSH 守护进程的原因，您必须记住，完成工作后，SSH 服务必须确保不受到攻击的可能性。因此，请始终记住，一个成功的故障排除会话不仅会修复问题，还将确保服务器的安全和稳定运行。

例如，对于那些没有使用`SELinux`、`fail2ban`或其他类似安全措施的用户，您可以随时通过输入以下命令来查看登录记录：

```
# cat /var/log/secure | grep 'sshd'

```

输出将如下所示：

```
May  3 13:57:24 centos7 sshd[2479]: pam_unix(sshd:session): session closed for user root
May  3 13:57:28 centos7 sshd[3313]: Accepted password for root from 192.168.1.17 port 51093 ssh2
May  3 13:57:28 centos7 sshd[3313]: pam_unix(sshd:session): session opened for user root by (uid=0)

```

如果您希望查看失败的登录尝试列表，可以尝试以下方法：

```
# cat /var/log/secure | grep 'sshd.*Failed'

```

可以通过以下方式查看接受的登录尝试：

```
# cat /var/log/secure | grep 'sshd.*Accepted'

```

# 使用 Tripwire 进行入侵检测

Tripwire 是一个 **基于主机的入侵检测系统** (**HIDS**)。它通过收集配置和文件系统的详细信息，利用这些信息提供一个参考点，用于比较系统的先前状态和当前状态，这一过程通过监控哪些文件或目录最近被添加或修改，谁修改了它们，做了哪些更改，以及何时进行了更改来实现。

如前一章所讨论的，你需要访问 EPEL 仓库才能获得 Tripwire。当你准备好时，可以像这样安装：

```
# yum install tripwire

```

要开始使用 Tripwire，你需要使用以下语法创建适当的本地和站点密钥：

```
# tripwire-setup-keyfiles

```

当提示时，为站点和本地密钥文件添加一个密码短语。Tripwire 会建议你使用大写字母、小写字母、数字和标点符号的组合，完成后，你将被要求使用先前创建的站点密码短语签署配置文件。

一旦此过程完成，你可以通过修改以下文件来自定义 Tripwire：

```
# nano /etc/tripwire/twpol.txt

```

在开始之前，建议你先阅读 `twpol.txt`，因为许多默认的目录未必在你的系统上可用。这些额外的行不会引发特定问题，但如果你希望避免无意义的错误消息，应将它们注释掉。

你可以通过注释掉以下行来实现这一点：

```
### Filename: /root/.Xauthority
### No such file or directory
### Continuing...
```

此外，你还应该花一些时间查看以下文件，以便为适当的用途定制 Tripwire：

```
# nano /etc/tripwire/twcfg.txt

```

所以，在做出相关更改后，你现在应该通过以下方式更新 Tripwire 策略文件：

```
# tripwire --update-policy --secure-mode low /etc/tripwire/twpol.txt

```

Tripwire 现在将通过各种屏幕阶段来引用你的更改；完成后，你现在应该能够像这样初始化 Tripwire 数据库：

```
# tripwire --init

```

Tripwire 现在将开始扫描系统，但根据系统的整体大小，这可能需要一些时间：

```
Wrote database file: /var/lib/tripwire/server1.server1.com.twd
The database was successfully generated.

```

完成后，你可以使用以下语法运行 Tripwire 报告：

```
# tripwire --check --interactive

```

通过运行上述命令，Tripwire 将自动在 `vi` 中打开报告，从此以后，所有后续报告都将在比较模式下进行。因此，完成此操作后，何不利用这个机会在你的主文件夹中创建一些简单的文本文件或目录，然后重新运行报告，这样 Tripwire 的发现会变得更加明显。

请记住，如果对文件系统的任何更改被认为是系统入侵的结果，管理员将会收到通知，他们需要使用可以信任的文件和目录来恢复系统。因此，所有系统更改必须通过 Tripwire 进行验证。

你可以通过运行以下命令来验证当前的策略文件：

```
# tripwire --check

```

你可以通过像 `mutt` 这样的工具，通过电子邮件发送 Tripwire 报告，方法如下：

```
# yum install mutt
# tripwire --check | mutt -s "Tripwire report" email@domain.com

```

或者通过修改每日 `cron` 作业的后续部分：

```
# nano /etc/cron.daily/tripwire-check

```

通过添加以下行：

```
test -f /etc/tripwire/tw.cfg &&  /usr/sbin/tripwire --check | /bin/mail -s "Tripwire File Integrity Report" emailaddress@domain.com
```

当然，在短短的几段话中，我们已经迈出了构建全面主机入侵系统的一小步。它并不是纯粹意义上的故障排除，但在未来某个时刻，它将帮助您诊断问题。而且，从现在到那时，您可以通过查看手册来进一步了解`Tripwire`，例如这样：

```
# man tripwire

```

然而，在我们结束之前，此时我建议您确保`twpol.txt`和`twcfg.txt`文件的安全。充分意识到 Tripwire 的策略文件比这里建议的要更加可扩展，为了帮助您持续学习，我在本章的结尾提供了该项目主页的链接。

# Firewalld – 区域、服务和端口管理

Firewalld 的目的是替代 iptables，并通过允许配置更改而不停止当前连接来改善安全管理。Firewalld 作为守护进程运行，允许规则即时添加和更改，并使用网络区域来定义与所有关联网络连接的信任级别。对于故障排除人员来说，这提供了多种灵活的选项，但更重要的是，必须理解，虽然一个连接只能属于单一的一个区域，但一个区域可以跨多个网络连接使用。

要了解 Firewalld 当前是否正在运行，您可以输入：

```
# firewall-cmd --state

```

要列出预定义的区域，您可以使用：

```
# firewall-cmd --get-zones

```

### 注意

这些区域可以定义为：

+   `drop`：在此区域，传入的网络数据包会被丢弃（没有回复），仅允许发起外出的网络连接

+   `block`：在此区域，只有在本系统内发起的网络连接才是可能的，因为所有传入的网络连接都会被以`icmp-host-prohibited`消息拒绝

+   `public`：此区域用于您不信任其他计算机的区域；仅接受选定的传入连接

+   `external`：此区域用于启用了伪装的外部网络，仅接受选定的传入连接

+   `dmz`：这是非军事区

+   `work/home/internal`：此区域用于大多数信任网络中其他计算机的环境；同样，只有选定的传入连接被接受

+   `trusted`：此区域接受所有网络连接

然而，通过扩展此命令，我们还可以通过输入以下命令来发现默认区域：

```
# firewall-cmd --get-default-zone

```

这个值可以使用以下语法进行更新：

```
# firewall-cmd --set-default-zone=<new-zone-name>

```

更进一步，我们可以扩展此命令，不仅提供区域的列表，还能提供类似以下的网络接口信息：

```
# firewall-cmd --get-active-zones

```

在这种情况下，网络接口可以使用以下语法进行管理：

```
# firewall-cmd --zone=<zone-name> --add-interface=<device-name>
# firewall-cmd --zone=<zone-name> --change-interface=<device-name>
# firewall-cmd --zone=<zone-name> --remove-interface=<device-name>

```

我们可以使用以下命令更改网络接口的分配，并将其绑定到不同的区域：

```
# firewall-cmd --permanent --zone=<zone-name> --change-interface=<device-name>

```

最后，您可以通过输入以下命令获取关于任何特定区域的所有相关信息：

```
# firewall-cmd --zone=<zone-name> --list-all

```

您可以列出所有支持的服务，使用：

```
# firewall-cmd --get-services

```

然后，你可以使用以下命令管理某一区域内的附加服务：

```
# firewall-cmd --zone=<zone-name> --add-service=<service-name>
# firewall-cmd --zone=<zone-name> --remove-service=<service-name>

```

否则，使用以下命令列出特定区域内的所有开放端口：

```
# firewall-cmd --zone=<zone-name> --list-ports

```

你可以像这样管理 TCP/UDP 端口的添加或移除：

```
# firewall-cmd --zone=<zone-name> --add-port=<port-number/protocol>
# firewall-cmd --zone=<zone-name> --remove-port=<port-number/protocol>

```

最后，在不试图使命令数组过于复杂的情况下，你可以按以下方式配置地址伪装：

```
# firewall-cmd --zone=<zone-name> --add-masquerade
# firewall-cmd --zone=<zone-name> --remove-masquerade

```

然后像这样查询它：

```
# firewall-cmd --zone=<zone-name> --query-masquerade

```

你可以使用这些命令管理端口转发：

```
# firewall-cmd --zone=<zone-name> --add-forward-port=<port-number>
# firewall-cmd --zone=<zone-name> --remove-forward-port=<port-number>

```

例如，在典型配置中，如果你想将所有面向端口 22 的数据包转发到端口 2222，你可以使用以下语法：

```
# firewall-cmd --zone=external --add-forward-port=22:proto=tcp:toport=2222

```

如你所见，Firewalld 是全面的，但考虑到有许多命令可以讨论，本书的目的是展示故障排除人员能够动态管理防火墙架构，而无需停止或重启防火墙服务。这是 iptables 无法实现的；在许多方面，基于相关的学习曲线和大量新命令，这一特性可能会成为一种划时代的成功。

有关 Firewalld 的更多信息可以在本章的最后找到。

# 移除 Firewalld 并回到 iptables

Firewalld 可能并不适合每个人，你可能更喜欢 iptables。所以，最后，如果你发现自己不想使用 Firewalld，你可以轻松地回到 iptables。

首先，你应该通过以下方式禁用 Firewalld：

```
# systemctl disable firewalld
# systemctl stop firewalld

```

然后，你应该通过输入以下命令安装和配置 iptables：

```
# yum install iptables-services
# touch /etc/sysconfig/iptables
# touch /etc/sysconfig/ip6tables

```

现在，使用以下命令启动 `iptables` 服务：

```
# systemctl start iptables
# systemctl start ip6tables
# systemctl enable iptables
# systemctl enable ip6tables

```

从此以后，你将以 `iptables` 作为你首选的防火墙服务。然而，在离开之前，最好重启服务器，以便内核能处理新的配置。

为此，输入以下命令：

```
# reboot

```

# 总结

从 SSH 横幅和通知的更被动的定位，到构建基于主机的入侵检测系统的严格方法，本章介绍了多种方法，旨在提供解决安全问题的前进方向。我们不仅讨论了 Firewalld 的发布，以及它如何在不中断服务的情况下动态重新设计整个防火墙环境，还讨论了 OpenSSH，并展示了如何回归使用 iptables。

我相信大家都会同意，安全是一个大主题，其方法和途径贯穿了本书的每一页。然而，在浏览了本章之后，我希望这块小小的垫脚石能在未来为你提供帮助，接下来我们将讨论如何解决数据库服务的故障。

# 参考文献

+   红帽客户门户：[`access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/`](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/)

+   Fedora 项目 – SELinux 故障排除工具 (setroubleshoot): [`fedorahosted.org/setroubleshoot/wiki/SETroubleShoot%20Overview`](https://fedorahosted.org/setroubleshoot/wiki/SETroubleShoot%20Overview)

+   Oracle – 配置和使用 SELinux: [`docs.oracle.com/cd/E37670_01/E36387/html/ol_selinux_sec.html`](http://docs.oracle.com/cd/E37670_01/E36387/html/ol_selinux_sec.html)

+   Fedora 项目 – 文档/草稿/SELinux/SETroubleShoot/开发者常见问题: [`fedoraproject.org/wiki/Docs/Drafts/SELinux/SETroubleShoot/DeveloperFAQ`](http://fedoraproject.org/wiki/Docs/Drafts/SELinux/SETroubleShoot/DeveloperFAQ)

+   FIGlet 主页: [`www.figlet.org`](http://www.figlet.org)

+   Tripwire 项目主页: [`sourceforge.net/projects/tripwire/`](http://sourceforge.net/projects/tripwire/)

+   SSH 维基百科页面: [`en.wikipedia.org/wiki/Secure_Shell`](http://en.wikipedia.org/wiki/Secure_Shell)

+   SSH 项目主页: [`www.openssh.com`](http://www.openssh.com)

+   FirewallD Fedora 项目: [`fedoraproject.org/wiki/FirewallD`](https://fedoraproject.org/wiki/FirewallD)

+   RHEL – 使用防火墙: [`access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Security_Guide/sec-Using_Firewalls.html`](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Security_Guide/sec-Using_Firewalls.html)
