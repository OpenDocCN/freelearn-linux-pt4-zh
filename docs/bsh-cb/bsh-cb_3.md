# 第三章：理解并掌握文件系统管理

本章中，我们将介绍以下内容：

+   从不同角度查看文件——头部、尾部、less 和 more

+   按名称和/或扩展名搜索文件

+   创建两个文件的差异并打补丁

+   创建符号链接并有效使用它们

+   爬取文件系统目录并打印目录树

+   查找并删除重复的文件或目录

+   在任意位置合并和拆分文件

+   生成各种大小的数据集和随机文件

# 介绍

本章中，我们将扩展第二章的部分内容，*像打字机和文件浏览器一样操作*，但目标是让你在创建、查看和管理文件时更强大。毕竟，如何查看一个非常大的文件？如何找到二进制文件的外部软件依赖并操作文件？这些任务无疑是每个开发者、管理员或高级用户都能想到的基础任务。

例如，读者 Bob 已经了解了 VI，或许他有自己的 GUI 编辑器或应用程序，如 Open Office，但如果这个编辑器在打开完整文件时崩溃了怎么办？他可以只查看文件的前几行吗？完全可以。他能在 X 行处拆分该文件（如果结构已知，如 CSV 格式）吗？当然可以！

这些事情并非不可能，Bob 可以做的活动清单几乎是无止境的。本章的目的是让你了解如果生活不如意或你需要快速访问/控制系统中的文件时，你可以做的一些事情。

本章的脚本可以在 [`github.com/PacktPublishing/Bash-Cookbook/tree/master/chapter%2003`](https://github.com/PacktPublishing/Bash-Cookbook/tree/master/chapter%2003) 找到。

# 从不同角度查看文件——头部、尾部、less 和 more

目前，你的系统可能有许多不同大小的文本文件，其中还包括一个不断写入的日志文件。你可能还拥有一些包含大量代码的大文件（如 Linux 内核或软件项目），并希望能快速查看这些文件，而不会让系统变得非常缓慢。

为此，有四个基本命令，应该能够为你提供足够的功能来实现它们的目的：

+   **Head**：可以用于输出文件的开头部分

+   **Tail**：可以用于输出文件的末尾部分（也可以连续输出）

+   **More**：一个作为 *分页器* 用于逐页或逐行查看大文件的工具

+   **Less**：与 more 相同，但它有更多功能，包括向后滚动

有时，你可能会在嵌入式系统上看到 `more` 命令，而不是 `less` 命令。这是因为 less 命令比 more 更大。你的头痛了吗？

# 准备工作

除了打开终端之外，本食谱还需要几个大文本文件。如果你已经有了，那太好了；如果没有，可以安装以下内容：

```
$ wget http://www.randomtext.me/download/txt/lorem/ol-20/98-98.txt
$ fmt 98-98.txt > loremipsum.txt
```

`fmt` 命令是一个简单的文本格式化工具，用于清理输出内容，使命令行结果更为整洁。

# 如何操作...

打开终端并运行以下命令：

```
$ cat loremipsum.txt
$ head loremipsum.txt
$ head -n 1 loremipsum.txt
$ tail loremipsum.txt
$ tail -n 1 loremipsum.txt
```

有趣的是，`tail` 命令具有一个不同于 `head` 命令的特性：它可以持续监控文件的尾部，直到命令被退出或终止，前提是使用了 `-f` 或 `-F` 标志。运行以下命令：

```
$ tail -F /var/log/kern.log
```

保持 `tail` 命令运行，尝试断开你的无线或以太网端口。你看到了什么？

按 *Ctrl* + *C* 退出 `tail`，并运行以下命令：

```
$ more loremipsum.txt
```

按下空格键或 *Enter* 键将使你浏览文件直到末尾。按 `q` 将立即退出 `more`，并返回到控制台提示符。

接下来，尝试运行以下命令：

```
$ less loremipsum.txt 
```

尝试使用 *pg up*、*pg dn*、上下箭头键、*Enter* 和空格键浏览文件。注意到什么了吗？

# 它是如何工作的...

在继续之前，请注意，`loremipsum.txt` 文件的内容在每次下载时都会有所不同。**Lorem Ipsum** 是一种伪随机文本，广泛用于各种与文本相关的任务，通常作为占位符值，因为它 *看起来* 像某种语言，且在复制粘贴占位文本时能避免人类大脑的干扰。

太好了！我们开始吧：

1.  在第一步中，命令应该会生成类似于以下的输出（为了简洁，我们省略了部分输出以保持教程的一致性），但注意，`head` 从文件的开头，即 `loremipsum.txt` 的 *head* 开始，而 `tail` 从文件的末尾，即 `loremipsum.txt` 的 *tail* 开始。当我们指定带有小数数字（如 `1`）的 `-n` 标志时，这两个工具都会输出一行或用户输入的任意行数：

```
$ cat loremipsum.txt

Feugiat orci massa inceptos proin adipiscing urna vestibulum
hendrerit morbi convallis commodo porta magna, auctor cras nulla ligula
sit vehicula primis ultrices duis rutrum cras feugiat sit facilisis
fusce placerat sociosqu amet cursus quisque praesent mauris facilisis,
egestas curabitur imperdiet sit elementum ornare sed class ante pharetra
in, nisi luctus sit accumsan iaculis eu platea sit ullamcorper platea
erat convallis orci volutpat curabitur nostra tellus erat non nisl
condimentum, cubilia lacinia eget rhoncus pharetra euismod sagittis
morbi risus, nisl scelerisque fringilla arcu auctor turpis ultricies
imperdiet nibh eget felis leo enim auctor sed netus ultricies sit fames
...
$ head loremipsum.txt
 Feugiat orci massa inceptos proin adipiscing urna vestibulum
hendrerit morbi convallis commodo porta magna, auctor cras nulla ligula
sit vehicula primis ultrices duis rutrum cras feugiat sit facilisis
fusce placerat sociosqu amet cursus quisque praesent mauris facilisis,
egestas curabitur imperdiet sit elementum ornare sed class ante pharetra in
$ head -n 1 loremipsum.txt
 Feugiat orci massa inceptos proin adipiscing urna vestibulum
$ tail loremipsum.txt
euismod torquent primis mattis velit aptent risus accumsan cubilia eros
justo ad sodales dapibus tempor, donec mauris erat at lacinia senectus
luctus venenatis mollis ullamcorper ante mollis nisl leo sollicitudin
felis congue tempus nam curabitur viverra venenatis quis, felis pretium
enim posuere elit bibendum dictumst, bibendum mattis blandit sociosqu
adipiscing cursus quisque augue facilisis vehicula metus taciti conubia
odio proin rutrum aliquam lorem, erat lobortis etiam eget risus lectus
sodales mauris blandit, curabitur velit risus litora tincidunt inceptos
nam ipsum platea felis mi arcu consequat velit viverra, facilisis
 ulputate semper vitae suspendisse aliquam, amet proin potenti semper
$ tail -n 1 loremipsum.txt
 ulputate semper vitae suspendisse aliquam, amet proin potenti semper
```

1.  在第二步中，我们发现 `head` 和 `tail` 命令之间的一个关键区别。`tail` 能够监控文件，并持续将文件的尾部内容输出到标准输出 `(stdout)`。如果文件发生读取错误，或者被移动/旋转出去，`-f`（小写）通常会停止输出信息，而 `-F` 会重新打开文件并继续输出内容。

如果需要监视系统日志，通常在两者之间选择 `-F` 而非 `-f`。

1.  在 `tail` 命令仍然以连续模式运行的情况下，输出中应该会出现一些新的条目。这个示例来自于系统的无线适配器被强制重新连接到标准接入点（AP）时：

```
Dec 8 14:21:40 moon kernel: userif-2: sent link up event.
Dec 8 14:21:40 moon kernel: wlp3s0: authenticate with 18:d6:c7:fa:26:b0
Dec 8 14:21:40 moon kernel: wlp3s0: send auth to 18:d6:c7:fa:26:b0 (try 1/3)
Dec 8 14:21:40 moon kernel: wlp3s0: authenticated
Dec 8 14:21:40 moon kernel: wlp3s0: associate with 18:d6:c7:fa:26:b0 (try 1/3)
Dec 8 14:21:40 moon kernel: wlp3s0: RX AssocResp from 18:d6:c7:fa:26:b0 (capab=0x411 status=0 aid=1)
Dec 8 14:21:40 moon kernel: wlp3s0: associated
Dec 8 14:21:40 moon kernel: bridge-wlp3s0: device is wireless, enabling SMAC
Dec 8 14:21:40 moon kernel: userif-2: sent link down event.
Dec 8 14:21:40 moon kernel: userif-2: sent link up event.
```

1.  杀死 `tail` 命令后，控制台应该会恢复到提示符 `$`。

1.  运行 `more` 并使用空格键或 *Enter* 键可以浏览整个 `loremipsum.txt` 文件。`more` 命令只能从文件开头到末尾查看，无法前后跳转。

1.  `less` 命令无疑更强大，允许用户通过多个键盘组合来浏览 `loremipsum.txt` 文件，还提供了搜索功能等其他特性。

# 按名称和/或扩展名搜索文件

当我们有大量文件需要查看时，有时需要在众多文件中找到一个文件，而不使用图形界面搜索工具，或者提供更好的细粒度筛选器以减少返回的结果。要在命令行中搜索，有几个工具/命令可以使用：

+   `locate`（也是 `updatedb` 命令的兄弟命令）：用于通过文件索引更高效地查找文件

+   `find`：用于在特定目录中查找具有特定属性、扩展名甚至名称的文件

`find` 命令更适合用于命令行，并且广泛应用（通常在嵌入式设备上），而 `locate` 命令则是桌面、笔记本和服务器常用的工具。Locate 要简单得多，它通过递归索引所有配置跟踪的文件，并且能够快速生成文件列表。文件索引可以通过以下命令更新：

```
$ sudo updatedb
```

在第一次更新数据库，或者在大量文件被创建、移动或复制后，更新数据库的时间可能会长于平均时间。保持数据库自动频繁更新的一种特定机制是通过使用 **cron 调度器**。更多关于此主题的内容将在后续介绍。

`locate` 命令也可以在报告文件位置之前测试文件是否存在（数据库可能已经过时），并且可以限制返回的条目数量。

如前所述，`find` 没有一个复杂的数据库，但它有许多用户可配置的标志，这些标志可以在执行时传递给它。一些与 `find` 命令一起使用的常用标志如下：

+   `-type`：用于指定文件类型，可以是文件或目录

+   `-delete`：用于删除文件，但可能不可用，这意味着需要使用 `exec`

+   `-name`：用于指定按名称搜索功能

+   `-exec`：用于指定匹配后的操作

+   `-{a,c,m}time`：用于查找访问、创建和修改时间等信息

+   `-d, -depth`：用于指定搜索的递归深度

+   `-maxdepth`：用于指定每次递归的最大深度

+   `-mindepth`：用于指定递归搜索时的最小深度

+   `-L`, `-H`, `-P`：按顺序，`-L` 跟随符号链接，`-H` 除特定情况外不跟随符号链接，`-P` 永远不跟随符号链接

+   `-print`, `-print0`：这些命令用于打印当前文件的名称到标准输出

+   `!`, `-not`：用于指定逻辑操作，例如匹配所有文件，但不符合此标准

+   `-i`：用于指定在匹配时与用户的交互，如 `-iname test`

请注意，你的平台可能不支持 GNU find 的所有功能。这可能是由于嵌入式系统的限制、资源约束或安全原因。

# 准备工作

除了打开终端，还需要几个大文本文件来完成这个教程。如果你已经有了一些，太好了；如果没有，可以安装以下文件：

```
$ sudo apt-get install locate manpages manpages-posix
$ sudo updatedb
$ git clone https://github.com/PacktPublishing/Linux-Device-Drivers-Development.git Linux-Device-Drivers-Development # Another Packt title
$ mkdir -p ~/emptydir/makesure
```

如果使用 `locate` 命令找不到文件，数据库可能只是过时了，需要重新运行。也可能是 `updatedb` 没有索引如可移动媒体（如 USB 闪存）的分区，文件可能存在于那里，而不是常规系统分区。

在为这个教程做准备时，注意到两个概念不经意间被介绍了：`git` 和 `manpages`。`manpages` 是 Linux 中最古老的帮助文档之一，而 git 是一个版本控制系统，它简化了文件（如代码）的管理、版本控制和分发。了解如何使用它们肯定是有益的，但超出了本书的范围。如需了解更多关于 git 的信息，可以参考 Packt 另一部书籍：*GIT 版本控制实用手册*。

# 如何操作...

1.  打开一个终端，运行以下命令以理解 `locate` 命令：

```
$ locate stdio.h
$ sudo touch /usr/filethatlocatedoesntknow.txt /usr/filethatlocatedoesntknow2.txt
$ sudo sh -c 'echo "My dear Watson ol\'boy" > /usr/filethatlocatedoesntknow.txt'
$ locate filethatlocatedoes
$ sudo updatedb
$ locate filethatlocatedoesntknow
```

1.  接下来，运行以下命令来演示 `find` 的一些强大功能：

```
$ sudo find ${HOME} -name ".*" -ls
$ sudo find / -type d -name ".git"
$ find ${HOME} -type f \( -name "*.sh" -o -name "*.txt" \)

```

1.  接下来，我们可以使用 `&&` 将 `find` 命令链在一起，最终执行 `exec`，而不是将输出传递给另一个进程、命令或脚本。尝试以下命令：

```
$ find . -type d -name ".git" && find . -name ".gitignore" && find . -name ".gitmodules"
$ sudo find / -type f -exec grep -Hi 'My dear Watson ol boy' {} +
```

1.  最后，`find` 最常见的用途之一是删除文件，可以使用内建的 `-delete` 标志，或者通过 `exec` 与 `rm -rf` 结合使用：

```
$ find ~/emptydir -type d -empty -delete
$ find Linux-Device-Drivers-Development -name ".git*" -exec rm -rf {} \;
```

# 它是如何工作的...

跟我重复—"*locate 很简单*，但需要更新，`find` 在困境中有效，但功能强大且晦涩，*可能会破坏系统*。" 让我们继续，并开始讲解：

1.  如前所述，`locate` 命令是一个相对简单的搜索工具，它使用数据库作为后端，该数据库包含所有文件的索引列表，便于快速高效地进行搜索。与 `find` 命令不同，`locate` 不是实时的，`find` 命令会搜索在执行时存在的所有内容（具体取决于提供给 `find` 的参数）。在定位 `stdio.h` 时，结果会根据你的系统不同而有所不同。然而，当我们再次运行 `locate` 时，它并不了解或包含关于 `/usr/filethatlocatedoesntknow.txt` 和 `/usr/filethatlocatedoesntknow2.txt` 文件的信息。运行 `updatedb` 会重新索引文件，之后使用 `locate` 命令将返回相应的结果。注意，`locate` 支持部分名称或完整路径匹配：

```
$ locate stdio.h
/usr/include/stdio.h
/usr/include/c++/5/tr1/stdio.h
/usr/include/x86_64-linux-gnu/bits/stdio.h
/usr/include/x86_64-linux-gnu/unicode/ustdio.h
/usr/lib/x86_64-linux-gnu/perl/5.22.1/CORE/nostdio.h
/usr/share/man/man7/stdio.h.7posix.gz
$ sudo touch /usr/filethatlocatedoesntknow.txt /usr/filethatlocatedoesntknow2.txt
$ sudo sh -c 'echo "My dear Watson ol\'boy" > /usr/filethatlocatedoesntknow.txt'
$ locate filethatlocatedoes
$ sudo updatedb
$ locate filethatlocatedoes
/usr/filethatlocatedoesntknow.txt
/usr/filethatlocatedoesntknow2.txt
```

1.  在第二步中，我们介绍了 `find` 命令提供的一些惊人功能。

再次提醒，使用 `find` 执行如删除操作等可能会破坏你的系统，如果没有适当处理或没有仔细监控和过滤输入。

1.  至少，`find` 命令应按以下方式执行：`$ find ${START_SEARCH_HERE} ${OPTIONAL_PARAMETERS ...}`。在第一次使用 `find` 命令时，我们从用户的主目录（`${HOME}` 环境变量）开始搜索，然后使用通配符查找以 `.` 开头的**隐藏文件**。最后，我们使用 `-ls` 来创建文件列表。这并非偶然，正如你可能已经观察到的那样；你可以创建在 GUI 文件浏览器中首次查看时缺失的文件（特别是在用户的主目录中）或在控制台中（例如，除非你使用带有 `-a` 标志的 `ls` 命令）。在接下来的命令中，我们使用 `find -type d` 来查找名为 `.git` 的目录。接着，我们使用特殊符号 `-type f \( -name "*.sh" -o -name "*.txt" \)` 来查找与 `*.sh` 或 `*.txt` 匹配的文件。注意正斜杠 `\` 和括号 `(`。我们可以使用 `-o -name "string"` 来指定多个名称匹配参数。

1.  在第三步中，我们使用 `find` 来查找子目录，使用 `-type d`，这些子目录通常出现在克隆或导出的 git 相关目录中。我们可以按照如下格式将 `find` 命令链式连接起来：`$ cmd 1 && cmd2 && cmd3 && ...`。这样可以确保如果前面的命令返回为 true，接下来的命令将会执行，依此类推。然后，我们引入了 `-exec` 标志，用于在找到匹配项后执行另一个命令。在这种情况下，我们首先查找所有文件，然后立即使用 `grep` 来在文件中进行搜索。请注意 `grep` 后面的 `{} +`。这是因为 `{}` 会被 `find` 返回的结果替换。`+` 字符表示 `exec` 命令的结束，并将结果追加，使得 `rm -rf` 执行的次数比找到/匹配的文件总数少。

1.  在最后一步，我们使用两种方法来删除文件。第一种方法使用 `-delete` 标志，可能并非所有的 `find` 实现或版本都支持，但一旦匹配，它将删除文件。与对大量文件执行子进程 `rm` 相比，它更高效。其次，使用 `-exec rm -rf {} \;`，我们可以以一种便捷且可移植的方式轻松删除找到的文件。然而，`\;` 和 `+` 之间是有区别的，区别在于使用 `\;` 时，`rm -rf` 会对每个找到/匹配的文件执行一次。使用这个命令时要小心，因为它不是交互式的。

# 创建两个文件的 diff 并进行补丁操作

在什么情况下你需要知道什么是 diff？或者补丁（patch）？在 Linux 世界中，它是一种确定文件差异的方法，也用于解决操作系统层面的问题（特别是如果你在 Linux 内核中遇到坏的驱动程序）。然而，对于一本食谱（cookbook）来说，diff 和补丁主要有以下几个用途：

+   当确定某个脚本或配置文件是否被修改时

+   当绘制版本之间的差异，或者将数据从旧脚本迁移到新脚本时

那么，什么是**差异（diff）**或**差分（differential）**？差异是描述两个文件（文件 A 和文件 B）之间差异的输出。文件 A 是源文件，文件 B 是假定被修改的文件。如果没有生成差异输出，那么文件 A 和 B 要么为空，要么没有差异。统一格式的差异通常看起来像这样：

```
$ diff -urN fileA.txt fileB.txt 
--- fileA.txt 2017-12-11 15:06:49.972849620 -0500
+++ fileB.txt 2017-12-11 15:08:09.201177398 -0500
@@ -1,3 +1,4 @@
 12345
-abcdef
+abcZZZ
+789aaa

```

有多种格式的差异，但统一格式是最流行的格式之一（并且被 FOSS（自由和开源软件）社区广泛使用）。它包含关于两个文件（A 和 B）的信息，行号及每个文件中的行数，以及添加或更改的内容。如果我们查看前面的示例，我们可以看到在原始文件中，字符串`abcdef`被删除`(-)`，然后作为`abcdZZZ`重新添加`(+)`。此外，还添加了一个新行，包含`789aaa`（在这里也可以看到：`@@ -1,3 +1,4 @@`）。

补丁是一个统一差异（unified diff），包含对一个或多个文件的更改，这些更改需要按照特定的顺序或方法应用，因此补丁的概念是应用一个补丁（其中包含差异信息）的过程。一个补丁也可以由几个差异串联在一起组成。

# 准备工作

除了打开终端，还需要安装这两个工具：

```
$ sudo apt-get install patch diff
```

接下来，我们创建一个假的配置文件，它是从一个真实文件复制的：

```
$ cp /etc/updatedb.conf ~/updatedb-v2.conf
```

打开`updatedb-v2.conf`并将内容更改为如下所示：

```
PRUNE_BIND_MOUNTS="yes"
# PRUNENAMES=".git .bzr .hg .svn"
PRUNEPATHS="/tmp /var/spool /media /home/.ecryptfs /var/lib/schroot /media /mount"
PRUNEFS="NFS nfs nfs4 rpc_pipefs afs binfmt_misc proc smbfs autofs iso9660 ncpfs coda devpts ftpfs devfs mfs shfs sysfs cifs lustre tmpfs usbfs udf fuse.glusterfs fuse.sshfs curlftpfs ecryptfs fusesmb devtmpfs"
```

如果你的`updatedb-v2.conf`文件看起来有很大不同，请将`/media /mount`添加到`PRUNEPATHS`变量中。注意它们之间用空格分隔。

# 如何操作...

1.  打开终端，并运行以下命令以了解`diff`命令：

```
$ diff /etc/updatedb.conf ~/updatedb-v2.conf
$ diff -urN /etc/updatedb.conf ~/updatedb-v2.conf
```

1.  此时，只有差异信息被输出到控制台的标准输出中，尚未创建补丁文件。要创建实际的补丁文件，请执行以下命令：

```
$ diff -urN /etc/updatedb.conf ~/updatedb-v2.conf > 001-myfirst-patch-for-updatedb.patch
```

补丁可以有很多种形式，但通常它们具有`.patch`扩展名，并且在前面有一个数字和一个易于理解的名称。

1.  现在，在应用补丁之前，还可以进行测试，以确保结果符合预期。尝试以下命令：

```
$ echo "NEW LINE" > ~/updatedb-v3.conf
$ cat ~/updatedb-v2.conf >> ~/updatedb-v3.conf
$ patch --verbose /etc/updatedb.conf < 001-myfirst-patch-for-updatedb.patch
```

1.  让我们看看使用以下命令时，当补丁应用失败会发生什么：

```
$ patch --verbose --dry-run ~/updatedb-v1.conf < 001-myfirst-patch-for-updatedb.patch 
$ patch --verbose ~/fileA.txt < 001-myfirst-patch-for-updatedb.patch 
```

# 它是如何工作的...

跟我重复——“*locate 简单且需要更新，find 在遇到问题时有效，但强大且难懂，且可能会破坏一些东西*。”接下来，我们继续并开始解释：

1.  第一个`diff`命令以简单的差异格式输出更改。但是，在第二个运行`diff`命令时，我们使用了`-urN`标志。`-u`表示统一格式，`-r`表示递归，`-N`表示新文件：

```
$ diff /etc/updatedb.conf ~/updatedb-v2.conf
3c3
< PRUNEPATHS="/tmp /var/spool /media /home/.ecryptfs /var/lib/schroot"
---
> PRUNEPATHS="/tmp /var/spool /media /home/.ecryptfs /var/lib/schroot /media /mount"
$ diff -urN /etc/updatedb.conf ~/updatedb-v2.conf
--- /etc/updatedb.conf 2014-11-18 02:54:29.000000000 -0500
+++ /home/rbrash/updatedb-v2.conf 2017-12-11 15:26:33.172955754 -0500
@@ -1,4 +1,4 @@
 PRUNE_BIND_MOUNTS="yes"
 # PRUNENAMES=".git .bzr .hg .svn"
-PRUNEPATHS="/tmp /var/spool /media /home/.ecryptfs /var/lib/schroot"
+PRUNEPATHS="/tmp /var/spool /media /home/.ecryptfs /var/lib/schroot /media /mount"
 PRUNEFS="NFS nfs nfs4 rpc_pipefs afs binfmt_misc proc smbfs autofs iso9660 ncpfs coda devpts ftpfs devfs mfs shfs sysfs cifs lustre tmpfs usbfs udf fuse.glusterfs fuse.sshfs curlftpfs ecryptfs fusesmb devtmpfs"

```

1.  现在，我们已经通过将标准输出重定向到`001-myfirst-patch-for-updatedb.patch`文件来创建了一个补丁：

```
$ diff -urN /etc/updatedb.conf ~/updatedb-v2.conf > 001-myfirst-patch-for-updatedb.patch
```

1.  现在我们已经创建了`~/updatedb-v3`的修改版本，注意到干运行时的任何情况吗？忽略`/etc/updatedb.conf`只有只读权限（我们只是为了举例而使用它，因为干运行本身不会改变内容），我们可以看到 HUNK #1 已经成功应用。**hunk**表示差异的一个部分，你可以为一个文件或多个文件拥有多个 hunk。在补丁中的行号没有完全匹配吗？它仍然应用了补丁，因为它知道足够的信息，并**修正**了数据，使其匹配以便成功应用。在处理大文件时要注意这种功能，这些文件可能有相似的匹配标准：

```
$ patch --verbose --dry-run /etc/updatedb.conf < 001-myfirst-patch-for-updatedb.patch 
Hmm... Looks like a unified diff to me...
The text leading up to this was:
--------------------------
|--- /etc/updatedb.conf 2014-11-18 02:54:29.000000000 -0500
|+++ /home/rbrash/updatedb-v2.conf 2017-12-11 15:26:33.172955754 -0500
--------------------------
File /etc/updatedb.conf is read-only; trying to patch anyway
checking file /etc/updatedb.conf
Using Plan A...
Hunk #1 succeeded at 1.
done
```

1.  如果我们尝试将补丁应用到与文件不匹配的文件上，它将失败，像下面的输出所示（如果指定了`--dry-run`）。如果没有指定`--dry-run`，失败将被存储在一个拒绝文件中，正如这一行所述：`1 out of 1 hunk FAILED -- saving rejects to file /home/rbrash/fileA.txt.rej`：

```
$ patch --verbose --dry-run /etc/updatedb.conf1 < 001-myfirst-patch-for-updatedb.patch 
Hmm... Looks like a unified diff to me...
The text leading up to this was:
--------------------------
|--- /etc/updatedb.conf 2014-11-18 02:54:29.000000000 -0500
|+++ /home/rbrash/updatedb-v2.conf 2017-12-11 15:26:33.172955754 -0500
--------------------------
checking file /etc/updatedb.conf1
Using Plan A...
Hunk #1 FAILED at 1.
1 out of 1 hunk FAILED
done
$
$ patch --verbose ~/fileA.txt < 001-myfirst-patch-for-updatedb.patch 
Hmm... Looks like a unified diff to me...
The text leading up to this was:
--------------------------
|--- /etc/updatedb.conf 2014-11-18 02:54:29.000000000 -0500
|+++ /home/rbrash/updatedb-v2.conf 2017-12-11 15:26:33.172955754 -0500
--------------------------
patching file /home/rbrash/fileA.txt
Using Plan A...
Hunk #1 FAILED at 1.
1 out of 1 hunk FAILED -- saving rejects to file /home/rbrash/fileA.txt.rej
done
```

# 创建符号链接并有效使用它们

符号链接意味着快捷方式，对吧？如果你曾听过这个解释，它只是部分正确，它们出现在大多数现代操作系统中。事实上，从文件的角度来看，符号链接有两种类型：硬链接和软链接：

| **硬链接** | **软链接** |
| --- | --- |
| 仅链接文件 | 可以链接目录和文件 |
| 本地磁盘内的内容链接 | 可以跨磁盘或网络引用文件/文件夹 |
| 引用 inode/物理位置 | 如果原始文件被删除，硬链接将保留（在自己的 inode 中） |
| 移动文件仍然允许链接工作 | 如果链接文件被移动，链接将无法跟随原文件 |

软链接最可能符合你对快捷方式的预期，行为可能不会让你感到意外，但硬链接有什么用呢？使用硬链接的一个突出例子是当你不想通过移动它指向的文件来破坏链接时！软链接显然更加灵活，可以跨文件系统工作，而硬链接则不同，但如果文件被移动，软链接将无法工作。

除了创建快捷方式外，你还可以做一些巧妙的操作，比如在使用符号链接时重命名`argv[0]`。Busybox shell 就是一个例子，它包含通过指向`./busybox`的符号链接执行的**applets**。例如，`ls`指向与`cd`相同的二进制文件！它们都指向`./busybox`。这是一种巧妙的方法，可以节省空间并在不使用标志的情况下提高运行时标志的效率。

软链接也用于`/usr/lib`或`/lib`文件夹中的共享`library`。事实上，符号链接对于为路径创建别名或让软件与二进制文件中硬编码的路径一起工作非常有用。

# 如何操作...

1.  打开终端，创建`whoami.sh`脚本：

```
#!/bin/bash
VAR=$0
echo "I was ran as: $VAR"
```

1.  执行`whoami.sh`并观察发生了什么：

```
$ bash whoami.sh
```

1.  接下来，使用`ln`命令创建一个指向`whoami.sh`的软链接：

```
$ ln -s whoami.sh ghosts-there-be.sh
```

1.  接下来，运行`ls` `-la`。注意有任何不同吗？

```
ls -la ghosts-there-be.sh whoami.sh 
```

1.  接下来是硬链接，它是通过使用`ln`命令以这种方式创建的：

```
$ ln ghosts-there-be.sh gentle-ghosts-there-be.sh
$ ln whoami.sh real-ghosts-there-be.sh
```

1.  接下来，让我们看一下运行命令时结果的不同：

```
$ ls -la ghosts-there-be.sh whoami.sh real-ghosts-there-be.sh gentle-ghosts-there-be.sh 
lrwxrwxrwx 1 rbrash rbrash 18 Dec 12 15:07 gentle-ghosts-there-be.sh -> ghosts-there-be.sh
lrwxrwxrwx 1 rbrash rbrash 9 Dec 12 14:57 ghosts-there-be.sh -> whoami.sh
-rw-rw-r-- 2 rbrash rbrash 45 Dec 12 14:56 real-ghosts-there-be.sh
-rw-rw-r-- 2 rbrash rbrash 45 Dec 12 14:56 whoami.sh
$ mv whoami.sh nobody.sh
$ bash ghosts-there-be.sh 
bash: ghosts-there-be.sh: No such file or directory
$ bash real-ghosts-there-be.sh 
I was ran as: real-ghosts-there-be.sh
$ bash gentle-ghosts-there-be.sh 
bash: gentle-ghosts-there-be.sh: No such file or directory
```

# 它是如何工作的...

在第一步中，我们创建了`whoami.sh`。它类似于`whoami`命令，但不同之处在于，我们并没有打印`$USER`变量，而是打印参数`0`（通常被称为`arg0`）或`$0`。通俗地说，我们打印的是用于执行代码或脚本的名称。

当我们执行`whoami.sh`时，它会打印到控制台：

```
$ bash whoami.sh 
I was ran as: whoami.sh
```

要创建符号软链接，我们使用带有`-s`标志（符号模式）的`ln`命令。`ln`命令需要按以下方式执行：`$ ln -s 原始文件路径 新文件路径`。

正如我们在以下代码中看到的，执行`ghosts-there-be.sh`会运行`whoami.sh`中的代码，但`arg0`是`ghosts-there-be.sh`。然后，当运行带有`-l -a`标志（`-la`）的`ls`命令时，我们可以看到指向`whoami.sh`的软链接。注意它只有 9 字节这么小！

```
$ bash ghosts-there-be.sh 
I was ran as: ghosts-there-be.sh
$ ls -la ghosts-there-be.sh whoami.sh 
lrwxrwxrwx 1 rbrash rbrash 9 Dec 12 14:57 ghosts-there-be.sh -> whoami.sh
-rw-rw-r-- 1 rbrash rbrash 45 Dec 12 14:56 whoami.sh
```

接下来，我们通过不带`-s`标志的`ls`命令创建一个硬链接。

硬链接`real-ghosts-there-be.sh`执行与`ghosts-there-be.sh`相同的内容，但指向`whoami.sh`的实际内容，即使它被移动并重命名为`nobody.sh`：

```
$ ls -la ghosts-there-be.sh whoami.sh real-ghosts-there-be.sh gentle-ghosts-there-be.sh 
lrwxrwxrwx 1 rbrash rbrash 18 Dec 12 15:07 gentle-ghosts-there-be.sh -> ghosts-there-be.sh
lrwxrwxrwx 1 rbrash rbrash 9 Dec 12 14:57 ghosts-there-be.sh -> whoami.sh
-rw-rw-r-- 2 rbrash rbrash 45 Dec 12 14:56 real-ghosts-there-be.sh
-rw-rw-r-- 2 rbrash rbrash 45 Dec 12 14:56 whoami.sh
 mv whoami.sh nobody.sh
$ bash ghosts-there-be.sh 
bash: ghosts-there-be.sh: No such file or directory
$ bash real-ghosts-there-be.sh 
I was ran as: real-ghosts-there-be.sh
$ bash gentle-ghosts-there-be.sh 
bash: gentle-ghosts-there-be.sh: No such file or directory 
$ bash gentle-ghosts-there-be.sh 
bash: gentle-ghosts-there-be.sh: No such file or directory
```

# 爬取文件系统目录并打印树状结构

到目前为止，我们已经了解了 locate、find 和 grep（以及正则表达式），但如果我们想要创建一个简单的目录爬虫/抓取器/索引器呢？它肯定不是最快的，也没有优化，但我们可以使用递归功能和文件测试来打印树状结构。

这个练习是一个有趣的练习，当然也算是在重做“轮子”。通过运行 tree 命令可以轻松做到这一点，然而，这在接下来的练习中将会有用，我们将构建文件的数组数组。

# 准备工作

除了打开终端，让我们创建一些测试数据：

```
$ mkdir -p parentdir/child_with_kids
$ mkdir -p parentdir/second_child_with_kids
$ mkdir -p parentdir/child_with_kids/grand_kid/
$ touch parentdir/child.txt parentdir/child_with_kids/child.txt parentdir/child_with_kids/grand_kid/gkid1.txt
$ touch parentdir/second_child_with_kids/cousin1.txt parentdir/z_child.txt parentdir/child.txt parentdir/child2.txt
```

# 如何做...

1.  打开终端并创建`mytree.sh`脚本：

```
#!/bin/bash
CURRENT_LVL=0

function tab_creator() {

  local X=0
  local LVL=$1
  local TABS="."
  while [ $X -lt $LVL ]
  do
    # Concatonate strings
    TABS="${TABS}${TABS}"
    X=$[$X+1]
  done
  echo -en "$TABS"
}
function recursive_tree() {

  local ENTRY=$1
  for LEAF in ${ENTRY}/*
  do
    if [ -d $LEAF ];then
      # If LEAF is a directory & not empty
      TABS=$(tab_creator $CURRENT_LVL)
      printf "%s\_ %s\n" "$TABS" "$LEAF" 
      CURRENT_LVL=$(( CURRENT_LVL + 1 ))
      recursive_tree $LEAF $CURRENT_LVL
      CURRENT_LVL=$(( CURRENT_LVL - 1 ))
    elif [ -f $LEAF ];then
      # Print only the bar and not the backwards slash
      # And only if a file
      TABS=$(tab_creator $CURRENT_LVL)
      printf "%s|_%s\n" "$TABS" "$LEAF" 
      continue
    fi

  done
}

PARENTDIR=$1
recursive_tree $PARENTDIR 1
```

1.  在你的终端中，现在运行：

```
$ bash mytree.sh parentdir 
```

# 它是如何工作的...

创建`mytree.sh`是一个微不足道的任务，但其中的逻辑遵循递归函数。还有`${CURRENT_LVL}`的概念，它用于表示脚本从最初起点**parentdir**开始的深度（即期间的数量或层级）。在每个目录中，我们创建一个 for 循环来测试其中的每个文件/目录。逻辑会测试该条目是文件还是目录。如果是目录，我们递增`${CURRENT_LVL}`，然后**递归**地执行`recursive_tree`函数中的**相同**逻辑，直到完成并返回。如果是文件，我们只是打印出来并**继续**。`tab_creator`函数根据`${CURRENT_LVL}`和拼接生成表示期间的变量字符串。

执行脚本时应产生类似如下的输出，但请注意脚本会记住它可能有多少层深度，并且目录显示时使用的是`\_`而不是`|_`：

```
$ bash mytree.sh parentdir
.|_parentdir/child2.txt
.|_parentdir/child.txt
.\_ parentdir/child_with_kids
..|_parentdir/child_with_kids/child.txt
..\_ parentdir/child_with_kids/grand_kid
....|_parentdir/child_with_kids/grand_kid/gkid1.txt
.\_ parentdir/empty_dir
.\_ parentdir/second_child_with_kids
..|_parentdir/second_child_with_kids/cousin1.txt
.|_parentdir/z_child.txt
```

# 查找和删除重复的文件或目录

曾经我们已经讨论过检查文件中字符串是否唯一以及是否可以对其进行排序，但我们还没有对文件进行类似的操作。然而，在深入之前，让我们对什么构成重复文件做一些假设：重复文件是指可能有不同名称，但内容与其他文件相同的文件。

调查文件内容的一种方法是移除所有空白字符，仅检查文件中包含的字符串，或者我们也可以仅使用**SHA**/**512sum**和**MD**/**5sum**等工具生成文件内容的唯一哈希值（可以理解为充满乱码的唯一字符串）。整体流程如下：

1.  使用这个哈希值，我们可以将其与已经计算的哈希值列表进行比较。

1.  如果哈希值匹配，我们已经见过这个文件的内容，因此可以将其删除。

1.  如果哈希值是新的，我们可以记录该条目并继续计算下一个文件的哈希值，直到所有文件都被*哈希化*。

使用哈希不需要你了解数学是如何运作的，而是要了解它在安全实现的情况下应该如何工作，并且要有足够的可能性使得找到重复项在计算上不可行。哈希应该是单向的，这意味着它不同于加密/解密，因此一旦哈希值被创建，就不应该能够从哈希值本身确定原始输入。

MD5 哈希被认为是完全不安全的（尽管在安全要求较低的场合仍有其用途），而 SHA1/2 被认为可能会随着 SHA3 中的 SPONGE 算法的使用而逐渐失宠（在可能的情况下使用 SHA3）。更多信息，请参见[NIST 指南](https://csrc.nist.gov/Projects/Hash-Functions)。

# 准备工作

打开终端并使用`dsetmkr.sh`脚本创建一个包含多个文件的数据集：

```
$ #!/bin/bash
BDIR="files_galore"
rm -rf ${BDIR}
mkdir -p ${BDIR}

touch $BDIR/file1; echo "1111111111111111111111111111111" > $BDIR/file1;
touch $BDIR/file2; echo "2222222222222222222222222222222" > $BDIR/file2;
touch $BDIR/file3; echo "3333333333333333333333333333333" > $BDIR/file3;
touch $BDIR/file4; echo "4444444444444444444444444444444" > $BDIR/file4;
touch $BDIR/file5; echo "4444444444444444444444444444444" > $BDIR/file5;
touch $BDIR/sameas5; echo "4444444444444444444444444444444" > $BDIR/sameas5;
touch $BDIR/sameas1; echo "1111111111111111111111111111111" > $BDIR/sameas1;
```

然后，在开始编写脚本之前，需要讨论一个核心概念，即数组是静态的还是动态的；如果性能是目标，了解数组实现的核心原理是一个关键原则。

数组非常有用，但 Bash 脚本的性能通常不如编译程序或选择具有适当数据结构的语言。在 Bash 中，数组是链表且是动态的，这意味着如果你调整数组的大小，不会有巨大的性能损失。

对于我们的目的，我们将创建一个动态数组，一旦数组变得相当大，搜索该数组将成为性能瓶颈。这种简单的迭代方法通常在一个任意的数量（假设为`N`）之前效果良好，在这个点上，使用其他机制的好处可能会超过当前方法的简单性。对于那些想了解更多数据结构及其性能的人，可以查阅大 O 符号和复杂度理论。

# 如何做到……

1.  打开终端，并创建`file-deduplicator.sh`脚本。

以下是脚本的代码片段：

```
#!/bin/bash

declare -a FILE_ARRAY=()

function add_file() {
  # echo $2 $1
  local NUM_OR_ELEMENTS=${#FILE_ARRAY[@]}
  FILE_ARRAY[$NUM_OR_ELEMENTS+1]=$1
}

function del_file() {
  rm "$1" 2>/dev/null
}
```

1.  如果还未执行，请运行`setup`命令：运行`$ bash dsetmkr.sh`，然后运行`$ bash ./file-deduplicator.sh`。在提示符处输入`files_galore/`并按*Enter*键：

```
$ bash dsetmkr.sh 
$ bash file-deduplicator.sh
Enter directory name to being searching and deduplicating:
Press [ENTER] when ready

files_galore/
#1 f559f33eee087ea5ac75b2639332e97512f305fc646cf422675927d4147500d4c4aa573bd3585bb866799d08c373c0427ece87b60a5c42dbee9c011640e04d75
#2 f7559990a03f2479bf49c85cb215daf60417cb59875b875a8a517c069716eb9417dfdb907e50c0fd5bd47127105b7df9e68a0c45a907dc5254ce6bc64d7ec82a
#3 2811ce292f38147613a84fdb406ef921929f864a627f78ef0ef16271d4996ed598d0f5c5f410f7ae75f9902ff0f63126b567e5f24882db3686be81f2a79f1bb3
#4 89f5df2b9f4908adca6a36f92b344d4a8ff96d04184e99d8dd31a86e96d45a1aa16a8b574d5815f17d649d521c9472670441a56f54dc1c2640e20567581d9b4e
```

1.  审查结果并验证`files_galore`的内容。

```
$ ls files_galore/
```

# 它是如何工作的……

在开始之前，请注意：`file-deduplicator.sh`脚本会删除它所针对目录中的重复文件。

1.  在开始时（特别是使用`dsetmkr.sh`脚本），我们将生成一个名为`files_galore`的目录，并且该目录包含几个文件：四个是唯一的，三个包含重复的内容：

```
$ bash dsetmkr.sh 
```

密码学、安全性和数学的研究都是非常有趣且广泛的信息领域！哈希值有许多其他用途，例如文件的完整性检查、查找值以快速找到数据、唯一标识符等。

1.  当你运行`file-deduplicator.sh`时，它会首先通过`read`命令向用户请求输入，然后打印出四个不同的值，显示看似随机的字符字符串。*看似随机*是完全正确的——它们是 SHA512 哈希值！每个字符串是其内部内容的哈希值。即使内容仅有微小的差别（例如，一个比特从`0`翻转成`1`），也会生成一个完全不同的哈希值。再次强调，这个 bash 脚本利用了数组这一外部概念（使用全局数组变量意味着在脚本的任何地方都可以访问），并结合**SHA512sum**工具和**awk**来检索正确的值。这个脚本并不是递归的，它只查看`files_galore`中的文件，以生成一个文件列表，每个文件一个哈希值，并搜索一个包含所有*已知*哈希值的数组。如果一个哈希值是未知的，那么它代表一个新文件，并会被插入到数组中进行存储。否则，如果一个哈希值出现了两次，文件将被删除，因为它包含了重复的内容（即使文件名不同）。还有一个方面是使用返回值作为字符串。如你所记得，返回值只能返回数字值：

```

$ bash file-deduplicator.sh 
Enter directory name to being searching and deduplicating:
Press [ENTER] when ready

files_galore/
#1 f559f33eee087ea5ac75b2639332e97512f305fc646cf422675927d4147500d4c4aa573bd3585bb866799d08c373c0427ece87b60a5c42dbee9c011640e04d75
#2 f7559990a03f2479bf49c85cb215daf60417cb59875b875a8a517c069716eb9417dfdb907e50c0fd5bd47127105b7df9e68a0c45a907dc5254ce6bc64d7ec82a
#3 2811ce292f38147613a84fdb406ef921929f864a627f78ef0ef16271d4996ed598d0f5c5f410f7ae75f9902ff0f63126b567e5f24882db3686be81f2a79f1bb3
#4 89f5df2b9f4908adca6a36f92b344d4a8ff96d04184e99d8dd31a86e96d45a1aa16a8b574d5815f17d649d521c9472670441a56f54dc1c2640e20567581d9b4e
```

1.  执行操作后，我们可以看到`files_galore`目录中只剩下四个文件，原本的七个文件中的重复数据已被移除！

```
$ ls files_galore/
file1 file2 file3 file4
```

# 在任意位置连接和拆分文件

让我们不要害羞！谁曾经因意外或故意使用应用程序打开一个大文件，结果没有按计划进行的？我肯定有过，而且我也确实见过一些限制，比如在 Excel 或 OpenOffice 计算器中加载的行数。在这些情况下，我们使用一个方便的工具，它可以在任意位置拆分文件，类似下面这样：

+   在 X 行之前

+   在 Z 字节/字符之前

在这个教程中，你将创建一个具有双重用途的脚本：一个可以使用输入文件并生成*拆分*或多个文件的脚本，另一个是使用合并方法将文件合并的脚本。传递字符串变量时有一些注意事项：

+   有时可能会丢失特殊字符，如换行符

+   （二进制文件）应该由与常规命令不同的工具处理

这个文件还重用了我们在第一章《Bash 快速入门》中看到的`getopts`参数解析，但它还引入了`mktemp`命令和带有`PAGESIZE`参数的`getconf`命令。`Mktemp`是一个有用的命令，因为它可以生成位于`/tmp`目录中的唯一临时文件，甚至可以生成遵循模板的唯一文件（注意`XXX`——它会被替换为随机值，但`uniquefile.`将保持不变）：

```
$ mktemp uniquefile.XXXX
```

另一个有用的命令是`getconf`编程工具，它是一个符合标准的工具，用于获取有用的系统变量。特别是其中一个叫做`PAGESIZE`的变量，它有助于确定内存的块大小。显然，这是非常简单的描述，但选择合适的大小来写入数据在性能上是非常有益的。

# 准备就绪

除了打开一个终端外，还需要创建一个名为`input-lines`的文本文件，文件内容如下（每行一个字符）：

```
1
2
3
4
5
6
7
8
9
0
a
b
c
d
e
f
g
h
i
j
k
```

接下来，创建另一个名为`merge-lines`的文件，内容如下：

```
It's -17 outside
```

# 如何做到...

打开终端并创建一个名为`file-splitter.sh`的脚本。

以下是代码片段：

```
#!/bin/bash
FNAME=""
LEN=10
TYPE="line" 
OPT_ERROR=0
set -f

function determine_type_of_file() {
  local FILE="$1"
  file -b "${FILE}" | grep "ASCII text" > /dev/null
  RES=$?
  if [ $RES -eq 0 ]; then
    echo "ASCII file - continuing"

  else
    echo "Not an ASCII file, perhaps it is Binary?"
  fi 
}
```

接下来，使用以下命令和标志（`-i`，`-t`，`-l`）运行`file-splitter.sh`：

```
$ bash file-splitter.sh -i input-lines -t line -l 10
```

查看输出并观察`-t size`与`-l line`使用时的差异。当使用`-l 1`或`-l 100`时会有什么不同？记得使用`$ rm input-lines.*`删除拆分后的文件：

```
$ rm input-lines.*
$ bash file-splitter.sh -i input-lines -t line -l 10
$ rm input-lines.*
$ bash file-splitter.sh -i input-lines -t line -l 1
$ rm input-lines.*
$ bash file-splitter.sh -i input-lines -t line -l 100
$ rm input-lines.*
$ bash file-splitter.sh -i input-lines -t size -l 10
```

在下一步中，创建另一个名为`file-joiner.sh`的脚本。

以下是代码片段：

```
#!/bin/bash
INAME=""
ONAME=""
FNAME=""
WHERE=""
OPT_ERROR=0

TMPFILE1=$(mktemp)

function determine_type_of_file() {
  local FILE="$1"
  file -b "${FILE}" | grep "ASCII text" > /dev/null
  RES=$?
  if [ $RES -eq 0 ]; then
    echo "ASCII file - continuing"

  else
    echo "Not an ASCII file, perhaps it is Binary?"
  fi 
}
```

接下来，使用以下命令运行脚本：

```
$ bash file-joiner.sh -i input-lines -o merge-lines -f final-join.txt -w 2
```

# 它是如何工作的...

在继续之前，请注意，`final-join.txt`上的类型选项（`-t`）在一次读取字符时会忽略`\n`换行符。`read`足以满足本教程的需求，但读者应意识到，`read/cat`并不是这种工作类型的最佳工具。

1.  创建这个脚本很简单，大部分情况下它看起来不会像是来自火星的作品。

1.  运行命令`$ bash file-splitter.sh -i input-lines -t line -l 10`应该会生成三个文件，文件名为 input-lines {1,...,3}。之所以有三个文件，是因为如果使用的是相同的输入，即 22 行数据，它会生成三个文件（10+10+2）。使用`read`和`echo`配合连接缓冲区（`${BUFFER}`），我们可以根据特定的标准（由`-l`提供）将数据写入文件。如果遇到**EOF**（文件结束）并且循环完成，我们需要将缓冲区写入文件，因为它可能低于写入标准的阈值——这会导致`splitter`脚本生成的最后一个文件中的字节丢失或缺失。

```
$ bash file-splitter.sh -i input-lines -t line -l 10
ASCII file - continuing
Wrote buffer to file: input-lines.1
Wrote buffer to file: input-lines.2
Wrote buffer to file: input-lines.3
```

1.  根据`-l`标志的使用情况，值为`1`时会为每一行生成一个文件，值为 100 时则会生成一个单独的文件，因为它符合阈值要求。使用辅助功能`-t size`，可以根据字节进行拆分，但`read`命令有一个不幸的副作用：当我们传递缓冲区时，它会被修改，导致新行丢失。如果我们使用类似`dd`的工具，这类操作会更好，因为`dd`更适合复制、写入以及创建原始数据到文件或设备中。

1.  接下来，我们创建了一个名为`file-joiners.sh`的脚本。它同样使用了`getopts`，并需要四个输入参数：`-i originalFile -o`，`otherFileToMerge -f`，`finalMergedFile -w`，以及`whereInjectTheOtherFile`。该脚本相对简单，但使用了`mktemp`命令来创建一个临时文件，作为存储缓冲区，而不会修改原文件。当完成操作后，我们可以使用`mv`命令将文件从`/tmp`移动到终端当前目录（`.`）。`mv`命令也可以用于重命名文件，并且通常比`cp`命令更快（虽然在这个例子中差别不大），因为它不执行复制操作，而只是进行文件系统级的重命名。

1.  使用`cat`查看`final-join.txt`应该显示如下输出：

```
$ cat final-join.txt 
1
2
It's -17 outside
3
4
5
6
7
8
9
0
a
b
c
d
e
f
g
h
i
j
k

```

# 生成各种大小的数据集和随机文件

通常，模仿真实数据的数据总是最好的，但有时我们需要各种内容和大小的文件集合来进行验证测试，而不希望有任何延迟。想象一下，你有一个 Web 服务器，并且它正在运行某个应用程序，该应用程序接收文件进行存储。然而，文件的大小有限制。要是能够瞬间*搞定*一批文件，不是很棒吗？

为此，我们可以使用一些文件系统功能，比如`/dev/random`和一个有用的程序`dd`。`dd`命令是一个用于转换和复制文件的工具（包括设备，因为在 Linux 中一切皆文件）。它可以在后续的教程中用于备份 SD 卡上的数据（记得你最爱的 Raspberry Pi 项目吗？），或者逐字节读取文件而不丢失数据。`dd`的典型最小用法为`$ dd if="inputFile" of="outputFile" bs=1M count=10`。从这个命令中，我们可以看到：

+   `if=`：表示输入文件

+   `of=`：表示输出文件

+   `bs=`：表示块大小

+   `count=`：表示要复制的块数

如果要执行文件的纯复制（1:1），则选项`bs=`和`count=`是可选的，因为`dd`命令会尝试使用合理有效的参数来提供足够的性能。`dd`命令还具有许多其他选项，如`seek=`，将在另一个配方中介绍，用于执行低级备份时通常不需要 count 选项，因为通常复制整个文件而不是部分文件（在执行备份时）。

`/dev/random`是 Linux 中的一个设备（因此使用`/dev`路径），可用于生成用于脚本或应用程序中的随机数。还有其他`/dev`路径，如控制台和各种适配器（例如，USB 存储设备或鼠标），所有这些都可能是可访问的，建议您了解它们。

# 做好准备

为此配方做好准备，请按以下步骤安装`dd`命令，并创建一个名为`qa-data/`的新目录：

```
$ sudo apt-get install dd bsdmainutils
$ mkdir qa-data
```

本教程使用`dmesg`命令，该命令用于返回系统信息，如接口状态或系统引导过程。它几乎在所有系统上都存在，因此是合理的系统级“lorem ipsum”的良好替代。如果您希望使用其他类型的随机文本或词典，则可以轻松替换`dmesg`！另外使用的两个命令是`seq`和`hexdump`。**`seq`**命令可以使用指定的增量从起始点生成一个*n*个数字的数组，而`hexdump`会以十六进制格式生成二进制（或可执行）的人类可读表示。

# 如何执行它……

打开终端并创建一个名为`data-maker.sh`的新脚本。

以下是脚本的代码片段：

```
#!/bin/bash

N_FILES=3
TYPE=binary
DIRECTORY="qa-data"
NAME="garbage"
EXT=".bin"
UNIT="M"
RANDOM=$$
TMP_FILE="/tmp/tmp.datamaker.sh"

function get_random_number() { 
  SEED=$(($(date +%s%N)/100000))
  RANDOM=$SEED
  # Sleep is needed to make sure that the next time rnadom is ran, everything is good.
  sleep 3
  local STEP=$1
  local VARIANCE=$2
  local UPPER=$3
  local LOWER=$VARIANCE
  local ARR;

  INC=0
  for N in $( seq ${LOWER} ${STEP} ${UPPER} ); 
  do
    ARR[$INC]=$N
    INC=$(($INC+1))
  done

  RAND=$[$RANDOM % ${#ARR[@]}]
  echo $RAND
}
```

让我们使用以下命令开始执行脚本。它使用`-t`标志指定类型为`text`，使用`-n`来指定文件数量为`5`，`-l`设置下限为 1 个字符，`-u`设置为`1000`个字符：

```
$ bash data-maker.sh -t text -n 5 -l 1 -u 1000
```

要检查输出，请使用以下命令：

```
$ ls -la qa-data/*.txt
$ tail qa-data/garbage4.txt
```

再次运行`data-maker.sh`脚本，但是这次生成的是二进制文件。大小限制不再是 1 个字符（1 字节）或`1000`个字符（`1000`字节或略少于 1 千字节），而是以 MB 为单位，生成`1`到`10` MB 的文件：

```
$ bash data-maker.sh -t binary -n 5 -l 1 -u 10
```

要查看输出，请使用以下命令。由于我们无法像处理“常规”ASCII 文本文件那样“转储”或“cat”二进制文件，因此使用了一个名为`hexdump`的新命令：

```
$ ls -la qa-data/*.bin
$ hexdump qa-data/garbage0.bin 
0000000 0000 0000 0000 0000 0000 0000 0000 0000
*
```

# 它的工作原理……

让我们了解一下，事情是如何发生的：

1.  首先，我们创建了` ;data-maker.sh` 脚本。这个脚本引入了几个新概念，包括一直令人着迷的随机化概念。在计算机中，或者说在生活中的任何事情里，真正的随机事件或数字生成是无法实现的，它们需要几个数学原理，如熵。虽然这超出了本书的范围，但需要知道的是，无论是随机重复使用还是初次使用时，都应该给它一个唯一的初始化向量或种子。通过使用`for`循环，我们可以利用`seq`命令构建一个数字数组。一旦数组构建完成，我们从中选择一个“随机”的值。在每种类型的文件输出操作（无论是二进制文件还是文本文件）中，我们大致确定最小值（`-l`或下限）和最大值（`-u`或上限），以控制输出数据的大小。

1.  在第 2 步中，我们使用`dmesg`的输出和我们的伪随机化过程创建了`5`个文本文件。我们可以看到，我们通过`dd`命令迭代，直到创建了五个不同大小和起始点的文本文件。

1.  在第 3 步中，我们验证了确实创建了五个文件，并且在第五个文件中，我们查看了`garbage4.txt`文件的`tail`部分。

1.  在第 4 步中，我们使用`dd`命令创建了五个二进制文件（全是零）。我们没有使用字符数，而是使用了兆字节（MB）作为单位。

1.  在第 5 步中，我们验证了确实创建了五个二进制文件，并且在第五个文件中，我们使用`hexdump`命令查看了二进制文件的内容。`hexdump`命令创建了一个简化的“转储”，展示了`garbage0.bin`文件中的所有字节。
