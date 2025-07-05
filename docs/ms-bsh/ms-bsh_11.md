# 作为守护进程运行

在我们翻阅本书的过程中，我们看到了很多有趣的东西，玩弄了进程，发送了信号，将任务放到后台，并编写了复杂的脚本。到目前为止所做的一切都有一个目标：让我们从 Bash 中获得最佳的使用效果，让它为我们处理重复任务，并使用内建命令、循环和外部命令来简化我们作为高级用户的日常生活。然而，有时我们需要让我们的脚本长期运行，可能是无限期地活跃，所以下面就需要使用守护进程，而不仅仅是作为普通程序运行。我们必须踏上成为守护进程的那条模糊之路。

# 什么是守护进程？

那么，守护进程与普通程序有什么不同呢？我们通常想用守护进程来获得以下一些功能：

+   永久运行

+   提供服务

+   即使调用会话结束，仍然可以存活

+   不会锁定终端

+   不会锁定任何子目录

这就是我们所知的守护进程的基本功能。想象一下 SSHD 守护进程、FTPD 或 Apache：

+   在后台运行

+   提供你与之交互的服务，通过一个套接字

+   可以启动或停止，但无法从命令行进行进一步的直接交互

+   在你登录时可用，并且在你注销时依然存在

+   它们在后台运行

你实际上根本不知道它们是如何做到这一切的。

# 示例

那么，我们如何将一个脚本转变为守护进程呢？一个初步尝试可能是使用 `&`。尾随的 `&` 是 Bash 内建命令，它指示 shell 在子 shell 中后台运行命令。一旦命令执行，shell 不会等待它完成，而是返回代码 `0`（表示成功），并继续执行其他命令：

```
zarrelli:~$ ls -lah & ps -jf [1] 13704 total 48K drwxr-xr-x 4 zarrelli zarrelli 4.0K Apr 12 14:12 . drwxr-xr-x 4 zarrelli zarrelli 4.0K Apr 12 19:37 .. -rw-r--r-- 1 zarrelli zarrelli 11 Apr 10 09:20 controller -rwxr--r-- 1 zarrelli zarrelli 121 Apr 11 18:30 coproc.sh -rwxr--r-- 1 zarrelli zarrelli 961 Apr 11 12:19 environment.sh -rwxr--r-- 1 zarrelli zarrelli 382 Apr 11 10:08 looping.sh -rw-r--r-- 1 zarrelli zarrelli 0 Apr 10 09:20 myfile.txt -rw-r--r-- 1 zarrelli zarrelli 122 Apr 10 09:20 myfile.txt.tgz -rw-r--r-- 1 zarrelli zarrelli 367 Apr 12 14:12 my_index.html prw-r--r-- 1 zarrelli zarrelli 0 Apr 9 13:05 mypipefile -rwxr--r-- 1 zarrelli zarrelli 223 Apr 9 12:44 pipe.sh drwxr-xr-x 2 zarrelli zarrelli 4.0K Apr 10 12:20 test 1 drwxr-xr-x 2 zarrelli zarrelli 4.0K Apr 10 12:20 test 2 -rw-r--r-- 1 zarrelli zarrelli 29 Apr 12 12:06 testfile.txt UID PID PPID PGID SID C STIME TTY TIME CMD zarrelli 1385 1272 1385 1385 0 08:14 pts/0 00:00:00 /bin/bash zarrelli 13705 1385 13705 1385 0 10:55 pts/0 00:00:00 ps -jf [1]+ Done ls --color=auto -lah 

```

我们在示例中看到的是，shell 执行了第一个 `ls` 命令并返回了这个：

```
[1] The job number 13704 The process ID

```

但是，它并没有等待 `ls` 进程完成工作；它只是将其分叉到一个子 shell 中，然后继续执行 `ps` 命令。为了进行我们的实验，让我们创建一个空的 shell 和一个具有无限循环的脚本，这个脚本实际上什么也不做：

```
#!/bin/bash
while true
do
:
done

```

没什么特别的，唯一有趣的是，一旦启动，脚本将一直执行，直到我们停止它。现在，让我们运行它：

```
zarrelli:~$ ./while.sh 

```

好吧，我们说它什么都不做，但实际上它在做一些事情：它占据了你的终端，直到终止或被送入后台，它才会把终端交还给你。所以，我们有两个选择：

*Ctrl* + *C* 发送 `SIGKILL` 信号到进程并终止它：

```
zarrelli:~$ ./while.sh ^C

```

*Ctrl* + *Z* 发送 `SIGTSTP`，它会暂停执行：

```
zarrelli:~$ ./while.sh ^Z [1]+ Stopped ./while.sh

```

一旦被暂停，我们可以使用其作业 ID 将任务放入后台，在我们的例子中是 `[1]`：

```
zarrelli:~$ bg %1 [1]+ ./while.sh &

```

如果现在检查作业的状态，它将是这样的：

```
zarrelli:~$ jobs [1]+ Running ./while.sh &

```

我们可以看到，脚本不再停止，而是实际上在后台运行。此时，你可能已经忘记了运行脚本的子壳的 PID，或者你根本不知道有一种快速的方法可以召回它，因为它保存在`$!`变量中：

```
zarrelli:~$ echo $! 18672

```

现在，让我们将这个过程带到前台：

```
zarrelli:~$ fg %1 ./while.sh

```

杀死它，因为它再次占用了终端：

```
zarrelli:~$ ./while.sh ^C

```

如果我们直接在后台使用`&`运行多个脚本实例，会发生什么情况呢？

```
zarrelli:~$ ./while.sh & ./while.sh & ./while.sh & [1] 20167 [2] 20168 [3] 20169

```

所以，它们都在后台运行，并且拥有各自的作业 ID：

```
zarrelli:~$ jobs [1] Running ./while.sh & [2]- Running ./while.sh & [3]+ Running ./while.sh &

```

在`jobs`的输出中有一些新内容，那些是靠近作业 ID 的`–`和`+`字符：

+   `+`：这标识了`fg`或`bg`默认会操作的作业

+   `-`：这标识了如果当前默认作业退出，将成为默认作业的作业

让我们进行一个测试。首先，检查作业的状态：

```
zarrelli:~$ jobs [1] Running ./while.sh & [2]- Running ./while.sh & [3]+ Running ./while.sh &

```

它们都在后台运行。让我们将默认作业召回前台：

```
zarrelli:~$ fg ./while.sh

```

现在，让我们用*Ctrl*+*Z*暂停它：

```
./while.sh ^Z [3]+ Stopped ./while.sh

```

所以，我们只是给出了没有参数的`fg`命令；如预期的那样，ID 为`3`且带有`+`字符的作业被拉回了前台。现在，让我们检查作业的状态：

```
zarrelli:~$ jobs [1] Running ./while.sh & [2]- Running ./while.sh & [3]+ Stopped ./while.sh

```

第三个作业被停止，但我们可以看到`+`字符。让我们再次将默认作业召回前台，然后停止它：

```
zarrelli:~$ fg ./while.sh ^Z [3]+ Stopped ./while.sh

```

再次，第三个作业是默认的，因为它从未终止，它只是被挂起了。所以，现在是时候优雅地终止它了：

```
zarrelli:~$ kill -15 %3 [3]+ Terminated ./while.sh

```

让我们看看在杀死默认作业之后，作业的状态：

```
zarrelli:~$ jobs [1]- Running ./while.sh & [2]+ Running ./while.sh &

```

就这样，作业 ID 为`2`的作业现在是默认的，而编号为`1`的作业排在第二位。

# `nohup`

`nohup`是一个**可移植操作系统接口**（**POSIX**）命令，防止作为参数传递的进程接收到**挂起（HUP）**信号。如果我们在脚本前加上`nohup`运行，它将被保护，不受交互式会话关闭时发送给所有进程的 HUP 信号影响。如果标准输出是终端，`nohup`会将其附加到本地目录中的`nohup.out`文件中，如果无法在用户的主目录中写入，它会将标准错误重定向到`stdout`。所以下面是这样的情况：

```
zarrelli:~$ nohup ./while.sh & [1] 14247
nohup: ignoring input and appending output to 'nohup.out'

```

脚本在后台运行，正如`jobs`命令正确报告的那样：

```
zarrelli:~$ jobs [1]+ Running nohup ./while.sh &

```

所以，脚本已经被分离，`stdout`被重定向到`nohup.out`文件，而`stdin`被忽略：

```
zarrelli:~$ ls -lah total 12K drwxr-xr-x 2 zarrelli zarrelli 4.0K Apr 13 14:35 . drwxr-xr-x 4 zarrelli zarrelli 4.0K Apr 13 14:32 .. -rw------- 1 zarrelli zarrelli 0 Apr 13 14:35 nohup.out -rwxr--r-- 1 zarrelli zarrelli 35 Apr 13 11:06 while.sh

```

现在，让我们使用`exit`退出我们的交互式会话，并重新创建一个新的会话。我们只需打开一个新终端并使用`jobs`命令：

```
zarrelli:~$ jobs

```

没有任何内容，没有作业被列出。为什么？进程还在吗？我们来看看：

```
zarrelli:~$ ps ax | grep while 14247 ? R 8:49 /bin/bash ./while.sh 14839 pts/0 S+ 0:00 grep while

```

脚本仍在运行，`PID`没有变化，那么为什么我们在作业列表中看不到它？因为我们关闭了旧的 shell 并打开了新的 shell；因此，旧的作业列表与旧的 shell 相关联，已经被销毁。这是可取的，因为没有作业 ID，shell 就无法直接控制进程或与之干扰。然后，看看进程列表中的第二个字段：

```
14247 ? R 8:49 /bin/bash ./while.sh 14839 pts/0 S+ 0:00 grep while

```

虽然 `grep` 关联了一个终端 `pts/0`，但是 `while` 脚本没有任何关联的终端，因此我们看到了 `?`，这正是我们一开始想要的。在继续之前，让我们清理一下，杀掉脚本：

```
zarrelli:~$ kill 14247

```

很好，一切都清晰、简单、易懂，是吧？不对。有时我们只是通过 SSH 在远程服务器上启动一个应用程序。我们使用 `nohup` 和 `&` 完全从终端中分离，并将它从会话关闭时的 HUP 信号中保护起来；然后当我们尝试注销时，我们的连接就会无限期地挂起。发生了什么？为什么一切看起来都挂起了？这个行为是由于处理 SSH 连接的 OpenSSH 服务器所致：在关闭连接之前，OpenSSH 会等待读取连接到用户运行的进程的 `stdout` 和 `stderr` 管道的 **文件结束符 (eof)**。这里的问题与 Unix 中文件如何返回 eof 有关，只有在所有引用都被关闭时它才会返回 eof。但当你在通过 SSH 连接的 shell 后台运行一个进程时，该进程会得到与其运行的 shell 的 `stdout` 和 `stderr` 的标准引用。当你关闭 shell 时，OpenSSH 服务器会失去这些引用，因为 shell 已经死掉，因此它永远看不到来自这些引用的 eof 信号。所以，它会使连接无限期挂起。那么，如何防止这种情况呢？其实，解决方法是手动在退出前关闭该进程，或者在启动进程时重定向标准流（`stdin`、`stdout`、`stderr`）的引用。

```
nohup command > foo.out 2> foo.err < /dev/null &

```

不幸的是，即使重定向有时也不起作用，因为 OpenSSH 对一堆原因和情况非常敏感，不会向进程发送任何 HUP 信号。

# disown

如果我们启动一个进程，然后想要在交互式 shell 关闭后仍然保持它的运行状态，该怎么办？让我们回顾一下 shell 退出时会发生什么：在退出之前，它会向所有正在运行的作业发送 `SIGHUP` 信号。如果一个作业处于暂停状态，shell 会向它发送 `SIGCONT` 信号以恢复执行，这样它就可以接收到 `SIGHUP` 信号并优雅地退出。为了完成这个任务，shell 会浏览一个包含所有作业的表格，接下来就是诀窍。让我们在后台启动一个脚本几次：

```
zarrelli:~$ ./while.sh & ./while.sh & ./while.sh & [1] 8944 [2] 8945 [3] 8946

```

现在，让我们看看 shell 的作业表：

```
zarrelli:~$ jobs [1] Running ./while.sh & [2]- Running ./while.sh & [3]+ Running ./while.sh &

```

我们可以看到预期中的所有三个进程都在运行。现在，开始做有趣的部分：

```
zarrelli:~$ disown %2

```

ID 为 `2` 的作业发生了什么？

```
zarrelli:~$ jobs [1]- Running ./while.sh & [3]+ Running ./while.sh &

```

好的，它已经从作业表中消失，但它仍然在那里：

```
zarrelli:~$ ps -p 8945 PID TTY TIME CMD 8945 pts/0 00:05:18 while.sh

```

`ps` 命令后面加上 `-p` 和 `pid` 只是用来显示我们选定的进程 `PID`。它刚刚显示了我们的“脱离”的作业仍然在运行。所以，使用 `disown`，我们只是把一个作业从 shell 的作业列表中移除了；因此，当 shell 退出时，它不会向该作业发送 `SIGHUP` 信号，正如如果没有使用 `disown` 时会发送的那样。实际上，我们甚至可以进一步操作：

```
zarrelli:~$ disown -h %1 zarrelli:~$ jobs [1]- Running ./while.sh & [3]+ Running ./while.sh &

```

任务仍然存在，但已经标记为在 shell 退出时不会收到 `SIGHUP` 信号。你可以选择使用没有 `ID` 和 `-a` 的 `disown` 来移除或标记任务表中的所有 ID。如果没有 `ID` 和 `-r`，则操作将仅限于运行中的任务。

在你的交互式 shell 关闭后，背景进程没有被终止吗？我们来检查一下：

```
shopt | grep huponexit

```

`huponexit` 设置为关闭。这可能是背景进程在 shell 退出时未被终止的原因。我们可以通过以下方式暂时将其开启：

```
shopt -s huponexit; shopt | grep huponexit

```

为了使其永久生效，可以将其设置在 `~/.bashrc` 或 `/etc/bashrc` 文件中，使用 `shopt -s huponexit`。

# 双重派生和 setsid

有几种方法可以将进程转为守护进程，虽然可能不太常见，但非常有趣；这些方法包括**双重派生**和**setsid**。

双重派生是将进程转为守护进程的常用方法，意味着派生一个子进程，即通过复制父进程来创建一个子进程。在应用于守护进程化的双重派生中，父进程先派生一个子进程，然后终止它。接着，子进程再次派生自己的子进程并终止。这样，链条末端的两个父进程会死亡，只有孙进程存活并作为守护进程运行。这样做的原因与会话的控制终端分配方式有关，因为被派生的子进程会继承其父进程的控制终端。

在交互式会话中，shell 是第一个被执行的进程，因此它是终端的控制进程，也是会话的会话领导进程，所有在该会话中派生的进程都会继承它的控制终端。通过派生并终止父进程，我们得到了一个孤儿进程，它会自动被重新父化为 `init`，成为系统主进程的子进程。所有这一切的目的是为了防止子进程成为会话领导进程并获取控制终端；这就是为什么我们需要双重派生并两次终止父进程的原因：我们希望让子进程成为孤儿进程，以便系统将其重新父化为 `init`，从而防止它变成僵尸进程。因为它不是管道中的第一个进程，所以不能成为会话领导进程，也不能获得控制终端。这样，子进程就被移动到一个不同的会话中，不再控制控制终端，实际上变成了守护进程。

让我们来看看，并将脚本放到后台运行：

```
zarrelli:~$ ./while.sh & [1] 17460

```

让我们看看进程的 ID：

```
zarrelli:~$ ps -Ho pid,ppid,pgid,tpgid,sess,args PID PPID PGID TPGID SESS COMMAND 10355 1401 10355 17515 10355 /bin/bash 17460 10355 17460 17515 10355 /bin/bash ./while.sh 17515 10355 17515 17515 10355 ps -Ho pid,ppid,pgid,tpgid,sess,args

```

会话 ID 与它派生自的 shell 相同，但它有自己的进程组 ID 和**父进程 ID**（**PPID**）等于它的父进程 ID。让我们看看脚本在进程树中所处的位置：

```
zarrelli:~$ pstree | grep -B3 while | `-{gmain} |-login---bash-+-grep | |-pstree | `-while.sh

```

正如预期的那样，它嵌套在登录会话中，因此它是该会话的一部分。现在，让我们进行双重派生：

```
zarrelli:~$ (./while.sh &) & [1] 17846

```

看一下这个过程：

```
zarrelli:~$ ps -Ho pid,ppid,pgid,tpgid,sess,args PID PPID PGID TPGID SESS COMMAND 10355 1401 10355 17970 10355 /bin/bash 17970 10355 17970 17970 10355 ps -Ho pid,ppid,pgid,tpgid,sess,args 17847 1 17846 17970 10355 /bin/bash ./while.sh

```

执行`while`的 shell 的 PPID 现在变得非常有趣；它的值变成了`1`。这意味着它的父进程不再是登录会话中启动的 shell，而是`init`进程。但请注意，它仍然共享相同的会话 ID 和相同的终端。我们可以通过`pstree`再次确认：

```
zarrelli:~$ pstree | grep -B3 while | `-{probing-thread} |-upowerd-+-{gdbus} | `-{gmain} |-while.sh

```

由于我们直接被重新归属于第一层`init`，所以没有任何嵌套。

使用`setsid`，我们得到一个稍微不同的结果。每当一个不是进程组领导的进程调用`setsid`时，它会创建一个新的会话，并使调用进程成为该会话的会话领导，成为新创建进程组的进程组领导，并且没有控制终端。因此，我们本质上创建了一个新会话，持有一个新进程组，并且只有一个进程，即调用进程。会话和进程组 ID 都设置为调用进程的进程 ID。我们希望将进程转为守护进程，但有一个缺点，那就是除非我们重定向到文件，否则没有任何输出：

```
setsid command > file.log

```

让我们将脚本转为守护进程：

```
zarrelli:~$ setsid ./while.sh zarrelli:~$ ps -e -Ho pid,ppid,pgid,tpgid,sess,args | grep while 22853 10355 22852 22852 10355 grep while 22572 1 22572 -1 22572 /bin/bash ./while.sh

```

这一次，我们需要使用`ps`的`-e`选项来显示所有进程，并且使用`grep`，因为`ps`默认情况下只显示与当前用户具有相同效应用户 ID 并且与当前终端相同的进程。在这种情况下，我们更改了终端，所以它不会显示。最后，让我们看一下`pstree`：

```
zarrelli:~$ pstree | grep -B3 while | `-{probing-thread} |-upowerd-+-{gdbus} | `-{gmain} |-while.sh

```

正如我们所预期的，由于`PPID`为`1`，我们在第一层看到了嵌套。该进程，在我们的例子中是执行脚本的 shell，被重新归属于`init`，没有任何控制终端。

现在我们已经检查了几种将进程有效地放到后台并使其避免会话关闭的方法，我们可以继续进一步，看看如何编写可以自动转为守护进程的脚本，使其进入后台并且无需用户交互地工作。当然，也有一些变通方法，比如使用工具：屏幕和终端复用器，这些工具允许你将会话从终端中分离，以便即使用户注销，进程仍然可以继续运行。无论如何，这不是我们的目标，我们不是在回顾外部工具，而是在尝试从 Bash 中找出最佳方法，所以接下来的段落将会探讨一些不同的方法，如何让 Bash 将我们的脚本转为守护进程。

# 成为守护进程

守护进程的生活并不轻松，需要经历父进程的无数“残酷死亡”。

使进程成为守护进程所需的第一步是通过`fork`创建一个新进程，以便父进程可以退出，命令行提示符返回给调用的 shell。这确保新进程不是进程组领导者，因为进程组领导者不能通过调用`setsid`创建新会话。因此，新的子进程现在可以通过调用`setsid`提升为进程组领导者和会话领导者。到目前为止，新会话没有控制终端，新的子进程也没有。因此，我们再次调用`fork`以确保会话和组领导者能够退出。现在，孙子进程不是一个会话，因此它将要打开的终端不能是其控制终端。这就是 Linux 进程生活的艰难之处；如果它不是会话领导者，它将要打开的终端就不是调用进程的控制终端。

现在，进程已经与控制终端分离，但我们仍然有一个问题：它正在锁定它被调用的目录，所以如果我们尝试卸载它，我们会失败。下一步是让进程将工作目录更改为`/`，即文件系统的根目录（`chdir` `/`），或者更改为任何包含进程运行所需文件的目录。我们快完成了。一个好的做法是为进程设置`umask 0`，以便重置`umask`。进程可能已经继承了`umask`并会根据`open()`调用创建具有相应权限的文件。我们已经接近完成；下一步是让进程关闭从父进程继承的标准文件描述符（`stdin`，`stdout`，`stderr`），并打开一组新的文件描述符。

# 捕获守护进程

在投入创建守护进程的黑魔法之前，你应该学会如何防止它受到任何可能导致其死亡的信号。如我们在前几章中所见，如果进程死亡，它可能会留下混乱，因为它没有时间清理*房子*。这很可怕，但我们可以做些事情来防止这一切发生：使用陷阱帮助我们处理信号并创建更强大、功能更完善的脚本。在我们的案例中，内建的`trap`将非常有用，用来监控我们的脚本行为，因为它是一个信号处理程序，可以修改进程如何响应信号。`trap`的通用语法如下：

```
trap commands signal_list

```

命令作为可以执行的列表，其中包括接收到信号时要执行的函数。我们已经看过一些信号及其数值，但`trap`可以使用一些关键字来处理最常见的信号，如下表所示：

| **信号** | **数字值** |  |
| --- | --- | --- |
| `HUP` | `1` | 挂断。意味着控制终端已退出。 |
| `INT` | `2` | 中断，当按下*Ctrl* + *C*时会发生。 |
| `QUIT` | `3` | 退出。 |
| `KILL` | `9` | 这是一个无法捕获的信号。接收到该信号时，进程必须退出。 |
| `TERM` | `15` | 终止，是默认的杀死信号，可以处理，否则进程会优雅地退出。 |
| `EXIT` | `0` | 退出陷阱在退出时触发。 |

你可以在一个陷阱中指定一个或多个信号，也可以通过调用 `– signal` 来重置陷阱的默认行为。

信号，多少种？谁能记得所有的信号？除了 `kill` 命令，没人能记得：

```
zarrelli:~$ kill -l 1) SIGHUP 2) SIGINT 3) SIGQUIT 4) SIGILL 5) SIGTRAP 6) SIGABRT 7) SIGBUS 8) SIGFPE 9) SIGKILL 10) SIGUSR1 11) SIGSEGV 12) SIGUSR2 13) SIGPIPE 14) SIGALRM 15) SIGTERM 16) SIGSTKFLT 17) SIGCHLD 18) SIGCONT 19) SIGSTOP 20) SIGTSTP 21) SIGTTIN 22) SIGTTOU 23) SIGURG 24) SIGXCPU 25) SIGXFSZ 26) SIGVTALRM 27) SIGPROF 28) SIGWINCH 29) SIGIO 30) SIGPWR 31) SIGSYS 34) SIGRTMIN 35) SIGRTMIN+1 36) SIGRTMIN+2 37) SIGRTMIN+3 38) SIGRTMIN+4 39) SIGRTMIN+5 40) SIGRTMIN+6 41) SIGRTMIN+7 
42) SIGRTMIN+8 43) SIGRTMIN+9 44) SIGRTMIN+10 45) SIGRTMIN+11 
46) SIGRTMIN+12 47) SIGRTMIN+13 48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-13 52) SIGRTMAX-12 53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9 56) SIGRTMAX-8 57) SIGRTMAX-7 58) SIGRTMAX-6 59) SIGRTMAX-5 60) SIGRTMAX-4 61) SIGRTMAX-3 62) SIGRTMAX-2 63) SIGRTMAX-1 64) SIGRTMAX 

```

现在，让我们看看如何使用陷阱进行干净的退出，通过这个小例子：

```
#!/bin/bash
x=0
while true
do
for i in {1..1000}
do
x="$i"
if (( x == 500 ))
then
echo "The value of x is: $x" >> write.log
fi
done
done

```

这个脚本有一个无限的 `while` 循环，其中嵌套着一个 `for` 循环，遍历 `1` 到 `1000` 的范围。当 `x` 的值达到 `500` 时，它会在 `write.log` 文件中打印一条消息。退出时，内部循环会重新启动，但外部结构是一个无限循环，将会一直运行下去。让我们运行它，几秒钟后按 *Ctrl*+*C*：

```
zarrelli:~$ ./write.sh ^C

```

所以，我们的脚本把终端锁定在前台运行，为了重新获得控制权，我们必须通过按 *Ctrl*+*C* 发出 `kill -15`，即 *TERM* 信号。让我们来看看目录：

```
zarrelli:~$ ls -lh total 28K -rw-r--r-- 1 zarrelli zarrelli 20 Apr 16 07:54 open -rwxr--r-- 1 zarrelli zarrelli 193 Apr 16 13:27 test.sh -rwxr--r-- 1 zarrelli zarrelli 35 Apr 16 11:54 while.sh -rw-r--r-- 1 zarrelli zarrelli 5.2K Apr 16 13:32 write.log -rwxr-xr-x 1 zarrelli zarrelli 152 Apr 16 13:05 write.sh -rwxr-xr-x 1 zarrelli zarrelli 293 Apr 16 13:28 write-term.sh

```

看起来日志被留下了：

```
zarrelli:~$ tail -5 write.log The value of x is: 500 The value of x is: 500 The value of x is: 500 The value of x is: 500 The value of x is: 500

```

是的，实际上是我们的日志，里面充满了我们设置的消息。作为日志，它被留下来也无妨，但如果这只是一个临时文件呢？每次脚本因终止或其他信号退出时，是否愿意在文件系统中留下临时文件？让我们通过创建一个清理函数来改进它：

```
clean_exit() { echo "ouch, we received a iINT signal. Outta here but first a bit of cleaning" rm write.log exit 0 }

```

一旦调用，这个函数将在 `stdout` 上回显一条有意义的信息，删除 `write.log` 文件，并以成功状态退出。最后一部分是实际的信号处理程序：

```
trap 'clean_exit' INT

```

就这些，运行脚本后稍等片刻，然后按 *Ctrl*+*C*：

```
zarrelli:~$ ./write-term.sh ^Couch, we received a INT signal. Outta here but first a bit of cleaning

```

看起来有效；让我们来看看文件系统：

```
zarrelli:~$ ls -lh ttotal 20K -rw-r--r-- 1 zarrelli zarrelli 20 Apr 16 07:54 open -rwxr--r-- 1 zarrelli zarrelli 193 Apr 16 13:27 test.sh -rwxr--r-- 1 zarrelli zarrelli 35 Apr 16 11:54 while.sh -rwxr-xr-x 1 zarrelli zarrelli 152 Apr 16 13:05 write.sh -rwxr-xr-x 1 zarrelli zarrelli 293 Apr 16 13:28 write-term.sh

```

干净，`write.log` 在退出时已被清理。这是预期并期望的行为。我们还可以更进一步，屏蔽进程不受信号影响，让它被忽略。让我们在脚本中添加以下行：

```
trap ‘' TERM

```

现在，让我们在后台执行脚本：

```
zarrelli:~$ ./write-term.sh & [1] 16831

```

好吧，既然我们要处理守护进程，我们就不必担心杀死无辜的进程：

```
zarrelli:~$ kill 16831

```

哈哈！我们杀死了你！

```
zarrelli:~$ jobs [1]+ Running ./write-term.sh &

```

嗯，我们得重新考虑一下我们的说法。看起来我们的陷阱工作得非常好。事实上，带有信号但仅用 `‘'` 作为参数的陷阱，实际上会让信号被忽略。好吧，我们还有其他的破坏方式，因为我们可以调用 `INT`：

```
zarrelli:~$ kill -INT 16831 zarrelli:~$ ouch, we received a INT signal. Outta here but first a bit of cleaning [1]+ Done ./write-term.sh

```

最后，我们以有序的方式退出了脚本，未留下任何日志：

```
zarrelli:~$ ls -lh total 16K -rw-r--r-- 1 zarrelli zarrelli 20 Apr 16 07:54 open -rwxr--r-- 1 zarrelli zarrelli 35 Apr 16 11:54 while.sh -rwxr-xr-x 1 zarrelli zarrelli 152 Apr 16 13:05 write.sh -rwxr-xr-x 1 zarrelli zarrelli 307 Apr 16 13:39 write-term.sh

```

文件系统是干净的，`write.log` 文件没有留下。现在，让我们看一下通过给脚本添加一些内容来使用陷阱的一个巧妙方法。我们先在脚本开头放入 `y=0`，然后是稍作修改的循环：

```
for i in {1..3} do if (( x == 3 ))then y="$x"
echo "The value of x is: $x" >> write.log fi trap 'echo "The value of \$y is \"${y}\""' DEBUG done

```

现在，让我们运行脚本：

```
zarrelli:~$ ./write-debug.sh The value of $y is "0"
The value of $y is "0"
The value of $y is "0"
The value of $y is "0"
The value of $y is "0"
The value of $y is "0"
The value of $y is "0"
The value of $y is "0"
The value of $y is "3"
The value of $y is "3"

```

如果在 Bash 等待命令完成时接收到信号，陷阱将在命令执行完成后才会执行。如果使用内建的 `wait`，它将在收到设置了 `trap` 的信号时立即返回，随后 `trap` 本身会被执行。请注意，`trap` 通常会以 0 的状态退出，但在这种情况下，退出状态的值将大于 128。

每次执行命令时，变量的值都会打印出来，正如我们所见，使用`-x`选项调试 Bash 脚本时：

```
zarrelli:~$ /bin/bash -x ./write-debug.sh + y=0
+ trap clean_exit INT
+ trap '' TERM
+ for i in {1..3}
+ x=1
+ (( x == 3 ))
+ trap 'echo "The value of \$y is \"${y}\""' DEBUG
+ for i in {1..3}
++ echo 'The value of $y is "0"'
The value of $y is "0"
++ echo 'The value of $y is "0"'
The value of $y is "0"
+ x=2
++ echo 'The value of $y is "0"'
The value of $y is "0"
+ (( x == 3 ))
++ echo 'The value of $y is "0"'
The value of $y is "0"
+ trap 'echo "The value of \$y is \"${y}\""' DEBUG
+ for i in {1..3}
++ echo 'The value of $y is "0"'
The value of $y is "0"
++ echo 'The value of $y is "0"'
The value of $y is "0"
+ x=3
++ echo 'The value of $y is "0"'
The value of $y is "0"
+ (( x == 3 ))
++ echo 'The value of $y is "0"'
The value of $y is "0"
+ y=3
++ echo 'The value of $y is "3"'
The value of $y is "3"
+ echo 'The value of x is: 3'
++ echo 'The value of $y is "3"'
The value of $y is "3"
+ trap 'echo "The value of \$y is \"${y}\""' DEBUG

```

移动到`trap`行，看看你能通过修改它来收集多少信息，以满足你的需求。所以，玩一会儿，享受为最终魔法触摸做准备的乐趣吧。

# 与守护进程一起走向黑暗

你认为做守护进程是一项复杂的任务吗？是的，除非你使用一个叫做*daemon*的好用工具。这个程序的任务是以简单整洁的方式将其他命令或脚本转变为守护进程。这个工具有没有采取任何捷径？没有，它只是通过我们已经看到的所有步骤，将进程从控制终端中分离出来，放到后台，启动一个新会话，清除 umask，并关闭旧的文件描述符。嗯，如果我们自己在 Bash 脚本中做这个，确实会是一项相当困难的任务。这个程序让一切变得简单明了，无需手动处理任何事情。但也有一个缺点：这不是一个标准工具，必须由用户手动安装。其实这不算大问题，因为许多发行版如 Debian 或 Red Hat 都有这个工具的安装包。

该是尝试这个工具的时候了，所以让我们把`write.sh`脚本转变为守护进程：

```
root:# daemon -r /root/write.sh

```

我们刚刚调用了 daemon 程序，传递了脚本的完整路径以及`-r`选项，这样如果脚本被停止，它会重新启动。让我们看看在我们的系统上会发生什么：

```
root:# ps -Heo tty,pid,ppid,pgid,tpgid,sess,args | grep write pts/0 2458 2298 2457 2457 2298 grep write ? 2455 1 2454 -1 2454 daemon -r /root/write.sh ? 2456 2455 2454 -1 2454 /bin/bash /root/write.sh

```

很好，我们的脚本没有控制终端；它正在后台运行，并将日志文件写入我们文件系统的根目录。现在，让我们使用`-9`选项终止它，因为没有进程可以忽略它：

```
root:# kill -9 2456

```

所以我们杀死了这个进程；让我们验证一下：

```
root:# ps -Heo tty,pid,ppid,pgid,tpgid,sess,args | grep write pts/0 2461 2298 2460 2460 2298 grep write ? 2455 1 2454 -1 2454 daemon -r /root/write.sh ? 2459 2455 2454 -1 2454 /bin/bash /root/write.sh

```

脚本就在这里。我们实际上终止了它的进程，但由于守护进程的`-r`选项，它强制重新启动了脚本；于是我们看到了，我们的守护进程正在运行，即使我们终止了它。如果我们真的想读取它，我们必须先终止守护进程程序，然后终止脚本进程：

```
root:# kill -9 2455 2459 root:# ps -Heo tty,pid,ppid,pgid,tpgid,sess,args | grep write pts/0 2481 2298 2480 2480 2298 grep write

```

这是运行守护进程的最简单方式，实际上它有很多选项。例如，假设我们希望以用户`zarrelli`身份运行脚本，并让它更改目录到一个子目录：

```
root:# daemon -D /home/zarrelli/tmp/ -u zarrelli /home/zarrelli/write.sh

```

使用`-D`，我们为`chdir`指定了一个新的目标，而`-u`为进程指定了一个新的运行用户，正如我们从`ps`命令中看到的：

```
root:# ps -Heo user,tty,pid,ppid,pgid,args | grep write zarrelli ? 2607 1 2606 daemon -D /home/zarrelli/tmp/ -u zarrelli /home/zarrelli/write.sh zarrelli ? 2608 2607 2606 /bin/bash /home/zarrelli/write.sh

```

正如预期的那样，新的日志文件属于名为`zarrelli`的用户：

```
root:# ls -lah /home/zarrelli/tmp/ total 780K drwxr-xr-x 2 zarrelli zarrelli 4.0K Apr 17 04:45 . drwxr-xr-x 3 zarrelli zarrelli 4.0K Apr 17 04:43 .. -rw-r--r-- 1 zarrelli zarrelli 769K Apr 17 04:50 write.log

```

简单，但我们可能希望有一个在系统启动时运行并在关机时停止的服务。那么，为什么不使用`systemd`来满足我们的需求呢？第一步，我们创建`/etc/systemd/system/writing.service`文件。

我们将为`systemd`管理的服务创建一个基本单元，所以让我们在文件中写入以下单元配置行：

```
[Unit] Description=Write.sh Daemon After=syslog.target [Service] ExecStart=/root/write.sh Type=simple [Install] WantedBy=default.target

```

这里没有什么特别的；根据我们要守护的脚本类型，可以选择合适的目标。如果是网络脚本，需要在网络启动后运行，因此`network.target`在这里更为合适。如果我们只想添加一些日志功能，可以使用`syslog.target`。我们也可以设置多个目标，这完全取决于我们要守护的是什么。在`[Service]`下，我们只需要指定脚本可执行文件，并且更重要的是指定执行类型：由于我们的脚本将会无限期运行，永远不会退出。因此，我们需要指定*简单*的启动方式，这样`systemd`就会执行该脚本并继续处理，而不是像*fork*模式那样等待脚本退出。其余的部分非常直接，因此我们保存文件并赋予它适当的权限：

```
root:# chmod 664 /etc/systemd/system/writing.service

```

现在，到了启用服务的时刻：

```
root:# systemctl enable writing.service

```

从`/etc/systemd/system/default.target.wants/writing.service`创建一个符号链接到`/etc/systemd/system/writing.service`。

重新加载`systemd`守护进程将有助于使新服务被识别：

```
root:# systemctl daemon-reload

```

好了，我们准备好首次执行我们的服务了：

```
root:# systemctl start writing

```

这里没有输出，但由于这个服务是由`systemd`管理的，我们可以询问它当前的状态：

```
root:# systemctl status writing writing.service - Write.sh Daemon Loaded: loaded (/etc/systemd/system/writing.service; enabled) Active: active (running) since Mon 2017-04-17 06:20:25 EDT; 57s ago Main PID: 1582 (write.sh) CGroup: /system.slice/writing.service └─1582 /bin/bash /root/write.sh Apr 17 06:20:25 spoton systemd[1]: Started Write.sh Daemon.

```

脚本正在运行；让我们做些其他检查：

```
root:# ls -lah /write.log -rw-r--r-- 1 root root 469K Apr 17 06:23 /write.log

```

`log`文件在那里，正在填充；现在让我们检查终端：

```
root:# ps -Heo user,tty,pid,ppid,pgid,args | grep write root pts/0 1605 1048 1604 grep write root ? 1582 1 1582 /bin/bash /root/write.sh

```

这里它来了，没有关联的控制终端。最后一步，当我们不想让守护进程继续运行时，我们需要停止它：

```
root:# systemctl stop writing No output so let's verify: root:# systemctl status writing writing.service - Write.sh Daemon Loaded: loaded (/etc/systemd/system/writing.service; enabled) Active: inactive (dead) since Mon 2017-04-17 06:25:51 EDT; 3s ago Process: 1582 ExecStart=/root/write.sh (code=killed, signal=TERM) Main PID: 1582 (code=killed, signal=TERM) Apr 17 06:20:25 spoton systemd[1]: Started Write.sh Daemon. Apr 17 06:25:51 spoton systemd[1]: Stopping Write.sh Daemon... Apr 17 06:25:51 spoton systemd[1]: Stopped Write.sh Daemon.

```

最后，如果我们不希望`systemd`再管理我们的守护进程，可以直接取消链接它：

```
root:# systemctl disable writing.service Removed symlink /etc/systemd/system/default.target.wants/writing.service.

```

现在，重新启动`systemd`守护进程：

```
root:# systemctl daemon-reload 

```

# 总结：

在这一章中，我们查看了如何将一个进程放到后台并使其在我们注销后继续运行，以及如何让它抵抗我们可能发送给它的大部分信号。接下来的步骤是如何将进程守护化，并利用`systemd`将其转变为一个系统管理的服务。就这样了吗？当然不是。通过一点创造力，我们可以将现有的碎片和构件拼接在一起，创建我们自己的守护进程脚本和服务，所以这可以成为一个不错的作业，尤其是在雨天的时候。

现在我们将离开守护进程部分，转向一些更相关的系统管理任务，我们将看看如何使用一些简单而强大的工具和服务来定制我们工作的环境，并且如何以最小的努力使其保持合理的安全性。
