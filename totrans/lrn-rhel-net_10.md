# 第十章：使用 SELinux 保护系统

很多时候，你会发现一些教程或博客建议你禁用 SELinux。这是为了让 vservice 的某个特性能够工作。在许多情况下，人们不得不按照博客或教程的指示操作，因为关于 SELinux 了解甚少。本章的目的是为你提供一个解决方案，帮助你更好地理解 SELinux 的工作原理。通过本章的内容，你将更了解 SELinux 提供的保护，以便下次遇到博客建议你将车钥匙留在停车场的点火器上时，你能做出更明智的决策。

在本章中，我们将涵盖以下主题：

+   什么是 SELinux

+   了解 SELinux

+   使用目标策略类型

+   SELinux 中的策略

+   SELinux 工具

+   SELinux 故障排除

# 什么是 SELinux

SELinux 是一个**MAC**（**强制访问控制**）系统，和我们熟悉的现有**DAC**（**自主访问控制**）列表共同工作，比如文件权限列表。

### 提示

SELinux 只能限制权限；它不能添加权限。如果 DAC 不允许访问，SELinux 也无法允许。

为了与标记的对象进行交互，访问是基于这些标签进行授权的，并通过策略进行控制。所有对象——如用户、进程和文件——都有标签。你所拥有的标签或（更常见的）你运行的进程，必须与资源所提供的标签相匹配。简单来说，想象一下厕所；拥有 MEN 标签的人可以进入标记为 MEN 的厕所。在 Linux 术语中，Apache web 服务器进程被标记为`httpd_t`，并可以访问具有`httpd_sys_content_t`标签的文件。通过这种方式，系统能够防止恶意或**被攻破**（被妥协）的 web 服务器，因为它能够访问的文件范围受限于 SELinux。

SELinux 由 Red Hat、NSA 和 Secure Computing 维护，因此它有着丰富的背景。它由四个主要组件组成，我们将在本章中详细探讨：

+   模式

+   标签

+   策略类型

+   策略包

为了帮助你使用 SELinux，我们将安装一些附加的包。这些 RPM 包在以下命令行中列出。为了方便布局，我们添加了换行符：

```
$ sudo yum install policycoreutils-python policycoreutils-gui \
 setools-console setools-gui setroubleshoot \
 setroubleshoot-server

```

# 了解 SELinux

让我们开始揭开 SELinux 的面纱，深入了解这些控制机制如何工作，从 SELinux 的模式开始。

## 模式

首先，我们将讨论三种可以与 SELinux 一起运行的模式。这些模式将在下图中为你展示：

![Modes](img/image00304.jpeg)

### 禁用模式

当 SELinux 被禁用时，SELinux 不会被使用，且对象不会被标记。在禁用模式下，我们完全依赖原始的 DAC。如果稍后需要启用 SELinux，启动过程会变得更长，因为所有对象需要重新标记。像这样完全禁用 SELinux 可能不是一个好主意，但如果需要，可以通过更改`/etc/selinux/config`文件中的以下行来设置：

```
SELINUX=disabled

```

这样做不是一个好主意的原因之一是，必须重启才能生效。正如前面提到的，如果稍后启用 SELinux，则需要重新标记文件。我们可以通过运行以下命令强制重新标记所有文件系统对象：

```
# fixfiles relabel

```

或者，我们可以创建`/.autorelabel`文件，如以下命令所示：

```
# touch /.autorelabel

```

### 宽松模式

如果你遇到服务问题并希望检查 SELinux 是否是可能的原因，你可以选择将 SELinux 设置为宽松模式。通过这种方式，SELinux 仍然启用，且对象保持其标签；然而，事件不会被阻止，而是被记录到`/var/log/audit/audit.log`文件中。

要进入宽松模式，我们可以在系统运行时执行此操作，而无需重启系统。以下行演示了如何实现这一点：

```
# setenforce Permissive

```

如果以这种方式进行更改，则在重新启动时，宽松模式将从`/etc/selinux/config`中应用。要将模式永久设置为宽松模式，我们应在以下行中设置宽松模式：

```
SELINUX=permissive

```

虽然我确实认为将模式设置为宽松模式作为快速和简单的测试是可以接受的，但你对 SELinux 了解得越多，就越不可能从强制模式下转移，强制模式能保证你的保护。在本章中，你将学习如何纠正问题，甚至为某个进程添加宽松模式，而不是将整个系统设置为宽松模式。

也可以通过启动加载程序设置宽松模式或强制模式，在内核行末尾添加以下命令（其中`0`表示关闭或宽松模式，`1`表示开启或强制模式）：

```
enforcing=0
enforcing=1

```

### 强制模式

强制模式与宽松模式非常相似，你可以在命令行中通过`setenforce`命令在宽松和强制模式之间切换。顾名思义，在此模式下，SELinux 被强制执行，并且也会报告到日志文件中。

要查询当前的 SELinux 模式，可以执行`getenforce`命令。如果你已安装其他工具，还可以运行`sestatus`命令，它是`policycoreutils`包的一部分。此命令显示当前模式及配置文件中的模式；`sestatus`的输出如下图所示：

![强制模式](img/image00305.jpeg)

## 标签

如前所述，当 SELinux 处于宽松或强制模式时，所有对象（如文件、用户和进程）都会有标签。在访问资源时，这些标签会被比较，以查看是否匹配兼容。

每个标签由四个冒号分隔的值组成：

+   SELinux 用户

+   SELinux 角色

+   SELinux 类型

+   SELinux 等级

通常情况下，级别仅在非常安全的政府环境中使用，其中用户的保密级别必须与文件或资源的保密级别相匹配。这里的想法是总统将能够阅读任何东西，但只能写入与其安全级别匹配的文件。这甚至阻止他写入具有较低安全级别的文件。当然，这些可以被较低授权的人员阅读，可能构成安全漏洞。

使用`ls`命令，我们可以使用`–Z`选项列出文件的标签。以下命令是列出`/etc/hosts`文件中 SELinux 标签的示例：

```
$ ls –Z /etc/hosts

```

输出应该类似于以下截图：

![Labels](img/image00306.jpeg)

阅读标签后，我们可以确定从前述截图中从左到右读取以下数值：

| SELinux 用户 | `system_u` |
| --- | --- |
| SELinux 角色 | `object_r` |
| SELinux 类型 | `net_conf_t` |
| SELinux 等级 | `s0` |

从 Linux 用户的角度来看，我们可以使用`id –Z <username>`命令来读取标签。下面的截图显示了当前登录用户的情况，其中`<username>`字段可以留空：

![Labels](img/image00307.jpeg)

同样，我们可以使用`ps`命令和`–Z`选项检查进程的标签，如下命令所示：

```
$ ps -eZ | grep ssh

```

![Labels](img/image00308.jpeg)

## 策略类型

默认的 SELinux 策略类型是有针对性的，但列出了三种策略类型如下：

+   最小

+   有针对性的

+   MLS

所有这些都包含在与`selinux-policy-minimum`、`selinux-policy-targeted`和`selinux-policy-mls`名称匹配的软件包中。

### 最小

正如名称所示，这是为 SELinux 设计的最小配置。听起来可能很奇怪，但这是针对只想针对一个服务（例如 Apache Web 服务器）的情况。从基础开始，可以轻松地将其他策略包含在您的`Minimum`类型中。以下命令显示了如何使用`semodule`添加 Apache 策略：

```
# semodule -i /usr/share/selinux/minimum/Apache.pp.bz2

```

要配置 SELinux 使用`Minimum`策略，我们使用`/etc/selinux/config`文件设置`SELINUXTYPE`指令：

```
SELINUXTYPE=minimum

```

### 有针对性的

这是默认的策略类型；默认情况下包含许多策略。在演示系统上，除了基本策略外，安装了 395 个策略，我们可以使用`semodule`列出所有模块：

```
# semodule -l

```

### MLS

多级安全或 MLS 策略类型将允许您添加额外的安全级别。可以通过标签查询这些级别，以帮助您控制对资源的访问。这通常仅在高安全性部署中使用。在 MLS 外，标签的级别元素不被使用。要启用 MLS，需要在`/etc/selinux/config`文件中配置以下指令：

```
SELINUXTYPE=mls

```

## 策略

一旦策略被安装，单个策略将安装在适当的策略类型目录中；对于默认的 targeted 策略，进入`/etc/selinux/targeted/modules/active/modules/`。策略文件具有`.pp`后缀。

# 使用 targeted 策略类型

默认的策略类型是 targeted。因此，大多数 SELinux 部署将使用这种策略类型。在 targeted 策略类型的情况下，用于强制执行的标签的主要属性是`type`。因此，targeted 策略类型通常被称为 TE 或`type` enforcement。下图强调了在 targeted 策略类型中标签的 type 属性的重要性：

![使用 targeted 策略类型](img/image00309.jpeg)

使用`seinfo`命令，这是`setools-console`包的一部分，我们可以显示有关当前 SELinux 环境的特定信息。让我们看一下可以使用的类型。要列出所有类型，我们将使用以下命令：

```
# seinfo -t

```

哇，真是太多了。如果我们数一数，RHEL 7 上大约有 4500 个；而 RHEL 6 上有 3500 个。这两个数字只是简单地说明了 SELinux 产品的增长及其持续的采用，但 Linux 软件开发人员。

我们还可以看到如何在带有用户属性的标签中导入 type 属性：

```
# seinfo -u

```

在这里，数字并不特别令人印象深刻；只是 8 个。这些不是 Linux 用户，而是 SELinux 用户；Linux 用户可以映射到 SELinux 用户，以帮助控制对资源的访问。要显示任何映射，我们可以使用`semanage`，如下所示：

```
# semanage login -l

```

在没有任何映射设置的情况下，我们将看到 root 被映射到`unconfined_u`，因为这是默认值。这个设置意味着所有没有特定映射的其他用户帐户将被映射到`unconfined_u` SELinux 用户，这意味着我们在 SELinux 上忽略了标签中的用户属性，因为它是未受限的。同样，让我们使用`seinfo`查看 ROLE 属性：

```
# seinfo -r

```

输出应显示 14 个角色；再说，这并不是一个很大的数字。角色属性在 targeted 策略类型中并没有被广泛使用。

## 未受限的域

`TYPE`属性通常在设置到进程时被称为`DOMAIN`；记住，我们可以使用以下进程状态命令查看正在运行的进程的 SELinux 标签：

```
$ ps -eZ

```

许多在用户空间中启动的进程也将是未受限的，可能是`TYPE`属性设置为`unconfined_t`。如果在用户空间中启动的进程通常是未受限的，我们可以说，服务，特别是面向网络的服务，将以某种方式受到强制执行，这也是 SELinux 存在的一个重要原因：保护免受网络暴露带来的攻击。并非只有`aunconfined_t`标签是未受限的，SELinux 也未限制其他标签。要显示所有未受限的类型或域，我们可以再次使用`seinfo`，并以 root 身份运行，如下所示：

```
# seinfo -aunconfined_domain_type  -x

```

`-a`选项告诉`seinfo`我们正在搜索一个属性；这个属性需要紧接在选项后面，且不应有额外的空格。`-x`属性则扩展显示所有具有该属性的`TYPE`，而不仅仅是列出该属性本身。输出结果应确认，主要是那些不会面向网络的域（如`bootloader_t`）是不受限制的。

以下截图显示了我的系统输出的开始部分。总共显示了 86 个不受限制的域；考虑到我们一开始有 4500 种类型，这个数量还不错：

![Unconfined domains](img/image00310.jpeg)

当策略生效时，默认的访问级别是拒绝的；这意味着必须在策略包中存在规则，才能允许用户、角色或类型访问资源。默认拒绝访问确保了安全性，防止某些场景未被考虑到；另一方面，这也意味着如果你的特定场景未被考虑，就需要添加访问权限。可能需要一定的管理操作来调整环境以满足你的需求；然而，一旦设置完成，你将拥有一个安全的系统，它将持续可靠运行，同时降低暴露于风险的可能性。

当然，尽管策略中的默认设置是拒绝访问资源，但这些策略中默认提供了成千上万的`允许规则`。使用`sesearch`命令，我们可以显示它们；将结果发送给`wc`命令可以统计规则的数量。以下命令演示了这一过程：

```
# sesearch --allow #display all allow rules
# sesearch --allow | grep wc -l #count the output

```

在我的系统上，默认情况下创建了超过 100,000 条规则。如果我们想更详细地查看这些规则，可以搜索`httpd_sys_content_t`字符串。许多规则带有这个标签，但如果我们只看其中一条，最简单的就是考虑带有命令 tail 的最后一条。在这里，我们可以看到，资源访问权限已授予`httpd_sys_content_t`标签的资源，以便与`ftpd_t`标签的进程交互。简单来说，FTP 服务器可以访问你的网站内容，如下命令所示：

```
# sesearch --allow | grep httpd_sys_content_t | tail -n 1 
allow ftpd_t httpd_sys_content_t : dir { getattr search open } ;

```

现在，我们对默认的目标策略类型有了更多了解，接下来让我们看看如何使用一些工具并查看 SELinux 的实际工作。

# SELinux 工具

让我们来看看 SELinux 工具。

## chcon 和 restorecon

我们可以使用的两种主要工具来帮助管理 SELinux 是`chcon`和`restorecon`。`chcon`命令帮助更改 SELinux 上下文或类型，通常是单个文件或有时是一些可以通过某种形式的通配符轻松引用的文件。`restorecon`命令可以用来重置文件或目录及其内容为默认的 SELinux 上下文。这些目录的默认设置存储在`/etc/selinux/targeted/contexts/files/file-context`文件中。

使用`grep`，我们可以搜索`httpd_sys_content_t`，在输出中，我们应该看到`/var/www`下文件的默认标签。这是我们期望找到网络服务器内容的目录：

```
# grep httpd_sys_content_t \ 
/etc/selinux/targeted/contexts/files/file_contexts

```

上述命令的输出如下：

```
/var/www(/.*)? system_u:object_r:httpd_sys_content_t:s0

```

现在，我们可以尝试通过更改`index.html`页面的 SELinux 上下文来破坏系统。我们可以使用`chcon`命令如下所示：

```
# chcon -t user_home_t /var/www/html/index.html

```

现在，如果我们通过`localhost` URL 访问该网站，应该会看到某种访问被拒绝的消息。这是因为我们已将文件的`TYPE`设置为`user_home_t`；在运行网络服务器的`httpd_t`上下文中不允许访问该类型。以下截图展示了`chcon`的使用和随后的拒绝消息：

![chcon 和 restorecon](img/image00311.jpeg)

当然，我们可以通过使用`chcon`将类型重新设置为`httpd_sys_content_t`手动修复此问题；然而，如果我们不确定正确的上下文，可以运行`restorecon`命令，如以下命令行所示：

```
# restorecon /var/www/html/index.html

```

现在访问网页应该可以正常工作。从技术上讲，我们可以通过在重启时创建`/.autorelabel`文件来重新标记整个文件系统，从而实现与`restorecon`相同的效果；如你所料，这样做有些过头，并且需要一些时间。不过，这样做的效果是对整个文件系统运行`restorecon`。

## 布尔值

还有一些简单的布尔值，我们可以根据需要开关，以帮助调优系统，使其与我们的环境匹配。在本书使用的 RHEL 7.1 系统中，有 294 个布尔值可以调整。我们可以使用简单的`getsebool`命令来显示它们：

```
# getsebool -a

```

我们将进一步深入，并列出与`httpd`进程相关的那些。我们可以在以下截图中看到这一点：

![布尔值](img/image00312.jpeg)

要更改布尔值，我们可以使用`setsebool`命令，这可以是临时的或永久的修复。如果我们希望布尔值更改为永久性的，则需要使用`-P`选项。这也需要一段时间，因为活跃的策略将被写入并重新编译。

如果我们回到之前的设置，其中`index.html`页面已设置为与用户主目录相关联的上下文，我们可以使用`setsebool`来修复。如果不适合更改上下文，例如，如果我们需要暂时将用户主目录托管在网络服务器上，直到下次启动，我们可以使用以下命令：

```
# setsebool httpd_read_user_content on

```

如果我们需要将此设置为永久性，我们将使用以下命令：

```
# setsebool -P httpd_read_user_content on

```

临时设置显示在以下截图中。它还显示了成功访问网页，尽管该网页仍然具有不正确的上下文：

![布尔值](img/image00313.jpeg)

使用这些布尔值可以在解决与 SELinux 相关的问题时发挥重要作用。

# SELinux 故障排除

让我们来看看不同的 SELinux 故障排除方法。

## 日志文件

如果我们还不确定之前在遇到 web 服务器错误时问题的原因，那么我们的故障排查应始终从日志文件开始。对于 SELinux，日志文件是 `/var/log/audit/audit.log`。从 SELinux 登录的记录会标记为 **AVC**（**访问向量缓存**）。我们可以使用 `grep` 来搜索日志文件，类似于以下命令：

```
# grep AVC /var/log/audit/audit.log

```

然而，更合适的做法是使用 `ausearch` 命令。如果刚刚发生了错误，我们可以使用 `recent` 时间起始代码来帮助减少返回的结果。这是一个用于显示过去 10 分钟内错误的快捷方式：

```
# ausearch -m avc -ts recent

```

除此之外，我们还可以提供实际的时间、日期或两者。以下示例中，我们将使用 `16:00` 作为开始时间进行搜索。如果没有提供日期，默认使用今天的日期，如下所示：

```
# ausearch -m avc -ts 16:00

```

通过查看下面截图中的命令输出，我们可以看到进程和资源的标签不兼容：

![日志文件](img/image00314.jpeg)

## audit2allow 命令

现在，即使在检查了日志文件之后，你仍然可能不完全清楚问题的原因或可能的修复方法。为了帮助排查，你可以尝试使用 `audit2allow` 命令。如果使用 `-w` 选项，它会在输出中提供问题的解释以及可能的解决方案。我们仍然需要查看日志，但这次我们会将输出通过管道传递给 `audit2allow` 命令，如下所示：

```
# ausearch -m avc -ts 16:00 | audit2allow -w

```

当我们重置原始布尔值时，测试系统的输出如下：

![audit2allow 命令](img/image00315.jpeg)

我们可以看到，建议与我们之前展示的布尔值设置相符。如果问题比更改布尔值更复杂，我们可以使用 `-M` 选项创建一个新的策略包。然后使用 `semodule` 导入 `.pp` 文件，如下所示：

```
# ausearch -m avc -ts 16:00 | audit2allow -M web.local
# semodule -i web.local.pp

```

## 宽容域

我们可以看到，有一些强大的工具设计来帮助我们部署 SELinux，但如果一切失败，还有一个叫做 **宽容域** 的选项。

与其将 SELinux 模式设置为宽容模式，我们可以仅为单个域或进程上下文启用宽容状态。默认情况下，宽容域是启用的，这些被称为内置宽容域。我们添加的域则是自定义域。

虽然 web 服务器是一个主要的面向网络的攻击向量，也许如果我们无法使 SELinux 与 `httpd` 配合使用，但我们又不想冒险让整个系统禁用 SELinux。我们可以通过以下命令为 `httpd_t` 启用宽容行为：

```
# semanage permissive -a httpd_t

```

如果我们以后需要删除这个行为，我们可以使用以下命令来恢复：

```
# semanage permissive -d httpd_t

```

在这两种情况下，我们都将写入活动策略，这需要一点时间。

# 总结

在本章中，你学习了如何管理 SELinux。我真诚地希望你对涉及的机制有了更深入的理解。SELinux 的目标是保护系统，特别是在涉及面向网络的服务时。禁用 SELinux 或设置为 Permissive 模式，通常不是正确的做法。通过这一章的学习，你现在应该能够选择正确的解决方案。

在下一章，我们将探讨 RHEL 7 中包含的新防火墙机制，以及相对于过去使用的标准`IPtables`机制所做的改进。同样，我们希望能够说服你认识到`firewalld`的优势，并保持该服务启用。
