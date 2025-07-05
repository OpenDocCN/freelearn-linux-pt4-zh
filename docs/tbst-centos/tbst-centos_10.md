# 第十章 故障排除 DNS 服务

由于域名系统（DNS）可能是 IT 领域最具影响力的技术之一，在您的职业生涯中，您应该预期它会成为一个需要持续管理和解决的问题。DNS 服务器有多种形式，您的控制级别可能取决于您可以访问的基础设施类型，但无论如何，问题无疑会在整体或部分上保持一致。

在已经回顾了`dig`的优点和各种其他网络工具后，在本书的最后几页，我们将讨论故障排除 CentOS 7 的过程，特别是针对与 DNS 相关的监控问题，作为整体系统健康检查的一部分。

本章中，我们将：

+   学习如何使用`hostnamectl`更改系统主机名，并通过`/etc/hosts`管理`fqdn`。

+   学习如何在按下恐慌按钮之前执行一些基本的系统和基于 BIND 的完整性检查

+   学习如何使用`iftop`监控带宽

+   学习如何清空缓存

# 更改主机名并管理 FQDN

更改一个权威服务器或递归（缓存）服务器的主机名不一定是一个 DNS 问题（本质上），但是，作为经验法则，系统的主机名与 DNS 密切相关，作为整体系统的一部分；因此，我们现在将回顾更改主机系统主机名的过程。

要查看当前主机名，您可以使用以下语法：

```
# hostnamectl status

```

但是，您可以选择使用以下命令查看静态、临时或漂亮的名称：

```
# hostnamectl status --static
# hostnamectl status --transient
# hostnamectl status --pretty

```

基于此，要更改主机名，您应该使用以下命令：

```
# hostnamectl set-hostname <new-host-name>

```

或者，您可以选择使用以下命令更新静态主机名：

```
# hostnamectl --static set-hostname <new-host-name>

```

完成此操作后，主机名更改将自动应用于内核，无需重启。随后，任何对主机名的更改都必须反映在系统的网络配置中。

为此，您必须检查并确认`/etc/hosts`已更新，方法是输入：

```
# nano /etc/hosts

```

一个典型的安装后（或默认）文件如下所示：

```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
```

要继续，请通过替换适合您系统配置的相关值，按如下方式修改文件：

```
127.0.0.1       localhost localhost.localdomain localhost4 localhost4.localdomain4
192.168.1.183   <servername>.<domain-name>.<tld> <server-name>
::1             localhost localhost.localdomain localhost6 localhost6.localdomain6
```

例如，如果您的服务器名称是`centos7`（主机名），并且它位于`centos.local`域中，则您的文件应如下所示：

```
127.0.0.1       localhost localhost.localdomain localhost4 localhost4.localdomain4
192.168.1.200   centos7.centos.local centos7
::1             localhost localhost.localdomain localhost6 localhost6.localdomain6
```

完成后，您可以通过输入以下命令来确认主机名：

```
# hostname

```

然后，您可以通过输入以下命令来确认并打印当前的完全限定域名（FQDN）：

```
# hostname –f

```

但是，如果您发现新的主机名没有被接受或更新，您只需输入以下命令来解决任何未解决的问题：

```
# systemctl restart systemd-hostnamed

```

然后，您可以通过以以下方式运行`ping`来确认这些更改：

```
# ping –c 4 <hostname>

```

基于前面的示例，如果您运行：

```
# ping –c 4 centos

```

`ping`的输出现在应显示如下所示的 FQDN：

```
PING centos7.centos.local (192.168.1.200) 56(84) bytes of data.
64 bytes from centos7.centos.local (192.168.1.183): icmp_seq=1 ttl=64 time=0.048 ms
64 bytes from centos7.centos.local (192.168.1.183): icmp_seq=2 ttl=64 time=0.057 ms
64 bytes from centos7.centos.local (192.168.1.183): icmp_seq=3 ttl=64 time=0.038 ms
64 bytes from centos7.centos.local (192.168.1.183): icmp_seq=4 ttl=64 time=0.094 ms
```

# 使用 BIND 执行系统完整性检查

DNS 故障可能是由一些看似无害的问题引起的，这些问题通常源于基本的配置错误或系统近期的更改（和更新）。这种情况是可能发生的，因此，在遇到紧急情况之前，进行一系列常规检查总是有用的。

所以，从基本的工具开始，比如`ping`、`nslookup`或`dig`，你可以开始测试可能存在的问题区域。例如，你可以像这样使用`telnet`：

```
# telnet <remote-server-address> 53

```

`telnet`命令是一个既简单又实用的工具，如果连接被拒绝或连接时间过长，你可以通过简单地重命名反向 DNS 文件并重试来排除 RDNS 错误的可能性。

现在，如果你这样做，请确保正向 DNS 保持功能正常，并重新尝试 telnet 连接。如果成功，你就知道是 RDNS 出了问题，可以通过使用`nslookup`确认正向区域来再次检查：

```
# nslookup <remote-server-address>

```

然而，如果你仍然无法连接，那么你应该恢复 RDNS 文件，检查防火墙配置（如果使用`SELinux`，也需要检查），然后使用以下命令查看端口 53 的当前状态：

```
# netstat -tulpn | grep :53

```

如果你仍然遇到困难，那么你还应该检查日志文件，如下所示：

```
# tail -f /var/log/messages

```

现在确认 named 服务的状态：

```
# ps ax | grep named

```

然而，如果你使用以下代码，会获得更有用的信息：

```
# systemctl status named

```

该命令的输出可能会先显示以下提示信息，然后列出当前的解析错误：

```
named.service - Berkeley Internet Name Domain (DNS)
 Loaded: loaded (/usr/lib/systemd/system/named.service; disabled)
 Active: active (running) since Sun 2015-05-03 07:55:14 EDT; 16s ago
 Process: 2853 ExecStart=/usr/sbin/named -u named $OPTIONS (code=exited, status=0/SUCCESS)
 Process: 2851 ExecStartPre=/usr/sbin/named-checkconf -z /etc/named.conf (code=exited, status=0/SUCCESS)
 Main PID: 2855 (named)
 CGroup: /system.slice/named.service
 └─2855 /usr/sbin/named -u named

May 03 07:55:15 centos7 named[2855]: error (network unreachable) resolving 'pdns196.ultradns.info/AAAA/IN': 2001:500:49::1#53
May 03 07:55:15 centos7 named[2855]: error (network unreachable) resolving 'pdns196.ultradns.info/A/IN': 2001:500:1b::1#53
May 03 07:55:15 centos7 named[2855]: error (network unreachable) resolving 'pdns196.ultradns.info/AAAA/IN': 2001:500:1b::1#53
May 03 07:55:15 centos7 named[2855]: error (network unreachable) resolving 'ns1.isc.ultradns.net/AAAA/IN': 2610:a1:1014::e8#53
May 03 08:55:15 centos7 named[2855]: error (network unreachable) resolving './DNSKEY/IN': 2001:7fd::1#53
May 03 08:55:15 centos7 named[2855]: error (network unreachable) resolving './NS/IN': 2001:7fd::1#53
May 03 08:55:15 centos7 named[2855]: error (network unreachable) resolving './DNSKEY/IN': 2001:dc3::35#53
May 03 08:55:15 centos7 named[2855]: error (network unreachable) resolving './NS/IN': 2001:dc3::35#53

```

如果一切正常，下一步将是检查你的区域文件是否有错误：

```
# named-checkconf /etc/named.conf

```

请记住，如果没有发现错误，则不会提供任何输出。然而，如果你遇到错误，根据系统的不同，输出可能如下所示：

```
/etc/named.conf:106: missing ';' before 'zone'

```

在此基础上，你还可以使用提供的工具`named-checkzone`来检查并确认区域文件的完整性和语法。它通过制造虚假的负载情况来工作，从而运行与文件加载到系统之前相同的验证规则。通过这种方式，它提供了一种非常可靠的方式来确认新区域文件在添加到生产服务器之前的完整性。

要使用`named-checkzone`，你需要输入：

```
# named-checkzone localhost /var/named/<filename>

```

或者，你可以使用以下命令检查特定主机名：

```
# named-checkzone <hostname> /var/named/<filename>

```

另一方面，如果你当前没有配置任何名字服务器，或者发现名字服务器无法访问，首先要检查系统的解析器配置，确保其配置正确，可以通过打开以下文件来确认：

```
# nano /etc/resolv.conf

```

打开该文件后，你会注意到，第一项应该指向主名字服务器，后面跟着次名字服务器。一般来说，你应该期望看到至少两个名字服务器，但三个也是常见的。根据系统的配置，`/etc/resolv.conf`文件的示例可能如下所示：

```
search local
    nameserver XXX.XXX.XXX.XXX
    nameserver XXX.XXX.XXX.XXX
```

例如，如果你使用的是 Google 的 DNS，那么主 DNS 服务器将是 8.8.8.8，辅助 DNS 服务器将是 8.8.4.4。因此，你的文件将如下所示：

```
search local
    nameserver 8.8.8.8
    nameserver 8.8.4.4
```

假如系统使用 `127.0.0.1` 或 `localhost` 作为主要或唯一的 DNS 服务器，那么你还应该预期在主 DNS 配置文件中使用转发器。具体的操作方式取决于你当前使用的 DNS 应用程序。然而，在 `BIND` 的情况下，这将在`/etc/named.conf`的`options`部分中找到，如下所示：

```
forwarders {
XXX.XXX.XXX.XXX;
XXX.XXX.XXX.XXX;
};
```

此外，在检查`BIND`配置文件时，你还应确保禁用递归查询，以减少递归查询，从而保护系统免受`DDoS 放大`攻击的风险，示例如下：

```
allow-transfer     { 127.0.0.1; XXX.XXX.XXX.XXX; };
recursion no;
```

现在，确认 BIND 是否确实在端口 53 上监听：

```
listen-on port 53 { 127.0.0.1; XXX.XXX.XXX.XXX; };
```

请记住，在测试 DNS 时，你应确保 UDP 和 TCP 端口上的 53 端口是开放的。因为大于 512 字节的 UDP 查询可能会被截断，而 TCP 被用作故障转移解决方案。所以，记得检查防火墙配置，看看是否需要做出任何必要的更改；如果需要，记得重启防火墙。

完成此操作后，你可以通过输入以下命令检查`BIND`的当前状态：

```
# systemctl status named

```

然后，根据需要，使用以下一个或多个系统命令来启动或停止服务：

```
# systemctl start named
# systemctl restart named
# systemctl stop named

```

基于此，并假设区域文件不仅正确（包括正向和反向区域），而且维护了正确的权限，你还应该检查`/etc/hosts`，以确认系统是否具备适当的配置。

要做到这一点，请使用你喜欢的文本编辑器打开`/etc/hosts`，像这样：

```
# nano /etc/hosts

```

这个文件的示例如下：

```
127.0.0.1       localhost.localdomain localhost
XXX.XXX.XXX.XXX	<servername>.<domainname>.<tld>  <servername>
::1             localhost6.localdomain6 localhost6
```

如果你需要将任何特定或专用的 IP 地址添加到主机文件中，只需像这样将它们添加到前三行下面：

```
127.0.0.1       localhost.localdomain localhost
XXX.XXX.XXX.XXX	<servername>.<domainname>.<tld>  <servername>
::1             localhost6.localdomain6 localhost6

XXX.XXX.XXX.XXX        live.intranet.local
XXX.XXX.XXX.XXX        staging.intranet.local
XXX.XXX.XXX.XXX        production.intranet.local
```

最后，你还应该检查并确认以太网设备的配置，可以通过在`/etc/sysconfig/network-scripts`中打开相关的以太网连接来访问。如你所见，这个配置文件通常会指明对特定的`Gateway`、`DNS`和`Domain`设置的偏好。在这方面，确保这些设置的准确性被视为良好的做法。

如此处所示，基于之前的示例，`cat /etc/sysconfig/network-scripts/ifcfg-eth0`的输出可能如下所示：

```
TYPE=Ethernet
BOOTPROTO=none
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
NAME=eth0
UUID=30420474-a7c9-4ea1-97b2-17f3b3a104cc
ONBOOT=yes
HWADDR=00:1C:42:1F:E6:BA
IPADDR0= XXX.XXX.XXX.XXX
PREFIX0= XXX.XXX.XXX.XXX
GATEWAY0= XXX.XXX.XXX.XXX
DNS1= XXX.XXX.XXX.XXX
DNS2= XXX.XXX.XXX.XXX
DOMAIN=local
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes

```

在此示例中，重要的是要记住，如果系统使用 CentOS 网络管理器，则必须添加 DNS 条目。此外，由于这里只显示了单一的以太网接口，如果系统具有多个网卡，则务必确保此检查清单中包括额外的以太网配置文件。

最后，如果你进行任何网络更改，请运行以下命令以确保新设置生效：

```
# systemctl restart network

```

现在，其中`XXX.XXX.XXX.XXX`是相关名称服务器的 IP 地址，在通过这些基本的健全性检查后，你应该能够通过运行一个成功的`nslookup`请求，像这样确认或重新确认你的系统配置：

```
# nslookup www.google.com XXX.XXX.XXX.XXX

```

或者，你可以通过以下方式使用`dig`：

```
# dig www.google.com XXX.XXX.XXX.XXX

```

因此，在检查系统的关键组件并确保防火墙没有阻止任何重要的 UDP 和 TCP 流量之后，你应该有信心，任何此时报告的非递归问题或警告（很可能）都会基于与 DNS 应用程序本身相关的特定配置问题。

# 使用 iftop 监控带宽

配置不当或有问题的 DNS 服务器可能导致多种问题，包括应用服务器故障和整体网络变慢。然而，在深入了解 DNS 的内部工作原理之前，重要的是要意识到网络变慢可能由多种原因引起；考虑到这一点，通常建议参考一个叫做`iftop`的包。

正如前面章节中讨论的那样，你会注意到`iftop`与`top`命令类似，但不同于`top`命令的是，你会发现它的目的是专门关注测量主机服务器与外部参考点（IP 地址）之间的网络连接带宽。

要安装此包，你需要 EPEL 仓库，并且在启用此仓库后，通过阅读前面章节的说明，`iftop`可以通过以下命令安装：

```
# yum install iftop

```

运行基本包无需任何参数。事实上，一个典型的`iftop`会话可以通过以下命令启动：

```
# iftop

```

否则，在拥有多个以太网接口的服务器上，其中`X`是分配给你的以太网接口的名称，你可以使用以下参数来显示特定设备的结果：

```
# iftop -i ethX

```

默认情况下，`iftop`将尝试将所有 IP 地址解析为主机名，但你可以使用`-n`选项避免这种情况，像这样：

```
# iftop -n -i ethX

```

此外，由于默认显示用于显示主机之间的总体带宽，一个有用的功能是切换此显示模式，改为显示每个主机用于通信的端口。为此，只需在`iftop`会话中按下*P*键，即可在显示所有端口和标准带宽读数之间切换。

### 注意

**其他有用的键盘快捷键包括：**

+   使用*T*键在显示类型之间切换

+   使用*Shift* + *P*键暂停当前显示

+   使用*J*和*K*键滚动显示

由于显示屏相对详细且不言自明，我将不会详细介绍该包本身。然而，在某些情况下，你应该意识到你可以使用`iftop`通过初始化适当的过滤器，像这样，来调查网络范围内的包流：

```
# iftop -F 192.168.1.0/255.255.255.0 -i eth0

```

你可以使用以下语法选择显示特定端口的结果：

```
# iftop -i eth0 -f 'port http'

```

或者，使用`-B`选项以以下方式，显示的带宽速率将以每秒字节（bytes per second）显示，而不是以比特每秒（bits per second）显示：

```
# iftop -B -F 192.168.1.0/255.255.255.0 -i eth0

```

请记住，故障排除的艺术在于尽可能快速地识别问题的根源。网络缓慢确实可能由配置错误或有问题的 DNS 引起，但同样也可以说，DNS 有时会被冤枉，问题可能源于带宽问题。因此，通过使用这个小巧但极其有用的工具包，你可能已经为自己节省了大量时间和精力，并永久解决了原本被认为是 DNS 问题的问题。

你可以通过输入以下命令了解更多关于`iftop`的信息：

```
# man iftop

```

# 刷新缓存

Time to Live（TTL）因子也可能影响当前问题。在某些情况下，简单的 dig 请求显示域名服务器与本地 DNS 显示的记录不同，那么（除了等待自动更新发生之外）一种不同的处理方法是刷新缓存。

对于 BIND，只需像这样重启服务：

```
# systemctl restart named

```

然而，如果不想采取那么极端的措施，你也可以使用以下语法：

```
# rndc flush

```

然后运行服务状态检查：

```
# systemctl status named

```

在这方面，你现在应该能看到以下通知：

```
received control channel command 'flush'
flushing caches in all views succeeded

```

或者，你可以使用以下语法针对特定域进行操作：

```
# rndc flushname google.com

```

运行了`systemctl status named`命令后，你将看到以下报告：

```
received control channel command 'flushname google.com'
flushing name 'google.com' in all cache views succeeded

```

现在，在将即时更改传播到本地桌面客户端时，你需要关闭所有正在运行的应用程序，然后遵循以下其中一个步骤。

对于 Windows 用户，只需打开命令提示符并使用：

```
# ipconfig /flushdns

```

对于 Mac OS-X 10.10，打开终端并使用以下命令：

```
# sudo discoveryutil udnsflushcaches

```

对于 Mac OS-X 10.5.2 到 10.6，使用以下命令：

```
# dscacheutil -flushcache

```

对于旧版本的 OS-X，使用此命令：

```
# lookupd -flushcache

```

# 总结

本章的目的并非一定要解决与 DNS 配置相关的问题，因为有许多书籍已经全面讨论了这一主题。然而，本章的内容旨在从 DNS 作为网络服务功能的角度来探讨这一主题。其结果是通过介绍一些基本技巧，帮助我们解决在权威或缓存域名服务器（DNS 服务器）出现故障时进行故障排除的艺术。

因此，从更改主机名到检查带宽，从刷新缓存到运行一些标准的系统检查，你应该意识到，故障排除并不仅仅是安装和配置软件包。在很多方面，真正的专业故障排除员很少会从头开始构建系统，但他们会被要求解决一个明显的问题，这个问题在本书的上下文中，通常是一些更紧迫的问题，甚至可能是网络环境本身固有的问题。

# 参考文献

+   红帽客户门户: [`access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/`](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/)

+   iftop 维基百科主页: [`en.wikipedia.org/wiki/Iftop`](http://en.wikipedia.org/wiki/Iftop)

+   域名系统安全扩展维基百科主页: [`en.wikipedia.org/wiki/Domain_Name_System_Security_Extensions`](http://en.wikipedia.org/wiki/Domain_Name_System_Security_Extensions)

+   BIND 项目主页: [`www.isc.org/downloads/bind/`](https://www.isc.org/downloads/bind/)

+   BIND 维基百科主页: [`en.wikipedia.org/wiki/BIND`](http://en.wikipedia.org/wiki/BIND)

+   RHEL 客户门户 – 第十九章: [`access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/5/html/Deployment_Guide/ch-bind.html`](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/5/html/Deployment_Guide/ch-bind.html)
