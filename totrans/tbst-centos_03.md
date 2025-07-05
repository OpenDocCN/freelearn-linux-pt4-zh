# 第三章 网络环境故障排除

从幽灵连接到数据包错误，从流失败到连接错误，再到缺少路由，故障排除网络环境可能是一个缓慢且艰难的过程，通常从物理层开始。然而，一旦你确认物理节点正常工作，下一步就是考虑和查阅你系统中可用的各种工具。

在本章中，我们将：

+   了解一些基本工具，这些工具将帮助你排查与网络环境相关的各种问题。本讨论将涵盖 `ping`、`dig`、`host`、`traceroute` 和 `mtr` 的详细使用。

+   了解如何使用 `ss` 命令监控网络连接。

+   学习如何使用 `tcpdump` 检查数据包传输。

# 使用 ping、dig、host、traceroute 和 mtr

故障排除员最常用的一些工具有 `ping`、`dig`、`host`、`traceroute` 和 `mtr`。将这些工具一起使用，可以为故障排除员提供做出判断所需的证据，几乎可以解决任何网络相关的问题。这是网络工具包的基础，但需要强调的是，这些命令用于不同的目的，因此我们将分别介绍它们。

## ping 命令

`ping` 命令是一个小型工具，可以用来确定是否可以访问特定的 IP 地址。`ping` 命令在大多数计算机系统中都是常见的，它允许你查询 IP 地址或完全限定的域名，以检查是否有可用的连接。

`ping` 命令的基本语法如下：

```
# ping <ip_address>
# ping <domain_name>

```

`ping` 命令通过向指定的目标发出 ICMP 回显请求来验证和检查网络连接，正是这一简单的命令使其成为诊断任何基于网络的连接问题时非常有用的工具。

例如，如果你向 Google 发出 ping 请求（`# ping google.com`），那么根据你的网络环境和条件，输出将类似于以下内容：

```
PING google.com (216.58.210.14) 56(84) bytes of data.
64 bytes from lhr08s06-in-f14.1e100.net (216.58.210.14): icmp_seq=1 ttl=55 time=10.5 ms
64 bytes from lhr08s06-in-f14.1e100.net (216.58.210.14): icmp_seq=2 ttl=55 time=10.9 ms
64 bytes from lhr08s06-in-f14.1e100.net (216.58.210.14): icmp_seq=3 ttl=55 time=36.2 ms
64 bytes from lhr08s06-in-f14.1e100.net (216.58.210.14): icmp_seq=4 ttl=55 time=11.0 ms
64 bytes from lhr08s06-in-f14.1e100.net (216.58.210.14): icmp_seq=5 ttl=55 time=10.1 ms
64 bytes from lhr08s06-in-f14.1e100.net (216.58.210.14): icmp_seq=6 ttl=55 time=32.0 ms
64 bytes from lhr08s06-in-f14.1e100.net (216.58.210.14): icmp_seq=7 ttl=55 time=10.6 ms

```

这些结果展示了一个成功的 ping，可以描述为一个本地计算机系统向 `google.com (216.58.210.14)` 发出的回显请求的旅程。

请求从主机计算机发起，然后通过本地网络传送，最后经过互联网。一旦请求成功接收，目标将进行响应，并且测量完成此过程所需的时间，以生成平均响应或延迟时间。但是，如果没有响应，则很可能是网络本身存在物理问题，或者是一些基本问题，如位置不正确或不可用，目标计算机未响应 ping 请求，或者主机路由表错误。

### 注意

在在线游戏中，ping 请求（也称为“高 ping”或“低 ping”）通常与从本地机器到外部游戏服务器的速度测量相关。例如，一个低 ping（例如 10 毫秒）的玩家会比一个 180 毫秒 ping 的玩家拥有更好的游戏体验。

此外，你还应该意识到，如果你的 ping 值超过 500 毫秒，则意味着任何请求需要超过半秒才能到达服务器并返回。这个条件很明显，因为你可能会经历“帧抖动”或“帧跳跃”——在在线游戏中被称为“橡皮带现象”。

`ping` 命令很简单易用，但在其默认形式下，它会一直执行，直到被取消。在某些情况下，这可能会很有用，但通常更容易使用 `-c` 选项，以减少回显请求的次数，并获取事件的总结。

例如，如果你想将回显请求的数量限制为 4 次，你需要输入以下命令：

```
# ping -c 4 google.com

```

在前面的例子中，`ping` 命令将在 4 次循环后停止发出回显请求，基于我们之前的例子，输出会类似于此：

```
PING google.com (216.58.208.78) 56(84) bytes of data.
64 bytes from lhr14s27-in-f14.1e100.net (216.58.208.78): icmp_seq=1 ttl=55 time=11.9 ms
64 bytes from lhr14s27-in-f14.1e100.net (216.58.208.78): icmp_seq=2 ttl=55 time=16.7 ms
64 bytes from lhr14s27-in-f14.1e100.net (216.58.208.78): icmp_seq=3 ttl=55 time=35.4 ms
64 bytes from lhr14s27-in-f14.1e100.net (216.58.208.78): icmp_seq=4 ttl=55 time=15.1 ms

--- google.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3005ms
rtt min/avg/max/mdev = 11.985/19.827/35.462/9.187 ms

```

在我们结束对 `ping` 命令的介绍之前，有几点关于任何 ping 请求需要考虑的事项。这些点可能不一定表示问题，但它们会影响 ping 测试的结果：

+   **与目标的距离**：假设你住在美国，并尝试连接到欧洲的服务器。在这种情况下，你应该预期 ping 值会高于你尝试连接的另一台距离你更近的美国服务器。此外，你还应该预期，不同地理位置之间的速度可能会有所不同。

+   **互联网连接速度**：如果你的网络带宽较低（即上传和下载速度差），则 ping 请求的响应时间会比高速带宽的宽带连接（即上传和下载速度较好）更长。

+   **跳数**：跳数是一个通用术语，指的是 ping 请求必须经过的路线和服务器，以便到达目标并返回。因此，就像在现实生活中一样，如果你住得离主要火车线路较远，你就需要做更多的“连接”或“跳跃”，才能到达最终目的地。

基本原则始终表明，低 ping 始终是可取的，因为它对基于时间的指令至关重要。然而，在进行 ping 测试时，你不仅要考虑实际到达目标站点的 ping 总数，还要仔细记录相关 ping 的平均值和标准差。

从这个角度看：如果 ping 请求没有到达，这可能表明由于你计算机和目标之间的互联网连接不稳定，可能会出现数据包丢失的情况。然而，如果 ping 率较低，但在特定时间段内显示出一个逐渐增大的变化率，那么在某些情况下，这种环境并不总是比同一时间段内的恒定速率更可取。

## `dig`和`host`命令

`dig`命令可以用来验证 DNS 映射、互联网连接、主机地址和 MX 记录，并且可以帮助发现与潜在的反向 DNS 问题相关的信息，这些问题可能导致垃圾邮件和被列入黑名单。通过`bind-utils`包提供的`dig`命令，返回的信息分为四个主要部分，包括：头部部分（使用的选项列表）、问题部分（查询类型）、答案部分（相关位置的地址）、查询部分（包含关于查询时间、名称服务器等的统计信息）。`dig`命令是为了替代`nslookup`而推出的，基本语法如下：

```
# dig google.com

```

结果将如下所示：

```
; <<>> DiG 9.9.4-RedHat-9.9.4-18.el7_1.1 <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 18657
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             299     IN      A       216.58.210.78

;; Query time: 100 msec 
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Sat Apr 25 13:45:02 EDT 2015
;; MSG SIZE  rcvd: 55

```

你会注意到，这种输出中包含了大量信息，因此让我们从全局选项部分开始逐步解析：

```
; <<>> DiG 9.9.4-RedHat-9.9.4-18.el7_1.1 <<>> google.com
;; global options: +cmd

```

然后是一个输出，报告查询的答案：

```
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 18657
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1
;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512

```

紧接着，`dig`会重复原始问题：

```
;; QUESTION SECTION:
;google.com.                    IN      A

```

答案如下所示：

```
;; ANSWER SECTION:
google.com.             299     IN      A       216.58.210.78

```

最后，我们将看到一些关于查询本身的一般统计数据：

```
;; Query time: 100 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Sat Apr 25 13:45:02 EDT 2015
;; MSG SIZE  rcvd: 55

```

同样，通过将`XXX.XXX.XXX.XXX`替换为相关的 IP 地址，你可以像这样查询特定的名称服务器：

```
# dig google.com @XXX.XXX.XXX.XXX

```

所以，如果你运行以下命令：

```
# dig google.com @8.8.8.8

```

你可以期望看到以下结果：

```
; <<>> DiG 9.9.4-RedHat-9.9.4-18.el7_1.1 <<>> google.com @8.8.8.8
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 5496
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             299     IN      A       216.58.210.78

;; Query time: 92 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Sat Apr 25 13:46:54 EDT 2015
;; MSG SIZE  rcvd: 55

```

此外，`dig`命令的默认操作是查找`A`记录，你可以通过调整`dig`语法来获取基于特定记录类型的信息，方法如下：

```
# dig google.com MX
# dig google.com TXT
# dig google.com NS
# dig google.com SOA

```

作为替代方法，并为了使结果更具普适性，你可以通过输入`ANY`查询来尽可能多地获取信息：

```
# dig google.com ANY

```

此外，`dig`命令还可以用来执行反向查找，以根据特定的 IP 地址获取相关的 DNS 信息。

这可以通过输入以下命令来实现：

```
# dig -x 8.8.8.8

```

前述命令随后会以以下方式响应：

```
; <<>> DiG 9.9.4-RedHat-9.9.4-18.el7_1.1 <<>> -x 8.8.8.8
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 34651
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;8.8.8.8.in-addr.arpa.          IN      PTR

;; ANSWER SECTION:
8.8.8.8.in-addr.arpa.   21599   IN      PTR     google-public-dns-a.google.com.

;; Query time: 35 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Sat Apr 25 13:49:13 EDT 2015
;; MSG SIZE  rcvd: 93

```

如你所见，`dig`是一个灵活的命令行工具，它可以让你执行有效的 DNS 查询。它的输出非常详细，但可以通过`+short`开关来简化并提供一个简洁的答案，像这样：

```
# dig -x 209.132.183.81 +short

```

上述命令应该以以下方式响应：

```
www.redhat.com.

```

`dig`命令是一个极其有用的网络故障排除工具，它的成功主要归功于它能够返回问题、答案、权威和附加部分的能力。

然而，话虽如此，另一种选择是使用`host`命令，方法如下：

```
# host –a google.com

```

`host` 命令的工作方式与 `dig` 命令类似，但其输出如下所示：

```
Trying "google.com"
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 60735
;; flags: qr rd ra; QUERY: 1, ANSWER: 14, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;google.com.                    IN      ANY

;; ANSWER SECTION:
google.com.             299     IN      A       216.58.210.78
google.com.             299     IN      AAAA    2a00:1450:4009:801::200e
google.com.             21599   IN      NS      ns3.google.com.
google.com.             21599   IN      TYPE257 \# 19 0005697373756573796D616E7465632E636F6D
google.com.             599     IN      MX      20 alt1.aspmx.l.google.com.
google.com.             21599   IN      NS      ns4.google.com.
google.com.             21599   IN      SOA     ns1.google.com. dns-admin.google.com. 2015041501 7200 1800 1209600 300
google.com.             3599    IN      TXT     "v=spf1 include:_spf.google.com ip4:216.73.93.70/31 ip4:216.73.93.72/31 ~all"
google.com.             599     IN      MX      40 alt3.aspmx.l.google.com.
google.com.             21599   IN      NS      ns1.google.com.
google.com.             599     IN      MX      30 alt2.aspmx.l.google.com.
google.com.             599     IN      MX      50 alt4.aspmx.l.google.com.
google.com.             21599   IN      NS      ns2.google.com.
google.com.             599     IN      MX      10 aspmx.l.google.com.

```

如你所见，`host` 命令与 `dig` 的作用类似，但它更加简洁。

例如，一个基本的 `host` 查询如下所示：

```
# host www.google.com

```

返回的输出将如下所示：

```
www.google.com has address 216.58.210.68
www.google.com has IPv6 address 2a00:1450:4009:801::2004

```

另外，你也可以通过以下方式指定第三方 DNS 服务器：

```
# host www.redhat.com 8.8.8.8

```

这将报告使用替代 DNS 服务器的情况，并显示如下内容：

```
Using domain server:
Name: 8.8.8.8
Address: 8.8.8.8#53
Aliases:

www.redhat.com is an alias for wildcard.redhat.com.edgekey.net.
wildcard.redhat.com.edgekey.net is an alias for wildcard.redhat.com.edgekey.net.globalredir.akadns.net.
wildcard.redhat.com.edgekey.net.globalredir.akadns.net is an alias for e1890.b.akamaiedge.net.
e1890.b.akamaiedge.net has address 23.195.127.72

```

最后，`host` 可以执行反向查找，如下所示：

```
# host XXX.XXX.XXX.XXX

```

所以，假设你对 Red Hat 的 Akamai Edge 服务器进行反向查找，使用以下命令：

```
# host 23.195.127.72

```

输出将如下所示：

```
72.127.195.23.in-addr.arpa domain name pointer a23-195-127-72.deploy.static.akamaitechnologies.com.

```

那么从这个角度来看：为了故障排除网络，你可以使用 `dig` 或 `host`。这两个命令在用途和功能上是相似的，但 `host` 提供了简单性，而 `dig` 则提供了更高级的、适合脚本使用的选项。

## `traceroute` 命令

`traceroute` 命令旨在显示到远程目的地的路径，以及每个停靠点发生的延迟。大多数管理员都熟悉 `traceroute`，它有三个主要目标，可以总结如下：

+   提供数据包将要经过的整个路径的详细信息

+   提供路径上找到的设备和路由器的名称及其身份

+   报告网络延迟的情况，评估在给定路径上向每个设备发送和接收数据所花费的时间

简单来说，`traceroute` 命令是一个工具，用于验证数据将要经过的路径，而不会使用任何数据，所使用的语法基于以下内容：

```
# traceroute google.com

```

输出将提供指定的主机、该域的 IP 地址、所需的最大跳数以及将使用的包大小。随后的行将报告跳数、主机名、IP 地址和数据包的往返时间。你当然也可以使用 `-n` 选项以以下方式避免反向 DNS：

```
# traceroute -n google.com

```

`traceroute` 命令每次发送三个包，带有每个 TTL，并将显示往返时间 (RTT)，这表示从发出探测到接收到响应包之间的时间差。这对于发现网络瓶颈非常有用，如果你开始看到星号（*），则表明路由到该主机可能存在问题，因为星号可能表示数据包丢失或丢弃数据包。然而，也需要认识到，解释 `traceroute` 结果需要理解并接受它固有的特点。

`traceroute`命令被认为是 TCP/IP 故障排除的基石。它开始时会发出一个基于 UDP 的数据包，TTL 值为 1。如果数据包到达目标，网关会发送响应包并报告其结果。然而，如果数据包没有到达目标，那么接收网关会将 TTL 值减少 1。如果 TTL 值为 0，则网关会丢弃数据包，并在发出一个增加了 TTL 值的新数据包后报告结果，以绕过下一个阶段的同一网关。这个过程会一直重复，直到到达目标主机或达到最大 TTL 值为止。

有三种不同类型的`traceroute`实现，分别涵盖 UDP、TCP 和 ICMP。

例如，如果你想使用 ICMP 变种，你可以输入：

```
# traceroute –I google.com

```

同样，你可以像这样使用`–n`选项绕过 DNS：

```
# traceroute -I -n 8.8.8.8

```

这种变种的工作方式与之前使用 UDP 的示例类似，`traceroute`程序将发送回显请求，途中经过的每一跳都会做出回应。然而，与 UDP 版本不同的是，这个过程将使用 ICMP。

最终的方法是使用以下方式的 TCP 变种：

```
# traceroute -T google.com

```

在许多方面，TCP 选项可能是最有效的方法，因为大多数网络都会允许这种流量。如果你的目标是 80 端口，尤其如此。然而，没有硬性规定来确定你可以或想要使用哪种版本的`traceroute`。这些规则将由网络配置设定，因为某些网络默认会阻止 UDP 请求（通常是 33434 到 33534 端口）。所以，基于这一点，为什么不尝试所有版本，看看哪个最适合你的环境呢？

让我们这样来看：了解`traceroute`的工作原理仅仅是赢得了半场胜利。如果`traceroute`可以到达主机，但无法到达目标，那么问题很可能出在目标上。然而，如果`traceroute`无法到达主机，那么问题可能出在路由本身，这不仅涉及到一些路由器拒绝`traceroute`数据包，还有其他一些路由器在带宽和延迟上表现出显著的差异、防火墙，以及过滤`traceroute`数据包的其他各种陷阱。在这种情况下，应选择多个目标（你还应该考虑使用 UDP、ICMP 和 TCP 发送请求，以绕过任何网络问题），并且考虑到互联网本质上是非对称的，通常最好在两个方向上执行`traceroute`操作，以便评估整体网络效率。

总的来说，`traceroute`是一个很好的工具，但它可能会误导你，因此在分析结果时要小心，并始终用额外的调查来补充你的工作。

## mtr 命令

作为`traceroute`的替代方案，有`mtr`命令。在某些 Linux 系统上，你需要以 root 用户身份运行此命令，或者与`sudo`一起使用，但无论你使用哪种方法，该命令的语法非常简单，并按如下方式工作：

```
# mtr google.com

```

输出可能类似于`traceroute`，但显示是实时的，从而使你能够监控趋势和平均值，反映网络性能随时间的变化。因此，与`traceroute`不同，`mtr`通过收集更长时间的数据，不仅能简单地拍摄一次旅程的快照，还能检查间歇性的数据包问题。此外，作为实时更新的替代方案，`mtr`还将提供一个报告选项，向每个遇到的跳点发出 10 个数据包的结果：

```
# mtr --report google.com

```

所以，经过反思，可以认为`mtr`在监控网络连接方面优于`traceroute`。它确实有许多优点，并且可以提供大量细节，但考虑到你无法控制外部网络的工作方式，经验丰富的故障排除人员应始终保持警觉，并选择检查所有可用的工具。

# 使用`ss`命令监控网络连接

套接字统计命令（`ss`）是`netstat`的继任者；它不仅更快，而且能够显示更多的信息。然而，与通过`/proc`目录中的各种文件获取信息的`netstat`不同，`ss`命令直接从内核空间获取信息。

`ss`命令的基本语法如下：

```
# ss | less

```

使用这种语法，我们只是调用了所有 TCP、UDP 和 Unix 套接字连接的输出，并可选择通过管道将其发送到`less`，以确保结果可以在屏幕上查看。当然，您可以将此命令与`-t`、`-u`或`-x`选项结合使用，以限制输出仅显示 TCP、UDP 或 Unix 套接字连接，但为了使输出更加信息丰富，你可能想要将这些附加选项与`-a`选项结合使用，以报告连接和监听套接字，如下所示：

```
# ss -ta

```

正如你所注意到的，在前面的例子中，我们只报告了当前的 TCP 环境，可以通过类似的方式将其更改为适用于 UDP（`ss -ua`）或 Unix 套接字连接（`ss -xa`）。然而，如果你喜欢一定程度的精确性，你会欣慰地知道，`ss`命令可以通过使用`–A`选项与查询结合使用，如下所示：

```
# ss -a -A tcp

```

限制输出确实能使信息更加简洁，但要进一步提升，可以使用以下语法应用附加的过滤器：

```
# ss [ OPTIONS ] [ STATE-FILTER ] [ ADDRESS-FILTER ]

```

例如，考虑到所有标准的 TCP 状态，你可以以下列方式显示所有已建立的 IPv4 TCP 套接字：

```
# ss -t4 state established

```

你可以像这样显示所有关闭的 TCP 状态：

```
# ss -t4 state closed

```

现在，可以说使用`ss`命令的效果与使用`netstat -a`类似。部分是正确的，但（记住你可以将`-t`替换为`-u`或`-x`）考虑到通过不解析主机名来提高执行速度（`ss -nt`），只显示监听套接字（`ss -ltn`），显示套接字内存使用情况（`ss -t -m`），显示使用特定套接字的进程（`ss -t -p`），打印进程名称（`ss -ltp`），显示 IPv4 或 IPv6（`ss -tl4`或`ss -tl6`），并显示时间信息（`ss -tn -o`），你会发现我们才刚刚触及`ss`命令的表面。

例如，你甚至可以运行查询，通过使用以下语法来发现谁在使用端口 22（SSH）：

```
# ss -lpn | grep 22

```

另外，你可以使用以下语法来显示从远程 IP 地址连接的所有端口：

```
# ss dst XXX.XXX.XXX.XXX

```

然后使用以下变体将查询过滤到特定端口：

```
# ss dst XXX.XXX.XXX.XXX:22

```

记住，熟悉你的网络环境总是会有所帮助，掌握了这个命令后，你应该能在问题连接发生之前就能识别它们。

# 使用 tcpdump 进行数据包分析

`tcpdump`命令是一种数据包分析工具，它能够捕获并提供通过网络接口传输的流量描述。它是大多数 Linux 发行版的标准工具，提供了一个独特的网络数据包层级视图，在网络故障排除时非常有用。

使用`tcpdump`的基本语法如下：

```
# tcpdump -i <device_name>

```

你还可以像这样指定协议：

```
# tcpdump -i <device_name> tcp

```

端口值可以按以下方式使用：

```
# tcpdump -i <device_name> port 22

```

可以使用`-v`或`-vv`选项发出详细信息选项，而通过使用`-n`选项可以避免 DNS 解析。然而，由于`tcpdump`会一直运行，直到请求被取消，因此最好使用`-c`选项来捕获预定数量的事件，像这样：

```
# tcpdump -c 10 -i <device_name>

```

更进一步，你可以通过调用`src`选项（源）或`dst`选项（目的地）来从特定 IP 地址捕获`10`个数据包，像这样：

```
# tcpdump -c 10 -i <device_name> src XXX.XXX.XXX.XXX

```

设备名称本身可以通过运行以下选项获取：

```
# tcpdump -D

```

`tcpdump`命令可以在读取模式和写入模式下运行。然而，后者意味着使用`-w`选项，这会导致`tcpdump`将数据包数据保存到文件中以供后续分析，而前者，通过使用`-r`选项，意味着`tcpdump`只会从已保存的数据包文件中读取。如你所知，在写入模式下你应该指定相关的设备名称（即`eth0`），但在这两种情况下，只有匹配表达式的数据包才会被匹配并显示出来。

例如，在读取模式下，这个命令的基本语法如下：

```
# tcpdump -r <file_name>

```

在写入模式下，你可以以以下方式发送整个以太网帧进行进一步分析：

```
# tcpdump -w /path/to/file -i <device_name>

```

如你所见，`tcpdump`最常见的应用是验证双向通信是否正常。`tcpdump`命令可以用来记录网络片段，在认识到这一点后，我们仅仅触及了它灵活性的表面。

因此，我希望你能已经看到，当排查你的网络环境问题时，这个小工具可以成为一个重要的工具。

# 总结

本章的目的是提供一个起点，以便在尝试排查网络环境中的问题时使用。当然，学习的内容总是有更多，而了解这些将带你踏上远超`dig`、`ping`甚至`tcpdump`基本语法的旅程。然而，通过学习多个命令和工具，你现在可以看到，成为一个有效的故障排除者正变成一个可以实现的目标。为了进一步推动这一目标，我们将把注意力转向包管理的故障排除需求。

# 参考文献

+   TCP 的维基百科页面：[`en.wikipedia.org/wiki/Transmission_Control_Protocol`](http://en.wikipedia.org/wiki/Transmission_Control_Protocol)

+   Ping 的维基百科页面：[`en.wikipedia.org/wiki/Ping_(networking_utility)`](http://en.wikipedia.org/wiki/Ping_(networking_utility))

+   Traceroute 的维基百科页面：[`en.wikipedia.org/wiki/Traceroute`](http://en.wikipedia.org/wiki/Traceroute)

+   `ss`命令的官方页面：[`www.cyberciti.biz/files/ss.html`](http://www.cyberciti.biz/files/ss.html)

+   ARP 的维基百科页面：[`en.wikipedia.org/wiki/Address_Resolution_Protocol`](http://en.wikipedia.org/wiki/Address_Resolution_Protocol)

+   `dig`命令的维基百科页面：[`en.wikipedia.org/wiki/Dig_(command)`](http://en.wikipedia.org/wiki/Dig_(command))

+   `tcpdump`的维基百科页面：[`en.wikipedia.org/wiki/Tcpdump`](http://en.wikipedia.org/wiki/Tcpdump)
