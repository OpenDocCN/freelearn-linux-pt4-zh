# 第十一章. 使用 firewalld 的网络安全

在 RHEL7 中，`netfilter`（基于内核的防火墙）的默认用户界面是`firewalld`。管理员现在可以选择使用`firewalld`或`iptables`来管理防火墙。无论选择哪种方式，底层都可以实现基于内核的`netfilter`防火墙。此新界面的前端命令是`firewall-cmd`。它的主要优点是能够在防火墙运行时刷新`netfilter`设置。使用`iptables`接口无法做到这一点；此外，我们还可以使用区域管理功能。这使我们能够根据连接的网络配置不同的防火墙规则。

在本章中，我们将涵盖以下主题：

+   防火墙状态

+   路由

+   区域管理

+   源管理

+   使用服务的防火墙规则

+   使用端口的防火墙规则

+   伪装和网络地址转换

+   使用丰富的规则

+   实现直接规则

+   回退到 iptables

# 防火墙状态

防火墙服务可以为您的 RHEL 系统及服务提供来自本地网络或互联网其他主机的保护。尽管防火墙通常在网络的边界路由器上维护，但通过基于主机的防火墙（如 Linux 内核中的`netfilter`防火墙）可以提供额外的保护。在 RHEL 7 上，`netfilter`防火墙可以通过`iptables`或`firewalld`服务来实现，后者为默认选项。

可以使用`systemctl`命令以正常方式查询`firewalld`服务的状态。如果该服务正在运行，将提供详细的输出，其中包括`firewalld`的**PID**（**进程 ID**）以及最近的日志消息。以下是一个命令提取及来自 RHEL7.1 的输出截图：

```
# systemctl status firewalld

```

![防火墙状态](img/image00316.jpeg)

如果您只需要快速检查并获取较少的输出，可以使用`firewall-cmd`命令。这是用于管理`firewalld`的主要管理工具。`--state`选项将提供所需的所有信息，如下截图所示：

![防火墙状态](img/image00317.jpeg)

如果`firewalld`没有激活，输出将显示为`not running`（未运行）。

# 路由

虽然防火墙中不严格需要网络路由，但您可能需要在 RHEL7 系统上实现路由。通常，这与拥有多个网络接口卡的多网卡系统相关；但是，这并不是网络路由的要求，网络路由允许将数据包转发到正确的目标网络。网络路由在`procfs`中的`/proc/sys/net/ipv4/ip_forward`文件中启用。如果该文件的值为`0`，则禁用路由；如果值为`1`，则启用路由。可以使用`echo`命令设置此值，如下所示：

```
# echo 1 > /proc/sys/net/ipv4/ip_forward

```

然而，这将在下一次重启之前保持开启，重启后路由将恢复为配置的设置。要使此设置永久生效，传统上我们使用`/etc/sysctl.conf`文件。现在建议将您的自定义配置添加到`/etc/sysctl.d/`。以下是一个示例：

```
# echo "net.ipv4.ip_forward  =  1" > /etc/sysctl.d/ipforward.conf

```

这将创建一个文件并设置其指令。为了使此设置在下一次重启之前生效，我们可以使用`sysctl`命令，如以下命令所示：

```
# sysctl -p /etc/sysctl.d/ipforward.conf

```

这在以下截图中也可以看到。这里，您可以看到我们在实施更改之前从`procfs`读取运行配置。它还显示了对运行中的`procfs`的更改：

![Routing](img/image00318.jpeg)

# 区域管理

在`firewalld`中，您会发现一个新功能，它更侧重于移动系统——例如笔记本电脑——那就是区域的引入。然而，这些区域同样可以在多宿主系统中使用，将不同的 NIC 与适当的区域关联。无论是在移动系统还是多宿主系统中使用区域，防火墙规则可以分配给区域，并且这些规则将与该区域中包含的 NIC 相关联。如果一个接口没有显式分配到某个区域，它将成为默认区域的一部分。要查询您系统上的默认区域，我们可以使用`firewall-cmd`命令，如下所示：

```
# firewall-cmd --get-default-zone

```

如果您需要列出系统上所有配置的区域，可以使用以下命令：

```
# firewall-cmd --get-zones

```

以下截图展示了此命令及 RHEL 7.1 上的默认区域：

![Zone management](img/image00319.jpeg)

更有用的是，我们可以显示已分配接口的区域；如果没有进行分配，所有接口将属于公共区域。`--get-active-zones`选项将帮助我们做到这一点，如以下命令所示：

```
# firewall-cmd --get-active-zones

```

如果我们需要更详细的输出，可以列出所有区域名称、关联的规则和接口。以下命令展示了如何实现这一点：

```
# firewall-cmd --list-all-zones

```

如果您需要使用区域，您可以选择默认区域并将接口分配给特定的区域。首先，按如下方式分配一个新的默认区域：

```
# firewall-cmd --set-default-zone=work

```

在这里，我们将默认区域重定向到工作区域。这样，所有未显式分配的 NIC 将参与工作区域。前面的命令应返回`success`。请查看以下截图，了解它是如何工作的：

![Zone management](img/image00320.jpeg)

我们还可以按如下方式显式地将区域分配给接口：

```
# firewall-cmd --zone=public --change-interface=eno16777736

```

通过此命令所做的更改将是临时的，直到下一次重启；要使其永久生效，我们需要添加`--permanent`选项：

```
# firewall-cmd --zone=public --change-interface=eno16777736 --permanent

```

使设置永久生效将使配置保留在位于`/etc/firewalld/zones/`目录中的区域文件中。在我们的例子中，文件是`/etc/firewalld/zones/public.xml`。在此详细实现永久更改后，我们可以使用`cat`命令列出 XML 文件的内容。以下截图显示了这一点：

![区域管理](img/image00321.jpeg)

我们可以通过查询单个 NIC 来查看它所关联的区域，或者列出区域内的所有接口；以下命令展示了这一点：

```
# firewall-cmd --get-zone-of-interface=eno16777736
# firewall-cmd --zone=public --list-all

```

### 提示

您可以使用 Tab 补全来辅助`firewall-cmd`的选项和参数。

如果提供的区域不足，或者名称可能不适合您的命名方案，可以创建自己的区域并添加接口和规则。添加区域后，您可以重新加载配置以立即使用，方法如下：

```
# firewall-cmd --permanent --new-zone=packt
# firewall-cmd --reload

```

`--reload`选项可以重新加载配置，允许当前连接继续不中断；而`--complete-reload`选项将在过程中停止所有连接。

# 源管理

使用分配给区域的接口时，可能遇到的问题是它不会区分网络地址。通常，这不是问题，因为每个网络接口卡（NIC）只绑定一个网络地址；但是，如果您有多个地址绑定到 NIC 上，可能希望实现`firewalld`源功能。像接口一样，源也可以分配给区域。在以下命令中，我们将添加一个网络范围到`trusted`区域，并将另一个范围（可能在同一个 NIC 上）添加到`public`区域：

```
# firewall-cmd --permanent --zone=trusted --add-source=192.168.1.0/24
# firewall-cmd --permanent --zone=public --add-source=172.17.0.0/16

```

类似于接口，将源绑定到区域将激活该区域，并且可以通过`--get-active-zones`选项列出该区域。

# 使用服务的防火墙规则

当我们想到防火墙时，通常会想到允许或拒绝端口访问。使用服务的 XML 文件可以简化端口管理，每个服务可能列出多个端口。另一个需要注意的点是，`firewalld`守护进程的默认策略是拒绝访问，因此任何所需的访问必须明确授予与服务关联的端口。要列出在默认区域上已允许的服务，我们可以简单地使用`--list-services`选项，如下面的示例所示：

```
# firewall-cmd --list-services

```

类似地，我们可以通过包含`--zone=`选项来访问特定区域中允许的服务。这可以在以下示例中看到：

```
# firewall-cmd --zone=home --list-services

```

该命令的输出如下面的截图所示。它列出了与`home`区域相关的服务：

![使用服务的防火墙规则](img/image00322.jpeg)

当您开始启用服务时，可以轻松地通过区域允许预定义的服务。预定义服务以 XML 文件形式列出，存储在`/usr/lib/firewalld/services`目录中。它们在以下截图中列出，供您查看：

![使用服务的防火墙规则](img/image00323.jpeg)

### 提示

RHEL 7 代表着更成熟的 Linux 发行版，因此它认识到将`/usr`目录与`root`文件系统分开的需求已不再推荐，`/lib`、`/bin`和`/sbin`目录都被软连接到`/usr/`后的相应目录。因此，`/lib`现在与`/usr/lib`相同。以下屏幕截图展示了这一点：

![使用服务的防火墙规则](img/image00324.jpeg)

在定义您自己的服务时，您可以在`/etc/firewalld/services`目录下创建 XML 文件。squid 代理服务器没有自己的服务文件，如果我们选择将其作为服务而不仅仅是开放所需端口，文件会类似于`/etc/firewalld/services/squid.xml`，如下所示：

```
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>Squid</short>
  <description>Squid Web Proxy</description>
  <port protocol="tcp" port="3128"/>
</service>
```

假设我们在`Enforcing`模式下使用 SELinux，我们需要使用以下命令为新文件设置正确的上下文：

```
# cd /etc/firewalld/services
# restorecon squid.xml

```

该文件的权限应为`640`，并可以使用以下命令进行设置：

```
# chmod 640 /etc/firewalld/services/squid.xml

```

`ls -lZ`的输出应类似于以下屏幕截图，显示正确的 SELinux 上下文和权限：

![使用服务的防火墙规则](img/image00325.jpeg)

现在定义了新服务或使用现有服务后，我们可以将它们添加到某个区域。如果我们使用默认区域，可以通过以下命令轻松完成。请注意，我们在开始时重新加载配置，以便识别新的 squid 服务，如下所示：

```
# firewall-cmd --reload
# firewall-cmd --permanent --add-service=squid
# firewall-cmd --reload

```

类似地，要更新指定的非默认区域，我们将使用以下命令：

```
# firewall-cmd --permanent --add-service=squid --zone=work
# firewall-cmd --reload

```

如果我们稍后需要将此服务从工作区域中移除，可以使用以下命令：

```
# firewall-cmd --permanent --remove-service=squid --zone=work
# firewall-cmd --reload

```

# 使用端口的防火墙规则

在前面的示例中，squid 服务只需要一个端口，我们可以轻松地添加一个端口规则来允许访问服务。尽管过程简单，但在某些组织中，偏好仍然是创建服务文件，在描述字段中记录对端口的需求。

如果我们需要添加端口，我们可以使用`--add-port`和`--remove-port`等选项。以下命令展示了如何在工作区域中添加 squid 的 TCP 端口`3128`，而不需要定义服务文件：

```
# firewall-cmd --permanent --add-port=3128/tcp --zone=work
# firewall-cmd --reload

```

# 伪装和网络地址转换

如果您的`firewalld`服务器是运行 RHEL 7 的网络路由器，您可能希望为内部私有网络中的主机提供互联网访问。如果是这种情况，我们可以启用伪装。这也叫做**NAT**（**网络地址转换**），即服务器的公共 IP 地址被内部客户端使用。为此，我们可以利用内置的内部和外部区域，并在外部区域上配置伪装。内部网卡应分配给内部区域，外部网卡应分配给外部区域。

要在外部区域建立伪装，我们可以使用以下命令：

```
# firewall-cmd --zone=external --add-masquerade

```

使用`--remove-masquerade`选项可以删除伪装。我们还可以使用`--query-masquerade`选项查询某个区域的伪装状态。在下面的截图中，我们可以看到伪装被启用，然后通过查询返回`yes`结果：

![伪装和网络地址转换](img/image00326.jpeg)

# 使用丰富规则

`firewalld`丰富语言使管理员能够轻松配置更复杂的防火墙规则，而无需了解`iptables`语法。这可以包括日志记录和源地址检查。

要添加允许 NTP 连接的规则到默认区域，并且对连接进行每分钟最多 1 次的日志记录，可以使用以下命令：

```
# firewall-cmd --permanent \
--add-rich-rule='rule service name="ntp" audit limit value="1/m" accept'
# firewall-cmd --reload

```

同样，我们可以添加一个规则，仅允许来自一个子网的访问到 squid 服务：

```
# firewall-cmd --permanent \
--add-rich-rule='rule family="ipv4" \ 
source address="192.166.0.0/24" service name="squid" accept'
# firewall-cmd --reload

```

从以下截图中，我们可以看到添加的丰富规则：

![使用丰富规则](img/image00327.jpeg)

### 注意

Fedora 项目维护了`firewalld`丰富规则的文档，您可以通过[`fedoraproject.org/wiki/Features/FirewalldRichLanguage`](https://fedoraproject.org/wiki/Features/FirewalldRichLanguage)访问它们，以获取更多详细示例。

# 实现直接规则

如果您之前有使用`iptables`的经验，并且希望将`iptables`的知识与`firewalld`中的功能结合使用，直接规则将帮助您进行迁移。首先，如果我们想在 INPUT 链上实现一个规则，可以使用以下命令检查当前设置：

```
# firewall-cmd --direct --get-rules ipv4 filter INPUT

```

如果您没有添加任何规则，输出将为空。我们将添加一个新规则，并使用优先级`0`。这意味着它将排在链的顶部；然而，当没有其他规则时，这个优先级并没有太大意义。我们确实需要验证规则是否按正确的顺序添加，以便在有其他规则的情况下能够处理：

```
# firewall-cmd --permanent --direct --add-rule ipv4 filter \
INPUT 0 -p tcp --dport 3128 -j ACCEPT
# firewall-cmd --reload

```

## 还原为 iptables

此外，如果您最熟悉`iptables`服务，也没有任何限制阻止您继续使用它。

首先，我们可以使用以下命令安装`iptables`：

```
# yum install iptables-service

```

我们可以屏蔽`firewalld`服务，以更有效地禁用该服务，防止它在没有先取消屏蔽的情况下被启动：

```
# systemctl mask firewalld

```

我们可以使用以下命令启用`iptables`：

```
# systemctl enable iptables
# systemctl enable ip6tables
# systemctl start iptables
# systemctl start ip6tables

```

永久规则像往常一样添加，通过`/etc/sysconfig`目录和`iptables`与`ip6tables`文件。

# 总结

`firewalld` 项目由 Fedora 维护，是 Linux 内核上 `netfilter` 防火墙的新管理服务和接口。作为管理员，我们可以选择使用这个默认服务，或者切换回 `iptables`；然而，`firewalld` 可以为我们提供重新加载配置而不丢失连接的能力，并提供从 `iptables` 迁移的机制。我们已经看到，当我们需要在单一网卡上共享地址范围时，如何使用区域来隔离网络接口和源地址。无论是网卡还是源地址，都不与区域绑定。然后，我们可以向区域添加规则来控制对资源的访问。这些规则是基于服务或端口的。如果需要更复杂的配置，我们可以选择使用富规则或直接规则。富规则是用 `firewalld` 的富语言编写的，而直接规则则使用 `iptables` 语法编写。
