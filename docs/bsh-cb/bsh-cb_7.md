# 第七章：编写 Bash 赢得和利润

在本章中，我们将介绍以下主题：

+   创建一个简陋的实用 HTTP 服务器

+   解析 RSS 订阅并输出 HTML

+   网络爬虫和文件收集

+   制作一个简单的 IRC 聊天机器人日志记录器

+   阻止由于 SSH 尝试失败而导致的 IP 地址

+   从 Bash 播放和管理音频

+   创建一个简单的 NAT 和 DMZ 防火墙

+   解析 GitHub 项目并生成报告

+   创建一个简易的增量远程备份系统

+   使用 Bash 脚本监控 udev 输入

+   使用 Bash 监控电池寿命并进行优化

+   使用 chroot 和受限 Bash shell 来保护脚本

# 介绍

本章将帮助您学习如何为多个任务使用命令和脚本。您将了解如何编写监控特定任务的 Bash 脚本。您将学习如何托管文件，解析 RSS 订阅并输出 HTML。您还将学习如何从网站复制内容，创建简单的 IRC 聊天机器人日志记录器，阻止 IP 地址，以及如何从命令行播放和管理音频，使用 iptables 创建简单的 NAT 防火墙和 DMZ，解析 GitHub 项目，使用 rsync 创建备份，监控设备事件、系统电池寿命和电源事件，并使用 chroot 提升安全性。

# 创建一个简陋的实用 HTTP 服务器

在此示例中，我们将讨论 Linux 中的 cURL 工具。cURL 用于从服务器传输数据。它支持许多协议，`http` 是其中之一。cURL 用于从 URL 传输数据。它有许多技巧可提供，如 http post、ftp post、用户认证、代理支持、文件传输、SSL 连接、cookies 等。

# 准备工作

除了打开终端，您需要确保系统上安装了 `curl`。

# 如何操作…

我们将学习如何在 Linux 中使用 `curl` 学习 HTTP `GET` 和 `POST` 方法。现在，我们将看到 `GET` 和 `POST` 的示例：

1.  `GET`：HTTP GET 方法用于请求指定资源的数据。该命令是 HTTP GET 的示例，接受 JSON 格式的数据：

```
$ curl -H "Content-Type:application/json" -H "Accept:application/json" -X GET http://host/resource/name
```

另一个接受 XML 格式数据的 HTTP GET 示例如下所示：

```
$ curl -H "Content-Type:application/xml" -H "Accept:application/xml" -X GET http://host/resource/name
```

1.  `POST`：HTTP POST 方法用于向服务器发送数据。它用于创建和更新资源。

一个简单的 POST 示例如下所示：

```
$ curl --data "name1=value1&name2=value2" http://host/resource/name
```

运行以下 `curl` 命令来上传文件：

```
$ curl -T "{File,names,separated,by,comma}" http://host/resource/name
```

# 它是如何工作的…

现在我们将学习 `curl` 命令中使用的选项：

1.  首先，我们学习了 `curl` 命令中的 `GET` 方法。我们接受两种类型的数据：JSON 和 XML。我们使用了 `-X` 选项并指定了请求。`-H` 指定了头部。

1.  其次，我们学习了 `curl` 命令中的 `POST` 方法。`--data` 标志用于 `POST` 请求。在简单示例中，我们仅仅是发送了简单的数据，在下一个示例中，我们使用 `-T` 选项来上传文件。

# 解析 RSS 订阅并输出 HTML

在此示例中，我们将使用 Linux 的 `curl` 和 `xml2` 工具来解析 RSS 订阅。

# 准备工作

+   您需要确保系统上安装了 `curl`。

+   你需要确保系统上已安装`xml2`

# 如何操作…

我们将在`curl`中提供一个 URL，并使用`xml2`工具解析 RSS 提要。在终端中运行以下命令：

```
$ curl -sL https://imdb.com | xml2 | head
```

# 它是如何工作的…

我们使用了`xml2`和`curl`Linux 工具来解析 XML 格式的 RSS 提要；`xml2`将 XML 文档转换为文本。在输出中，每一行都是一个键，即 XML 路径，值通过`=`分隔。

# 爬取网页并收集文件

在这个教程中，我们将学习如何通过网络爬虫收集数据。我们将为此编写一个脚本。

# 准备工作

除了打开终端，你还需要掌握基本的`grep`和`wget`命令。

# 如何操作…

现在，我们将编写一个脚本来抓取`imdb.com`的内容。我们将在脚本中使用`grep`和`wget`命令来获取内容。创建`scrap_contents.sh`脚本，并写入以下代码：

```
$ mkdir -p data
$ cd data
$ wget -q -r -l5 -x 5  https://imdb.com
$ cd ..
$ grep -r -Po -h '(?<=href=")[^"]*' data/ > links.csv
$ grep "^http" links.csv > links_filtered.csv
$ sort -u links_filtered.csv > links_final.csv
$ rm -rf data links.csv links_filtered.csv
```

# 它是如何工作的…

在前面的脚本中，我们编写了代码来获取网站的内容。**wget**工具用于通过`http`、`https`和`ftp`协议从网络上下载文件。在这个例子中，我们从`imdb.com`获取数据，因此我们在`wget`中指定了网站名称。`grep`是一个命令行工具，用于搜索匹配正则表达式的数据。在这里，我们正在搜索特定的链接，这些链接会在网页抓取后保存在`link_final.csv`中。

# 创建一个简单的 IRC 聊天机器人日志记录器

在这个教程中，我们将创建一个简单的机器人日志记录器。这个脚本将记录几个频道并处理 pings。

# 准备工作

除了打开终端，你还需要掌握 IRC 的基本知识。

# 如何操作…

现在，我们将编写一个 IRC 日志记录机器人脚本。创建`logging_bot.sh`脚本，并写入以下代码：

```
#!/bin/bash
nick="blb$$"
channel=testchannel
server=irc.freenode.net
config=/tmp/irclog
[ -n "$1" ] && channel=$1
[ -n "$2" ] && server=$2
config="${config}_${channel}"
echo "NICK $nick" > $config
echo "USER $nick +i * :$0" >> $config
echo "JOIN #$channel" >> $config
trap "rm -f $config;exit 0" INT TERM EXIT
tail -f $config | nc $server 6667 | while read MESSAGE
do
    case "$MESSAGE" in
        PING*) echo "PONG${MESSAGE#PING}" >> $config;;                                  *QUIT*) ;;
        *PART*) ;;
        *JOIN*) ;;
        *NICK*) ;;
        *PRIVMSG*) echo "${MESSAGE}" | sed -nr "s/^:([^!]+).*PRIVMSG[^:]+:(.*)/[$(date '+%R')] \1> \2/p";;
        *) echo "${MESSAGE}";;
    esac
done
```

# 它是如何工作的…

在这个脚本中，我们处理了 pings，同时也在记录一些频道。

# 阻止 SSH 登录失败的 IP 地址

在这个教程中，我们将学习如何找到失败的 SSH 登录尝试并阻止这些 IP 地址。为了查找失败的尝试，我们将使用`grep`和`cat`命令。SSH 服务器的登录尝试会被追踪并记录到`rsyslog`守护进程中。

# 准备工作

除了打开终端，我们需要记住一些概念：

+   `grep`和`cat`命令的基础知识

+   确保安装了`grep`

# 如何操作…

我们将使用`grep`和`cat`命令查找失败的 SSH 登录尝试。首先，成为 root 用户，输入`sudo su`命令。接下来，运行以下命令，通过`grep`命令获取失败的尝试：

```
# grep "Failed password" /var/log/auth.log
```

你也可以使用`cat`命令来完成这个操作。运行以下命令：

```
# cat /var/log/auth.log | grep "Failed password"
```

你可以使用 tcp-wrapper 阻止某个特定 IP 地址的 SSH 登录失败尝试。首先进入`/etc`目录，查找`hosts.deny`文件，在文件中添加以下内容并保存：

```
sshd: 192.168.0.1/255.255.255.0
```

# 它是如何工作的…

在这其中，我们使用了`cat`和`grep`命令。`cat`命令最常见的用途是显示文件内容，而`grep`是一个 Linux 工具，用于在文件中搜索特定模式；然后，它将显示包含该特定模式的行。

在前面的示例中，我们在查找失败的登录尝试。我们使用`grep`命令匹配这些关键字，然后用`cat`命令显示它们。

要阻止一个 IP 地址，我们只需在`hosts.deny`文件中添加一行，这将阻止该特定的 IP 地址。

# 从 Bash 播放和管理音频

在本教程中，我们将了解如何使用一个名为**SoX**的命令行播放器从命令行播放音乐。SoX 支持大多数音频格式，如 mp3、wav、mpg 等。

# 准备工作

除了打开终端，我们还需要记住一些概念：

+   确保您的系统中已安装 SoX

+   确保您已安装 sox libsox-fmt-all

# 如何操作…

1.  我们将从命令行播放音频。为此，我们将使用 SoX 命令行播放器。在成功安装`sox`和`libsox-fmt-all`后，导航到存有音频文件的目录，并运行以下命令以播放所有`.mp3`文件：

```
$ play *mp3
```

1.  要播放特定的歌曲，请运行以下命令：

```
$ play file_name.mp3
```

# 工作原理…

SoX 用于读取和写入音频文件。`libsox`库是 SoX 工具的核心。`play`命令用于播放音频文件。要播放所有 mp3 文件，我们使用`*mp3`。要播放特定文件，只需在`play`命令后写入文件名及扩展名。

# 创建一个简单的 NAT 和 DMZ 防火墙

在本教程中，我们将使用 iptables 创建一个简单的 NAT 防火墙和 DMZ。

# 准备工作

除了打开终端，您还需要确保您的机器中已安装`iptables`。

# 如何操作…

我们将编写一个脚本来使用`iptables`设置 DMZ。创建一个`dmz_iptables.sh`脚本，并在其中写入以下代码：

```
# set the default policy to DROP
iptables -P INPUT DROP
iptables -P OUTPUT DROP
iptables -P FORWARD DROP
# to configure the system as a router, enable ip forwarding by
sysctl -w net.ipv4.ip_forward=1
# allow traffic from internal (eth0) to DMZ (eth2)
iptables -t filter -A FORWARD -i eth0 -o eth2 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT
iptables -t filter -A FORWARD -i eth2 -o eth0 -m state --state ESTABLISHED,RELATED -j ACCEPT
# allow traffic from internet (ens33) to DMZ (eth2)
iptables -t filter -A FORWARD -i ens33 -o eth2 -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT
iptables -t filter -A FORWARD -i eth2 -o ens33 -m state --state ESTABLISHED,RELATED -j ACCEPT
#redirect incoming web requests at ens33 (200.0.0.1) of FIREWALL to web server at 192.168.20.2
iptables -t nat -A PREROUTING -p tcp -i ens33 -d 200.0.0.1 --dport 80 -j DNAT --to-dest 192.168.20.2 
iptables -t nat -A PREROUTING -p tcp -i ens33 -d 200.0.0.1 --dport 443 -j DNAT --to-dest 192.168.20.2
#redirect incoming mail (SMTP) requests at ens33 (200.0.0.1) of FIREWALL to Mail server at 192.168.20.3
iptables -t nat -A PREROUTING -p tcp -i ens33 -d 200.0.0.1 --dport 25 -j DNAT --to-dest 192.168.20.3
#redirect incoming DNS requests at ens33 (200.0.0.1) of FIREWALL to DNS server at 192.168.20.4
iptables -t nat -A PREROUTING -p udp -i ens33 -d 200.0.0.1 --dport 53 -j DNAT --to-dest 192.168.20.4
iptables -t nat -A PREROUTING -p tcp -i ens33 -d 200.0.0.1 --dport 53 -j DNAT --to-dest 192.168.20.4
```

# 工作原理…

在前面的代码中，我们使用了`iptables`来设置 DMZ。在这个脚本中，我们允许从互联网到 DMZ 的内部流量。

# 解析 GitHub 项目并生成报告

在本教程中，我们将使用 git 命令解析 GitHub 项目。

# 准备工作

除了打开终端，您还需要确保系统中已安装`git`。

# 如何操作…

我们将使用 git 工具来解析项目。运行此命令来解析一个项目：

```
$ git https://github.com/torvalds/linux.git
```

# 工作原理…

工具 git 用于处理大型项目。此命令用于从服务器克隆 git 仓库。

# 创建一个简易的增量远程备份

在本教程中，我们将学习如何创建备份和增量备份。我们将编写一个脚本来获取增量备份。

# 准备工作

除了打开终端，我们还需要记住一些概念：

+   `tar`、`gunzip`和`gzip`命令的基础知识。

+   确保你的系统中存在必要的目录。

# 如何操作……

1.  首先，选择一个你希望备份的目录。我们将使用 `tar` 命令。假设你要备份 `/work` 目录：

```
$ tar cvfz work.tar.gz /work
```

1.  现在，我们将编写一个脚本来执行增量备份。创建一个 `incr_backup.sh` 脚本并在其中编写以下代码：

```
#!/bin/bash
gunzip /work/tar.gz
tar uvf /work.tar /work/
gzip /work.tar
```

# 它是如何工作的……

现在，我们将了解前面命令中使用的选项以及脚本：

1.  在命令中，我们使用的选项如下：

    1.  `c`：此选项将创建一个归档文件。

    1.  `v`：此选项用于详细模式。我们可以查看正在归档的文件。

    1.  `f`：此选项将带你回到文件。

    1.  `z`：此选项用于用 gzip 压缩文件。

1.  在此脚本中，我们使用的选项如下：

    1.  `u`：此选项用于更新归档。

    1.  `v`：表示详细模式。

    1.  `f`：此选项用于返回文件。

# 使用 Bash 脚本监控 udev 输入

在本教程中，我们将学习 **evtest** Linux 工具。此工具用于监控输入设备事件。

# 准备就绪

除了打开一个终端外，你还需要确保系统中已安装 `evtest`。

# 如何操作……

`evtest` 是一个命令行工具。它将显示输入设备的信息。它会显示设备支持的所有事件，然后监控该设备。我们只需以超级用户权限运行 `evtest` 命令。按照如下方式运行该命令：

```
$ sudo evtest /dev/input/event3
```

# 它是如何工作的……

`evtest` 命令将输出如下内容：

```
Input driver version is 1.0.1
Input device ID: bus 0x11 vendor 0x2 product 0x13 version 0x6
Input device name: "VirtualPS/2 VMware VMMouse"
Supported events:
 Event type 0 (EV_SYN)
 Event type 1 (EV_KEY)
 Event code 272 (BTN_LEFT)
 Event code 273 (BTN_RIGHT)
 Event type 2 (EV_REL)
 Event code 0 (REL_X)
 Event code 1 (REL_Y)
 Event code 8 (REL_WHEEL)
Properties:
 Property type 0 (INPUT_PROP_POINTER)
Testing ... (interrupt to exit)
```

输出显示由内核呈现的信息。

# 使用 Bash 监控电池寿命并优化其性能

在本教程中，我们将学习 TLP Linux 工具。**TLP** 是一个命令行工具，用于电源管理，可以优化电池寿命。

# 准备就绪

除了打开一个终端外，你还需要确保系统中已安装 TLP。

# 如何操作……

TLP 的配置文件位于 `/etc/default/` 目录下，文件名为 `tlp`。安装后，它会自动作为服务启动。我们可以通过运行 `systemctl` 命令检查它是否正在运行，如下所示：

```
$ sudo systemctl status tlp
```

运行以下命令以获取操作模式：

```
$ sudo tlp start
```

要获取系统信息以及 TLP 状态，运行以下命令：

```
$ sudo tlp-stat -s
```

要查看 TLP 配置，运行以下命令：

```
$ sudo tlp-stat -c
```

要获取所有电源配置，请运行以下命令：

```
$ sudo tlp-stat
```

要获取电池信息，请使用以下命令：

```
$ sudo tlp-stat -b
```

要获取系统的风扇速度和温度，运行下一个命令：

```
$ sudo tlp-stat -t
```

要获取处理器数据，运行以下命令：

```
$ sudo tlp-stat -p
```

# 它是如何工作的……

TLP 是一个命令行工具，具有自动化后台任务。TLP 有助于优化 Linux 操作系统笔记本电脑的电池寿命。

我们通过运行 `sudo tlp-stat` 并使用各种选项来获取电池寿命、处理器数据、温度和风扇速度的信息。`tlp-stat` 显示电源管理设置。我们与 `tlp-stat` 一起使用的选项如下：

+   `-b`：电池

+   `-t`：温度

+   `-p`：处理器数据

+   `-c`：配置

+   `-s`：系统信息

# 使用 chroot 和受限的 Bash Shell 来确保脚本的安全

在本食谱中，我们将学习 chroot 和受限 bash(rbash)。`chroot`命令用于更改根目录。使用 rbash，我们可以限制 bash shell 的某些功能，以达到一定的安全目的。

# 准备就绪

除了打开终端外，您还需要确保系统中已安装`rbash`。

# 如何操作……

1.  现在，我们来看看启动`rbash`的命令。运行以下命令：

```
$ bash -r
or
$ rbash
```

1.  现在我们将测试一些限制。首先，我们将尝试更改目录。运行以下命令：

```
$ cd work/
```

接下来，我们将尝试向文件中写入一些内容。运行给定的命令将内容写入文件：

```
ls > log.txt
```

# 它是如何工作的……

使用`rbash`后，系统的访问将受到限制。在之前的示例中，我们通过输入`bash -r`或`rbash`来启动受限 shell。

接下来，我们尝试更改目录，但我们收到`rbash: cd: restricted`的消息，因此无法在`rbash`中更改目录。另外，我们也无法向文件中写入内容。
