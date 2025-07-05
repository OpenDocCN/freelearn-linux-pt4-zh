# 第六章. 故障排除共享资源

在当今世界，共享信息的需求无处不在。从家庭办公室到小型办公室，再到公司和企业办公室以及公共场所，保持协作信息的能力是解决故障排除角色的重要组成部分。毫无疑问，共享资源或提供对远程位置的访问现在被视为一种普遍期望，本章的目的是讨论并强调与通过网络访问共享资源相关的若干问题。

在本章中，我们将：

+   发现如何在 CentOS 7 服务器上提供 NFS 共享

+   了解更多关于 NFS 导出的信息

+   学习如何在 CentOS 7 客户端工作站上访问 NFS 共享资源

+   学习如何使用 CIFS 挂载外部驱动器

+   学习如何使用`autofs`来提供持久挂载

# 在 CentOS 7 服务器上提供 NFS 共享

NFS 共享已经存在一段时间，正如大多数此类问题一样，故障排除文件共享服务的基本方法是基于您对安装过程的了解，因为您将花费大量时间审查权限和网络环境，并诊断服务启动失败的原因。

首先，您应该确保客户端工作站和服务器能够互相 ping 通：

```
# ping -c 4 XXX.XXX.XXX.XXX

```

如果`ping`命令在服务器和客户端工作站上都成功，为继续操作，您需要在服务器上安装`nfs-utils`包，如下所示：

```
# yum install nfs-utils

```

安装完成后，我们现在需要以以下方式创建一个永久的发布目录：

```
# mkdir /path/to/nfs/publication/directory

```

例如，一种方法是使用`/home`目录，如下所示：

```
# mkdir /home/nfs-share

```

由于此位置将存储客户端文件，因此您需要通过输入以下命令确保权限正确：

```
# chmod -R 775 /path/to/nfs/publication/directory

```

### 提示

请记住，如果您打算使用服务器上的`/home`目录，在修改`/home`目录的权限时要小心，并直接定位到适当的子目录，因为您不想影响其他文件夹。

下一步是启动相关服务，方法如下：

```
# systemctl enable rpcbind
# systemctl enable nfs-server
# systemctl enable nfs-lock
# systemctl enable nfs-idmap
# systemctl start rpcbind
# systemctl start nfs-server
# systemctl start nfs-lock
# systemctl start nfs-idmap

```

此时，我们将决定通过对以下文件进行一些修改，在网络上共享 NFS 目录：

```
# nano /etc/exports

```

在`XXX.XXX.XXX.XXX`为客户端工作站的 IP 地址的地方，按以下方式添加您的网络共享点：

```
/path/to/nfs/publication/directory    XXX.XXX.XXX.XXX(rw,sync,root_squash,no_all_squash)
```

自然地，您可以根据需要为一个或多个用户添加多个发布目录；每个用户可以根据其 IP 地址拥有一个独特的目录。但是，如果您希望创建一个全局发布目录——即一个为所有客户端工作站服务的单一目录——那么您应该使用以下语法：

```
/path/to/nfs/publication/directory    *(rw,sync,no_root_squash,no_all_squash)
```

请注意，星号符号（`*`）已替代了 IP 地址。在生产服务器或类似环境中，您应该避免这种做法，改用您尝试共享的网络/子网。这是一个特别需要排错人员注意的问题，但为了便于解释安装过程，我们将使用这种方法，原因是它是一种常见的做法。

因此，根据我们在这里详细描述的原始工作示例，条目可能如下所示：

```
/home/nfs-share    *(rw,sync,no_root_squash,no_all_squash)
```

最后，您可能需要考虑所需的防火墙更改。为此，请逐个添加以下行：

```
# firewall-cmd --permanent --zone=public --add-service=nfs
# firewall-cmd --reload

```

然后，按照以下方式重新启动 NFS 服务：

```
# systemctl restart nfs-server

```

# 关于 NFS 导出

在此阶段，我想讨论一下 CentOS 7 上的 NFS 导出问题。如前面的示例中所述，我们使用了以下语法：

```
(rw,sync,root_squash,no_all_squash)
```

在这里，大部分选项对您来说应该是显而易见的；`root_squash` 将允许客户端的 root 用户以 root 身份访问和创建 NFS 服务器上的文件。技术上讲，此选项将强制 NFS 将客户端的 root 更改为匿名 ID，实际上，通过防止系统间 root 账户所有权迁移，这将增加安全性。如果您在 NFS 服务器上托管 root 文件系统（尤其是对于无盘客户端），则需要此设置；考虑到这一点，它可以（适度）用于选定的主机，但除非您了解其后果，否则不应使用 `no_root_squash`。

### 注意

**其他基本的导出选项包括：**

+   `no_all_squash`：此选项禁用所有的 squashing（压缩）。

+   `rw`：此选项使 NFS 服务器能够在 NFS 卷上同时进行读写请求。

+   `ro`：此选项使 NFS 服务器在 NFS 卷上使用只读请求。

+   `sync`：此选项使 NFS 服务器仅在更改已提交到稳定存储后才回复请求。

+   `async`：此选项使 NFS 服务器违反 NFS 协议，在任何更改提交到稳定存储之前就回复请求。

+   `secure`：此选项要求请求来源于一个互联网端口。

+   `insecure`：此选项接受任何或所有端口。

+   `wdelay`：此选项使 NFS 服务器在怀疑另一个相关的写请求可能正在进行或即将到来时，延迟提交写请求到磁盘。

+   `no_wdelay`：此选项使 NFS 服务器允许多个写请求在单次操作中提交到磁盘。此功能可以提高性能，但如果 NFS 服务器接收到许多小请求，这种行为可能会降低性能。您应该知道，如果同时设置了 `async`，此选项将没有任何效果。

+   `subtree_check`：此选项启用 `subtree`（子树）检查。

+   `no_subtree_check`：此选项禁用 `subtree`（子树）检查，虽然它带有一些隐含的安全问题，但它可以提高可靠性。

+   `anonuid=UID`：这些选项明确设置匿名账户的`uid`和`gid`；当你希望所有请求看起来都来自同一个用户时，这个选项很有用。

+   `anongid=GID`：此选项将禁用`anonuid=UID`。

# 在 CentOS 客户端上挂载 NFS 共享

假设你的服务器当前正在提供 NFS 共享，我们现在将检查客户端工作站，以确保一切正常工作。这是每个故障排除人员都需要掌握并完善的任务。

首先，客户端必须像这样使用`nfs-utils`包：

```
# yum install nfs-utils

```

完成`nfs-utils`包的安装后，你现在必须按照以下方式创建挂载点：

```
# mkdir -p /path/to/mount/point

```

例如，为了满足你的需求，前面的命令可能如下所示：

```
# mkdir -p /mnt/nfs/home

```

现在可以像这样启动相关服务：

```
# systemctl enable rpcbind
# systemctl enable nfs-server
# systemctl enable nfs-lock
# systemctl enable nfs-idmap
# systemctl start rpcbind
# systemctl start nfs-server
# systemctl start nfs-lock
# systemctl start nfs-idmap

```

最后，我们可以通过以下方式在客户端工作站上挂载 NFS 共享：

```
# mount -t nfs XXX.XXX.XXX.XXX:/path/to/folder /path/to/mount/point

```

这里，`XXX.XXX.XXX.XXX`是 NFS 服务器的 IP 地址，`:/path/to/folder`是共享资源的位置。`/path/to/mount/point`部分表示共享资源在客户端工作站上应找到的位置。

这个命令的工作示例如下所示：

```
# mount -t nfs 192.168.1.100:/home /mnt/nfs/home/

```

为了确认一切正常工作，你可能需要运行以下命令：

```
# df -h

```

根据此处提供的工作示例，命令输出应显示一个或多个文件系统的添加，类似如下：

```
192.168.1.100:/home           21G   33M   21G   1% /mnt/nfs/home

```

然后，你可以通过以下方式轻松确认 NFS 资源的读写权限，通过创建一个新的文本文件：

```
# touch /path/to/mount/point/nfs-test.txt

```

使用`ls`命令，你应该能够在服务器和客户端工作站上看到这个文件。然而，由于这只是一个临时解决方案，如果你决定将其设置为永久或持久挂载，那么你应该打开以下文件：

```
# nano /etc/fstab

```

现在为每个挂载点添加一个条目，如下所示：

```
XXX.XXX.XXX.XXX:/path/to/folder   /path/to/mount/point   nfs defaults 0 0
```

完成这些步骤后，你现在可以重新启动客户端机器，并且完全知道 NFS 服务将始终可用。

# 使用 CIFS 挂载外部驱动器

在 CentOS 7 工作站或服务器上挂载外部驱动器被认为是一个相对简单的过程，在很多方面，这将是一个资深故障排除人员的日常任务。然而，在某些情况下，整个过程确实会引起很多关于需要哪些步骤的困惑，鉴于此，我们的目标是提供一些急需的清晰指导。

我们将首先确认`cifs`是否已安装。为此，输入以下命令：

```
# rpm -q cifs-utils

```

CIFS，即通用互联网文件系统，是一种文件共享协议，它为跨多个文件系统的远程文件访问提供了一种标准。基于**服务器消息块**（**SMB**）并通过 TCP/IP 运行，`cifs`提供了典型的文件操作，如打开、关闭、读取、写入、安全缓存和查找。它支持扩展的非文件系统属性、批量请求和分布式复制虚拟卷。然而，如果系统以以下方式回应，您就知道`cifs`当前没有安装：

```
package cifs-utils is not installed

```

要解决此问题，只需输入以下命令来安装`cifs`软件包：

```
# yum install cifs-utils

```

目前，您需要决定将设备挂载到哪里，这是一个可以通过以下方式实现的简单过程：

```
# mkdir /path/to/mount/folder

```

然后使用`cifs-utils`软件包通过输入以下命令来挂载外部驱动器：

```
# mount -t cifs //XXX.XXX.XXX.XXX/path/to/folder /path/to/mount/folder

```

然而，如果您希望传递一个字符串或一系列变量，如用户名和密码，那么完整的命令应该调用`-o`选项，像这样：

```
# mount -t cifs //XXX.XXX.XXX.XXX/path/to/folder /path/to/mount/folder -o user=<username> password=<password>

```

需要注意的是，这个过程也可以用来挂载许多不同类型的共享资源，并且如果外部资源的相关文件夹名称中包含空格，那么可以按照常规方式处理：

```
# mount -t cifs //XXX.XXX.XXX.XXX/path/to/folder\ name /path/to/mount/folder -o user=<username>,password=<password>

```

您使用的字符串类型还可以利用手册中提到的其他功能。然而，重要的一点是要记住，前述解决方案只会保持挂载的驱动器，直到该驱动器被卸载、断开连接或重启。因此，当前阶段的任何设置都不是永久的或持久的。

# 使用 autofs 挂载外部驱动器

如果您希望使连接外部驱动器（通过`cifs`）的过程永久（持久化），那么您需要先按照以下方式安装`autofs`软件包：

```
# yum install autofs

```

安装好该软件包后，您需要确保像这样启动并启用`autofs`服务：

```
# systemctl start autofs
# systemctl enable autofs

```

完成此操作后，假设我们将使用本文讨论的相同挂载点，您应该开始配置`autofs`服务，通过输入以下命令，在您喜欢的文本编辑器中创建一个凭据文件：

```
# nano /path/to/credentials.txt

```

现在，像这样添加所需的网络凭证：

```
username=<access_username>
password=<access_password>
```

保存并关闭此文件后，请确保修改权限以确保系统安全：

```
# chmod 600 /path/to/credentials.txt

```

这个过程的下一阶段是打开`autofs`配置文件：

```
# nano /etc/auto.master

```

在此文件末尾添加以下行，但请确保根据您的需求自定义这里显示的值：

```
/path/to/mount/folder  /etc/auto.cifs  --timeout=600 --ghost
```

如您所见，我们使用了各种选项，其中`--timeout`选项设置了未访问共享时卸载前的空闲时间（以秒为单位，默认时间为 10 分钟）。建议使用`--ghost`选项，因为它在挂载点未挂载期间创建一个`ghost`或空的持有文件夹。

现在，创建并编辑以下挂载文件：

```
# nano /etc/auto.cifs

```

添加并自定义以下行以满足您的需求：

```
<Local-Name>  -fstype=cifs,rw,noperm,credentials=/path/to/credentials.txt    ://XXX.XXX.XXX.XXX/path/to/share/folder
```

使用的`Local-Name`将显示在挂载点目录中（`/path/to/mount/point/Local-Name`），而随后的参数只是调用文件系统类型，无论您是否提供读写访问权限以及预期的访问凭证。因此，如果您计划提供只读访问权限，请记得将`rw`替换为`ro`。此外，鉴于所做的自定义结构，您可以看到如何以以下方式添加多个位置（这也意味着可以调用多个凭证）：

```
<Local-Name>  -fstype=cifs,rw,noperm,credentials=/path/to/credentials1.txt    ://XXX.XXX.XXX.XXX/path/to/share/folder1
<Local-Name>  -fstype=cifs,rw,noperm,credentials=/path/to/credentials2.txt    ://XXX.XXX.XXX.XXX/path/to/share/folder2
<Local-Name>  -fstype=cifs,rw,noperm,credentials=/path/to/credentials3.txt    ://XXX.XXX.XXX.XXX/path/to/share/folder3
```

最后，如果您希望避免使用 IP 地址，那么您应该确保通过`/etc/hosts`或 DNS 将主机名映射到您的系统，以便使用以下语法更改：

```
<Local-Name>  -fstype=cifs,rw,noperm,credentials=/path/to/credentials.txt    ://hostname/path/to/share/folder
```

完成这些步骤后，下一步是重启您的系统，但作为替代，您也可以简单地重新启动`autofs`服务，以便享受您工作的成果：

```
# systemctl restart autofs

```

# 总结

在本章中，我们考虑了通过网络提供和访问共享资源的独特方法。从 NFS 到`cifs`，从`fstab`到`autofs`，我们已经覆盖了使挂载点临时或永久的过程，并确保我们论证了 CentOS 可以支持所有操作系统。然而，与迄今为止的其他章节不同，我们没有直接诊断问题，而是将故障排除者的角色定义为对安装过程的回顾，以便您的理解和对共享资源常见问题的认识是基于初始安装，而不是安装后发生的事件。通过这种方式，并且通过了解安装过程，您也将知道如何修复与该服务相关的任何问题。考虑到这一点，我们将继续讨论安全问题的故障排除。

# 参考资料

红帽客户门户：[`access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/`](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/)
