# 第五章：发现运行在集群中的服务

当部署中有大量成员时，系统需要具备易于管理的特性，且尽量减少人工干预。人工干预往往会带来人为错误，使系统不稳定。设想一个场景：有一个负载均衡器，将 HTTP 流量分发到多个服务器。如果任何服务器宕机或上线，负载均衡器必须自动知道节点或服务的增加或删除，而无需人工干预，否则管理这样的部署将是一场噩梦。服务发现确保负载均衡器能够了解当前活跃的服务实例；基于此，它可以做出路由决策。

本章解释了发现运行在集群中的服务的需求和机制。

本章涵盖以下主题：

+   服务发现的介绍和必要性

+   服务发现机制

# 服务发现的介绍和必要性

在 CoreOS 环境中，所有用户应用程序都将作为服务部署在容器内。对于大多数应用程序，用户应用需要协同工作，因此需要一种机制来发现这些服务及其参数。通过 etcd 进行服务发现提供了一种将服务及其所需参数发布给系统中其他服务的方法。服务发现机制不仅对服务参数的发现有用，还涉及成员状态的检测（如新成员的加入、提供服务的成员的移除或成员宕机），以及服务状态（如提供应用的服务启动或停止）和服务参数（如服务提供的 IP 和端口、数据库连接端点等）。服务信息必须在所有成员之间始终可用，这意味着服务发现机制应通过多个成员进行复制，并始终可用，以避免单点故障。

# 服务发现机制

CoreOS 服务`etcd`和`fleetd`提供的功能可以用于发现服务。下图解释了用于服务发现的典型机制：

![服务发现机制](img/00022.jpeg)

在前面的章节中，我们已经看到如何使用`etcd`和`fleetd`来发现集群中的成员节点。etcd 服务不仅限于节点发现，还可以用于发现或发布与应用程序或服务相关的信息。本章接下来的部分将介绍如何使用 etcd 发布和发现与服务相关的信息。

集群中有两种类型的成员节点：前端服务节点和后端服务节点。

+   前端服务处理所有的服务请求，并将请求路由到`backend`服务进行实际处理。这是任何高容量系统的简化但典型的架构。在前端服务节点中，将运行以下服务：

    +   `Discovery`服务

    +   `etcd`服务

    +   `fleetd`服务

    +   前端或路由服务

+   后端服务节点负责运行前端服务节点调度或路由的服务。在后端服务节点中，将运行以下服务：

    +   `Register`服务：如果需要简单的服务发现，可以将其包含在后端服务单元文件中的`ExeStartPost`中。

    +   `etcd`服务

    +   `fleetd`服务

    +   后端或实际服务

`fleetd`和`etcd`服务已经在前面的章节中详细讨论过。运行在后端节点的 Register 服务会更新服务信息到 etcd 键值存储，这些信息将发布到发现服务。运行在前端节点的发现服务用于通过 etcd 键值存储信息来发现后端成员的服务信息。

让我们一步步了解发现的完整流程：

+   启动了`Frontend`成员。fleetd 服务启动并调度`Frontend`和`Discovery`服务。使用服务级亲和性来确保它们一起运行在同一个成员上。

+   `Discovery`服务使用`etcd`键值存储功能来查找后端成员信息。它还设置了一个监视，以便了解服务发现信息的任何变化。我们将在本章稍后了解如何读取、写入并设置 etcd 的监视。由于后端成员尚未启动，因此没有可用的服务信息，因此发现服务处于等待模式，等待服务信息的任何更新。

+   启动了其中一个`backend`成员。`fleetd`服务再次启动并调度`Backend`和`Register`服务。这里同样使用服务级亲和性来确保它们一起运行在同一个成员上。Register 服务将服务信息更新到`etcd`键值存储中。这些服务信息对前端成员很有用。服务信息可以是端点信息，如 IP 和端口、服务类型或任何其他元数据，这些都对前端在调度请求时做出明智决定非常必要。设置数据的生存时间并定期重写数据到 etcd 上也很重要。为数据设置生存时间可确保当服务终止时，服务数据也会被移除。

+   由于前端成员上的`Discovery`服务已经在 etcd 上设置了监视，它可以得知新的服务实例的增加。然后，它会更新前端服务，添加该服务实例。根据可用的信息，前端服务可以开始将传入的请求调度到该服务实例。

+   一旦其他成员启动，服务发现会继续按前面步骤所述进行，前端服务将意识到更多的服务实例用于调度请求。

+   现在，假设某个成员出现故障。如果服务信息在超时时间后没有再次更新，它会从 `etcd` 中删除。类似地，当成员恢复正常，但后端服务出现故障且无法再次启动时，`fleetd` 会关闭注册服务，因为它们是绑定在一起的。如果它们在成员上无法再次启动，服务参数将在超时时间后再次在 `etcd` 上过期。

+   参数删除会被 `Discovery` 服务检测到，并将信息传递给 `Frontend` 服务。现在，`Frontend` 服务知道有一个服务实例被删除了。

请注意，这只是整个过程的概念性表示。并没有限制你的前端应用程序和发现服务必须是两个独立的应用或服务。你的前端应用程序也可能包含发现服务。当你使用第三方或现成的前端应用程序如 `HAProxy` 时，你可能需要编写一个简化的发现服务，或者你也可以使用 `confd`，这是另一个现成的发现应用程序。同样，后端服务和注册服务可以结合在一起，或者你可以使用另一个现成的应用程序 `forest`，直接更新 `etcd`，而无需编写注册服务。

## etcd 操作

`etcd` 提供以下三种操作来操作键值存储：

+   etcd 写操作

+   etcd 读取

+   etcd 监听

CoreOS 提供了两种主要的接口来执行前述 `etcd` 操作：

+   `etcdctl`

+   基于 REST 的接口。可以使用 `cURL` 来调用 REST API。

### 使用 etcdctl 的操作

`etcdctl` 是 `etcd` 的命令行客户端。使用 `etcdctl`，你可以读取、写入和监听 `etcd` 的键值存储。`etcdctl` 可以作为配置键值存储的独立工具使用，也可以在脚本中使用。`etcdctl` 会向 `etcd` 服务发送请求消息，并等待来自 etcd 的响应。`etcdctl` 可以返回以下任一返回码：

| 返回值 | 语义 |
| --- | --- |
| 0 | 成功 |
| 1 | etcdctl 参数格式错误 |
| 2 | 无法连接到主机 |
| 3 | 认证失败（客户端证书被拒绝，CA 验证失败等） |
| 4 | 来自 etcd 的 400 错误 |
| 5 | 来自 etcd 的 500 错误 |

#### 使用 etcdctl 进行 etcd 写操作

etcd 写操作服务将被后端节点用于通过键值数据存储发布服务信息到前端服务。

使用 `etcdctl` 可以执行以下写操作。下表列出了 `etcdctl` 提供的命令选项，包括语法和示例：

| 操作 | 命令语法 | 示例 |
| --- | --- | --- |
| 设置一个键的值 | `etcdctl set <key> <value>` | `$ etcdctl set /foo/bar "foo bar"` |
| 设置一个带有过期时间（秒）的键值 | `etcdctl set <key> <value> –ttl` | `$ etcdctl set /foo/bar "foo bar" –ttl 10` |
| 基于先前值有条件地设置键的值 | `etcdctl set /<key> <old-value> --swap-with-value <new-value>` | `$ etcdctl set /foo/bar "foo bar" --swap-with-value "bar foo"` |
| 创建一个新键 | `etcdctl mk <key> <value>` | `$ etcdctl mk /foo/bar "foo bar"` |
| 创建一个新目录 | `etcdctl mkdir <dir>` | `$ etcdctl mkdir /foo/bar` |
| 更新一个键的值 | `etcdctl update <key> <value>` | `$ etcdctl set /foo/bar "bar foo"` |
| 删除一个键 | `etcdctl rm <key>` | `$ etcdctl rm /foo/bar` |
| 递归删除一个键及其所有子键 | `etcdctl rm <key> --recursive` | `$ etcdctl rm /foo/bar –recursive` |
| 有条件地删除一个键 | `etcdctl rm <key> --with-value <value>` | `$ etcdctl rm /foo/bar --with-value "foo bar"` |
| 删除一个目录 | `etcdctl rmdir <dir>` | `$ etcdctl rmdir /foo/bar` |

#### etcd 使用 etcdctl 进行读取

`etcd read`服务将由前端节点用于通过键值数据存储来发现服务信息。

以下是使用`etcdctl`可以执行的读取操作。下表列出了`etcdctl`提供的命令选项、语法和示例：

| 操作 | 命令语法 | 示例 |
| --- | --- | --- |
| 检索一个键值 | `etcdctl get <key>` | `$ etcdctl get /foo/bar``foo bar` |
| 带有附加元数据的检索键值 | `etcdctl -o extended get <key>` | `$ etcdctl -o extended get /foo/bar`键：/foo/bar 修改索引：72TTL：0Etcd 索引：72Raft 索引：5611Raft 任期：1foo bar |
| 创建一个新键 | `etcdctl mk <key> <value>` | `$ etcdctl mk /foo/bar "foo bar"` |
| 列出目录 | `etcdctl ls` | `$ etcdctl ls``/foo` |
| 递归列出目录 | `etcdctl ls –recursive` | `$ etcdctl ls --recursive``/foo``/foo/bar` |

使用我们的`testservices`示例，以下命令用于通过`etcdctl`读取参数。请注意，我们在这里使用`ls`命令获取服务列表，然后查询特定的服务实例。

```
etcdctl ls /testservice/backend/1
/testservice/backend/1
/testservice/backend/2

etcdctl get /testservice/backend/2
172.17.8.102:55555

```

#### etcd 使用 etcdctl 进行监视

`etcd`的`watch`服务将由前端节点用于监控或观察键值数据存储中的任何变化。

以下是使用`etcdctl`可以执行的监视操作。下表列出了`etcdctl`提供的命令选项、语法和示例：

| 操作 | 命令语法 | 示例 |
| --- | --- | --- |
| 监视键值的任何变化。 | `etcdctl watch <key>` | `$ etcdctl watch /foo/bar` |
| 持续监控键值的任何变化。在此情况下，etcdctl 会一直挂起，直到按下*Ctrl* + *C*，并且当键值发生变化时，它会打印该值。 | `etcdctl watch <key> --forever` | `$ etcdctl watch /foo/bar --forever``foo bar` |
| 持续监视密钥值的任何变化，并在密钥值发生变化时执行程序。 | `etcdctl exec-watch <key> --sh -c 要执行的程序` | `$ etcdctl exec-watch -- sh -c env &#124; grep ETCD` |

#### 使用 etcdctl 的 etcd 示例

到目前为止，我们已经以理论方式了解了如何发布和发现服务参数。现在是时候进行一些实际操作了。让我们从熟悉用于发现的 etcd 键值存储功能开始，并通过一个名为`testservices`的服务示例来展示如何使用它，该服务发布了该服务运行的 IP 地址和端口号。

这里，`testservices`是添加所有新服务信息的目录。IP 是成员的 IP 地址，`5555`是服务运行的端口（此示例中选择的端口）。

以下是用于将 IP 地址和端口（使用 IP 作为键）写入的命令：

```
etcdctl set /testservices/ip '172.17.8.101:55555' –ttl 30

```

可选参数`–ttl 30`被添加，用于设置密钥的生命周期为 30 秒。

### 提示

在我们的示例中，我们选择展示如何将 IP 地址和端口号键写入到密钥存储中。请注意，了解服务运行的 IP 地址有多种方式。可以使用环境变量`COREOS_PRIVATE_IPV4`和`COREOS_PUBLIC_IPV4`，或者使用`ipconfig`命令来查找为成员分配的 IP 地址。

要获取`testservices`发布的参数，请使用以下命令：

```
etcdctl get /testservices/ip 172.17.8.101:55555

```

要监视这些参数，应使用以下命令：

```
etcdctl watch /testservices/ip

```

以下是我们之前使用`etcdctl`查询的命令，用于写入条目。

```
etcdctl set /testservices/backend/1 172.17.8.101:55555 –ttl 30
etcdctl set /testservices/backend/2 172.17.8.102:55555 –ttl 30

```

### 使用 cURL 的操作

**cURL**，通常称为`curl`，是一个命令行工具，用于通过各种协议将数据传输到应用服务器或从服务器传输数据。cURL 支持多种协议，包括**HTTPS**、**HTTP**、**FTPS**、**FTP**、**SCP**、**TFTP**、**SFTP**、**DAP**、**LDAP**、**DICT**、**TELNET**、**IMAP**、**FILE**、**POP3**、**SMTP**和**RTSP**。它通常用于使用类似 URL 的语法获取或发送文件。像`etcdctl`一样，curl 也可以作为独立工具配置键值存储，或者在脚本中使用。所有可以使用`etcdctl`完成的操作，也可以使用 curl 完成。curl 还提供了更多操作来操作键值存储。curl 向 etcd 服务发送请求消息并等待响应。响应包含以下参数/属性：

+   操作：操作字段表示发送的 curl 请求的类型。操作可以取值为`get, set, create, delete, update, expire, watch`等。

+   节点：`node`字段表示键值存储的目录。它由键、值、`createIndex`和`modifiedIndex`组成。

    +   `key`字段表示键值存储的键。

    +   `value`字段表示键值存储的值。

    +   每个节点都有一个名为 `index` 的字段，每次更改 `etcd` 时，该字段的值会递增。`createdIndex` 字段则填充此索引。

    +   `modifiedIndex` 还表示节点的索引。然而，它表示对该节点应用的操作次数，这会改变此键值存储的值。

`curl set` 命令的示例输出如下：

```
curl -L http://127.0.0.1:4001/v2/keys/foo/bar -XPUT -d value="foo bar"

{
 "action": "set",
 "node": {
 "createdIndex": 2,
 "key": "/foo/bar",
 "modifiedIndex": 2,
 "value": "foo bar"
 }
}

```

#### 使用 curl 读取 etcd

以下是可以使用 curl 执行的读取操作。下表列出了 curl 提供的命令选项及其语法和示例：

| 操作 | 命令语法 | 示例 |
| --- | --- | --- |

| 获取键值 | `curl -L <URL>` | `curl -L http://127.0.0.1:4001/v2/keys/foo/bar`

```
{
    "action": "get",
    "node": {
        "createdIndex": 2,
        "key": "/foo/bar",
        "modifiedIndex": 2,
        "value": "foo bar"
    }
}

```

|

| 递归地获取键值 | `curl -L <URL> ? ?recursive=true&sorted=true` | `curl -L 'http://127.0.0.1:4001/v2/keys/foo/bar?recursive=true&sorted=true'`

```
{
    "action": "get",
    "node": {
        "createdIndex": 2,
        "key": "/foo/bar",
        "modifiedIndex": 2,
        "value": "foo bar"
    }
}

```

|

#### 使用 curl 写入 etcd

以下写入操作可以通过 `curl` 执行。下表列出了 curl 提供的命令选项及其语法和示例：

| 操作 | 命令语法 | 示例 |
| --- | --- | --- |

| 设置键值 | `curl –L <URL> -XPUT -d value=<value>` | `curl -L http://127.0.0.1:4001/v2/keys/foo/bar -XPUT -d value="foo bar"`

```
{
    "action": "set",
    "node": {
        "createdIndex": 2,
        "key": "/foo/bar",
        "modifiedIndex": 2,
        "value": "foo bar "
    }
}

```

|

| 设置带过期时间的键值（秒） | `curl –L <URL> -XPUT -d value=<value> -d ttl=<value>` | `curl -L http://127.0.0.1:4001/v2/keys/foo/bar -XPUT -d value="foo bar" –d ttl=5`

```
{
    "action": "set",
    "node": {
        "createdIndex": 2,
        "expiration": "2015-12-04T12:11:11.824823581-08:00",
        "key": "/foo/bar",
        "modifiedIndex": 2,
        "ttl": 5,
        "value": "foo bar"
    }
}

```

|

| 更新键值 | `curl –L <URL> -XPUT -d value=<value>` | `curl -L http://127.0.0.1:4001/v2/keys/foo/bar -XPUT -d value="foo bar2"`

```
{
    "action": "set",
    "node": {
        "createdIndex": 3,
        "key": "/foo/bar",
        "modifiedIndex": 3,
        "value": "foo bar2 "
    },
    "prevNode": {
        "createdIndex": 2,
        "key": "/foo/bar",
        "modifiedIndex": 2,
        "value": "foo bar "
    }
}

```

|

| 删除键 | `curl –L <URL> -XDELETE` | `curl -L http://127.0.0.1:4001/v2/keys/foo/bar -XDELETE`

```
{
    "action": "delete",
    "node": {
        "createdIndex": 3,
        "key": "/foo/bar",
        "modifiedIndex": 3
    },
    "prevNode": {
        "createdIndex": 2,
        "key": "/foo/bar",
        "modifiedIndex": 2,
        "value": "foo bar "
    }
}

```

|

#### 使用 curl 监听 etcd

以下是可以使用 `curl` 执行的监听操作。下表列出了 curl 提供的命令选项及其语法和示例：

| 操作 | 命令语法 | 示例 |
| --- | --- | --- |
| 监听键值的任何变化 | `curl –L <URL>?wait=true` | `curl -L http://127.0.0.1:4001/v2/keys/foo/bar?wait=true` |

#### 使用 curl 的示例

让我们看看如何使用 curl 配合我们的 `testservices`，它希望发布此服务运行的 IP 地址和端口号。

在这里，`testservices` 是添加所有新服务信息的目录。IP 是成员的 IP 地址，`5555` 是服务运行的端口（在本示例中选择）。

以下是将服务添加到指定 IP 地址和端口的命令，键值为 IP：

```
curl -L http://127.0.0.1:4001/v2/keys/testservices/ip -XPUT -d value="'172.17.8.101:5555"

{
 "action": "set",
 "node": {
 "createdIndex": 3,
 "key": "/ testservices/ip ",
 "modifiedIndex": 3,
 "value": "'172.17.8.101:5555"
 }
}

```

要获取 `testservices` 发布的参数，应使用以下命令：

```
curl -L 'http://127.0.0.1:4001/v2/keys/testservices/ip'

{
 "action": "get",
 "node": {
 "createdIndex": 4,
 "key": "/ testservices/ip ",
 "modifiedIndex": 4,
 "value": "'172.17.8.101:5555"
 }
}

```

要监听这些参数，应该使用以下命令：

```
curl -L http://127.0.0.1:4001/v2/keys/ testservices/ip'?wait=true

```

# HAProxy 和服务发现

在本节中，我们使用服务发现创建一个具有多个后端节点的 Web 服务，前端通过 HAProxy 进行负载均衡并处理服务请求。HAProxy 是一个常用于 TCP 和 HTTP 应用的负载均衡器。

让我们从理解一个典型的 HAProxy 配置开始。我们不会全面覆盖 HAProxy 配置，而是专注于与服务发现相关的配置：

```
frontend testloadbalancer
 bind *:80
 mode http
 balance roundrobin
 server testserver01 172.17.18.101:80 check
 server testserver02 172.17.18.102:80 check

```

该配置指示 HAProxy 绑定到端口 `80` 并以轮询方式将 HTTP 流量转发到服务器 `172.17.18.10` 和 `172.17.18.102`。当我们获得每个后端服务器的信息时，可以静态配置 HAProxy，设置就会生效。但假设有一个场景，其中没有可用的 IP 信息。例如，当 IP 动态分配时，或者随着服务器流量的增加，节点数量不断增加。我们可以使用服务发现来动态地保持 HAProxy 更新，处理后端服务器的增删。接下来，我们将熟悉另一个工具——confd，我们将使用 confd 作为服务发现工具。confd 能够监视 etcd 键值存储，然后基于模板准备配置文件并将其复制到应用所需的位置，最后调用命令让应用重新加载配置。

confd 需要在目录 `/etc/confd/templates` 中一个模板应用配置文件，并在目录 `/etc/confd/conf.d` 中一个 confd 配置文件。

以下是 `testconfd.toml` 文件的配置，用于 confd：

```
[template]
src = "haproxy.cfg.tmpl"
dest = "/etc/haproxy/haproxy.cfg"
keys = [
  "/testservice/backend",
]
reload_cmd = "/usr/sbin/service haproxy reload"
```

该配置文件提到 HAProxy 模板文件名为 `haproxy.cfg.tmpl`。根据模板文件准备的配置文件必须复制到 `/etc/haproxy/haproxy.cfg`。配置文件还提到 etcd 键为 `/testservice/backend`。最后，它会调用命令来重新加载 HAProxy。

以下是我们之前见过的 HAProxy 配置文件 `haproxy.cfg.tmpl` 的模板文件内容：

```
frontend testloadbalancer
 bind *:80
 mode http
 balance roundrobin
 {{range $serveraddr := . testservice backend}}
 server {{Base $serveraddr.Key}} {{$serveraddr.Value}} check
 {{end}}

```

range 指令用于遍历 etcd 键，并为每个名称准备条目。所使用的 base 指令与 Linux 的 base-name 工具有些相似。对于本章前面提到的 etcd 键，对应的条目如下：

```
server 1 172.17.18.101:55555 check
server 2 172.17.18.102:55555 check

```

现在，关于后端服务器，我们可以添加命令（`etcdctl` 或 `curl`）通过 `ExeStartPost` 更新系统 IP 地址到 etcd。然后，confd 会在后端服务器启动时更新前端 HAProxy 配置。

# 摘要

在本章中，我们理解了发现服务的重要性，尤其在 CoreOS 环境中开发服务或应用时，如何发布服务及其参数，如何监控服务状态的变化。我们还了解了广泛用于服务发现的两个重要工具：etcdctl 和 curl，并且通过一些示例学习了它们的使用。

在下一章，我们将学习如何通过服务链机制使运行在 CoreOS 集群中的不同服务能够相互通信。
