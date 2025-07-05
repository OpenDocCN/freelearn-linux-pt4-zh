# 让脚本像守护进程一样运行

在本章中，我们将介绍以下主题：

+   使用循环结构或递归使程序持续运行（永远）

+   在注销后保持程序/脚本运行

+   在需要权限时调用命令

+   清理用户输入并确保结果可重复

+   使用 select 创建一个简单的多级用户菜单

+   生成和捕获信号以进行清理

+   在程序中使用临时文件和锁文件

+   利用超时等待命令完成

+   创建文件输入-文件输出程序并并行运行进程

+   在启动时执行脚本

# 介绍

本章内容是创建模拟应用程序功能的组件，例如菜单或守护进程。为了实现这一点，我们先停下来思考一下：什么定义了一个应用程序或守护进程？是菜单吗？是能够*永远*运行吗？还是能够在后台*无头*运行？所有这些都定义了一个应用程序可能表现出的行为，但没有什么能阻止脚本也具备这些行为！

例如，如果一个 bash 脚本没有扩展名（例如，`.sh`），并且没有显式通过 Bash 解释器运行，你怎么知道第一次检查时它是一个脚本而不是一个二进制文件？虽然有很多方法可以确定，比如打开文件或使用`file`命令，但表面上看，脚本和程序可能是一样的！

# 使用循环结构或递归使程序持续运行（永远）

到目前为止，本食谱大多展示了只执行单一任务并在任务完成后退出的脚本。这非常适合一次性脚本，但如果我们希望通过菜单执行多个脚本，或者在后台自动执行任务而无需每次通过调度进程（如 cron）执行，应该怎么做呢？本食谱介绍了几种方法，使脚本能够永远运行，直到被杀死或退出。

# 准备工作

除了保持终端开启，我们还需要记住几个概念：

+   递归函数与提示符结合（例如，`read`命令）可以产生一个根据用户输入循环的脚本。

+   循环结构如`for`、`while`和`until`可以以一种方式执行，使得条件永远无法满足，从而无法退出

因此，循环或某些会引发循环的东西会强制程序运行一个不确定的时间，直到发生退出事件。

在许多编程语言中，程序将通过主函数的概念来执行。在这个`main`函数中，程序员通常会创建所谓的运行循环，这使得程序能够永远运行（即使它什么也不做）。

使用递归方法时，操作可能如下所示：

1.  脚本或程序进入递归函数

1.  递归函数可以继续无限次调用自身，或者等待一个阻塞输入（例如，`read`命令）

1.  基于 `read` 命令提供的输入，您可以再次调用相同的函数。

1.  回到步骤 1，直到退出

另外，循环机制相似，不同的是函数不一定会执行。一个有效不可达的条件下的循环将持续运行，除非某些东西中断了执行（例如，`sleep`）。

一个持续运行的循环将消耗 CPU 资源，这些资源本可以用于其他地方，或浪费 CPU 周期。如果你在电池供电或资源受限的平台上运行，最好避免不必要的额外 CPU 活动。

使用 `sleep` 命令是在简单脚本中使用循环时限制 CPU 使用率的绝佳方式。然而，如果你运行一个长时间的脚本，时间会累积！

# 如何操作...

让我们按以下步骤开始我们的活动：

1.  打开终端并创建 `recursive_read_input.sh` 脚本：

```
#!/bin/bash

function recursive_func() {

    echo -n "Press anything to continue loop "
    read input
    recursive_func
}

recursive_func
exit 0
```

1.  执行 `$ bash recursive_read_input.sh` 脚本——在提示符下按 *Enter* 并等待下一个提示符。

1.  使用 *Ctrl* + C 退出程序。

1.  打开终端并创建 `loop_for_input.sh` 脚本：

```
#!/bin/bash
for (( ; ; ))
do
   echo "Shall run for ever"
   sleep 1
done
exit 0
```

1.  执行 `$ bash loop_for_input.sh` 脚本——在提示符下按 *Enter* 并等待下一个提示符。

1.  使用 *Ctrl* + *C* 退出程序。

1.  打开终端并创建 `loop_while_input.sh` 脚本：

```
#!/bin/bash
EXIT_PLEASE=0
while : # Notice no conditions?
do
   echo "Pres CTRL+C to stop..."
   sleep 1
   if [ $EXIT_PLEASE != 0 ]; then
      break 
    fi 
done
exit 0
```

1.  执行 `$ bash loop_while_input.sh` 脚本——在提示符下按 *Enter* 并等待下一个提示符。

1.  使用 *Ctrl* + *C* 退出程序。

1.  打开终端并创建 `loop_until_input.sh` 脚本：

```
#!/bin/bash
EXIT_PLEASE=0
until [ $EXIT_PLEASE != 0 ] # EXIT_PLEASE is set to 0, until will never be satisfied
do
   echo "Pres CTRL+C to stop..."
   sleep 1
done
exit 0
```

1.  执行 `$ bash loop_until_input.sh` 脚本——在提示符下按 *Enter* 并等待下一个提示符。

1.  使用 *Ctrl* + *C* 退出程序。

# 它是如何工作的...

让我们详细了解一下我们的脚本：

1.  创建 `recursive_read_input.sh` 脚本是一个简单的过程。我们可以看到，`read` 命令期待输入（并将其存储在 `$input` 变量中），然后脚本会在每次 `read` 执行完后再次调用 `recursive_func()`，并且对 *每次* 输入都会如此。

1.  执行 `$ bash recursive_read_input.sh` 脚本将使脚本无限运行。无论输入什么，*Ctrl + C* 或终止脚本都会退出。

1.  创建 `loop_for_input.sh` 脚本相对来说也比较简单。我们可以注意到两点：for 循环没有参数，除了 `(( ; ; ))` 和 `sleep` 命令。这将使它永远运行，但在每次执行循环时，它会向控制台输出 `Shall run for ever`，然后休眠一秒钟再继续下一次循环。

1.  执行 `$ bash loop_for_input.sh` 脚本将导致脚本永远循环。

1.  *Ctrl* + *C* 将导致脚本退出。

1.  使用 `loop_while_input.sh` 脚本时，通过使用带有 `: noop` 命令的 `while` 循环。然而，存在一个小差异，即一个 `if` 语句（该语句永远不会为真），但它仍然可以在其他脚本中使用，以设置条件，使得脚本中断 `while` 循环并退出。

1.  执行 `$ bash loop_while_input.sh` 脚本将导致脚本永远循环。

1.  *Ctrl* + *C* 将导致脚本退出。

1.  `loop_until_input.sh`脚本类似于`while`循环永远运行的示例，但它有所不同，因为你也可以嵌入一个条件，该条件永远不会评估为`true`。这将导致脚本永远循环，除非`$EXIT_PLEASE`变量被设置为`1`。

1.  执行`$ bash loop_until_input.sh`脚本将使脚本永远循环。

1.  *Ctrl* + *C*将使脚本退出。

# 在退出登录后保持程序/脚本运行

在让我们的脚本作为守护进程运行之前，我们需要了解如何在用户退出登录后保持命令运行（或者更好的是，让系统自己启动它们——我们稍后会更详细地讨论）。当用户登录时，会为该用户创建一个会话，但当他们退出登录时——除非系统拥有它——进程和脚本通常会被杀死或关闭。

这个方法是关于在你退出登录后，如何让你的脚本和活动在后台继续运行。

# 准备工作

除了打开一个终端，我们还需要记住一些概念：

+   当用户退出登录时，当前用户拥有的所有应用程序或进程将退出（shell 会发送信号）。

+   Shell 可以配置为不向进程发送关机信号。

+   应用程序和脚本使用 stdin 和 stdout 进行常规操作。

+   后台运行的应用程序或脚本可以被称为**作业**。

本章的目的是不仅展示进程管理，还要教你如何操作 shell 来保持程序运行。一个巧妙的方法是使用`&`，它的使用方式是：`$ bash runforver.sh &`。不幸的是，仅使用这种技术，我们回到了原点——当我们退出时，二进制文件仍然会停止运行。因此，我们需要使用诸如**screen**、**disown**和**sighup**之类的程序。

并非所有系统都支持 screen 命令。建议我们使用其他命令，以防**screen**缺失（但它仍然很有用！）。

# 如何操作...

我们按以下步骤开始活动：

1.  打开一个终端并创建`loop_and_print.sh`脚本：

```
#!/bin/bash
EXIT_PLEASE=0
INC=0

until [ ${EXIT_PLEASE} != 0 ] # EXIT_PLEASE is set to 0, until will never be satisfied
do
   echo "Boo $INC" > /dev/null
   INC=$((INC + 1))
   sleep 1
done
exit 0
```

1.  打开一个终端并运行以下命令：

```
$ bash loop_and_print.sh &
$ ps aux | grep loop_and_print.sh # Take note of the PID - write it down
```

1.  接下来，退出登录，然后在新的终端中登录并运行以下命令：

```
$ ps aux | grep loop_and_print.sh # Take note of the PID - write it down
```

1.  你能找到正在运行的进程吗？接下来，运行以下命令：

```
$ bash loop_and_print.sh & # note the PID againg
$ disown
```

1.  接下来，退出登录，然后在新的终端中登录并运行以下命令：

```
$ ps aux | grep loop_and_print.sh # Take note of the PID - write it down
```

1.  接下来，运行以下命令：

```
$ nohup bash loop_and_print.sh &
```

1.  接下来，退出登录，然后在新的终端中登录并运行以下命令：

```
$ ps aux | grep loop_and_print.sh # Take note of the PID - write it down
```

# 它的工作原理...

1.  在第 1 步中，我们打开了一个终端并创建了`loop_and_print.sh`脚本。这个脚本只是简单地永远循环，并在循环过程中打印内容。

1.  以下命令将使用`loop_and_print.sh`脚本并在后台作为**作业**运行。`ps`命令输出进程信息，并通过 grep 简化输出。在命令中，我们可以看到用户名列旁边的进程 ID（PID）。记下 PID，以便你可以杀死僵尸进程或停止不必要的应用程序：

```
$ bash loop_and_print.sh &
[1]4510
$ ps aux | grep loop_and_print.sh # Take note of the PID - write it down
rbrash 4510 0.0 0.0 12548 3024 pts/2 S 12:58 0:00 bash loop_and_print.sh
```

1.  重新登录并运行 `ps` 命令将产生*零*结果。这是因为我们用 `&` 将脚本放到后台后，已经收到关闭或终止的信号。

1.  再次运行 `loop_and_print.sh` 脚本；该命令将其放入*后台*，而 `disown` 将后台进程从已知的作业列表中移除。这*断开*了脚本和*所有*输出与终端的连接。

1.  重新登录并使用 `ps` 命令时，你将看到命令的 PID：

```
rbrash 8097 0.0 0.0 12548 3024 pts/2 S 13:02 0:00 bash loop_and_print.sh
```

1.  `nohup` 命令类似于 `disown` 命令，不同之处在于它明确*断开*了脚本与当前 shell 的连接。它与 `disown` 的不同之处在于，`nohup` 允许你在脚本执行后仍保留输出，并且可以通过其他应用程序访问位于 `nohup.out` 文件中的输出：

```
$ nohup bash loop_and_print.sh &
[2] 14256
$ nohup: ignoring input and appending output to 'nohup.out'
```

1.  重新登录并使用 `ps` 命令时，你将看到*两个*存活下来的脚本的 PID：

```
$ ps aux | grep loop_and_print.sh
rbrash 4510 0.0 0.0 12548 3024 pts/2 S 12:58 0:00 bash loop_and_print.sh
rbrash 14256 0.0 0.0 12548 3024 pts/2 S 13:02 0:00 bash loop_and_print.sh
```

# 执行需要权限的命令时

以 root 身份运行是危险的，尽管有时很方便——尤其是当你刚接触 Linux，且密码提示看起来很麻烦时。到目前为止，作为一个 Linux 用户，你可能已经见过 `sudo` 命令或 `su` 命令。这些命令可以让用户在控制台上切换用户或临时执行具有更高权限的命令（如果用户有 `sudo` 权限的话）。`Sudo`，或**替代用户执行**，使普通用户能够将其用户权限提升到更高的特权级别，仅针对某一条命令。

或者，替代用户命令 `su` 也允许你运行特权命令，甚至切换 shell（例如，成为 root 用户）。与 `su` 命令不同，`sudo` 不会激活 root shell，也不允许访问其他用户账户。

这里是这两个命令的一些示例用法：

```
$ sudo ls /root
$ su -c 'ls /root'
$ su -
```

尽管这两个命令都需要知道 root 密码，但 `sudo` 还要求执行 `sudo` 命令的用户在 `/etc/sudoers` 文件中列出：

```
$ sudo /etc/sudoers
[sudo] password for rbrash: 
#
# This file MUST be edited with the 'visudo' command as root.
#
# Please consider adding local content in /etc/sudoers.d/ instead of
# directly modifying this file.
#
# See the man page for details on how to write a sudoers file.
#
Defaults env_reset
Defaults mail_badpass
Defaults secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"

# Host alias specification

# User alias specification

# Cmnd alias specification

# User privilege specification
root ALL=(ALL:ALL) ALL

# Members of the admin group may gain root privileges
%admin ALL=(ALL) ALL

# Allow members of group sudo to execute any command
%sudo ALL=(ALL:ALL) ALL

# See sudoers(5) for more information on "#include" directives:

#includedir /etc/sudoers.d

```

在前面的标准 Ubuntu sudoers 文件中，我们可以看到管理员组的用户可以使用 `sudo` 命令（这也是你能够使用的原因，且无需调整）。我们还可以看到可以为特定用户配置权限执行：

```
root ALL=(ALL:ALL) ALL
```

这表示 root 用户可以运行系统上所有可用的命令。实际上，我们可以为名为 `rbrash` 的用户添加一行，比如 `rbrash ALL=(ALL) ALL`。

`/etc/sudoers` 可以通过具有 root 权限的用户使用 `visudo` 命令进行编辑：

```
$ sudo visudo
```

在为用户添加权限或进行修改时要小心。如果账户不安全，这可能会成为一个安全隐患！

最终，你可能会想知道为什么这对 Bash 脚本如此重要（除了能够提升权限之外）。嗯，想象一下你可能有一个系统，它执行持续集成或持续构建软件的过程（例如，Jenkins）——可能希望在没有你输入的情况下运行各种命令，因此需要授予用户访问特定命令的权限（特别是当它们是**沙箱**或位于**虚拟机**中时）。

# 准备工作

除了保持终端打开，我们还需要记住一些概念：

+   `sudo`需要密码（除非特别指定）

+   `sudo`还可以限制特定的命令、用户或主机

+   `sudo`命令也会记录在`/var/log/secure`或`/var/log/auth.log`中：

```
Dec 23 16:16:19 moon sudo: rbrash : TTY=pts/2 ; PWD=/home/rbrash/Desktop/book ; USER=root ; COMMAND=/usr/bin/vi /var/log/auth.log
Dec 23 16:16:19 moon sudo: pam_unix(sudo:session): session opened for user root by (uid=0)
```

此外，我们还可以为此步骤创建一个新用户：

```
$ sudo useradd bob
$ sudo passwd bob #use password
```

# 如何实现...

我们的活动开始如下：

1.  在新的终端中运行命令，*不要*以`root`身份运行，也不要使用任何先前的`sudo`授权：

```
$ shutdown -h 10
$ shutdown -c
```

1.  现在，执行**`$ sudo visudo`**命令，并编辑脚本以包含以下行：

```
$ sudo visudo
[sudo] password for rbrash: 
#
Defaults env_reset
Defaults mail_badpass
Defaults secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"

# Host alias specification

# User alias specification

# Cmnd alias specification

Cmnd_Alias READ_CMDS = /sbin/halt, /sbin/shutdown

# User privilege specification
root ALL=(ALL:ALL) ALL

bob ALL=(ALL:ALL) NOPASSWD: READ_CMDS

# Members of the admin group may gain root privileges
%admin ALL=(ALL) ALL

# Allow members of group sudo to execute any command
%sudo ALL=(ALL:ALL) ALL

# See sudoers(5) for more information on "#include" directives:

#includedir /etc/sudoers.d

```

1.  在新的终端中运行命令，*不要*以`root`身份运行，也不要使用任何先前的`sudo`授权：

```
$ shutdown -h 10
$ shutdown -c
```

1.  注意到有什么不同吗？现在，确保使用之前的命令取消关机：`$ shutdown -c`。

# 它是如何工作的...

上述步骤虽然简洁，但需要一些关于`sudo`的假设和知识。首先，要小心。其次，更小心。最后，要确保通过适当的密码策略保护你的账户安全：

1.  在第一步中，我们尝试运行两个需要用户权限的命令。通常，重新启动或关闭系统需要权限提升（除非通过 GUI 完成）。`shutdown -c`命令取消关机。如果现在使用`shutdown -h`，系统将立即关闭，且无法停止。

1.  在第二步中，我们使用新的`visudo`命令对`/etc/sudoers`文件进行编辑。加粗部分`Cmnd_Alias`允许你定义一组命令，但你必须使用二进制文件的完整路径。用户 Bob 也被分配到这个别名中。`NOPASSWD:`用于指定这些命令不需要密码。

1.  在第三步中，可以运行关机命令而无需输入密码。

1.  最后一步是确保意外关机被取消。

# 清理用户输入并确保结果可重复

脚本（或程序）的最佳实践之一是控制用户输入，这不仅仅是出于安全考虑，更是为了控制功能，使输入产生可预测的结果。例如，假设一个用户输入了一个数字而不是一个字符串。你检查过吗？会导致脚本提前退出吗？或者会发生一些意外事件，例如用户输入了`rm -rf /*`而不是有效的用户名？

无论如何，限制程序用户输入对你作为作者也是有益的，因为它可以限制用户的操作路径，并减少未定义行为或程序错误。因此，如果质量保证很重要，测试用例和输入/输出验证可以减少。

# 准备工作

这个过程可能会让一些读者接触到他们想避免的概念：软件工程。确实，你可能是在编写脚本以快速完成任务，但如果脚本需要由其他人使用（或者长期使用），最好在错误发生时及早捕获并防止程序出现异常行为。

即使没有正式的计算机科学或工程训练，使用案例的概念也是基于有一个特定的功能片段，通过 X 输入，查看 Y 是否按预期工作。有时可以施加限制或范围，某个操作可能完成或失败，并且任何比较的结果可以得出“用例”是否通过的结论。

让我们通过一个逐步示例来看一下，使用一个程序，通过提示来`echo`执行脚本的用户的用户名：

1.  脚本期望通过`read`命令将输入读取到一个变量中（例如）。

1.  该变量假定为字符串，但它也可以是用户的姓名、数字、外国地址、电子邮件，甚至是恶意命令。

1.  脚本读取变量并运行`echo`命令。

1.  返回的结果可能是垃圾数据，但也可能被其他脚本执行——这会出什么问题呢？

例如，默认情况下，用户名应该包含字母和数字，但除了下划线、点、破折号和名字结尾的美元符号(`$`)外，不应包含其他特殊字符。

如果安全性不重要，那么应用程序的健壮性可能就显得不那么重要了！

# 如何操作...

让我们开始我们的活动，步骤如下：

1.  首先，打开终端并创建一个名为`bad_input.sh`的新 shell 脚本，内容如下：

```
#!/bin/bash

FILE_NAME=$1
echo $FILE_NAME
ls $FILE_NAME
```

1.  现在，运行以下命令：

```
$ touch TEST.txt
$ mkdir new_dir/
$ bash bad_input.sh "."
$ bash bad_input.sh "../"
```

1.  创建一个名为`better_input.sh`的第二个脚本：

```
#!/bin/bash
FILE_NAME=$1

# first, strip underscores
FILE_NAME_CLEAN=${FILE_NAME//_/}

FILE_NAME_CLEAN=$(sed 's/..//g' <<< ${FILE_NAME_CLEAN})

# next, replace spaces with underscores
FILE_NAME_CLEAN=${FILE_NAME_CLEAN// /_}

# now, clean out anything that's not alphanumeric or an underscore
FILE_NAME_CLEAN=${FILE_NAME_CLEAN//[^a-zA-Z0-9_.]/}

# here you should check to see if the file exists before running the command
ls "${FILE_NAME_CLEAN}"
```

1.  接下来，使用以下命令运行脚本，而不是输出结果：

```
$ bash better_input.sh "."
$ bash better_input.sh "../"
$ bash better_input.sh "anyfile"
```

1.  接下来，创建一个名为`validate_email.sh`的新脚本，用于验证电子邮件地址（类似于验证 DNS 名称的方式）：

```
#!/bin/bash

EMAIL=$1
echo "${EMAIL}" | grep '^[a-zA-Z0-9._]*@[a-zA-Z0-9]*\.[a-zA-Z0-9]*
RES=$?
if [ $RES -ne 1 ]; then
    echo "${EMAIL} is valid"
else
    echo "${EMAIL} is NOT valid"
fi >/dev/null
RES=$?
if [ $RES -ne 1 ]; then
    echo "${EMAIL} is valid"
else
    echo "${EMAIL} is NOT valid"
fi
```

1.  同样，我们可以测试输出：

```
$ bash validate_email.sh ron.brash@somedomain.com
ron.brash@somedomain.com is valid
$ bash validate_email.sh ron.brashsomedomain.com
ron.brashsomedomain.com is NOT valid
```

1.  另一个常见的任务是验证 IP 地址。创建另一个名为`validate_ip.sh`的脚本，内容如下：

```
#!/bin/bash

IP_ADDR=$1
IFS=.
if echo "$IP_ADDR" | { read octet1 octet2 octet3 octet4 extra;
  [[ "$octet1" == *[[:digit:]]* ]] && 
  test "$octet1" -ge 0 && test "$octet1" -le 255 &&
  [[ "$octet2" == *[[:digit:]]* ]] && 
  test "$octet2" -ge 0 && test "$octet2" -le 255 &&
  [[ "$octet3" == *[[:digit:]]* ]] && 
  test "$octet3" -ge 0 && test "$octet3" -le 255 &&
  [[ "$octet4" == *[[:digit:]]* ]] && 
  test "$octet4" -ge 0 && test "$octet4" -le 255 &&
  test -z "$extra" 2> /dev/null; }; then
  echo "${IP_ADDR} is valid"
else
    echo "${IP_ADDR} is NOT valid"
fi
```

1.  尝试运行以下命令：

```
$ bash validate_ip.sh "a.a.a.a"
$ bash validate_ip.sh "0.a.a.a"
$ bash validate_ip.sh "255.255.255.255"
$ bash validate_ip.sh "0.0.0.0"
$ bash validate_ip.sh "192.168.0.10"
```

# 它是如何工作的...

让我们详细了解我们的脚本：

1.  首先，我们从创建`bad_input.sh`脚本开始——它接受`$1`（或参数 1），并运行列表或`ls`命令。

1.  运行以下命令时，我们可以列出目录、子目录中的所有内容，甚至可以向后遍历目录！这显然是不好的，安全漏洞甚至允许恶意黑客穿透 Web 服务器——其思路是将输入限制在可预测的结果内，并控制输入，而不是允许任何输入：

```
$ touch TEST.txt
$ mkdir new_dir/
$ bash bad_input.sh "."
...
$ bash bad_input.sh "../"
../all the files backwards
```

1.  在第二个脚本`better_input.sh`中，输入通过以下步骤进行了清理。此外，还可以检查列出的文件是否确实存在：

    1.  删除任何下划线（必要）。

    1.  删除任何一对双空格。

    1.  将空格替换为下划线。

    1.  删除任何非字母数字字符或其他不是下划线的内容。

    1.  然后，运行`ls`命令。

1.  接下来，运行`better_input.sh`将允许我们查看当前工作目录或其中包含的任何文件。通配符已经被移除，现在我们无法遍历目录。

1.  要验证电子邮件的格式，我们使用`grep`命令结合正则表达式。我们只是检查电子邮件账户名的格式、一个`@`符号，以及一个域名，格式如 acme.x。需要注意的是，我们并不是要验证电子邮件是否真正有效，或者是否能到达预期的目的地，而只是检查它是否符合电子邮件的格式。通过测试域名的 MX 或 DNS 邮件记录等额外测试，可以扩展此功能，以提高用户输入有效电子邮件的可能性。

1.  在下一步中，我们测试两个域名——一个没有`@`符号（无效），一个有`@`符号（有效）。可以尝试多个组合。

1.  验证 IP 地址通常可以通过正则表达式来完成，但为了方便使用的工具来完成任务，**读取**和简单的测试使用**test**（和评估）就可以了。IP 地址的基本形式由四个八位字节（或通俗来说，四个由句点分隔的值）组成。在不深入探讨什么是一个真正有效的 IP 地址的情况下，通常一个有效的八位字节值在`0`到`255`之间（既不大于也不小于）。IP 地址可以有不同的类别和子类，称为**子网**。

1.  在我们的示例中，我们知道包含字母字符的 IP 地址不是有效的 IP 地址（排除句点），并且每个八位字节的值范围在`0`到`255`之间。`192.168.0.x`（或`192.168.1.x`）是许多人在家用路由器上看到的 IP 子网。

# 使用 select 制作一个简单的多级用户菜单

在本书的前面部分，我们看到你可以创建一个使用递归函数和条件逻辑的脚本来制作一个简单的菜单。它有效，但另一个可以使用的工具是`select`。Select 通过提供的列表工作（例如，可以是文件的通配符选择），并将为你提供一个列表，如下所示：

```
Select a file from the list:
1.) myfirst.file
2.) mysecond.file
You chose: mysecond.file
```

显然，像“关于”这样的菜单非常简单；它对于实用功能以及诸如删除用户或修改文件/归档等可重复的子任务非常有用。

简单的 select 脚本还可以用于许多活动，例如挂载 Dropbox、解密或挂载驱动器，或者生成管理报告。

# 准备就绪

Select 已经是 Bash shell 的一部分，但它有一些不太显而易见的要点。Select 依赖于三个变量：

+   `PS3`：在创建菜单之前回显给用户的提示符

+   `REPLY`：从数组中选择的项的索引

+   `opt`：从数组中选择的项的值——而不是索引

从技术上讲，`opt`不是必须的，但它是我们示例中由 Select 迭代的元素的值。你可以使用其他名称，例如**element**。

# 如何操作...

我们的活动从以下开始：

1.  打开终端并创建一个名为`select_menu.sh`的脚本，内容如下：

```
#!/bin/bash
TITLE="Select file menu"
PROMPT="Pick a task:"
OPTIONS=("list" "delete" "modify" "create")

function list_files() {
  PS3="Choose a file from the list or type \"back\" to go back to the main: "
  select OPT in *; do 
    if [[ $REPLY -le ${#OPT[@]} ]]; then
      if [[ "$REPLY" == "back" ]]; then 
        return
      fi
      echo "$OPT was selected"
    else
      list_files
    fi
  done
}

function main_menu() {
  echo "${TITLE}"
  PS3="${PROMPT} "
  select OPT in "${OPTIONS[@]}" "quit"; do 
    case "$REPLY" in
      1 ) 
        # List
        list_files
        main_menu # Recursive call to regenerate the menu
      ;;
      2 ) 
        echo "not used"
      ;;
      3 ) 
        echo "not used"
      ;;
      4 )
        echo "not used"
      ;;
      $(( ${#OPTIONS[@]}+1 )) ) echo "Exiting!"; break;;
      *) echo "Invalid option. Try another one.";continue;;
    esac
  done
}

main_menu # Enter recursive loop
```

1.  使用以下命令执行脚本：

```
$ bash select_menu.sh
```

1.  按*1*进入文件列表功能。在菜单中输入任何文件的编号。一旦满意，输入“back”并按*Enter*返回主菜单。

# 它是如何工作的...

让我们详细了解我们的脚本：

1.  创建`select_menu.sh`脚本非常简单，但除了使用 select 外，一些概念应该看起来很熟悉：函数、返回、case 语句和递归。

1.  脚本通过调用`main_menu`函数进入菜单，然后使用 select 从`${OPTIONS}`数组生成菜单。硬编码的变量`PS3`将在菜单之前输出提示符，而`$REPLY`包含所选项的索引。

1.  按*1*并按*Enter*将导致 select 遍历各个项，然后执行`list_files`函数。此函数使用 select 第二次列出目录中的所有文件，创建一个子菜单。选择任何目录都会返回`$OPT was selected`消息，但如果输入`back`，则脚本将从此函数返回，并在其中调用`main_menu`（递归）。此时，你可以选择主菜单中的任何项。

# 生成并捕获信号以进行清理

在本书中，你可能在不知情的情况下按过*Ctrl* + *C*或*Ctrl* + *Z*——这就像在其他操作系统中按*Ctrl* + *Alt* + *Delete*对吧？嗯，从某种程度上来说，是的——它是一个信号，但在 Linux 中，它的操作非常不同。硬件层面的信号类似于一个标志或某种即时通知，表示*嘿——这里发生了某些事情*。如果已设置适当的监听器，该信号可以执行某种功能。

另一方面，软件信号要灵活得多，我们可以将信号作为*简单*的通知机制，它们比硬件信号更具灵活性。在 Linux 中，*Ctrl* + *C*等同于 SIGINT（程序中断），它通常会退出程序。它可以被停止，并且可以执行其他功能，比如清理。*Ctrl* + *Z*或 SIGTSTP（键盘停止）通常会告诉程序被**挂起**并推到后台（更多关于作业的内容将在后面部分讲解），但它也可以被阻止——就像 SIGINT 一样。

SIGHUP 是我们已经熟悉的信号——与 SIGKILL 相同。当我们使用 `disown` 或退出一个 shell 时，我们就会看到它们。有关信号的更多信息，请参阅[关于信号的详细手册页](http://man7.org/linux/man-pages/man7/signal.7.html)。

# 准备工作

除了在程序中使用键盘，我们还可以通过 `kill` 命令向程序发送信号。`kill` 命令可以终止程序，但这些信号也可以用于重新加载配置或发送用户定义的信号。你可能使用的最常见信号有：SIGHUP (1)、SIGINT (2)、SIGKILL (9)、SIGTERM (15)、SIGSTOP (17,18,23)、SIGSEGV (12) 和 SIGUSR1 (10)/SIGUSR2 (12)。后两个信号可以在你的程序中定义，或者被其他开发者利用。

`kill` 命令可以像下面这样轻松使用：

```
$ kill -s SIGUSR1 <processID>
$ kill -9 <processID>
$ kill -9 `pidof myprogram.sh`
```

`kill` 命令可以通过信号编号或信号名称来引用。它确实需要一个进程 ID 来定位目标。你可以通过 `ps | grep X` 来搜索，也可以通过使用前面的 `final` 和 `pidof` 来查找。

# 如何做到……

让我们开始进行以下操作：

1.  打开终端，创建一个新的脚本，命名为 `mytrap.sh`，内容如下：

```
#!/bin/bash

function setup() {
  trap "cleanup" SIGINT SIGTERM
  echo "PID of script is $$"
}

function cleanup() {
  echo "cleaning up"
  exit 1
}

setup

# Loop forever with a noop (:)
while :
do
  sleep 1
done

```

1.  使用 `$ bash mtrap.sh` 执行脚本。

1.  按 *Enter* 多次并观察程序的行为。

1.  按 *Ctrl* + *C*；注意到有什么不同吗？

# 它是如何工作的……

让我们详细了解一下脚本：

1.  `mytrap.sh` 脚本利用了函数和 `trap` 调用。在 `setup` 函数中，我们设置了通过 `trap` 命令调用的函数。因此，当按下 *Ctrl* + *C* 时，`cleanup` 函数会被执行。

1.  运行脚本会导致脚本在打印出 PID 后一直运行。

1.  按常规键如 *Enter* 对程序不会产生影响。

1.  按 *Ctrl* + *C* 会在控制台上输出 `cleanup`，并且脚本会使用 `exit` 命令退出。

# 在程序中使用临时文件和锁文件

另一个程序和脚本常用的机制或组件是锁文件。它通常是临时的（存储在 `/tmp` 目录下），有时用于多个实体依赖单一数据源或需要知道其他程序存在的情况。有时，仅仅是文件的存在、文件上的特定时间戳，或其他简单的工件。

有几种方法可以测试文件是否存在，但有一个重要的属性尚未演示或探讨，那就是 **隐藏** 文件的概念。Linux 中的隐藏文件并不像 Windows 中那样真正“隐藏”，但通常不会显示，除非运行了特定的标志或命令。例如，`ls` 命令不会返回隐藏文件，而 `ls` 命令加上 `-a` 标志（`-a` 代表所有）则会显示它们。

大多数文件浏览器默认不会显示隐藏文件。在 Ubuntu 中，按 *Ctrl* + *H* 可以切换该功能。

要创建一个隐藏文件，文件名的开头需要有一个 `.`（点）：

```
$ touch .myfirsthiddenfile.txt
```

除了常规文件外，我们还可以使用`mktemp`命令来创建锁文件。

# 准备工作

我们简要提到过临时文件可以存放在`/tmp`目录中。通常，`/tmp`存放的是短暂的文件，如锁文件或可以被销毁（在电源事件发生时不会对系统造成任何损害）的信息。它通常也是基于 RAM 的，这可以提供性能上的好处，特别是当它作为进程间通信系统的一部分时（更多内容将在另一个示例中讲解）。

然而，重要的是要知道其他程序可以访问`/tmp`目录中的文件，因此应该使用足够的权限来保护它。它还应该给定一个足够随机的名称，以避免文件名冲突。

# 如何操作…

让我们按如下方式开始我们的活动：

1.  打开一个新的终端，创建一个名为`mylock.sh`的新脚本，内容如下：

```
#!/bin.bash

LOCKFILE="/tmp/mylock"

function setup() { 
  # $$ will provide the process ID
  TMPFILE="${LOCKFILE}.$$"
  echo "$$" > "${TMPFILE}"

  # Now we use hard links for atomic file operations 
  if ln "${TMPFILE}" "${LOCKFILE}" 2>&- ; then
      echo "Created tmp lock"
  else
      echo "Locked by" $(<$LOCKFILE)
      rm "${TMPFILE}"
      exit 1
  fi
  trap "rm ${TMPFILE} ${LOCKFILE}" SIGINT SIGTERM SIGKILL
}

setup

echo "Door was left unlocked"

exit 0
```

1.  执行脚本`$ bash mylock.sh`并查看控制台输出。

1.  接下来，我们知道脚本正在寻找一个特定的锁文件。那么，当我们创建锁文件并重新运行脚本时会发生什么？

```
$ echo "1000" > /tmp/mylock
$ bash mylock.sh
$ rm /tmp/mylock
$ bash mylock.sh
```

# 它是如何工作的…

让我们详细了解一下我们的脚本：

1.  `mylock.sh`脚本重新使用了我们已经熟悉的几个概念：捕捉信号（traps）和符号链接（symbolic links）。我们知道，如果一个捕捉信号被调用，或者说它捕获了一个特定的信号，它可以清理锁文件（正如这个脚本中的情况）。符号链接被使用是因为它们可以在网络文件系统上的原子操作中存活。如果`LOCKFILE`位置存在文件，则表示锁定已发生。如果`LOCKFILE`缺失，则表示门已打开。

1.  当我们运行`mylock.sh`时，由于尚不存在锁文件（包括任何临时文件），我们将看到以下内容：

```
$ bash mylock.sh
Created tmp lock
Door was left unlocked
```

1.  由于前面的脚本正确退出，`SIGKILL`信号被处理并且临时锁文件被删除。在这种情况下，我们希望创建自己的锁文件，绕过这个机制。创建一个假设 PID 为`1000`的锁文件；运行脚本将返回`Locked by 1000`，删除锁文件后，常规行为将恢复（门被解锁）。

# 在等待命令完成时利用超时

有时，等待命令完成执行或忽略命令直到完成可能不是脚本中的最佳实践，尽管它确实有应用场景：

+   在某些命令执行需要不确定时间的情况下（例如，ping 一个网络主机）

+   当任务或命令以某种方式执行时，*主*脚本会等待多个操作的成功或失败。

然而，重要的是要注意，超时/等待需要一个进程，甚至是一个子 shell，以便它可以通过进程 ID（PID）进行监控。在本食谱中，我们将演示如何使用超时命令（该命令已被添加到 coreutils 包 7.0 中）来等待子 shell，并展示如何通过 trap 和 kill 命令（用于警报/计时器）实现这一点。

# 准备工作

在早期的食谱中，我们介绍了使用`trap`捕获信号，以及使用`kill`向进程发送信号。这些内容将在本食谱中进一步解释，但这里有三个新的本地 Bash 变量：

+   `$$`：返回当前脚本的 PID。

+   `$?`：返回最后一个被发送到后台的作业的 PID。

+   `$@`：返回输入变量的数组（例如，`$!`、`$2`）。

我们正在绕开作业、任务、后台和前台的概念，它们将在稍后的管理食谱中详细出现。

# 如何实现...

我们开始这本食谱时知道，Bash shell 中有一个叫做`timeout`的命令。然而，它无法提供在脚本中函数的超时功能。通过使用`trap`、`kill`和信号，我们可以设置定时器或`警报`（`ALRM`），以便从失控的函数或命令中执行干净的退出。让我们开始：

1.  打开一个新的终端，并创建一个名为`mytimeout.sh`的新脚本，内容如下：

```
#!/bin/bash

SUBPID=0

function func_timer() {
  trap "clean_up" SIGALRM
  sleep $1& wait
  kill -s SIGALRM $$
}

function clean_up() {
  trap - ALRM
  kill -s SIGALRM $SUBPID
  kill $! 2>/dev/null
}

# Call function for timer & notice we record the job's PID
func_timer $1& SUBPID=$!

# Shift the parameters to ignore $0 and $1
shift 

# Setup the trap for signals SIGALRM and SIGINT
trap "clean_up" ALRM INT

# Execute the command passed and then wait for a signal
"$@" & wait $!

# kill the running subpid and wait before exit
kill -ALRM $SUBPID 
wait $SUBPID 
exit 0
```

1.  使用带有变量次数的命令（`ping`），我们可以使用第一个参数作为超时变量来测试`mytimeout.sh`！

```
$ bash mytimeout.sh 1 ping -c 10 google.ca
$ bash mytimeout.sh 10 ping -c 10 google.ca
```

# 它是如何工作的...

你可能会问自己，*我可以将函数放到后台吗？* 当然可以——你甚至可以使用一个名为`export`并带有-f 标志的命令（尽管它可能在所有环境中不被支持）。如果你使用超时命令，你必须要么只运行你希望监控的命令，要么将该函数放到一个第二个脚本中，通过超时命令调用。显然，在某些情况下，这并不理想。在本食谱中，我们使用信号，或者更准确地说，使用`alarm`信号作为定时器。当我们设置一个特定变量的警报时，它将在计时器到期后触发`SIGALRM`！如果进程仍然存活，我们只需杀死它并退出脚本（如果我们还没有退出的话）。

1.  在步骤 1 中，我们创建了`mytimeout.sh`脚本。它使用了我们的一些新原语，例如`$!`，来监控我们发送到后台作为作业（或者在这个情况下是子 shell）执行的函数的 PID。我们“启动”计时器，然后继续执行脚本。接着，我们使用 shift 命令字面上*移动*传递给脚本的参数，忽略`$1`（或超时变量）。最后，我们监视`SIGALRM`信号，并在必要时执行清理操作。

1.  在步骤 2 中，`mytimeout.sh`使用`ping`命令执行两次，目标是`google.ca`。第一次执行时，使用`1`秒的超时，第二次执行时，使用`10`秒的超时。在这两次操作中，Ping 将进行 10 次 ping 操作（例如，ping 一次，往返到响应 ICMP 请求的任何主机，针对 google.ca 的 DNS 条目）。第一次会较早执行，而第二次则允许 10 次 ping 操作顺利执行并退出：

```
$ bash mytimeout.sh 1 ping -c 10 google.ca
PING google.ca (172.217.13.99) 56(84) bytes of data.
64 bytes from yul02s04-in-f3.1e100.net (172.217.13.99): icmp_seq=1 ttl=57 time=10.0 ms

$ bash mytimeout.sh 10 ping -c 10 google.ca
PING google.ca (172.217.13.99) 56(84) bytes of data.
64 bytes from yul02s04-in-f3.1e100.net (172.217.13.99): icmp_seq=1 ttl=57 time=11.8 ms
64 bytes from yul02s04-in-f3.1e100.net (172.217.13.99): icmp_seq=2 ttl=57 time=14.5 ms
64 bytes from yul02s04-in-f3.1e100.net (172.217.13.99): icmp_seq=3 ttl=57 time=10.8 ms
64 bytes from yul02s04-in-f3.1e100.net (172.217.13.99): icmp_seq=4 ttl=57 time=13.1 ms
64 bytes from yul02s04-in-f3.1e100.net (172.217.13.99): icmp_seq=5 ttl=57 time=12.7 ms
64 bytes from yul02s04-in-f3.1e100.net (172.217.13.99): icmp_seq=6 ttl=57 time=13.4 ms
64 bytes from yul02s04-in-f3.1e100.net (172.217.13.99): icmp_seq=7 ttl=57 time=9.15 ms
64 bytes from yul02s04-in-f3.1e100.net (172.217.13.99): icmp_seq=8 ttl=57 time=14.0 ms
64 bytes from yul02s04-in-f3.1e100.net (172.217.13.99): icmp_seq=9 ttl=57 time=12.0 ms
64 bytes from yul02s04-in-f3.1e100.net (172.217.13.99): icmp_seq=10 ttl=57 time=11.2 ms

--- google.ca ping statistics ---
10 packets transmitted, 10 received, 0% packet loss, time 9015ms
rtt min/avg/max/mdev = 9.155/12.307/14.520/1.545 ms
```

请注意，google.ca 可以替换为其他 DNS 名称，但执行时间可能因您的位置而异。因此，10 次 PING 可能实际上无法完全执行。

# 创建文件进文件出程序并并行运行进程

在这个方案中，我们使用了一个叫做文件进文件出（FIFO）的概念，也叫做**管道**，通过它将参数传递给多个“工作”脚本。这些工作脚本并行运行（换句话说，主要独立于主进程），读取输入并执行命令。FIFO 非常有用，因为它们可以减少文件系统操作或输入/输出（IO），数据可以直接流向监听者或接收者。它们在文件系统中表现为文件，并且是双向的——可以同时读取和写入。

# 准备工作

要创建 FIFO，我们使用`mkinfo`命令来创建一个看似文件的东西（在 Linux 中，一切皆文件）。不过，这个文件有一个特殊的属性，与普通文件不同，也与我们之前使用的管道不同：在这种情况下，管道允许多个读者和写者！

与任何文件一样，您还可以使用`-m`标志提供权限，例如：`-m a=rw`，或者使用`mknod`命令（这里不再详细说明，因为它需要您使用第二个命令`chown`来更改**`权限`**，这是文件创建后的操作）。

# 如何操作...

为了开始本练习，我们将引入两个术语：领导者和跟随者，或者主机和工作者。在这种情况下，主机（中央主机）将创建工作者（或从属进程）。尽管这个方案稍显牵强，但它应该是一个简单的**命名管道**或 FIFO 模式的易用模板。从本质上讲，主机创建了五个工作者，而这些新创建的工作者通过命名管道回显出提供给它们的数据：

1.  为了开始，打开一个新的终端并创建两个新的脚本：`master.sh`和`worker.sh`。

1.  在`master.sh`中，添加以下内容：

```
#!/bin/bash

FIFO_FILE=/tmp/WORK_QUEUE_FIFO
mkfifo "${FIFO_FILE}"

NUM_WORKERS=5
I=0
while [ $I -lt $NUM_WORKERS ]; do

  bash worker.sh "$I" &
  I=$((I+1))

done 

I=0
while [ $I -lt $NUM_WORKERS ]; do

  echo "$I" > "${FIFO_FILE}"
  I=$((I+1))

done 

sleep 5 
rm -rf "${FIFO_FILE}"
exit 0
```

1.  在`worker.sh`中，添加以下内容：

```
#!/bin/bash

FIFO_FILE=/tmp/WORK_QUEUE_FIFO

BUFFER=""
echo "WORKER started: $1"

while :
do

read BUFFER < "${FIFO_FILE}"

if [ "${BUFFER}" != "" ]; then
  echo "Worker received: $BUFFER"
  exit 1
fi

done

exit 0
```

1.  在终端中运行以下命令，并观察输出：

```
$ bash master.sh
```

# 它是如何工作的...

本方案的理念是，如果你有多个重复性任务，例如批量操作和可能的多核处理，你可以并行执行任务（在 Linux 世界中通常称为作业）。该方案创建一个主脚本，并将多个工作脚本分配到后台，它们等待来自命名管道的输入。一旦它们从命名管道读取到输入，就会将其回显到屏幕并退出。最终，主脚本也会退出，并将管道一同删除：

1.  在第 1 步中，我们打开一个新的终端并创建两个脚本：`master.sh` 和 `worker.sh`。

1.  在第 2 步中，我们创建了 `master.sh` 脚本。它使用两个 while 循环创建 *n* 个工作脚本，并为它们分配 `$I` 标识符，然后将相同数量的值发送到 FIFO 队列中，之后休眠或退出。

1.  在第 3 步中，我们创建了 `worker.sh` 脚本，它回显初始化消息，并等待直到 `$BUFFER` 不为空（有时也称为 NULL）。一旦 `$BUFFER` 被填满，或者说，包含了一个消息，它会将其回显到控制台并退出脚本。

1.  在第 4 步中，控制台应该输出类似以下内容：

```
$ bash master.sh 
WORKER started: 0
WORKER started: 1
We got 0
We got 1
WORKER started: 4
We got 2
WORKER started: 2
We got 3
WORKER started: 3
We got 4

```

通过这两个脚本在 FIFO 上协同工作，一个数字值在它们之间传递，工作脚本则执行它们的 *工作*。这些值或消息可以轻松修改，使得工作脚本执行命令！

请注意，输出的顺序可能不同。这是因为 Linux 是非确定性的，生成进程或读取 FIFO 可能会被阻塞，或者某些进程可能会先到达（由于调度的原因）。请记住，FIFO 也不是原子操作或同步的——如果你希望指定哪个消息发送到哪个主机，你可以创建一个标识符或消息传递方案。

# 在启动时执行脚本

本方案不仅限于在启动时运行应用程序或服务，还可以在系统启动（开机）时启动脚本。例如，如果你的系统启动后，你希望对操作系统应用一些调整，如性能增强或电池优化，你可以通过 `systemd` 或 `init.d` 脚本在启动时完成这些调整。另一个例子是运行一个永不结束的脚本，生成日志事件，就像电子版的脉搏监视器一样。

简而言之，Linux 或大多数 *NIX 系统使用古老的 `rc.d` 系统或较新且争议较大的 systemd 系统来管理系统资源的启动和停止。无需深入了解整个 Linux 启动序列，以下是其工作原理：

1.  Linux 内核被加载并挂载根文件系统。

1.  根文件系统在特定路径下包含一个 shell（初始化级别）。

1.  然后，systemd 会依次启动一系列服务（运行级别）。

如果添加了服务或脚本，它很可能会被添加到 **运行级别**。它还可以随时通过命令行启动、停止、重载和重启。当系统启动时，它仅使用 `init.d` 或 `system.d` 脚本提供的启动功能。然而，尽管 rc.d 或 systemd 系统的语义有所不同，它们仍然需要以下内容：

+   脚本或服务需要为特定的系统启动级别启用。

+   要启动和/或停止的脚本可以配置为按特定顺序启动。

+   启动（至少）、停止、重启和/或重载的指令会根据执行这些操作（或块）时的参数执行。例如，调用 start 时，也可以执行多个命令。

当你在网上寻找资源时，可能会注意到有许多不同的初始化系统：upstart、SysVinit、rc.d、procd，等等。你可以查阅你的发行版文档，了解当前使用的启动系统的相关说明。

# 准备工作

在 Ubuntu 16.04 LTS（及其他发行版）中，使用 systemd。了解 init.d 和 systemd 服务控制系统是非常值得的，因为许多嵌入式系统使用 Busybox。BusyBox 使用 init.d 系统而不是 systemd。

在进入“如何做”部分之前，我们将创建一个模板初始化脚本，以便以后参考，并确保你在遇到它们时了解它。它将被命名为 `myscript`，并将运行 `myscript.sh`。至少，一个兼容 system.d 的脚本看起来应该如下所示：

```
#!/bin/sh
### BEGIN INIT INFO
# Provides: myscript
# Required-Start: $local_fs
# Required-Stop: $local_fs
# Default-Start: 2 3 4 5
# Default-Stop: 0 1 6
# Short-Description: Start script or daemons example
### END INIT INFO

PATH=/sbin:/bin:/usr/sbin:/usr/bin
DAEMON_NAME=myscript.sh
DESC=myscript

DAEMON=/usr/sbin/$DAEMON_NAME

case $1 in
  start)
  log_daemon_msg "Starting $DESC"
  $DAEMON_NAME
  log_end_msg 0
  ;;
  stop)
  log_daemon_msg "Stopping $DESC"
  killall $DAEMON_NAME
  log_end_msg 0
  ;;
  restart|force-reload)
  $0 stop
  sleep 1
  $0 start
  ;;
  status)
  status_of_proc "$DAEMON" "$DESC" && exit 0 || exit $?
  ;;
  *)
  N=/etc/init.d/$DESC
  echo "Usage: $N {start|stop|restart|force-reload|status}" >&2
  exit 1
  ;;
esac

exit 0

# vim:noet
```

如果你已经按照本手册的内容进行过操作，那么这些内容应该比较容易理解。它依赖于头部的注释来确定名称、运行级别、顺序和依赖关系。在此之后，它使用一个 switch 语句，查找一些预定的/标准化的参数：start、stop、restart、force-reload 和 status。在 start 的 case 中，我们启动二进制文件，而在 stop 的 case 中，我们使用 `killall` 函数来停止二进制文件。

需要知道的是，单纯安装初始化脚本并不能保证它会在启动时执行。必须有一个过程来**启用**该服务或脚本。在较旧的系统（SysV）中，你可能听说过/见过使用 `chkconfig` 命令。在 systemd 中，你可以使用 `systemctl` 命令来启用/禁用服务。在本节中，我们只关注 systemd。

SysV 会根据文件名中的数字顺序（例如，`S99-myinit`）按顺序执行脚本。Systemd 则不同，因为它还会检查依赖关系并等待其完成。

# 如何做...

让我们按如下方式开始我们的活动：

1.  创建一个名为 `myscript.sh` 的脚本，内容如下：

```
#!/bin/bash

for (( ; ; ))
do
   sleep 1
done
```

1.  接下来，让我们为脚本添加正确的权限，以便我们可以使用它创建 systemd 服务。请注意使用 `sudo` 命令——在适当的地方输入密码：

```
$ sudo cp myscript.sh /usr/bin
$ sudo chmod a+x /usr/bin/myscript.sh
```

1.  现在我们有了启动时执行的内容，我们需要创建一个服务配置文件来描述我们的服务；在这个示例中我们使用了`vi`和`sudo`（记住它的位置）：

```
$ sudo vi /etc/systemd/system/myscript.service 
[Unit]
Description=myscript

[Service]
ExecStart=/usr/bin/myscript.sh
ExecStop=killall myscript.sh

[Install]
WantedBy=multi-user.target
```

1.  要启用`myscript`服务，请运行以下命令：

```
$ sudo systemctl enable myscript # disable instead of enable will disable the service
```

1.  要启动并验证进程的存在，请运行以下命令：

```
$ sudo systemctl start myscript
$ sudo systemctl status myscript
```

1.  您可以重新启动系统以查看我们的服务在启动时的运行情况。

# 工作原理...

让我们详细了解我们的脚本：

1.  在步骤 1 中，我们创建了一个简单的循环程序，用于系统启动时运行，名为`myscript.sh`。

1.  在步骤 2 中，我们将脚本复制到`/usr/bin`目录，并使用`chmod`命令为所有人添加执行权限（`chmod a+x myscript.sh`）。注意使用`sudo`权限在此目录中创建文件并应用权限。

1.  在第三步中，我们创建了描述 systemd 服务单元的服务配置文件。它的名字叫做`myscript`，在`[Service]`指令中，有两个最重要的参数：`ExecStart`和`ExecStop`。注意启动和停止部分看起来类似于 SysV/init.d 的方法。

1.  接下来，我们使用`systemctl`命令来启用`myscript`。相反，它可以用以下方式来禁用`myscript：` `` $systemctl disable myscript`.` ``

1.  然后，我们使用`systemctl`来启动`myscript`并验证我们脚本的状态。你应该会得到类似以下的输出（请注意我们使用`ps`进行了双重检查）：

```
$ sudo systemctl status myscript
myscript.service - myscript
   Loaded: loaded (/etc/systemd/system/myscript.service; enabled; vendor preset:
   Active: active (running) since Tue 2017-12-26 14:28:51 EST; 6min ago
 Main PID: 17966 (myscript.sh)
   CGroup: /system.slice/myscript.service
           ├─17966 /bin/bash /usr/bin/myscript.sh
           └─18600 sleep 1

Dec 26 14:28:51 moon systemd[1]: Started myscript.
$ ps aux | grep myscript
root 17966 0.0 0.0 20992 3324 ? Ss 14:28 0:00 /bin/bash /usr/bin/myscript.sh
rbrash 18608 0.0 0.0 14228 1016 pts/20 S+ 14:35 0:00 grep --color=auto myscript
```

1.  在重启时，如果启用了，我们的脚本将按预期运行。
