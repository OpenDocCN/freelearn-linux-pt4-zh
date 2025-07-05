# 第九章。故障排除 VCM

本章将涵盖以下方法：

+   故障排除工具 - EcmDebugEventViewer

+   故障排除工具 - 作业管理器 | 历史记录

+   故障排除工具 - 机器收集状态

+   故障排除代理通信问题

+   故障排除代理升级问题

+   故障排除 SCR 下载问题

+   故障排除 VCM 控制台登录失败

+   故障排除 vCenter 和 vCloud 数据收集问题

+   故障排除推荐操作：调查 Linux 服务器补丁错误

+   故障排除无法在控制台上看到任何任务

+   故障排除在调度作业页面看不到每月选项

# 介绍

故障排除是识别问题来源并消除它的过程。开始故障排除时，你从最明显的可能问题入手，然后确定具体问题。做到这一点需要两样东西：故障排除对象的经验和能够查看详细信息（如日志和错误代码）的好工具。

本章将讨论这两个问题。在前三个方法中，你将学习到这些工具，因为它们是 VCM 故障排除中必要的基础，然后我们将讨论一些常见问题及其解决方案。

我们将介绍的故障排除工具包括**EcmDebugEventViewer.exe**、**VCM 日志**、**作业管理器 | 历史记录**和**SQL 日志查看器**。如果你熟悉 SQL，你也可以尝试 SQL 跟踪。

常见问题包括数据收集问题、补丁安装问题以及代理通信问题。

代理通信消息过于晦涩；它们对每个问题都显示**Pingfailed**，你需要检查到底发生了什么问题。

那么，让我们开始故障排除 VCM 吧。

# 故障排除工具 - EcmDebugEventViewer

你可以使用 EcmDebugEventViewer 工具，深入查看存储在 VCM 数据库中的所有事件。这将帮助你查找环境中的事件错误。

## 准备就绪

我们需要以管理员权限访问 VCM 操作系统。

该工具名为 EcmDebugEventViewer.exe。它是一个独立的可执行文件，位于`C:\Program Files (x86)\VMware\VCM\Tools`（如果你按默认设置安装了 VCM）。

## 如何操作...

这个方法分为两个部分：启动和过滤。

### 访问事件

该工具可以从其所在位置启动，或者你可以将`.exe`文件复制到任何位置并启动。按照以下步骤操作：

1.  以管理员权限连接到你的 VCM 操作系统。

1.  打开资源管理器窗口并浏览到`X:\Program Files (x86)\VMware\VCM\Tools`（*X*是安装 VCM 的驱动器）。

1.  右键点击**EcmDebugEventViewer.exe**并选择**以管理员身份运行**。

1.  现在，你会看到一个空白的屏幕。你可以从三种选项中选择：**数据库**、**收集器数据库**或**打开文件**以打开保存的`.dbe`文件。在本示例中，选择**数据库**。

1.  按下*F5*，它将从数据库中获取日志内容。![访问事件](img/image_09_001.jpg)

1.  双击某一行以打开更多细节。以下截图显示了一个错误消息，表示 VCM 服务器无法解析邮件服务器。![访问事件](img/image_09_002.jpg)

1.  您可以使用详情窗口中的**上**和**下**按钮查看其他日志条目。

1.  点击**关闭**以关闭事件详情。

### 过滤事件

由于条目很多，您可能想通过部署过滤器来减少数量。继续按照以下步骤操作：

1.  点击**筛选设置**，然后定义筛选条件，例如消息的来源或类型，如下图所示：![过滤事件](img/image_09_003.jpg)

1.  您还可以选择应显示的特定时间段。点击顶部菜单中的**日期/时间**并修改持续时间，如下图所示。![过滤事件](img/image_09_004.jpg)

1.  您可以通过转到**文件 | 另存为 dbe**来导出选定的日志。这将保存所有的行。

1.  要仅保存选定的行，选择这些行，然后转到**文件 | 另存为 dbe**。

这就总结了我们对 EcmDebugEventViewer 工具的概述。

## 工作原理

您可以打开主`VCM`数据库以及`VCM_Coll`数据库或任何已保存的文件。

当您排查 VCM 问题时，这是开始的第一步。始终选择主要的 VCM 数据库，因为它包含来自 VCM 代理的系统数据以及所有正在执行的进程。

让我们举个例子：

您正在尝试为一台 Linux 机器打补丁，但失败了，并且返回了 PingFailed 错误代码。您检查后发现，您可以成功 ping 通该受管 Linux 机器，并且可以 Telnet 到该端口——关于与 Linux 机器的通信一切正常。这里的小问题是，VCM 首先尝试通过端口`26542`与 SCR 服务器通信，如果由于某种原因无法访问该端口，它会返回一个含糊的 PingFailed 错误。现在，如果您启动 ECMDebug 工具，并且已将调试日志级别设置为 info，您可以看到它正在尝试与 SCR 服务器进行通信，但失败了。

进一步排查时，您发现代理由于某种原因没有进行通信。重新启动后，您可以继续进行修补操作。

下一个选项是 VCM 收集数据库。`VCM_Coll`数据库包含有关 UI 的信息，例如收集器设置和选项。坦白说，我几乎没有使用过它，因此没有实际的用例。

最后一个选项是打开一个 DBE 文件。VCM 代理也将其日志保存在`.dbe`文件格式中，因此在排查代理问题时，您可以将该文件从代理复制到服务器并读取，或者有人可以从他们的 VCM 安装中将已保存的`.dbe`文件发送给您，您可以查看该文件并提供建议。当您与 VMware 合作时，他们会要求您生成这个`.dbe`文件并将其发送给他们，以便他们可以进行审查并提供解决方案。

# 故障排除工具 – 作业管理器 | 历史

历史工具内置到 VCM 中。使用此工具，您可以查看之前完成的作业的状态，并提供日志以进一步进行故障排除。

## 准备工作

需要访问 VCM 控制台并具备基本的 VCM 理解才能在控制台中漫游。

## 如何操作...

要查看之前完成的作业的详细信息，请启动 VCM 控制台，然后转到**管理 | 作业管理器 | 历史**。

这里您有四个选项：**即时收集**、**计划收集**、**其他作业**和**VCM 远程**。如果您不是在解决收集问题，则最有可能需要转到**其他作业**并在左上窗格中选择作业，然后在底部窗格中查看详细信息。

选择**作业历史**，然后点击**查看详细信息**。如果作业涉及多台机器，您将可以选择其中之一，然后查看出了什么问题。

在以下屏幕截图中，当我们悬停在**状态**上时，我们可以看到**双向认证**存在问题。

我们可以使用顶部的**菜单**选项复制数据，然后可以用于在互联网上搜索或向 VMware 提供精确的日志以进一步进行故障排除。

![如何操作...](img/image_09_005.jpg)

VCM 作业管理器 | 历史

## 如何工作

每次 VCM 执行计划或按需作业时，都会在此位置记录；您可以查看它们，并查看它们是失败还是成功。如果它们失败，它将提供错误信息，供进一步故障排除使用。

当您点击作业并选择**查看详细信息**时，将会弹出一个窗口，您可以选择查看选定的机器或所有机器。您可以从这里重新提交作业。

让我们看看一个实际的经验。我们无法为 Linux 服务器打补丁。我们已按预期配置了 SCR 服务器和 VCM，但仍然失败。在这里检查日志后，我们发现 VCM 正在寻找一个名为`REDHAT-rt.ptoperties`的文件，而根据文档，我们已配置了一个名为`redhat-rt.properties`的文件。在更正文件名称（改为大写字母）后，我们成功地为服务器打了补丁。

# 故障排除工具 – 机器集合状态

在 VCM 控制台中的**机器集合状态**，您可以查看从哪个服务器收集了哪些数据，以及它是完整集合还是增量集合。在解决数据收集问题时，这可能会派上用场。

## 准备工作

需要访问 VCM 控制台并具备基本的 VCM 理解才能在控制台中漫游。

## 如何操作...

转到**管理 | 机器管理器 | 机器集合状态**。

正如提到的，我们可以查看收集了哪些数据，使用了哪个过滤器集以及哪些过滤器成功获取了数据。

假设您正在创建一个带有特定动态规则的机器组，并且机器没有被填充，可能是因为您用于筛选的数据类型没有被收集。

您正在运行合规性报告，发现进行合规性测试时显示**数据不可用**；如前所述，可能是数据未被收集。

您可以通过在控制台应用多个筛选器来查看详细信息，如果您的基础设施很大，这将是一个很大的表格。

![如何操作...](img/image_09_006.jpg)

VCM 下的机器收集日志位于管理部分

## 工作原理

当我们从受管机器收集数据时，我们收集的数据以及收集时间都会被记录。

我们正在与一台 RHEL 7 机器一起收集数据。作业成功，但当我们尝试修补服务器时，它没有显示状态，即是否需要修补。当我们在机器收集状态工具中检查时，未找到我们的机器，这意味着详细信息没有被捕获。检查日志后，我们得出结论：VCM 收集器无法捕获详细信息。在解决机器问题后，我们进行了另一次收集操作，发现详细信息可用，并且机器出现在机器收集状态中。

# 排查代理通信问题

现在，我们将查看代理通信中遇到的多个问题及其解决方法。

## 准备就绪

在本指南中，我们将查看以下问题：

+   *问题 01：相互认证失败*

+   *问题 02：重新抛出的证书问题*

+   *问题 03：Ping 失败*

+   *问题 04：凭证失败*

### 问题 01：相互认证失败

升级后可能会发生相互认证失败的情况。当您尝试收集数据时，它会失败，并显示如下截图中的错误：

![问题 01：相互认证失败](img/image_09_007.jpg)

VCM 代理通信问题：**相互认证失败**

您可以在**管理**|**作业管理器**|**即时收集**下看到错误。

然后，选择适当的子部分。之后，选择左上角的**作业**，您将在左下角的窗格中看到详细信息。

## 如何操作...

要解决此问题，请按照以下步骤操作：

1.  选择无法收集数据的机器。

1.  点击**重新建立相互认证**（菜单栏中的握手符号）。

1.  选择遇到问题的机器。

1.  点击**完成**关闭向导。

1.  通过再次执行收集操作来确认问题是否已解决。

### 问题 02：重新抛出的证书问题

代理-服务器通信基于我们在安装代理时使用的证书；如果代理上的证书与服务器上的证书不匹配，则会出现以下错误信息：

![问题 02：重新抛出的证书问题](img/image_09_008.jpg)

VCM 代理通信：重新抛出的证书问题

## 如何操作...

要解决此问题，请执行以下操作：

1.  使用正确的证书重新安装代理，并重新执行数据收集。

1.  如果是 Linux 服务器，使用来自正确 VCM 服务器的安装程序，因为证书是嵌入在安装程序中的。

### 问题 03: PingFailed

这个问题可能由多种原因引起：

+   管理的机器无法进行 DNS 解析

+   防火墙正在阻止端口 26542（可能是代理上，或者是服务器和代理之间的防火墙）

+   服务未运行

+   代理未安装

+   机器已关闭

+   存在路由问题，因此机器 IP 无法访问

![问题 03: PingFailed](img/image_09_009.jpg)

VCM 代理通信问题：PingFailed

## 如何操作...

要解决此问题，请按以下步骤操作：

1.  确保管理机器已开机。

1.  使用主机名或 FQDN ping 管理机器，检查是否能解析出 IP 地址。

    使用以下命令：

    ```
     ping -a 10.20.30.40 
     ping server01.study.local

    ```

    确保后两个主机名解析到相同的 IP 地址，并且第一个命令返回正确的主机名。已经观察到有时名称解析未按预期工作，需要进行修正。

1.  跟踪从 VCM 服务器到目标 IP，检查是否可以从 VCM 服务器访问目标机器，且没有路由问题；如果有，添加所需的路由。

    使用以下命令执行此操作：

    ```
     tracert server01.study.local

    ```

    从管理服务器上运行以下命令：

    ```
     tracert vcm.study.local

    ```

    您必须能从两端进行访问。

1.  从 VCM 服务器检查 Telnet 端口 26542；如果不可用，检查管理机器上的本地防火墙。如果防火墙已关闭，也请与网络和安全团队确认，是否存在防火墙在管理机器与 VCM 服务器之间，且是否已打开端口 26542。

    从 VCM 服务器，尝试执行以下命令：

    ```
     telnet server01.study.local 26542

    ```

    有关所需端口的详细信息，请查看 第一章，*安装 VCM* 部分的 *理解 VCM 的要求*。

1.  检查本地服务器上的防火墙。

    这应该被禁用或确保相关端口已打开。

    Windows 和 Linux 的处理方法不同。请查看以下文章：

    **对于 Windows**：[`technet.microsoft.com/en-us/library/cc753558.aspx`](https://technet.microsoft.com/en-us/library/cc753558.aspx)

    **对于 RedHat**：[`access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Security_Guide/sect-Security_Guide-Firewalls-Common_IPTables_Filtering.html`](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Security_Guide/sect-Security_Guide-Firewalls-Common_IPTables_Filtering.html)

1.  检查是否已安装 CMAgent。可以通过多种方式进行检查：

    **对于 Windows**：

    1.  检查 CMagnet 服务是否可用。

    1.  检查 `C:\Windows\CMAgent` 文件夹是否存在且不为空。

    1.  检查 CMAgent 是否出现在 **添加/删除程序** 中。

    **对于 Linux**：

    运行 `netstat -an | grep 26542`。

    确保服务正在监听端口 `26542`。

    执行以下命令检查服务是否正在运行：

    ```
     netstat -l | grep csi-agent

    ```

    该命令应返回 `tcp 0 0 :csi-agent *: LISTEN`。

    使用 top 实用程序监控进程。如果已安装并可用，你可以使用它来监控进程。操作方法如下：

    1.  启动 top 实用程序。

    1.  输入`u`。

    1.  在**用户（空白表示所有用户）：**提示符下，输入代理安装时使用的用户账户（`csi_acct`）。

    1.  输入`s`。

    1.  在**更新之间的延迟：**提示符下，输入`1`。

1.  检查 CMAgent 服务是否正在运行。

    根据操作系统的特定操作检查状态。

    **对于 Windows**：

    运行 `services.msc` 并确保 **CM Socket Listener** 服务已启动。

    **对于 Linux**：

    使用 root 权限登录到被管理服务器。

    进入 `/etc/init.d`。

    执行`./csi-service status`并确保它正在运行。

### 问题 04：CredentialsFailed

如下图所示，你可能会遇到一个名为 **CredentialsFailed** 的错误。这发生在为被管理机器配置的网络权限账户没有在该机器上具有足够权限时。

很可能，你正在管理的机器来自另一个域或处于工作组中，且你没有将所需的凭据作为网络权限账户添加。

![问题 04：CredentialsFailed](img/image_09_010.jpg)

VCM 代理通信问题：**CredentialsFailed**

## 如何操作...

查看 第四章中*在多域环境和工作组中修补机器*的配方、*Windows 补丁管理*，以及 第二章中的*添加网络权限账户以管理多个域中的机器*的配方，*配置 VCM 来管理你的基础设施*，然后为被管理的机器添加适当的网络权限账户，再尝试重新收集。

## 如何操作

我们在这里讨论了多个问题：如果我们想要管理代理，VCM 必须与代理进行通信。VCM 与代理的通信基于服务器上的证书；如果证书不符合预期，则服务器无法通信，我们需要通过重新安装代理并使用正确的证书来替换它。

我们需要确保可以解析被管理机器的主机名并连接到 26542 端口，因此需要开放中间的防火墙端口。

# 故障排除代理升级问题

在将 VCM 升级到版本 5.8.1 后，你可能在将代理升级到最新版本时遇到一些问题。升级失败，但如果你从 VCM 控制台选择**安装并移除当前版本**，代理就会安装/升级到最新版本。

### 注意

请注意，在撰写本书时，这个问题已经在 VCM 的最新版本（VCM 5.8.2）中得到解决。仅在你升级到 VCM 5.8.1 时适用。

## 准备就绪

当您在 VCM 控制台选择一台机器，选择升级选项并运行任务时，在为托管机器创建服务时应该会失败，并根据以下代码抛出错误，该代码位于`C:\Windows\CMAgent`。

```
    ***  Installation Started 03/03/2016 3:57  ***
    Title: EcmComSocketListenerService
    Source: C:\windows\TEMP\GLB90C6.tmp | 03-03-2016 | 03:57:04 | 71680
    Rem Wise Error Number: 141
    Rem Function Name: EcmCreateService
    Rem Error Message: Caught an exception from wise : Call to 
    EcmCreateService for service CSI Socket Listener failed with error code 
    of 1072 : error code 141
    141

```

## 如何操作...

在 5.8.1 版本中，VMware 推行了 HTTP 监听器模块，它在升级前会停止监听器服务，同时采集器会等待它。在监听器升级时，采集器会联系代理以获取升级状态。因此，采集器会收到 ping 失败的响应，并将信号发送给采集器以使升级任务失败。

以下设置允许在 HTTP 连接失败时，采集器回退到 DCOM：

1.  在选择 VCM 作为数据库时，在 SQL 管理服务器中运行以下查询：

    ```
     UPDATE dbo.ecm_sysdat_configuration_values SET configuration_value = 
     '1' WHERE configuration_name = 'config_allow_http_failover_to_dcom' 

    ```

1.  同时，使用此查询修正`ECMSocketListenerService`模块的文件名：

    ```
     UPDATE dbo.ecm_sysdat_install_modules SET module_file_name =
     'EcmComSocketListenerService6_6.exe' WHERE module_name = 
     'EcmComSocketListenerService'   AND module_version = '6.6' 

    ```

1.  重启 SSRS 和 VCM 采集器服务。

之后，您可以从控制台升级代理。

# 排查 SCR 下载问题

正如您所知道的，SCR 是一个工具，用于将 Linux/Unix 操作系统的补丁从供应商网站同步到本地仓库。有时，同步失败，无法从供应商网站下载内容。

## 准备中

我们能够使用相同的账户登录 RedHat 门户，但补丁下载失败并出现错误。当我们尝试从 SCR 与供应商（此处为 RedHat）同步补丁时，抛出如下错误，代码如下：

```
    Nov 19, 2015 9:11:32 AM com.lumension.scr.log.CommonsLogging error
    SEVERE: Error processing architecture X86_64
    com.lumension.scr.exception.AuthenticationFailedException: Failed to
    establish login session with RHN
            at
    com.lumension.scr.pojo.SCPackage.getAllPackageFiles(SCPackage.java:508)
            at
    com.lumension.scr.pojo.SCPackage.download(SCPackage.java:363)
            at  
    com.lumension.scr.client.StandaloneSCRepositoryClient.download 
    (StandaloneSCRepositoryClient.java:596)
            at
    com.lumension.scr.client.StandaloneSCRepositoryClient.process
    (StandaloneSCRepositoryClient.java:492)
            at
    com.lumension.scr.client.StandaloneSCRepositoryClient.main
    (StandaloneSCRepositoryClient.java:644)

```

由于这表示登录问题，首先怀疑的，和往常一样，是用于与 RHN 同步仓库的账户。我们可以用相同的账户登录 RHN，并且如果更换为其他账户，问题依然存在。

## 如何操作...

解决方法如下：

1.  重新同步 SCR 工具与 RedHat 仓库。

1.  选择 SCR 工具输出文件夹并删除所有`SystemId*.xml`文件：

    ```
     cd "PatchRepo/Repos"/unix 
           rm SystemId*.xml
     [root@scr unix]# pwd/opt/vcmpatches/unix[root@scr unix]# 
           lsSystemId_5Server_i386.xml    
           SystemId_6Server_i386.xml
    SystemId_5Server_x86_64.xml  
           SystemId_6Server_x86_64.xml

    ```

    对每个文件运行以下命令备份文件：

    ```
    [root@scr unix]# cp SystemId_5Server_i386.xml 
              SystemId_5Server_i386.xml.bak
    [root@scr unix]# rm -f *.xml
    [root@scr unix]# ls
    [root@scr unix]#

    ```

    `unix`文件夹的路径位于`properties`文件中，并通过`folder=value`参数定义。

    例如，`folder=/PatchRepo/Repos`。

1.  手动运行复制过程或允许其按计划运行：

    ```
     [root@scr unix]# cd /etc/cron.daily/ 
           [root@scr cron.daily]# ./SCR

    ```

1.  XML 文件将在之前的位置（`/PatchRepo/Repos`）创建，并且同步将开始：

    ```
     [root@scr unix]# pwd 
           /opt/vcmpatches/unix 
           [root@scr unix]# ls 
           SystemId_5Server_i386.xml    SystemId_6Server_i386.xml 
           SystemId_5Server_x86_64.xml  SystemId_6Server_x86_64.xml

    ```

## 它是如何工作的

真正的问题是源机器信息在**Red Hat Network**（**RHN**）上被更改或删除。

当我们删除 XML 文件并重新启动同步时，机器信息会再次更新，并且我们可以从 RHN 下载补丁。

# 排查 VCM 控制台登录失败问题

您尝试登录 VCM 时，系统抛出错误：**您的 ID 已禁用**。

## 准备中

您将需要访问 SQL 服务器来解决此问题。

![准备中](img/image_09_011.jpg)

请考虑以下情况：你没有任何人可以帮助你启用该 ID，或者更糟的是，你正在设置 VCM，却还没有添加任何人，因此你是唯一可以登录并进行必要设置的帐户。

不用担心，我们有解决方案。

你需要 SQL Management Studio 和对存放 VCM 数据库的 SQL 服务器的访问权限。

## 如何操作…

按照以下步骤解锁用户：

1.  登录到 VCM SQL 服务器并启动 SQL Management Studio。

    选择 VCM 作为数据库并运行第一个查询：

    ```
              select * from ecm_sysdat_logins 

    ```

    这将显示所有用户及其登录 ID。

1.  请记下你尝试的用户的登录 ID，并运行以下查询：

    ```
              update ecm_sysdat_logins set login_active = 1 where login_id = X 

    ```

    `X` 是你之前记录的用户登录 ID。

现在已经启用登录。尝试登录到 VCM 控制台，如果你是唯一的管理员，开始添加其他管理员。

## 它是如何工作的

VCM 将它所执行的所有操作信息存储在 SQL 数据库中。在此案例中，我们发现我们尝试登录的帐户被锁定，需要重置。

运行第一个 SQL 查询后，我们检查了数据库中帐户的状态，发现它被锁定（即 `login_active = 0`），于是我们运行了另一个查询并启用了该帐户（即 `login_active = 1`）。

之后，我们可以登录到 VCM 控制台。

建议将一个 AD 组添加到 VCM 中以便进行管理，具体内容请参考第七章中的*用户管理*部分，*VCM 维护*。

# 排查 vCenter 和 vCloud 数据采集问题

问题：尝试从 vCenter 和 vCloud Director 收集数据时，操作失败。

## 正在准备中

你可能遇到的问题是：当你尝试从 vCenter 或 vCloud Director 收集数据时，它要么失败，要么你无法在 VCM 控制台的**控制台 | 虚拟环境**下看到任何详细信息。

## 如何操作…

可能有几个原因导致此问题。

可以使用 **EcmDebugEventViewer** 工具或在 **作业管理器 | 历史记录** 中查看错误或失败的详细信息；根据错误代码，你可以尝试以下任何或所有解决方案：

+   配置 vCenter 和/或 vCloud 时设置的凭证可能不正确，请尝试更改它们。

+   配置 vCenter 和/或 vCloud 时设置的证书指纹可能不正确。

    如果是这种情况，那么在**作业管理器 | 历史记录**中，VCM 会显示配置的指纹和预期的指纹；你可以复制预期的指纹。确保它是正确的，然后更改错误的证书指纹。

+   如你所知，我们有过滤器集，大多数情况下，我们选择默认的过滤器集来执行数据采集。验证默认的过滤器集是否包含所有 vCenter 和/或 vCloud 相关的过滤器；如果没有，添加它们并进行数据采集。

    设置位于**管理**|**收集过滤器**|**过滤器集**下。选择**默认过滤器**，点击**编辑**，并添加任何缺失的 vCenter 和/或 vCloud 过滤器，如下图所示。

    您可以定义一个过滤器，使用类似于截图中的条件来限制过滤器：

    ![如何操作...](img/image_09_012.jpg)

    这将限制过滤器仅应用于 vCenter、vCloud 和 vShield。确保这些已经是默认过滤器集的一部分；如果没有，使用下拉箭头将它们添加到默认过滤器中，然后点击**下一步**。完成向导后，执行 vCenter、vCloud 和/或 vShield 的收集操作。它会生效。

    ![如何操作...](img/image_09_013.jpg)

    修改默认过滤器集

## 工作原理

VCM 使用过滤器集，这些过滤器集包含用于从各种托管对象收集数据的过滤器。过滤器定义了需要收集的数据，例如操作系统名称、某些注册表值或文件权限。有成千上万的过滤器可用，并非所有过滤器都在同一个过滤器集中使用。我们有一个默认的过滤器集，其中包括适用于所有托管机器类型的过滤器，如 vCenter、vCloud Director、Windows 和 Linux。有时，某些必需的过滤器会从过滤器集中丢失。我们需要将它们添加回来，以便下次使用时，VCM 知道需要收集哪些数据。

在这个故障排除步骤中，我们做了同样的事情，并将缺失的过滤器添加到默认过滤器集中。

# 故障排除：推荐操作“调查问题”导致的 Linux 服务器补丁错误

在尝试为 Linux 服务器安装补丁时，您可能会收到一条消息，显示**建议操作：调查问题**，因此无法为服务器安装补丁。

## 准备就绪

错误将类似于以下截图。推荐的操作是**调查问题**：

![准备就绪](img/B05400_09_15.jpg)

## 如何操作...

1.  要解决此问题，请在 VCM 数据库上按照以下步骤操作：

1.  运行此 SQL 查询以了解显示**调查问题**的`machine_id`：

    ```
           select machine_id, machine_name from ecm_dat_machines where 
           machine_name = '<Machine Name>' 

    ```

1.  机器名称是导致问题的服务的名称。

1.  使用以下查询更新`machine_id`值的时间戳：

    ```
           update ecm_sysdat_imd_machine_timestamp_xref set timestamp = 0 where 
           machine_id = <> 

    ```

1.  对该机器进行另一次评估，它会告诉您需要安装补丁。

# 排除无法在控制台中查看任何任务的故障排除

问题：VCM 没有运行或显示任何任务。当手动启动任务时，它们无法执行，这使得 VCM 几乎变得毫无用处。

## 准备就绪

当我们运行任何任务，如数据收集、补丁安装或合规性检查时，它们可能无法启动，并且在**任务**窗口中无法显示。因此，VCM 数据库没有更新，您无法为托管机器安装补丁，也无法执行合规性检查。

我们需要访问托管 VCM 数据库的 SQL 服务器。

## 如何操作...

这是解决方案。要确定是否启用了 SQL Server 服务代理，请在 SQL Server Management Studio 中运行以下脚本：

1.  启动 SQL Server Management Studio 并创建一个新的查询，如下所示。

    ```
           USE master;
           GO
           SELECT name, is_broker_enabled FROM sys.databases;
           GO 

    ```

1.  检查 VCM 数据库中 `is_broker_enabled` 的值是否为 `0`（零），并通过在 SQL Server 中运行以下脚本重新启用 SQL Server 服务代理：

    ```
           USE master;
           GO
           ALTER DATABASE {db-name} SET ENABLE_BROKER WITH ROLLBACK IMMEDIATE;      
           GO 

    ```

这里，`db-name` 是你的 VCM 数据库的名称。默认情况下，数据库名称是 VCM。

## 它是如何工作的

通过 Microsoft SQL Server 中的服务代理功能，内部或外部进程可以发送和接收保证的异步消息。消息可以发送到与发送者相同数据库中的队列、同一 SQL Server 实例中的另一个数据库，或另一个 SQL Server 实例，无论是在同一服务器上还是远程服务器上。我们修复了 VCM 数据库中损坏的代理服务，使其能够重新运行。

# 无法在调度作业页面看到月度选项的故障排除

有时，当我们尝试在 VCM 中安排任务时，快要完成向导时，在调度页面上，**月度**选项会被禁用（灰色显示），我们无法设置月度调度。

## 准备中

如下所示，**月度**调度选项被禁用：

![准备中](img/image_09_015.jpg)

## 如何操作...

如果启动控制台的 VCM 用户的时区设置与 VCM 数据库的时区设置不同，则**月度**调度选项将被禁用。

确保用于连接到 VCM 服务器的机器的时区与 VCM 数据库服务器的时区相同。
