# 第二章 管理 Nginx

在一个全负载运行的 Web 服务器中，每秒钟发生成千上万的事件。显然，无法对这些事件进行微观管理，但即便是小的故障也能导致服务质量严重下降，进而影响用户体验。

为了防止这些问题的发生，专职网站管理员或站点可靠性工程师必须能够理解并正确管理后台的进程。

本章将介绍如何管理正在运行的 Nginx 实例，并讨论以下主题：

+   启动和停止 Nginx

+   重新加载和重新配置进程

+   分配工作进程

+   其他管理问题

# Nginx 连接处理架构

在你学习 Nginx 的管理过程之前，你需要了解 Nginx 如何处理连接。在全负载模式下，单个 Nginx 实例由**主进程**和**工作进程**组成，如下图所示：

![Nginx 连接处理架构](img/B04282_02_01.jpg)

主进程生成工作进程并通过发送和转发信号以及监听来自工作进程的退出通知来控制它们。工作进程在监听套接字上等待并接受传入连接。操作系统以轮询方式将传入连接分配给工作进程。

主进程负责所有启动、关闭和维护任务，诸如以下内容：

+   读取和重新读取配置文件

+   打开和重新打开日志文件

+   创建监听套接字

+   启动和重启工作进程

+   向工作进程转发信号

+   启动新的二进制文件

因此，主进程确保在环境变化和工作进程偶尔崩溃的情况下，Nginx 实例能够持续运行。

工作进程负责处理连接并接受新连接。工作进程还可以执行某些维护任务。例如，在主进程确保操作安全后，工作进程会自行重新打开日志文件。每个工作进程处理多个连接。这是通过运行事件循环来实现的，该循环通过特殊的系统调用从操作系统中获取在打开的套接字上发生的事件，并通过读取和写入活动套接字快速处理所有获取到的事件。维护连接所需的资源会在工作进程启动时分配。工作进程同时处理的最大连接数由`worker_connections`指令配置，默认值为 512。

在**集群架构**中，使用负载均衡器或另一个 Nginx 实例等专用路由设备，来平衡进入连接到一组相同的 Nginx 实例，这些实例每个都包含一个主进程和一组工作进程。如下图所示：

![Nginx 连接处理架构](img/B04282_02_02.jpg)

在此设置中，负载均衡器仅将连接路由到那些正在监听传入连接的实例。负载均衡器确保每个活动实例接收到大致相等的流量，并在某个实例出现连接问题时将流量从该实例路由出去。

由于架构差异，集群设置的管理程序与**独立实例**的管理程序略有不同。我们将在稍后讨论这些差异。

# 启动和停止 Nginx

在上一章中，你学习了如何启动 Nginx 实例。在 Ubuntu、Debian 或类似 Redhat 的系统上，你可以运行以下命令：

```
# service nginx start

```

如果没有启动脚本，你可以简单地使用以下命令运行二进制文件：

```
# sbin/nginx

```

Nginx 将读取并解析配置文件，创建 PID 文件（包含其进程 ID 的文件），打开日志文件，创建监听套接字，并启动工作进程。一旦工作进程启动，Nginx 实例就能响应传入的连接。这是运行中的 Nginx 实例在进程列表中的样子：

```
# ps -C nginx -f
UID        PID  PPID  C STIME TTY          TIME CMD
root      2324     1  0 15:30 ?        00:00:00 nginx: master process /usr/sbin/nginx
www-data  2325  2324  0 15:30 ?        00:00:00 nginx: worker process
www-data  2326  2324  0 15:30 ?        00:00:00 nginx: worker process
www-data  2327  2324  0 15:30 ?        00:00:00 nginx: worker process
www-data  2328  2324  0 15:30 ?        00:00:00 nginx: worker process

```

每个 Nginx 进程都会设置其进程标题，以便方便地反映该进程的角色。例如，在这里，你可以看到实例的主进程 ID 为`2324`，并且有四个工作进程，进程 ID 分别为`2325`、`2326`、`2327`和`2328`。注意，**父进程 ID**（**PPID**）列指向主进程。我们将在本节中进一步讨论主进程的 ID。

如果你在进程列表中找不到你的实例，或者在启动时控制台显示错误信息，说明某些因素阻止了 Nginx 的启动。以下表格列出了可能的问题及其解决方案：

| 信息 | 问题 | 解决方案 |
| --- | --- | --- |
| `[emerg] bind() to x.x.x.x:x failed (98: Address already in use)` | 监听端点冲突 | 确保`listen`指令指定的端点与其他服务不冲突 |
| `[emerg] open() "<path to file>" failed (2: No such file or directory)` | 文件路径无效 | 确保配置中的所有路径指向现有目录 |
| `[emerg] open() "<path to file>" failed (13: Permission denied)` | 权限不足 | 确保配置中的所有路径指向 Nginx 有权限访问的目录 |

要停止 Nginx，如果有启动脚本可用，你可以运行以下命令：

```
# service nginx stop

```

另外，你可以向实例的主进程发送`TERM`或`INT`信号来触发快速关闭，或者发送`QUIT`信号来触发优雅关闭，如下所示：

```
# kill -QUIT 2324

```

前述命令将触发实例的优雅关闭过程，所有进程最终会退出。在这里，我们引用了前面进程列表中主进程的进程 ID。

# 控制信号及其使用

Nginx 像其他 Unix 后台服务一样，由信号控制。信号是异步事件，它会中断进程的正常执行并激活某些功能。下表列出了 Nginx 支持的所有信号及其触发的功能：

| 信号 | 功能 |
| --- | --- |
| `TERM`, `INT` | 快速关闭 |
| `QUIT` | 优雅关闭 |
| `HUP` | 重新配置 |
| `USR1` | 日志文件重新打开 |
| `USR2` | Nginx 二进制升级 |
| `WINCH` | 优雅关闭工作进程 |

所有信号必须发送到实例的主进程。可以通过在进程列表中查找来定位实例的主进程：

```
# ps -C nginx -f
UID        PID  PPID  C STIME TTY          TIME CMD
root      4754  3201  0 11:10 ?        00:00:00 nginx: master process /usr/sbin/nginx
www-data  4755  4754  0 11:10 ?        00:00:00 nginx: worker process
www-data  4756  4754  0 11:10 ?        00:00:00 nginx: worker process
www-data  4757  4754  0 11:10 ?        00:00:00 nginx: worker process
www-data  4758  4754  0 11:10 ?        00:00:00 nginx: worker process

```

在此列表中，主进程的进程 ID 是 `4754`，并且有四个工作进程。主进程的进程 ID 也可以通过查看 PID 文件的内容获得：

```
# cat /var/run/nginx.pid
4754

```

### 提示

**注意**：`nginx.pid` 的路径在不同系统中可能有所不同。你可以使用 `/usr/sbin/nginx -V` 命令来查找确切路径。

要向实例发送信号，请使用 `kill` 命令，并将主进程的进程 ID 作为最后一个参数：

```
# kill -HUP 4754

```

另外，你也可以使用命令替换，从 PID 文件中直接获取主进程的进程 ID：

```
# kill -HUP `cat /var/run/nginx.pid`

```

你还可以使用以下命令：

```
# kill - HUP $(cat /var/run/nginx.pid)

```

前述的三个命令将触发实例的重新配置。接下来我们将讨论信号在 Nginx 中触发的各个功能。

## 快速关闭

`TERM` 和 `INT` 信号被发送到 Nginx 实例的主进程，以触发快速关闭程序。每个工作进程所拥有的所有资源，如连接、打开的文件和日志文件，都将立即关闭。之后，每个工作进程退出，主进程收到通知。一旦所有工作进程退出，主进程也会退出，关闭操作完成。

一个快速关闭显然会导致明显的服务中断。因此，它必须仅在紧急情况下或你完全确定没有人在使用你的实例时使用。

## 优雅关闭

一旦 Nginx 收到 `QUIT` 信号，它进入优雅关闭模式。Nginx 关闭监听套接字，并从此不再接受新的连接。现有的连接仍然会被服务，直到不再需要为止。因此，优雅关闭可能需要较长时间，特别是当一些连接正在进行长时间的下载或上传时。

在你向 Nginx 发送了优雅关闭信号后，你可以监控进程列表，查看哪些 Nginx 工作进程仍在运行，并跟踪关闭进程的进度：

```
# ps -C nginx -f
UID        PID  PPID  C STIME TTY          TIME CMD
root      5813  3201  0 12:07 ?        00:00:00 nginx: master process /usr/sbin/nginx
www-data  5814  5813 11 12:07 ?        00:00:01 nginx: worker process is shutting down

```

在此列表中，你可以看到在触发优雅关闭后，一个实例的状态。一个工作进程标有 `is shutting down` 标签，并且其进程标题标示该进程正在关闭。

一旦所有工作进程处理的连接都被关闭，工作进程退出，并通知主进程。一旦所有工作进程退出，主进程也会退出，关闭操作完成。

在集群或负载均衡设置中，优雅关闭是使实例停止服务的典型方式。使用优雅关闭可以确保由于服务器重新配置或维护，服务不会出现明显的中断。

在单实例中，优雅关闭只能确保现有连接不会被突然关闭。一旦单实例触发了优雅关闭，服务将立即无法为新访客提供服务。为了确保单实例的连续可用性，可以使用如重新配置、日志文件重新打开和 Nginx 二进制更新等维护程序。

## 重新配置

`HUP`信号可以用来通知 Nginx 重新读取配置文件并重新启动工作进程。此过程必须重启工作进程，因为在工作进程运行时，配置数据结构无法更改。

一旦主进程接收到`HUP`信号，它会尝试重新读取配置文件。如果配置文件能够被解析且没有错误，主进程会向所有现有的工作进程发送信号，要求它们优雅地关闭。发送信号后，主进程会使用新的配置启动新的工作进程。

与优雅关闭一样，重新配置过程可能需要很长时间才能完成。在你向 Nginx 发送重新配置信号后，可以监控你的进程列表，查看哪些旧的 Nginx 工作进程仍在运行，并跟踪重新配置的进展。

### 注意

如果在运行中的重新配置过程中触发了另一次重新配置，Nginx 会启动一组新的工作进程，即使过去两轮的工作进程尚未完成。原则上，这可能会导致过度使用进程表，因此建议在当前的重新配置过程完成后再启动新的配置过程。

这里是一个重新配置过程的示例：

```
# ps -C nginx -f
UID        PID  PPID  C STIME TTY          TIME CMD
root      5887  3201  0 12:14 ?        00:00:00 nginx: master process /usr/sbin/nginx
www-data  5888  5887  0 12:14 ?        00:00:00 nginx: worker process
www-data  5889  5887  0 12:14 ?        00:00:00 nginx: worker process
www-data  5890  5887  0 12:14 ?        00:00:00 nginx: worker process
www-data  5891  5887  0 12:14 ?        00:00:00 nginx: worker process

```

该列表显示了一个正在运行的 Nginx 实例。主进程的进程 ID 为`5887`。我们来向该实例的主进程发送一个`HUP`信号：

```
# kill -HUP 5887

```

该实例将发生以下变化：

```
# ps -C nginx -f
UID        PID  PPID  C STIME TTY          TIME CMD
root      5887  3201  0 12:14 ?        00:00:00 nginx: master process /usr/sbin/nginx
www-data  5888  5887  5 12:14 ?        00:00:07 nginx: worker process is shutting down
www-data  5889  5887  0 12:14 ?        00:00:01 nginx: worker process is shutting down
www-data  5890  5887  0 12:14 ?        00:00:00 nginx: worker process is shutting down
www-data  5891  5887  0 12:14 ?        00:00:00 nginx: worker process is shutting down
www-data  5918  5887  0 12:16 ?        00:00:00 nginx: worker process
www-data  5919  5887  0 12:16 ?        00:00:00 nginx: worker process
www-data  5920  5887  0 12:16 ?        00:00:00 nginx: worker process
www-data  5921  5887  0 12:16 ?        00:00:00 nginx: worker process

```

如你所见，旧的工作进程（进程 ID 为`5888`、`5889`、`5890`和`5891`）正在关闭。主进程已重新读取配置文件，并启动了一组新的工作进程，进程 ID 为`5918`、`5919`、`5920`和`5921`。

一段时间后，旧的工作进程将终止，实例将恢复到之前的状态：

```
# ps -C nginx -f
UID        PID  PPID  C STIME TTY          TIME CMD
root      5887  3201  0 12:14 ?        00:00:00 nginx: master process /usr/sbin/nginx
www-data  5918  5887  1 12:16 ?        00:00:01 nginx: worker process
www-data  5919  5887  3 12:16 ?        00:00:02 nginx: worker process
www-data  5920  5887  6 12:16 ?        00:00:03 nginx: worker process
www-data  5921  5887  3 12:16 ?        00:00:02 nginx: worker process

```

新的工作进程现在已加载新的配置。

## 重新打开日志文件

重新打开日志文件既简单又极其重要，对于服务器的持续运行至关重要。当通过 `USR1` 信号触发日志文件重新打开时，实例的主进程会获取已配置的日志文件列表并逐一打开它们。如果成功，它会关闭旧的日志文件，并通知工作进程重新打开日志文件。工作进程现在可以安全地重复相同的过程，之后日志输出将被重定向到新文件。之后，工作进程会关闭它们当前打开的所有旧日志文件描述符。

### 注意

在此过程中，日志文件的路径不会发生变化。Nginx 期望在触发此功能之前，旧的日志文件已经被重命名。这就是为什么当使用相同路径打开日志文件时，Nginx 会有效地创建或打开新的文件。

日志文件重新打开过程的步骤如下：

1.  日志文件通过外部工具被重命名或移动到新位置。

1.  你向 Nginx 发送 `USR1` 信号，Nginx 会关闭旧文件并打开新文件。

1.  旧文件现在已关闭，可以进行归档。

1.  新的文件现在已激活并正在使用中。

一个典型的 Nginx 日志文件管理工具是 **logrotate**。logrotate 是一个相当常见的工具，可以在许多 Linux 发行版中找到。以下是一个自动执行日志文件轮转程序的 logrotate 配置文件示例：

```
/var/log/nginx/*.log {
        daily
        missingok
        rotate 7
        compress
        delaycompress
        notifempty
        create 640 nginx adm
        sharedscripts
        postrotate
                [ -f /var/run/nginx.pid ] && kill -USR1 `cat /var/run/nginx.pid`
        endscript
}
```

上述脚本会每天轮转它能找到的 `/var/log/nginx` 文件夹中的每个日志文件。日志文件会保存直到积累了七个文件为止。`delaycompress` 选项指定日志文件在轮转后不会立即压缩，以避免 Nginx 在压缩文件时继续写入该文件。

日志文件轮转过程中的问题可能会导致数据丢失。以下是一个检查表，帮助你正确配置日志文件轮转程序：

+   确保在日志文件被移动之后再发送 `USR1` 信号。如果没有按此顺序操作，Nginx 会写入被轮转的文件而不是新的文件。

+   确保 Nginx 拥有足够的权限在日志文件夹中创建文件。如果 Nginx 无法打开新的日志文件，轮转过程将失败。

## Nginx 二进制升级

Nginx 可以在运行时更新其二进制文件。这是通过将监听的套接字传递给新的二进制文件，并通过一个特殊的环境变量继续监听它们来实现的。

如果你使用带插件的自定义二进制文件，此功能可以用于在不中断服务的情况下安全地升级二进制文件或尝试新的功能。

### 注意

对于其他 Web 服务器，这一操作需要完全停止服务器并用新的二进制文件重新启动。这样会导致服务暂时不可用。Nginx 的二进制升级功能旨在避免服务中断，并提供一个回退选项，以防新二进制文件出现问题。

要升级你的二进制文件，首先确保它与旧的二进制文件有相同的源代码配置。参考 第一章 中的 *从预构建包复制源代码配置* 部分，学习如何通过另一个二进制文件构建具有源代码配置的二进制文件。

当新的二进制文件构建完成后，重命名旧的文件，并将新的二进制文件放入其位置：

```
# mv /usr/sbin/nginx /usr/sbin/nginx.old
# mv objs/nginx /usr/sbin/nginx

```

前面的序列假设你当前的工作目录包含 Nginx 源代码树。

接下来，发送 `USR2` 信号到正在运行实例的主进程：

```
# kill -USR2 12995

```

主进程将通过添加 `.oldbin` 后缀来重命名其 PID 文件，并启动新的二进制文件，这将创建一个新的主进程。新的主进程将读取并解析配置，生成新的工作进程。现在实例看起来是这样的：

```
UID        PID  PPID  C STIME TTY          TIME CMD
root     12995     1  0 13:28 ?        00:00:00 nginx: master process /usr/sbin/nginx
www-data 12996 12995  0 13:28 ?        00:00:00 nginx: worker process
www-data 12997 12995  0 13:28 ?        00:00:00 nginx: worker process
www-data 12998 12995  0 13:28 ?        00:00:00 nginx: worker process
www-data 12999 12995  0 13:28 ?        00:00:00 nginx: worker process
root     13119 12995  0 13:30 ?        00:00:00 nginx: master process /usr/sbin/nginx
www-data 13120 13119  2 13:30 ?        00:00:00 nginx: worker process
www-data 13121 13119  0 13:30 ?        00:00:00 nginx: worker process
www-data 13122 13119  0 13:30 ?        00:00:00 nginx: worker process
www-data 13123 13119  0 13:30 ?        00:00:00 nginx: worker process

```

在前面的代码中，我们可以看到两个主进程：一个是旧的二进制文件的主进程（`12995`），另一个是新的二进制文件的主进程（`13119`）。新的主进程继承了旧的主进程的监听套接字，两个实例的工作进程都接受传入的连接。

## 优雅地关闭工作进程

为了全面测试新的二进制文件，我们需要请求旧的主进程优雅地关闭其工作进程。一旦新的二进制文件已经启动，并且新的工作进程正在运行，就使用以下命令向旧实例的主进程发送 `WINCH` 信号：

```
# kill -WINCH 12995

```

然后，只有新的实例的工作进程会接受连接。旧实例的工作进程将优雅地关闭：

```
UID        PID  PPID  C STIME TTY          TIME CMD
root     12995     1  0 13:28 ?        00:00:00 nginx: master process /usr/sbin/nginx
www-data 12996 12995  2 13:28 ?        00:00:17 nginx: worker process is shutting down
www-data 12998 12995  1 13:28 ?        00:00:13 nginx: worker process is shutting down
www-data 12999 12995  2 13:28 ?        00:00:18 nginx: worker process is shutting down
root     13119 12995  0 13:30 ?        00:00:00 nginx: master process /usr/sbin/nginx
www-data 13120 13119  2 13:30 ?        00:00:18 nginx: worker process
www-data 13121 13119  2 13:30 ?        00:00:16 nginx: worker process
www-data 13122 13119  2 13:30 ?        00:00:12 nginx: worker process
www-data 13123 13119  2 13:30 ?        00:00:15 nginx: worker process

```

最终，旧的二进制文件的工作进程将退出，只有新的二进制文件的工作进程会保留下来：

```
UID        PID  PPID  C STIME TTY          TIME CMD
root     12995     1  0 13:28 ?        00:00:00 nginx: master process /usr/sbin/nginx
root     13119 12995  0 13:30 ?        00:00:00 nginx: master process /usr/sbin/nginx
www-data 13120 13119  3 13:30 ?        00:00:20 nginx: worker process
www-data 13121 13119  3 13:30 ?        00:00:20 nginx: worker process
www-data 13122 13119  2 13:30 ?        00:00:16 nginx: worker process
www-data 13123 13119  2 13:30 ?        00:00:17 nginx: worker process

```

现在，只有新的二进制文件的工作进程在接收和处理传入的连接。

## 完成升级程序

一旦只有新的二进制文件的工作进程在运行，你有两个选择。

如果新的二进制文件工作正常，可以通过发送 `QUIT` 信号来终止旧的主进程：

```
# kill -QUIT 12995

```

旧的主进程将删除其 PID 文件，实例现在准备好进行下次升级。之后，如果你发现新的二进制文件存在问题，可以通过重复整个二进制升级过程来降级到旧的二进制文件。

如果新的二进制文件工作不正常，你可以通过发送 `HUP` 信号来重新启动旧主进程的工作进程：

```
# kill -HUP 12995

```

旧的主进程将重新启动其工作进程，而不重新读取配置文件，旧的和新的二进制文件的工作进程现在将接收传入连接：

```
# ps -C nginx -f

UID        PID  PPID  C STIME TTY          TIME CMD
root     12995     1  0 13:28 ?        00:00:00 nginx: master process /usr/sbin/nginx
root     13119 12995  0 13:30 ?        00:00:00 nginx: master process /usr/sbin/nginx
www-data 13120 13119  4 13:30 ?        00:01:25 nginx: worker process
www-data 13121 13119  4 13:30 ?        00:01:29 nginx: worker process
www-data 13122 13119  4 13:30 ?        00:01:21 nginx: worker process
www-data 13123 13119  4 13:30 ?        00:01:27 nginx: worker process
www-data 13397 12995  4 14:02 ?        00:00:00 nginx: worker process
www-data 13398 12995  0 14:02 ?        00:00:00 nginx: worker process
www-data 13399 12995  0 14:02 ?        00:00:00 nginx: worker process
www-data 13400 12995  0 14:02 ?        00:00:00 nginx: worker process

```

通过向新的主进程发送 `QUIT` 信号，可以优雅地关闭新的二进制文件的进程：

```
# kill -QUIT  13119

```

之后，你需要将旧的二进制文件恢复到原来的位置：

```
# mv /usr/sbin/nginx.old /usr/sbin/nginx

```

实例现在准备好进行下次升级。

### 注意

如果某个工作进程因为某种原因需要很长时间才能退出，你可以通过直接发送 `KILL` 信号强制它退出。

如果新二进制文件运行不正常，且需要紧急解决方案，你可以通过发送 `TERM` 信号紧急关闭新的主进程：

```
# kill -TERM  13119

```

新二进制文件的进程将立即退出。旧的主进程将收到通知，并会启动新的工作进程。旧的主进程还会将其 PID 文件移回原始位置，以便它替换掉新二进制文件的 PID 文件。之后，你需要将旧的二进制文件恢复到原来的位置：

```
# mv /usr/sbin/nginx.old /usr/sbin/nginx

```

实例现在已准备好进行进一步操作或下一个升级。

## 处理困难情况

在极为罕见的情况下，你可能会遇到困难的情况。如果工作进程在合理时间内没有关闭，可能存在问题。以下是这类问题的典型迹象：

+   一个进程在运行状态（R）下花费了太多时间，且没有关闭

+   一个进程在不可中断的休眠状态（D）下花费了太多时间，且没有关闭

+   一个进程处于休眠状态（S）且没有关闭

在这些情况下，你可以通过先向工作进程发送 `TERM` 信号来强制关闭该工作进程。如果工作进程在 30 秒内没有响应，你可以通过发送 `KILL` 信号来强制终止进程。

# 发行版特定的启动脚本

在 Ubuntu、Debian 和 RHEL 上，启动脚本会自动执行上述控制序列。通过使用启动脚本，你无需记住命令和信号名称的确切顺序。下表展示了启动脚本的使用：

| 命令 | 等同于 |
| --- | --- |
| `service nginx start` | `sbin`/`nginx` |
| `service nginx stop` | `TERM`，`等待 30 秒`，然后 `KILL` |
| `service nginx restart` | `service nginx stop` 和 `service nginx start` |
| `service nginx configtest` | `nginx -t <配置文件>` |
| `service nginx reload` | `service nginx configtest` 和 `HUP` |
| `service nginx rotate` | `USR1` |
| `service nginx upgrade` | `USR2` 和 `QUIT 给旧的主进程` |
| `service nginx status` | `显示实例的状态` |

二进制升级过程仅限于启动新的二进制文件并向旧的主进程发送信号以优雅地关闭，因此在这种情况下，你没有测试新二进制文件的选项。

# 分配工作进程

现在我们来讨论分配工作进程的建议。首先，稍微了解一下背景。Nginx 是一个异步的 Web 服务器，这意味着实际的输入/输出操作是异步进行的，和工作进程的执行是分开的。每个工作进程运行一个事件循环，通过一个特殊的系统调用获取所有需要处理的文件描述符，然后使用非阻塞的 I/O 操作处理这些文件描述符。因此，每个工作进程可以服务多个连接。

在这种情况下，事件发生在文件描述符上的时间，以及文件描述符能被服务的时间（即延迟）取决于完成整个事件处理周期的速度。因此，为了实现更高的延迟，合理的做法是让工作进程之间在 CPU 资源的竞争上受一些惩罚，转而支持每个进程更多的连接，因为这将减少工作进程之间的上下文切换次数。

因此，*在 CPU 受限的系统上*，为每个 CPU 核心分配相同数量的工作进程是有意义的。例如，考虑以下 `top` 命令的输出（可以通过在 `top` 启动后按 *1* 键来获取此输出）：

```
top - 10:52:54 up 48 min,  2 users,  load average: 0.11, 0.18, 0.27
Tasks: 273 total,   2 running, 271 sleeping,   0 stopped,   0 zombie
%Cpu0  :  1.7 us,  0.3 sy,  0.0 ni, 97.7 id,  0.3 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu1  :  0.7 us,  0.3 sy,  0.0 ni, 94.7 id,  4.0 wa,  0.0 hi,  0.3 si,  0.0 st
%Cpu2  :  1.7 us,  1.0 sy,  0.0 ni, 97.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu3  :  3.0 us,  1.0 sy,  0.0 ni, 95.0 id,  1.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu4  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu5  :  0.3 us,  0.3 sy,  0.0 ni, 99.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu6  :  0.3 us,  0.0 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
%Cpu7  :  0.0 us,  0.3 sy,  0.0 ni, 99.7 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st

```

该系统具有八个独立的 CPU 核心。因此，在该系统上，最大数量的工作进程为八个，且这些工作进程不会与 CPU 核心发生竞争。要配置 Nginx 启动指定数量的工作进程，可以在主配置文件中使用 `worker_processes` 指令：

```
worker_processes 8;
```

上述命令将指示 Nginx 启动八个工作进程以处理传入的连接。

### 注意

如果工作进程的数量设置低于 CPU 核心的数量，Nginx 将无法充分利用系统中所有可用的并行处理能力。

要扩展工作进程可以处理的最大连接数，请使用 `worker_connections` 指令：

```
events {
    worker_connections 10000;
}
```

上述命令将把可分配的最大连接数扩展到 10,000。这包括传入连接（来自客户端的连接）和传出连接（连接到代理服务器和其他外部资源）。

*在磁盘 I/O 受限的系统上*，如果没有 AIO 功能，可能会因阻塞的磁盘 I/O 操作而引入额外的延迟。当前一个工作进程在某个文件描述符上等待阻塞的磁盘 I/O 操作完成时，其他文件描述符无法被服务。然而，其他进程仍然可以使用可用的 CPU 资源。因此，在 I/O 通道数目已满的情况下，增加工作进程可能不会提高性能。

在资源需求混合的系统上，可能需要采用不同于前述两种的工作进程分配策略，以实现更好的性能。尝试调整工作进程的数量，以获得最佳的配置。这可以是一个工作进程到数百个工作进程不等。

# 设置 Nginx 以提供静态数据

现在你已经更熟练地掌握了安装、配置和管理 Nginx，我们可以继续进行一些实际问题的探讨。让我们看看如何设置 Nginx 来提供静态数据，如图像、CSS 或 JavaScript 文件。

首先，我们将使用上一章节中的示例配置，并使其支持通过通配符包含多个虚拟主机：

```
error_log  logs/error.log;

worker_processes 8;

events {
    use epoll;
    worker_connections  10000;
}

http {
    include           mime.types;
    default_type      application/octet-stream;

    include /etc/nginx/site-enabled/*.conf;
}
```

我们已将 Nginx 配置为利用八个处理器核心，并包括位于`/etc/nginx/site-enabled`目录中的所有配置文件。

接下来，我们将配置一个虚拟主机`static.example.com`用于服务静态数据。以下内容需要放入文件`/etc/nginx/site-enabled/static.example.com.conf`：

```
server {
    listen       80;
    server_name  static.example.com;

    access_log  /var/log/nginx/static.example.com-access.log  main;

    sendfile on;
    sendfile_max_chunk 1M;
    tcp_nopush on;
    gzip_static on;

    root /usr/local/www/static.example.com;
}
```

该文件配置了虚拟主机`static.example.com`。虚拟主机根目录位置设置为`/usr/local/www/static.example.com`。为了提高静态文件的获取效率，我们建议 Nginx 使用 sendfile()系统调用（`sendfile on`），并将最大 sendfile 块大小设置为 1 MB。同时，我们启用“TCP_NOPUSH”选项，以提高在使用 sendfile()时的 TCP 段利用率（`tcp_nopush on`）。

`gzip_static on`指令告诉 Nginx 检查静态文件的 gzip 压缩版本，比如`main.js.gz`对应`main.js`，`styles.css.gz`对应`styles.css`。如果找到了这些文件，Nginx 会指示存在`.gzip`内容编码，并使用压缩文件的内容，而不是原始文件。

这个配置适用于提供中小型静态文件的虚拟主机。

# 安装 SSL 证书

今天，超过 60%的互联网 HTTP 流量都通过 SSL 保护。在面对复杂的攻击（如缓存投毒和 DNS 劫持）时，如果您的网站内容具有任何价值，SSL 是必需的。

Nginx 具有高级 SSL 支持，且使配置变得简单。让我们一起走过 SSL 虚拟主机的安装过程。

在我们开始之前，确保您的系统已安装`openssl`包：

```
# apt-get install openssl

```

这将确保您拥有必要的工具，来完成 SSL 证书的颁发流程。

## 创建证书签名请求

您需要一个 SSL 证书来设置 SSL 虚拟主机。为了获得真实的证书，您需要联系认证机构申请 SSL 证书。认证机构通常会收取相关费用。

要颁发 SSL 证书，认证机构需要您提供**证书签名请求**（**CSR**）。CSR 是您创建并发送给认证机构的消息，包含您的身份信息，如区分名、地址和公钥。

要生成 CSR，请运行以下命令：

```
openssl req -new -newkey rsa:2048 -nodes -keyout your_domain_name.key -out your_domain_name.csr

```

这将开始生成两个文件的过程：一个用于解密 SSL 证书的私钥（`your_domain_name.key`），以及一个用于申请新 SSL 证书的证书签名请求（`your_domain_name.csr`）。

此命令将要求您提供身份信息：

+   **国家名称**（**C**）：这是一个两字母的国家代码，例如，NL 或 US。

+   **州或省份**（**S**）：这是您或您公司所在州的完整名称，例如，北荷兰。

+   **地区或城市**（**L**）：这是您或您公司所在的城市或城镇，例如，阿姆斯特丹。

+   **组织**（**O**）：如果您的公司或部门名称中有 *&*、*@* 或其他需要按 *Shift* 键输入的符号，您必须将该符号拼写出来或省略才能完成注册。例如，XY & Z Corporation 应为 XYZ Corporation 或 XY and Z Corporation。

+   **组织单位**（**OU**）：此字段是发起请求的部门或组织单位的名称。

+   **通用名称**（**CN**）：这是您要保护的主机的完整名称。

最后一项字段尤为重要。它必须与您要保护的主机的完整名称匹配。例如，如果您注册了域名 `example.com` 并且用户将连接到 `www.example.com`，则必须在通用名称字段中输入 `www.example.com`。如果您输入的是 `example.com`，那么该证书将无法在 `www.example.com` 上使用。

### 注意

在生成 CSR 时，请勿填写诸如电子邮件地址、挑战密码或可选的公司名称等可选属性。这些内容没有太大价值，反而会暴露更多个人数据。

您的 CSR 现在已准备好。将您的私钥保存在某个安全位置后，您可以继续联系认证机构并申请 SSL 证书。根据要求提交您的 CSR。

## 安装已签发的 SSL 证书

一旦您的证书签发完成，您可以继续设置您的 SSL 服务器。将证书保存为具有描述性的名称，例如 `your_domain_name.crt`。将其移动到一个只有 Nginx 和超级用户能够访问的安全目录。为简便起见，我们将使用 `/etc/ssl` 作为此类目录的示例。

现在，您可以开始为您的安全虚拟主机添加配置：

```
server     {
        listen 443;
  server_name your.domain.com;
  ssl on;
  ssl_certificate /etc/ssl/your_domain_name.crt;
  ssl_certificate_key /etc/ssl/your_domain_name.key;
  [… the rest of the configuration ...]
}
```

`server_name` 指令中的域名必须与您的证书签名请求中的通用名称字段的值匹配。

配置保存后，请使用以下命令重启 Nginx：

```
# service nginx restart

```

现在，导航到 `https://your.domain.com`，以建立与您服务器的安全连接。

## 永久重定向非安全虚拟主机

上述配置仅处理发往您服务器 HTTPS 服务（端口 `443`）的请求。大多数情况下，您会同时运行普通 HTTP 服务（端口 `80`）和安全的 HTTPS 服务。

出于多个原因，不建议对同一主机名的普通 HTTP 和 HTTPS 服务设置不同的配置。如果某些资源仅通过普通 HTTP 提供而不通过 SSL，或反之，这可能会导致坏链接，因为如果指向某个资源的 URL 是以不区分协议的方式处理的，可能会产生问题。

同样，如果某些资源意外地通过普通 HTTP 和 SSL 同时提供，那么这就是一个安全错误，因为只需将 `https://` 协议更改为 `http://`，就可以通过不安全的方式访问和操作该资源。

为避免这些问题并简化配置，您可以设置一个简单的永久重定向，将非 SSL 虚拟主机的请求重定向到 SSL 虚拟主机：

```
server {
    listen       80;
    server_name  your.domain.com;

    rewrite ^/(.*)$ https://your.domain.com/$1 permanent;
}
```

这确保所有通过普通 HTTP 访问你网站上的任何资源的请求都会被重定向到 SSL 虚拟主机上的相同资源。

# 管理临时文件

管理临时文件通常不是一件大事，但你必须对此有所了解。Nginx 使用临时文件来存储以下临时数据：

+   从用户接收的大请求体

+   从代理服务器或通过 FastCGI、SCGI 或 UWCGI 协议接收的大响应体。

在 第一章 *安装 Nginx* 部分中，你看到了这些文件的默认临时文件夹位置。下表列出了为各种 Nginx 核心模块指定临时文件夹的配置指令：

| 指令 | 目的 |
| --- | --- |
| `client_body_temp_path` | 为客户端请求体数据指定临时路径 |
| `proxy_temp_path` | 为代理服务器响应指定临时路径 |
| `fastcgi_temp_path` | 为 FastCGI 服务器响应指定临时路径 |
| `scgi_temp_path` | 为 SCGI 服务器响应指定临时路径 |
| `uwsgi_temp_path` | 为 UWCGI 服务器响应指定临时路径 |

前述指令的参数如下：

```
proxy_temp_path <path> [<level1> [<level2> [<level3>]]]
```

在前面的代码中，`<path>` 指定包含临时文件的目录路径，而级别则指定每个哈希目录级别中的字符数。

什么是哈希目录？在 UNIX 中，文件系统中的目录本质上是一个文件，里面仅包含该目录的条目列表。所以，假设你的临时目录包含了 100,000 个条目。每次在该目录中的搜索都会扫描这 100,000 个条目，这效率很低。为了避免这种情况，你可以将临时目录拆分为若干个子目录，每个子目录包含一个有限数量的临时文件。

通过指定级别，你可以指示 Nginx 将临时目录拆分为一组子目录，每个子目录的名称具有指定的字符数，例如，指令：

```
proxy_temp_path /var/lib/nginx/proxy 2;
```

前面的代码行指示 Nginx 将名为 `3924510929` 的临时文件存储在路径 `/var/lib/nginx/proxy/29/3924510929` 下。

同样，指令 `proxy_temp_path /var/lib/nginx/proxy 1 2` 指示 Nginx 将名为 `1673539942` 的临时文件存储在路径 `/var/lib/nginx/proxy/2/94/1673539942` 下。

如你所见，构成中介目录名称的字符是从临时文件名的尾部提取的。

无论是层次结构的还是非层次结构的临时目录结构，都必须定期清理。这可以通过遍历目录树并删除所有位于这些目录中的文件来实现。你可以使用如下命令：

```
find /var/lib/nginx/proxy -type f -regex '.+/[0-9]+$' | xargs -I '{}' rm "{}"

```

你可以在交互式 shell 中使用该命令。此命令将找到所有以数字结尾的临时目录中的文件，并通过运行 `rm` 删除这些文件。如果发现异常，它会提示删除。

对于非交互模式，你可以使用更危险的命令：

```
find /var/lib/nginx/proxy -type f -regex '.+/[0-9]+$' | xargs -I '{}' rm -f "{}"

```

此命令不会提示删除文件。

### 注意

此命令危险，因为它会盲目删除一个广泛指定的文件集。为了避免数据丢失，请在管理临时目录时遵循以下原则：

+   永远只在临时目录中存储临时文件

+   在 `find` 命令的第一个参数中始终使用绝对路径

+   如果可能，通过将 `rm` 替换为 `echo` 来检查即将删除的内容，以便打印出将传递给 `rm` 的文件列表。

+   确保 Nginx 在一个专门指定的用户（如 `nobody` 或 `www-data`）下存储临时文件，绝不在超级用户下存储

+   确保上述命令在专门指定的用户（如 `nobody` 或 `www-data`）下运行，绝不在超级用户下运行

# 向开发者反馈问题

如果你正在运行非稳定版本的 Nginx 进行试用，或使用自定义或第三方模块，你的实例可能偶尔会遇到崩溃。如果你决定将这些问题反馈给开发者，这里有一份指南，帮助你更高效地进行沟通。

开发者通常无法访问生产系统，但了解你的 Nginx 实例所运行的环境对追踪问题的根本原因至关重要。

因此，你需要提供关于问题的详细信息。崩溃的详细信息可以在崩溃后创建的核心文件中找到。

### 注意

**警告！**

核心文件包含崩溃时工作进程的内存转储，因此可能包含敏感信息，如密码、密钥或私人数据。因此，永远不要与不信任的人分享核心文件。

另请使用以下程序来获取关于崩溃的详细信息：

1.  获取你运行的带有调试信息的 Nginx 二进制文件副本（参见以下说明）

1.  如果核心文件可用，请在带有调试信息的二进制文件上运行 `gdb`：

    ```
    # gdb ./nginx-binary core
    ```

1.  如果运行成功，这将打开 `gdb` 提示符。在其中输入 `bt full`：

    ```
    (gdb) bt full
    [… produces a dump … ]

    ```

上述命令将在崩溃时生成一长串堆栈转储，通常足以调试多种问题。总结导致崩溃的配置，并将其与完整的堆栈跟踪一起发送给开发者。

## 创建带有调试信息的二进制文件

只有带有调试信息的二进制文件才能获取详细的堆栈跟踪。你不一定需要运行带有调试信息的二进制文件，只需要确保使用的二进制文件与运行的文件相同，但额外包含了调试信息。

通过在源代码树中配置一个额外的`–with-debug`选项，你可以从你正在运行的二进制文件的源代码中生成这样的二进制文件。步骤如下：

1.  首先，从你正在运行的二进制文件中获取配置脚本参数：

    ```
    $ /usr/sbin/nginx -V

    ```

1.  在参数字符串前添加`–with-debug`选项，并运行配置脚本：

    ```
    $ ./configure –with-debug --with-cc-opt='-g -O2 -fstack-protector --param=ssp-buffer-size=4 -Wformat -Werror=format-security -D_FORTIFY_SOURCE=2' --with-ld-opt='-Wl,-Bsymbolic-functions -Wl,-z,relro' …

    ```

按照构建过程的剩余步骤进行操作（详细信息请参见上一章）。完成后，你将在源代码树的`objs`目录中找到一个与正在运行的二进制文件相同，但带有调试信息的二进制文件：

```
$ file objs/nginx
objs/nginx: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked (uses shared libs), for GNU/Linux 2.6.32, BuildID[sha1]=7afba0f9be717c965a3cfaaefb6e2325bdcea676, not stripped

```

现在，你可以使用这个二进制文件从其对应的二进制文件生成的核心文件中获取完整的堆栈跟踪。

请参阅上一节，了解如何生成堆栈跟踪。

# 总结

在本章中，你学习了很多 Nginx 管理技巧。我们几乎涵盖了 Nginx 操作的所有内容，除了依赖问题的细节。在接下来的章节中，你将开始学习 Nginx 的特定功能以及如何应用它们。这将进一步丰富你的 Nginx 核心技能。
