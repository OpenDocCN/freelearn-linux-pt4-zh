# 第四章：迁移 KVM 实例

在本章中，我们将演示以下 libvirt KVM 迁移概念：

+   使用 iSCSI 存储池进行手动离线迁移

+   使用 GlusterFS 共享卷进行手动离线迁移

+   使用共享存储通过 virsh 命令进行在线迁移

+   使用 virsh 命令和本地镜像进行离线迁移

+   使用 virsh 命令和本地镜像进行在线迁移

# 介绍

迁移 KVM 实例是将客户虚拟机的内存、CPU 状态以及与之附加的虚拟化设备的状态发送到不同服务器的过程。迁移 KVM 实例是一个相对复杂的过程，具体取决于虚拟机使用的后端存储（即目录、镜像文件、iSCSI 卷、共享存储或存储池）、网络基础设施和附加到客户机的块设备数量。就 libvirt 而言，以下是两种迁移类型：

+   离线迁移涉及实例的停机。它的工作方式是先暂停客户虚拟机，然后将客户虚拟机的内存镜像复制到目标虚拟化主机。然后，KVM 虚拟机在目标主机上恢复运行。如果虚拟机的文件系统不在共享存储中，它还需要被移动到目标服务器。

+   实时迁移通过在没有感知停机的情况下移动实例的当前状态来工作，保持内存和 CPU 寄存器状态不变。

广义来说，离线迁移涉及以下步骤：

+   停止实例

+   将其 XML 定义导出到文件中

+   将客户文件系统镜像复制到目标服务器（如果未使用共享存储）

+   在目标主机上定义实例并启动它

相比之下，在线迁移需要共享存储，如 NFS 或 GlusterFS，避免了将客户机文件系统传输到目标服务器的需求。迁移的速度取决于源实例的内存更新/写入频率、内存的大小，以及源主机和目标主机之间的可用网络带宽。

实时迁移遵循以下过程：

+   原始虚拟机在内存内容传输到目标主机时继续运行

+   Libvirt 监控已传输内存页的任何变化，如果它们已被更新，则会重新传输它们

+   一旦内存内容传输到目标主机，原始实例将被暂停，目标主机上的新实例将被恢复

在本章中，我们将通过存储池的帮助，使用 iSCSI 和 GlusterFS 执行离线和实时迁移。

# 使用 iSCSI 存储池进行手动离线迁移

在此方案中，我们将设置一个 iSCSI 目标，配置一个存储池，并使用附加的 iSCSI 块设备作为后端卷创建一个新的 KVM 实例。接着，我们将执行实例的手动离线迁移到新主机。

# 准备工作

对于此方案，我们需要以下内容：

+   两台已安装并配置了 `libvirt` 和 `qemu` 的服务器，分别命名为 `kvm1` 和 `kvm2`。这两台主机必须能够通过 SSH 密钥和简短的主机名互相连接。

+   一台带有可用块设备的服务器，该块设备将作为 iSCSI 目标导出，并且可以从两台 `libvirt` 服务器访问。如果没有可用块设备，请参阅此食谱中的 *更多信息...* 部分，了解如何使用常规文件创建一个。此食谱中的 iSCSI 目标服务器名称是 `iscsi_target`。

+   需要连接到 Linux 仓库以安装客户机操作系统。

# 如何操作……

要执行使用 iSCSI 存储池的 KVM 客户机的手动脱机迁移，请按照以下步骤进行：

1.  在 iSCSI 目标主机上安装 `iscsitarget` 包和内核模块包：

```
root@iscsi_target:~# apt-get update && apt-get install iscsitarget iscsitarget-dkms

```

1.  启用目标功能：

```
root@iscsi_target:~# sed -i 's/ISCSITARGET_ENABLE=false/ISCSITARGET_ENABLE=true/g' /etc/default/iscsitarget
root@iscsi_target:~# cat /etc/default/iscsitarget
ISCSITARGET_ENABLE=true
ISCSITARGET_MAX_SLEEP=3

# ietd options
# See ietd(8) for details
ISCSITARGET_OPTIONS=""
root@iscsi_target:~#

```

1.  配置块设备以通过 iSCSI 导出：

```
root@iscsi_target:~# cat /etc/iet/ietd.conf
Target iqn.2001-04.com.example:kvm
 Lun 0 Path=/dev/loop1,Type=fileio
 Alias kvm_lun

root@iscsi_target:~#

```

用您要通过 iSCSI 导出的块设备替换 `/dev/loop1` 设备。

1.  重启 iSCSI 目标服务：

```
root@iscsi_target:~# /etc/init.d/iscsitarget restart
 * Removing iSCSI enterprise target devices: [ OK ]
 * Stopping iSCSI enterprise target service: [ OK ]
 * Removing iSCSI enterprise target modules: [ OK ]
 * Starting iSCSI enterprise target service  [ OK ]
root@iscsi_target:~#

```

1.  在两个 `libvirt` 主机上安装 iSCSI 启动器：

```
root@kvm1/2:~# apt-get update && apt-get install open-iscsi

```

1.  在两个 `libvirt` 服务器上启用 iSCSI 启动器服务并启动它：

```
root@kvm1/2:~# sed -i 's/node.startup = manual/node.startup = automatic/g' /etc/iscsi/iscsid.conf
root@kvm1/2:~# /etc/init.d/open-iscsi restart

```

1.  从两个 `libvirt` 启动器主机上，通过查询 iSCSI 目标服务器，列出可用的 iSCSI 卷：

```
root@kvm1/2:~# iscsiadm -m discovery -t sendtargets -p iscsi_target
10.184.226.74:3260,1 iqn.2001-04.com.example:kvm
172.99.88.246:3260,1 iqn.2001-04.com.example:kvm
192.168.122.1:3260,1 iqn.2001-04.com.example:kvm
root@kvm:~#

```

1.  在其中一台 `libvirt` 服务器上创建新的 iSCSI 存储池：

```
root@kvm1:~# cat iscsi_pool.xml
<pool type="iscsi">
 <name>iscsi_pool</name>
 <source>
 <host name="iscsi_target.example.com"/>
 <device path="iqn.2001-04.com.example:kvm"/>
 </source>
 <target>
 <path>/dev/disk/by-path</path>
 </target>
</pool>
root@kvm1:~# virsh pool-define iscsi_pool.xml
Pool iscsi_pool defined from iscsi_pool.xml

root@kvm1:~# virsh pool-list --all
 Name        State      Autostart
-------------------------------------------
 iscsi_pool  inactive   no

root@kvm1:~#

```

确保将 iSCSI 目标服务器的主机名替换为适合您环境的主机名。指定 iSCSI 目标主机时，可以使用主机名和 IP 地址。

1.  启动新的 iSCSI 存储池：

```
root@kvm1:~# virsh pool-start iscsi_pool
Pool iscsi_pool started

root@kvm1:~# virsh pool-list --all
 Name         State   Autostart
-------------------------------------------
 iscsi_pool   active   no

root@kvm1:~#

```

1.  列出来自存储池的可用 iSCSI 卷并获取更多信息：

```
root@kvm1:~# virsh vol-list --pool iscsi_pool
 Name Path
------------------------------------------------------------------------------
 unit:0:0:0 /dev/disk/by-path/ip-10.184.22.74:3260-iscsi-iqn.2001-04.com.example:kvm-lun-0

root@kvm1:~# virsh vol-info unit:0:0:0 --pool iscsi_pool
Name:       unit:0:0:0
Type:       block
Capacity:   10.00 GiB
Allocation: 10.00 GiB

root@kvm1:~#

```

1.  列出 iSCSI 会话及其相关的块设备：

```
root@kvm1:~# iscsiadm -m session
tcp: [5] 10.184.226.74:3260,1 iqn.2001-04.com.example:kvm
root@kvm1:~# ls -la /dev/disk/by-path/
total 0
drwxr-xr-x 2 root root 100 Apr 12 16:24 .
drwxr-xr-x 6 root root 120 Mar 21 22:14 ..
lrwxrwxrwx 1 root root 9 Apr 12 16:24 ip-10.184.22.74:3260-iscsi-iqn.2001-04.com.example:kvm-lun-0 -> ../../sdf
root@kvm1:~#

```

1.  检查 iSCSI 块设备的分区方案：

```
root@kvm1:~# fdisk -l /dev/disk/by-path/ip-10.184.22.74\:3260-iscsi-iqn.2001-04.com.example\:kvm-lun-0

Disk /dev/disk/by-path/ip-10.184.22.74:3260-iscsi-iqn.2001-04.com.example:kvm-lun-0: 10.7 GB, 10737418240 bytes
64 heads, 32 sectors/track, 10240 cylinders, total 20971520 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000

Disk /dev/disk/by-path/ip-10.184.22.74:3260-iscsi-iqn.2001-04.com.example:kvm-lun-0 doesn't contain a valid partition table
root@kvm1:~#

```

1.  使用 iSCSI 卷和存储池安装新的 KVM 客户机：

```
root@kvm1:~# virt-install --name iscsi_kvm --ram 1024 --extra-args="text console=tty0 utf8 console=ttyS0,115200" --graphics vnc,listen=0.0.0.0 --hvm --location=http://ftp.us.debian.org/debian/dists/stable/main/installer-amd64/ --disk vol=iscsi_pool/unit:0:0:0

Starting install...
Retrieving file MANIFEST... | 3.3 kB 00:00 ...
Retrieving file linux...
...
root@kvm1:~# virsh console iscsi_kvm
...
Requesting system reboot
[ 305.315002] reboot: Restarting system
root@kvm1:~#

```

1.  刷新分区表列表并检查安装后的新块设备：

```
root@kvm1:~# partprobe
root@kvm1:~# ls -la /dev/disk/by-path/
total 0
drwxr-xr-x 2 root root 160 Apr 12 16:36 .
drwxr-xr-x 6 root root 120 Mar 21 22:14 ..
lrwxrwxrwx 1 root root 9 Apr 12 16:36 ip-10.184.22.74:3260-iscsi-iqn.2001-04.com.example:kvm-lun-0 -> ../../sdf
lrwxrwxrwx 1 root root 10 Apr 12 16:36 ip-10.184.22.74:3260-iscsi-iqn.2001-04.com.example:kvm-lun-0-part1 -> ../../sdf1
lrwxrwxrwx 1 root root 10 Apr 12 16:36 ip-10.184.22.74:3260-iscsi-iqn.2001-04.com.example:kvm-lun-0-part2 -> ../../sdf2
lrwxrwxrwx 1 root root 10 Apr 12 16:36 ip-10.184.22.74:3260-iscsi-iqn.2001-04.com.example:kvm-lun-0-part5 -> ../../sdf5
root@kvm1:~# fdisk -l /dev/sdf

Disk /dev/sdf: 10.7 GB, 10737418240 bytes
255 heads, 63 sectors/track, 1305 cylinders, total 20971520 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x37eb1540

 Device Boot Start End Blocks Id System
/dev/sdf1 * 2048 20013055 10005504 83 Linux
/dev/sdf2 20015102 20969471 477185 5 Extended
/dev/sdf5 20015104 20969471 477184 82 Linux swap / Solaris
root@kvm1:~# 

```

1.  启动新的 KVM 客户机并确保它正在运行，并且可以通过 VNC 客户端连接到它：

```
root@kvm1:~# virsh start iscsi_kvm
Domain iscsi_kvm started

root@kvm1:~# virsh list --all
 Id  Name        State
----------------------------------------------------
 19  iscsi_kvm   running

root@kvm1:~#

```

1.  若要手动迁移实例到新主机，首先停止虚拟机和 iSCSI 存储池：

```
root@kvm1:~# virsh destroy iscsi_kvm
Domain iscsi_kvm destroyed

root@kvm1:~# virsh pool-destroy iscsi_pool
Pool iscsi_pool destroyed

root@kvm1:~# iscsiadm -m session
iscsiadm: No active sessions.
root@kvm1:~#

```

1.  将 KVM 实例的 XML 配置转储到文件并检查：

```
root@kvm1:~# virsh dumpxml iscsi_kvm > iscsi_kvm.xml
root@kvm1:~# cat iscsi_kvm.xml
<domain type='kvm'>
 <name>iscsi_kvm</name>
 <uuid>306e05ed-e398-ef33-d6e2-3708e90b89a6</uuid>
 <memory unit='KiB'>1048576</memory>
 <currentMemory unit='KiB'>1048576</currentMemory>
 <vcpu placement='static'>1</vcpu>
 <os>
 <type arch='x86_64' machine='pc-i440fx-trusty'>hvm</type>
 <boot dev='hd'/>
 </os>
 <features>
 <acpi/>
 <apic/>
 <pae/>
 </features>
 <clock offset='utc'/>
 <on_poweroff>destroy</on_poweroff>
 <on_reboot>restart</on_reboot>
 <on_crash>restart</on_crash>
 <devices>
 <emulator>/usr/bin/qemu-system-x86_64</emulator>
 <disk type='block' device='disk'>
 <driver name='qemu' type='raw'/>
 <source dev='/dev/disk/by-path/ip-10.184.22.74:3260-iscsi-iqn.2001-04.com.example:kvm-lun-0'/>
 <target dev='hda' bus='ide'/>
 <address type='drive' controller='0' bus='0' target='0' unit='0'/>
 </disk>
 <controller type='usb' index='0'>
 <address type='pci' domain='0x0000' bus='0x00' slot='0x01' function='0x2'/>
 </controller>
 <controller type='pci' index='0' model='pci-root'/>
 <controller type='ide' index='0'>
 <address type='pci' domain='0x0000' bus='0x00' slot='0x01' function='0x1'/>
 </controller>
 <interface type='network'>
 <mac address='52:54:00:8b:b8:e3'/>
 <source network='default'/>
 <model type='rtl8139'/>
 <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x0'/>
 </interface>
 <serial type='pty'>
 <target port='0'/>
 </serial>
 <console type='pty'>
 <target type='serial' port='0'/>
 </console>
 <input type='mouse' bus='ps2'/>
 <input type='keyboard' bus='ps2'/>
 <graphics type='vnc' port='-1' autoport='yes' listen='0.0.0.0'>
 <listen type='address' address='0.0.0.0'/>
 </graphics>
 <video>
 <model type='cirrus' vram='9216' heads='1'/>
 <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x0'/>
 </video>
 <memballoon model='virtio'>
 <address type='pci' domain='0x0000' bus='0x00' slot='0x04' function='0x0'/>
 </memballoon>
 </devices>
</domain>

root@kvm1:~#

```

1.  从 `kvm1` 主机到 `kvm2` 主机远程创建 iSCSI 存储池：

```
root@kvm1:~# virsh --connect qemu+ssh://kvm2/system pool-define iscsi_pool.xml
Pool iscsi_pool defined from iscsi_pool.xml

root@kvm1:~#

```

如果您在两个 KVM 主机之间未使用 SSH 密钥连接，系统会提示您在执行 `libvirt` 命令之前提供密码。我们建议在迁移的 `libvirt` 主机上使用 SSH 密钥。

1.  在 `kvm2` 服务器上远程启动 iSCSI 存储池并确保它正在运行：

```
root@kvm1:~# virsh --connect qemu+ssh://kvm2/system pool-start iscsi_pool
Pool iscsi_pool started

root@kvm1:~# virsh --connect qemu+ssh://kvm2/system pool-list --all
 Name        State   Autostart
-------------------------------------------
 iscsi_pool  active  no

root@kvm1:~#

```

您也可以通过 SSH 登录到 `kvm2` 服务器，执行所有存储池和卷操作。我们远程操作是为了演示这一概念。

1.  从源主机远程列出 `kvm2` 节点上的可用 iSCSI 卷：

```
root@kvm1:~# virsh --connect qemu+ssh://kvm2/system vol-list --pool iscsi_pool
 Name Path
--------------------------------------------------------------------
 unit:0:0:0 /dev/disk/by-path/ip-10.184.22.74:3260-iscsi-iqn.2001-04.com.example:kvm-lun-0

root@kvm1:~#

```

1.  SSH 登录到第二台 KVM 服务器，并确保 iSCSI 块设备现在在主机操作系统中可用：

```
root@kvm2:~# iscsiadm -m session
tcp: [3] 10.184.226.74:3260,1 iqn.2001-04.com.example:kvm
root@kvm2:~# ls -la /dev/disk/by-path/
total 0
drwxr-xr-x 2 root root 120 Apr 12 17:44 .
drwxr-xr-x 6 root root 120 Apr 12 17:44 ..
lrwxrwxrwx 1 root root 9 Apr 12 17:44 ip-10.184.22.74:3260-iscsi-iqn.2001-04.com.example:kvm-lun-0 -> ../../sdc
lrwxrwxrwx 1 root root 10 Apr 12 17:44 ip-10.184.22.74:3260-iscsi-iqn.2001-04.com.example:kvm-lun-0-part1 -> ../../sdc1
lrwxrwxrwx 1 root root 10 Apr 12 17:44 ip-10.184.22.74:3260-iscsi-iqn.2001-04.com.example:kvm-lun-0-part2 -> ../../sdc2
lrwxrwxrwx 1 root root 10 Apr 12 17:44 ip-10.184.22.74:3260-iscsi-iqn.2001-04.com.example:kvm-lun-0-part5 -> ../../sdc5
root@kvm2:~#

```

1.  通过远程定义 KVM 实例并在目标主机上启动它来完成迁移：

```
root@kvm1:~# virsh --connect qemu+ssh://kvm2/system define iscsi_kvm.xml
Domain iscsi_kvm defined from iscsi_kvm.xml

root@kvm1:~# virsh --connect qemu+ssh://kvm2/system list --all
 Id   Name        State
----------------------------------------------------
 -    iscsi_kvm   shut off

root@kvm:~# virsh --connect qemu+ssh://kvm2/system start iscsi_kvm
Domain iscsi_kvm started

root@kvm:~# virsh --connect qemu+ssh://kvm2/system list --all
 Id    Name        State
----------------------------------------------------
 3     iscsi_kvm   running

root@kvm1:~#

```

# 它是如何工作的……

在本配方中，我们演示了如何手动执行 KVM 实例的离线迁移，从一个主机迁移到另一个主机，使用 iSCSI 存储池。在本章稍后的*使用 virsh 命令进行在线迁移*配方中，我们将使用相同的 iSCSI 存储池和实例，通过 `virsh` 命令执行实时迁移，从而避免实例的停机时间。

让我们一步步探索，并更详细地了解如何完成手动离线迁移。

我们从将要提供 iSCSI 目标的服务器开始，首先在步骤 1 中安装所需的 iSCSI 目标服务器软件包。

在步骤 2 中，我们启用了 iSCSI 目标功能，使服务器能够通过 iSCSI 协议导出块设备。

在步骤 3 中，我们为 iSCSI 目标设备指定了一个标识符（iSCSI 合格名称）`iqn.2001-04.com.example:kvm`，该设备将供启动器使用。我们在这个示例中使用的是 `/dev/loop1` 块设备。iSCSI 合格名称的格式为 **iqn.yyyy-mm.naming-authority:unique name**，其中：

+   **iqn**：这是 iSCSI 合格名称标识符

+   **yyyy-mm**：这是命名机构成立的年份和月份

+   **命名机构**：这通常是命名机构的互联网域名或服务器域名的反向语法

+   **唯一名称**：这是你想要使用的任何名称

有关 iSCSI 和它使用的命名方案的更多信息，请参阅 [`en.wikipedia.org/wiki/ISCSI`](https://en.wikipedia.org/wiki/ISCSI)。

在目标定义完成后，我们在步骤 4 中重启了服务器上的 iSCSI 服务。

在步骤 5 和 6 中，我们在两个 KVM 节点上安装并配置了 iSCSI 启动器服务，在步骤 7 中，我们请求所有可用的 iSCSI 目标。在步骤 8 和 9 中，我们定义并启动了一个新的基于 iSCSI 的存储池。如果你已经完成了 第二章中的*使用存储池*配方，你会发现存储池定义的语法非常熟悉，*使用 libvirt 管理 KVM*。

在创建 iSCSI 存储池后，我们继续在步骤 10 中列出该存储池中的卷。请注意，当我们启动存储池时，它登录了 iSCSI 目标，导致在 `/dev/disk/by-path/` 目录下出现了一个新的块设备，正如我们在步骤 11 中进一步看到的那样。我们现在可以在本地使用这个块设备来安装新的 Linux 操作系统。在步骤 12 中，我们可以看到提供给主机操作系统的 iSCSI 块设备尚未包含任何分区。

随着新的块设备出现，我们继续在步骤 13 中创建一个新的 KVM 实例，指定存储池和卷作为安装目标。操作系统安装完成后，我们现在可以看到 iSCSI 块设备上有多个分区（见步骤 14）。接着，我们在步骤 15 中启动新的客户机。

现在我们有一个使用 iSCSI 块设备的运行 KVM 实例，我们可以继续进行从 `kvm1` 主机到 `kvm2` 主机的离线手动迁移。

我们通过首先在第 16 步停止运行的 KVM 实例及其关联的存储池来启动迁移过程。然后在第 17 步中，我们将 KVM 客户端的 XML 配置导出到文件中。我们将使用它来在目标服务器上定义客户机。我们有几个选项：我们可以将文件复制到目标服务器并在那里定义实例，或者我们可以从原始主机远程执行此操作。

在第 18 和 19 步中，我们从原始主机到目标主机远程创建 iSCSI 存储池。我们本来也可以登录到目标主机并执行相同的操作，结果也一样。这里的重点是，我们可以使用 `qemu+ssh` 连接字符串通过 SSH 远程连接到其他 qemu 实例。在第 20 和 21 步中，我们确保相同的 iSCSI 卷已成功登录到目标主机。

最后，在第 22 步中，我们使用第 17 步中导出的 XML 配置定义目标主机上的实例，并启动它。因为我们使用的是相同的 XML 定义文件和包含客户操作系统文件系统的相同 iSCSI 块设备，所以我们现在在新服务器上创建了完全相同的实例，从而完成了离线迁移。

# 还有更多…

如果 iSCSI 目标服务器没有可用的块设备可以导出，我们可以通过按照此处列出的步骤使用常规文件创建一个新的块设备：

1.  创建一个给定大小的新映像文件：

```
root@iscsi_target:~# truncate --size 10G xvdb.img
root@iscsi_target:~# file -s xvdb.img
xvdb.img: data
root@kvmiscsi_target:~# qemu-img info xvdb.img
image: xvdb.img
file format: raw
virtual size: 10G (10737418240 bytes)
disk size: 0
root@iscsi_target:~#

```

1.  确保环回内核模块已编译（或使用 `modprobe loop` 加载它），并找到第一个可用的环回设备供使用：

```
root@iscsi_target:~# grep 'loop' /lib/modules/`uname -r`/modules.builtin
kernel/drivers/block/loop.ko
root@iscsi_target:~# losetup --find
/dev/loop0
root@iscsi_target:~#

```

1.  将原始文件映像与第一个可用的环回设备关联：

```
root@iscsi_target:~# losetup /dev/loop0 xvdb.img
root@iscsi_target:~# losetup --all
/dev/loop0: [10300]:263347 (/root/xvdb.img)
root@iscsi_target:~#

```

在第 1 步中，我们使用 `truncate` 命令创建一个新映像文件。

在第 2 步中，我们列出第一个可用的块设备，并在第 3 步中将其与第 1 步中创建的原始映像文件关联。结果是一个新的块设备 `/dev/loop0` 可供我们用于 iSCSI 导出。

# 使用 GlusterFS 共享卷的手动离线迁移

在 *使用 iSCSI 存储池的手动离线迁移* 食谱中，我们创建了一个 iSCSI 存储池并在执行手动离线迁移时使用了它。使用存储池时，我们可以将共享存储的操作委托给 libvirt，而无需手动登录/退出 iSCSI 目标等。这在我们执行带有 `virsh` 命令的实时迁移时尤其有用，正如我们在下一个食谱中将看到的那样。尽管使用存储池不是必须的，但它简化并集中管理后端卷。

在这个食谱中，我们将使用 GlusterFS 网络文件系统来演示手动迁移 KVM 实例的另一种方法，这次不使用存储池。

GlusterFS 有以下两个组件：

+   **服务器组件**：这运行 `GlusterFS` 守护进程，并将本地块设备命名为 **砖块** 作为卷导出，客户端组件可以挂载这些卷

+   **客户端组件**：这通过 TCP/IP 连接到 GlusterFS 集群，并可以挂载导出的卷

有以下三种类型的卷：

+   **分布式**：这些是将文件分布到整个集群的卷

+   **复制**：这些是跨两个或更多节点在存储集群中复制数据的卷

+   **条带化**：这些是跨多个存储节点的条带文件

为了实现高可用性，我们将使用两个 GFS 节点，使用复制卷（两个砖块包含相同的数据）。

# 准备就绪

为了完成这个食谱，我们将使用以下内容：

+   两台将托管 GlusterFS 共享文件系统的服务器。

+   运行 `libvirt` 和 `qemu` 的两台主机，将用于迁移 KVM 客户机。

+   所有服务器应该能够通过主机名相互通信。

+   托管共享卷的两台服务器应该有一个块设备可用，作为 GlusterFS 的砖块。如果没有可用的块设备，请参考本章中 *使用 iSCSI 存储池进行手动离线迁移* 处的 *更多内容...* 部分，了解如何使用常规文件创建块设备。

+   连接到 Linux 仓库以安装客户机操作系统。

# 如何操作...

要使用共享的 GlusterFS 后端存储迁移 KVM 客户机，请运行以下命令：

1.  在两台将托管共享卷的服务器上，安装 GlusterFS：

```
root@glusterfs1/2:~# apt-get update && apt-get install glusterfs-server

```

1.  从一个 GlusterFS 节点，探测另一个节点以形成集群：

```
root@glusterfs1:~# gluster peer status
peer status: No peers present
root@glusterfs1:~# gluster peer probe glusterfs2
peer probe: success
root@glusterfs1:~#

```

1.  验证 GlusterFS 节点是否互相识别：

```
root@glusterfs1:~# gluster peer status
Number of Peers: 1

Hostname: glusterfs2
Port: 24007
Uuid: 923d152d-df3b-4dfd-9def-18dbebf2b76a
State: Peer in Cluster (Connected)
root@glusterfs1:~#

```

1.  在两台 GlusterFS 主机上，创建将用作 GlusterFS 砖块的块设备文件系统并挂载它们：

```
root@glusterfs1/2:~# mkfs.ext4 /dev/loop5
...
Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
root@glusterfs1/2:~# mount /dev/loop5 /mnt/
root@glusterfs1/2:~# mkdir /mnt/bricks
root@glusterfs1/2:~#

```

确保将块设备名称替换为适合你系统的名称。

1.  从其中一个 GlusterFS 节点，使用来自两台服务器的砖块创建复制存储卷，然后列出它：

```
root@glusterfs1:~# gluster volume create kvm_gfs replica 2 transport tcp glusterfs1:/mnt/bricks/gfs1 glusterfs2:/mnt/bricks/gfs2
volume create: kvm_gfs: success: please start the volume to access data
root@glusterfs1:~# gluster volume list
kvm_gfs
root@glusterfs1:~#

```

1.  从其中一台 GlusterFS 主机，启动新的卷并获取更多信息：

```
root@glusterfs1:~# gluster volume start kvm_gfs
volume start: kvm_gfs: success
root@glusterfs1:~# gluster volume info

Volume Name: kvm_gfs
Type: Replicate
Volume ID: 69823a48-8b1b-469f-b06a-14ef6f33a6f5
Status: Started
Number of Bricks: 1 x 2 = 2
Transport-type: tcp
Bricks:
Brick1: glusterfs1:/mnt/bricks/gfs1
Brick2: glusterfs2:/mnt/bricks/gfs2
root@glusterfs1:~#

```

1.  在两个 `libvirt` 节点上，安装 GlusterFS 客户端并挂载将用于托管 KVM 镜像的 GlusterFS 卷：

```
root@kvm1/2:~# apt-get update && apt-get install glusterfs-client 
root@kvm1/2:~# mkdir /tmp/kvm_gfs
root@kvm1/2:~# mount -t glusterfs glusterfs1:/kvm_gfs /tmp/kvm_gfs
root@kvm1/2:~#

```

挂载 GlusterFS 卷时，你可以指定集群中的任一节点。在前面的示例中，我们从 `glusterfs1` 节点挂载。

1.  在其中一个 `libvirt` 节点上，创建一个新的 KVM 实例，使用挂载的 GlusterFS 卷：

```
root@kvm1:~# virt-install --name kvm_gfs --ram 1024 --extra-args="text console=tty0 utf8 console=ttyS0,115200" --graphics vnc,listen=0.0.0.0 --hvm --location=http://ftp.us.debian.org/debian/dists/stable/main/installer-amd64/ --disk /tmp/kvm_gfs/gluster_kvm.img,size=5
...
root@kvm1:~#

```

1.  确保两个 `libvirt` 节点可以看到客户机镜像：

```
root@kvm1/2:~# ls -al /tmp/kvm_gfs/
total 1820300
drwxr-xr-x 3 root root 4096 Apr 13 14:48 .
drwxrwxrwt 6 root root 4096 Apr 13 15:00 ..
-rwxr-xr-x 1 root root 5368709120 Apr 13 14:59 gluster_kvm.img
root@kvm1/2:~#

```

1.  要手动迁移 KVM 实例从一个 `libvirt` 节点到另一个，首先停止实例并导出其 XML 定义：

```
root@kvm1:~# virsh destroy kvm_gfs
Domain kvm_gfs destroyed

root@kvm1:~# virsh dumpxml kvm_gfs > kvm_gfs.xml
root@kvm1:~#

```

1.  从源 `libvirt` 节点，在目标主机上定义实例：

```
root@kvm1:~# virsh --connect qemu+ssh://kvm2/system define kvm_gfs.xml
Domain kvm_gfs defined from kvm_gfs.xml

root@kvm1:~# virsh --connect qemu+ssh://kvm2/system list --all
 Id    Name       State
----------------------------------------------------
 -     kvm_gfs    shut off

root@kvm1:~#

```

1.  在目标主机上启动 KVM 实例以完成迁移：

```
root@kvm2:~# virsh start kvm_gfs
Domain kvm_gfs started

root@kvm2:~#

```

我们还可以使用 `qemu+ssh` 连接从源主机启动目标主机上的 KVM 实例，如下所示：

`root@kvm1:~# virsh --connect qemu+ssh://kvm2/system start kvm_gfs`

# 它是如何工作的...

我们首先在步骤 1 中在两个服务器上安装 GlusterFS 服务端包。然后，在步骤 2 中，我们通过从第一个 GlusterFS 节点发送探测来形成集群。如果探测成功，我们将在步骤 3 中进一步获取集群信息。在步骤 4 中，我们通过在 GlusterFS 服务器上创建文件系统并挂载它们来准备块设备。挂载后，块设备将包含形成虚拟复制卷的砖块，供 GlusterFS 导出。

在步骤 5 中，我们在其中一个节点上创建新的复制卷（这将影响整个集群，且只需从一个 GlusterFS 节点运行）。我们指定类型为复制，使用 TCP 协议，并指定将要使用的砖块位置。卷创建完成后，在步骤 6 中我们启动它，并获取更多信息。注意，从卷信息的输出中，我们可以看到正在使用的砖块数量以及它们在集群中的位置。

在步骤 7 中，我们在两个`libvirt`服务器上安装 GlusterFS 客户端组件，并挂载 GFS 卷。现在，两个 KVM 主机共享同一物理托管在 GlusterFS 节点上的复制存储。我们将使用该共享存储来托管新的 KVM 镜像文件。

在步骤 8 中，我们继续安装新的 KVM 实例，使用在前一步挂载的 GlusterFS 卷。安装完成后，我们在步骤 9 中验证两个`libvirt`服务器是否可以看到新的 KVM 镜像。

我们在步骤 10 中开始手动迁移，首先停止正在运行的 KVM 实例，然后将其配置保存到磁盘。在步骤 11 中，我们使用 XML 转储远程定义 KVM 客户端，并验证它是否已成功在目标主机上定义。最后，我们在目标服务器上启动 KVM 实例，完成迁移。

# 使用 virsh 命令进行的在线迁移和共享存储

`virsh`命令提供了一个迁移参数，我们可以用它来在主机之间迁移 KVM 实例。在前面的两个配方中，我们了解了如何手动迁移实例并停机。在本配方中，我们将对使用 iSCSI 存储池或本章前面提到的 GlusterFS 共享卷的实例执行在线迁移。

如果你记得，在线迁移只有在客户端文件系统位于某种共享介质上时才有效，例如 NFS、iSCSI、GlusterFS，或者如果我们先将镜像文件复制到所有节点，并在使用`virsh migrate`时带上`--copy-storage-all`选项，就像我们将在本章后面看到的那样。

# 准备工作

为了完成本配方，我们需要以下内容：

+   两个`libvirt`主机之间具有共享存储。如果你已经完成了本章前面的配方，你可以使用我们创建的 iSCSI 存储池和正在使用它的 KVM 实例，或者使用带有 KVM 客户端的 GFS 共享存储。

+   两个 `libvirt` 主机应能够使用短主机名相互通信。

# 如何操作...

要执行使用共享存储的实时迁移，请执行此处列出的操作：

1.  确保我们之前创建的 iSCSI KVM 实例在源主机上运行：

```
root@kvm1:~# virsh list --all
 Id  Name        State
----------------------------------------------------
 26  iscsi_kvm   running

root@kvm1:~#

```

1.  将实例实时迁移到第二个 `libvirt` 服务器（目标节点应已配置 iSCSI 存储池）。如果此操作失败，请参阅本食谱的 *还有更多内容...* 部分以获取故障排除提示：

```
root@kvm1:~# virsh migrate --live iscsi_kvm qemu+ssh://kvm2/system

root@kvm1:~#

```

1.  确保 KVM 实例已在源主机上停止，并在目标服务器上启动：

```
root@kvm1:~# virsh list --all
 Id   Name       State
----------------------------------------------------
 -    iscsi_kvm  shut off

root@kvm1:~# virsh --connect qemu+ssh://kvm2/system list --all
 Id   Name     State
----------------------------------------------------
 10   iscsi_kvm  running

root@kvm1:~#

```

1.  要将实例迁移回原服务器，从 `kvm2` 节点执行以下命令：

```
root@kvm2:~# virsh migrate --live iscsi_kvm qemu+ssh://kvm1/system

root@kvm2:~# virsh list --all
 Id Name State
----------------------------------------------------

root@kvm2:~# virsh --connect qemu+ssh://kvm1/system list --all
 Id   Name        State
----------------------------------------------------
 28   iscsi_kvm   running

root@kvm2:~#

```

# 它是如何工作的...

当迁移一个使用共享存储的 KVM 实例时，例如本示例中的 iSCSI 存储池，一旦我们启动带有 `migrate --live` 参数的迁移，libvirt 会自动处理退出原主机的 iSCSI 会话并登录到目标服务器，从而使包含虚拟机文件系统的块设备在目标服务器上可用，而无需复制所有数据。你可能已经注意到，迁移只用了几秒钟，因为迁移的唯一数据是运行中的虚拟机在源主机上的内存页。

# 还有更多内容...

根据你运行该操作的 Linux 发行版和服务器类型（物理机或云实例），你可能会遇到一些常见错误，当尝试迁移实例时。

**错误**：错误：不安全的迁移：如果磁盘使用的缓存设置不为 none，迁移可能会导致数据损坏。

**解决方案**：编辑你正在尝试迁移的实例的 XML 定义，并更新块设备的驱动程序部分，添加 `cache=none` 属性：

```
root@kvm1:~# virsh edit iscsi_kvm
...
<devices>
 ...
 <disk type='block' device='disk'>
 <driver name='qemu' type='raw' *cache='none'*/>
 <source dev='/dev/disk/by-path/ip-10.184.22.74:3260-iscsi-iqn.2001-04.com.example:kvm-lun-0'/>
 <target dev='hda' bus='ide'/>
 <address type='drive' controller='0' bus='0' target='0' unit='0'/>
 </disk>
 ...
</devices>

```

**错误**：错误：内部错误：尝试将虚拟机迁移到相同的主机 `02000100-0300-0400-0005-000600070008`。

**解决方案**：某些服务器，通常是虚拟化的，可能会返回相同的系统 UUID，这会导致迁移失败。要查看是否是这种情况，请在源机器和目标机器上运行以下命令：

```
root@kvm1/2:~# virsh sysinfo | grep -B5 -A3 uuid
 <system>
 <entry name='manufacturer'>FOXCONN</entry>
 <entry name='product'>CL7100</entry>
 <entry name='version'>PVT1-X05</entry>
 <entry name='serial'>2M2542Z069</entry>
 <entry name='uuid'>*02000100-0300-0400-0005-000600070008*</entry>
 <entry name='sku'>NULL</entry>
 <entry name='family'>Intel Grantley EP</entry>
 </system>
root@kvm1/2:~#

```

如果两个服务器的 UUID 相同，请编辑 `libvirt` 配置文件并分配一个唯一的 UUID，然后重启 `libvirt`：

```
root@kvm2:~# vim /etc/libvirt/libvirtd.conf
...
host_uuid = "02000100-0300-0400-0006-000600070008"
...
root@kvm2:~# /etc/init.d/libvirt-bin restart
libvirt-bin stop/waiting
libvirt-bin start/running, process 12167
root@kvm2:~#

```

**错误**：错误：无法解析地址 `kvm2.localdomain` 服务 49152：名称或服务无法识别。

**解决方案**：这表明 `libvirt` 无法解析实例的主机名。确保主机名不会解析为 localhost，并且你可以使用主机名而不是服务器的 IP 地址，在源主机和目标主机之间进行 ping 或 SSH 连接。以下是两个 `libvirt` 节点的工作主机文件示例：

```
root@kvm1:~# cat /etc/hosts
127.0.0.1 localhost

10.184.226.106 kvm1.example.com kvm1
10.184.226.74  kvm2.example.com kvm2
root@kvm1:~#

root@kvm2:~# cat /etc/hosts
127.0.0.1 localhost

10.184.226.106 kvm1.example.com kvm1
10.184.226.74  kvm2.example.com kvm2
root@kvm2:~# 

```

你可以通过检查以下日志来获取更多关于实例操作的信息：

```
root@kvm1:~# cat /var/log/libvirt/libvirtd.log
...
2017-04-12 19:26:02.297+0000: 33149: error : virCommandWait:2399 : internal error: Child process (/usr/bin/iscsiadm --mode session) unexpected exit status 21
...
root@kvm1:~# cat /var/log/libvirt/qemu/iscsi_kvm.log
...
2017-04-13 17:59:48.040+0000: starting up
LC_ALL=C PATH=/usr/local/sbin:/usr/local/bin:/usr/bin:/usr/sbin:/sbin:/bin QEMU_AUDIO_DRV=none /usr/bin/qemu-system-x86_64 -name iscsi_kvm -S -machine pc-i440fx-trusty,accel=kvm,usb=off -m 1024 -realtime mlock=off -smp 1,sockets=1,cores=1,threads=1 -uuid 306e05ed-e398-ef33-d6e2-3708e90b89a6 -no-user-config -nodefaults -chardev socket,id=charmonitor,path=/var/lib/libvirt/qemu/iscsi_kvm.monitor,server,nowait -mon chardev=charmonitor,id=monitor,mode=control -rtc base=utc -no-shutdown -boot strict=on -device piix3-usb-uhci,id=usb,bus=pci.0,addr=0x1.0x2 -drive file=/dev/disk/by-path/ip-10.184.226.74:3260-iscsi-iqn.2001-04.com.example:kvm-lun-0,if=none,id=drive-ide0-0-0,format=raw,cache=none -device ide-hd,bus=ide.0,unit=0,drive=drive-ide0-0-0,id=ide0-0-0,bootindex=1 -netdev tap,fd=24,id=hostnet0 -device rtl8139,netdev=hostnet0,id=net0,mac=52:54:00:8b:b8:e3,bus=pci.0,addr=0x3 -chardev pty,id=charserial0 -device isa-serial,chardev=charserial0,id=serial0 -vnc 0.0.0.0:0 -device cirrus-vga,id=video0,bus=pci.0,addr=0x2 -incoming tcp:[::]:49153 -device virtio-balloon-pci,id=balloon0,bus=pci.0,addr=0x4
char device redirected to /dev/pts/0 (label charserial0)
qemu: terminating on signal 15 from pid 33148
2017-04-13 18:34:49.684+0000: shutting down
...
root@kvm1:~#

```

# 使用 virsh 命令和本地镜像进行离线迁移

使用 virsh 执行离线迁移时不需要共享存储；但是我们需要为新主机提供客户机文件系统（通过复制镜像文件等）。离线迁移会传输实例定义，而不会在目标主机上启动客户机，也不会停止源主机上的客户机。在这个示例中，我们将使用 virsh 命令对运行中的 KVM 客户机进行离线迁移，使用其文件系统的镜像文件。

# 准备工作

对于这个简单的示例，我们需要以下内容：

+   两台 `libvirt` 主机和一个运行中的 KVM 实例。如果你的主机上没有其中之一，可以使用本地镜像文件安装并启动一个新的客户机虚拟机：

```
root@kvm:~# virt-install --name kvm_no_sharedfs --ram 1024 --extra-args="text console=tty0 utf8 console=ttyS0,115200" --graphics vnc,listen=0.0.0.0 --hvm --location=http://ftp.us.debian.org/debian/dists/stable/main/installer-amd64/ --disk /tmp/kvm_no_sharedfs.img,size=5

```

+   两台主机应该能够通过主机名互相通信。

# 如何操作...

要使用 `virsh` 命令执行离线迁移，请运行以下命令：

1.  确保我们有一个运行中的 KVM 实例：

```
root@kvm1:~# virsh list --all
 Id  Name              State
----------------------------------------------------
 26  kvm_no_sharedfs   running

root@kvm1:~#

```

1.  使用离线模式迁移实例。如果此操作出错，请参阅 *更多...* 部分中的 *使用 virsh 命令进行在线迁移* 示例，获取故障排除提示：

```
root@kvm1:~# virsh migrate --offline --persistent kvm_no_sharedfs qemu+ssh://kvm2/system

root@kvm1:~#

```

1.  与实时迁移不同，源实例仍在运行，而目标实例已停止：

```
root@kvm1:~# virsh list --all
 Id    Name              State
----------------------------------------------------
 29    kvm_no_sharedfs   running

root@kvm1:~# virsh --connect qemu+ssh://kvm2/system list --all
 Id   Name               State
----------------------------------------------------
 -    kvm_no_sharedfs    shut off

root@kvm1:~#

```

# 它是如何工作的...

离线迁移相当简单；`virsh` 命令将定义文件从目标主机传输到目标并定义实例。原始的 KVM 客户机保持运行状态。为了启动迁移后的实例，首先需要将其镜像文件传输到目标主机，并且文件路径必须与源服务器上的路径完全一致。与直接转储 XML 文件并在目标主机上定义它的主要区别在于，`libvirt` 会对目标 XML 文件进行更新，比如分配新的 UUID。

在前面提到的示例中，唯一的两个新标志是离线标志和持久标志。前者指定了离线类型的迁移，后者将域设置为在目标主机上持久存在。

# 使用 virsh 命令和本地镜像进行在线迁移

在这个示例中，我们将进行一个实时迁移运行中的实例，且没有共享存储。

# 准备工作

对于这个示例，我们需要以下内容：

+   两台 `libvirt` 服务器，运行中的 KVM 实例，使用本地镜像文件。我们将使用在之前的示例中构建的 KVM 客户机，*使用 virsh 命令和本地镜像进行离线迁移*。

+   两台服务器必须能够通过主机名互相通信。

# 如何操作...

要迁移没有共享存储的实例，请按照以下步骤操作：

1.  确保 KVM 客户机正在运行：

```
root@kvm1:~# virsh list --all
 Id   Name              State
----------------------------------------------------
 33   kvm_no_sharedfs   running

root@kvm1:~#

```

1.  查找镜像文件的位置：

```
root@kvm1:~# virsh dumpxml kvm_no_sharedfs | grep "source file"
 <source file='/tmp/kvm_no_sharedfs.img'/>
root@kvm1:~#

```

1.  将镜像文件传输到目标主机：

```
root@kvm1:~# scp /tmp/kvm_no_sharedfs.img kvm2:/tmp/
kvm_no_sharedfs.img 100% 5120MB 243.8MB/s 00:21
root@kvm1:~#

```

1.  迁移实例并确保它在目标主机上运行：

```
root@kvm1:~# virsh migrate --live --persistent --verbose --copy-storage-all kvm_no_sharedfs qemu+ssh://kvm2/system
Migration: [100 %]
root@kvm1:~# virsh list --all
 Id     Name               State
----------------------------------------------------
 -      kvm_no_sharedfs    shut off

root@kvm1:~# virsh --connect qemu+ssh://kvm2/system list --all
 Id     Name               State
----------------------------------------------------
 17     kvm_no_sharedfs    running

root@kvm1:~#

```

1.  从目标主机迁移实例，使用增量镜像传输：

```
root@kvm2:~# virsh migrate --live --persistent --verbose --copy-storage-inc kvm_no_sharedfs qemu+ssh://kvm/system
Migration: [100 %]
root@kvm2:~#

```

# 它是如何工作的...

在步骤 1 中确保源实例处于运行状态后，我们将镜像文件传输到目标文件，并确保它与源实例在步骤 3 中的位置完全相同。镜像文件就位后，我们可以执行实时迁移，这将在步骤 4 中完成，随后回到步骤 5。

我们到目前为止还没有使用的两个新参数是`--copy-storage-all`和`copy-storage-inc.`。第一个参数指示`libvirt`将整个镜像文件传输到目标位置，而第二个则执行增量传输，仅复制已更改的数据，从而减少传输时间。
