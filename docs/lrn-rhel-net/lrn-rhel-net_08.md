# 第八章：将 RHEL 7 集成到 Microsoft Active Directory 域中

在上一章中，我们讨论了如何将资源共享给微软的客户端。现在，在真正的共生精神下，我们将看到 RHEL 如何利用 Active Directory 用户和组，将域作为身份存储。如果用户可以访问 RHEL 服务器的控制台，那么仅凭他们的 Active Directory 凭据，他们就可以访问 RHEL。这不仅简化了控制台的访问，还简化了访问 RHEL 7 Samba 服务器上任何共享文件夹的过程。

我们将按照这样的结构来安排这一章内容，让你在深入了解如何使这些简单工具正常工作之前，先看到 Active Directory 集成所能提供的各项功能和优势（好东西）。

在本章中，我们将讨论以下主题：

+   身份管理概述

+   实验环境概述

+   准备加入 Active Directory 域

+   使用领域（realm）管理域的注册

+   使用 Active Directory 凭据登录 RHEL 7

+   使用 `adcli` 进行用户和组管理

+   使用 `sudo` 委派 Active Directory 账户

+   离开域

+   将 Active Directory 作为 `sssd` 的身份提供者

# 身份管理概述

为了开启我们的盛宴，我们将首先聚焦于身份管理在企业中的重要性。如果没有使用某种身份存储或保险库来集中管理用户账户，这些账户将需要被复制，因为它们需要被其他系统访问。正如你能想象的那样，随着大量用户账户的创建，每个系统上都会形成独立的账户孤岛，这些账户很容易失控。然而，我们不必太担心这些账户的创建和管理，除了这点外，它本身并不会构成安全问题。如果某个用户无法访问某个资源，他们很快就会通知你。账户孤岛的问题在于当用户离职时会发生什么；你是否认为每个离开组织的用户的账户都会被删除或（至少）禁用。无论系统多么完善，总会有一些账户未被处理，从而引发安全问题。良好的身份管理，确保每个用户只有一个账户，可以解决管理负担，更重要的是，解决安全漏洞。

当然，较小的问题与这些账户的管理有关，例如密码更改，以及随着时间的推移，可能需要更改名称。理想情况下，组织中的每个用户应该只有一个身份，即他们用于访问任何具有权限的资源的唯一凭据集。这可以通过某种形式的中央目录服务来实现，作为身份库。这可能是 Active Directory，但也可以是其他形式的**LDAP**（**轻量级目录访问协议**）服务器。在小型到中型环境中，Active Directory 可能就足够了，但随着组织的发展，身份库的规模不断扩大，可能需要为用户创建一个完全独立的目录。中央用户存储可以同步更改到其他连接的系统。

微软有其身份管理套件来围绕 Active Directory 构建，Red Hat 也有其身份管理目录服务器。本章将重点介绍并直接将 RHEL 7 集成到单一域 Active Directory 环境中。

# 实验环境概述

本章的演示将使用两台虚拟机运行在**Oracle VirtualBox**虚拟化环境中。

我们有一台 Microsoft Server 2008R2 的 Active Directory 域控制器，IP 地址为 `192.168.0.252`，以及一台 IP 地址为 `192.168.0.69` 的 RHEL 7.1 主机。这与我们在第七章，*使用 Samba 4 实现 Windows 共享*中使用的设置相同；我们已经将时间和 DNS 配置为相同方式。如果你没有完成第七章，*使用 Samba 4 实现 Windows 共享*，请确保你已经将 RHEL 服务器配置为使用域控制器进行时间和名称解析。

# 准备加入 Active Directory 域

从我们在第七章，*使用 Samba 4 实现 Windows 共享*中所见，使用 Samba 共享文件，我们可以理解这是非常了不起的技术。我们始终需要提醒自己，这一切都没有任何价格标签，也不需要客户端访问许可证。

### 提示

Samba 文件共享是免费的，即*无成本*且自由的；你可以按照自己的意愿使用它。这是开源软件的基本前提，也是 Linux 的核心所在。

可能成为潜在障碍的一个大问题是需要在 RHEL 服务器和工作站所属的 AD 域中维护用户账户。如果我们实现多个服务器，这个问题会因为需要在每一台服务器及 AD 域上都有账户而更加复杂。简单的解决方案是将 RHEL 服务器并入 AD 域，并使用 AD 账户进行资源访问。这样，我们可以通过单一的登录方式访问 Active Directory，并访问 RHEL Samba 服务器上的共享资源。

### 提示

如果 Active Directory 未设置，可以通过在 RHEL 上安装 openLDAP 服务器来建立集中账户共享。这样，某一台 RHEL 服务器可以作为身份库，将账户共享给其他服务器上的 LDAP 客户端。

无论是 Samba 文件共享，你的 Active Directory 用户可能还需要通过 SSH 或其他机制访问 RHEL 服务器。为此，他们需要在每一台 RHEL 服务器上定义账户。将 RHEL 服务器加入 AD 域后，可以在登录任何成员服务器时使用用户的 AD 账户，这些成员服务器包括 RHEL 服务器或桌面。此外，还可以通过 `/etc/sudoers` 文件和正常的文件权限机制将权限委派给这些账户。

在加入 AD 域之前，我们需要确保已设置时间服务和 DNS，具体步骤详见 第七章，*实现 Windows 共享与 Samba 4*。在这些基础设施服务就绪后，我们还需要在 RHEL 服务器上安装以下软件包：

+   `realmd`：用于管理加入和隶属 Active Directory 域

+   `samba`：表示 Samba 服务

+   `samba-common`：表示用于服务器和客户端的共享工具

+   `oddjob`：这是一个运行客户端偶尔任务的 D-bus 服务

+   `oddjob-mkhomedir`：与 odd job 服务一起使用，为 AD 账户创建主目录（如果需要的话）

+   `sssd`：系统安全服务守护进程，可根据需要转发客户端认证

+   `adcli`：用于加入和管理 AD 域的工具

以下命令显示了必要软件包的安装：

```
$ sudo yum install oddjob realmd samba samba-common oddjob-mkhomedir sssd adcli

```

# 使用 realm 来管理域的加入

安装这些软件包后，我们可以使用 `realm` 命令来管理我们的加入。该命令是我们已安装的 `realmd` 包的一部分。我们可以使用 list 子命令确保我们当前没有加入任何域：

```
$ realm list

```

输出应该是空的。现在，我们可以继续进行下一步：加入域。在一个简单的环境中，你应该知道你要加入的域；至少我们希望你知道。在我们的案例中，我们确实知道它是 `example.com`。通过使用 discover 子命令，我们可以验证是否已安装所有必需的软件包，如下所示：

```
$ realm discover example.com

```

此命令的输出将显示这是一个 Active Directory 域，以及在加入 AD 域之前应该具备的所需包。以下截图展示了这一点：

![使用 realm 管理域注册](img/image00284.jpeg)

根据你的 Active Directory 功能级别，可能需要安装 `samba-windbind` 或 `sssd` 包。我们在 2008R2 的 Active Directory 上使用，默认配置为 Windows Server 2003 的级别。此时，你应该确认已安装所有必需的包。

### 提示

如果我们不需要共享资源，就不需要安装 `samba` 包；`samba` 仅用于共享，而不是用于加入域。

由于这是一个 Kerberos 域类型，`join` 子命令将把服务器作为成员服务器加入域，并初始化 `/etc/krb5.keytab` Kerberos 密钥表文件和 `/etc/krb5.conf` 配置文件。有关这些文件的详细信息将在本章结束时提供。要加入 AD 域，使用以下命令将计算机添加到 AD 域中的默认文件夹：

```
$ sudo realm join --user=administrator@example.com example.com

```

如果你想将其添加到 Active Directory 中的指定组织单位（OU），你首先需要创建该 OU，或者至少确保它已经存在。在 OU 存在的情况下，命令将类似于以下内容，其中我们将其添加到 Linux OU：

```
$ sudo realm join --computer-ou="OU=Linux" \ --user=administrator@example.com example.com

```

这是我们将用来将 RHEL 服务器加入路径的方法：

```
OU=Linux,DC=example,DC=com

```

使用这两种方法中的任何一种，你将被提示输入域管理员密码或具有将计算机添加到 AD 域权限的用户密码，以及你的 `sudo` 用户密码（如果需要）。该命令可能需要几分钟才能生效，请耐心等待，直到返回命令行提示符。作为标准用户，你可以再次使用 `realm list` 命令列出已加入的域。我们应该注意到，第一次的输出可能看起来与我们之前运行的 `realm discover example.com` 命令类似；然而，经过仔细检查后，我们会发现我们现在是一个成员服务器，命令输出中会显示 `configured: kerberos-member`，如以下命令所示：

```
$ realm list

```

前面命令的输出如以下截图所示：

![使用 realm 管理域注册](img/image00285.jpeg)

# 使用 Active Directory 凭证登录 RHEL 7

欢迎来到集中式账户的世界。我想你会承认，使用 RHEL 7 这个过程非常简单，远比之前的 RHEL 版本要简单得多。我们现在可以开始使用来自 Active Directory 的中央用户账户了。

要登录 RHEL 7 服务器，我们可以使用 Active Directory 的 **UPN**（**用户主体名称**）。格式为 `user@<完全限定域名>`。例如，如果我们在 `example.com` 域中有一个名为 `jjones` 的账户，我们可以使用以下命令登录 RHEL 服务器：

```
jjones@example.com

```

下图显示了当我们使用`switch user`命令以`jjones`的 AD 帐户登录时的过程。请注意，由于`jjones`的主目录不存在，`oddjob`会友好地为我们创建它，如下图所示：

![使用 Active Directory 凭据登录 RHEL 7](img/image00286.jpeg)

要使用 SSH 工具（例如 Windows 的 PuTTY）远程连接，我们将使用以下语法，使用两个`@`符号；这看起来可能有点奇怪，但它是正确的：

```
jjones@example.com@192.168.0.69

```

以下截图显示了从 Windows 的 PuTTY 客户端连接到 RHEL 的 SSH 连接：

![使用 Active Directory 凭据登录 RHEL 7](img/image00287.jpeg)

我们现在已经看到，我们可以在 Linux 系统上使用 Active Directory 帐户。通过将 Red Hat 服务器作为我们域的一部分，我们可以使用一套凭据登录 Linux。当用户离开组织时，现在只需要删除或禁用一个用户帐户。我们已经在单台服务器上看到了这一点，但这同样适用于所有的 RHEL 7 或 CentOS 7 服务器和桌面；这个过程在各个设备上都是一样的，使我们更高效、更安全。

# 使用 adcli 进行用户和组管理

我们不仅仅局限于使用这些域帐户；我们还可以通过 Linux 服务器的命令行管理 Active Directory。在 Active Directory 中具有正确的权限后，我们可以：

+   创建用户和组

+   修改组成员资格

+   删除用户和组

虽然这些工具的功能不如本地操作系统中的工具丰富，特别是在使用 PowerShell 时，但 Linux 设备提供的某些管理功能仍然有其需求和优势。

如果你是 Linux 管理员并且主要在 Linux 上工作，那么将 Active Directory 用户添加到你用来在 Linux 上进行委派的组中是有意义的。例如，你可以维护一个名为`LinuxAdmins`的 Active Directory 组，并通过`/etc/sudoers`文件将权限委派给该组。完全正确的是，你管理和控制 AD 组，而不一定是 AD 中的`Domain Admins`组。

## 列出 Active Directory 信息

首先，我们将使用`adcli`命令，来看一下`info`子命令。它可以显示已发现的域及域控制器的详细信息。我们可以作为标准用户运行此命令，如下所示：

```
$ adcli info example.com

```

输出将显示域控制器的 Active Directory 角色以及站点的详细信息，如下图所示：

![列出 Active Directory 信息](img/image00288.jpeg)

通过这种方式，我们将能够验证连接以及我们连接的域控制器。

## 创建 Active Directory 用户

该命令可能不是最有用的工具，因为我们可以创建用户，但无法启用帐户或为新用户设置密码。因此，这个命令的使用价值不如`adcli`中的其他一些工具。示例命令如下：

```
$ adcli create-user fjones --domain=example.com --display-name="Fred Jones"

```

该命令将尝试以管理员身份登录到域，并会提示输入密码。若要以其他用户身份登录，您可以使用`-U`或`--login-user`选项。

为了完整性，我们覆盖了`create user`命令，但实际上，用户仍然需要在 Active Directory 中启用并设置密码。

要删除我们刚刚创建的帐户，我们将使用以下命令：

```
$ adcli delete-user  --domain=example.com fjones

```

## 创建 Active Directory 组

在许多方面，`adcli`命令对我们作为 Linux 管理员非常有用。因此，只要我们的域帐户具有在 AD 中创建和管理组的权限，我们就应该管理影响 Linux 访问的组成员身份。假设用户帐户已经创建，我们无需关注这些组的密码管理、创建和成员身份。我们将像以前一样使用管理员帐户访问域，但如果我们的帐户具有相应权限，也可以使用我们自己的帐户。

要在我们放置服务器的 Linux 组织单位（OU）中创建 Linux 用户组，我们将使用以下命令：

```
$ adcli create-group --domain=example.com \ --domain-ou="OU=Linux,DC=example,dc=com" "Linux Users"

```

我们可以通过导航到域控制器上的 Active Directory 用户和计算机中的**OU**（**组织单位**）来验证这一点。在以下截图中，我们可以看到我们有服务器组和新组：

![创建 Active Directory 组](img/image00289.jpeg)

我们将保留创建的组，因为我们将向其中添加用户；删除组的过程类似于删除用户的过程，如下命令所示：

```
$ adcli delete-group  --domain=example.com "Linux Users"

```

如果我们需要任何命令的帮助，可以使用类似以下命令的语法帮助：

```
$ adcli delete-group --help

```

只需使用您需要帮助的正确子命令。

## 管理 Active Directory 组成员身份

现在我们有了 Linux 用户组，我们可以管理该组的成员身份。在 AD 域中，我们有`jjones`用户，可以将其添加到此组中。以下命令展示了如何在我们的域中执行此操作：

```
$ adcli add-member  --domain=example.com "Linux Users" jjones

```

### 提示

除非在特定上下文中创建组或用户，否则我们可以仅通过`SAMAccountName`属性（即用户或组名称）来引用该对象。这是域中的唯一标识符。在前面的示例中，我们可以简单地将该组称为`Linux Users`，将用户称为`jjones`。如果组名中有空格，则需要使用引号保护。

# 委派 Active Directory 帐户使用 sudo

能够管理 Active Directory 组成员身份是我们管理 Linux 的基础。我们可以将文件和目录的所有权分配给这些组，并（更重要的是）通过`/etc/sudoers`文件委派系统上的权限。

让我们看看这种委派是如何工作的。我们将在 Active Directory 中创建一个新组，并将管理员添加到该组中。作为一个简单的设置，我们仅限于已创建的用户，如以下命令所示：

```
$ adcli create-group  --domain=example.com \ --domain-ou="OU=Linux,DC=example,dc=com" "Linux Admins"
$ adcli add-member  --domain=example.com "Linux Admins" Administrator

```

现在我们有两个组，可能用于委派：`Linux Users`和`Linux Admins`。为了使用`sudoers`系统进行委派，我们以 root 用户身份或使用`sudo`运行`visudo`命令。此文件可以用作委派，使选定的用户能够以 root 身份运行某些命令。这些命令必须以`sudo`命令开头。你可以将`sudo`视为类似于 Windows 系统中的`runas`命令：

```
$ sudo visudo

```

这将打开`/etc/sudoers`文件进行编辑。我们可以使用`G`移动到文件末尾，然后按`o`插入新行。

我们将向`/etc/sudoers`文件中添加这两行代码：

```
%Linux\ Admins@example.com ALL=(root) ALL %Linux\ Users@example.com ALL=(root) /sbin/mount /mnt/cdrom, /sbin/umount /mnt/cdrom

```

### 提示

请注意这里使用`\`来保护空格。这是必需的，因为`sudoers`文件不允许使用引号。

Linux 管理员组被允许使用`sudo`以 root 身份运行系统上的所有命令。Linux 用户组只能运行`mount`和`umount`命令，来挂载和卸载`cdrom`设备。

在`vi`中完成所有更改后，我们可以按`ESC`键退出，再按`:x`保存并退出插入模式。

以下是示例系统的截图，显示了应有的更改：

![使用 sudo 委派 Active Directory 账户](img/image00290.jpeg)

当我们以`jjones`身份登录时，我们现在会发现我们属于`Linux Users`组和`Administrator`的`Linux Admins`组。此外，两个用户都将属于`Domain Users`组。

我们可以作为任一组的用户运行以下命令：

```
$ id -Gn

```

上面的命令将显示用户所属的组名。域管理员账户将有多个组成员身份，但最重要的是会包括`Linux Admins`组。这将允许用户以 root 身份运行所有以`sudo`开头的命令，就像我们看到的具有类似权限委派的`andrew`账户。

以下截图显示了以管理员身份运行`id`命令时的输出：

![使用 sudo 委派 Active Directory 账户](img/image00291.jpeg)

我们还可以将文件系统的所有权分配给用户和组。从目录中，我们可以进行这一操作。当前我们仍然以域管理员账户登录到 RHEL 7.1 系统，我们将通过更改目录的组所有权来验证`sudo`条目的正确性；这通常是 root 用户的特权操作：

```
$ sudo chgrp Linux\ Usersexample.com /data
$ ls -ld /data

```

在前面的示例中，我们将`/data`目录的组所有权更改为`Linux Users`组；随后，我们还展示了该目录的所有权。为了更加清晰，我们提供了一张截图来演示这一过程：

![使用 sudo 委托 Active Directory 帐户](img/image00292.jpeg)

# 离开一个域

到目前为止，我们已经能够通过使用`sudo`的委托权限以及文件和目录的所有权和文件系统，展示与 Active Directory 的真正互操作性。这非常出色，完全符合企业级 Linux 系统的预期；然而，尽管这一点非常突出，但仍然会有需要将 Linux 服务器从域中移除的情况。通常，这发生在服务器从一个域中移除后，再加入另一个域。若需要这样操作，`realm`命令使得这一过程变得简单，可以通过将操作反向到`join`子命令来实现，示例如下：

```
$ sudo realm leave example.com --remove

```

额外选项：`--remove`将确保计算机帐户也从域中删除；否则，它需要单独删除。暂时，我们将计算机保留在域中。

# 理解 Active Directory 作为 sssd 的身份提供者

在许多方面，Linux 上能实现如此简单的功能是非常受欢迎的；然而，这种简单性掩盖了幕后的复杂事件和过程。现在是时候深入了解`sssd`是如何工作的了。

我们首先需要提醒自己回顾在过程中唯一的手动操作部分——即设置集成到 Active Directory 所需的时间和 DNS 基础设施服务。下图展示了 RHEL 服务器与 Active Directory 之间的关系：

![理解 Active Directory 作为 sssd 的身份提供者](img/image00293.jpeg)

当我们使用`realm`命令查询 Active Directory 域时，从结果信息中可以看到，我们需要安装`sssd`包等其他组件。系统安全服务守护进程（`sssd`）提供了一组守护进程，用于管理对远程目录和身份验证机制的访问，在我们的案例中，指的是 Active Directory。`sssd`服务为我们的系统提供**NSS**（**名称服务切换**）和**PAM**（**可插拔认证机制**）接口，以及一个模块化的后端系统，用于连接多个不同的帐户来源和 D-bus 接口。考虑到这一点，我们应该理解系统上已经为我们添加并配置了 NSS 和 PAM 模块。

在远程 Active Directory 上识别帐户是通过 LDAP 进行的，身份验证是通过 Kerberos 完成的，连接到 AD 域。LDAP 帐户查找参考并调用`/usr/lib64/libnss_sss.so.2` NSS 模块和`/etc/nsswitch.conf`文件。身份验证将通过`/lib64/security/pam_nss.so`进行引用。

我们可以扩展关系图，加入`sssd`，如下所示：

![理解 Active Directory 作为 sssd 的身份提供者](img/image00294.jpeg)

## 配置 NSS

**名称服务切换**（**NSS**）配置文件 `/etc/nsswitch.conf`，被各种 NSS 库使用；其中一个 NSS 库是 `/usr/lib64/libnss_sss.so.2`。NSS 配置文件确定您可以从中获取名称服务信息的源和其顺序，范围包括各种类别。每个信息类别由资源数据库名称标识；这可以是用于名称解析的`hosts`和用于查找用户账户的`passwd`。

在我看来，最简单的方法是使用 `hosts` 数据库来解释这些功能是如何工作的。在 `/etc/nsswitch.conf` 中 `hosts` 的条目如下所示：

```
hosts:      files dns

```

通过生效的设置，首先通过解析`/etc/hosts`本地文件来实现前述的名称解析，然后通过 DNS 解析库。如果颠倒这些条目，DNS 将在本地文件之前检查。

如果我们在文件中检查`sss`，我们可以看到所有依赖特定库的数据库。`grep`命令可用于隔离这些条目，如以下命令所示：

```
$ grep sss /etc/nsswitch.conf

```

查询的结果应该类似于以下的屏幕截图：

![配置 NSS](img/image00295.jpeg)

这些是默认设置，但我们不必接受它们；如果需要，我们可以实施更改。但是，这个顺序可能是最好的，因为它允许在搜索域之前解析本地账户（并非域账户将冲突，因为它们是使用用户的完整 UPN 指定的）。

这里解释了数据库名称：

+   `passwd`: 这指定了用户账户。

+   `shadow`: 这表示密码信息。

+   `group`: 这指定了组账户。

+   `services`: 这表示服务名称解析。

+   `netgroup`: 这指定了可以在访问控制规则中使用的主机组。

+   `automount`: 这指定了由 `autofs` 自动挂载的目录。

在许多设置中，禁用底部三个元素 `services`、`netgroup` 和 `automount` 很容易。例如，如果在运行工具（如`netstat`）时保留默认设置，可以帮助调整目录访问，它将在 `/etc/nsswitch.conf` 的服务数据库条目中运行 LDAP 查询以解析端口地址到服务名称。

防止 LDAP 查询的服务条目将类似于以下命令：

```
services: files

```

## 配置 PAM

我们通常可以将**可插入认证模块**（**PAM**）保留为原样，但是我们将在这里查看它们的配置。

可以使用 PAM 的服务通过`/etc/pam.d`目录中的相关 PAM 模块进行配置。它们可以是单独的文件，比如`/etc/pam.d/login`，也可以通过被多个服务引用的命令文件（如`/etc/pam.d/system-auth-ac`）。

我们可以使用`grep`再次从`/etc/pam.d/system-auth-ac`文件中筛选`sss`，以显示`sssd`与 PAM 一起使用的配置。输出如下截图所示：

![配置 PAM](img/image00296.jpeg)

我们可以看到认证模块在所有可能的触发条件下都会被使用：

+   `auth`：在认证过程中使用

+   `account`：用于账户限制

+   `password`：用于密码更改事件

+   `session`：在登录会话期间使用

让我们看看一些与认证模块一起使用的参数（例如`use_first_pass`）。一些可能的参数如下：

+   `forward_pass`：输入的密码可以用于其他模块

+   `use_first_pass`：不会提示输入密码，而是使用先前输入的密码。

+   `use_authtok`：更改密码时，可以使用先前输入的密码来认证密码更改。

+   `retry=N`：如果设置了此项，当用户输入错误的密码时，可以提示用户多次输入密码。

## 配置 Kerberos

当你使用 realm 加入域时，会创建`/etc/krb5.conf`密钥表文件来将 RHEL 系统认证到域，同时也会创建`/etc/krb5.conf`文件。清理文件后，移除我们域的注释，文件看起来类似于以下截图：

![配置 Kerberos](img/image00297.jpeg)

我们可以看到`/etc/krb5.conf`文件有四个部分：

+   `logging`

+   `libdefaults`

+   `realms`

+   `domain_realm`

由于演示实验室非常小，只有一个域控制器，因此无需进行更改。如果你有更大的设置，可能需要在 realm 中添加更多的详细信息。你可以指定持有正确角色的本地域控制器；否则，只需让 DNS 服务记录按以下方式解析这些记录：

```
[realms]
EXAMPLE.COM {
 kdc = ad1.example.com
 admin_server = ad1.example.com
}

```

## 配置 SSSD

`sssd`的配置可以在`/etc/sssd/sssd.conf`文件中找到。我们已经看到这默认情况下对我们有效，但也有自定义的空间，如下截图所示：

![配置 SSSD](img/image00298.jpeg)

这里的一个简单修改是更改 AD 用户的主目录位置。默认情况下，这是`/home/example.com/username`。如果你已将 Unix 扩展添加到 Active Directory 中，则我们将设置`ldap_id_mapping`为`false`，UID 和 GID 将在 Active Directory 中设置。

# 总结

在本章中，我们研究了如何使用 Active Directory 作为身份存储，利用 Linux 上的用户和组。设置的简便性使其成为全球企业中非常实用且迫切需要的解决方案。

在设置时间和 DNS 之前需要做一些基础工作。一旦完成此设置，使用 realm 命令配置`sssd`以将 Active Directory 作为身份源就非常简单。

在 AD 域中的 RHEL 系统，我们可以通过`adcli`在一定程度上管理此域，并通过控制台或 SSH 为用户提供访问 Linux 命令行的权限。

接下来，我们将继续探讨文件共享，但这次我们将使用 Apache HTTPD web 服务器。
