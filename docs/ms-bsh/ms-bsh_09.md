# 第九章：子 Shell、信号与作业控制

到目前为止，我们看到的内容都相当直接。我们启动了一个脚本，执行了一些命令、实例、变量，并从中做出了些事情，仅此而已——一条命令接着一条命令，一条指令堆叠在上一条指令上。这就是我们所说的串行执行，一条命令接着另一条，就像多米诺骨牌：第一条先到，第一条先被处理；这让人联想到 FIFO 队列的概念，**先进先出**（First In First Out）。

如果我们想同时处理多个指令怎么办？好吧，我们不能这么做，这也不会是错的：CPU 是一个串行设备，它一次只能处理一条指令。我们用来给我们带来*多任务处理的感觉*的，是让 CPU 在指令之间快速切换。所以，CPU 不会在完全处理完一条指令后再转到另一条，而是会先处理一点第一条指令，再处理一点第二条，然后再回到第一条指令，继续处理一段时间。正是这种来回切换的方式让我们产生了 CPU 同时处理多项任务的错觉。接着，我们有了多处理器系统，我们可以利用这种架构，将进程分配到不同的 CPU 上，从而实现真正的并行处理。

无论我们决定做什么，一切都始于一个调用，一次对主 shell 新实例的信任跃迁，从而诞生了一个新的子 shell，并且它将致力于我们的任务。那么，首先，什么是子 shell？

# 什么是子 shell？

让我们从一个更基础的问题开始。什么是 shell？简单来说，shell 是用户与底层操作系统之间的接口。它可以是命令行解释器或图形界面，但 shell 的目的是充当用户与系统核心之间的中介，使前者能够访问后者提供的服务。例如，bash shell 通过终端为我们提供了命令行接口，通过一系列命令让我们与操作系统进行交互，利用其服务来执行任务。

在一个 shell 中，每条命令通常是在前一条命令完成任务之后执行的，但我们可以在一定程度上通过利用一些关键概念来改变这种行为：后台进程、信号和子 shell。

# 后台进程

让我们从对后台进程的直观定义开始，将其定义为与其启动的终端没有交互的进程。这实际上意味着后台进程与用户没有交互，仅此而已。从技术上讲，我们可以说后台进程是其进程 ID 组与其启动终端的进程 ID 组不同的进程。我们可以将进程组定义为一组具有相同进程组 ID 的进程，该 ID 是一个允许系统整体管理所有进程的标识符。进程组 ID 由组的第一个进程（也称为进程组长）的进程 ID 确定；组中的每个后续进程将从组长的进程 ID 中获取进程组 ID；每个子进程都放置在其父进程的进程组 ID 中。类似地，会话是一组进程组的集合；会话中的第一个进程也是会话领导者，它是唯一允许控制终端（如果有的话）的进程。因此，为用户准备登录会话的进程也是所有“用户会话”期间生成的进程的会话领导者，并且所有进程都将处于会话下的进程组中。当用户会话关闭时，内核会向保持终端前台进程组的会话领导者发送**挂起信号**（**SIGHUP**）。这是因为当用户关闭其交互式会话时，与终端的连接关闭，前台进程将无法再访问终端，因此它们必须被终止。话虽如此，如果后台进程尝试从终端读取，它将被禁止，并且在尝试向终端写入时会因**终端输入信号**（**SIGTTIN**）和**终端输出信号**（**SIGTTOU**）而受到阻止。

# 信号

在计算机早期，信号是处理异常事件的一种方式，通常用于将条件重置为默认状态。如今，借助诸如作业控制之类的设施，信号被用来实际指导进程执行操作，更多地成为进程间的通信设施，而非最初设想的重置机制。每个信号都与接收该信号的进程必须执行的操作相关联，以下是内核可以发送给进程的一些较为有趣的信号的简要列表：

+   `SIGCHLD`: 当子进程终止或停止时，会向父进程发送此信号。

+   `SIGCONT`: 这告诉已被`SIGSTOP`或`SGSTP`暂停的进程恢复其执行。这三个信号用于作业控制。

+   `SIGHUP`：当进程的终端关闭时，会发送此信号并终止进程。它得名于早期使用串行线路连接时的时代，那时候连接因线路掉线而挂起。如果发送给守护进程，通常会强制它们重新加载配置文件并重新打开日志文件。

+   `SIGINT`：当用户按下*Ctrl*+*C*时，会向进程发送此信号，打断并终止该进程。该信号可以被进程忽略。

+   `SIGKILL`：此信号立即终止一个进程。该信号不能被忽略，进程必须立即终止，且不会关闭或保存任何内容。（kill -9）

+   `SIGQUIT`：当进程接收到此信号时，它会退出并生成核心转储。核心转储是进程使用的内存的复制品，我们可以在其中找到很多有用的信息，比如处理器、寄存器、标志、数据等，这些信息对于调试进程的工作状态非常有用。

+   `SIGSTOP`：这个信号用于停止一个进程。它无法被忽略。

+   `SIGTERM`：这是一个终止请求。这是杀死进程的首选方法，因为它允许进程正常关闭，释放资源并保存状态，同时有序地终止所有子进程。进程可以忽略此信号。（kill -15）

+   `SIGTRAP`：这是当异常或陷阱发生时发送给进程的信号。我们已经略微了解了陷阱，接下来我们会进一步探讨它们。

+   `SIGTSTP`：这是一个交互式停止信号，用户按下*Ctrl*+*Z*时可以发送该信号。进程可以忽略此信号。进程会在当前状态下暂停。

+   `SIGTTIN`：当一个后台进程尝试从终端读取数据时，会发送此信号。

+   `SIGTTOU`：当一个后台进程尝试写入终端时，会发送此信号。

+   `SIGSEV`：当进程发生段错误时，会发送此信号。段错误发生在进程尝试访问它没有权限访问的内存位置时。

所以，我们有了信号、进程组和会话，这引出了 Unix 作业控制。那它是什么呢？在 Unix 中，我们可以控制我们所说的作业，而且我们已经很熟悉这些，因为这是进程组的另一个术语。作业不是一个进程，它是一个进程组。那么，控制是什么意思呢？简单来说，我们可以挂起、恢复或终止一个作业，并向它发送信号。

当在终端上启动一个 shell 会话时，它的进程组将获得访问终端的权限，并成为该终端的前台进程组。这意味着，属于前台组的进程可以从终端读取和写入数据，而其他进程组的进程则无法访问终端，如果它们尝试访问，进程会被停止。所以，从 shell 中，你可以与终端交互并执行不同的操作，例如，检索进程列表及其作业 ID：

```
zarrelli:~$ ps -fj | awk '{print $2 " -> " $4 " -> " $10 }'
PID -> PGID -> CMD
1422 -> 1422 -> /bin/bash
7886 -> 7886 -> ps
7887 -> 7886 -> awk

```

如我们所见，`ps`和`awk`进程有相同的进程组 ID，它是组内第一个命令`ps`的进程 ID。那么，作业控制怎么样呢？让我们看看如何在后台启动一个进程：

```
zarrelli:~$ sleep 10 &
[1] 8163

```

sleep 命令只是等待我们指定的秒数作为参数，但关键在于&符号；它会将进程放入后台。我们得到的返回值是作业号`[1]`和进程 ID；`ps`会显示更多的详细信息：

```
zarrelli:~$ ps -jf
UID PID PPID PGID SID C STIME TTY TIME CMD
zarrelli 1422 1281 1422 1422 0 08:46 pts/0 00:00:00 
/bin/bash
zarrelli 8163 1422 8163 1422 0 10:25 pts/0 00:00:00 sleep 
10
zarrelli 8166 1422 8166 1422 0 10:25 pts/0 00:00:00 ps -jf

```

现在，让我们来看一下这个：

```
zarrelli:~$ (sleep 100 &) ; sleep 20 &
[1] 8632
zarrelli:~$:~$ ps -jf
UID PID PPID PGID SID C STIME TTY TIME CMD
zarrelli 1422 1281 1422 1422 0 08:46 pts/0 00:00:00 
/bin/bash
zarrelli 8631 1 8630 1422 0 10:39 pts/0 00:00:00 sleep 
100
zarrelli 8632 1422 8632 1422 0 10:39 pts/0 00:00:00 sleep 
20
zarrelli 8637 1422 8637 1422 0 10:40 pts/0 00:00:00 ps -jf

```

在这里，我们将一个 sleep 进程放入后台，但使用了`()`将其在子 shell 中执行，实际上它是在前台执行的；不过现在，主 shell 并未报告任何作业或进程 ID，因为子 shell 无法将任何信息报告给父 shell。我们收到的唯一作业信息是父 shell 中执行的第二个 sleep 指令，且有趣的是，这两个 sleep 进程有相同的组 ID。

# 作业控制

所以，我们有作业 ID、进程 ID、前台和后台进程，但我们如何控制这些作业呢？我们有一堆可用的命令，来看看如何使用它们：

+   **`kill`：** 我们可以将作业 ID 传递给这个命令，它将向属于该作业的所有进程发送`SIGTERM`信号：

```
zarrelli:~$ sleep 100 &
[1] 9909
zarrelli:~$ kill %1
zarrelli:~$ 
[1]+ Terminated sleep 100

```

你也可以传递一个特定的信号来发送给进程。例如，`kill -15`会优雅地终止进程，发送`SIGTERM`信号；如果进程拒绝终止，`kill -9`将发送`SIGKILL`，立即终止进程。

我们可以向进程发送哪些信号？无论是`kill -l`还是`cat /usr/include/asm-generic/signal.h`都可以给出所有支持的信号列表。

+   **`killall`：** 如果我们知道进程的名称，杀死它最简单的方法就是使用`killall`命令，后跟进程名称：

```
zarrelli:~$ sleep 100 & 
[1] 10595
zarrelli:~$ killall sleep
[1]+ Terminated sleep 10

```

但`killall`有一个有趣的用法。让我们运行 sleep 命令四次，每次使用不同的参数：

```
zarrelli:~$ sleep 100 &
[1] 10672
zarrelli:~$ sleep 200 &
[2] 10689
zarrelli:~$ sleep 300 &
[3] 10690
zarrelli:~$ sleep 400 &
[4] 10693

```

现在，让我们检查进程列表：

```
zarrelli:~$ ps -jf
UID PID PPID PGID SID C STIME TTY TIME CMD
zarrelli 1422 1281 1422 1422 0 08:46 pts/0 00:00:00 
/bin/bash
zarrelli 10672 1422 10672 1422 0 11:16 pts/0 00:00:00 sleep 
100
zarrelli 10689 1422 10689 1422 0 11:16 pts/0 00:00:00 sleep 
200
zarrelli 10690 1422 10690 1422 0 11:16 pts/0 00:00:00 sleep 
300
zarrelli 10693 1422 10693 1422 0 11:16 pts/0 00:00:00 sleep 
400
zarrelli 10699 1422 10699 1422 0 11:16 pts/0 00:00:00 ps -jf

```

我们可以看到四个进程：相同的名称和不同的参数。现在，让我们使用`killall`并给出进程名称 sleep 作为它的参数：

```
zarrelli:~$ killall sleep
[1] Terminated sleep 100
[2] Terminated sleep 200
[4]+ Terminated sleep 400
[3]+ Terminated sleep 300

```

所有进程都已一次性被终止。相当方便，不是吗？让我们做最后的检查：

```
zarrelli:~$ ps -jf
UID PID PPID PGID SID C STIME TTY TIME CMD
zarrelli 1422 1281 1422 1422 0 08:46 pts/0 00:00:00 
/bin/bash
zarrelli 10709 1422 10709 1422 0 11:16 pts/0 00:00:00 ps -jf

```

现在没有更多的 sleep 实例在运行；我们通过一次运行`killall`将所有进程都杀死了。

+   **`jobs`：** 这显示在后台运行的进程及其作业 ID：

```
zarrelli:~$ sleep 100 &
[1] 8892
zarrelli:~$ sleep 200 &
[2] 8893
zarrelli:~$ jobs
[1]-  Running sleep                 100 &
[2]+  Running sleep                 200 &

```

+   **`fg`：** 这将后台运行的作业发送到前台。它接受作业 ID 作为参数。如果没有提供作业 ID，则会影响当前作业：

```
zarrelli@moveaway:~$ sleep 100 &
[1] 9045
zarrelli@moveaway:~$ fg %1
sleep 100

```

+   **`bg`：** 这将前台作业发送到后台。如果没有提供作业 ID，则会影响当前作业。

+   **`suspend`：** 这会挂起 shell，直到接收到`SIGCONT`信号。

+   **`logout`：** 这将从登录 shell 注销。

+   **`disown`：** 这将从 shell 的活动作业表中移除一个作业。

+   **`wait`:**  这个有趣的命令会暂停脚本执行，直到所有后台作业结束，或者如果作为参数传入作业 ID 或 PID，它会等到该作业或 PID 结束，并返回它等待的进程的退出状态。

| **作业 ID** | **含义** | **示例** |
| --- | --- | --- |
| `%n` | 作业编号 | Kill %1 |
| `%s` | 命令执行开始的字符串 | sleep 200 &[1] 9486kill %sl[1]+ 已终止 sleep 200 |
| `%?s` | 命令执行包含的字符串 | sleep 200 &[1] 9504kill %?ee[1]+ 已终止 sleep 200 |
| `%%` | 最近被停止的前台作业或开始的后台作业 | sleep 200 &[1] 9536kill %%[1]+ 已终止 sleep 200 |
| `%+` | 最近被停止的前台作业或开始的后台作业 | sleep 200 &[1] 9618fg %+sleep 200 |
| `%-` | 上一个任务 | sleep 200 &[1] 9626kill %-[1]+ 已终止 sleep 200 |
| `$!` | 最近的后台进程 | sleep 200 &[1] 9646sleep 300 &[2] 9647sleep 400 &[3] 9648kill $![3]+ 已终止 sleep 400 |

+   **`times`:**  我们在本书开篇时已经看过这个命令。它为我们提供执行命令期间所用时间的统计数据。

+   **`builtin`:**  这会执行一个内建命令，禁用与内建命令同名的功能和非内建命令。

+   **`command`:**  这会禁用指定命令的所有别名和功能：

```
zarrelli@moveaway:~$ ls
Desktop Documents Downloads First session Music Pictures 
progetti Projects Public Templates tmp Videos
[1]- Done sleep 200
[2]+ Done sleep 300
zarrelli:~$ ls
Desktop Documents Downloads First session Music Pictures 
progetti Projects Public Templates tmp Videos
zarrelli:~$ alias ls="ps -jf"
zarrelli:~$ ls
UID PID PPID PGID SID C STIME TTY TIME CMD
zarrelli 1373 1267 1373 1373 0 07:36 pts/0 00:00:00 
/bin/bash
zarrelli 10738 1373 10738 1373 0 10:17 pts/0 00:00:00 ps -jf
zarrelli:~$ command ls
Desktop Documents Downloads First session Music Pictures 
progetti Projects Public Templates tmp Videos
zarrelli@moveaway:~$ ls
UID PID PPID PGID SID C STIME TTY TIME CMD
zarrelli 1373 1267 1373 1373 0 07:36 pts/0 00:00:00 /bin/bash
zarrelli 10742 1373 10742 1373 0 10:17 pts/0 00:00:00 ps -jf

```

+   **`enable`:**  这个命令启用或禁用 `-n` 内建命令，所以如果我们有一个内建命令和一个外部命令，当调用时，内建命令会被忽略，外部命令会被执行。指定 `-a` 选项会显示所有内建命令及其状态列表，而 `-f` 选项会将内建命令作为共享库模块从编译的对象文件中加载。

+   **`Autoload`:**  这个在 Bash 中默认没有启用，必须通过启用 `-f` 来加载。它将一个名字标记为函数名，而不是内建命令或外部命令的引用。该命名的函数必须存放在外部文件中，并将从那里加载。

所以，我们已经看过前台和后台进程以及作业控制命令；现在，我们可以看到如何使用子 shell 以及它们为脚本带来的好处。

# 子 shell 与并行处理

我们在本书的前几章已经稍微讨论过子 shell；它们可以被定义为主 shell 的子进程。所以，子 shell 是命令解释器内的命令解释器。什么时候会发生这种情况呢？通常，当我们运行一个脚本时，它会启动自己的 shell 并从那里执行所有列出的命令；但请注意一个细节：外部命令，除非使用 `exec` 调用，否则会启动一个子进程，但内建命令则不会。这也是为什么内建命令的执行时间比对应的外部命令执行时间要快的原因，正如我们在本书的前面几页所看到的。

好吧，子 shell 有什么用呢？让我们看一个简单的例子，这样一切都会变得更容易理解：

```
#!/bin/bash
echo "This is the main subshell"
(echo "And this is the second" ; for i in {1..10} ; do echo $i ; 
done)

```

没什么特别的。我们在脚本生成的第一个子壳程序中进行回显，然后从子壳程序内部打开一个子壳程序，并使用从`1`到`10`的范围回显变量$i：

```
zarrelli:~$ ./subshell.sh 
This is the main subshell
And this is the second
1
2
3
4
5
6
7
8
9
10

```

正如我刚才说的，这个脚本没有什么特别的地方，唯一特殊的是我们用`(command_1; command_2; command_n)`这种方式调用了一个子壳程序。

括号中的内容会在一个新的子壳程序中执行，该子壳程序与父壳程序隔离；因为发生在子壳程序中的任何事情仅对该环境局部有效：

```
#!/bin/bash
a=10
echo "The value of a in the main subshell is $a"
(echo "The value of a in the child subshell is $a"; echo "...but 
now it changes"...; a=20; echo "and now a is $a")
echo "But coming back to the main subshell, the value of a has not 
been altered here since the subshell variables are local, a: $a"

```

现在，让我们运行这段代码：

```
zarrelli:~$ ./local.sh 
The value of a in the main subshell is 10
The value of a in the child subshell is 10
...but now it changes...
and now a is 20
But coming back to the main subshell, the value of a has not been altered here since the subshell variables are local, a: 10

```

正如我们从这个例子中看到的，这是从父壳程序到子壳程序的单向继承，没有任何东西可以爬回父壳程序。但我们可以在子壳程序内部生成子壳程序，所以可以有一个嵌套结构，这很不错；但是我们可能会失去对当前所处位置的追踪。最好有一个像`$BASH_SUBSHELL`这样的方便变量可用：

```
#!/bin/bash
(
echo "Bash nesting level: $BASH_SUBSHELL. Shell PID: $BASHPID"
(
echo "Bash nesting level: $BASH_SUBSHELL. Shell PID: $BASHPID"
(
echo "Bash nesting level: $BASH_SUBSHELL. Shell PID: $BASHPID"
)
)
)

```

首先，我们以这种复杂的方式编写代码只是为了突出显示壳程序的嵌套结构；在生产脚本中，我们可以使用更简洁的符号表示法。请注意这两个变量：

+   `$BASH_SUBSHELL`：这个内部变量从 Bash 版本 3 开始可用，表示子壳程序的级别。

+   `$BASHPID`：表示壳程序实例的进程 ID。

让我们运行这个脚本，看看输出：

```
zarrelli:~$ ./nesting.sh
Bash nesting level: 1\. Shell PID: 19787
Bash nesting level: 2\. Shell PID: 19788
Bash nesting level: 3\. Shell PID: 19789

```

好的，我们有了子壳程序级别的清晰输出，显示了每个壳程序实例的 PID，这表明它们实际上是由每个父壳程序生成的不同进程。我们可能会想使用内部的`$SHLVL`变量来跟踪壳程序级别，但不幸的是，正如下面的示例所示，它不会受到嵌套壳程序的影响：

```
echo "Bash level: $BASH_SUBSHELL - $SHLVL" ; (echo "Bash level: $BASH_SUBSHELL - $SHLVL"; (echo "Bash level: $BASH_SUBSHELL - 
$SHLVL")) 
Bash level: 0 - 1
Bash level: 1 - 1
Bash level: 2 – 1

```

很好，但当我们从嵌套的壳程序中退出时会发生什么？是时候来看另一个例子了：

```
#!/bin/bash
echo "This is the main subshell"
(
echo "This is the second level subshell";
for i in {1..10}; do if (( i==5 )); then exit; else echo $i; fi; 
done
) 
echo "Out of the second level subshell but still kicking inside 
the first level!"
for i in {1..3}
do echo $i
done

```

在这些代码行中，我们从`1`到`10`生成了一个内部子壳程序并打印到`stdout`，直到我们到达`5`：在这种情况下，我们退出子壳程序并返回到第一层。脚本会继续执行并打印剩下的三个数字吗？运行它就能揭示答案：

```
zarrelli:~$ ./exit.sh 
This is the main subshell
This is the second level subshell
1 2
3
4
Out of the second level subshell but still kicking inside the 
first level! 1
2
3

```

是的，退出调用仅影响了内部子壳程序，其余部分的脚本仍然在上层继续运行。

好的，我们看到了关于子壳程序的一些有趣的内容，但我们也可以用它们进行并行执行，那该如何实现呢？像往常一样，我们从一个脚本开始：

```
#!/bin/bash
(while true
do
  :
done)&
(for i in {1..3}
do
  echo "$i"
done)

```

首先要注意的是`&`字符，它的作用是将其后面的命令或壳程序放入后台。在这个例子中，第一个子壳程序有一个无限循环，如果我们不将其放到后台，它会阻止第二个子壳程序的生成及其内容执行。但让我们看看当我们将其放到后台时会发生什么：

```
./parallel.sh 
1
2
3

```

所以，第二个子壳程序被正确生成了，`for`循环也执行了，但第一个无限`while`循环发生了什么？

```
ps -fj
UID PID PPID PGID SID C STIME TTY TIME CMD
zarrelli 17311 1223 17311 17311 0 09:07 pts/0 00:00:01 
/bin/bash
zarrelli 21843 1 21842 17311 99 10:46 pts/0 00:00:16 
/bin/bash ./parallel.sh
zarrelli 21863 17311 21863 17311 0 10:47 pts/0 00:00:00 ps -fj

```

好的，它仍然在内存中运行。你可以使用`&`不仅仅是为子壳程序，还可以用于任何其他命令：

```
zarrelli:~$ ls &
[1] 22064
zarrelli:~$ exit.sh local.sh nesting.sh parallel.sh sub
shell.sh
[1]+ Done ls --color=auto

```

你想让你发出的命令在注销系统后仍然运行吗？只需运行以下命令：

```
nohup command &

```

它将在后台的子 shell 中运行，nohup 将捕捉到发送给所有子 shell 和进程的`SIGHUP`信号，该信号会在主 shell 终止时发送。这样，子 shell 和相关的命令将不会受到终止信号的影响，继续执行。

回到子 shell，为什么你要将整个子 shell 送到后台，而不是单独的命令或复合命令？将子 shell 看作容器：将问题分解为更简单的任务，将后者封装在子 shell 中，并让它们在后台执行，这样你就能节省时间，并使它们并行执行。

我们刚才提到 parallel，但实际上 Bash 并没有为多核架构优化命令和脚本的执行。如果我们想要更好的核心利用率，可以安装一个名为*parallel*的好工具。我们不会深入讨论这个程序，因为它与 Bash 关系不大，但它是一个值得读者探索的好工具，非常适合核心优化。

```
zarrelli:~$ parallel --number-of-cpus
1
zarrelli:~$ parallel --number-of-cores
4

```

parallel 的基本语法相当简单：

```
parallel command ::: argument_1 argument_2 argument_n

```

它类似于以下例子：

```
zarrelli:~$ parallel echo ::: 1 2 3
1
2
3

```

给出更多用`:::`分隔的参数将导致 parallel 将它们传递给命令，生成所有可能的组合：

```
zarrelli:~$ parallel echo ::: 1 2 3 ::: A B C
1 A
1 B
1 C
2 A
2 B
2 C
3 A
3 B
3 C

```

这里执行的作业数量等于可用核心的数量，但我们可以通过`-j+n`修改该值，将`n`个作业添加到核心上。使用`-j0`启动 parallel，它会尽可能执行尽量多的作业：

```
zarrelli:~$ parallel --eta --joblog sleep echo {} ::: 1 2 3 4 5 
10
Computers / CPU cores / Max jobs to run
1:local / 4 / 4Computer:jobs running/jobs completed/%of started jobs/Average seconds to complete
ETA: 0s Left: 6 AVG: 0.00s local:4/0/100%/0.0s 1
ETA: 0s Left: 5 AVG: 0.00s local:4/1/100%/0.0s 2
ETA: 0s Left: 4 AVG: 0.00s local:4/2/100%/0.0s 3
ETA: 0s Left: 3 AVG: 0.00s local:3/3/100%/0.0s 4
ETA: 0s Left: 2 AVG: 0.00s local:2/4/100%/0.0s 5
ETA: 0s Left: 1 AVG: 0.00s local:1/5/100%/0.0s 10 ETA: 0s Left: 0 AVG: 0.00s local:0/6/100%/0.0s 
zarrelli:~$ parallel -j0 --eta --joblog sleep echo {} ::: 1 2 3 
4 5 10
Computers / CPU cores / Max jobs to run
1:local / 4 / 6
Computer:jobs running/jobs completed/%of started jobs/Average se
conds to complete
ETA: 0s Left: 6 AVG: 0.00s local:6/0/100%/0.0s 1
ETA: 0s Left: 5 AVG: 0.00s local:5/1/100%/0.0s 2
ETA: 0s Left: 4 AVG: 0.00s local:4/2/100%/0.0s 3
ETA: 0s Left: 3 AVG: 0.00s local:3/3/100%/0.0s 4
ETA: 0s Left: 2 AVG: 0.00s local:2/4/100%/0.0s 5
ETA: 0s Left: 1 AVG: 0.00s local:1/5/100%/0.0s 10 ETA: 0s Left: 0 AVG: 0.00s local:0/6/100%/0.0s 

```

我们可以用 parallel 做什么呢？嗯，有很多复杂的操作，但这些留给读者去尝试和实验这个好用的工具；我相信在玩弄过程中会出现许多新想法。

# 总结

我们已经窥探了一些 Bash 的内部细节，学习了 pid 文件、会话、作业，且有很多东西可以尝试。我们还介绍了 parallel 和子 shell。这可能是那些需要一些练习的章节之一。花时间实验并尝试各种想法，以便熟练掌握作业控制和后台进程。现在我们已经了解了进程及如何管理它们，接下来我们将学习如何让它们相互通信并交换信息。是时候谈谈进程间通信（IPC）了！
