# 第六章 NFS 文件共享

使用**网络文件系统**（**NFS**）共享文件是 Unix 和 Linux 上远程主机能够通过网络挂载文件系统并像本地挂载一样与之交互的传统方式。虽然 RHEL 7 支持 NFSv3 和 NFSv4，但不再支持 NFSv2。RHEL 7 客户端默认使用 NFSv4，如果无法建立连接，则会回退到 NFSv3。使用 NFSv4 简化了在防火墙后定位服务的过程，仅需要 TCP 端口`2049`供客户端访问；然而，我们将演示 NFSv4 和 v3 的防火墙配置。本章将涉及以下内容：

+   NFS 概述

+   实验环境概述

+   NFS 服务器配置

+   使用`exportfs`

+   在防火墙后托管 NFSv4

+   在防火墙后托管 NFSv3

+   NFS 客户端配置

+   使用`autofs`自动挂载 NFS

# NFS 概述

我们已经习惯了 NFSv4 在 Red Hat Enterprise Linux 6 中默认包含。RHEL 7 新增了对`pNFS`（并行 NFS）版本 4.1 的支持。`pNFS`提供了安全性和性能增强，使得在防火墙和**网络地址转换**（**NAT**）路由器后更加高效地连接客户端。

不再支持 NFSv2，这并不是什么大损失，因为它不支持超过 2GB 的文件大小，并且没有版本 3 和 4 那样健壮。

使用 NFSv4 时，挂载和锁定协议都采用了*一切打包*的理念。这使得仅使用一个 TCP 端口`2049`就足够。然而，使用 NFSv3 时，我们必须使用`rpcbind`并为额外的服务设置静态端口，以便配置防火墙。这简化了防火墙配置，稍后您将看到，配置只需要访问 TCP 端口`2049`。

服务器和客户端工具都来自`nfs-utils`包进行安装。此包包含 NFSv4 和 v3 协议的工具，还包括其他有用的工具，例如`nfsiostat`，可以用于监控 NFS 服务器上共享文件的使用情况。要列出已安装包的内容，可以使用`rpm`命令，以下命令行可以作为普通用户运行：

```
$ rpm -ql nfs-utils #lists all files in the package
$ rpm -qd nfs-utils #lists just the documentation files
$ rpm -qc nfs-utils #list only the configuration files
$ rpm -qi nfs-utils #displays descriptive information on the package

```

# 实验环境概述

本章演示中，我们将使用在**Oracle VirtualBox**虚拟化环境中运行的两台虚拟机。VirtualBox 可以从[`www.virtualbox.org/`](https://www.virtualbox.org/)免费下载，并且支持 Windows、Mac OS X、Linux 和 Solaris 主机。

NFS 服务器将在 IP 地址为`192.168.10.10`、主机名为`nfshost`的 RHEL 7.1 主机上配置。NFS 客户端将在 IP 地址为`192.168.10.11`、主机名为`nfsclient`的 RHEL 7.1 主机上配置。

两台机器都以最小配置安装；我们在两台主机上都安装了`nfs-utils`包，如下所示：

```
$ sudo yum install -y nfs-utils

```

此外，在`nfshost`主机上，我们安装了 net-tools 软件包，以便我们可以使用`netstat`命令显示打开的端口。安装`net-tools`的命令如下：

```
$ sudo yum install -y net-tools

```

防火墙正在运行默认设置，并通过`firewall-cmd`命令进行管理。为了允许 NFSv4 连接到`nfshost`，我们使用以下命令额外开放了 TCP 端口`2049`：

```
$ sudo firewall-cmd --add-port=2049/tcp --permanent
$ sudo firewall-cmd --reload

```

### 提示

我们将在本章稍后介绍 RHEL 7 上的防火墙，并且会详细讲解如何在本书的第十一章，*使用 firewalld 进行网络安全*一章中使用`firewall-cmd`和`firewalld`服务。

NFS 不仅使用防火墙来保护服务器，还支持 TCP 包装器来控制访问。访问服务的权限可以通过使用`/etc/hosts.allow`和`/etc/hosts.deny`文件来决定。

# NFS 服务器配置

要配置 NFS 服务器，我们需要选择希望共享的目录。NFS 中用于共享目录的术语是*导出*目录；因此，共享的目录被称为**导出**。

要永久导出一个目录，我们需要将配置添加到/`etc/exports`文件中。该文件是存在的，但在新系统中会为空。`nfs-server`服务在启动时会读取此文件，以确定哪些目录应该对网络客户端可用。如果`/etc/exports`文件发生更改，重新加载`nfs-server`服务将强制该服务重新读取文件，如下所示的命令行：

```
$ sudo systemctl reload nfs-server

```

要显示服务器上的当前导出，我们可以使用`exportfs`或`showmount`命令。现在我们将花一点时间来启动所需的服务并创建我们的第一个简单导出。

首先，我们需要启动所需的服务。我们可以独立启动并启用每个服务，但为了自动化的精神，我们将在命令提示符下编写一个简单的循环，以节省一些打字和时间。我们将使用`sudo`；你的用户账户需要在`sudoers`文件中列出。一旦确保你有权限使用`sudo`，命令将如下执行：

```
for s in rpcbind nfs-server nfs-lock nfs-idmap ; do
 sudo systemctl enable $s
 sudo systemctl start $s
done

```

如果这使语法对你来说更清晰，下面的截图显示了在`nfshost`上执行的命令：

![NFS 服务器配置](img/image00254.jpeg)

## 简单的导出

如果不编辑`/etc/exports`文件，我们无法导出文件系统中的任何内容。因此，当我们使用`exportfs`显示本地导出时，将没有输出，如下所示的命令行：

```
$ sudo exportfs

```

我们在使用`showmount`命令时将不会有太多运气，如下图所示：

![简单导出](img/image00255.jpeg)

如你所见，`showmount`命令会显示导出列表的标题，但当然，列表为空，直到我们明确地定义一些导出的目录。

### 提示

`showmount` 命令可以在远程主机上使用，比如 `nfsclient`，以列出已导出的目录，但这将依赖于额外的服务。因此，`nfshost` 的防火墙需要为 NFSv3 配置。我们将在本章后面讨论这一点。

我承认什么都不共享，什么也没有，可能不是本书中最令人兴奋的功能，但至少我们发现了一些有用的工具，如 `exportfs` 和 `showmount`。接下来，我们将导出一个现有目录，仅仅是为了熟悉 NFS。为此，我们需要以 root 身份编辑 `/etc/exports` 文件；我们可以使用 `sudo` 来完成。你可以直接以 root 身份登录，也可以使用 `su` 命令。我们将添加以下行以导出或共享 `/usr/share/doc` 目录。这只是一个简单的测试，稍后我们将添加自己的目录和内容。为了演示，我们将坚持使用 `vi` 来编辑文件；不过，你也可以使用你喜欢的编辑器：

```
$ sudo vi /etc/exports

```

打开文件后，在没有新 NFS 服务器内容的情况下，我们可以添加以下行来导出 `/usr/share/doc` 目录：

```
/usr/share/doc *(ro)

```

使用 `cat` 命令，我们可以显示应编辑的文件名以及编辑完成后文件的内容，如以下截图所示：

![简单的导出](img/image00256.jpeg)

导出一个目录后，我们应该能够通过 `exportfs` 或 `showmount` 来查看它。

### 提示

`exportfs` 命令需要管理员权限，而 `showmount` 命令则不需要。

然而，在我们进一步讨论之前，我们需要回顾一下，`nfs-server` 服务在启动时会读取此文件，并且当前正在运行。我们可以重启该服务，但最好重新加载该服务。通过这种方式，如果远程主机当前已挂载了导出的目录，就无需停止该服务。运行以下命令将重新加载该服务，然后显示导出目录或多个目录：

```
$ sudo systemctl reload nfs-server
$ sudo exportfs

```

之前列出的两个命令的输出现在显示在以下截图中：

![简单的导出](img/image00257.jpeg)

当我们定义导出时，我们将一个目录导出到所有主机，这些主机用星号符号表示；任何导出的选项都包含在括号中。我们通过添加 `ro` 选项指定该导出为只读。

作为一个简单的测试，我们现在可以使用 `nfsclient` 主机访问此导出。从 `nfsclient` 的控制台中，我们可以访问已导出的目录，并使用以下命令将其挂载到 `nfsclient` 本地的 `/mnt` 目录：

```
$ sudo mount 192.168.10.10:/usr/share/doc /mnt

```

我们可以使用服务器的 IP 地址或主机名，只要该主机名能够通过 DNS、**mDNS**（**多播 DNS**）或本地主机文件解析。服务器主机名或 IP 地址的末尾必须以冒号表示。

我们可以使用标准的 `ls` 命令轻松列出导出目录的内容，针对的是 `/mnt` 目录。以下截图显示了 `ls` 命令的截断输出：

![简单导出](img/image00258.jpeg)

## 高级导出

我们已经看到，如果仅需要简单的目录导出，Linux 系统管理员的工作可以变得多么简单。然而，尽管这个选项可能适用于一些目录导出和服务器，但其他情况可能需要更多的时间和精力。

`/etc/exports` 文件中的基本指令如下：

```
export host(options)

```

这些变量的结构如下：

+   `export`：这是在 NFS 服务器上被导出或共享的目录。

+   `host`：这是导出目录共享的主机或网络。

+   `options`：这些是由主机或网络在括号后使用的特定选项。

还可以通过写一个单独的条目，将一个导出共享给不同的主机或网络，具有不同的选项，如下示例代码所示：

```
export host(options) host(options)

```

将这些变量扩展为实际值，替换后的工作示例如下：允许 `192.168.10.11` 具有读写访问权限，而对所有其他主机只有只读访问权限，我们可以查看以下代码：

```
/usr/share/doc *(ro) 192.168.10.11(rw,sync)

```

选项是以逗号分隔的，我们在 `nfsclient 192.168.10.11` 的选项中额外添加了 `sync` 选项。`sync` 选项将确保对该导出的写入操作会即时写入磁盘，而不是等待写缓冲区被刷新到磁盘。Linux 使用一种缓冲系统，促进脏缓存缓冲区的使用。这些缓冲区会随着数字的增长被写入磁盘。`sync` 选项确保这些缓冲区会立即写入磁盘。尽管这会对性能产生负面影响，但在连接不总是保持的情况下，它可以更可靠。

如果 `/etc/exports` 文件中的单行过长，则可以使用反斜杠（`\`）字符进行换行。在文件中，每个导出必须单独占一行。额外的空白行会被忽略，可以添加以提高可读性。如果行以井号（`#`）字符开头，则该行会被服务器忽略。

### 提示

如果导出授予了读写访问权限，而文件系统对用户是只读的，那么他们仍然只有只读访问权限。如果导出设置为只读，而文件系统通常会允许用户的读写访问，那么他们仍然只有只读访问权限。简单来说，当结合文件和导出权限时，最严格的权限将生效。

如果我们要确保百分百准确，导出的选项是可选的。如果没有设置选项，将应用默认选项。然后我们可以重写之前的示例，利用默认值，如下所示：

```
/usr/share/doc * 192.168.10.11(rw)

```

从修改后的示例中，您应该能够正确猜测，`ro` 和 `sync` 默认选项不再显式设置，但它们仍然有效。可以使用 `exportfs` 命令和 `-v` 选项查看导出目录的有效选项，如以下命令所示：

```
$ sudo exportfs -v

```

如果没有设置选项，并且在前一个命令的输出中没有显示该选项，则会看到默认选项。

NFS 服务器的默认选项包括以下内容；更多详情请参阅`man`页面：

+   `ro`：这使得导出的文件系统在远程主机上变为只读。

+   `sync`：NFS 服务器在响应新请求之前会将更改写入磁盘。

+   `wdelay`：与`sync`选项一起使用；NFS 服务器将延迟写入磁盘，预计会有更多写入操作马上进行。

+   `root_squash`：远程用户以 root 或`UID 0`身份连接时，会被更改为`nfsnobody`，因此我们将只能收集授予其他用户的权限。这有效地压制了 root 访问的权限，防止了未授权的 root 访问导出的文件系统。

我们现在将修改`/etc/exports`文件，以表示我们将导出到的两个主机集，并验证我们可以从指定的主机`192.168.10.11`连接。导出设置为`rw`，这将覆盖设置为所有主机`*`的`ro`选项。我们将使用`echo`命令覆盖`exports`文件，以便您可以看到文件中的编辑内容以及其他命令。以下示例列出了这些命令，并附有截图以展示导出选项：

```
$ sudo bash -c 'echo "/usr/share/doc * 192.168.10.11(rw)" > /etc/exports'
$ sudo systemctl reload nfs-server
$ sudo exportfs -v

```

![高级导出](img/image00259.jpeg)

从前面的截图中，我们可以看到`192.168.10.11`主机具有读写访问权限，而`<world>`或其他所有主机仅具有只读访问权限。

### 提示

**小心空格**

`/etc/exports`文件的格式非常精确，主机/网络前不应有空格，选项应紧跟其后。以下条目具有非常不同的含义：

```
/home server1(rw)
#correctly shares /home as read and write to server1
/home server1 (rw)
#shares the export /home to server1 using the default read-only and to <world> read-write is assigned, the asterisk can be omitted when designating all hosts.

```

## 伪根目录

如您在`/usr/share/doc`当前导出中看到的那样，访问服务器时，导出目录的完整路径是正常的。通过在服务器上使用伪根目录，可以简化访问导出目录所需的路径。这仅适用于 NFSv4 服务器和客户端。伪根目录设置完成后，我们可以将其他目录挂载到该路径。让我们在`nfshost`上查看这一点。

我们将清空当前的导出目录。这一次，我们将从头开始设置共享，稍加思考和规划。

首先，我们将在`nfshost`上创建一个新的目录，作为伪根目录：

```
$ sudo mkdir -m 1777 /var/exports

```

我们可以同时创建该目录并设置模式或权限。在这里，我们设置了所有用户的权限，并包含了粘滞位，以便用户只能删除他们拥有的文件。

接下来，我们将在`/etc/exports`文件中覆盖当前的导出设置，使用新创建的目录：

```
$ sudo bash -c 'echo "/var/exports 192.168.10.11(rw,fsid=0,crossmnt)" > /etc/exports'
$ sudo systemctl reload nfs-server

```

这些命令在`nfshost`上运行，并显示在以下截图中：

![伪根目录](img/image00260.jpeg)

这里我们实现了两个新选项：

+   `fsid=0`：这将该目录设置为通过 NFS 访问时服务器的根目录。这样，`/var/export`目录将从远程客户端以`192.168.10.10:/`的方式访问。

+   `crossmnt`：这是我们需要的一个巧妙选项，用于允许访问挂载点下的目录。为了将目录挂载到该导出点，我们将使用`mount --bind`命令。稍后将详细介绍。

设置导出选项为读/写模式可以让我们通过`nfshost`上的文件权限来控制访问权限。任何用户在从`nfsclient`访问时都会拥有完全权限，因此需要在文件系统中进行限制。

设置好 NFS 根目录后，我们可以让文件系统中的任何目录在该入口点之后变得可用。我们需要在`/var/exports`目录中创建子目录作为挂载点，然后将本地目标挂载到这些挂载点。我们将添加一个名为`/home/marketing`的中央共享目录，并将其与现有的`/usr/share/doc`目录一起挂载到新创建的`exports`目录后面。实现这一操作的命令如下所示：

```
$ sudo mkdir -m 1777 /home/marketing
$ touch /home/marketing/marketing.doc
$ sudo mkdir -m 777 /var/exports/{doc,marketing}
$ sudo mount --bind /usr/share/doc /var/exports/doc
$ sudo mount --bind /home/marketing /var/exports/marketing

```

以下要点解释了前述命令行步骤：

+   在执行完命令列表后，我们首先创建了一个中央共享目录，这个目录将在`/home`结构之后添加。这样做可能是由于分区和配额设置，要求`marketing`目录位于`/home`分区上。

+   我们在此目录中添加一个文档，以便查看一些内容。

+   列表中的第三条命令创建了`doc`和`marketing`目录。我们将把这些目录用作挂载点。这些目录是在`/var/exports`的 NFS 根目录中创建的。

+   最后的两条命令将本地目录挂载到其导出挂载点。通过这种方式，我们可以轻松地将任何目录直接添加到`/var/exports`之后可用。

列出`/var/exports/marketing`目录的内容应显示我们在`/home/marketing`目录中创建的文件。请参见以下截图：

![伪根](img/image00261.jpeg)

同样，查看`/var/exports/doc`的内容应该显示`/usr/share/doc`的内容。

为了确保本地挂载的持久性，我们需要将它们添加到`/etc/fstab`文件中，格式如下：

```
/originaldir /newdir none bind

```

现在我们将以 root 身份编辑`fstab`文件，并在文件末尾添加这两行，以确保在启动时挂载点被填充：

```
/usr/share/doc /var/exports/doc none bind
/home/marketing /var/exports/marketing none bind

```

当你返回到`nfsclient`系统时，可以测试这两个导出和权限。原始的`/home/marketing`目录是可写的，而`/usr/share/doc`目录则不可写：

在`nfsclient`系统上，我们可以执行以下命令：

```
$ sudo mount 192.168.10.10:/ /mnt
$ touch /mnt/marketing/file1
$ touch /mnt/doc/file1

```

现在文件路径变得更简洁，能够通过服务器的 NFS 根目录使用单一路径访问两个文件夹。我们还需要注意，尽管导出的目录是读/写的，我们可以通过第一个`touch`命令写入`marketing`目录，但第二个`touch`命令会失败，因为目标文件系统是只读的。

# 使用`exportfs`创建临时导出

在`/etc/exports`文件中创建永久导出并不总是理想的。如果您想临时定义一个新的导出，可以使用`exportfs`命令。由于我们已经将 NFS 根目录定义为`/var/exports`，因此我们导出的所有目录都必须位于该结构之后。让我们临时将`/var/export/doc`导出到所有主机。可以使用以下命令：

```
$ sudo exportfs *:/var/exports/doc

```

在下次重启`nfs-server`时，此导出将丢失；然而，如果您需要在此之前删除它，可以执行以下命令：

```
$ sudo exportfs -u *:/var/exports/doc

```

如果您需要为临时导出添加导出选项，请像以下命令所示，使用`-o`选项进行配置：

```
$ sudo exportfs *:/var/exports/doc -o ro,all_squash

```

要显示当前的导出信息，可以单独运行`exportfs`命令：

```
$ sudo exportfs

```

# 在防火墙后托管 NFSv4

当您使用 v4 协议访问 NFS 服务器时，客户端和服务器都使用 v4 协议，防火墙配置非常简单，只需要打开 TCP 端口`2049`。RHEL 7 的默认防火墙守护进程是`firewalld`，并且可以通过命令行使用`firewall-cmd`进行管理。

到目前为止，我们一直在使用标准防火墙进行演示，只打开了一个额外的端口`2049`，正如本节实验概述中所述。

我们可以使用以下命令列出当前的防火墙配置：

```
$ sudo firewall-cmd --list-all

```

输出如下截图所示：

![在防火墙后托管 NFSv4](img/image00262.jpeg)

如果需要移除我们添加的端口设置，可以使用以下命令：

```
$ sudo firewall-cmd --remove-port=2049/tcp --permanent
$ sudo firewall-cmd --reload

```

当然，客户端将无法再访问 NFS 导出。我们可以选择添加端口或服务条目。要添加服务条目，需要在`/etc/services`文件中定义端口及相关服务。可以使用`grep`命令轻松检查此文件。以下命令展示了一个例子：

```
$ grep 2049 /etc/services

```

我们确实有一个`2049`端口的条目，且该服务被称为`nfs`。若要在防火墙配置中使用服务名称，可以使用以下命令：

```
$ sudo firewall-cmd --add-service=nfs --permanent
$ sudo firewall-cmd --reload
$ sudo firewall-cmd --list-all

```

这一点在下图中有所说明：

![在防火墙后托管 NFSv4](img/image00263.jpeg)

现在，所有防火墙规则都允许该服务，我们可以继续从`nfsclient`访问 NFS 导出。如果您想远程使用`showmount`等工具，或者如果您有 NFSv3 客户端，则需要打开更多端口并静态设置一些端口。

# 在防火墙后托管 NFSv3

如果我们尝试从`nfsclient`使用`showmount`命令，我们应该能够列出远程 NFS 服务器上的导出。语法如下：

```
$ showmount -e 192.168.10.10

```

以下是命令和相应的错误截图：

![防火墙后面的 NFSv3 托管](img/image00264.jpeg)

在此阶段，我们可以选择以下选项：

+   收拾行李回家吧，也许明天会更好。

+   Google 搜索错误

+   自行调试错误

## 诊断 NFSv3 问题

现在，Google 通常在帮助我们方面做得很好，但如果你不学习故障排查技巧，那就失败了。所以让我们放弃第三个选项，安装`tcpdump`命令行数据包分析器，这样我们就能看到发生了什么。这可以通过`yum`在`nfsclient`上安装，命令如下：

```
$ sudo yum install -y tcpdump

```

要捕获`nfsclient`和`nfshost`之间的网络流量并打印被访问的端口号，我们可以使用以下命令：

```
$ sudo tcpdump -nn -i enp0s8 host 192.168.10.10

```

这里使用的`tcpdump`选项列出如下：

+   `-nn`：此选项显示主机的 IP 地址和端口号，而不是其名称。

+   `-i`：这是要使用的接口。你需要使用正确的接口名称，我们使用的是`enp0s8`，这是我们需要监听的接口。

+   `host 192.168.10.10`：这显示了该主机的进出流量。这是`nfshost`的 IP 地址。

在另一个控制台或 SSH 会话中，再次尝试`showmount`命令。与此同时，在运行`tcpdump`的控制台中，我们应尝试两次访问服务器上的 UDP 端口`111`并报告错误。以下截图展示了我系统的输出：

![诊断 NFSv3 问题](img/image00265.jpeg)

UDP 端口`111`在`nfshost`的防火墙配置中未开放。如果你还记得，我们刚才显示了防火墙允许的服务和端口，而`111`不在其中。

端口`111`由`rpcbind`运行的`portmapper`服务保持开放，并在`/etc/services`文件中显示为`sunrpc`服务。我们可以通过在`nfshost`上运行`netstat`来检查此项，命令如下：

```
$ sudo netstat -aunp

```

这里使用的`netstat`选项列出如下：

+   `-a`：此选项显示所有端口，包括监听和已建立的端口

+   `-u`：此选项仅显示 UDP 端口

+   `-n`：此选项显示端口和网络号，而不是将其解析为名称

+   `-p`：此选项显示保持端口或连接开放的进程名称

要使用`-p`选项，我们必须以 root 身份运行（使用`sudo`）；否则，进程列将保持空白。

`rpcbind`服务的原理是它会返回端口地址，以便所需的服务能够对请求的客户端提供服务。这就是 NFSv3 的工作原理，而`showmount`命令仍然使用这个旧协议。来自远程客户端的`showmount`请求要求 NFS 挂载守护进程的地址。这是作为进程`rpc.mountd`运行的服务。这些服务可以运行在动态分配的端口上。因此，它需要进一步配置，以确保它们能够长期稳定地通过防火墙。

`showmount`的处理过程如下图所示，首先是请求`rpc.mountd`端口：

![诊断 NFSv3 问题](img/image00266.jpeg)

我们可以从允许防火墙到`nfshost`的`rpcbind`流量开始：

```
$ sudo firewall-cmd --add-port=111/udp --permanent
$ sudo firewall-cmd --reload

```

### 提示

别忘了在添加端口后重新加载防火墙。完成这个步骤是很容易被忽略的。

现在我们可以连接到运行在 UDP 端口`111`上的`rpcbind portmapper`服务，接下来我们应该继续操作。记住，我们实际上是想调试这个过程，并学习一些有用的`tcpdump`分析技巧。我们可以重复前面的操作，一边在控制台上运行`tcpdump`，另一边在控制台上运行`showmount`（这两个控制台都运行在`nfsclient`上）。`showmount`命令报告的错误现在应该略有不同。为了说明这一点，以下截图显示了当前的错误，其中 UDP 端口`111`是开放的：

![诊断 NFSv3 问题](img/image00267.jpeg)

现在，错误略有不同，我们不再看到错误代码；但是，我们可以从`tcpdump`的输出中看到已经收到了来自`nfshost`的回复。随后，我们尝试重新连接到`20048`端口上的主机。

要识别此端口的用途，我们可以再次使用`netstat`，但这次我们将`-u`替换为`-t`，因为我们想要显示 TCP 端口。由于我们只需要查看监听端口，可以将`-a`替换为`-l`：

```
$ sudo netstat -ltnp

```

我们应该会看到我们尝试连接的端口是由`rpc.mountd`打开的。当然，这个端口不能通过防火墙访问。

### 提示

`rpc.mountd`监听的端口可能与您的系统上使用的端口不同，因此需要调整练习以适应您系统上的`rpc.mountd`端口以及客户端正在使用的端口。

从`tcpdump`获取的输出如下所示。我们可以通过附加的属性（如序列号`seq`和窗口大小`win`）将其识别为 TCP 流量，这些属性在下图中已被突出显示：

![诊断 NFSv3 问题](img/image00268.jpeg)

现在我们可以看到，我们还需要打开 NFS 服务器上的 TCP 端口`20048`，通过防火墙进行通信；记住，该端口在你的`nfshost`上可能不同；我们可以通过在`nfshost`上使用`firewall-cmd`快速解决这个问题，操作如下：

```
$ sudo firewall-cmd --add-port=20048/tcp --permanent
$ sudo firewall-cmd --reload

```

现在，我们可以回到`nfsclient`，因为`showmount`命令应该已经能够正常工作，如下图所示：

![诊断 NFSv3 问题](img/image00269.jpeg)

## 使用静态端口进行 NFSv3

`portmapper` 服务是必需的，它用于在非静态端口上操作的服务，包括 `rpc.mountd` 和其他基于 NFSv3 的服务。虽然配置 NFSv4 很简单，因为我们只需要将 TCP 端口 `2049` 作为防火墙的唯一要求来访问，但使用 v3 时，我们仍然需要访问更多的端口，而这些端口大多数是非静态的。不过，通过 `/etc/sysconfig/nfs` 文件可以提供帮助，在该文件中我们可以添加条目来为这些服务启用静态端口。RHEL 6 的配置与此不同。搜索引擎常常因提供过时文档而让你失望，其中包括 RHEL 7 的文档，它并未更新。在这里，我们展示了你在 `/etc/sysconfig/nfs` 文件中设置静态端口所需的正确设置。

当你以 root 用户在 `nfshost` 上工作并使用你选择的文本编辑器时，你需要编辑以下行：

```
RPCRQUOTADOPTS="-p 30001"
LOCKD_TCPPORT=30002
LOCKD_UDPPORT=30002
RPCMOUNTDOPTS="-p 30003"
STATDARG="-p 30004"

```

使用的端口是名义上的，你应选择在系统中未使用的端口。你可以看到一些服务使用 `-p` 选项来指定端口。`rpc.lockd` 工具有一个实际的端口配置。这是 RHEL 6 配置所有端口的方式，但在 RHEL 7 中已发生变化。

我们需要重新启动服务，可以单独重启它们，或者重新访问我们之前使用的 `for` 循环。编辑后的循环如下所示：

```
for s in rpcbind nfs-server nfs-lock nfs-idmap ; do
 sudo systemctl restart $s
done

```

我们现在已经配置了 `nfshost`，使其使用这些通常会因动态端口而导致问题的 NFS 服务的静态端口。我们仍然需要在防火墙规则中配置 UDP 端口 `111` 以允许访问 `portmapper`，但我们现在知道将为其他服务返回的端口，并且这些端口可以添加到配置中。使用我们配置的端口的最终 NFSv3 防火墙配置如下所示：

```
sudo firewall-cmd --add-port=111/udp --permanent
sudo firewall-cmd --add-port=2049/tcp --permanent
sudo firewall-cmd --add-port=30001/tcp --permanent
sudo firewall-cmd --add-port=30001/udp --permanent
sudo firewall-cmd --add-port=30002/tcp --permanent
sudo firewall-cmd --add-port=30002/udp --permanent
sudo firewall-cmd --add-port=30003/tcp --permanent
sudo firewall-cmd --add-port=30003/udp --permanent
sudo firewall-cmd --add-port=30004/tcp --permanent
sudo firewall-cmd --add-port=30004/udp --permanent
sudo firewall-cmd --reload

```

如果我们想要完全测试 NFSv4 配置，你需要从现有的导出定义中移除 `crossmnt` 和 `fsid` 选项，因为这些是 v4 选项。

## 配置 NFS 客户端

在从客户端挂载文件系统时，默认实现的协议是 NFSv4 在 RHEL 7 中。我们可以使用 `-t` 选项将协议显式设置为 v3 或 v4，如下所示：

```
$ sudo mount -t nfs4 192.168.10.10:/var/exports /mnt  #NFS 4
$ sudo mount -t nfs 192.168.10.10:/var/exports /mnt    #NFS 3

```

在下面的截图中，你可以看到我们能够从 `nfsclient` 使用 NFSv4 或 NFSv3 进行连接：

![配置 NFS 客户端](img/image00270.jpeg)

其他挂载选项可以通过 `-o` 选项应用于 mount 命令。你可以考虑以下命令选项：

+   `bg`：将挂载过程放到后台

+   `rsize=xxxx`：指定最大读取请求大小（以字节为单位）

+   `wsize=xxxx`：指定最大写入缓冲区大小（以字节为单位）

要了解更多 NFSv3 和 NFSv4 的挂载选项，可以通过适当的手册页详细阅读，命令行如下所示：

```
$ man 5 nfs

```

# 自动挂载 NFS 使用 autofs

有一个名为`autofs`的客户端服务，它充当本地和远程文件系统的自动挂载服务。它与内核模块和用户空间服务一起工作；当您进入某个目录时，挂载会自动创建。如果需要进行 NFS 挂载，还需要安装`autofs`软件包以及`nfs-utils`软件包。自动挂载功能不仅适用于 NFS，还可以与其他远程文件系统一起工作。要安装`autofs`，请使用以下命令：

```
$ sudo yum install -y autofs

```

安装完成后，默认行为是使用`/net`目录点来指向网络主机。然后，我们可以访问任何有权限访问的主机上的共享或导出目录，并在`/net`目录后输入与服务器名称或 IP 地址匹配的目录。我们只需要创建顶级目录，无需创建子目录。我们只需将目录更改为`/net/192.168.10.10`，该目录将自动创建。列出该目录的内容时，将列出`nfshost`上的导出根目录。这听起来可能过于完美，但让我们实际演示一下。首先，我们将创建目录，然后启动服务并启用它，如下命令所示：

```
$ sudo mkdir /net
$ sudo systemctl start autofs
$ sudo systemctl enable autofs

```

配置完成后，我们可以简单地列出`/net/192.168.10.10`目录的内容。我们应该会看到导出配置的顶级目录。对于我们来说，当前是`/var`目录，导出目录是`/var/export`。如果我们有更多导出的顶级目录，它们也会显示出来。`/net/192.168.10.10`目录会自动创建，且`autofs`的默认超时是 300 秒（即 5 分钟）。在 5 分钟不活动后，已挂载的文件系统将自动卸载，目录将消失，直到再次需要时才会出现。这是一个典型的安全值；不过，可以配置特定的超时设置。稍后我们将看到如何操作。

以下截图显示了按顺序执行的四个命令以及临时自动挂载目录的列出内容：

![使用 autofs 自动挂载 NFS](img/image00271.jpeg)

根据需要在客户端自动挂载目录，减少了客户端和服务器的负担，这是一种非常有效的挂载方式。要定义我们自己的挂载点，我们可以编辑`/etc/auto.master`配置文件。我们将像之前一样添加一个顶级目录。

在`/etc/auto.master`文件中，我们将添加以下命令：

```
/corp /etc/auto.corp --timeout=600

```

`auto.master`文件中的此设置告诉`autofs`服务，当进入`/corp`目录时，可以从`/etc/auto.corp`文件读取配置。此外，我们还将默认超时设置增加到 10 分钟，用于此自动挂载。我们将需要按以下方式创建顶级目录：

```
$ sudo mkdir /corp

```

对于该目录的配置文件，在我们的案例中应如下所示：

```
redhat -fstype=nfs4,rsize=4096,wsize=4096 192.168.10.10:/var/exports

```

通过这个条目，我们将能够看到服务器导出的内容，并进入`/corp/redhat`目录。我们不会创建`redhat`子目录。在测试之前，你需要重启`autofs`服务：

```
$ sudo systemctl restart autofs

```

现在，我们可以访问`/corp`目录，它会是空的。以下是截图展示：

![自动挂载 NFS 与 autofs](img/image00272.jpeg)

如果我们现在访问`redhat`目录，它还没有显示出来；但我们将能够列出服务器导出的内容。以下是截图展示：

![自动挂载 NFS 与 autofs](img/image00273.jpeg)

我已经使用这个好几年了；它仍然是 Linux 上最神奇的体验之一。

# 总结

我希望你能觉得这一章既紧凑又实用。内容比较多，而且由于需要同时讲解 NFSv4 和 NFSv3，内容变得有些复杂。像大多数技术一样，遗留客户端在一段时间内仍然需要得到支持。这也给我们带来了一个巨大优势，那就是可以通过`tcpdump`有效诊断防火墙问题。

使用 NFS 和防火墙的关键是尽可能使用 NFSv4，因为这样我们只需要打开一个静态端口：TCP 端口`2049`。而对于 NFSv3，我们需要分配静态端口，并且通常需要为每个协议同时打开 UDP 和 TCP 端口，具体取决于连接的客户端。

完成`autofs`这一章是一个真正的高光时刻，因为它非常简单且有效，能够根据需要自动创建目录并挂载它们。还有什么可以更好的呢！

在下一章，我们将继续探讨文件共享，但会研究如何使用 Samba 4 将共享内容提供给 Windows 系统。
