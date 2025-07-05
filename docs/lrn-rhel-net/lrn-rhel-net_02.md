# 第二章：配置网络设置

坐在这里急切地敲击键盘，我合理地希望本章标题能在某种程度上暗示我们将要讨论的内容。因此，我期待当我揭示我们将在本章学习如何配置 RHEL 7 系统的网络时，不会让你感到太惊讶。不过，稍微拆解一下，我们不仅仅讨论网络。首先，我们会确保你了解如何在 Linux 中获取管理员权限。虽然这与网络无关，但获取管理员权限是我们书中许多操作的基础。一旦我们完成了初步的权限部分，我们将迅速转向调查如何在 RHEL 7 中配置网络。在本章中，我们将讨论以下主题：

+   提升权限

+   使用 `ip` 和 `hostnamectl`

+   NetworkManager 和网络脚本

+   与 NetworkManager 交互

+   使用控制中心

+   使用 `nmtui` 菜单

+   与 `nmcli` 交互

# 提升权限

作为 RHEL 服务器或桌面系统的管理员，有时需要 root 访问权限。`root` 用户或用户 ID `0` 是系统的本地管理员。尽管可以直接以 `root` 用户身份登录系统，但与大多数系统一样，建议根据需要获取 `root` 访问权限。可以使用两种机制：

+   替代用户或 `su` 命令

+   使用 `sudo` 命令

首先，我们将介绍 `su` 命令。

## `su` 命令

当用户在没有指定用户名的情况下输入 `su` 命令时，系统将提示输入 root 密码。如果身份验证成功，用户将进入 root shell。以下是使用 `su` 获取 root 权限的有效机制：

+   `su -l`：这会启动一个完整的登录 shell，所有的环境变量都会为 root 设置。用户的工作目录会被更改为 root 用户的主目录，通常是 `/root`。

+   `su`：这与 `su -l` 相同。

+   `su`：这会启动一个非登录 shell，在这个 shell 中不会加载 root 用户的完整配置文件或环境变量。结果是，一些变量——比如 `$USER`——不会被重置，当前目录也保持不变。尽管是以非登录 shell 展示，但仍然需要输入正确的 root 密码进行身份验证。

使用 `su` 命令是一种简单的方式来获取权限。这可能是管理员的一种方便选择。对于一个小型环境，这可能是可以接受的；然而，在企业环境中，这通常不可行，因为审计受限。尽管可以追踪谁使用了 `su` 命令来获取权限，但所有这些操作都会记录在 `/var/log/secure` 日志文件中。由于从此时起所有活动都将以 `root` 用户身份记录，我们无法详细了解是哪位管理员执行了某个特定命令。另一大缺点是，用户需要知道 root 密码。这再次是一个重大的安全问题，对我来说完全不能接受。

虽然我们想要使用 `su` 命令，但可以通过 **PAM**（**可插拔认证模块**）和 `wheel` 组来控制谁可以访问 `su`。通过将用户添加到特殊的管理组 `wheel`，我们可以将对 `su` 命令的访问限制为该组的成员。

要将用户添加到 `wheel` 组，您需要以 root 用户身份运行 `# usermod -a -G wheel <用户名>`，其中 `<用户名>` 是应添加到 `wheel` 组的帐户的登录名。`-a` 选项用于将一个组追加到用户当前的组成员列表中。

为了确保只有 `wheel` 组的成员可以使用 `su` 命令，您必须以 `root` 用户身份编辑 `/etc/pam.d/su` 的 PAM 配置文件。在您喜欢的文本编辑器中打开该文件——例如 `vi` 或 `nano`——并通过删除行首的 `#` 字符来取消注释以下行：

```
#auth required pam_wheel.so use_uid
```

一旦此更改生效，只有 `wheel` 管理组的成员才能使用 `su` 命令切换到另一个用户 ID。

如果您愿意，您可以对 `/etc/pam.d/su` PAM 文件进行第二次更改，以确保 `wheel` 组的成员可以轻松访问 `su`。此文件的推荐设置将仅限于那些安全性不重要的系统——如教室或实验室计算机。

编辑 `/etc/pam.d/su` 文件，并通过删除行首的 `#` 字符来取消注释以下行：

```
#auth sufficient pam_wheel.so trust use_uid
```

有了这个更改，`wheel` 组的成员在使用 `su` 时不需要进行密码认证；这是 root 用户的默认行为。

### 提示

这两个 PAM 编辑在我们讨论的 Red Hat 变体中是一致的：RHEL 7、CentOS 7 和 Fedora 21。此外，默认情况下，`root` 用户是 `wheel` 组的成员。

## 使用 sudo 命令进行授权

在我看来，使用 `sudo` 系统是一种更加安全的授权管理员权限的方式。该系统通过 `sudo` 命令前缀来执行管理命令，并通过 `/etc/sudoers` 文件进行细粒度授权。

一旦用户被信任，并且任务被委派到他们的 `/etc/sudoers` 文件中，他们就可以使用 `sudo` 执行分配给他们的命令。基本命令语法如下：

```
$ sudo <command>

```

在前面的示例中，`<command>`将被替换为通常保留给 root 用户的管理命令，如以下命令所示：

```
$ sudo useradd bob

```

前面列出的命令字符串允许一个受信任的用户创建一个新的用户账户：`bob`。当第一次使用`sudo`运行命令时，系统会提示用户输入密码。系统默认缓存用户凭证 5 分钟。这样，如果他们需要在短时间内多次使用`sudo`作为 root 用户执行多个命令，他们只会被提示一次输入密码。

使用`sudo`时，我们不需要向管理员泄露 root 用户的密码，也不需要将特定命令或一组命令委托给个人或组。

要委托一个名为`sally`的用户能够运行`useradd`命令以及`passwd`命令，可以在`/etc/sudoers`文件中添加一个条目。我们还可以防止`sally`在同一条目中更改 root 密码。该条目类似于以下命令：

```
sally ALL=(root) /sbin/useradd, /bin/passwd , !/bin/passwd root

```

编辑应以`root`身份通过`visudo`命令进行。通过这种方式，在保存更改之前会先进行验证（防止文件损坏）。更多详细的配置示例可以通过查看`man`页面获得：

```
$ man sudoers

```

默认情况下，`sudo`允许`wheel`管理组的成员运行所有命令，而无需额外的管理操作。

为了提升安全性，以便在每次执行`sudo`命令时获取用户密码并覆盖默认的 5 分钟超时设置，请使用`visudo`并在`/etc/sudoers`文件中添加以下行：

```
Default    timestamp_timeout=0

```

在本书的其余部分，管理命令将作为标准用户运行，并以`sudo`命令为前缀。该用户将是`wheel`组的成员。通过这种方式，我们希望将最佳实践和安全性作为您思考的核心。

# 使用 ip 和 hostnamectl

许多 Linux 管理员习惯使用`ifconfig`命令来显示和设置 Linux 主机上的 IP 地址。尽管`ifconfig`命令仍然有效，但它已被标记为过时，推荐使用`ip`命令。对于从 Windows 迁移到 Linux 的管理员来说，使用`ifconfig`成为了显而易见的选择。由于`ipconfig`与 Windows 命令行非常相似，我鼓励您学习并掌握持续更新的`ip`命令及其功能。在 RHEL 7 上使用`ifconfig`或`ip`命令还会引入新的、统一的设备名称。这对那些习惯于`/dev/eth0`的人来说，可能会有些冲击。

最后，我们将探讨使用`hostnamectl`命令来设置 RHEL 中新颖的功能。该命令可以用于设置当前会话的`hostname`，并且一次性永久生效，而无需使用`hostname`命令并编辑`/etc/hostname`文件。

## 网络设备的一致命名

在我们服务器和桌面计算机上使用的硬件中，现在可以看到更多的多端口接口卡和**LOM**（**板载局域网**）接口。如果你依赖更传统的`eth0`和`eth1`命名方案，所有这些将导致网络设备命名不一致。

在 RHEL 7 及相关的类似发行版中，`udev`支持多种不同的网络设备命名方案。默认情况下，系统会根据固件、拓扑和设备本身返回的位置信息来分配固定名称。这样，命名与物理设备本身相关，即使更换了故障硬件，名称仍然保持一致且可预测。我们需要实现的是避免`eth0`设备变成`eth1`，反之亦然。缺点是名称可能较长，不易记住。参考我们在本书中将使用的 RHEL 7.1 系统，VMWare 托管系统上的单一以太网接口被命名为`eno16777736`。

命名方面由`systemd`（新的初始化守护进程）管理，并在启动阶段检测硬件时支持以下命名方案：

+   **方案 1**：此方案指定的名称可以包含从板载设备返回的固件或 BIOS 信息。这些名称可以采用`enoxxx`（字母`o`代表板载设备）的形式。如果失败，命名系统将回退到方案 2。

+   **方案 2**：此方案指定的名称可以包含从 PCI Express 插槽卡返回的固件或 BIOS 信息。这些名称可以采用`ensxxx`的形式。如果失败，命名系统将回退到方案 3。

+   **方案 3**：此方案指定的名称可以包含连接器的物理位置——例如主板上的插槽地址。这些名称可以采用`enpxxx`的形式。如果失败，命名系统将回退到方案 5（注意方案 4 是可选的）。

+   **方案 4**：此方案基于**MAC**（**媒体访问控制**）地址识别名称，该地址来自**NIC**（**网络接口卡**），并由管理员通过在网络配置文件中设置`HWADDR`（硬件地址）属性进行选择。这些名称采用在接口配置文件中`DEVICE`属性中提供的名称。例如，如果你想将一个 LOM 接口卡从`eno16777736`重命名为`internal`，在以 root 身份工作时，你需要编辑`/etc/sysconfig/network-scripts/ifcfg-eno16777736`文件。你需要添加`HWADDR`属性，并编辑`DEVICE`属性，使文件内容类似以下摘录：

    ```
    HWADDR="00:0c:29:57:ef:c4" #Using the MAC address for your NICDEVICE="internal"

    ```

+   **方案 5**：如果所有其他方法都失败，命名系统将回退到传统的内核不可预测命名方案，如`eth0`、`eth1`等。

总结一下，每个接口设备通常会有一个两字符的前缀。这个前缀表示 NIC 的协议类型。以下是这些前缀的示例：

+   `en`：这表示以太网

+   `wl`：这表示无线局域网

+   `ww`：这表示广域无线

前缀后面的字符表示使用的命名方案和检测到的硬件类型，如下表所示：

| 设备名称的位置 3 | 描述 |
| --- | --- |
| `o` | 这是板载设备 |
| `s` | 这是热插拔插槽 |
| `p` | 这是 PCI 或 USB 设备 |

## 一个实际的网络设备命名示例

为了展示我们迄今为止在物理机和虚拟机上使用的一致的网络设备命名系统，我们将走出我的戴尔笔记本电脑，它运行的是 Fedora 21 工作站。它有一个有线网卡（当前未连接）和一个无线端口（这是活动连接）。使用`ip address show`命令，我们可以看到两个物理接口和本地或`loopback`接口：

![一个实际的网络设备命名示例](img/image00192.jpeg)

当我们查看设备名称并忽略本地接口`lo`时，我们看到接口`2`为`enp9s0`，接口`3`为`wlp12s0`。

**对于接口 2**：

+   有线以太网是`en`

+   PCI 总线地址是`p9`

+   插槽编号是`s0`

我们可以使用`lspci`命令查看这个 PCI 设备；命令和输出如下：

```
$ lspci | grep 09:00.0
09:00.0 Ethernet controller: Broadcom Corporation NetXtreme BCM5755M Gigabit Ethernet PCI Express (rev 02)

```

我们可以看到这确实与命名方案中提到的物理设备相关（PCI 总线`9`和以太网卡中的插槽`0`）。

**对于接口 3：**

+   无线以太网是`wl`

+   PCI 总线地址是`p12`

+   插槽编号是`s0`

再次使用`lspci`和`grep`命令，我们可以看到这个设备。PCI 总线（`12`）的十六进制值在`lspci`的输出中显示为`0c`，这是因为使用了十六进制，而设备命名方案使用的是十进制值：

```
$ lspci | grep 0c:00.0
0c:00.0 Network controller: Intel Corporation PRO/Wireless 3945ABG [Golan] Network Connection (rev 02)

```

## 禁用一致的网络设备命名

为了简化，特别是当你只有单一接口时，你可以优先使用传统名称（`eth0`）。你可能也有需要此命名方案的旧版软件。这些旧版名称仍然可以使用，正如你在使用命名方案 4 时所学到的。通过向网络配置文件中添加`HWADDR`属性，并重命名`/etc/sysconfig/network-scripts/ifcfg-eth0`文件，或者将`DEVICE`名称属性配置为`eth0`，可以帮助你实现目标。

要在系统中全局设置此配置，以便所有接口都应用，您需要在启动时使用附加的内核参数。这可以通过在`/etc/default/grub`文件中设置`GRUB2`来实现。`GRUB_CMDLINE_LINUX`行应更改为以下代码，并附加`biosdevname`和`net.ifname`命令：

```
$ cat /etc/default/grub

```

![禁用一致的网络设备命名](img/image00193.jpeg)

编辑并保存文件后，我们可以使用以下命令更新`GRUB2`配置：

```
$ sudo grub2-mkconfig -o /boot/grub/grub.cfg

```

然后我们需要重启系统，以便查看接口名称的变化：

```
$ sudo shutdown -r now

```

### 提示

强烈建议坚持使用一致的名称，并接受这种命名方案，它解决了传统内核名称以前给管理员带来的设备命名不一致问题。

在本书的其余部分，我们将使用与 RHEL 7.1 系统上单个 NIC 相关的标准命名系统，即 `eno16777736`。

## 使用 ip 命令显示配置

本章开头我们提到，RHEL 7.1 中显示和配置 IP 地址的首选命令是 `ip`。`ip` 命令是 `iproute` RPM 包的一部分，取代了现在已经过时的 `ifconfig` 命令，后者属于 net-tools RPM 包。`ifconfig` 命令仍然可以使用，但建议使用 `ip`。

我们可以使用 `ip` 命令的 `address show` 选项来显示所有接口的 IP 地址。这可以通过以下三种方式之一来实现：

+   `$ ip address show`

+   `$ ip a s`

+   `$ ip a`

我们首先使用详细选项，其中使用了完整的 `address show` 命令。可以将其缩写为 `a s`，或者由于地址命令的默认动作是 `show`，只需使用 `ip a`。稍微扩展一下，我们可以仅显示单个接口或单个协议的 IP 地址，如下所示：

```
$ ip a s eno1677736
$ ip -4 a s eno1677736
$ ip -6 a s eno1677736

```

下图显示了在演示系统中查看配置的网络接口卡（NIC）的 IPv4 地址时，命令及其输出：

![使用 ip 命令显示配置](img/image00194.jpeg)

### 提示

`scope global dynamic eno16777766` 输出中第三行使用了动态一词，表示该地址是通过 **DHCP**（**动态主机配置协议**）分配的。

要查看此同一接口的传输统计信息，我们切换到 `link` 选项，如以下命令行和输出所示：

```
$ ip -s link show eno16777736

```

![使用 ip 命令显示配置](img/image00195.jpeg)

我们已经开始感受到这个命令的灵活性，但我们不仅限于 `link` 和 `address` 选项。在接下来的命令中，我们首先查看路由表，然后查看 **ARP**（**地址解析协议**）缓存。每个命令都展示了详细形式和简化形式。简化形式特别有用，如果你不会拼写 neighbor（邻居）的话：

```
$ ip route show
$ ip r
$ ip neighbor show
$ ip n

```

### 提示

ARP 缓存显示了你连接到同一网络中设备的 MAC 地址。

## 使用 ip 命令实现配置更改

作为展示配置信息的得力工具，`ip` 命令在更改 IP 地址的动态配置时也非常擅长，这次使用 `add` 来代替 `show`。例如，要向我们的接口添加一个额外的 IPv4 地址，我们将使用以下命令：

```
$ sudo ip address add 192.168.140.3/24 dev eno16777736

```

我们现在可以使用之前查看过的`show`命令来查看这些信息，如下所示的命令和输出：

```
$ ip -4 a s eno16777736

```

![使用 ip 命令实施配置更改](img/image00196.jpeg)

当我们仔细查看输出时，可以看到之前的 DHCP 地址和我们刚刚应用的附加地址。虽然这些设置仅针对本次会话添加，但在网络或接口重启时，我们将恢复为单一的 DHCP 分配地址。

要使用网络服务重启所有接口，我们将使用以下命令：

```
$ sudo systemctl restart network.service

```

如果系统中有多个接口，并且我们正在使用 NetworkManager 服务（默认接口），我们可以使用以下命令停止并启动单个接口：

```
$ sudo nmcli dev disconnect eno16777736
$ sudo nmcli con up ifname eno16777736

```

### 提示

本章稍后将有更多关于`nmcli`命令的信息。

因此，尽管我们可以动态地向运行中的系统添加 IP 地址，但如果我们希望更改永久生效，那么我们需要将配置添加到配置文件中。

## 持久化网络配置更改

要从我们在演示 RHEL 7.1 系统上使用的 DHCP 分配地址更改为静态地址，我们将在与`/etc/sysconfig/network-scripts/ifcfg-eno16777736`接口相关的网络配置文件中分配一个静态地址。要编辑文本文件，你可以使用你喜欢的文本编辑器：`vi`或`nano`；在这里，我们将使用`vi`：

```
$ sudo vi /etc/sysconfig/network-scripts/ifcfg-eno16777736

```

在前面的命令行中编辑文件后，它应类似于以下文件内容。当然，根据你的网络相关信息进行设置，而不是 IP 地址，我们使用：

```
TYPE="Ethernet"
BOOTPROTO="none" #Change from dhcp to none
DEVICE="eno16777736" #use your device name
ONBOOT="yes"
PEERDNS="yes"
PEERROUTES="yes"
IPADDR="192.168.40.3" #use the IP Address that you want to assign
NETMASK="255.255.255.0" #Use the appropriate subnet mask
DNS1="192.168.40.2" #the address or your DNS server
GATEWAY="192.168.40.2" #the default gateway to use
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
NAME="eno16777736" #use your device name
UUID="980c9e81-f018-42ae-9272-1233873f9135" #use your device UUID
IPV6_PEERDNS="yes"
IPV6_PEERROUTES="yes"
IPV6_PRIVACY="no"

```

一如既往，编辑文件时要小心。实际上，文件中的大部分内容可以保持不变，因为我们只编辑了以下一行：

```
BOOTPROTO="none"

```

添加四行新内容：

```
IPADDR="192.168.40.3" #use the IP Address that you want to assign
NETMASK="255.255.255.0" #Use the appropriate subnet mask
DNS1="192.168.40.2" #the address or your DNS server
GATEWAY="192.168.40.2" #the default gateway to use
```

在做出更改并保存后，我们需要使用以下命令刷新 NetworkManager：

```
$ sudo nmcli connection reload

```

这将重新缓存网络配置文件。完成此操作后，我们可以停止并启动以下接口：

```
$ sudo nmcli dev disconnect eno16777736
$ sudo nmcli con up ifname eno16777736

```

或者，像之前一样重新启动网络服务。这个单一命令替代了这里的三个命令。但是，它会中断所有接口。因此，以下命令应该仅在我们只有单一接口时使用：

```
$ sudo systemctl restart network.service

```

现在，我们已经为接口配置了静态 IPv4 地址，我们会发现从`ip address show`命令的输出中失去了`dynamic`关键字：

```
$ ip -4 a s eno16777736

```

我们现在已经看到如何通过命令行和配置文件成功配置 IPv4 设置。接下来，我们将进入网络配置的最后部分：主机名。

## 使用 hostnamectl 配置 RHEL 7 的主机名

随着`systemd`在 RHEL 7 及其衍生版上的出现，我们有了一种全新的方式来使用`hostnamectl`命令显示和设置主机名。这个工具的优势是可以一步配置静态名称和临时名称。

我们将编辑 `/etc/hostname` 文件，并添加新的静态主机名。然后，内核在系统启动时读取它，并显示为瞬时主机名，这经常作为您的 `BASH` shell 提示的一部分使用。可以使用 `hostname` 命令显示和设置瞬时主机名。这是一个两部分过程：使用 `hostname` 设置内核维护的瞬时名称，并编辑 `/etc/hostname` 文件以确保它在重新启动后保持不变。

在 `RHEL 7` 中，我们有这两个主机名和第三个主机名：漂亮名称。漂亮名称可以显示 `UTF-8` 字符，允许您嵌入空格和撇号。设置漂亮名称时，它将存储在 `/etc/machine-info` 文件中。

要显示配置的主机名，可以使用 `hostnamectl` 命令。如果配置的主机名包含不能构成静态主机名的字符，则只会显示漂亮名称。同样，如果使用漂亮名称存储与 `/etc/hostname` 文件不兼容的名称，则 `/etc/machine-info` 文件将存在。

要以标准用户身份显示主机名，可以发出以下命令：

```
$ hostnamectl

```

![使用 hostnamectl 配置 RHEL 7 主机名](img/image00197.jpeg)

如果主机名中包含空格，则 **漂亮主机名** 将显示出来。**漂亮** 和 **静态** 名称与分别对应 `/etc/machine-info` 和 `/etc/hostname` 文件，并可用于以下命令：

```
$ cat /etc/machine-info /etc/hostname

```

上述命令行的输出结果如下：

```
PRETTY_HOSTNAME="Red Hat 7-1.tup.com"
redhat7-1.tup.com

```

要使用 `hostnamectl` 配置主机名，我们使用 `set-name` 选项，如以下命令所示。如果用户是 `wheel` 管理组的成员，则此命令无需以 `sudo` 为前缀，但用户将被提示输入密码。这些权限是使用策略工具包配置的：

```
$ hostnamectl set-hostname "Red Hat 7.tup.com"

```

这将设置所有三个名称；要查看瞬时名称，应通过运行 `bash` 命令初始化新的 shell。要设置单个名称，请按照以下 `hostnamectl` 命令正确使用选项：

```
--transient
--static
--pretty

```

# 简介 Red Hat NetworkManager

自 `RHEL 6` 版本起，`NetworkManager` 服务已成为 `RHEL` 的一部分，最简单的形式允许用户配置网络配置设置（如加入 Wi-Fi 网络）。当然，考虑到使用 Fedora 或 RHEL 笔记本电脑的用户，这真的是非常必要的。此服务远远超出了 GUI，适用于安装有或没有 X 服务器环境的服务器产品。

随 RHEL 7 发布的 `NetworkManager` 服务是一个动态网络控制与配置守护进程，旨在保持网络接口在可用时处于活动状态。正如我们所看到的，`NetworkManager` 服务不仅维护对传统 `ifcfg-` 文件类型的支持，还扩展了对其他配置文件的支持。通过这种方式，我们可以轻松为你的笔记本配置静态 IP 地址，以适应你可能访问的不同办公室，而不必依赖每个站点的 DHCP。

`NetworkManager` 服务的配置可以通过 GUI 控制中心或 `nmtui` 命令行菜单进行维护。我们还看到，我们可以避免使用菜单，通过命令行启用脚本事件，使用 `nmcli` 命令。

要查询 `NetworkManager` 服务的状态，我们可以使用 `systemctl` 工具，如下所示的命令及相关输出截图：

```
$ sudo systemctl status NetworkManager.service

```

![Red Hat NetworkManager 介绍](img/image00198.jpeg)

用户和管理员可以通过以下工具与 NetworkManager 服务进行交互：

+   GNOME 通知区域图标

+   GNOME 网络设置控制中心

+   `nmtui` 菜单

+   `nmcli` 命令行工具

# 使用控制中心与 NetworkManager 进行交互

如果你在图形化环境下使用 RHEL、CentOS 或 Fedora，那么通过 GNOME 控制中心，我们可以与 NetworkManager 服务进行交互。我们也可以通过通知区域图标访问网络设置。这可以在下面的 RHEL 7.1 系统截图中看到：

![使用控制中心与 NetworkManager 进行交互](img/image00199.jpeg)

要通过控制中心访问相同内容，我们可以使用 **SUPER** 键。在搜索对话框中输入 `control network`，如下所示的截图：

![使用控制中心与 NetworkManager 进行交互](img/image00200.jpeg)

一旦我们进入 **网络设置**，就可以通过传统的 **飞行模式** 简单地禁用所有无线接口。通过这种方式，你可以确保在起飞和降落时不会因网络问题而面临生命危险，同时还能享受 *糖果传奇* 游戏的乐趣。

在左侧面板中，我们可以查看当前已知的接口以及 **网络代理** 设置。在这里，如果需要，可以添加 Web 代理。在本书使用的 RHEL 7.1 系统中，我们看到左侧面板有两个网络接口组：

+   **有线**

+   **未知**

在图示中，**有线**接口代表我的千兆以太网卡，而**未知**接口代表本地回环连接。如果您的系统包括无线网卡，您可能还会看到**Wi-Fi**作为选项。在左侧面板中选择**有线**接口时，右侧面板将显示当前的网络配置文件。由于我们只有一个配置文件，因此该配置文件的名称未显示，但它代表我们在本章中先前配置的默认系统配置文件：`eno16777736`。

在右侧面板的底部，我们可以通过**添加配置文件**按钮创建额外的配置文件，而右下角的齿轮图标将允许您更改当前配置文件的属性。以下截图展示了这一切：

![通过控制中心与 NetworkManager 互动](img/image00201.jpeg)

# 通过控制中心添加新配置文件

对于移动系统，如笔记本电脑和平板电脑，我们可以配置配置文件，以便轻松加载与您使用设备的地点相关的网络配置信息。例如，如果您在家使用笔记本电脑，您可能会设置一个特定的静态 IP 地址，而在工作时，您可能会使用 DHCP 分配的地址。配置文件可以轻松且无缝地处理这种情况。

使用**添加配置文件**按钮从网络设置控制中心打开**新配置文件**对话框。在左侧面板中，我们可以从以下给定选项中选择一个：

+   **安全性**

+   **身份**

+   **IPv4**

+   **IPv6**

我们将为家庭网络创建一个新的 DHCP 配置文件；如果您记得，我们在之前的章节中使用位于`/etc/sysconfig/network-scripts`目录下的传统`ifcfg-脚本`设置了静态 IPv4 地址。我们将保留此设置，并允许我们根据需要在静态地址和 DHCP 之间切换。

从左侧面板选择**身份**选项，我们将**名称**设置为`home-DHCP`。从下拉列表中选择与我们希望分配给此配置文件的接口相关的**MAC 地址**。最后，我们可以取消选中**自动连接**复选框，这样默认连接将仍然是我们之前选择的静态分配。我们可以根据需要手动选择此配置文件。其他所有设置保持不变，包括**IPv4**和**IPv6**设置中的自动 DHCP 地址分配。导航到对话框右下角并选择**添加**按钮来创建配置文件。以下截图展示了我们选择的设置：

![通过控制中心添加新配置文件](img/image00202.jpeg)

创建新配置文件后，我们可以轻松在两个配置文件之间进行选择，利用 GNOME 通知面板，简化了不同网络间的切换。以下截图显示了当前选中的**eno16777736**配置文件，以及如何切换到新创建的**home-DHCP**配置文件：

![通过控制中心添加新配置文件](img/image00203.jpeg)

现在，我们已经看到如何使用图形工具在 RHEL 7.1 上设置网络配置文件信息。对于没有 X 服务器的 RHEL 或 Fedora 系统，用户可以通过`nmtui` ncurses 菜单轻松管理`NetworkManager`连接。

# 使用 nmtui 与 NetworkManager 交互

就像在 GNOME 控制中心中的 GUI 配置文件管理一样，我们也可以使用`nmtui`命令提供的文本用户界面。这是由 ncurses 系统提供的传统蓝色屏幕命令行菜单。如果系统中没有该命令，可以通过`yum`进行安装，以下命令演示了安装方式：

```
$ sudo yum install NetworkManager-tui

```

安装完成后，可以使用以下命令访问 NetworkManager 菜单：

```
$ sudo nmtui

```

如果你通过 PuTTY 使用 SSH 连接到服务器，为了确保菜单边框显示正确，你应该将**字符集翻译**选项设置为**UTF-8**。这个设置可以在连接设置中的**窗口**|**翻译**找到。

本书中使用的 RHEL 7.1 系统显示的**NetworkManager**菜单看起来干净简洁，虽然有点简单，以下截图展示了该菜单：

![使用 nmtui 与 NetworkManager 交互](img/image00204.jpeg)

`nmtui`命令还提供了快捷方式，能快速执行菜单中的特定任务。这些快捷方式包括`nmtui-edit`、`nmtui-connect`和`nmtui-hostname`命令。前两个命令适用于你已经知道要激活或编辑的连接配置文件名称，而最后一个命令则用于全局设置主机名。

要激活我们之前创建的 home-DHCP 配置文件，我们将发出以下命令：

```
$ nmtui-connect home-DHCP

```

这将有效地将静态 IP 地址切换为自动分配的 DHCP 地址。你应该从控制台发出此命令，而不是远程操作，因为地址从静态切换到 DHCP 后，你的连接会丢失。

如果你足够“极客”，并在星巴克使用命令行版 Fedora，也可以使用这个命令来连接到新的 Wi-Fi SSID：

```
$ nmtui-connect Coffee-Shop-Wifi

```

要更改相同连接配置文件的属性，我们将使用以下命令：

```
$ sudo nmtui-edit home-DHCP

```

这将打开 homeDHCP 连接配置文件的属性页面，准备进行编辑。如果你想打开主机名菜单页面以进行编辑，可以使用以下命令：

```
$ sudo nmtui-hostname

```

# 使用 nmcli 与 NetworkManager 进行高级交互

对于那些认为 Linux 唯一的真正形式是没有菜单辅助的，只能依靠通过你那位绝地父母传授的智慧的人，我们为你们准备了极限运动 `nmcli`。开个玩笑，使用这种不依赖菜单互动的方式与 `NetworkManager` 配合工作，将让你能够在脚本中做出更改，进而可以在多个系统中实施。

作为简单的入门，我们可以使用 `nmcli` 扫描可用的 Wi-Fi 网络；输出应显示 Wi-Fi SSID 和信号强度，如下所示：

```
$ nmcli device wifi list

```

与我们之前使用 `iw` 命令显示 SSID 的传统命令行机制相比，这个过程大大简化了：

```
$ sudo iw wlp12s0 scan | grep SSID
 SSID: hobbit
 SSID: virginmedia1671684
 SSID: VM260970-2G
 SSID: virginmedia9066074
 SSID: Edinburgh2013
 SSID: TALKTALK-4C89F0

```

使用 `nmcli` 的过程对我们来说简化了，因为 `NetworkManager` 可以使用已配置的 `polkit` 权限。这些权限或操作（使用 `polkit` 语言）由系统管理员配置，并不打算由用户更改。策略文件位于 `/usr/share/polkit-1/actions/org.freedesktop.NetworkManager.policy` 位置。

我们可以使用 `nmcli` 显示已配置的权限，命令如下：

```
$ nmcli general permissions

```

![使用 nmcli 与 NetworkManager 进行极限互动](img/image00205.jpeg)

如果我们希望能够在有线接口启动并可用时创建一个连接，我们可以通过 `nmcli` 来实现。这个过程也可以根据需要轻松地在多个设备上编写脚本。首先，我们创建连接配置文件，如下所示命令和输出：

```
$ sudo nmcli connection add con-name wired-home \
 ifname enp9s0 type ethernet ip4 192.168.0.8 gw4 192.168.0.1

 Connection 'wired-home' (e17cb6b7-685f-4cf2-9e8b-16cbfae1f73a) successfully added.

```

当你知道启用了自动完成，甚至是对于我们作为 `ifname` 值添加的子命令和值（如 `enp9s0`）也会自动完成时，这个命令就大大简化了。

为了完成任务，我们需要向连接配置文件添加 DNS 配置，这可以通过以下命令实现：

```
$ sudo nmcli connection modify wired-home ipv4.dns "192.168.0.3 8.8.8.8"

```

我们现在可以通过以下命令显示属性：

```
$ nmcli -p connection show wired-home

```

这里使用的 `-p` 选项是为了得到漂亮的输出；对于简洁的输出，可以使用 `-t`。无论如何，输出都太冗长，无法作为书中的一部分展示。

我们现在已经复制了创建连接配置文件的过程，这是我们在使用控制中心时第一次看到的。我们并不认为这很简单，但能够通过脚本完成这一过程提供了许多不可通过任何形式的交互式菜单（无论是 `nmtui` 的文本菜单还是控制中心的 GUI）实现的选项。

# 总结

在这一章中，我们实际上已经建立了理解 RHEL 7 系列网络的基础知识。首先，你学会了如何使用 `su` 和 `sudo` 在 RHEL 上获取和管理权限。此外，我们还看了如何通过 PAM 限制 `su` 的使用，仅限于 `wheel` 组成员。我们也开始了我们将继续采用的管理方式，使用 `sudo` 来管理管理任务，而不是登录为 root 或使用 `su`。

在建立了正确设置的基础知识后，我们开始理解 Red Hat 发行版上网络设备的新命名约定。在转向网络配置之前，我们了解了相对传统命名更受欢迎的原因。

要配置网络接口，我们可以使用传统的 `ifcfg-` 脚本，默认情况下会使用这些脚本。我们可以扩展到其他网络配置文件，这对于连接到不同网络位置的移动设备（如笔记本电脑）可能是最有用的。我们看到可以通过从菜单到原始命令行工具的多种方式来配置这些内容。

接下来，我们将讨论如何配置关键网络服务，如 DNS、DHCP 和 SMTP。
