# 第六章：高级用户的脚本

本章将涵盖以下内容：

+   创建 Syslog 条目并生成警报

+   使用 DD 备份和擦除媒体、磁盘和分区

+   在 CLI 上创建图形和演示文稿

+   检查文件完整性和篡改

+   挂载网络文件系统并检索文件

+   从 CLI 浏览网页

+   无头捕获网络流量

+   查找二进制依赖关系

+   从不同位置获取时间

+   从脚本加密/解密文件

# 简介

本章将帮助高级用户了解如何通过 shell 脚本执行特定任务。这些任务包括使用`logger`命令创建 syslog 条目、备份、在 CLI 上创建图形和演示文稿、检查文件完整性和篡改、挂载网络文件系统并检索文件、浏览网页、捕获网络流量、查找二进制依赖关系，以及加密和解密文件。在本章中，用户将学习如何使用脚本来完成这些任务。

# 创建 Syslog 条目并生成警报

在本节中，我们将讨论 syslog 协议。我们还将学习`logger`命令，它是一个 shell 命令，并充当 syslog 模块的接口。`logger`命令可以在系统日志中创建条目。在本节中，我们还将通过脚本创建一个警报。

# 准备工作

除了打开终端，我们还需要确保你有一个文件来进行条目记录。

# 如何操作...

1.  我们将使用`logger`命令将`file_name`输入到`syslog`文件中。运行以下命令：

```
$ logger -f file_name
```

1.  现在我们将编写一个脚本来创建一个警报。创建一个`create_alarm.sh`脚本并在其中写入以下代码：

```
#!/bin/bash
declare -i H
declare -i M
declare -i cur_H
declare -i cur_M
declare -i min_left
declare -i hour_left
echo -e "What time do you Wake Up?"
read H
echo -e "and Minutes?"
read  M
cur_H=`date +%H`
cur_M=`date +%M`
echo "You Selected "
echo "$H:$M"
echo -e "\nIt is Currently $cur_H:$cur_M"
if [  $cur_H -lt $H ]; then
    hour_left=`expr $H - $cur_H`
    echo "$H - $cur_H means You Have: $hour_left hours still"
fi
if [ $cur_H -gt $H ]; then
    hour_left=`expr $cur_H - $H`
    echo -e  "\n$cur_H - $H means you have $hour_left hours left \n"
fi
if [ $cur_H == $H ]; then
    hour_left=0
    echo -e "Taking a nap?\n"
fi
if [ $cur_M -lt $M ]; then
    min_left=`expr $M - $cur_M`
    echo -e "$M -$cur_M you have: $min_left minutes still"
fi
if [ $cur_M -gt $M ]; then
    min_left=`expr $cur_M - $M`
    echo -e "$cur_M - $M you have $min_left minutes left \n"
fi
if [ $cur_M == $M ]; then
    min_left=0
    echo -e "and no minutes\n"
fi

echo -e "Sleeping for $hour_left hours and $min_left minutes \n"
sleep $hour_left\h
sleep $min_left\m
mplayer ~/.alarm/alarm.mp3
```

# 它是如何工作的...

现在我们将看到我们刚刚编写的`logger`命令和`create_alarm`脚本的描述：

1.  `logger`命令在`syslog`文件中创建了关于你文件的条目，该文件位于你系统的`/var/log`目录下。你可以检查该文件。进入`/var/log`目录并运行`nano syslog`，你将找到该文件中的条目。

1.  我们创建了一个脚本来创建一个警报。我们使用了`date`命令获取日期和时间。我们还使用了`sleep`命令来阻止警报在特定时间触发。

# 使用 DD 备份和擦除媒体、磁盘和分区

在本节中，我们将讨论`dd`命令。`dd`命令代表*数据复制器*，主要用于转换和复制文件。在本节中，我们将学习如何备份和擦除媒体文件。

# 准备工作

除了打开终端，我们还需要确保你在当前目录中有必要的文件来进行备份、复制以及类似任务。

# 如何操作...

`dd`命令主要用于转换和复制文件。`if`参数代表输入文件，它是源文件。`of`代表输出文件，它是我们想要粘贴数据的目标。

1.  运行以下命令将一个文件的内容复制到另一个文件：

```
# create a file 01.txt and add some content in that file.
# create another file 02.txt and add some content in that file.
$ dd if=/home/student/work/01.txt of=/home/student/work/02.txt bs=512 count=1
```

1.  运行以下命令备份分区或硬盘：

```
$ sudo dd if=/dev/sda2 of=/home/student/hdbackup.img
```

1.  `dd` 命令也可用于擦除磁盘的所有内容。运行以下命令删除内容：

```
$ sudo dd if=/home/student/work/1.sh
```

# 它是如何工作的……

现在我们将看到之前的命令如何工作：

1.  我们使用 `dd` 命令将 `01.txt` 文件的内容复制到了 `02.txt` 文件中。

1.  要运行此命令，我们必须具有超级用户权限。使用 `dd` 命令，我们创建了备份并将其存储在 `hdbackup.img` 文件中。

1.  使用 `dd` 命令，我们清除了 `1.sh` 文件的内容。

# 在 CLI 上创建图形和演示文稿

在这一部分，我们将学习如何制作演示文稿以及如何在 CLI 上创建图形。为此，我们将使用名为 **dialog** 的工具。`dialog` 是一个 Linux 命令行工具，用于从用户那里获取输入并创建消息框。

# 准备工作

除了打开终端外，请确保你的系统中已安装对话工具。使用 `apt` 命令安装它。APT 代表高级软件包工具。通过 `apt` 命令，你可以从命令行管理 Debian 系列 Linux 的软件。`apt` 命令可以轻松与 `dpkg` 包管理系统交互。

# 如何操作……

1.  我们将编写一个 `Yes`/`No` 框的脚本。在该脚本中，我们将使用 `if` 条件。创建 `yes_no.sh` 脚本，并将以下内容添加到其中：

```
dialog --yesno "Do you wish to continue?" 0 0
a=$?
if [ "${a}" == "0" ]; then
    echo Yes
else
    echo No
fi
```

1.  我们将使用 `dialog` 的日历功能。创建一个 `calendar_dialog.sh` 脚本。在其中，我们将选择一个特定的日期：

```
dialog --calendar "Select a date... " 0 0 1 1 2018
val=$?
```

1.  我们将使用 `dialog` 的清单选项。创建一个 `checklist_dialog.sh` 脚本。在其中，我们将选择多个选项：

```
dialog --stdout --checklist "Enable the account options you want:" 10 40 3 \
              1 "Home directory" on \
              2 "Signature file" off \
              3 "Simple password" off
```

1.  现在，我们将编写一个脚本来提高图像的边框。创建一个 `raise_border.sh` 脚本。我们将使用 `convert` 命令并加上 `raise` 选项：

```
convert -raise 5x5 mountain.png mountain-raised.png
```

# 它是如何工作的……

现在我们将看到前面脚本中选项和命令的描述：

1.  我们使用 Linux 中的对话工具编写了一个 `Yes`/`No` 框的代码。我们使用 `if` 条件来获取 `Yes` 或 `No` 的回答。

1.  我们使用了对话工具的 `--calendar` 选项，它要求选择一个日期。我们选择了 2018 年的一个日期。

1.  我们使用了对话工具的 `checklist` 选项，制作了一个包含三个选项的清单：主目录、签名文件和简单密码。

1.  我们使用 `convert` 命令和 `–raise` 选项提高了图像的边框，然后将新图像保存为 `mountain-raised.png`。

# 检查文件的完整性和篡改

在这一部分，我们将学习如何检查文件的完整性以及如何通过编写简单的 Shell 脚本检查文件是否被篡改。为什么我们需要检查完整性？答案很简单：当服务器上存在密码和库文件，或者文件包含高度敏感数据时，管理员需要检查完整性。

# 准备工作

除了打开终端，还需要确保必要的文件和目录已准备好。

# 如何操作...

1.  我们将编写一个脚本来检查目录中的文件是否被篡改。创建一个`integrity_check.sh`脚本，并将以下代码添加到其中：

```
#!/bin/bash
E_DIR_NOMATCH=50
E_BAD_DBFILE=51
dbfile=Filerec.md5
# storing records.
set_up_database ()
{
    echo ""$directory"" > "$dbfile"
    # Write directory name to first line of file.
    md5sum "$directory"/* >> "$dbfile"
    # Append md5 checksums and filenames.
}
check_database ()
{
    local n=0
    local filename
    local checksum
    if [ ! -r "$dbfile" ]
    then
        echo "Unable to read checksum database file!"
        exit $E_BAD_DBFILE
    fi

    while read rec[n]
    do
        directory_checked="${rec[0]}"
        if [ "$directory_checked" != "$directory" ]
        then
            echo "Directories do not match up!"
            # Tried to use file for a different directory.
            exit $E_DIR_NOMATCH
        fi
        if [ "$n" -gt 0 ]
        then
            filename[n]=$( echo ${rec[$n]} | awk '{ print $2 }' )
            # md5sum writes recs backwards,
            #+ checksum first, then filename.
            checksum[n]=$( md5sum "${filename[n]}" )
            if [ "${rec[n]}" = "${checksum[n]}" ]
            then
                echo "${filename[n]} unchanged."
            else
                echo "${filename[n]} : CHECKSUM ERROR!"
            fi
        fi
        let "n+=1"
        done <"$dbfile" # Read from checksum database file.
}
if [ -z "$1" ]
then
    directory="$PWD" # If not specified,
 else
    directory="$1"
fi
clear
if [ ! -r "$dbfile" ]
then
    echo "Setting up database file, \""$directory"/"$dbfile"\".";
    echo
    set_up_database
fi
check_database
echo
exit 0
```

# 它是如何工作的...

当我们运行这个脚本时，它将创建一个名为`filerec.md5`的数据库文件，其中包含该目录中所有文件的数据。我们将使用这些文件作为参考。

# 挂载网络文件系统并获取文件

在本节中，我们将学习`mount`命令。要将文件系统挂载到文件系统树上，使用`mount`命令。此命令会指示内核挂载在特定设备上找到的文件系统。树中每个挂载的分区都有一个挂载点。

# 准备工作

除了打开终端，还需确保必要的文件和目录已准备好以进行挂载

# 如何操作...

1.  我们将使用`mount`命令来挂载文件系统。然后，我们将使用`ro`和`noexec`选项进行挂载：

```
$ mount -t ext4 /directorytobemounted /directoryinwhichitismounted -o ro,noexec
```

1.  我们也可以使用默认选项挂载设备。运行以下命令使用默认选项挂载设备：

```
$ mount -t ext4 /directorytobemounted /directoryinwhichitismounted -o defaults
```

1.  `scp`命令用于在两台主机之间安全传输文件。我们可以将文件从本地主机传输到远程主机，也可以在两台远程主机之间传输文件。运行以下命令将文件从远程主机传输到本地主机：

```
$ scp *from_host_name*:filename ***/local_directory_name*** 
```

# 它是如何工作的...

1.  我们使用了`ext4`文件系统。在`mount`命令中，我们首先指定了要挂载的目录，然后是我们要挂载它的目标目录，并使用了`ro`和`noexec`选项。

1.  我们使用默认选项挂载了目录。

1.  我们使用了`scp`命令将文件从远程主机复制到本地主机。

# 从命令行浏览网页

在本节中，我们将学习如何通过命令行浏览网页。我们将使用 w3m 和 ELinks 浏览器通过命令行浏览网页。

**w3m**是一个基于文本的网页浏览器。使用 w3m，我们可以通过终端窗口浏览网页。

**ELinks**也是一个基于文本的网页浏览器。它支持基于菜单的配置、框架、表格、浏览和后台下载。ELinks 可以处理远程 URL 和本地文件。

# 准备工作

除了打开终端，我们还需要记住几点：

+   确保你已经安装了 w3m

+   确保你已经安装了 ELinks

# 如何操作...

1.  我们将看到如何使用`w3m`从命令行浏览网页。安装成功后，只需打开终端窗口，输入`w3m`，后跟网站名称：

```
$ w3m google.com
```

1.  我们将看到如何使用`elinks`从命令行浏览网页。安装成功后，只需打开终端窗口，输入`elinks`，后跟网站名称：

```
$ elinks google.com
```

# 它是如何工作的...

1.  要浏览网站，请使用以下键盘组合：

    1.  *Shift* + *U*：此组合将打开一个新网页

    1.  *Shift* + *B*：此组合将使你返回上一网页

    1.  *Shift* + *T*：此组合将打开一个新标签页

1.  以下是使用 ELinks 浏览网站的键盘快捷键：

    1.  转到网址 - g

    1.  打开新标签页 - t

    1.  向前 - u

    1.  返回 - 左

    1.  退出 - q

    1.  上一个标签页 - <

    1.  下一个标签页 - >

    1.  关闭标签页 - c

# 无头捕获网络流量

在本节中，我们将学习如何捕获流量。我们将使用一个名为**tcpdump**的抓包工具来捕获网络流量。该工具用于过滤或捕获通过网络传输或接收的 TCP/IP 数据包。

# 准备工作

除了打开终端外，我们还需要记住一些概念：

+   确保你的机器上安装了 tcpdump 工具

# 如何操作...

现在我们将使用一些`tcpdump`命令来捕获数据包：

1.  要从某个接口捕获数据包，请使用以下代码：

```
$ sudo tcpdump -i eth0
```

1.  要以 ASCII 值格式打印捕获的数据包，请使用以下代码：

```
$ sudo tcpdump -A -i eth0
```

1.  要捕获特定数量的数据包，请使用以下代码：

```
$ sudo tcpdump -c 10 -i eth0
```

1.  要以 HEX 和 ASCII 格式打印捕获的数据包，请使用以下代码：

```
$ sudo tcpdump -XX -i eth0
```

1.  要在特定文件中捕获并保存数据包，请使用以下代码：

```
$ sudo tcpdump -w 111.pcap -i eth0
```

1.  要捕获 IP 地址数据包，请使用以下代码：

```
$ sudo tcpdump -n -i eth0
```

1.  要读取捕获的数据包，请使用以下代码：

```
$ sudo tcpdump -r 111.pcap
```

现在我们将查看`tcpdump`及我们使用的命令的解释。

# 它是如何工作的...

我们使用了 tcpdump Linux 工具，它用于捕获或过滤数据包。tcpdump 用于捕获特定接口上的数据包。我们为此使用了`-i`选项。我们可以将捕获的数据包保存到文件中。只需指定文件名并在`tcpdump`命令中指定`-w`选项。我们可以通过在`tcpdump`命令中指定`-r`选项来读取该文件。

# 查找二进制依赖项

在本节中，我们将检查可执行文件。我们将通过使用`string`命令来查找其中的字符串。

# 准备工作

除了打开终端，还要确保目录中有二进制文件。

# 如何操作...

1.  首先，我们将检查可执行文件。运行以下命令：

```
$ file binary_name
```

1.  现在我们将编写一个命令来查找二进制文件中的字符串。运行以下命令：

```
$ strings binary_name
```

1.  我们可以通过运行以下命令对文件进行十六进制转储：

```
$ od -tx1 binary_name
```

1.  我们可以通过运行以下命令列出二进制文件中的符号：

```
$ nm binary_name
```

1.  你可以通过运行以下命令检查它已链接到哪个共享库：

```
$ ldd binary_name
```

# 它是如何工作的...

现在我们将查看之前命令的解释：

1.  我们使用了`file`命令来获取二进制文件的信息。我们还通过运行`file`命令获取了架构信息。

1.  `string`命令将返回该二进制文件中的字符串。

1.  通过运行`od`命令，你将获得文件的十六进制转储。

1.  二进制文件中存在符号。你可以通过运行`nm`命令列出这些符号。

1.  通过运行 `ldd` 命令，你可以检查你的二进制文件链接了哪些共享库。

# 从不同地点获取时间

在本节中，我们将学习如何使用 `date` 命令从不同的时区获取时间。

# 准备开始

除了打开终端，你还需要具备基本的 `date` 命令知识。

# 如何操作...

1.  我们将编写一个 shell 脚本来确定不同时间区域的时间。为此，我们将使用 `date` 命令。创建一个 `timezones.sh` shell 脚本并在其中写入以下代码：

```
TZ=":Antarctica/Casey" date
TZ=":Atlantic/Bermuda" date
TZ=":Asia/Calcutta" date
TZ=":Europe/Amsterdam" date
```

# 如何操作...

在前面的脚本中，我们使用了 `date` 命令来获取时间。我们从四个不同的大陆——南极洲、大西洋、亚洲和欧洲获取了时间。

你可以在系统的 `/usr/share/zoneinfo` 文件夹中找到所有的时区。

# 从脚本中加密/解密文件

在本节中，我们将学习 OpenSSL。在本节中，我们将使用 OpenSSL 加密和解密消息和文件。

# 准备开始

除了打开终端，你还需要具备基本的编码和解码方案知识。

# 如何操作...

1.  我们将加密和解密简单的消息。我们将使用 -base64 编码方案。首先，我们将加密一条消息。请在终端中运行以下命令：

```
$ echo "Welcome to Bash Cookbook" | openssl enc -base64
```

1.  要解密消息，请在终端中运行以下命令：

```
$ echo " V2VsY29tZSB0byBCYXNoIENvb2tib29rCg==" | openssl enc -base64 -d
```

1.  现在我们将加密和解密文件。首先，我们将加密一个文件。请在终端中运行以下命令：

```
$ openssl enc -aes-256-cbc -in /etc/services -out enc_services.dat
```

1.  现在我们将解密一个文件。请在终端中运行以下命令：

```
$ openssl enc -aes-256-cbc -d -in enc_services.dat > services.txt
```

# 如何工作...

**OpenSSL** 是用于加密和解密消息、文件和目录的工具。在前面的例子中，我们使用了 -base64 编码方案。`-d` 选项用于解密。

要加密一个文件，我们使用了 `-in` 选项，后面跟上我们要加密的文件，`-out` 选项指示 OpenSSL 存储该文件。然后它将加密后的文件按照指定的名称存储。
