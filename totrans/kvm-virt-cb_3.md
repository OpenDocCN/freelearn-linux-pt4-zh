# 使用 libvirt 进行 KVM 网络配置

在本章中，我们将涵盖以下主题：

+   Linux 桥接

+   Open vSwitch

+   配置 NAT 转发网络

+   配置桥接网络

+   配置 PCI 直通网络

+   操作网络接口

# 简介

使用 libvirt，我们可以为 KVM 客户机定义不同的网络类型，使用已熟悉的 XML 定义语法，以及`virsh`和`virt-install`用户空间工具。在本章中，我们将部署三种不同的网络类型，探索网络 XML 格式，并展示如何定义和操作 KVM 实例的虚拟接口的示例。

为了能够将虚拟机连接到主机操作系统或彼此之间，我们将使用 Linux 桥接和**Open vSwitch**（**OVS**）守护进程、用户空间工具和内核模块。这两种软件桥接技术都非常擅长以一致且易于操作的方式创建**软件定义网络**（**SDN**），无论其复杂性如何。Linux 桥接和 OVS 都充当桥接/交换机，KVM 客户机的虚拟接口可以连接到这些设备。

了解这些内容后，让我们开始深入学习 Linux 中的软件桥接。

# Linux 桥接

Linux 桥接是一个软件层 2 设备，提供了物理桥接设备的一些功能。它可以在 KVM 客户机、主机操作系统和运行在其他服务器或网络上的虚拟机之间转发数据帧。Linux 桥接由两个组件组成——一个用户空间管理工具，我们将在本示例中使用它，以及一个内核模块，负责将多个以太网段连接在一起。每个我们创建的软件桥接可以附加多个端口，网络流量可以从这些端口转发进出。在创建 KVM 实例时，我们可以将与之关联的虚拟接口连接到桥接上，这类似于将一根网络电缆从物理服务器的网卡插入到桥接/交换机设备中。作为一个层 2 设备，Linux 桥接使用 MAC 地址，并维护一个内核结构，以**内容可寻址存储器**（**CAM**）表的形式跟踪端口和关联的 MAC 地址。

在这个示例中，我们将创建一个新的 Linux 桥接，并使用`brctl`工具来操作它。

# 准备工作

对于本示例，我们需要以下内容：

+   启用了`802.1d Ethernet`桥接选项的最新 Linux 内核

要检查你的内核是否编译了这些特性，或者是否作为内核模块暴露，请运行以下命令：

```
root@kvm:~# cat /boot/config-`uname -r` | grep -i bridg
# PC-card bridges
CONFIG_BRIDGE_NETFILTER=y
CONFIG_NF_TABLES_BRIDGE=m
CONFIG_BRIDGE_NF_EBTABLES=m
CONFIG_BRIDGE_EBT_BROUTE=m
CONFIG_BRIDGE_EBT_T_FILTER=m
CONFIG_BRIDGE_EBT_T_NAT=m
CONFIG_BRIDGE_EBT_802_3=m
CONFIG_BRIDGE_EBT_AMONG=m
CONFIG_BRIDGE_EBT_ARP=m
CONFIG_BRIDGE_EBT_IP=m
CONFIG_BRIDGE_EBT_IP6=m
CONFIG_BRIDGE_EBT_LIMIT=m
CONFIG_BRIDGE_EBT_MARK=m
CONFIG_BRIDGE_EBT_PKTTYPE=m
CONFIG_BRIDGE_EBT_STP=m
CONFIG_BRIDGE_EBT_VLAN=m
CONFIG_BRIDGE_EBT_ARPREPLY=m
CONFIG_BRIDGE_EBT_DNAT=m
CONFIG_BRIDGE_EBT_MARK_T=m
CONFIG_BRIDGE_EBT_REDIRECT=m
CONFIG_BRIDGE_EBT_SNAT=m
CONFIG_BRIDGE_EBT_LOG=m
# CONFIG_BRIDGE_EBT_ULOG is not set
CONFIG_BRIDGE_EBT_NFLOG=m
CONFIG_BRIDGE=m
CONFIG_BRIDGE_IGMP_SNOOPING=y
CONFIG_BRIDGE_VLAN_FILTERING=y
CONFIG_SSB_B43_PCI_BRIDGE=y
CONFIG_DVB_DDBRIDGE=m
CONFIG_EDAC_SBRIDGE=m
# VME Bridge Drivers
root@kvm:~#

```

+   `bridge`内核模块

要验证模块是否已加载并获取有关其版本和功能的更多信息，请执行以下命令：

```
root@kvm:~# lsmod | grep bridge
bridge 110925 0
stp 12976 2 garp,bridge
llc 14552 3 stp,garp,bridge
root@kvm:~#
root@kvm:~# modinfo bridge
filename: /lib/modules/3.13.0-107-generic/kernel/net/bridge/bridge.ko
alias: rtnl-link-bridge
version: 2.3
license: GPL
srcversion: 49D4B615F0B11CA696D8623
depends: stp,llc
intree: Y
vermagic: 3.13.0-107-generic SMP mod_unload modversions
signer: Magrathea: Glacier signing key
sig_key: E1:07:B2:8D:F0:77:39:2F:D6:2D:FD:D7:92:BF:3B:1D:BD:57:0C:D8
sig_hashalgo: sha512
root@kvm:~#

```

+   提供创建和操作 Linux 桥接的工具的`bridge-utils`包

+   使用 libvirt 或 QEMU 工具，或使用前几章中的现有 KVM 实例创建新的 KVM 客户机的能力

# 如何操作…

要创建、列出并操作一个新的 Linux 桥接，请按照以下步骤操作：

1.  如果尚未安装，请安装 Linux 桥接包：

```
root@kvm:~# apt install bridge-utils

```

1.  使用来自 第一章*、QEMU 和 KVM 入门*章节中的 *使用 debootstrap 安装自定义操作系统镜像* 配方的原始镜像构建新的 KVM 实例，如果你不是通读本书：

```
root@kvm:~# virt-install --name kvm1 --ram 1024 --disk path=/tmp/debian.img,format=raw --graphics vnc,listen=146.20.141.158 --noautoconsole --hvm --import

Starting install...
Creating domain... | 0 B 00:00
Domain creation completed. You can restart your domain by running:
 virsh --connect qemu:///system start kvm1
root@kvm:~#

```

1.  列出所有可用的桥接设备：

```
root@kvm:~# brctl show
bridge name      bridge id             STP enabled     interfaces
virbr0           8000.fe5400559bd6     yes             vnet0
root@kvm:~#

```

1.  将虚拟桥接关闭，删除它，并确保它已被删除：

```
root@kvm:~# ifconfig virbr0 down
root@kvm:~# brctl delbr virbr0
root@kvm:~# brctl show
bridge name bridge id STP enabled interfaces
root@kvm:~#

```

1.  创建一个新的桥接并启用它：

```
root@kvm:~# brctl addbr virbr0
root@kvm:~# brctl show
bridge name     bridge id            STP enabled interfaces
virbr0          8000.000000000000    no
root@kvm:~# ifconfig virbr0 up
root@kvm:~#

```

1.  为桥接分配一个 IP 地址：

```
root@kvm:~# ip addr add 192.168.122.1 dev virbr0
root@kvm:~# ip addr show virbr0
39: virbr0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default
 link/ether 32:7d:3f:80:d7:c6 brd ff:ff:ff:ff:ff:ff
 inet 192.168.122.1/32 scope global virbr0
 valid_lft forever preferred_lft forever
 inet6 fe80::307d:3fff:fe80:d7c6/64 scope link
 valid_lft forever preferred_lft forever
root@kvm:~#

```

1.  列出主机操作系统上的虚拟接口：

```
root@kvm:~# ip a s | grep vnet
38: vnet0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UNKNOWN group default qlen 500
root@kvm:~#

```

1.  将虚拟接口 `vnet0` 添加到桥接中：

```
root@kvm:~# brctl addif virbr0 vnet0
root@kvm:~# brctl show virbr0
bridge name     bridge id             STP enabled     interfaces
virbr0          8000.fe5400559bd6     no              vnet0
root@kvm:~#

```

1.  启用 **生成树协议** (**STP**) 并获取更多信息：

```
root@kvm:~# brctl stp virbr0 on
root@kvm:~# brctl showstp virbr0
virbr0
 bridge id 8000.fe5400559bd6
 designated root 8000.fe5400559bd6
 root port 0 path cost 0
 max age 20.00 bridge max age 20.00
 hello time 2.00 bridge hello time 2.00
 forward delay 15.00 bridge forward delay 15.00
 ageing time 300.00
 hello timer 0.26 tcn timer 0.00
 topology change timer 0.00 gc timer 90.89
 flags

vnet0 (1)
 port id 8001 state forwarding
 designated root 8000.fe5400559bd6 path cost 100
 designated bridge 8000.fe5400559bd6 message age timer 0.00
 designated port 8001 forward delay timer 0.00
 designated cost 0 hold timer 0.00
 flags

root@kvm:~#

```

1.  从 KVM 实例内部，启用接口，请求 IP 地址，并测试与主机操作系统的连接：

```
root@debian:~# ifconfig eth0 up
root@debian:~# dhclient eth0
root@debian:~# ip a s eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
 link/ether 52:54:00:55:9b:d6 brd ff:ff:ff:ff:ff:ff
 inet 192.168.122.92/24 brd 192.168.122.255 scope global eth0
 valid_lft forever preferred_lft forever
 inet6 fe80::5054:ff:fe55:9bd6/64 scope link
 valid_lft forever preferred_lft forever
root@debian:~#
root@debian:~# ping 192.168.122.1 -c 3
PING 192.168.122.1 (192.168.122.1) 56(84) bytes of data.
64 bytes from 192.168.122.1: icmp_seq=1 ttl=64 time=0.276 ms
64 bytes from 192.168.122.1: icmp_seq=2 ttl=64 time=0.226 ms
64 bytes from 192.168.122.1: icmp_seq=3 ttl=64 time=0.259 ms

--- 192.168.122.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 1999ms
rtt min/avg/max/mdev = 0.226/0.253/0.276/0.027 ms
root@debian:~#

```

# 它是如何工作的……

当我们第一次安装并启动 `libvirt` 守护进程时，自动发生了几件事：

+   创建了一个新的 Linux 桥接，名称和 IP 地址在 `/etc/libvirt/qemu/networks/default.xml` 配置文件中定义。

+   `dnsmasq` 服务启动，并根据 `/var/lib/libvirt/dnsmasq/default.conf` 文件中的配置进行设置。

让我们检查默认的 `libvirt` 桥接配置：

```
root@kvm:~# cat /etc/libvirt/qemu/networks/default.xml
<network>
 <name>default</name>
 <bridge name="virbr0"/>
 <forward/>
 <ip address="192.168.122.1" netmask="255.255.255.0">
 <dhcp>
 <range start="192.168.122.2" end="192.168.122.254"/>
 </dhcp>
 </ip>
</network>
root@kvm:~#

```

这是 libvirt 为我们创建的默认网络，指定了桥接名称、IP 地址以及 DHCP 服务器启动时使用的 IP 范围。我们将在本章稍后详细讨论 libvirt 网络；不过，我们在这里展示它，帮助你理解所有 IP 地址和桥接名称的来源。

我们可以通过运行以下命令查看主机操作系统上运行的 DHCP 服务器及其配置文件：

```
root@kvm:~# pgrep -lfa dnsmasq
38983 /usr/sbin/dnsmasq --conf-file=/var/lib/libvirt/dnsmasq/default.conf
root@kvm:~# cat /var/lib/libvirt/dnsmasq/default.conf
##WARNING: THIS IS AN AUTO-GENERATED FILE. CHANGES TO IT ARE LIKELY TO BE
##OVERWRITTEN AND LOST. Changes to this configuration should be made using:
## virsh net-edit default
## or other application using the libvirt API.
##
## dnsmasq conf file created by libvirt
strict-order
user=libvirt-dnsmasq
pid-file=/var/run/libvirt/network/default.pid
except-interface=lo
bind-dynamic
interface=virbr0
dhcp-range=192.168.122.2,192.168.122.254
dhcp-no-override
dhcp-leasefile=/var/lib/libvirt/dnsmasq/default.leases
dhcp-lease-max=253
dhcp-hostsfile=/var/lib/libvirt/dnsmasq/default.hostsfile
addn-hosts=/var/lib/libvirt/dnsmasq/default.addnhosts
root@kvm:~#

```

从之前的配置文件中，可以看到 DHCP 服务的 IP 地址范围和虚拟桥接的名称与我们刚才看到的默认 libvirt 网络文件中的配置相匹配。

牢记这些内容，让我们回顾一下我们之前执行的所有操作：

在第 1 步中，我们安装了用户空间工具 `brctl`，它用于创建、配置和检查 Linux 内核中的 Linux 桥接配置。

在第 2 步中，我们使用包含客户操作系统的自定义原始镜像创建了一个新的 KVM 实例。如果你已经完成了前面章节中的示例，这一步不需要执行。

在第 3 步中，我们调用了 `bridge` 工具列出所有可用的桥接设备。从输出中可以看到，目前有一个桥接，名为 `virbr0`，是 libvirt 自动创建的。注意，在接口列下，我们可以看到 `vnet0` 接口。这是我们启动 KVM 实例时暴露给主机操作系统的虚拟网卡。这意味着虚拟机连接到了主机桥接。

在第 4 步中，我们首先将桥接关闭以便删除它，然后再次使用 `brctl` 命令删除该桥接，并确保它不再出现在主机操作系统中。

在第 5 步中，我们重新创建了桥接器并将其重新启用。我们这样做是为了演示创建新桥接器所需的步骤。

在第 6 步中，我们将相同的 IP 地址重新分配给桥接器，并列出了该地址。

在第 7 步和第 8 步中，我们列出了主机操作系统上的所有虚拟接口。由于当前服务器上只运行了一个 KVM 客户机，我们只看到了一个虚拟接口，即`vnet0`。然后我们继续将虚拟网卡添加/连接到桥接器。

在第 9 步中，我们启用了桥接上的 STP。STP 是一种二层协议，有助于防止在存在冗余网络路径时出现网络环路。它在更大、更复杂的网络拓扑中尤其有用，在这种拓扑中，多个桥接器相互连接。

最后，在第 10 步中，我们通过控制台连接到 KVM 客户机，列出其接口配置，并确保我们可以在主机操作系统上 ping 到桥接器。为此，我们需要通过`ifconfig eth0 up`命令启动客户机内部的网络接口，然后通过运行`dhclient eth0`命令从主机上的`dnsmasq`服务器获取 IP 地址。

# 还有更多内容...

还有一些其他有用的命令可以在 Linux 桥接器上使用。

我们已经知道，桥接器是根据其中包含的 MAC 地址转发数据帧的。要查看桥接器已知的 MAC 地址表，请运行以下命令：

```
root@kvm:~# brctl showmacs virbr0
port no  mac addr               is local?      ageing timer
 1      52:54:00:55:9b:d6      no             268.02
 1      fe:54:00:55:9b:d6      yes            0.00
root@kvm:~#

```

从前面的输出中，我们可以看到桥接器在其唯一的端口上记录了两个 MAC 地址。第一个记录是非本地地址，属于 KVM 实例内部的网络接口。我们可以通过如下方式连接到 KVM 客户机来确认这一点：

```
root@kvm:~# virsh console kvm1
Connected to domain kvm1
Escape character is ^]

root@debian:~# ip a s eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
 link/ether 52:54:00:55:9b:d6 brd ff:ff:ff:ff:ff:ff
 inet6 fe80::5054:ff:fe55:9bd6/64 scope link
 valid_lft forever preferred_lft forever
root@debian:~#

```

第二个 MAC 地址是桥接器本身的地址，也是虚拟接口的 MAC 地址，该接口属于 KVM 虚拟机，并暴露给主机操作系统。要确认这一点，请运行以下命令：

```
root@kvm:~# ifconfig | grep "fe:54:00:55:9b:d6"
virbr0 Link encap:Ethernet HWaddr fe:54:00:55:9b:d6
vnet0  Link encap:Ethernet HWaddr fe:54:00:55:9b:d6
root@kvm:~#

```

当桥接器在其某个端口看到数据帧时，它会记录下时间，然后在一段时间内如果没有再次看到相同的 MAC 地址，它将从其 CAM 表中删除该记录。我们可以通过执行以下命令来设置桥接器在过期 MAC 地址条目之前的时间限制（以秒为单位）：

```
root@kvm:~# brctl setageing virbr0 600
root@kvm:~#

```

`brctl`命令有很好的文档说明；要列出所有可用的子命令，只需运行该命令而不带任何参数：

```
root@kvm:~# brctl
Usage: brctl [commands]
commands:
 addbr <bridge> add bridge
 delbr <bridge> delete bridge
 addif <bridge> <device> add interface to bridge
 delif <bridge> <device> delete interface from bridge
 hairpin <bridge> <port> {on|off} turn hairpin on/off
 setageing <bridge> <time> set ageing time
 setbridgeprio <bridge> <prio> set bridge priority
 setfd <bridge> <time> set bridge forward delay
 sethello <bridge> <time> set hello time
 setmaxage <bridge> <time> set max message age
 setpathcost <bridge> <port> <cost> set path cost
 setportprio <bridge> <port> <prio> set port priority
 show [ <bridge> ] show a list of bridges
 showmacs <bridge> show a list of mac addrs
 showstp <bridge> show bridge stp info
 stp <bridge> {on|off} turn stp on/off
root@kvm:~#

```

大多数 Linux 发行版都打包了`brctl`工具，这也是我们在本教程中使用的工具。然而，为了使用最新版本，或者如果你的发行版没有该工具的包，我们可以通过克隆项目并使用`git`构建工具的最新版本，然后进行配置和编译：

```
root@kvm:~# cd /usr/src/
root@kvm:/usr/src# apt-get update && apt-get install build-essential automake pkg-config git
root@kvm:/usr/src# git clone git://git.kernel.org/pub/scm/linux/kernel/git/shemminger/bridge-utils.git
Cloning into 'bridge-utils'...
remote: Counting objects: 654, done.
remote: Total 654 (delta 0), reused 0 (delta 0)
Receiving objects: 100% (654/654), 131.72 KiB | 198.00 KiB/s, done.
Resolving deltas: 100% (425/425), done.
Checking connectivity... done.
root@kvm:/usr/src# cd bridge-utils/
root@kvm:/usr/src/bridge-utils# autoconf
root@kvm:/usr/src/bridge-utils# ./configure && make && make install
root@kvm:/usr/src/bridge-utils# brctl --version
bridge-utils, 1.5
root@kvm:/usr/src/bridge-utils#

```

从前面的输出中，我们可以看到我们首先克隆了`git`仓库中的`bridge-utils`项目，然后编译了源代码。

在 RedHat/CentOS 主机上，过程类似：

```
[root@centos ~]# cd /usr/src/
[root@centos src]#
[root@centos src]# yum groupinstall "Development tools"
[root@centos src]# git clone git://git.kernel.org/pub/scm/linux/kernel/git/shemminger/bridge-utils.git
Cloning into 'bridge-utils'...
remote: Counting objects: 654, done.
remote: Total 654 (delta 0), reused 0 (delta 0)
Receiving objects: 100% (654/654), 131.72 KiB | 198.00 KiB/s, done.
Resolving deltas: 100% (425/425), done.
Checking connectivity... done.
[root@centos src]# cd bridge-utils
[root@centos bridge-utils]# autoconf
[root@centos bridge-utils]# ./configure && make && make install
[root@centos bridge-utils]# brctl --version
bridge-utils, 1.5
[root@centos bridge-utils]#

```

# Open vSwitch

OVS 是另一种可以用来创建各种虚拟网络拓扑并将 KVM 实例连接到其上的软件桥接/交换设备。OVS 可以替代 Linux 桥接，并提供广泛的功能集，包括策略路由、**访问控制列表** (**ACLs**)、**服务质量** (**QoS**) 管控、流量监控、流量管理、VLAN 标签、GRE 隧道等功能。

在本教程中，我们将安装、配置并使用 OVS 桥接，将 KVM 实例连接到宿主操作系统，方式类似于我们在上一教程中使用 Linux 桥接所做的操作。

# 准备工作

为了使本教程能够顺利进行，我们需要确保以下几点：

+   如果存在，Linux 桥接将被删除，并安装 OVS

+   我们至少有一个 KVM 实例在运行

# 如何操作...

创建新的 OVS 桥接并连接 KVM 客户机的虚拟接口，按照以下步骤操作：

1.  如果存在，删除现有的 Linux 桥接：

```
root@kvm:~# brctl show
bridge name      bridge id             STP enabled      interfaces
virbr0           8000.fe5400559bd6     yes              vnet0
root@kvm:~# ifconfig virbr0 down
root@kvm:~# brctl delbr virbr0
root@kvm:~# brctl show
bridge name bridge id STP enabled interfaces
root@kvm:~#

```

在某些 Linux 发行版中，使用 OVS 之前，卸载 Linux 桥接的内核模块是有帮助的。为此，执行 `root@kvm:/usr/src# modprobe -r bridge`。

1.  在 Ubuntu 上安装 OVS 软件包：

```
root@kvm:~# apt-get install openvswitch-switch
...
Setting up openvswitch-common (2.0.2-0ubuntu0.14.04.3) ...
Setting up openvswitch-switch (2.0.2-0ubuntu0.14.04.3) ...
openvswitch-switch start/running
...
root@kvm:~#

```

1.  确保 OVS 进程正在运行：

```
root@kvm:~# pgrep -lfa switch
22255 ovsdb-server /etc/openvswitch/conf.db -vconsole:emer -vsyslog:err -vfile:info --remote=punix:/var/run/openvswitch/db.sock --private-key=db:Open_vSwitch,SSL,private_key --certificate=db:Open_vSwitch,SSL,certificate --bootstrap-ca-cert=db:Open_vSwitch,SSL,ca_cert --no-chdir --log-file=/var/log/openvswitch/ovsdb-server.log --pidfile=/var/run/openvswitch/ovsdb-server.pid --detach --monitor
22264 ovs-vswitchd: monitoring pid 22265 (healthy)
22265 ovs-vswitchd unix:/var/run/openvswitch/db.sock -vconsole:emer -vsyslog:err -vfile:info --mlockall --no-chdir --log-file=/var/log/openvswitch/ovs-vswitchd.log --pidfile=/var/run/openvswitch/ovs-vswitchd.pid --detach --monitor
root@kvm:~#

```

1.  确保 OVS 内核模块已加载：

```
root@kvm:~# lsmod | grep switch
openvswitch       70989   0
gre               13796   1 openvswitch
vxlan             37611   1 openvswitch
libcrc32c         12644   1 openvswitch
root@kvm:~#

```

1.  列出可用的 OVS 交换机：

```
root@kvm:~# ovs-vsctl show
e5164e3e-7897-4717-b766-eae1918077b0
 ovs_version: "2.0.2"
root@kvm:~#

```

1.  创建新的 OVS 交换机：

```
root@kvm:~# ovs-vsctl add-br virbr1
root@kvm:~# ovs-vsctl show
e5164e3e-7897-4717-b766-eae1918077b0
 Bridge "virbr1"
 Port "virbr1"
 Interface "virbr1"
 type: internal
 ovs_version: "2.0.2"
root@kvm:~#

```

1.  将正在运行的 KVM 实例的接口添加到 OVS 交换机：

```
root@kvm:~# ovs-vsctl add-port virbr1 vnet0
root@kvm:~# ovs-vsctl show
e5164e3e-7897-4717-b766-eae1918077b0
 Bridge "virbr1"
 Port "virbr1"
 Interface "virbr1"
 type: internal
 Port "vnet0"
 Interface "vnet0"
 ovs_version: "2.0.2"
root@kvm:~#

```

1.  在 OVS 交换机上配置 IP 地址：

```
root@kvm:~# ip addr add 192.168.122.1/24 dev virbr1
root@kvm:~# ip addr show virbr1
41: virbr1: <BROADCAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default
 link/ether b2:52:e0:73:89:4e brd ff:ff:ff:ff:ff:ff
 inet 192.168.122.1/24 scope global virbr1
 valid_lft forever preferred_lft forever
 inet6 fe80::b0a8:c2ff:fed4:bb3f/64 scope link
 valid_lft forever preferred_lft forever
root@kvm:~#

```

1.  在 KVM 客户机内配置 IP 地址，并确保可以连接到宿主操作系统（如果镜像没有控制台访问权限，则配置并使用 VNC 连接）：

```
root@kvm:~# virsh console kvm1
Connected to domain kvm1
Escape character is ^]

root@debian:~# ifconfig eth0 up && ip addr add 192.168.122.210/24 dev eth0
root@debian:~# ip addr show eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
 link/ether 52:54:00:55:9b:d6 brd ff:ff:ff:ff:ff:ff
 inet 192.168.122.210/24 scope global eth0
 valid_lft forever preferred_lft forever
 inet6 fe80::5054:ff:fe55:9bd6/64 scope link
 valid_lft forever preferred_lft forever
root@debian:~# ping 192.168.122.1
PING 192.168.122.1 (192.168.122.1) 56(84) bytes of data.
64 bytes from 192.168.122.1: icmp_seq=1 ttl=64 time=0.711 ms
64 bytes from 192.168.122.1: icmp_seq=2 ttl=64 time=0.394 ms
64 bytes from 192.168.122.1: icmp_seq=3 ttl=64 time=0.243 ms
^C
--- 192.168.122.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2001ms
rtt min/avg/max/mdev = 0.243/0.449/0.711/0.195 ms
root@debian:~#

```

# 工作原理...

为了简化设置并避免冲突，最好先删除 Linux 桥接，然后再创建新的 OVS 交换机。我们在第 1 步中删除了桥接，并选择性地卸载了内核模块。

在第 2 步中，我们安装了 OVS 软件包，该软件包还启动了负责在宿主操作系统上创建和修改桥接/交换机的主要 OVS 守护进程 `ovs-vswitchd`。

在第 4 步中，我们确保 OVS 内核模块已经加载，并在第 5 步中列出了宿主上所有可用的 OVS 交换机。

在步骤 6 和 7 中，我们创建一个新的 OVS 交换机并将 KVM 虚拟接口添加到该交换机。

安装软件包后启动的 `ovsdb`-server 进程，如第 3 步中的输出所示，是一个数据库引擎，通过 JSON **远程过程调用** (**RPC**) 与主要 OVS 守护进程进行通信。`ovsdb` 服务器进程存储了信息，如交换机网络流、端口和 `QoS` 等。你可以通过以下命令查询数据库：

```
root@kvm:~# ovsdb-client list-dbs
Open_vSwitch
root@kvm:~# ovsdb-client list-tables
Table
-------------------------
Port
Manager
Bridge
Interface
SSL
IPFIX
Open_vSwitch
Queue
NetFlow
Mirror
QoS
Controller
Flow_Table
sFlow
Flow_Sample_Collector_Set
root@kvm:~# ovsdb-client dump Open_vSwitch
...
Port table
_uuid bond_downdelay bond_fake_iface bond_mode bond_updelay external_ids fake_bridge interfaces lacp mac name other_config qos statistics status tag trunks vlan_mode
------------------------------------ -------------- --------------- --------- ------------ ------------ ----------- -------------------------------------- ---- --- -------- ------------ --- ---------- ------ --- ------ ---------
9b4b743d-66b2-4779-9dd8-404b3aa55e18 0 false [] 0 {} false [e7ed4e2b-a73c-46c7-adeb-a203be56587c] [] [] "virbr1" {} [] {} {} [] [] []
f2a033aa-9072-4be3-808e-6e0fce67ce7b 0 false [] 0 {} false [86a10eed-698f-4ccc-b3b7-dd20c13e3ee3] [] [] "vnet0" {} [] {} {} [] [] []
...
root@kvm:~#

```

从前面的输出中可以看到，新的交换机 `virbr1` 和端口 `vnet0` 现在出现在查询 OVS 数据库时。

在步骤 8 和 9 中，我们为 OVS 交换机和 KVM 客户机分配了 IP 地址，并确保能够从虚拟机内部访问宿主桥接。

# 还有更多...

OVS 是一个相当复杂的软件交换机；在这个步骤中，我们只是粗略了解了它。接下来的几个步骤中，我们将结合使用 Linux 桥接和 OVS，只需在 libvirt 中做些许配置更改，我们将在操作中指出这些变化。

要从 OVS 开关中移除 KVM 虚拟接口，请执行以下命令：

```
root@kvm:~# ovs-vsctl del-port virbr1 vnet0
root@kvm:~#

```

要完全删除 OVS 开关，请运行以下命令：

```
root@kvm:~# ovs-vsctl del-br virbr1 && ovs-vsctl show
e5164e3e-7897-4717-b766-eae1918077b0
 ovs_version: "2.0.2"
root@kvm:~#

```

要了解有关 OVS 的更多信息，请访问该项目网站：[`openvswitch.org/`](http://openvswitch.org/)。

# 配置 NAT 转发网络

当 `libvirt` 守护进程启动时，它会创建一个默认网络，该网络在 `/etc/libvirt/qemu/networks/default.xml` 配置文件中定义。当创建一个新的 KVM 客户机而没有指定任何网络选项时，它将使用默认网络与主机操作系统和其他客户机以及网络进行通信。默认的 `libvirt` 网络使用 **网络地址转换**（**NAT**）方法。NAT 通过修改 IP 数据报头中的 IP 地址来将一个 IP 地址空间映射到另一个地址空间。这在主机操作系统提供一个 IP 地址并允许多个客户机共享该地址以建立外部连接时特别有用。虚拟机的 IP 地址本质上会被转换为主机机器的 IP 地址。

默认的 NAT 转发网络定义并设置了一个 Linux 桥接，供客户机连接。在这个步骤中，我们将探索默认的 NAT 网络并了解用于定义它的 XML 属性。接着，我们将创建一个新的 NAT 网络，并将 KVM 客户机连接到它。

# 准备工作

本步骤中，我们需要以下内容：

+   一台安装了 libvirt 并正在运行守护进程的 Linux 主机。

+   主机操作系统上已安装 `iptables` 和 `iproute2` 包。如果你是通过包管理安装 libvirt，那么 `iptables` 和 `iproute2` 很可能作为 `libvirt` 的依赖包一起安装。如果你是从源代码编译安装 libvirt，可能需要手动安装这些包。

+   一个运行中的 KVM 实例。

# 如何操作...

要配置新的 NAT 网络并将 KVM 实例连接到它，请运行以下命令：

1.  列出所有可用的网络：

```
root@kvm:~# virsh net-list --all
 Name       State    Autostart     Persistent
----------------------------------------------------------
 default    active    yes          yes

root@kvm:~#

```

1.  导出默认网络的配置：

```
root@kvm:~# virsh net-dumpxml default
<network connections='1'>
 <name>default</name>
 <uuid>2ab5d22c-5928-4304-920e-bc43b8731bcf</uuid>
 <forward mode='nat'>
 <nat>
 <port start='1024' end='65535'/>
 </nat>
 </forward>
 <bridge name='virbr0' stp='on' delay='0'/>
 <ip address='192.168.122.1' netmask='255.255.255.0'>
 <dhcp>
 <range start='192.168.122.2' end='192.168.122.254'/>
 </dhcp>
 </ip>
</network>

root@kvm:~#

```

1.  与默认网络的 XML 定义文件进行比较：

```
root@kvm:~# cat /etc/libvirt/qemu/networks/default.xml
<network>
 <name>default</name>
 <bridge name="virbr0"/>
 <forward/>
 <ip address="192.168.122.1" netmask="255.255.255.0">
 <dhcp>
 <range start="192.168.122.2" end="192.168.122.254"/>
 </dhcp>
 </ip>
</network>
root@kvm:~#

```

1.  列出主机上所有运行中的实例：

```
root@kvm:~# virsh list --all
 Id    Name    State
----------------------------------------------------
 3    kvm1     running

root@kvm:~#

```

1.  确保 KVM 实例连接到默认的 Linux 桥接网络：

```
root@kvm:~# brctl show
bridge name    bridge id           STP enabled    interfaces
virbr0         8000.fe5400559bd6   yes            vnet0
root@kvm:~#

```

1.  创建一个新的 NAT 网络定义：

```
root@kvm:~# cat nat_net.xml
<network>
 <name>nat_net</name>
 <bridge name="virbr1"/>
 <forward/>
 <ip address="10.10.10.1" netmask="255.255.255.0">
 <dhcp>
 <range start="10.10.10.2" end="10.10.10.254"/>
 </dhcp>
 </ip>
</network>
root@kvm:~#

```

1.  定义新网络：

```
root@kvm:~# virsh net-define nat_net.xml
Network nat_net defined from nat_net.xml

root@kvm:~# virsh net-list --all
 Name       State      Autostart    Persistent
----------------------------------------------------------
 default    active     yes          yes
 nat_net    inactive   no           yes

root@kvm:~#

```

1.  启动新网络并启用自动启动：

```
root@kvm:~# virsh net-start nat_net
Network nat_net started

root@kvm:~# virsh net-autostart nat_net
Network nat_net marked as autostarted

root@kvm:~# virsh net-list
 Name      State     Autostart    Persistent
----------------------------------------------------------
 default   active    yes          yes
 nat_net   active    yes          yes

root@kvm:~#

```

1.  获取有关新网络的更多信息：

```
root@kvm:~# virsh net-info nat_net
Name: nat_net
UUID: fba2ca2b-8ca7-4dbb-beee-14799ee04bc3
Active: yes
Persistent: yes
Autostart: yes
Bridge: virbr1

root@kvm:~#

```

1.  编辑 `kvm1` 实例的 XML 定义，并更改源网络的名称：

```
root@kvm:~# virsh edit kvm1
...
 <interface type='network'>
 ...
 <source network='nat_net'/>
 ...
 </interface>
...

Domain kvm1 XML configuration edited.

root@kvm:~#

```

1.  重启 KVM 客户机：

```
root@kvm:~# virsh destroy kvm1
Domain kvm1 destroyed

root@kvm:~# virsh start kvm1
Domain kvm1 started

root@kvm:~#

```

1.  列出主机上的所有软件桥接：

```
root@kvm:~# brctl show
bridge name     bridge id         STP enabled interfaces
virbr0          8000.000000000000 yes
virbr1          8000.525400ba8e2c yes         virbr1-nic
 vnet0
root@kvm:~#

```

1.  连接到 KVM 实例并检查 `eth0` 接口的 IP 地址，确保可以连接到主机桥接（如果镜像未配置控制台访问，请改用 VNC 客户端）：

```
root@kvm:~# virsh console kvm1
Connected to domain kvm1
Escape character is ^]

Debian GNU/Linux 8 debian ttyS0

debian login: root
Password:
...
root@debian:~# ip a s eth0 | grep inet
 inet 10.10.10.92/24 brd 10.10.10.255 scope global eth0
 inet6 fe80::5054:ff:fe55:9bd6/64 scope link
root@debian:~# ifconfig eth0 up && dhclient eth0
root@debian:~# ping 10.10.10.1 -c 3
PING 10.10.10.1 (10.10.10.1) 56(84) bytes of data.
64 bytes from 10.10.10.1: icmp_seq=1 ttl=64 time=0.313 ms
64 bytes from 10.10.10.1: icmp_seq=2 ttl=64 time=0.136 ms
64 bytes from 10.10.10.1: icmp_seq=3 ttl=64 time=0.253 ms

--- 10.10.10.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2000ms
rtt min/avg/max/mdev = 0.136/0.234/0.313/0.073 ms
root@debian:~#

```

1.  在主机操作系统上，检查正在运行的 DHCP 服务：

```
root@kvm:~# pgrep -lfa dnsmasq
38983 /usr/sbin/dnsmasq --conf-file=/var/lib/libvirt/dnsmasq/default.conf
40098 /usr/sbin/dnsmasq --conf-file=/var/lib/libvirt/dnsmasq/nat_net.conf
root@kvm:~#

```

1.  检查新桥接接口的 IP：

```
root@kvm:~# ip a s virbr1
43: virbr1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
 link/ether 52:54:00:ba:8e:2c brd ff:ff:ff:ff:ff:ff
 inet 10.10.10.1/24 brd 10.10.10.255 scope global virbr1
 valid_lft forever preferred_lft forever
root@kvm:~#

```

1.  列出`iptables`规则以供 NAT 表使用：

```
root@kvm:~# iptables -L -n -t nat
Chain PREROUTING (policy ACCEPT)
target prot opt source destination

Chain INPUT (policy ACCEPT)
target prot opt source destination

Chain OUTPUT (policy ACCEPT)
target prot opt source destination

Chain POSTROUTING (policy ACCEPT)
target prot opt source destination
RETURN all -- 10.10.10.0/24 224.0.0.0/24
RETURN all -- 10.10.10.0/24 255.255.255.255
MASQUERADE tcp -- 10.10.10.0/24 !10.10.10.0/24 masq ports: 1024-65535
MASQUERADE udp -- 10.10.10.0/24 !10.10.10.0/24 masq ports: 1024-65535
MASQUERADE all -- 10.10.10.0/24 !10.10.10.0/24
RETURN all -- 192.168.122.0/24 224.0.0.0/24
RETURN all -- 192.168.122.0/24 255.255.255.255
MASQUERADE tcp -- 192.168.122.0/24 !192.168.122.0/24 masq ports: 1024-65535
MASQUERADE udp -- 192.168.122.0/24 !192.168.122.0/24 masq ports: 1024-65535
MASQUERADE all -- 192.168.122.0/24 !192.168.122.0/24
RETURN all -- 192.168.122.0/24 224.0.0.0/24
RETURN all -- 192.168.122.0/24 255.255.255.255
MASQUERADE tcp -- 192.168.122.0/24 !192.168.122.0/24 masq ports: 1024-65535
MASQUERADE udp -- 192.168.122.0/24 !192.168.122.0/24 masq ports: 1024-65535
MASQUERADE all -- 192.168.122.0/24 !192.168.122.0/24
root@kvm:~#

```

# 它是如何工作的……

我们从第一步开始，列出主机操作系统上所有可用的网络。从`virsh`命令的输出中可以看到，当前只运行了一个默认网络。

在第二步中，我们检查了默认网络的配置。XML 定义使用了以下属性：

+   `<network>`属性是根元素，指示 libvirt 我们正在定义一个网络。

+   `<name>`元素指定网络的名称，且该名称必须唯一。

+   `<uuid>`属性提供了虚拟网络的全球唯一标识符，若省略该属性，它将自动生成。

+   `<forward>`元素及其模式属性将网络定义为连接到主机网络堆栈，使用 NAT。如果缺少此元素，libvirt 将创建一个隔离网络。

+   `<nat>`子元素进一步定义了主机执行 NAT 时将使用的`<port>`范围。

+   `<bridge>`元素指定要创建的桥接、其名称和 STP 选项。

+   `<ip>`属性定义了 DHCP 服务器为来宾虚拟机分配地址的 IP 范围。

在第三步中，我们查看该默认网络的配置文件。注意，有些属性缺失。Libvirt 会自动生成某些属性，并在适当的地方分配默认值。

在第 4 步和第 5 步中，我们确保有一个运行实例连接到默认的 Linux 桥接。

在第 6 步中，我们使用默认网络作为模板创建新的网络定义。我们更改了网络的名称并定义了新的 IP 范围。

准备好新的网络定义文件后，在第 7 步和第 8 步中，我们定义了新网络，启动它，并确保在`libvirt`守护进程启动时它会自动启动，以防服务器重启。

在第 9 步获取了有关新创建的网络的更多信息后，我们继续在第 10 步编辑 KVM 来宾的 XML 定义。为了让虚拟机成为新网络的一部分，我们只需更新`<source network>`元素。

在第 11 步重新启动 KVM 来宾后，我们继续在第 12 步列出主机操作系统上所有可用的软件桥接。请注意，现在我们有了两个桥接，其中新的桥接连接了虚拟机的虚拟接口`vnet0`。

然后，我们连接到运行中的 KVM 来宾，确保其 eth0 网络接口从主机上的 DHCP 服务器获取到了 IP 地址，且该 IP 属于我们先前配置的地址范围。我们还通过`ping`命令确保能够连接到主机桥接。

返回主机操作系统，在第 14 步和第 15 步中，我们检查了正在运行的 DHCP 服务。注意，从`pgrep`命令的输出中，我们现在有两个`dnsmasq`进程在运行：每个网络各一个。

NAT 转发是通过设置 iptables 规则来实现的，如第 18 步所示。每次我们定义并启动一个新的 NAT 网络时，libvirt 都会在 iptables 中创建所需的规则。从第 18 步的输出中，我们可以观察到有两组 NAT 规则，每个运行的 NAT 网络都有一组。

# 配置桥接网络

使用完全桥接后，我们可以直接将 KVM 客户机连接到主机网络，而不使用 NAT。然而，这种设置需要为每个虚拟机分配一个属于主机子网的 IP 地址。如果你无法分配这么多 IP 地址，可以考虑使用之前提到的 *配置 NAT 转发网络* 的设置。在这种网络模式下，虚拟机仍然使用主机操作系统的桥接来进行连接；然而，桥接会控制将用于虚拟机的物理接口。

# 准备工作

对于这个配方，我们需要以下内容：

+   至少有两个物理接口的服务器

+   使用 libvirt 配置和启动 KVM 实例的能力

+   一个正在运行的 KVM 实例

# 如何操作...

要定义一个新的桥接网络并将客户机连接到它，请按照以下步骤操作：

1.  关闭我们将要桥接的接口：

```
root@kvm:~# ifdown eth1
root@kvm:~#

```

1.  如果你的主机操作系统是 Debian/Ubuntu，请编辑主机上的网络配置文件，并用以下内容替换 `eth1` 块：

```
root@kvm:~# vim /etc/network/interfaces
...
 auto virbr2
 iface virbr2 inet static
 address 192.168.1.2
 netmask 255.255.255.0
 network 192.168.1.0
 broadcast 192.168.1.255
 gateway 192.168.1.1
 bridge_ports eth1
 bridge_stp on
 bridge_maxwait 0
...
root@kvm:~#

```

1.  如果使用 RedHat/CentOS 发行版，请编辑以下两个文件：

```
root@kvm:~# cat /etc/sysconfig/ifcfg-eth1
DEVICE=eth1
NAME=eth1
NM_CONTROLLED=yes
ONBOOT=yes
TYPE=Ethernet
BRIDGE=virbr2
root@kvm:~# cat /etc/sysconfig/ifcfg-bridge_net
DEVICE=virbr2
NAME=virbr2
NM_CONTROLLED=yes
ONBOOT=yes
TYPE=Bridge
STP=on
IPADDR=192.168.1.2
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
root@kvm:~#

```

1.  启动新接口：

```
root@kvm:~# ifup virbr2
root@kvm:~#

```

1.  禁止将来自客户机虚拟机的包发送到 `iptables`：

```
root@kvm:~# sysctl -w net.bridge.bridge-nf-call-iptables=0
net.bridge.bridge-nf-call-iptables = 0
root@kvm:~# sysctl -w net.bridge.bridge-nf-call-iptables=0
net.bridge.bridge-nf-call-iptables = 0
root@kvm:~# sysctl -w net.bridge.bridge-nf-call-arptables=0
net.bridge.bridge-nf-call-arptables = 0
root@kvm:~#

```

1.  列出主机上的所有桥接：

```
root@kvm:~# # brctl show
 bridge name     bridge id            STP enabled     interfaces
 virbr0          8000.000000000000    yes
 virbr2          8000.000a0ac60210    yes             eth1
root@kvm:~#

```

1.  编辑 KVM 实例的 XML 定义：

```
root@kvm:~# virsh edit kvm1
...
 <interface type='bridge'>
 <source bridge='virbr2'/>
 </interface>
...

Domain kvm1 XML configuration edited.

root@kvm:~#

```

1.  重启 KVM 实例：

```
root@kvm:~# virsh destroy kvm1
Domain kvm1 destroyed

root@kvm:~# virsh start kvm1
Domain kvm1 started

root@kvm:~#

```

# 它是如何工作的...

要设置桥接网络，在第 1 步和第 2 步中，我们首先将物理接口（本例中为 `eth1`）关闭，以便将其奴役（使其成为我们将要创建的新桥的一部分）。然后，我们创建网络配置，指定新的桥接和将成为该桥接一部分的物理接口。这样，实际上将物理接口上配置的子网映射到桥接上。如果你的服务器只有一个网络接口，仍然可以将其奴役。然而，你将需要另一种连接到服务器的方式，因为一旦关闭主接口，你将失去连接，且通过 SSH 连接进行故障排除可能变得不可能。

一旦新桥接配置完成，我们将在第 3 步启动它。

在第 4 步中，我们指示内核不要对连接到 Linux 桥接的虚拟客户机发出的任何流量应用 iptable 规则，因为我们不使用任何 NAT 规则。

在新接口启动后，我们可以在第 5 步看到桥接和附加到它的物理接口。

在步骤 6 中，我们编辑`kvm1`实例的 XML 定义，在其中指定我们希望使用的网络类型；对于本教程，这是桥接网络。如果你还记得*配置 NAT 转发网络*教程，我们使用的是网络类型而非桥接，并且我们指定了`libvirt`网络名称，而不是桥接名称。

最后，在步骤 7 中重新启动 KVM 实例后，客户操作系统现在应该能够在不使用 NAT 的情况下访问属于同一子网的其他实例。

# 配置 PCI 直通网络

KVM 虚拟化管理程序支持将主机操作系统中的 PCI 设备直接附加到虚拟机上。我们可以利用这个功能将网络接口直接附加到客户操作系统，而无需使用 NAT 或软件桥接。

在本教程中，我们将连接支持 SR-IOV **单根 I/O 虚拟化**（**SR-IOV**）的**网络接口卡**（**NIC**），将其从主机附加到 KVM 客户机。SR-IOV 是一种规范，允许**外设组件互连快速**（**PCIe**）设备表现为多个独立的物理设备，可以在同一主机上的多个虚拟机之间共享，绕过虚拟化层，从而实现原生网络速度。像 Amazon AWS 这样的云服务提供商通过 API 调用将这一功能暴露给其 EC2 计算实例。

# 准备工作

为了完成本教程，我们需要以下内容：

+   支持 SR-IOV 的 NIC 的物理主机

+   具有物理服务器连接的`802.1Qbh`兼容交换机

+   配备 Intel VT-d 或 AMD IOMMU 扩展的 CPU

+   安装了`libvirt`的 Linux 主机，准备好部署 KVM 实例

# 操作步骤...

设置新的 PCI 直通网络，按照以下步骤进行：

1.  枚举主机操作系统上的所有设备：

```
root@kvm:~# virsh nodedev-list --tree
computer
 |
 +- net_lo_00_00_00_00_00_00
 +- net_ovs_system_0a_c6_62_34_19_b4
 +- net_virbr1_nic_52_54_00_ba_8e_2c
 +- net_vnet0_fe_54_00_55_9b_d6
 ...
 |
 +- pci_0000_00_03_0
 | |
 | +- pci_0000_03_00_0
 | | |
 | | +- net_eth0_58_20_b1_00_b8_61
 | |
 | +- pci_0000_03_00_1
 | |
 | +- net_eth1_58_20_b1_00_b8_61
 |
 ...
 root@kvm:~#

```

1.  列出所有 PCI `以太网`适配器：

```
root@kvm:~# lspci | grep Ethernet
03:00.0 Ethernet controller: Intel Corporation 82599ES 10-Gigabit SFI/SFP+ Network Connection (rev 01)
03:00.1 Ethernet controller: Intel Corporation 82599ES 10-Gigabit SFI/SFP+ Network Connection (rev 01)
root@kvm:~#

```

1.  获取关于`eth1`设备使用的 NIC 的更多信息：

```
root@kvm:~# virsh nodedev-dumpxml pci_0000_03_00_1
<device>
 <name>pci_0000_03_00_1</name>
 <path>/sys/devices/pci0000:00/0000:00:03.0/0000:03:00.1</path>
 <parent>pci_0000_00_03_0</parent>
 <driver>
 <name>ixgbe</name>
 </driver>
 <capability type='pci'>
 <domain>0</domain>
 <bus>3</bus>
 <slot>0</slot>
 <function>1</function>
 <product id='0x10fb'>82599ES 10-Gigabit SFI/SFP+ Network Connection</product>
 <vendor id='0x8086'>Intel Corporation</vendor>
 </capability>
</device>

root@kvm:~#

```

1.  将域、总线、插槽和功能值转换为十六进制：

```
root@kvm:~# printf %x 0
0
root@kvm:~# printf %x 3
3
root@kvm:~# printf %x 0
0
root@kvm:~# printf %x 1
1
root@kvm:~#

```

1.  创建新的`libvirt`网络定义文件：

```
root@kvm:~# cat passthrough_net.xml
<network>
 <name>passthrough_net</name>
 <forward mode='hostdev' managed='yes'>
 <pf dev='eth1'/>
 </forward>
</network>
root@kvm:~#

```

1.  定义、启动并启用新的`libvirt`网络的自动启动：

```
root@kvm:~# virsh net-define passthrough_net.xml
Network passthrough_net defined from passthrough_net.xml

root@kvm:~# virsh net-start passthrough_net
Network passthrough_nett started

root@kvm:~# virsh net-autostart passthrough_net
Network passthrough_net marked as autostarted

root@kvm:~# virsh net-list
 Name               State      Autostart    Persistent
----------------------------------------------------------
 default            active    yes           yes
 passthrough_net    active    yes           yes

root@kvm:~#

```

1.  编辑 KVM 客户机的 XML 定义：

```
root@kvm:~# virsh edit kvm1
...
 <devices>
 ...
 <interface type='hostdev' managed='yes'>
 <source>
 <address type='pci' domain='0x0' bus='0x00' slot='0x07' function='0x0'/>
 </source>
 <virtualport type='802.1Qbh' />
 </interface>
 <interface type='network'>
 <source network='passthrough_net'>
 </interface>
 ...
 </devices>
...

Domain kvm1 XML configuration edited.

root@kvm:~#

```

1.  重启 KVM 实例：

```
root@kvm:~# virsh destroy kvm1
Domain kvm1 destroyed

root@kvm:~# virsh start kvm1
Domain kvm1 started

root@kvm:~#

```

1.  列出 SR-IOV NIC 提供的**虚拟功能**（**VFs**）：

```
root@kvm:~# virsh net-dumpxml passthrough_net
<network connections='1'>
 <name>passthrough_net</name>
 <uuid>a4233231-d353-a112-3422-3451ac78623a</uuid>
 <forward mode='hostdev' managed='yes'>
 <pf dev='eth1'/>
 <address type='pci' domain='0x0000' bus='0x02' slot='0x10' function='0x1'/>
 <address type='pci' domain='0x0000' bus='0x02' slot='0x10' function='0x3'/>
 <address type='pci' domain='0x0000' bus='0x02' slot='0x10' function='0x5'/>
 <address type='pci' domain='0x0000' bus='0x02' slot='0x10' function='0x7'/>
 <address type='pci' domain='0x0000' bus='0x02' slot='0x11' function='0x1'/>
 <address type='pci' domain='0x0000' bus='0x02' slot='0x11' function='0x3'/>
 <address type='pci' domain='0x0000' bus='0x02' slot='0x11' function='0x5'/>
 </forward>
</network>
root@kvm:~#

```

# 工作原理...

为了将主机操作系统中的 PCI NIC 直接附加到客户虚拟机，我们首先需要收集一些关于设备的硬件信息，比如域、总线、插槽和功能 ID。在步骤 1 中，我们收集了主机服务器上所有可用设备的信息。我们希望使用 eth1 网络接口作为本例中的设备，因此我们从输出中记下了唯一的 PCI 标识符——`pci_0000_03_00_1`。

为了确认这是我们希望暴露给客户机的 NIC，我们在步骤 2 中列出了所有 PCI 设备。从输出中可以看到，PCI ID 是相同的`03:00.1`。

使用第 1 步中的 PCI ID，我们继续在第 3 步收集有关 NIC 的更多信息。请注意，`0000_03_00_1 ID`被分解为域 ID、总线 ID、插槽 ID 和功能 ID，如 XML 属性所示。我们将在第 7 步使用这些 ID；但是，我们首先需要将它们转换为十六进制格式，这将在第 4 步完成。

在第 5 和第 6 步中，我们为我们的虚拟机定义了一个新的`libvirt`网络，启动网络，并启用自动启动，以防主机服务器重新启动。如果你完成了本章中的其他食谱，你应该已经熟悉我们刚刚创建的网络 XML 定义文件中的大多数属性。`<forward>`属性中定义的`hostdev`模式指示`libvirt`新网络将使用 PCI 直通。`<forward>`属性中指定的`managed=yes`参数告诉`libvirt`在将 PCI 设备传递给虚拟机之前，首先将其从主机中分离，并在虚拟机终止后重新将其附加回主机。最后，`<pf>`子元素指定将被虚拟化并呈现给虚拟机的物理接口。

有关可用 XML 属性的更多信息，请参考[`libvirt.org/formatdomain.html`](http://libvirt.org/formatdomain.html)。

在第 7 步中，我们编辑 KVM 实例的 XML 定义，指定在第 3 步中获取的 PCI ID，并定义一个将使用我们在第 5 和第 6 步中创建的新的 PCI 直通网络的接口。

我们在第 8 步重启 KVM 实例，最后验证物理 PCI NIC 设备现在已成为我们之前定义的新直通网络的一部分。请注意，出现了多个 PCI 类型的设备。这是因为我们使用的 PCI 直通设备支持 SR-IOV。所有将使用此网络的 KVM 虚拟机现在都可以通过分配其中一个列出的虚拟 PCI 设备来直接使用主机的 NIC。

# 操作网络接口

Libvirt 提供了一种方便的方式，通过已经熟悉的 XML 定义语法来管理主机上的网络接口。我们可以使用`virsh`命令来定义、配置和删除 Linux 桥接，并获取有关现有网络接口的更多信息，就像你在本章中已经看到的那样。

在本食谱中，我们将定义一个新的 Linux 桥接、创建它，并最终使用`virsh`将其删除。如果你还记得之前的食谱，我们可以通过`brctl`等工具来操作 Linux 桥接。然而，使用 libvirt，我们有一种通过编写定义文件并使用 API 绑定来以编程方式控制它的方法，正如我们在第七章*《使用 Python 构建和管理 KVM 实例》*中将看到的那样。

# 准备工作

对于这个食谱，我们需要以下材料：

+   主机上安装了`libvirt`包

+   具有桥接内核模块的 Linux 主机

# 操作步骤...

要使用 libvirt 创建新的桥接接口，请运行以下命令：

1.  创建一个新的桥接接口配置文件：

```
root@kvm:~# cat test_bridge.xml
<interface type='bridge' name='test_bridge'>
 <start mode="onboot"/>
 <protocol family='ipv4'>
 <ip address='192.168.1.100' prefix='24'/>
 </protocol>
 <bridge>
 <interface type='ethernet' name='vnet0'>
 <mac address='fe:54:00:55:9b:d6'/>
 </interface>
 </bridge>
</interface>
root@kvm:~#

```

1.  定义新的接口：

```
root@kvm:~# virsh iface-define test_bridge.xml
Interface test_bridge defined from test_bridge.xml

root@kvm:~#

```

1.  列出所有 libvirt 知道的接口：

```
root@kvm:~# virsh iface-list --all
 Name            State       MAC Address
---------------------------------------------------
 bond0           active      58:20:b1:00:b8:61
 bond0.129       active      bc:76:4e:20:10:6b
 bond0.229       active      bc:76:4e:20:17:7e
 eth0            active      58:20:b1:00:b8:61
 eth1            active      58:20:b1:00:b8:61
 lo              active      00:00:00:00:00:00
 test_bridge     inactive

root@kvm:~#

```

1.  启动新的桥接接口：

```
root@kvm:~# virsh iface-start test_bridge
Interface test_bridge started

root@kvm:~# virsh iface-list --all | grep test_bridge
 test_bridge    active    4a:1e:48:e1:e7:de
root@kvm:~#

```

1.  列出主机上的所有桥接设备：

```
root@kvm:~# brctl show
bridge name       bridge id           STP enabled   interfaces
test_bridge       8000.000000000000   no
virbr0            8000.000000000000   yes
virbr1            8000.525400ba8e2c   yes           virbr1-nic
 vnet0
root@kvm:~#

```

1.  检查新桥接的活动网络配置：

```
root@kvm:~# ip a s test_bridge
46: test_bridge: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default
 link/ether 4a:1e:48:e1:e7:de brd ff:ff:ff:ff:ff:ff
 inet 192.168.1.100/24 brd 192.168.1.255 scope global test_bridge
 valid_lft forever preferred_lft forever
 inet6 fe80::481e:48ff:fee1:e7de/64 scope link
 valid_lft forever preferred_lft forever
root@kvm:~#

```

1.  获取桥接的 MAC 地址：

```
root@kvm:~# virsh iface-mac test_bridge
4a:1e:48:e1:e7:de

root@kvm:~#

```

1.  通过提供 MAC 地址获取桥接的名称：

```
root@kvm:~# virsh iface-name 4a:1e:48:e1:e7:de
test_bridge

root@kvm:~#

```

1.  销毁接口，方法如下：

```
root@kvm:~# virsh iface-destroy test_bridge
Interface test_bridge destroyed

root@kvm:~# virsh iface-list --all | grep test_bridge
 test_bridge inactive
root@kvm:~# virsh iface-undefine test_bridge
Interface test_bridge undefined

root@kvm:~# virsh iface-list --all | grep test_bridge
root@kvm:~#

```

# 它是如何工作的...

在第 1 步中，我们为新的网络接口编写 XML 定义。我们指定桥接（bridge）作为类型，为接口指定一个 IP 地址，并可选地指定一个 MAC 地址。

在第 2 步和第 3 步中，我们定义新的桥接接口并列出它。定义接口并不会自动使其生效，因此我们在第 4 步中激活它。

启动桥接会在主机上创建实际的接口，如第 5 步所示。

在第 6 步中，我们确认分配给桥接的 IP 地址和 MAC 地址确实是我们在第 1 步中指定的。

在第 7 步和第 8 步中，我们使用 `virsh` 工具获得接口的名称和 MAC 地址，最后在第 9 步中，我们移除 `bridge` 接口。
