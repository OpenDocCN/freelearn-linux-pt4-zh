# 系统管理任务的脚本

在本章中，我们将介绍以下主题：

+   收集和聚合系统信息

+   收集网络信息和连接诊断

+   配置基础网络连接

+   监控目录和文件

+   压缩和归档文件

+   将日志文件从 RAM 转移到存储器进行日志轮转

+   使用 Linux iptables 作为防火墙

+   远程或本地访问 SQL 数据库

+   为无密码远程访问创建 SSH 密钥

+   为任务调度创建和配置 cron 作业

+   系统地创建用户和组

# 介绍

本章将讨论执行几乎所有用户常见的系统管理任务，我们将查看日志、归档日志、作业/任务管理、网络连接、使用防火墙（IPtables）保护系统、监控目录变化、创建用户。我们还将承认，用户和管理员经常需要从其他系统访问资源，例如 SQL，或者他们必须使用 SSH 通过加密密钥登录到另一个系统——无需密码！

本章还充当了关于今天计算环境中一些关键组件的速成课程：**网络**。用户可能不知道端口是什么，IP 地址是什么，或者如何查找计算机上的**网络接口**（**NICs**）。本章结束时，初学者应该能够配置网络，并且在使用网络术语时能够提高他们的技能水平。

# 收集和聚合系统信息

在本节中，我们将讨论 `dmidecode` Linux 工具，它将收集系统信息，如 CPU 信息、服务器、内存和网络。

# 准备工作

除了保持终端打开外，我们还需要记住一些概念：

+   我们将使用一些命令，这些命令将为我们提供关于内核、Linux 发行版、物理服务器信息、系统运行时间、网络信息、内存信息和 CPU 信息等方面的信息。

+   通过使用此方法，任何人都可以创建脚本来收集系统信息。

# 如何操作...

1.  我们可以获取关于您正在使用的 Linux 发行版的详细信息。这些发行版有一个发行文件，您可以在`/etc/`文件夹中找到它。现在，打开终端并输入以下命令来获取有关您正在使用的 Linux 发行版的信息：

```
cat /etc/*-release
```

1.  上述选项有一个替代方案，替代方案是位于`/proc`文件夹中的版本文件。所以，运行以下命令：

```
cat /proc/version
```

1.  现在，运行以下命令以获取内核信息：

```
uname -a
```

1.  系统`uptime`：有关此信息，请创建一个名为`server_uptime.sh`的脚本：

```
server_uptime=`uptime | awk '{print $3,$4}'| sed 's/,//'| grep "day"`;
if [[ -z "$server_uptime" ]]; then
 server_uptime=`uptime | awk '{print $3}'| sed 's/,//'`
 echo $server_uptime
else
 :;
fi;
```

1.  现在，我们将运行以下命令来获取物理服务器的信息：

```
$ sudo dmidecode -s system-manufacture
$ sudo dmidecode -s system-product-name
$ sudo dmidecode -s system-serial-number
```

1.  然后，我们将运行以下命令来获取 CPU 的信息：

```
sudo dmidecode -t4|awk '/Handle / {print $2}' |sed 's/,//'
```

您可以从`/proc`目录中的`cpuinfo`文件获取每个 CPU 的信息。

1.  网络信息：我们将通过运行以下命令来获取 IP 地址：

```
ip a
```

要获取 MAC 地址，请运行以下命令：

```
ip addr show ens33
```

你可以根据你的网络接口信息将`ens33`替换为`eth0`。

1.  内存信息：要获取插槽总数，请运行以下命令：

```
sudo dmidecode -t17 |awk '/Handle / {print $2}'|wc -l
```

要获取已填充的插槽总数，请运行以下命令：

```
sudo dmidecode -t17 |awk '/Size:/'|awk '!/No/'|wc -l
```

要获取未填充的插槽总数，请运行以下命令：

```
sudo dmidecode -t17 |awk '/Size:/'|awk '/No/'|wc -l
```

# 它是如何工作的...

在运行`cat /etc/*-release`命令后，按*Enter*键，你将根据你的 Linux 发行版得到相应的输出。在我的情况下，输出如下：

```
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=16.04
DISTRIB_CODENAME=xenial
DISTRIB_DESCRIPTION="Ubuntu 16.04.4 LTS"
NAME="Ubuntu"
VERSION="16.04.4 LTS (Xenial Xerus)"
ID=ubuntu
ID_LIKE=Debian
PRETTY_NAME="Ubuntu 16.04.4 LTS"
VERSION_ID="16.04"
HOME_URL="http://www.ubuntu.com/"
SUPPORT_URL="http://help.ubuntu.com/"
BUG_REPORT_URL="http://bugs.launchpad.net/ubuntu/"
VERSION_CODENAME=xenial
UBUNTU_CODENAME=xenial
```

获取系统信息的另一种方式是通过运行`/proc`目录下的版本文件。运行命令后，按*Enter*键，你将根据你的 Linux 发行版得到相应的输出。在我的情况下，输出如下：

```
Linux version 4.13.0-45-generic (buildd@lgw01-amd64-011) (gcc version 5.4.0 20160609 (Ubuntu 5.4.0-6ubuntu1~16.04.9)) #50~16.04.1-Ubuntu SMP Wed May 30 11:18:27 UTC 2018
```

`uname`是我们用来收集这些信息的工具。运行命令后按*Enter*键，你将得到内核信息。在我的情况下，输出如下：

```
Linux ubuntu 4.13.0-45-generic #50~16.04.1-Ubuntu SMP Wed May 30 11:18:27 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
```

执行`$ bash server_uptime.sh`脚本——在提示符下按*Enter*键，你将得到服务器的运行时间。

为了获取物理服务器信息，我们使用了`dmidecode`工具。第一个命令用于获取制造商信息，第二个命令用于获取型号名称，第三个命令用于获取序列号。在运行`dmidecode`命令时，你必须是 root 用户。

输出片段如下所示：

```
0x0004
0x0005
0x0006
0x0007
```

为了获取每个 CPU 的信息，我们创建了一个脚本。执行`$ sudo bash cpu_info.sh`脚本——在提示符下按*Enter*键，你将得到每个 CPU 的信息。

输出片段如下所示：

```
CPU#000
Unknown
GenuineIntel
1800MHz
3.3V
1
CPU#001
Unknown
GenuineIntel
1800MHz
3.3V
1
CPU#002
Unknown
GenuineIntel
1800MHz
3.3V
1
CPU#003
Unknown
GenuineIntel
1800MHz
3.3V
1
```

网络信息：我们运行了`ip`命令以获取 IP 地址和 MAC 地址。

内存信息：我们使用`dmidecode`工具来获取内存信息，如插槽的总数，以及已填充和未填充的插槽数量。你必须是 root 用户才能运行此命令。

# 收集网络信息和连接诊断

在本节中，我们将测试 IPv4 的连接性并为其编写脚本。

# 准备工作

除了打开终端外，我们还需要记住一些概念：

+   Shell 脚本中的 If..Else 条件语句

+   设备的 IP 地址

+   `curl`命令必须已安装（你可以通过运行以下命令来安装它：`sudo apt install curl`）

本节的目的是向你展示如何检查网络连接。

# 如何操作...

1.  打开终端并创建`test_ipv4.sh`脚本：

```
if ping -q -c 1 -W 1 8.8.8.8 >/dev/null; then
 echo "IPv4 is up"
else
 echo "IPv4 is down"
fi
```

1.  现在，为了测试 IP 连接性和 DNS，创建一个名为`test_ip_dns.sh`的脚本：

```
if ping -q -c 1 -W 1 google.com >/dev/null
then
 echo "The network is up"
else
 echo "The network is down"
fi
```

1.  最后，创建一个名为`test_web.sh`的脚本来测试网页连接：

```
case "$(curl -s --max-time 2 -I http://google.com | sed 's/^[^ ]*  *\([0-9]\).*/\1/; 1q')" in
 [23]) echo "HTTP connectivity is up";;
 5) echo "The web proxy won't let us through";;
 *) echo "The network is down or very slow";;
esac
```

# 它是如何工作的...

1.  执行脚本命令`$ bash test_ipv4.sh`。在这里，我们通过`8.8.8.8` IP 地址检查连接。我们使用`ping`命令在`if`条件中进行测试。如果条件为`true`，我们将在屏幕上显示`if`块中的语句。如果不是，`else`中的语句将被打印出来。

1.  执行脚本命令`$ bash test_ip_dns.sh`。在这里，我们通过主机名测试网络连接。同时，我们在`if`条件中传递`ping`命令，检查网络是否正常。如果条件为`true`，我们将在屏幕上打印`if`块中的语句。如果不是，`else`中的语句将被打印出来。

1.  执行脚本命令`$ bash test_web.sh`。在这里，我们测试 Web 连接。我们在这里使用`case`语句。我们使用 curl 工具进行数据传输，它可以与服务器进行数据交换。

# 配置基本的网络连接

在这一部分，我们将使用`wpa_supplicant`配置基本的网络连接。

# 准备工作

除了打开一个终端，我们还需要记住几个概念：

+   检查`wpa_supplicant`是否已安装。

+   你应该知道 SSID 和密码。

+   记住，你必须以 root 用户身份运行你的程序。

# 如何操作...

创建一个名为`wifi_conn.sh`的脚本，并在其中写入以下代码：

```
#!/bin/bash
ifdown wlan0
rm /etc/network/interfaces
touch /etc/network/interfaces
echo 'auto lo' >> /etc/network/interfaces
echo 'iface lo inet loopback' >> /etc/network/interfaces
echo 'iface eth0 inet dhcp' >> /etc/network/interfaces
echo 'allow-hotplug wlan0' >> /etc/network/interfaces
echo 'iface wlan0 inet manual' >> /etc/network/interfaces
echo 'wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf' >> /etc/network/interfaces
echo 'iface default inet dhcp' >> /etc/network/interfaces
rm /etc/wpa_supplicant/wpa_supplicant.conf
touch /etc/wpa_supplicant/wpa_supplicant.conf
echo 'ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev' >> /etc/wpa_supplicant/wpa_supplicant.conf
echo 'update_config=1' >> /etc/wpa_supplicant/wpa_supplicant.conf
wpa_passphrase $1 $2 >> /etc/wpa_supplicant/wpa_supplicant.conf
ifup wlan0
```

# 它是如何工作的...

使用`sudo`执行脚本：

```
sudo bash wifi_conn.sh <SSID> <password>
```

# 监控目录和文件

`inotify`是 Linux 中的一个工具，用于报告文件系统事件的发生。使用`inotify`，你可以监控单独的文件或目录。

# 准备工作

确保你的系统已安装`inotify`工具。

# 如何操作...

创建一个名为`inotify_example.sh`的脚本：

```
#! /bin/bash
folder=~/Desktop/abc
cdate=$(date +"%Y-%m-%d-%H:%M")
inotifywait -m -q -e create -r --format '%:e %w%f' $folder | while read file
do
    mv ~/Desktop/abc/output.txt ~/Desktop/Old_abc/${cdate}-output.txt
done
```

# 它是如何工作的...

`inotifywait`命令主要用于 Shell 脚本中。`inotify`工具的主要目的是监控目录和新文件。它还监控文件的变化。

# 压缩和归档文件

压缩文件和归档文件是不同的。那么，什么是归档文件呢？它是将多个文件和目录存储在一个文件中的集合。归档文件不是压缩文件。

什么是压缩文件？这是一个将文件和目录存储在一个文件中的集合。它使用更少的磁盘空间进行存储。

在这一部分，我们将讨论两个压缩工具：

+   `bzip2`

+   `zip`

# 准备工作

你应该有文件可以进行归档和压缩。

# 如何操作...

首先，让我们看看如何压缩文件：

1.  我们将通过`bzip2`来进行压缩。选择要压缩的文件并运行以下命令：

```
$ bzip2 *filename*
```

使用`bzip2`，我们可以同时压缩多个文件和目录。只需在每个文件之间加上空格即可。你可以这样压缩：

```
$ bzip2 file1 file2 file3 /student/work
```

1.  我们将使用`zip`进行压缩。使用 zip 压缩工具时，文件是单独压缩的：

```
$ zip -r file_name.zip files_dir
```

执行上述命令后，将被压缩的文件或目录（如`files_zip`）将被压缩，得到一个名为`file_name.zip`的压缩文件。`-r`选项递归包括`files_dir`目录中的所有文件。

现在，让我们讨论如何归档文件：

归档文件包含多个文件或目录。**Tar** 用于无压缩的文件归档。

以下是两种归档模式：

+   `-x`：解压归档文件

+   `-c`：创建归档文件

选项：

+   `-f`：归档文件的文件名——除非使用磁带驱动器进行归档，否则必须指定此项

+   `-v`：详细显示，列出所有正在归档/解压的文件

+   `-z`：使用 gzip/gunzip 创建/解压归档文件

+   `-j`：使用 bzip2/bunzip2 创建/解压归档文件

+   `-J`：使用 XZ 创建/解压归档文件

要创建一个 TAR 文件，使用以下命令：

```
$ tar -cvf filename.tar directory/file
```

在这个示例中，我们表示一个文件，该文件是在使用`tar`命令时创建的。它的名字是`filename.tar`。我们可以指定一个目录或文件来存放归档文件。我们必须在输入`filename.tar`之后指定它。多个文件和目录可以同时归档，只需在文件和目录名称之间留一个空格：

```
$ tar -cvf filename.tar /home/student/work /home/student/school
```

执行上述命令后，所有来自工作和学校目录的内容将被存储在新文件`filename.tar`中。

执行上述命令后，所有来自工作和学校子目录的内容将被存储在新文件`filename.tar`中。

以下命令用于列出 TAR 文件的内容：

```
$ tar -tvf filename.tar
```

要提取 TAR 文件的内容，使用以下命令：

```
$ tar -xvf filename.tar
```

这个命令用于解压，它不会删除 TAR 文件。然而，在当前工作目录中，你将找到解压后的内容。

# 它是如何工作的……

**bzip2** 用于同时压缩文件和文件夹。只需在终端中输入命令，文件将被压缩。压缩后，压缩文件将获得`.bz2`扩展名。

**zip** 将文件单独压缩，然后将它们汇集到一个文件中。

TAR 文件是一个包含不同文件和目录的集合，它将这些文件和目录合并成一个文件。TAR 用于创建备份和归档。

# 将文件从 RAM 转移到存储设备进行日志轮换

在本节中，我们将讨论 logrotate Linux 工具。使用此工具，系统管理变得更加轻松。系统会生成大量的日志文件。它允许自动轮换、删除、压缩和发送日志文件。

我们可以处理每一个日志文件。我们可以按天、周、月来处理它们。通过使用此工具，我们可以在节省磁盘空间的同时，保持日志文件更长时间。默认的配置文件是`/etc/logrotate.conf`。运行以下命令查看此文件的内容：

```
$ cat /etc/logrotate.conf
```

你将看到以下内容：

```
weekly
rotate 4
create

include /etc/logrotate.d
# no packages own wtmp, or btmp -- we'll rotate them here
/var/log/wtmp {
 missingok
 monthly
 create 0664 root utmp
 rotate 1
}

/var/log/btmp {
 missingok
 monthly
 create 0660 root utmp
 rotate 1
}

# system-specific logs may be configured here
```

# 准备工作

要使用 logrotator，必须了解`logrotate`命令。

# 如何操作……

我们将查看一个示例配置。

我们有两种管理日志文件的选项：

1.  创建一个新的配置文件并将其存储在 `/etc/logrotate.d/` 中。这个配置文件会与其他标准任务一起每日执行，并且会使用根用户权限。

1.  创建一个新的配置文件并独立执行。这样会以非根用户权限执行。通过这种方式，你可以手动执行，随时按需求执行。

# 将配置添加到 /etc/logrotate.d/

现在，我们将配置一个 Web 服务器。它将像 `access.log` 和 `error.log` 这样的信息放入 `/var/log/example-app`。它将充当数据用户或组。

要向 `/etc/logrotate.d/` 添加配置，首先打开一个新文件：

```
$ sudo nano /etc/logrotate.d/example-app
```

在 `example-app` 中写入以下代码：

```
/var/log/example-app/*.log {
    daily
    missingok
    rotate 14
    compress
    notifempty
    create 0640 www-data www-data
    sharedscripts
    postrotate
        systemctl reload example-app
    endscript
}

```

此文件中的一些新配置指令如下：

+   `create 0640 www-data www-data`：旋转后，这将创建一个新的空日志文件，并为所有者和组设置指定的权限。

+   `sharedscripts`：这意味着配置脚本在每次运行时仅执行一次，而不是为每个文件进行轮换。

+   `postrotate` 到 `endscript`：这个特定的块包含在日志文件旋转后执行的脚本。使用这个，我们的应用程序可以切换到新创建的日志文件。

# 它是如何工作的...

我们可以根据需要自定义 `.config` 文件，然后将其保存在 `/etc/logrotate.d` 中。为此，运行以下命令：

```
 $ sudo logrotate /etc/logrotate.conf --debug
```

运行此命令后，logrotate 将指向标准配置文件，然后进入调试模式。它会提供有关 logrotate 正在处理的文件的信息。

# 使用 Linux iptables 作为防火墙

在这一部分中，我们将使用 iptables 设置防火墙。**iptables** 是大多数 Linux 发行版中存在的标准防火墙软件。我们将使用这些规则来过滤网络流量。通过指定源或目标 IP 地址、端口地址、协议类型、网络接口等，可以通过过滤数据包来保护服务器免受不必要的流量。我们可以配置它来接受、拒绝或转发网络数据包。

规则按链排列。默认情况下，有三个链（input、output 和 forward）。输入链处理传入流量，而输出链处理传出流量。转发链处理路由流量。如果网络数据包与链中的任何策略不匹配，则每个链都有一个默认策略。

# 准备工作

请在继续下一项活动之前检查以下要求是否已满足：

+   根用户权限

+   SSH 访问（命令行访问服务器）

+   确保你的 Linux 环境中已安装 gt 和 looptools

+   在 Linux 环境中工作的基本技能

# 如何操作...

现在，我们将查看一些 `iptables` 命令：

1.  运行以下命令列出服务器上设置的所有规则：

```
$ sudo iptables -L
```

1.  要允许来自特定端口的传入流量，请使用以下命令：

```
$ sudo iptables -A INPUT -p tcp --dport 4321 -j ACCEPT
```

这条规则将允许来自端口`4321`的传入流量。需要重启防火墙才能使此规则生效。

使用 `iptables`，你可以阻止传入的流量。为此，运行以下命令：

```
$ sudo iptables -A INPUT -j DROP
```

1.  如果在 `iptables` 中添加了新的规则，我们应该先保存它们。否则，在系统重启后，它们将消失。添加新规则后，运行以下命令以保存 `iptables`：

```
$ sudo iptables-save
```

1.  默认保存规则的文件可能会有所不同，这取决于你使用的 Linux 发行版。

1.  我们可以使用以下命令将规则保存到特定文件中：

```
$ sudo iptables-save > /path/to/the/file
```

1.  你可以恢复保存在文件中的这些规则。运行以下命令：

```
$ sudo iptables-restore > /path/to/the/file
```

# 它是如何工作的...

使用 iptables，我们可以控制传入的流量、丢弃特定端口的流量、添加新规则并保存它们。

# 远程或本地访问 SQL 数据库

在本节中，我们将学习如何通过使用 shell 脚本连接到服务器来自动化 SQL 查询。Bash 脚本用于自动化操作。

# 准备就绪

确保已经安装了 `mysql`、`postgres` 和 `sqlite`。确保 MySQL 中已创建用户，并且你已经为该用户授予了权限。

# 如何操作...

1.  脚本中的 MySQL 查询：我们将编写一个名为 `mysql_version.sh` 的脚本，以获取 MySQL 的最新版本：

```
#!/bin/bash
mysql -u root -pTraining2@^ <<MY_QUERY
SELECT VERSION();
MY_QUERY
```

现在，我们将创建一个名为 `create_database.sh` 的脚本来创建数据库：

```
#!/bin/bash
mysql -u root -pTraining2@^ <<MY_QUERY
create database testdb;
MY_QUERY
```

1.  脚本中的 SQLite 查询：现在，我们将创建一个 `sqlite` 数据库。你只需写下 `sqlite3` 和数据库的名称即可创建 `sqlite` 数据库。例如：

```
$ sqlite3 testdb
```

现在，我们将在 `sqlite` 控制台中创建一个表。输入 `sqlite3 testdb` 并按 *Enter*——你将看到 sqlite3 控制台。现在，写入 `create table` 命令以创建表：

```
$  sqlite3 testdb
SQLite version 3.11.0 2016-02-15 17:29:24
Enter ".help" for usage hints.
sqlite> .databases
seq  name             file 
---  ---------------  ----------------------------------------------------------
0   main             /home/student/testdb 
sqlite> CREATE TABLE bookslist(title text, author text);
sqlite> .tables
bookslist
```

1.  脚本中的 Postgres 查询：现在，我们将检查 PostgreSQL 数据库的版本。这里，`testdb`是我们的数据库名称，也是我们之前创建的。为此，运行以下命令：

```
student@ubuntu:~$ sudo -i -u postgres
postgres@ubuntu:~$ psql
psql (9.5.13)
Type "help" for help.
postgres=# create database testdb;
CREATE DATABASE
postgres=# \quit
postgres@ubuntu:~$ psql testdb
psql (9.5.13)
Type "help" for help.
testdb=# select version();
 version 
-------------------------------------------------------------------------------------------
 PostgreSQL 9.5.13 on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 5.4.0-6ubuntu1~16.04.9) 5.4.0 20160609, 64-bit
(1 row)
```

1.  现在，我们将创建一个表。为此，运行以下命令：

```
postgres@ubuntu:~$ psql testdb
psql (9.5.13)
Type "help" for help.
testdb=# create table employee(id integer, name text, address text, designation text, salary integer);
CREATE TABLE
testdb=#
```

# 它是如何工作的...

1.  我们正在编写 bash 脚本，以检查数据库的版本并创建新数据库。在这些脚本中，我们使用 root 用户，并且该用户的密码会紧随 `-p` 后。你可以使用 `root` 用户，或者可以创建一个新用户，分配密码，并在脚本中使用该用户。

1.  SQLite 软件为我们提供了一个简单的命令行界面。通过这个界面，我们可以手动输入和执行 SQL 命令。你可以使用点（`.`）操作符列出数据库。`.databases` 和 `.tables` 用于列出数据库中的所有表。

1.  在 PostgreSQL 中，首先我们将用户从 student 更改为 postgres。然后，输入 `psql` 启动 `postgres` 命令行控制台。在该控制台中，我们必须创建 `testdb` 数据库。要退出控制台，请运行 `\quit` 命令。现在，再次启动 `testdb` 控制台，输入 `psql testdb` 并按 *Enter.* 现在，在该数据库中创建一个表。

# 为密码无障碍远程访问创建 SSH 密钥

在本节中，我们将学习如何使用 SSH 无密码登录。**SSH** 是一个开源网络协议，用于登录远程服务器执行某些操作。我们可以使用 SSH 协议将文件从一台计算机传输到另一台计算机。SSH 使用公钥加密。

# 准备就绪

确保您具有 SSH 访问权限。

# 如何做...

1.  首先，我们将创建一个 SSH 密钥。使用 `ssh-keygen` 命令来创建 SSH 密钥。运行以下命令：

```
$ ssh-keygen
```

您将获得以下输出：

```
Generating public/private rsa key pair.
Enter file in which to save the key (/home/student/.ssh/id_rsa): /home/student/keytext
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/student/keytext.
Your public key has been saved in /home/student/keytext.pub.
The key fingerprint is:
SHA256:6wmj6l9EcjufZhvwQ+iKIqEchO1mtEwC/x5rMyoKyeY student@ubuntu
The key's randomart image is:
+---[RSA 2048]----+
|                 |
|.                |
|oo  . o          |
|o.=  + o         |
|.* o  * S        |
|oo* oo * o       |
|=*.. o= X        |
|O. .*+ * =       |
|+E==+o  +        |
+----[SHA256]-----+
```

1.  现在，我们将 SSH 公钥复制到远程主机。运行以下命令：

```
$ ssh-copy-id remote_hostname
```

1.  现在，您可以无密码登录到远程主机。运行以下 `ssh` 命令：

```
$ ssh remote_hostname
```

# 创建和配置 cron 作业以进行任务调度

在本节中，我们将学习如何配置 Cron 作业。我们将使用 crontab 设置 Cron 作业。

# 如何做...

1.  打开您的终端并转到 `/etc` 文件夹并检查 `/cron` 文件夹。您将看到以下 cron 文件夹：

    +   `/etc/cron.hourly`

    +   `/etc/cron.daily`

    +   `/etc/cron.weekly`

    +   `/etc/cron.monthly`

1.  现在，我们将把我们的 Shell 脚本复制到其中一个上述文件夹中。

1.  如果需要每天运行您的 Shell 脚本，请将其放在 `cron.daily` 文件夹中。如果需要每小时运行，请将其放在 `cron.hourly` 文件夹中，依此类推。

1.  示例：编写一个脚本并将其放在 `cron.daily` 文件夹中。通过授予必要的权限使脚本可执行。

1.  现在，运行 `crontab` 命令：

```
$ crontab -e
```

1.  按下*Enter*，它将询问您的编辑器类型。默认情况下，它将打开`vi`编辑器。在我的情况下，我选择了`nano`。现在，创建一个`cron`命令。创建`cron`命令的语法是：

    +   小时后的分钟数（0 到 59）

    +   小时的军用时间格式（24 小时制）（0 到 23）

    +   月份的日期（1 到 31）

    +   月份（1 到 12）

    +   一周的日期（0 或 7 是星期日，或使用适当的名称）

    +   要运行的命令

1.  如果在脚本名称之前的所有选项中输入`*`，则脚本将每分钟执行，每小时执行一次，每月的每一天执行一次，每年的每个月和每周的每一天执行一次。

1.  现在，在您的脚本中添加以下行：

```
 * * * * * /etc/name_of_cron_folder/script.sh
```

1.  保存文件并退出。

1.  您可以通过运行以下命令列出现有作业：

```
$ crontab -l
```

1.  要删除现有的 Cron 作业，请删除包含您的 Cron 作业的行并保存它。运行以下命令：

```
$ crontab -e
```

# 工作原理...

我们使用 `crontab` 命令添加和删除 Cron 作业。使用适当的设置每天、每小时、每月和每周执行您的脚本。

# 有系统地创建用户和组

在本节中，我们将学习如何通过 shell 脚本创建用户和组。

# 如何实现...

现在，我们将创建一个脚本来添加用户。`useradd` 命令用于创建用户。我们将使用 `while` 循环，它将读取我们的 `.csv` 文件，并且我们将使用 `for` 循环来添加 `.csv` 文件中存在的每个用户。

使用 `add_user.sh` 创建脚本：

```
#!/bin/bash
#set -x
MY_INPUT='/home/mansijoshi/Desktop'
declare -a SURNAME
declare -a NAME
declare -a USERNAME
declare -a DEPARTMENT
declare -a PASSWORD
while IFS=, read -r COL1 COL2 COL3 COL4 COL5 TRASH;
do
    SURNAME+=("$COL1")
    NAME+=("$COL2")
    USERNAME+=("$COL3")
    DEPARTMENT+=("$COL4")
    PASSWORD+=("$COL5")
done <"$MY_INPUT"
for index in "${!USERNAME[@]}"; do
    useradd -g "${DEPARTMENT[$index]}" -d "/home/${USERNAME[$index]}" -s /bin/bash -p "$(echo "${PASSWORD[$index]}" | openssl passwd -1 -stdin)" "${USERNAME[$index]}"
done
```

# 它是如何工作的...

在这个例子中，我们使用了 `while` 和 `for` 循环。`while` 循环将读取我们的 `.csv` 文件，它将为每一列创建数组。我们使用了 `-d` 来指定主目录，`-s` 来指定 bash shell，`-p` 来指定密码。
