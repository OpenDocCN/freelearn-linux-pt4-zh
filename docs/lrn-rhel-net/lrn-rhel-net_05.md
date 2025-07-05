# 第五章：实现 `btrfs`

在本章中，我们将探讨 `btrfs`（发音为 *Better FS*）提供的功能。尽管与网络无关，我们很快会讨论如何共享文件系统；正因为如此，且 `btrfs` 如此强大，我们现在就来看看它。`Btrfs` 是一个本地文件系统，提供集成的卷管理操作、易于扩展以及内置容错功能。它并未完全得到 Red Hat 的支持，并作为技术预览发布；必须说，Red Hat 对此非常谨慎，因为 SUSE 从 Enterprise Linux 11 SP2 开始便将 `btrfs` 作为默认文件系统，并且在 SLES 12 上继续使用。

本章将涵盖以下主题：

+   `btrfs` 概述

+   实验环境概述

+   创建 `btrfs` 文件系统

+   写时复制技术

+   调整 `btrfs` 文件系统大小

+   向 `btrfs` 文件系统添加设备

+   从 `/etc/fstab` 挂载多磁盘 `btrfs` 卷

+   使用 `btrfs` 实现 RAID

+   优化固态硬盘

+   使用快照进行时间点数据备份

+   使用 snappers 进行快照管理

# `btrfs` 概述

如果现在有一样东西是 Linux 能够提供的，那就是拥有超过 55 个基于内核的文件系统，在 Linux 内核树上。那我们为什么还需要更多呢？我们已经看到，像 `xfs` 这样的老旧文件系统正在复苏，Red Hat 正在大力推广这个来自 SGI 的原始文件系统。`btrfs` 文件系统提供了一种独特的解决方案，将卷管理和文件系统合并为一个统一的解决方案。`Btrfs` 采用 **通用公共许可证**（**GPL**）授权，并作为标准随 Red Hat Enterprise 7 和 7.1 提供。它不仅提供文件管理访问，还提供卷管理和 **冗余廉价磁盘阵列**（**RAID**）管理。这种简单的管理方式意味着你可以通过单个命令创建 RAID 设备或扩展卷，而不需要依赖 LVM 进行逻辑卷管理或使用 `mdadm` 进行 RAID 管理。可扩展性也是选择 `btrfs` 的一个重要因素。它可以扩展到 16 EB（艾字节），并带来了以下以前未见过的可靠性特性：

+   非常快速的文件系统创建

+   数据和元数据校验和

+   快照功能

+   在线清理以修复问题

当你看到像 Facebook 和 TripAdvisor 等使用 `btrfs` 作为生产环境的组织时，你就会理解将它纳入本书的重要性。

在许多方面，`btrfs` 文件系统诞生于 ReiserFS 文件系统的失败，该文件系统在失去其首席开发者 Hans Reiser 后陷入困境。Chris Mason 曾在 SUSE 转向开发高端文件系统前参与了 ReiserFS 的开发，他被 Oracle 雇佣来开发高端文件系统。`btrfs` 的诞生正是从此开始的。

# 实验环境概述

本书中使用的 Red Hat Enterprise Linux 7.1 虚拟机将为本节添加额外的磁盘。目前，我们将使用三块磁盘：

+   `/dev/sda`：此磁盘用于根文件系统

+   `/dev/sdb`：此磁盘用于存储 yum 仓库

+   `/dev/sdc`：此磁盘作为 iSCSI LUN 存储

为了演示`btrfs`的关键功能，我们将向系统添加四个额外的虚拟磁盘，以便在演示`btrfs`文件系统时使用。如果你使用的是虚拟化系统，也可以进行相同的操作。我们将添加的不同磁盘如下：

+   `/dev/sdd`

+   `/dev/sde`

+   `/dev/sdf`

+   `/dev/sdg`

在演示系统上使用`lsblk`命令，你将能够查看我们从此时起将要使用的初始配置，正如以下截图所示：

![实验环境概览](img/image00239.jpeg)

# 安装 btrfs

使用 Red Hat Enterprise Linux 7 或更高版本时，即使是最小安装，`btrfs`也会默认安装。然而，如果你使用的是早期版本，可以像通常一样通过 yum 安装`btrfs`文件系统，如下所示：

```
# yum install -y btrfs-progs

```

文件系统安装完成后，我们可以使用以下命令检查已实现的版本：

```
$ btrfs --version

```

在 RHEL 7 中，版本为`3.12`，而在 RHEL 7.1 中，版本为`3.16.2`。

现在我们对`btrfs`背后的强大功能有了些许了解，接下来让我们开始一些简单的实现示例。

# 创建 btrfs 文件系统

首先，我们将在`/dev/sdd`整个磁盘上创建一个`btrfs`文件系统。我们无需先分区，从一开始就节省了时间。以下是命令行展示：

```
# mkfs.btrfs /dev/sdd

```

文件系统创建完成后，我们可以花些时间熟悉完整性检查工具：

```
# btrfsck /dev/sdd

```

以下截图显示了我的系统的输出：

![创建 btrfs 文件系统](img/image00240.jpeg)

为了验证`btrfs`文件系统是否在运行，我们将创建一个目录并挂载它。我们还将复制一些数据，并显示磁盘的使用情况信息：

```
# mkdir -p /data/simple
# mount /dev/sdd /data/simple
# find /usr/share/doc -name '*.pdf' -exec cp {} /data/simple \;
# btrfs filesystem show /dev/sdd

```

最后一个命令的输出显示在以下截图中。我们可以看到，已经使用了 5.96 MiB 的文件空间：

![创建 btrfs 文件系统](img/image00241.jpeg)

额外使用的空间（显示为`138.38MiB`）包括与任何文件系统相关的典型元数据，但另外，默认情况下，`btrfs`文件系统将空闲空间信息存储在磁盘上，以便快速检索，而不是扫描磁盘。这是通过`space_cache`挂载选项控制的，默认情况下已经设置。如果你希望禁用此功能，可以使用`nospace_cache`挂载选项。

# 写时复制技术

`btrfs` 文件系统成功的一个基础技术是 **写时复制**（**CoW**）。`CoW` 被用在逻辑卷管理文件系统中，包括 Solaris 中使用的 **ZFS**（一个 Oracle 产品）、微软的 **卷影复制**（Volume Shadow Copy）和 `btrfs`。

这些 CoW 文件系统允许你进行即时快照或备份。这是因为在写入文件时会创建其副本，因此称为写时复制（Copy-on-Write）。当传统文件系统实现此功能时，虚拟磁盘技术也可以在 `qcow2` 中实现这一 `CoW` 技术。这样，`qcow2` 磁盘文件中分配的任何磁盘空间，在写入之前不会在主机上使用。

对于通用文件系统，你会发现 `CoW` 技术非常有用。能够恢复到先前的文件版本，就像传统文件服务器上的黄金一样。然而，如果你使用 `btrfs` 来存储非常大的数据文件，比如虚拟磁盘文件，`CoW` 技术可能会导致写入速度变慢。

在 Linux 中使用 `chattr` 命令，我们可以设置或更改文件和/或目录的属性。`btrfs` 文件系统支持一个禁用 CoW 的文件属性。这个属性仅在设置为空文件时有效。为了确保其有效性，我们通常将此属性设置在目录上，以便所有文件在创建时都会继承此属性。以下命令展示了如何实现这一点：

```
# mkdir /data/simple/cow
# chattr +C /data/simple/cow
# lsattr -d /data/simple/cow
# touch /data/simple/cow/vdisk1
# lsattr /data/simple/cow/vdisk1

```

在以下截图中，我们可以看到，创建新文件时会自动分配 `NoDataCoW` 选项。无论该文件是如何创建的，这一点都不重要：

![写时复制技术](img/image00242.jpeg)

# 调整 btrfs 文件系统大小

使用 `btrfs` 时，当文件系统在线并且被用户访问时，可以调整 `btrfs` 文件系统的大小。如果我们添加或移除设备，文件系统的大小会自动增长；我们将在本章的下一小节中看到这一点；然而，即使在我们创建的单个设备上，我们也可以根据需要调整文件系统的大小。使用以下命令，我们将文件系统分配的空间缩小 500MiB：

```
# btrfs filesystem resize -500m /data/simple

```

如果我们检查文件系统大小的变化，可以看到动态变化的过程：

![调整 btrfs 文件系统大小](img/image00243.jpeg)

# 向 btrfs 文件系统添加设备

我们已经在第四章《实现 iSCSI SANs》中略微了解了使用 LVM 的卷管理，*实现 iSCSI SANs*，它并不简单。

## 传统方式的卷管理

以下命令用于以传统的方式管理磁盘卷：

```
# pvcreate /dev/sde1
# vgextend vg1 /dev/sde1
# lvextend -L+1000M /dev/vg1/data_lv
# resize2fs /dev/vg1/data

```

## 使用 btrfs 进行卷管理

首先，我们将卷恢复到原始大小，然后再添加第二个磁盘。使用 `max` 选项，我们将确保 `btrfs` 文件系统使用我们目前拥有的单个磁盘上的最大可用空间：

```
# btrfs filesystem resize max /data/simple

```

在 LVM 和传统文件系统中，需要执行四个命令。在`btrfs`中，我们可以用一个命令来完成这一切：

```
# btrfs device add /dev/sde /data/simple

```

这就是我们需要做的。设备被添加后，文件系统会自动扩展到可用的最大空间。我们可以使用`btrfs filesystem show`命令来检查`/dev/sdd`或`/dev/sde`，因为默认情况下，这两个设备都会保存元数据的副本。在以下命令中，我们可以看到这一点，截图也会进一步验证这个信息：

```
# btrfs filesystem show /dev/sdd
# df -hT /data/simple

```

在查看以下截图后，我们可以看到生成的命令和输出：

![使用 btrfs 进行卷管理](img/image00244.jpeg)

将元数据存储在两个设备上可实现容错，并减弱了对设备的查询需求：

```
# btrfs fi show /dev/sdd
# btrfs filesystem show /dev/sde

```

### 提示

请注意，一些子命令可以缩短；在这种情况下，`fi`相当于文件系统。

# 平衡 btrfs 文件系统

如果添加额外磁盘到卷中的原因是磁盘空间不足，那么我们可以选择通过将数据分散到两个设备来提升性能。这是通过使用`balance`子命令来实现的：

```
# btrfs filesystem balance start -d -m /data/simple

```

`-m`参数代表元数据，`-d`代表数据。这样，磁盘就能以相等的比例使用。

演示系统的输出在以下命令中显示；请注意，您可以在`balance`子命令中省略`filesystem`，因为在这种情况下它是可选的：

![平衡 btrfs 文件系统](img/image00245.jpeg)

# 从`/etc/fstab`挂载多磁盘 btrfs 卷

如果我们是从`/etc/fstab`文件挂载`btrfs`卷，则需要确保在挂载`/data/simple`目录之前执行`btrfs`扫描。这将定位所有参与卷的设备。`initramfs`文件系统可以在稍后的系统中为我们完成此任务，包括 RHEL 7。如果您的现有文件系统已经使用了`btrfs`，那么扫描将被内建在您当前的`initramfs`中。如果`btrfs`是您的新系统，您需要生成一个新的初始 RAM 磁盘。运行以下命令时，请确保使用与您的系统相匹配的`initramfs`和内核版本：

```
# dracut -v -a btrfs -f /boot/initramfs-$(uname -r) /boot/vmlinuz-$(uname -r)

```

然后，我们可以在`/etc/fstab`文件中添加类似以下的条目：

```
/dev/sdd  /data/simple  btrfs  defaults  0 0

```

## 创建 RAID1 镜像

**RAID**（**廉价磁盘冗余阵列**）软件也得到了`btrfs`的支持。以下是当前支持的 RAID 级别：

+   **RAID 0**：没有冗余的条带化

+   **RAID 1**：磁盘镜像

+   **RAID 10**：条带化镜像

目前，我们有一个多磁盘的`btrfs`文件系统，但没有容错功能。我们使用的实现方式是 RAID 0 / 条带化，没有奇偶校验。我们可以将其转换为 RAID 1 系统，并通过以下方式镜像元数据和文件系统数据：

```
# btrfs balance start -dconvert=raid1 -mconvert=raid1 /data/simple

```

正如您从上面的命令中看到的，元数据和文件系统数据已被转换为 RAID 1 的软镜像。

我们可以很容易且快速地使用`btrfs`从一开始就创建一个镜像设备。镜像不会为我们提供额外的磁盘空间，但如果发生最坏情况，比如磁盘故障，它可以提供很好的容错性。我们可以使用目前未使用的额外磁盘在演示系统上展示这一点：

```
# mkfs.btrfs -m raid1 -d raid1 /dev/sdf /dev/sdg
# mkdir /data/mirror
# mount /dev/sdg /data/mirror

```

要创建一个镜像，我们将使用 RAID1 来处理元数据和`-m`、`-d`数据，正如我们在前面的转换示例中所做的那样。可用的磁盘空间是 1 GB。我们写入`/dev/sdf`的数据会被镜像到`/dev/sdg`；使用镜像时，我们会失去 50%的数据存储空间，但却获得了高度的冗余。我们同样需要在`/etc/fstab`文件中添加一项条目，以确保 RAID 系统在启动时能正确挂载。由于`initramfs`现在通过设备扫描来支持`btrfs`，因此在此阶段不需要创建`initramfs`：

```
/dev/sdf  /data/mirror  btrfs  defaults  0 0

```

使用标准工具（如`df`）显示空闲磁盘空间不会提供正确的信息；我们需要使用`btrfs`工具。以下命令将列出`/data/mirror`挂载点的可用空闲空间：

```
# btrfs fi df /data/mirror

```

以下截图显示了命令的输出：

![创建 RAID1 镜像](img/image00246.jpeg)

我知道，甚至谈论它都会带来七年的霉运；然而，镜像确实可能会损坏。创建镜像的部分原因是为了提供容错能力。这本身就是对硬盘可能出现故障的接受。

在这个演示中，我们将销毁`/data/simple/`卷，并重新使用之前用于简单卷的设备。为了销毁`btrfs`元数据，推荐使用`wipefs`，它是`util-linux`包的一部分。首先，我们需要运行`wipefs`命令，作用于我们需要清除的磁盘或分区，然后使用`-o`选项指定偏移量。看看如何清除`/dev/sdd`和`/dev/sde`：

```
# umount /data/simple
# wipefs /dev/sdd
# wipefs -o 0x10040 /dev/sdd
# wipefs /dev/sde
# wipefs -o 0x10040 /dev/sde

```

第一个磁盘的输出如下所示，方便起见，截图中列出了此输出；第二个磁盘的操作会重复进行。别忘了从`/etc/fstab`文件中删除相应条目：

![创建 RAID1 镜像](img/image00247.jpeg)

清除这些磁盘后，我们可以将它们重新用于其他阵列。

我们将以与简单卷相同的方式向镜像卷中添加数据。通过这种方式，我们可以确保数据保持完整：

```
# find /usr/share/doc -name '*.pdf' -exec cp {} /data/mirror \;

```

我们现在将卸载镜像卷，并模拟其中一个磁盘的故障，具体步骤如下：

```
# umount /data/mirror
# wipefs -o 0x10040 /dev/sdg

```

当我们尝试使用 mount 命令重新挂载镜像卷时，会遇到问题，我们需要使用`-o`降级选项挂载镜像卷：

```
# mount -o degraded /dev/sdf /data/mirror

```

在此阶段，我们的数据是可用的，因此可以松一口气：

```
# ls /data/mirror

```

我们仍然有一个 RAID 1 阵列，创建这个阵列的最少成员数量是两个，因此我们需要按如下方式添加一个新设备：

```
# btrfs device add /dev/sdd /data/mirror

```

现在我们可以移除故障或缺失的设备：

```
# btrfs device delete missing /data/mirror

```

`missing` 关键字将搜索数组中第一个缺失的成员。我们可以删除此设备。RAID 1 阵列现在已经完全投入使用，并通过两个设备重新提供软件镜像。

# 使用 btrfs 快照

希望到目前为止，您在 `btrfs` 中看到的内容能引起您的兴趣，但当然，总有更多的内容等待您去探索和学习。接下来，我们将讨论快照。Btrfs 快照可以作为数据的只读或读写副本使用。由于 `btrfs` 是基于写时复制（Copy-on-Write）的文件系统，因此无需复制大量数据，因为我们只需在数据发生变化时复制它。与此同时，原始数据会链接到新位置。通过这种方式，可以瞬间创建大型文件系统的快照。快照可以通过以下几种方式使用：

+   作为备份解决方案的一部分，您可能担心打开的文件会影响备份；快照将以只读方式创建。随后，您将实施对快照的备份。通过这种方式，备份将是快照创建时的主机文件系统状态。

+   快照在需要恢复到原始数据时非常有用，特别是在测试环境中，您可能需要进行大量更改，并能够非常快速地恢复到原始数据。

Btrfs 快照依赖于子卷；源子卷和目标子卷必须位于同一文件系统内。如果您还记得，数据仅在发生变化时才会被复制；这与传统硬链接的处理方式相同。

`btrfs` 文件系统中的子卷是离散的管理单元，使得对单一文件系统的元素能够更精细地控制。我们将首先创建一个单一的子卷，以便在创建快照之前对这项技术有一些了解。我们将重新使用 `/dev/sde` 磁盘作为我们的简单卷，并从头开始重新格式化镜像卷：

```
# mkfs.btrfs /dev/sde
# mount /dev/sde /data/simple

```

此时，`/dev/sde` 的完整文件系统已可用，并挂载在 `/data/simple` 目录下。这里尚未存储任何数据，但我们实际上已经通过简单目录拥有了文件系统的单一视图。子卷使您能够通过将文件系统的元素（子卷）挂载到我们选择的目录并使用适当的挂载选项，以不同的方式查看同一个文件系统。

我们将在现有的 `/data/simple` 目录后创建一个新的子卷：

```
# btrfs subvolume create /data/simple/vol1

```

输出非常简洁，如下图所示：

![使用 btrfs 快照](img/image00248.jpeg)

我们可以列出子卷，如以下命令和截图所示：

```
# btrfs subvolume list /data/simple
# ls /data/simple

```

以下截图显示了前述命令的输出：

![使用 btrfs 快照](img/image00249.jpeg)

我们还可以看到，创建子卷也在文件系统内创建了目录。由于这不仅仅是一个目录，还是一个子卷，我们将无法从文件系统中删除该目录。要删除目录，你需要删除子卷。

我们不会删除该目录，但如果以后需要删除它，删除命令如下：

```
# btrfs subvolume delete /data/simple/vol1

```

这将删除子卷及其中的目录，方式与创建子卷时也在文件系统内创建目录的方式非常相似。

现在我们将向子卷添加一些数据；如果你删除了它，可以重新创建它。我们可以将已经熟悉的 PDF 文件复制到这个卷中：

```
# cp /data/mirror/* /data/simple/vol1

```

如果我们需要将这些数据提供到其他地方，我们可以将子卷挂载到任何需要的位置，并使用我们认为合适的挂载选项。例如，我们在这个目录中有文档，因此可以将其以只读方式挂载到另一个目录：

```
# mount -o ro,subvol=vol1 /dev/sde /mnt

```

在`/mnt`挂载点的根目录下，我们将看到我们添加到`vol1`目录中的 PDF 文件。它们仍然可以在原始位置`/data/simple/vol1`中访问。通过这种方式，我们可以通过挂载的方式控制对数据的访问。

现在我们已经了解了子卷，接下来我们将研究快照。快照必须在与目标数据相同的文件系统中创建；如前所述，快照的即时生成受到文件系统内一种内部链接形式的影响。

我们将生成现有`vol1`数据的快照，并指定`-r`选项以确保备份为只读。通过这种方式，我们可以通过将数据从`backup`目录复制回来，恢复到这个*时间点*的备份。除非原始数据被更改，否则不会使用额外的磁盘空间：

```
# btrfs subvolume snapshot -r /data/simple/vol1/ /data/simple/backup

```

我们可以使用以下命令轻松列出子卷：

```
# btrfs subvolume list /data/simple

```

我们可能会将备份场景建立在文档可能频繁写入的事实基础上。此外，我们希望有一个解决方案，能够迅速恢复不当编辑带来的损失。

要创建工作子卷的只读快照，请使用以下命令：

```
# btrfs subvolume snapshot -r /data/simple/vol1 /data/simple/backup/

```

列出工作目录和备份目录的内容应该会发现它们的内容是相同的：

```
# ls /data/simple/vol1
# ls /data/simple/backup

```

`backup`的名称并不重要，但在其使用的上下文中是有用的。像往常一样，一个好的命名方案有助于理解目录的用途，而不是像我们给`vol1`取的名称那样。

如果我们不小心删除了`/data/simple/vol1`中的所有文件，`btrfs`中的 CoW 技术会将变更后的数据写入备份快照`/data/simple/backup`。如果文件以任何方式被修改而不是删除，也会发生这种情况；快照会保留创建快照时的文件状态。在发生灾难时，我们可以简单地将文件复制回原始位置。

目前，我们将看看如何删除此快照。本章后面，我们将看到如何在 LVM 和 `btrfs` 系统上使用 snapper 作为简单的机制来管理快照：

```
# btrfs subvolume delete /data/simple/backup

```

# 优化用于固态驱动器的 btrfs

在多个 SSD 上创建 `btrfs` 文件系统时，使用单一 `-m` 选项将确保元数据不会重复。在 SSD 上，重复元数据被认为是一种空间浪费，并且有一个会减少磁盘寿命的开销，如以下代码所示：

```
# mkfs.btrfs -m single /dev/sdb

```

第二种方法是使用 `ssd` 挂载选项。此选项将设置一些性能选项：

+   允许大的元数据簇

+   允许更多的顺序数据分配

+   禁用叶写入以匹配 b 树数据库中的键和块顺序

+   提交 b 树日志片段而不批处理多个进程

# 使用 snapper 管理快照

快照命令包含在 RHEL 7 中，可用于轻松管理快照并查看其与原始数据的差异。它可以与 LVM 或 btrfs `systems.h` 一起使用。

要安装 snapper，我们回到 RHEL 的包管理：

```
# yum install snapper

```

目前，如果 SELinux 强制执行，似乎存在一个防止 snapper 工作的 bug 或功能。我们可以通过创建一个新的策略或简单地将 `snapperd_t` 设置为宽容域来允许正确的 SELinux 访问我们的资源。这样，我们仍然可以使用 SELinx 的强大安全性，但只是对 snapper 禁用如下：

```
# semanage permissive -a snapperd_t

```

以后，您可以使用 `-d` 选项来删除启用的 snapper 和 SELinux 支持：

```
# semanage permissive -d snapperd_t

```

目前，我们将把 snapper 留在宽容模式，并继续为 snapper 和我们的 `/data/simple/vol1` 数据创建配置：

```
# snapper -c simple_data create-config -f btrfs /data/simple/

```

使用以下命令，我们可以列出我们拥有的配置：

```
# snapper list-configs

```

以下截图显示了配置和列表命令的创建：

![使用 snapper 管理快照](img/image00250.jpeg)

创建配置将在根目录 `/data/simple/vol1` 下创建一个隐藏目录 `.snapshots`。配置本身存储在 `/etc/snapper/configs` 中；来自 `/var/log/snapper.log` 的故障排除日志文件存在。

现在我们已经创建了基础，我们将创建快照：

```
# snapper --config simple_data create --description "Start"

```

我们可以看到这个过程非常简单、快速，并且节省了我们大量的精力。如果我们检查现在在 `/data/simple` 后存在的子卷，我们会看到 `.snapshots` 和此后的编号子卷：

```
# btrfs subvolume list /data/simple

```

输出显示在以下截图中：

![使用 snapper 管理快照](img/image00251.jpeg)

更容易和通常情况下，我们完全使用 snapper 来管理这一点，并且我们应该使用以下命令查看快照：

```
# snapper --config simple_data list 

```

为了展示我们如何查看数据的差异，我们将从原始 `vol1` 位置删除一个 PDF 文件：

```
# rm /data/simple/vol1/tutorial.pdf

```

删除这个文件后，我们将拥有原始数据和快照之间的差异。CoW（写时复制）系统会将已删除的文件写入快照位置，正如删除操作所发生的那样。我们可以使用以下命令查看数据的差异，其中`0`代表原始数据，`1`代表快照：

```
# snapper -c simple_data status 0..1

```

命令的输出如下图所示，表明现在快照中已经包含了这个额外的文件：

![使用 snapper 管理快照](img/image00252.jpeg)

为了恢复已删除的文件，我们将使用`undochange`命令；请注意，我们需要显示从快照到原始数据或`1..0`的效果，如以下命令所示：

```
# snapper -c simple_data undochange 1..0

```

我们现在可以在`vol1`目录中找到返回给我们的`tutorial.pdf`文件，如下所示：

```
# ls /data/simple/vol1/tutorial.pdf

```

从下图中，您将能够看到文件恢复命令和返回文件的列表：

![使用 snapper 管理快照](img/image00253.jpeg)

# 总结

在本章中，我们了解了使用`btrfs`文件系统能够释放的强大功能，以及与其他 Linux 逻辑卷系统（如 LVM）相比，使用它可以节省的时间。我们还看到了如何实现软件 RAID，并将文件管理、逻辑卷管理和 RAID 管理整合为一个命令。

使用 snapper 帮助管理快照在 LVM 和`btrfs`系统中都能很好地工作。本章中我们使用了 snapper 与`btrfs`文件系统。

在下一章中，我们将学习如何使用**NFS**（**网络文件系统**）共享文件，这是传统 UNIX 方式在网络上共享文件资源的方式。
