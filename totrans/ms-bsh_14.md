# 安全性时刻

安全性无论我们身处何地都很重要。例如，在建筑工地上，像在新构建的操作系统中一样，安全性是确保事情以正确方式完成的关键因素。我们的 shell 在安全性方面也不例外：我们大多数时间都待在环境中，努力完成任务，保持一切井然有序。本章将为我们提供一些快速的解决方案和提示，帮助我们加强安全性并避免最常见的问题。我们不会使用像安全性或其他内核级别增强这样的更高级工具：这些工具本身就需要一本书的篇幅，并且它们是在清理我们的 shell 后才会使用的。我们将进行*家务管理*，没有什么真正侵入性的操作，只是做一些*修饰性工作*，力求在安全性、可靠性和可用性之间找到一个平衡；这其实是一个很难实现的目标：加强过多的话，即使是最简单的任务也几乎无法完成。如果我们过于注重安全性，可能会导致我们的系统过于暴露或不安全。因此，我们将尝试找到一个最佳平衡点，确保系统既可用，又相对安全；但最终还是由每个系统的管理员决定这个平衡点应该是什么：我们只能提供一些建议和展示可能的操作。

# 受限 shell

有多种方法可以限制用户在系统上的操作，并且有很多原因导致我们会限制用户与系统的交互：可能我们只希望用户能够将文件复制进出系统，或者为他们提供一个简单的主目录，让他们可以在里面工作而不去窥探系统中的其他内容。无论我们的目标是什么，我们都可以从使用受限 shell 开始。

Bash 本身提供了额外的安全层，使用以下选项：

+   `rbash`

+   `--restricted`

+   `-r`

使用 `--restricted` 或 `-r` 选项调用 `rbash` 或直接调用 `bash`，将启动一个受限的 Bash 实例，从而限制用户在该环境中可以执行的操作：

+   用户不能使用 `cd` 内建命令更改目录。用户将被阻止设置或取消设置以下环境变量的值：

    +   `BASH_ENV`

    +   `ENV`

    +   `SHELL`

    +   `PATH`

+   用户无法指定带有斜杠的命令名称，这意味着无法使用绝对路径的命令名称。不能将包含斜杠的文件名作为内建命令 `.` 的参数传递。因此，用户将无法从主目录外部加载（读取并执行）文件。

+   不能传递包含斜杠的文件名作为内建命令 `hash` 的参数，使用 `-p` 选项时尤为如此。`hash` 会通过在由环境变量 `$PATH` 指定的目录中查找，确定作为参数给定的命令的完整文件名。如果给定了 `-p filename` 选项，`hash` 会将文件名视为查找命令时的完整路径。因此，不能调用位于主目录以外的命令。

+   在 Shell 环境启动时，函数定义不会被导入。

+   启动时，环境变量 SHELLOPTS 的值不会被考虑，因此不会为 Shell 设置任何选项。

+   使用标准操作符 `>`, `>|`, `<>`, `>&`, `&>`, `>>` 不允许进行重定向。

+   无法使用内建的 exec 命令来替换 Shell 为其他命令。

+   无法使用 `enable` 内建命令通过 `-d` 或 `-f` 选项添加或删除内建命令。

+   无法使用 `enable` 内建命令来启用或禁用 Bash 内建命令。

+   对于内建命令，不允许使用 `-p` 选项，因此无法操作 `$PATH`。

+   无法使用 `set +r` 或 `set +o restricted` 关闭受限模式。

所以，尽管有这些限制，用户还是被限制在他的主目录中。但如何设置一个 `rbash 登录` Shell 呢？最简单的方法是找到 Bash 链接并将其重定向到 `rbash`：

```
root:# which bash /bin/bash root:# which rbash /bin/rbash root:# ls -lah /bin/rbash lrwxrwxrwx 1 root root 4 Nov 5 2016 /bin/rbash -> bash

```

在这种情况下，`rbash` 和 `bash` 之间已经存在链接，但在这种情况下它们没有任何链接，所以我们必须创建一个：

```
root:# cd /bin root:# ln -s bash rbash

```

然后，我们必须检查 `rbash` 是否列在 `/etc/shells` 中，该文件列出了有效登录 Shell 的完整路径：

```
root:# cat /etc/shells # /etc/shells: valid login shells /bin/sh /bin/dash /bin/bash /bin/rbash

```

现在，让我们创建一个具有限制 Shell 的用户：

```
root:# adduser --shell /bin/rbash restricted Adding user `restricted' ... Adding new group `restricted' (1000) ... Adding new user `restricted' (1000) with group `restricted' ... Creating home directory `/home/restricted' ... Copying files from `/etc/skel' ... Enter new UNIX password: Retype new UNIX password: passwd: password updated successfully Changing the user information for restricted Enter the new value, or press ENTER for the default Full Name []: Room Number []: Work Phone []: Home Phone []: Other []: Is the information correct? [Y/n] y

```

完成后，让我们 `su` 到该用户并测试 `cd` 命令：

```
root:# cd /home/restricted root:# su restricted restricted:~$ cd restricted:~$ cd: restricted

```

到这里了。`cd` 命令被如我们预期的那样限制。让我们检查一下其他的限制：

```
restricted:~$ pwd /home/restricted restricted:~$ test "Redirection test"> redirected_file rbash: redirected_file: restricted: cannot redirect output

```

很好，没有重定向，不过这个“笼子”并没有完全隔离：

```
restricted:~$ ls -lah /sbin/c* -rwxr-xr-x 1 root root 19K Mar 30 2015 /sbin/capsh -rwxr-xr-x 1 root root 243K Mar 29 2015 /sbin/cfdisk -rwxr-xr-x 1 root root 23K Mar 29 2015 /sbin/chcpu -rwxr-xr-x 1 root root 9.4K Aug 23 2014 /sbin/crda -rwxr-xr-x 1 root root 1.2K Jan 22 2015 /sbin/cryptdisks_start -rwxr-xr-x 1 root root 1.2K Jan 22 2015 /sbin/cryptdisks_stop -rwxr-xr-x 1 root root 58K Jan 22 2015 /sbin/cryptsetup -rwxr-xr-x 1 root root 45K Jan 22 2015 /sbin/cryptsetup-reencrypt -rwxr-xr-x 1 root root 11K Mar 29 2015 /sbin/ctrlaltdel

```

这个受限用户仍然可以在其目录外做一些事情。让我们通过使用本地配置文件来覆盖这个限制：

```
root:# mkdir /home/restricted/bin root:# ln -s /bin/df /home/restricted/bin/df

```

现在，让我们删除在主目录中找到的 `.bash_profile` 或 `.profile` 文件，如果它们不存在，就创建一个 `.bashrc` 文件，其中的唯一一行应该是：

```
PATH=$HOME/bin

```

现在，让我们阻止用户修改它：

```
root:# chown root. /home/restricted/.bashrc root:# chmod 755 /home/restricted/.bashrc

```

现在让我们 `su`：

```
root:# su restricted

```

让我们检查一下我们能做些什么：

```
restricted:~$ cd rbash: cd: restricted

```

我们不被允许这么做，这一点我们已经知道了。让我们试着列出一些文件：

```
restricted:~$ ls rbash: ls: command not found

```

很好，除了我们的 `$HOME/bin` 目录外没有其他命令可用。让我们再试一次：

```
restricted:~$ ping rbash: ping: command not found

```

正如预期的那样，出现了另一个失败。现在让我们尝试一下我们在用户的 `$HOME/bin` 目录中链接的 `df` 命令：

```
restricted:~$ df Filesystem 1K-blocks Used Available Use% Mounted on /dev/sda1 117913932 12494776 99406436 12% / udev 10240 0 10240 0% /dev tmpfs 779256 9192 770064 2% /run tmpfs 1948140 0 1948140 0% /dev/shm tmpfs 5120 4 5116 1% /run/lock tmpfs 1948140 0 1948140 0% /sys/fs/cgroup tmpfs 389628 0 389628 0% /run/user/0

```

这个方法有效，我们成功限制了用户可以访问的命令，并将其限制在他的主目录内。很棒，看起来已经被隔离了，但还是有一些限制：

受限用户可以通过运行具有 Shell 函数的程序逃离这个*笼子*。一个经典的例子是 `vi` 编辑器：

```
restricted:~$ vi :set shell=/bin/bash :shell restricted:~$ pwd /home/restricted restricted:~$ cd / restricted:~$ ls -lah total 11G drwxr-xr-x 22 root root 4.0K Apr 17 06:32 . drwxr-xr-x 22 root root 4.0K Apr 17 06:32 .. drwxr-xr-x 2 root root 4.0K May 8 05:10 bin drwxr-xr-x 3 root root 4.0K Apr 16 07:51 boot drwxr-xr-x 19 root root 3.2K May 9 04:05 dev drwxr-xr-x 131 root root 12K May 8 05:34 etc drwxr-xr-x 4 root root 4.0K May 8 05:34 home lrwxrwxrwx 1 root root 31 Apr 16 06:14 initrd.img -> /boot/initrd.img-3.16.0-4-amd64 drwxr-xr-x 21 root root 4.0K Apr 17 03:59 lib drwxr-xr-x 2 root root 4.0K Apr 16 06:13 lib64 drwx------ 2 root root 16K Apr 16 06:13 lost+found drwxr-xr-x 3 root root 4.0K Apr 16 06:13 media drwxr-xr-x 2 root root 4.0K Apr 16 06:13 mnt drwxr-xr-x 2 root root 4.0K Apr 16 06:13 opt -rw-r--r-- 1 root root 10G Apr 16 07:55 playground dr-xr-xr-x 113 root root 0 May 9 04:05 proc drwx------ 9 root root 4.0K May 9 04:07 root drwxr-xr-x 21 root root 840 May 9 04:10 run drwxr-xr-x 2 root root 4.0K Apr 17 03:59 sbin drwxr-xr-x 2 root root 4.0K Apr 16 06:13 srv dr-xr-xr-x 13 root root 0 May 9 04:05 sys drwxrwxrwt 8 root root 4.0K May 9 04:11 tmp drwxr-xr-x 10 root root 4.0K Apr 16 06:13 usr drwxr-xr-x 12 root root 4.0K Apr 16 06:32 var lrwxrwxrwx 1 root root 27 Apr 16 06:14 vmlinuz -> boot/
vmlinuz-3.16.0-4-amd64 restricted:~$

```

逃离受限 Shell 的另一种方法是启动一个没有限制的 Shell：

```
restricted:~$ cd rbash: cd: restricted restricted:~$ bash restricted:~$ cd / restricted:~$ pwd / restricted:~$

```

这也意味着任何具有有效 sha-bang 的脚本都会调用完整的 shell，从而逃脱任何限制。这些方法意味着用户可以访问 Bash 或任何具有 shell 功能的程序，否则逃出限制区就不容易。但我们需要牢记一点：这是将某些用户限制在其工作空间中的方法，它将把他们彼此隔离，给予他们独立的家目录，并防止他们无意中破坏系统的其他部分。它并不是一个完备的安全层；对于这类问题，我们应该依赖更为底层的解决方案，如内核级别的安全措施，这超出了本书的讨论范围，因为这需要大量的关于安全、内核编译、第三方产品、加固等方面的解释。再说一次，这将是一本独立的书。

因此，我们希望保持整洁，如何有序地承载远程连接呢？

# OpenSSH 的限制性 shell

尽管 OpenSSH 的限制性 shell（[`www.pizzashack.org/rssh/`](http://www.pizzashack.org/rssh/)）严格来说并不是一个 shell 工具，但它的简洁性使其成为帮助保持系统整洁的好工具，尤其是在某个访客敲门时。Rssh 支持多种发行版和平台，提供了一个限制性 shell，不仅支持`scp`和`ftp`，还支持`csv`、`rdist`和`rsync`。因此，我们可以创建仅允许文件复制或同步的账户，而不提供完全的 shell 访问；这对于保持低调并降低服务器遭受攻击的风险非常有用。

第一阶段是从包或源代码安装 rssh。在我们的示例中，我们将依赖于一个包，因为所使用的发行版 Debian 中已有此包；此外，使用包管理可以确保在需要时，维护者会对该工具进行升级和修补：

```
root:# apt-get install rssh Reading package lists... Done Building dependency tree Reading state information... Done Suggested packages: cvs rdist subversion makejail The following NEW packages will be installed: rssh 0 upgraded, 1 newly installed, 0 to remove and 0 not upgraded. Need to get 54.4 kB of archives. After this operation, 119 kB of additional disk space will be used. Get:1 http://ftp.us.debian.org/debian/ jessie/main rssh amd64 2.3.4-4+b1 [54.4 kB] Fetched 54.4 kB in 0s (103 kB/s) Preconfiguring packages ... Selecting previously unselected package rssh. (Reading database ... 62697 files and directories currently installed.) Preparing to unpack .../rssh_2.3.4-4+b1_amd64.deb ... Unpacking rssh (2.3.4-4+b1) ... Processing triggers for man-db (2.7.0.2-5) ... Setting up rssh (2.3.4-4+b1) ...

```

安装完成后，我们将获得一个新的 shell 二进制文件：

```
root:# ls -lah /usr/bin/rssh -rwxr-xr-x 1 root root 31K Nov 8 2014 /usr/bin/rssh

```

现在，让我们使用这个新的二进制文件作为限制性用户的 shell：

```
root:# chsh -s `which rssh` restricted

```

接下来，让我们直接在`/etc/passwd`中验证是否已分配 shell：

```
root:# egrep restricted /etc/passwd restricted:x:1000:1000:,,,:/home/restricted:/usr/bin/rssh

```

一切看起来正常，接下来让我们从远程连接到限制性用户所在的系统：

```
zarrelli:~$ ssh -p 9999 restricted@192.168.0.5 Warning: Permanently added '[192.168.0.5]:9999' (ECDSA) to the list of known hosts. Restricted@192.168.0.5's password: The programs included with the Debian GNU/Linux system are free software; the exact distribution terms for each program are described in the individual files in /usr/share/doc/*/copyright. Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent permitted by applicable law. This account is restricted by rssh. This user is locked out. If you believe this is in error, please contact your system administrator. Connection to 192.168.0.5 closed.

```

该账户已被锁定，这是`rssh`的默认行为，因为我们尚未进行配置。因此，接下来我们来看一下在`/etc/rssh.conf`文件中可以使用的一些主要配置指令，以启用某些协议和每个用户的配置：

+   `allowsftp`：允许 sftp 连接。

+   `allowcvs`：允许 cvs 连接。

+   `allowrdist`：允许 rdist 连接。

+   `allowrsync`：允许 rsync 连接。

+   `allowsvnserve`：允许 svnserve 连接。

+   `umask`：设置在 scp 或 sftp 会话期间创建的文件和目录的 umask。umask 通常由 shell 在用户登录时设置，因此为了避免这种情况，rssh 必须自己设置 umask。

+   `logfacility`：指定用于日志记录的 syslog 设施或 C 宏。

+   `chrootpath`：`rssh` 的一个辅助应用程序（`rssh_chroot_helper`）调用 `chroot()` 系统调用，改变会话的文件系统根目录。例如：`chrootpath=/opt/jails` 将会把虚拟文件系统的根目录更改为 `/opt/jails`，适用于其 shell 为 `rssh` 的用户。

登录后，`/var/caged/users` 目录将显示为用户的文件系统根目录，且无法超出该目录。如果使用了此指令，则必须设置 `chroot` 监狱，为用户提供最小化的环境。稍后我们将看到如何操作。如果在 `/etc/passwd` 中定义的用户主目录位于 `chrootpath` 指定的路径中，那么用户将被 `chdired` 到其主目录，否则将被 `chdired` 到 `chroot` 监狱的根目录。

+   `user`: 使用这个指令，我们可以为每个用户设置配置，覆盖所有其他指令。`user` 关键字出现在用冒号（`:`）分隔的字符串中，具有以下结构：

```
user = "username:unask:access_digits:chroot_path"

```

那么，接下来让我们看看每个字段代表什么：

+   **username**：这是我们希望为其设置配置的帐户名。

+   **umask**：表示用户的 umask，以八进制形式表示。它遵循与为 Bash shell 设置 umask 时相同的规范。

+   **access_digits**：这六个二进制数字指定用户是否被允许使用 `rsync`、`rdist`、`cvs`、`sftp`、`scp` 和 `svnserve`，按顺序列出。`0` 表示不允许用户使用，`1` 表示允许用户使用。

+   **path**：指定用户将被限制到的目录路径。

引号不是强制性的，除非路径字段中有空格。在这种情况下，我们可以使用单引号或双引号。`=` 两边的空格是可以的。所以，像 `user=restricted:022:100000:` 这样的表达式意味着用户 restricted 的 umask 是 `022`，并且该用户可以使用 `rsync` 连接。没有指定 `chroot`：

```
user = restricted:011:000110:"/usr/local/chroot jails"

```

之前的语句意味着用户 restricted 拥有 `011` 的 umask，`sftp` 和 `scp` 连接可用，并且它将被限制在 `/usr/local/chroot jails` 中。

了解更多配置后，让我们通过在 `/etc/rssh.conf` 中添加 `user = restricted:277:000010` 来仅启用受限用户的 `scp` 连接。

现在是检查我们是否终于可以访问远程系统的时候了：

```
zarrelli:~$ ssh -p 9999 restricted@192.168.0.5 Warning: Permanently added '[192.168.0.5]:9999' (ECDSA) to the list of known hosts. restricted@192.168.0.5's password: The programs included with the Debian GNU/Linux system are free software; the exact distribution terms for each program are described in the individual files in /usr/share/doc/*/copyright. Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent permitted by applicable law. Last login: Tue May 9 06:15:40 2017 from 192.168.0.10 This account is restricted by rssh. Allowed commands: scp If you believe this is in error, please contact your system administrator. Connection to 192.168.0.5 closed.

```

这很有趣。消息与上次尝试略有不同：我们仍然无法通过 ssh 登录到远程服务器，但它说明尽管帐户受到 rssh 的限制，我们仍然可以使用 `scp`。所以，让我们试试看：

```
zarrelli:~$ scp -P 9999 test_file restricted@192.168.0.5: Warning: Permanently added '[192.168.0.5]:9999' (ECDSA) to the list of known hosts. Restricted@192.168.0.5's password: test_file 

```

看起来它好像成功了。让我们看一下远程服务器：

```
root:# pwd /home/restricted root:# ls -lah test_file -r-------- 1 restricted rssh_users 0 May 10 05:24 test_file

```

来了。在受限用户的主目录中，我们的文件访问权限已设置为 `400`，如预期的那样。简单明了。但这里有一个小问题，我们可以在这里看到问题所在：

![](img/00047.jpeg)

使用 FileZilla，我们能够浏览远程主机的整个文件系统。

即使用户被限制在一个协议中，他仍然可以浏览远程文件系统，除了 Unix/POSIX 文件权限之外没有任何限制。在示例中，我们启用了 SFTP 协议，并实际以受限用户身份连接到系统，浏览 `/etc` 目录。我们能防止这种情况吗？可以，我们可以将用户 `chroot` 到一个理想的文件系统中，最好是挂载为 `nosuid`，如果支持的话，还可以选择 `noxec` 选项。这样，即使用户上传了可执行文件，他也无法运行它或利用任何 `suid` 权限。这个操作容易做吗？不，创建 `chroot` 监狱可能非常困难，因为这需要将相关的二进制文件和库文件复制到监狱中；版本和路径可能会根据所使用的发行版以及版本本身发生变化。实际上，rssh 的源 tarball 提供了一个脚本，通过一些修改，实际上可以帮助将所有必要的文件复制到监狱中。我们还可以在互联网上找到一些其他的脚本，这些脚本将帮助我们完成这项敏感的工作。无论如何，提供受限 SFTP 访问到服务器有一个更简单的方法，我们无需远离我们的环境，因为我们可以仅仅使用 OpenSSH 服务器来完成这项任务。

# 使用 OpenSSH 限制 SFTP 会话

使用 OpenSSH，一切可以通过五行配置和一些命令轻松完成；让我们看看如何操作。我们在远程服务器上。

首先，让我们打开 OpenSSH 配置文件，通常位于 `/etc/ssh/sshd_config`，并添加以下几行：

```
Match group sftp-only ChrootDirectory /opt/jails/%u/exchange X11Forwarding no AllowTcpForwarding no ForceCommand internal-sftp

```

我们应该已经知道这些指令是什么，但让我们回顾一下我们在第十二章中关于 SSH 远程连接的内容，*远程 SSH 连接*：

+   `Match`：使用这个指令，我们可以使用条件语句，这样如果满足条件，以下的配置行将覆盖主配置块中的内容。如果一个关键字/配置块在多个 `Match` 子句中出现，只有第一个实例会被考虑。作为匹配条件，我们可以使用以下指令：用户、组、主机、本地地址、本地端口、地址，或者所有条件。我们可以匹配一个列表、模式或否定。在我们的示例中，我们将匹配一个我们稍后会创建的组：属于 `sftp-only` 组的所有账户将受以下配置行的限制。

+   `ChrootDirectory`：通过指定目录的完整路径，我们可以在成功认证后将用户 `chroot` 到该目录中。这不是一项简单的任务，因为该目录必须由 `root` 拥有，并且不能对其他人可写。此外，我们还必须提供会话所需的一些文件，例如 shell、`/dev/null`、`/dev/zero`、`/dev/arandom`、`/dev/stdin`、`/dev/stdout`、`/dev/stderr` 和 `/dev/ttyx`。我们还可以使用一些令牌，例如 `%h` 代表认证账户的主目录，`%%` 代表简单的 `%`，`%u` 则会被替换为用户名。在我们的案例中，我们不需要提供任何二进制文件，因为我们只允许 `sftp` 连接，由于没有 shell，无法执行任何操作。

+   `X11Forwarding`：此选项允许/拒绝 `X11` 转发。如果设置为 `yes`，可能会使 `X11` 暴露于攻击之中；因此，这个选项需要谨慎使用。默认值为 `no`。我们禁止转发 `X11`：既然不需要此功能，它可能会暴露系统。

+   `AllowTcpForwarding`：此选项允许/拒绝 TCP 转发，参数可以为 `yes`、`all`、`no` 或 `local` 和 `remote`。前两个选项允许转发，第三个选项拒绝转发，而 `local` 仅允许本地转发；`remote` 仅允许远程转发。对于我们的示例来说，不涉及任何 shell 或 TCP 转发。

+   `ForceCommand`：覆盖客户端发送的任何命令或在认证账户的 `~/.ssh/rc` 文件中列出的命令，并强制执行此指令中列出的命令。该命令通过账户的 shell 执行，并附带 `-c` 选项。默认值为 `no`。在我们的案例中，我们强制执行 OpenSSH 内部的 `sftp` 子系统。

说到子系统，让我们验证在同一个配置文件中，OpenSSH 是否配置为使用 `internal-sftp` 子系统：

```
Subsystem sftp internal-sftp 

```

我们也许还想在配置文件末尾添加一些额外的限制：

```
PermitTunnel no AllowAgentForwarding no

```

+   `AllowAgentForwarding`：定义是否允许 ssh-agent 转发。默认值为 `yes`，以增加安全性，由于 `sftp` 账户实际上不需要此功能，因此我们将禁用它。

+   `PermitTunnel`：此选项允许/拒绝隧道设备转发。参数可以为 `yes`、点对点、以太网或 `no`。`yes` 启用点对点和以太网转发，默认值为 `no`，我们希望确保它被禁用，因为我们不需要为 `sftp` 账户启用隧道。

现在我们已经配置好了所有服务部分，接下来重启 OpenSSH 服务器，在我们的例子中就是这个：

```
root:# service ssh restart

```

该是时候添加我们的新组并将受限用户移到其中了：

```
root:# addgroup sftp-only root:# service ssh restart root:# usermod -g sftp-only restricted root:# usermod -s /bin/false restricted

```

所以，现在我们已经将受限用户添加到 `sftp-only` 组中，并且没有有效的 shell 用于登录系统：

```
restricted:x:1000:1003:,,,:/home/restricted:/bin/false

```

现在，让我们将用户的主目录设置为由 `root` 拥有，这样用户就无法向其写入：

```
root:# chown root. /home/restricted/

```

然后创建一个由 `root` 拥有的新主目录：

```
root:# mkdir -p /opt/sftp-jails/restricted/exchange root:# chown -R root. /opt/sftp-jails/

```

还可以让一个受限用户拥有一个子目录，允许其写入：

```
root:# chown restricted.root /opt/sftp-jails/restricted/exchange root:# chmod 750 /opt/sftp-jails/restricted/exchange/

```

一切正常，现在让我们尝试登录：

```
zarrelli:~$ ssh -p 9999 restricted@192.168.0.5 Warning: Permanently added '[192.168.0.5]:9999' (ECDSA) to the list of known hosts. restricted@192.168.0.5's password: Could not chdir to home directory /home/restricted: No such file or directory This service allows sftp connections only. Connection to 192.168.0.5 closed.

```

这没问题：我们不希望受限用户有完整的 shell，所以让我们尝试 sftp：

```
zarrelli:~$ sftp -P 9999 restricted@192.168.0.5 Warning: Permanently added '[192.168.0.5]:9999' (ECDSA) to the list of known hosts. Restricted@192.168.0.5's password: Connected to 192.168.0.5. sftp> 

```

太好了，我们进来了，但是让我们检查一下我们实际能做什么：

```
sftp> pwd Remote working directory: / sftp> 

```

好的，我们在我们的远程根目录中，但里面有什么？

```
sftp> ls -lah drwxr-xr-x 0 0 0 4.0K May 11 14:56 . drwxr-xr-x 0 0 0 4.0K May 11 14:56 .. drwxr-x--- 0 1000 0 4.0K May 11 15:00 exchange

```

听起来很熟悉。这是一个围栏，所以让我们逃出去：

```
sftp> cd / sftp> pwd Remote working directory: / sftp> ls exchange sftp> 

```

不行，实际上我们不能这样做。至少让我们试着上传一些东西：

```
sftp> put test_file Uploading test_file to /test_file remote open("/test_file"): Permission denied sftp> 

```

不行，用户 root 的目录是不可写的，所以让我们 `cd` 到交换目录再试一次上传：

```
sftp> cd exchange sftp> put test_file Uploading test_file to /exchange/test_file test_file 100% 0 0.0KB/s 00:00 sftp> 

```

它绝对有效。让我们把文件拿回来：

```
sftp> get test_file Fetching /exchange/test_file to test_file sftp> bye

```

到这里了，账户已经准备好供客户连接和共享数据。但如果我们想要使用密钥进行认证连接呢？

让我们首先修改 `/etc/ssh/ssd_config`，在文件的最末尾添加以下行：

```
AuthorizedKeysFile /opt/sftp-jails/authorized_keys/%u/authorized_keys

```

这将在匹配条件下，并且将被所有 `sftp-only` 组的用户触发；但为了使其生效，我们必须重新加载配置：

```
root:# service ssh restart

```

因此，由于新指令指示 OpenSSH 在 `/opt/sftp-jails/authorized_keys/{username}/authorized_keys` 中为所有属于 `sftp-only` 组的用户查找 `authorized_keys`，让我们开始创建正确的目录：

```
root:# mkdir -p /opt/sftp-jails/authorized_keys/restricted

```

这是包含用户认证密钥的用户目录的完整路径，该目录是受限用户的用户名。我们将需要为每个用户创建一个目录，并且最终目录的名称必须与用户名相同。现在，我们必须修剪所有权和访问权限：

```
root:# cd /opt/sftp-jails root:# chown -R root.sftp-only authorized_keys root:# chown -R restricted.root authorized_keys/restricted

```

`authorized_keys` 目录属于用户 root 和组：`sftp-only`，而子目录 restricted 属于用户 restricted 和组 root：

```
root:# chmod 750 /opt/sftp-jails/authorized_keys/ root:# chmod 500 /opt/sftp-jails/authorized_keys/restricted/

```

所有属于 `sftp-only` 组的用户都可以遍历 `authorized_keys` 目录，但只有受限用户可以遍历 restricted 目录。现在，让我们将我们的示例密钥复制到最终目标：

```
root:# cp id_ecdsa_to_spoton.pub /opt/sftp-jails/authorized_keys/restricted/authorized_keys

```

现在让我们给它正确的所有权和访问权限：

```
root:# chown restricted.root /opt/sftp-jails/authorized_keys/restricted/authorized_keys root:#chmod 400 /opt/sftp-jails/authorized_keys/restricted/authorized_keys

```

所以，我们应该以这样的配置结束：

```
root:# cd /opt/ root:# tree -pug . └── [drwxr-xr-x root root ] sftp-jails
 ├── [drwxr-x--- root sftp-only] authorized_keys │   └── [dr-x------ restricted root ] restricted │       └── [-r-------- restricted root ] authorized_keys └── [drwxr-xr-x root sftp-only] restricted └── [drwxr-x--- restricted root ] exchange └── [-rw-r--r-- restricted sftp-only] test_file

```

一切看起来都很好，所以我们只需测试一下我们到目前为止做了什么。让我们转到本地服务器，尝试连接远程服务器：

```
local_user:~$ sftp -i .ssh/id_ecdsa_to_spoton -P 9999 restricted@192.168.0.5 Warning: Permanently added '[192.168.0.5]:9999' (ECDSA) to the list of known hosts. Connected to 192.168.0.5. sftp> 

.ssh/config file of local_user:
```

```
Host spoton-sftp AddressFamily inet ConnectionAttempts 10 ForwardAgent no ForwardX11 no ForwardX11Trusted no GatewayPorts no HostBasedAuthentication no HostKeyAlias sftp-alias HostName 192.168.0.5 IdentityFile ~/.ssh/id_ecdsa_to_spoton PasswordAuthentication no Port 9999 Protocol 2 Compression yes CompressionLevel 9 ServerAliveCountMax 3 ServerAliveInterval 15 TCPKeepAlive no User restricted

```

让我们尝试一个 `ssh` 连接：

```
local_user:~$ ssh spoton-sftp Warning: Permanently added 'sftp-alias,[192.168.0.5]:9999' (ECDSA) to the list of known hosts. Could not chdir to home directory /home/restricted: No such file or directory This service allows sftp connections only. Connection to 192.168.0.5 closed.

```

这是正确的，我们不应该被允许通过 `ssh` 连接带有 shell 的用户或访问主目录。

让我们尝试一个 `sftp` 连接：

```
local_user:~$ sftp spoton-sftp Warning: Permanently added 'sftp-alias,[192.168.0.5]:9999' (ECDSA) to the list of known hosts. Connected to spoton-sftp.

```

很好，我们使用我们的身份密钥连接了，并且没有指定用户、文件或 IP 地址。所以，让我们进行一些测试：

```
sftp> pwd Remote working directory: /

```

我们在我们的用户根目录中；让我们尝试爬到系统根目录：

```
sftp> cd / sftp> pwd Remote working directory: / sftp> ls exchange 

```

不行，我们被困在我们的根目录中，无法进入任何上层目录。让我们寻找要上传的文件：

```
sftp> !ls test_local

```

让我们尝试将其上传到主目录：

```
sftp> put test_local Uploading test_local to /test_local remote open("/test_local"): Permission denied

```

我们没有权限，不出所料。我们需要使用交换子目录来实现我们的目的：

```
sftp> cd exchange sftp> put test_local Uploading test_local to /exchange/test_local test_local 100% 0 0.0KB/s 00:00 

```

上传成功！现在，让我们看看交换目录里面有什么：

```
sftp> ls test_file test_local Ok, we have the old and the new file. Let's grab the old file: sftp> get test_file Fetching /exchange/test_file to test_file sftp> 

```

完成！一切看起来都很好。或者说，差不多，因为我们对连接期间发生的事情视而不见。由于一切都在一个隔离的环境中，因此没有办法使用像`rsyslog`这样的系统设施来实际记录用户在`sftp`会话期间的操作。或者，至少，正常的`rsyslog`配置无法做到这一点，但有一些方法可以绕过这个限制。我们将要看到的一种方法涉及使用管道；它会让事情变得非常简单。首先，让我们修改`/etc/ssh/sshd_config`中的一些指令。

旧的`Subsystem`和`ForceCommand`现在必须重新编写为：

```
Subsystem sftp internal-sftp -l INFO ForceCommand internal-sftp -l INFO

```

现在，`internal-sftp`将以`INFO`级别记录日志，因此我们需要通过套接字将这些信息导出到主日志中。让我们创建一个文件：

```
root:# cat /etc/rsyslog.d/openssh-sftp.conf module(load="imuxsock") input(type="imuxsock" Socket="/opt/sftp-jails/restricted/dev/log" CreatePath="on") if $programname == 'internal-sftp' then /var/log/openssh-sftp.log & stop

```

所以，我们所做的是指示`rsyslog`在受限用户的`/dev`目录中创建一个 Unix 套接字；`sftp`子系统将能够通过这个套接字将日志消息发送给`rsyslog`。是的，但是这些消息是如何写入的呢？它们是通过简单地访问消息本身的某个属性来写入的。在这种情况下，如果生成消息的程序名称是`internal-sftp`，那么消息将被写入`/var/log/openssh-sftp.log`。完成后，让我们重启`sshd`和`rsyslog`：

```
root:# service ssh restart root:# service rsyslog restart

```

如果你收到来自`rsyslog`的消息，抱怨`imuxsock`模块已被加载，只需将第一行注释掉`#`。

现在，我们只需再建立一次连接并执行一些命令，以便填充日志文件：

```
root:# cat /var/log/openssh-sftp.log May 11 14:28:25 spoton internal-sftp[16080]: session opened for local user restricted from [192.168.0.10] May 11 14:28:26 spoton internal-sftp[16080]: opendir "/" May 11 14:28:26 spoton internal-sftp[16080]: closedir "/" May 11 14:28:35 spoton internal-sftp[16080]: opendir "/exchange" May 11 14:28:35 spoton internal-sftp[16080]: closedir "/exchange" May 11 14:28:39 spoton internal-sftp[16080]: open "/exchange/test_file" flags READ mode 0666 May 11 14:28:39 spoton internal-sftp[16080]: close "/exchange/test_file" bytes read 0 written 0 May 11 14:28:42 spoton internal-sftp[16080]: open "/exchange/test_file" flags WRITE,CREATE,TRUNCATE mode 0644 May 11 14:28:42 spoton internal-sftp[16080]: sent status Permission denied May 11 14:29:00 spoton internal-sftp[16080]: opendir "/exchange" May 11 14:29:00 spoton internal-sftp[16080]: closedir "/exchange" May 11 14:29:05 spoton internal-sftp[16080]: remove name "/exchange/test_file" May 11 14:29:06 spoton internal-sftp[16080]: opendir "/exchange" May 11 14:29:06 spoton internal-sftp[16080]: closedir "/exchange" May 11 14:29:09 spoton internal-sftp[16080]: open "/exchange/test_file" flags WRITE,CREATE,TRUNCATE mode 0644 May 11 14:29:09 spoton internal-sftp[16080]: close "/exchange/test_file" bytes read 0 written 0 May 11 14:29:10 spoton internal-sftp[16080]: opendir "/exchange" May 11 14:29:10 spoton internal-sftp[16080]: closedir "/exchange" May 11 14:30:45 spoton internal-sftp[16080]: session closed for local user restricted from [192.168.0.10]

```

就这样。现在我们有了一个很好的日志，显示了用户在其`sftp`会话期间所做的事情；而且该日志本身对任何`sftp`用户都是不可访问的。在我们的示例中，我们基于生成消息的程序名称来重定向消息；但是我们还有其他标签可以用来过滤。接下来，让我们看看其他更有用的标签：

+   `HOSTNAME`：消息中显示的主机名。

+   `FROMHOST`：消息接收到的系统主机名。在链式配置中，这是接收方旁边的系统，而不一定是第一个发送方。

+   `syslogfacility`：消息报告的设施，以数字形式显示。

+   `syslogfacility-text`：消息报告的设施，以文本形式显示。

+   `syslogseverity`：消息以数字形式报告的严重性。

+   `syslogseverity-text`：消息报告的严重性，以文本形式显示。

通过使用这些属性，我们可以做一些有趣的事情。让我们开始创建另一个与受限用户具有相同组和 shell 的用户：

```
root:# adduser --shell /bin/false --gid 1003 casualuser Adding user `casualuser' ... Adding new user `casualuser' (1003) with group `sftp-only' ... Creating home directory `/home/casualuser' ... Copying files from `/etc/skel' ... Enter new UNIX password: Retype new UNIX password: passwd: password updated successfully Changing the user information for casualuser Enter the new value, or press ENTER for the default Full Name []: Room Number []: Work Phone []: Home Phone []: Other []: Is the information correct? [Y/n] 

```

现在，让我们更改用户主目录的所有者：

```
root:# chown -R root. /home/casualuser/

```

让我们创建一个新监狱，复制我们已有的：

```
root:# cd /opt/sftp-jails
 root:# cp -ra restricted/ casualuser

```

我们现在只需要修复所有权：

```
root:# cd casualuser root:# chown -R casualuser.root exchange/ root:# cd exchange root:# chown casualuser.sftp-only *

```

现在是复制密钥的时候了：

```
root:# cd /opt/sftp-jails/authorized_keys root:# cp -ra restricted casualuser root:# chown -R casualuser.root casualuser/

```

在我们的示例中，为了简便起见，我们使用的是与受限用户相同的密钥，但我们始终可以创建一个新密钥并将`authorized_keys`文件复制过去，以便为每个用户分配他们自己的密钥。完成后，让我们尝试连接：

```
local_user:~$ sftp -i .ssh/id_ecdsa_to_spoton -P 9999 casualuser@192.168.0.5 Warning: Permanently added '[192.168.0.5]:9999' (ECDSA) to the list of known hosts. Connected to 192.168.0.5. sftp> ls dev exchange sftp> !ls test_file test_local sftp> put test_file Uploading test_file to /test_file remote open("/test_file"): Permission denied sftp> cd exchange sftp> put test_file Uploading test_file to /exchange/test_file test_file sftp>

```

好的，用户可以访问并且拥有正确的权限，但日志呢？什么也没有，我们没有为日志设置任何内容，因此让我们通过添加以下行来修改`/etc/rsyslog.d/openssh-sftp.conf`：

```
input(type="imuxsock" Socket="/opt/sftp-jails/casualuser/dev/log" CreatePath="on")

```

现在，为了使新的指令生效，让我们重启`rsyslog`：

```
service rsyslog restart

```

然后让我们再次连接，生成一些日志行：

```
local_user:~$ sftp -i .ssh/id_ecdsa_to_spoton -P 9999 casualuser@192.168.0.5 Warning: Permanently added '[192.168.0.5]:9999' (ECDSA) to the list of known hosts. Connected to 192.168.0.5. sftp> ls dev exchange sftp> cd exchange sftp> ls test_file test_local sftp> get test_local Fetching /exchange/test_local to test_local sftp> bye

```

让我们检查日志文件：

```
May 12 04:02:04 spoton internal-sftp[18573]: session opened for local user casualuser from [192.168.0.10] May 12 04:02:06 spoton internal-sftp[18573]: opendir "/" May 12 04:02:06 spoton internal-sftp[18573]: closedir "/" May 12 04:02:11 spoton internal-sftp[18573]: opendir "/exchange" May 12 04:02:11 spoton internal-sftp[18573]: closedir "/exchange" May 12 04:02:15 spoton internal-sftp[18573]: open "/exchange/test_local" flags READ mode 0666 May 12 04:02:15 spoton internal-sftp[18573]: close "/exchange/test_local" bytes read 0 written 0 May 12 04:02:18 spoton internal-sftp[18573]: session closed for local user casualuser from [192.168.35.219] May 12 04:06:45 spoton internal-sftp[18631]: session opened for local user casualuser from [192.168.0.10]

```

这是预期的结果。我们在新用户的监狱中创建了一个 Unix 套接字；并且我们正在接收由`internal-sftp`子系统为账户会话发送的消息。很好，但有些混淆。所有用户的日志消息都会包含在一个文件中，而像`May 12 04:02:11 spoton internal-sftp[18573]: closedir "/exchange"`这样的命令消息并没有被用户账户名标识，而是通过会话 ID[18631]来区分，因此可以追踪到会话期间执行的所有操作，并追溯到执行操作的用户。但总体来说，这并不容易阅读。我们能做什么呢？嗯，像往常一样，我们需要使用一些想象力和创造力，弯曲规则以获得一些优势。让我们动手修改`openssh-sftp`的`rsyslog`配置文件：

```
/etc/rsyslog.d/openssh-sftp.conf

```

让我们打开它并将内容替换为以下几行：

```
#module(load="imuxsock") input(type="imuxsock" HOSTNAME="restricted" Socket="/opt/sftp-jails/restricted/dev/log" CreatePath="on") input(type="imuxsock" HostName="casualuser" Socket="/opt/sftp-jails/casualuser/dev/log" CreatePath="on") if $hostname == 'restricted' then /var/log/openssh-sftp/restricted-sftp.log if $hostname == 'casualuser' then /var/log/openssh-sftp/casualuser-sftp.log & stop

```

我们做了什么？我们使用了一个属性操作字符串，使我们能够将主机名属性与来自特定套接字的消息关联。然后，我们添加了两条规则，将消息重定向到每个用户的日志文件，基于消息本身中找到的主机名属性。我们故意使用了不同大小写的主机名属性，目的是显示该属性名是大小写不敏感的。现在是时候重启`rsyslogd`了：

```
root:# service rsyslog restart

```

日志设施已经准备好了，让我们再连接一次，制造一些*噪音*：

```
local_user:~$ sftp -i .ssh/id_ecdsa_to_spoton -P 9999 casualuser@192.168.0.5 Warning: Permanently added '[192.168.0.5]:9999' (ECDSA) to the list of known hosts. Connected to 192.168.0.5. sftp> ls dev exchange sftp> cd exchange sftp> ls test_file test_local sftp> get test_local Fetching /exchange/test_local to test_local sftp> cd .. sftp> put test_file Uploading test_file to /test_file remote open("/test_file"): Permission denied sftp> cd / sftp> 

```

现在是检查的时候了：

```
root:# ls -lah /var/log/openssh-sftp/casualuser-sftp.log -rw-r----- 1 root adm 757 May 12 05:14 /var/log/openssh-sftp/casualuser-sftp.log

```

我们以普通用户身份连接，的确我们看到一个名为`casualuser-sftp.log`的文件，正如我们预期的那样出现在那里。让我们看一下里面的内容：

```
root:# cat /var/log/openssh-sftp/casualuser-sftp.log May 12 05:13:48 casualuser internal-sftp[19004]: session opened for local user casualuser from [192.168.0.10] May 12 05:13:49 casualuser internal-sftp[19004]: opendir "/" May 12 05:13:49 casualuser internal-sftp[19004]: closedir "/" May 12 05:13:54 casualuser internal-sftp[19004]: opendir "/exchange" May 12 05:13:54 casualuser internal-sftp[19004]: closedir "/exchange" May 12 05:14:13 casualuser internal-sftp[19004]: open "/exchange/test_local" flags READ mode 0666 May 12 05:14:13 casualuser internal-sftp[19004]: close "/exchange/test_local" bytes read 0 written 0 May 12 05:14:23 casualuser internal-sftp[19004]: open "/test_file" flags WRITE,CREATE,TRUNCATE mode 0644 May 12 05:14:23 casualuser internal-sftp[19004]: sent status Permission denied May 12 05:41:50 casualuser internal-sftp[19004]: session closed for local user casualuser from [192.168.0.10]

```

就这样。我们的` sftp`会话已经完全记录，现在如果我们想知道普通用户做了什么，我们只需要打开日志文件并阅读它。一个有趣的点是，每行消息现在都包含了属于该用户的名字。嗯，实际上它会是主机名，但我们通过扩展规则从系统中获取了我们想要的信息。真的是这样吗？让我们通过连接一个受限用户做最后检查，看是否生成了新的日志文件：

```
local_user:~$ sftp -i .ssh/id_ecdsa_to_spoton -P 9999 restricted@192.168.0.5 Warning: Permanently added '[192.168.0.5]:9999' (ECDSA) to the list of known hosts. Connected to 192.168.0.5. sftp> mkdir test Couldn't create directory: Permission denied sftp> cd exchange sftp> mkdir test sftp> ls test test_file test_local sftp> dc test Invalid command. sftp> cd test sftp> bye

```

现在，我们已经作为受限用户执行了一些命令，让我们看看我们需要的文件是否真的在预期的位置：

```
root:# ls -lah /var/log/openssh-sftp/restricted-sftp.log -rw-r----- 1 root adm 607 May 12 05:47 /var/log/openssh-sftp/restricted-sftp.log

```

文件正确地存在，所以让我们看看里面的内容：

```
root:# cat /var/log/openssh-sftp/restricted-sftp.log May 12 05:43:47 restricted internal-sftp[19147]: session opened for local user restricted from [192.168.0.10] May 12 05:47:04 restricted internal-sftp[19147]: mkdir name "/test" mode 0777 May 12 05:47:04 restricted internal-sftp[19147]: sent status Permission denied May 12 05:47:11 restricted internal-sftp[19147]: mkdir name "/exchange/test" mode 0777 May 12 05:47:13 restricted internal-sftp[19147]: opendir "/exchange" May 12 05:47:13 restricted internal-sftp[19147]: closedir "/exchange" May 12 05:47:22 restricted internal-sftp[19147]: session closed for local user restricted from [192.168.0.10]

```

内容已经在那里；所有受限用户执行的操作都已被记录，并且超出了他们的访问权限。最后再做一次检查：

```
root:# grep restricted casualuser-sftp.log | wc -l 0

```

这证实了没有任何标记为受限的行出现在`casualuser-sftp.log`中，因此每个用户都有自己的日志文件。

所以，我们现在拥有一个完全功能的`sftp`服务器，以及单独的监狱环境和每个用户的日志记录。现在只剩下一个小细节要给我们的服务器，这将带我们回到过去：我们正在谈论一个显示给用户的横幅。它看起来可能不像是那么有用，或者属于过去的时代，但事实并非如此。当有人连接到我们的服务器时，必须通知他这是一个私人设施，不允许进行任何非法操作。至少有两个原因说明这样做是有用的：

+   如果一个未经授权的用户错误地连接到我们的服务器，他必须知道自己并不在他所认为的地方；因此，我们给他一个断开连接的机会，并且不会采取进一步的行动。

+   如果一个不明身份的用户故意连接到我们的服务器，必须通知他不被允许这样做。如果他继续操作，那么我们以后可以将其作为他试图对我们的设施进行非法操作的证据。

它看起来可能很简单，但有了横幅，没人能再说“我不知道”。不，用户已被通知，这一点很重要。由于我们不想吓到访客，让我们用`figlet`制作一个带有爵士风格的横幅，`figlet`是一个可以为我们的消息应用漂亮字体的工具，准备在终端上显示。在我们的例子中，我们使用的是 Debian，因此输入`root:# apt-get install figlet`就能安装这个工具。默认的字体集并不丰富，但可以从该项目的官方网站下载更多：[`www.figlet.org/fontdb.cgi`](http://www.figlet.org/fontdb.cgi)。

让我们首先测试一下系统上已经安装的字体。在 Debian 环境中，字体文件位于`/usr/share/figlet/`，但这可能会根据所使用的发行版有所不同。因此，为了测试所有字体并查看我们最喜欢的字体，我们需要一个一行的 for 循环：

```
root:# for i in $(ls -1 /usr/share/figlet/*.flf | awk -F "." '{print $1}') ; do figlet -f $i test ; echo $i ; done

```

![](img/00048.jpeg)

一个简单的`for`循环将展示我们所有可用的字体中的消息。所以，让我们创建一个包含欢迎横幅的测试文件：

```
Welcome to our restricted sftp server

```

我们将其保存为一个名为`header.txt`的文件，并传递给`figlet stdin`：

```
root:# figlet -cf slant < header.txt > /opt/sftp/jails/sftp-banner

```

我们将得到类似下图所示的效果，字体会很漂亮地居中显示：

![](img/00049.jpeg)

一个简单的`for`循环将展示我们所有可用的字体中的消息。现在，让我们为`footer.txt`文件添加一些有意义的警告：

```
WARNING This service is restricted to authorized users only. All activities on this system are logged.

```

让我们通过`figlet`传递这条消息：

```
root:# figlet -cf digital < footer.txt >> /opt/sftp-jails/sftp-banner

```

既然我们已经有了横幅，让我们稍微整理一下它：

```
root:# rm footer.txt header.txt root:# chown root.root /opt/sftp-jails/sftp-banner root:# chmod 500 /opt/sftp-jails/sftp-banner

```

就这些。所以我们尝试重新连接服务器，并且我们会收到像这里显示的那样的欢迎消息：

![](img/00050.gif)

一条漂亮的欢迎消息将提醒访问者此 sftp 站点的限制。

很好，但假设我们看到了一种喜欢的字体，但它没有安装。为了这个示例，我们假设我们想使用鳄鱼字体：

```
root:# figlet -cf alligator test figlet: alligator: Unable to open font file

```

它没有安装，因此我们无法使用它，但这只是下载并复制字体目录的问题：

```
root:# wget -P /usr/share/figlet/ http://www.figlet.org/fonts/alligator.flf --2017-05-12 08:18:46-- http://www.figlet.org/fonts/alligator.flf Resolving www.figlet.org (www.figlet.org)... 188.226.162.120 Connecting to www.figlet.org (www.figlet.org)|188.226.162.120|:80... connected. HTTP request sent, awaiting response... 200 OK Length: 11285 (11K) [text/plain] Saving to: ‘/usr/share/figlet/alligator.flf' /usr/share/figlet/alligator.flf 100%[=======================================================================================================>] 11.02K --.-KB/s in 0s 2017-05-12 08:18:46 (44.3 MB/s) - ‘/usr/share/figlet/alligator.flf' saved [11285/11285]

```

现在我们只需再次输入之前的命令：

```
 root:# figlet -cf alligator test

```

字体可供 figlet 使用，因此结果就是我们在以下截图中看到的内容：

![](img/00051.jpeg)

安装字体其实只是将其复制到字体目录中

所以，让我们来点乐趣吧。尝试改变并创建你喜欢的样式：一个横幅不一定非得枯燥乏味。

# 总结

这是本书的最后总结，上一章的主题是 figlet；这并非偶然。我们在所有章节中试图阐明的是，Bash 是有趣的。我们是否涵盖了所有可能的话题和示例？不，根本没有，而这正是最伟大的地方：我们有太多东西可以探索，太多方法可以让 Bash 完成难以想象的任务。只要想想某件事，然后尝试使用 shell：在大多数情况下，稍微动点脑筋就能找到创造性的方式克服障碍，并且为完成的结果会心一笑。本书名为*《精通 Bash》*，但没有一本书能涵盖我们可以发现的关于这个 shell 的所有内容。所以，这不是终点，这只是一个更进一步的步骤，也许比平时更高，但依然是我们最喜爱的环境、我们钟爱的 GNU/Linux 操作系统中不断探索旅程的一小步。
