# 第七章：使用 Samba 4 实现 Windows 共享

您的 Linux 设备几乎肯定不会独立运行，其他操作系统将与它们共存。无论您的基础设施位于何处，您很可能至少需要与 Windows 系统进行互操作。这在家庭环境中和企业中都同样成立。考虑家庭市场，您认识多少人使用 Windows 桌面并且拥有一个 Linux 服务器作为中央文件存储。请记住，Linux 服务器可能嵌入在**网络附加存储**（**NAS**）中，这是您从街头购买的设备。在大大小小的企业中，Microsoft 的 Active Directory 是一个非常普遍的身份存储，它跨多个系统共享用户帐户。

为了帮助将 RHEL 7 集成到 Windows 环境中，本章将为您提供以下主题的基础知识：

+   Samba 和 Samba 服务概述

+   实验室环境概述

+   配置时间和 DNS

+   管理 Samba 服务

+   RHEL 7 上的 Samba 客户端

+   在 Samba 上配置文件共享

+   故障排除 Samba

# Samba 和 Samba 服务概述

在调查我们的主要服务并使用 Samba 时，我们需要安装 `samba` 软件包；可以通过以下方式安装此软件包：

```
$ sudo yum install -y samba

```

安装软件包后，以下服务将被添加到我们的系统中：

+   对于文件和打印共享，我们有 `smbd` 服务。对于此服务，我们需要打开 TCP 端口 `139` 和 `445`。

+   如果我们需要响应旧版 NetBIOS 名称请求，我们将需要启动 `nmbd` 服务。对于较旧的客户端，如 Windows 95、98 和 ME，我们需要此服务。Windows 2000 和 XP 客户端也可以使用此网络浏览协议。如果我们需要此服务，虽然我对此持怀疑态度，我们将需要打开 UDP 端口 `137`。

如果您想加入 Windows 域，那么可以添加 `samba-winbind` 或 `sssd-common` 等客户端工具包。

如果您使用 `firewall-cmd` 命令，添加 Samba 服务将为您启用三个端口。要将 Samba 服务添加到防火墙中，我们将使用以下命令：

```
$ sudo firewall-cmd --permanent --add-service=samba
$ sudo firewall-cmd --reload
$ sudo firewall-cmd --list-services

```

最后一条命令仅供参考，并非设置防火墙规则所必需。

# 实验室环境概述

在本章的演示中，我们将使用两台虚拟机，这些虚拟机运行在**Oracle VirtualBox**虚拟化环境中。

我们有一个 Microsoft Server 2008R2 活动目录域控制器，IP 地址为 `192.168.0.252`，以及一个 RHEL 7.1 主机，IP 地址为 `192.168.0.69`。

# 配置时间和 DNS

尽管如果我们使用 RHEL 主机进行简单的文件和打印共享，获取准确的时间和 DNS 并不是太大的问题；然而，我们很可能需要将 RHEL 服务器加入到 Active Directory 域，以便能够使用单点登录。用户将能够使用与 Active Directory 中相同的凭据访问他们的共享，而无需在 RHEL 服务器上创建用户账户和密码。

在本章中，我们将仅关注文件共享，而在第八章中，我们将服务器加入到 AD 域。

在第三章，*配置关键网络服务*中，我们配置了时间服务。现在，让我们来看看如何将`chronyd`时间源设置为 Active Directory 时间服务器。如果我们使用 NTP，则将 NTP 时间源设置为 Active Directory 服务器。或者，确保 Active Directory 时间源与 RHEL 主机使用的时间源相同。

要在 Windows 2008R2 Active Directory 服务器上配置时间源，你需要打开管理员命令提示符，并键入以下命令：

```
c:\> net stop w32time
c:\> w32tm /config /syncfromflags:manual /manualpeerlist:"uk.pool.ntp.org"
c:\> w32tm /config /reliable:yes
c:\> net start w32time
c:\> w32tm /config /update
c:\> w32tm /resync
c:\> w32tm /query /status

```

最终状态子命令的输出如下所示：

![配置时间和 DNS](img/image00274.jpeg)

我觉得在网络上拥有准确的时间是必须的，无论你是否打算加入一个域。

要加入 Active Directory 域，我们必须能够解析定位域控制器的服务记录。实现这一点的最简单方法是将 RHEL 的 DNS 解析器指向托管 DNS 的 Active Directory 服务器。我只有一个域控制器，并且它也是 DNS 服务器。为了确保我们能够正确解析这些名称，我们在 RHEL 7 上写入接口配置文件。我的系统中，这些文件如下：

```
/etc/sysconfig/network-scripts/ifcfg-eno1677736
/etc/sysconfig/network-scripts/ifcfg-eno33554992

```

确保 DNS1 指向 Active Directory 域控制器，并且在两个文件中都设置`PEERDNS=yes`。或者，配置`/etc/resolv.conf`文件，设置一个名称服务器，并确保在所有接口文件中设置`PEERDNS=no`属性。

以下截图显示了仅在接口文件中进行设置的第一个配置选项：

![配置时间和 DNS](img/image00275.jpeg)

在这两个接口文件中设置此项（如果你有两个网卡 NICs），你可以简单地重启`NetworkManager`服务，方法如下：

```
$ sudo systemctl restart NetworkManager

```

要测试配置，你可以尝试解析托管在 DNS 中的 Active Directory 域的名称服务器。对于本书，我们简单使用`example.com`。要从 RHEL 7 主机解析此域，我们运行以下命令：

```
$ dig -t ns example.com

```

`dig`命令查找`example.com`域的`ns`类型或名称服务器记录。输出应类似于以下截图：

![配置时间和 DNS](img/image00276.jpeg)

# 管理 Samba 服务

要访问 Samba 资源，用户需要一个可用的 POSIX（Linux）用户账户和一个 Samba 账户。POSIX 账户可以是`/etc/passwd`文件中的普通账户，或者该账户可以集中在 LDAP 或 Active Directory 中。当一个 POSIX 账户启用 Samba 时，会向用户账户添加 Windows 系统所需的其他属性。要为现有的 POSIX 账户启用 Samba，我们可以使用`/bin/pdbedit`命令。它可以与以下账户存储中的 Samba 账户配合使用：

+   `/etc/samba/smbpasswd` 文件

+   位于`/var/lib/samba/private/passdb.tdb`的`tdbsam`数据库（这是默认的 Samba 账户存储）

+   OpenLDAP 目录服务

由于现有的域账户已经具备了 Samba 所需的属性，因此无需为这些账户启用 Samba。

首先，我们将列出所有已启用的 Samba 账户。当然，我们只安装了 Samba 并未启用任何其他账户。此外，由于当前 RHEL 服务器并未加入域或 LDAP，因此我们只在`/etc/passwd`文件中拥有本地账户：

```
$ sudo pdbedit -L

```

该命令应无输出，因为我们没有任何启用 Samba 的账户。如果您以标准用户身份运行命令而没有错误，则在尝试访问数据库时会看到权限违规。

我们现在将为`root`和标准用户`andrew`启用现有账户。我们不仅会创建一个账户并为 Samba 启用它，还会添加存储在分配的 Samba 账户后端中的属性。在默认情况下，这是`tdbsam`数据库。要启用这两个账户，请使用以下命令：

```
$ sudo pdbedit -a -u root
$ sudo pdbedit -a -u andrew

```

系统会提示为每个账户设置一个新的 Samba 密码。理想情况下，这个密码应该与其 POSIX 密码不同。启用`andrew`的 Samba 账户后的截断输出显示在以下命令行截图中：

![管理 Samba 服务](img/image00277.jpeg)

神奇的是，当我们运行账户列表时，我们将能够看到这两个账户：

```
$ sudo pdbedit -L

```

输出应与以下截图相似，并将账户名称调整为与您自己的账户相符：

![管理 Samba 服务](img/image00278.jpeg)

您可以使用`-L`选项结合`-v`，以获取类似于启用账户时所看到的详细信息。

Samba 的主要配置文件是`/etc/samba/smb.conf`。该文件被划分为多个部分，每个部分（除了[global]）定义了一些形式的共享资源。每个部分通过方括号 `[]` 中的部分名称来表示。除了`[global]`部分外，还有两个其他特殊部分。这些部分在以下列表中定义：

+   `[global]`：在全局部分设置的属性指的是整个 Samba 服务器，而不是特定的共享资源。

+   `[homes]`：此部分的存在允许用户轻松连接到他们自己的主目录，而无需额外的管理操作来共享用户的主目录。这可以在用户名和账户名与主目录名不完全相同的情况下正常工作。用户 `andrew` 将通过类似以下命令的方式连接到他的主目录共享：

    ```
    //<server-name or IP>/andrew

    ```

    用户账户的主目录属性将被读取，然后用户将连接到分配给他们的主目录。

+   `[printers]`：此部分的存在使得本地打印机可以自动共享，无需进一步的管理操作。用户将使用以下 URL 连接到打印机：

    ```
    //<server-name or IP>/<printer-name>

    ```

你会发现 `/etc/samba/smb.conf` 文件中有很多注释，部分注释使用 `#` 符号，另一些则使用分号（`;`）符号。这两者都是有效的注释，但这种不一致性有些烦人。

如果你想创建一个备份并删除空行和注释行，可以尝试从 `/etc/samba` 目录运行以下命令：

```
$ sudo sed -i.bak  '/^\s*[;#]/d;/^$/d;' smb.conf

```

我们用来搜索要删除的行的正则表达式比通常的稍微复杂一些，但它仍然能为我们做出惊人的效果。示例中有两个 `sed` 表达式，`/^\s*[;#]/d`。

上述表达式将删除文件中的注释行。

我们寻找包含（`^`）和任何空白字符（`\s`）的行。此外，使用 `*` 来检查零个或多个空白字符。通过这种方式，我们允许行首有一个可选的空白字符，但它必须紧跟着 `;` 或 `#`（`[;#]`）符号。用简单的英语来说，这扩展为查找被注释的行，不管它们是以空格/制表符开始，还是直接以注释开始。搜索字符串在两个 `/` 字符之间，并且 `d` 命令用于删除匹配的行：

```
/^$/d

```

上述表达式删除文件中的空行或空白行。

其效果是将文件的行数从 320 行减少到 20 行，仅需几秒钟。我们保留原始文件作为备份，即 `smb.conf.bak`。

一个简单的 `[global]` 部分可能看起来像下面的截图：

![管理 Samba 服务](img/image00279.jpeg)

这些设置的详细信息如下：

+   `workgroup = MYGROUP`：此项表示要加入的 NetBIOS 工作组，在网络邻居视图中显示。

+   `server string = Samba Server Version %v`：这将在网络视图中的服务器名称旁边显示描述。`%v` 变量将显示 Samba 版本；在我们的例子中，版本是 `4.1.2`。

+   `log file = /var/log/samba/log.%m`：此项指定日志文件的路径；`%m` 变量代表机器名（主机名）。

+   `max log size = 50`：此项指定日志文件在轮换之前的最大大小（以 KB 为单位）。

+   `idmap config * : backend = tdb`：这是映射机制，用于将 POSIX 用户 ID 和组 ID 映射到 **SIDs**（**安全标识符**）。

+   `cups options = raw`：这些是打印机设置，告诉 cupsCUPS 进程直接将作业发送到打印机，而不是尝试解释它。它已经由 Windows 打印机驱动程序处理过，不需要进一步处理。

您还可以使用指令控制哪些主机或网络可以访问系统，使用`hosts allow`和`hosts deny`指令。例如，以下`[global]`部分中的属性设置将仅允许给定网络访问任何共享。`127.0.0.1`主机将始终具有访问权限，除非被显式拒绝。以下任何一种方法都正确，允许访问`192.168.0.0/24`：

```
hosts allow = 192.168.0.

```

或者：

```
hosts allow = 192.168.0.0/255.255.255.0

```

这些`host allow`设置及其兄弟`host deny`设置也可以在共享级别进行管理，但`[global]`设置将优先于共享级别的任何配置。

如果对正在运行的配置进行更改，则可以运行预检查，测试我们更改的完整性，然后再重新启动服务。为此，我们使用`testparm`命令。我们仅需以 root 身份运行，如下所示：

```
$ sudo testparm

```

`nmb`和`smb`服务是独立管理的。我们可以使用以下命令启动并启用 Samba 文件服务器：

```
$ sudo systemctl enable smb
$ sudo systemctl start smb

```

由于`[homes]`部分是默认配置，我们现在可以开始测试系统。

# RHEL 7 上的 Samba 客户端

目前，我的防火墙服务已禁用，因此我不需要担心防火墙；但是，我只需要添加 TCP 端口`445`和`139`，或者`samba`服务。

如果我们安装了 Samba 客户端包，就可以列出给定用户可以访问的所有共享。以下是该命令的输出示例：

```
$ sudo yum install -y samba-client
$ smbclient -U andrew -L //localhost

```

一旦安装了 Samba 客户端，我们可以使用它以`andrew`身份登录；系统会提示输入密码，并列出本地主机上的共享。我们应该看到来自`[homes]`特殊共享部分的列出的主目录。默认情况下，始终会有此共享。我们将看到预期的输出，而不是共享名称`andrew`，如下截图所示：

![RHEL 7 上的 Samba 客户端](img/image00280.jpeg)

当然，我们可以在此处停留，并隐瞒当前设置中存在一些 SELinux 陷阱的事实。我们可以连接到一个共享，但 SELinux 会阻止访问用户的主目录。虽然第十章，*使用 SELinux 保护系统*，将详细讲解 SELinux，但我们可以通过简单的布尔值更改，快速且安全地访问共享。首先，我们将测试当前配置。SELinux 处于强制模式，防火墙未运行。为了挂载 Samba 共享，我们可以使用以下命令，以单行形式编写：

```
$ sudo mount -t cifs -o username=andrew,password=Password1 //192.168.0.69/andrew /mnt

```

我们不需要在挂载选项中包含密码；如果不包含密码，系统会在挂载过程中提示输入用户的 Samba 密码。这应该能成功，但如果我们尝试访问挂载目标（在本例中是 `/mnt`），则会被拒绝访问。为了解决这个问题，我们可以查询 SELinux 配置中的 Samba 用户主目录设置。以下命令演示了如何实现这一点：

```
$ getsebool samba_enable_home_dirs

```

在 Samba 中，默认配置设置为 `off`，并且设置为 `as`，这会阻止对 `home` 目录的访问。我们使用以下命令启用 Samba 服务：

```
$ sudo setsebool -P samba_enable_home_dirs on

```

启用 Boolean 后，我们即可立即访问共享。无需卸载并重新挂载用户主目录。`-P` 选项使该更改成为永久性更改，这样我们可以确保该更改一直生效，直到我们需要禁用该设置，且系统仍然在 SELinux 的保护下。

要查看 SELinux、Samba Booleans 及其设置列表，可以使用以下命令：

```
$ getsebool -a | grep samba

```

当然，只要我们拥有正确的凭证，也可以通过 Windows 设备连接到共享。以下截图展示了来自 Windows 2008R2 服务器映射的驱动器，连接到 Andrew 的主目录：

![RHEL 7 上的 Samba 客户端](img/image00281.jpeg)

# 在 Samba 中配置文件共享

我们已经看到了 Samba 使文件共享的主要功能，尤其是对用户主目录的支持。虽然此功能默认已启用，但我们仍然需要通过向 `smb.conf` 添加自己的配置段来创建我们自己的文件共享。在 Red Hat 服务器上，我们有一个 `/data` 目录，之前在 第五章，*实施 btrfs* 中使用过。如果我们需要将此目录共享给 Windows 客户端，那么我们就需要使用 Samba 工具。

我们可以以 root 权限编辑 `smb.conf` 文件并添加新的配置段。我们在该段中使用的属性控制共享的访问和使用权限。至少，我们需要为共享定义路径属性，才能使其有意义。有关完整的选项列表，可以参考 `smb.conf` 文件的 `man` 页面。以下截图展示了我们为服务器上的 `/data` 目录添加的共享定义：

![在 Samba 中配置文件共享](img/image00282.jpeg)

读取列表限制共享数据的访问，仅允许列出的用户读取；在本例中，我们添加了一个组。`@` 符号表示列表中的一个组。

我们还需要通过更改 `/data` 目录及其内容的 SELinux 上下文来进一步处理 SELinux。我们可以使用 `chcon` 命令来修改上下文：

```
$ sudo chcon -R -t samba_share_t /data

```

不要忘记使用 `testparm` 测试配置；如果一切正常，我们可以按照如下方式重新启动 `smb` 服务：

```
$ sudo systemctl restart sm
b

```

现在，我们可以浏览并访问我们定义的 Samba 共享。使用 Windows 2008R2 服务器，我们现在可以浏览网络，并看到 `andrew share` 和 `data share` 的 `home` 目录。以下截图展示了这一点：

![在 Samba 中配置文件共享](img/image00283.jpeg)

我们可以理解为什么 Samba 在小型和大型环境中都是如此重要的工具；文件共享如此简单，并且能够融入我们现有的基础设施中。

# 故障排除 Samba

如果我们遇到 Samba 问题，随时可以再次检查 `testparm` 命令的输出，确保没有遗漏任何重要的设置。我们可以从 `smb.conf` 的手册页中检查哪些设置是有效的，以便关注一些特定的配置项。

我们还可以检查日志文件，这些文件位于 `/var/log/samba/` 目录中。

会有代表客户端访问的日志，具体命令如下所示：

```
log.<client ip-address>

```

或者：

```
log.<client-hostname>

```

还会有 `log.smbd` 守护进程日志。

如果需要更多关于守护进程日志的详细信息，可以在 `smb.conf` 的 `[global]` 部分设置日志级别属性，具体如下所示`:`  

```
log level = 3

```

这将增加日志记录的详细程度，可能有助于排查问题。我们建议不要长时间将日志级别设置得这么高，完成后应删除该设置。日志级别可以从 `0` 设置到 `10`，其中 `0` 表示低级别，`10` 表示高级别，但 `3` 级别对于大多数人来说已经足够详细。

# 概述

在本章中，我们探讨了如何在不禁用 SELinux 的情况下与 Windows 客户共享文件系统。我希望你能理解 SELinux 的重要性，并且一旦掌握其基础，便能轻松保持 SELinux 的启用状态。

尽管我们在本章中没有实现防火墙，但我们再次审查了 `firewalld` 的设置以使其生效。有关使用 `firewalld` 进行防火墙配置的内容，请参阅第十一章，*使用 firewalld 进行网络安全*。

在下一章中，我们将探讨如何将 RHEL 7 集成到 Windows Active Directory 域中。
