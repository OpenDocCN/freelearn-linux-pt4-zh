# 第九章：Mesos 大数据框架 2

本章将介绍如何在 Mesos 上部署重要的大数据存储框架，如 Cassandra、Elasticsearch-Logstash-Kibana（ELK）堆栈和 Kafka。

# Cassandra 在 Mesos 上的使用

本节将介绍 Cassandra，并解释如何在 Mesos 上部署 Cassandra，同时讨论在设置过程中常遇到的问题。

## Cassandra 介绍

**Cassandra** 是一个开源、可扩展的 NoSQL 数据库，完全分布式且没有单点故障，对于大多数标准使用案例具有高性能。它既支持横向扩展也支持纵向扩展。**横向扩展** 或 **扩展解决方案** 涉及通过添加更多的普通硬件节点来扩展现有集群，而 **纵向扩展** 或 **扩展升级解决方案** 则意味着通过为节点添加更多的 CPU 和内存资源来使用专用硬件。

Cassandra 是由 Facebook 工程师开发的，旨在解决收件箱搜索的使用案例，灵感来自 Google Bigtable（它为 Cassandra 的存储模型提供了基础）以及 Amazon DynamoDB（它为 Cassandra 的分布式模型提供了基础）。Cassandra 于 2008 年开源，并在 2010 年初成为 Apache 顶级项目。它提供了一种名为 **Cassandra 查询语言** 或 **CQL** 的查询语言，语法类似 SQL，用于与数据库进行通信。

Cassandra 提供了多种功能，例如：

+   高性能

+   持续的正常运行时间（无单点故障）

+   易用性

+   数据在数据中心之间的复制和分发

Cassandra 的架构采用优雅且简洁的 **环形设计**，没有任何主节点，而不是使用传统的主从或分片设计。这使得它能够提供前述所有特性和优势。

Cassandra 环形设计图如下所示（来源：[www.planetcassandra.org](http://www.planetcassandra.org)）：

![Cassandra 介绍](img/B05186_09_00.jpg)

许多公司在生产环境中使用 Cassandra，包括 Apple、Instagram、eBay、Spotify、Comcast 和 Netflix 等公司。

Cassandra 最适用于以下场景：

+   无单点故障

+   实时写入

+   灵活性

+   横向扩展

+   可靠性

+   在 NoSQL 环境中拥有明确定义的表结构

以下是一些常见的使用场景：

+   存储、管理以及对由消息应用程序生成的数据进行分析（Instagram 和 Comcast 等公司都使用 Cassandra 来处理这类数据）

+   存储用于检测欺诈活动的数据模式

+   存储用户选择和策划的项目（购物车、播放列表等）

+   推荐和个性化

**性能基准**

以下由独立数据库公司进行的性能基准测试显示，对于混合的操作和分析工作负载，Cassandra 明显优于其他开源 NoSQL 技术（来源：[www.datastax.com](http://www.datastax.com)）：

![Cassandra 介绍](img/B05186_09_01.jpg)

## 在 Mesos 上设置 Cassandra

本节介绍了在 Mesos 上部署 Cassandra 的过程。推荐的 Cassandra 在 Mesos 上的部署方式是通过 Marathon。在编写本书时，Cassandra 在 Mesos 上的配置仍处于实验阶段，此处描述的配置可能在未来的版本中发生变化。

Mesosphere 团队已经将所需的 JAR 文件和 Cassandra 执行器打包在一个 tar 包中，可以通过以下 JSON 代码直接提交给 Mesos 并通过 Marathon 运行：

```
  {
    "healthChecks": [
      {
        "timeoutSeconds": 5,
        "protocol": "HTTP",
        "portIndex": 0,
        "path": "/health/cluster",
        "maxConsecutiveFailures": 0,
        "intervalSeconds": 30,
        "gracePeriodSeconds": 120
      },
      {
        "timeoutSeconds": 5,
        "protocol": "HTTP",
        "portIndex": 0,
        "path": "/health/process",
        "maxConsecutiveFailures": 3,
        "intervalSeconds": 30,
        "gracePeriodSeconds": 120
      }
    ],
    "id": "/cassandra/dev-test",
    "instances": 1,
    "cpus": 0.5,
    "mem": 512,
    "ports": [0],
    "uris": [
      "https://downloads.mesosphere.io/cassandra-mesos/artifacts/0.2.1-SNAPSHOT-608-master-d1c2cf30c8/cassandra-mesos-0.2.1-SNAPSHOT-608-master-d1c2cf30c8.tar.gz",
      "https://downloads.mesosphere.io/java/jre-7u76-linux-x64.tar.gz"
    ],
    "env": {
      "CASSANDRA_ZK_TIMEOUT_MS": "10000",
      "CASSANDRA_HEALTH_CHECK_INTERVAL_SECONDS": "60",
      "MESOS_ZK": "zk://localhost:2181/mesos",
      "JAVA_OPTS": "-Xms256m -Xmx256m",
      "CASSANDRA_CLUSTER_NAME": "dev-test",
      "CASSANDRA_ZK": "zk://localhost:2181/cassandra-mesos",
      "CASSANDRA_NODE_COUNT": "3",
      "CASSANDRA_RESOURCE_CPU_CORES": "2.0",
      "CASSANDRA_RESOURCE_MEM_MB": "2048",
      "CASSANDRA_RESOURCE_DISK_MB": "2048"
    },
    "cmd": "$(pwd)/jre*/bin/java $JAVA_OPTS -classpath cassandra-mesos-framework.jar io.mesosphere.mesos.frameworks.cassandra.framework.Main"
  }
```

编辑 JSON 代码，指向`MESOS_ZK`及任何其他需要更改的参数，将此 JSON 代码保存为`cassandra-mesos.json`，然后通过以下命令将其提交给 Marathon：

```
$ curl -X POST -H "Content-Type: application/json" -d cassandra-mesos.json http://marathon-machine:8080/v2/apps

```

一旦提交，框架将自我引导。我们还需要扩展每个 Mesos 节点管理的端口范围，以包括标准的 Cassandra 端口。我们可以在启动过程中将端口范围作为资源传递。

这是一个示例：

```
--resources='ports:[31000-32000,7000-7001,7199-7199,9042-9042,9160-9160]'
```

Mesos 上的 Cassandra 提供了一个 REST 端点，用于调整设置。我们可以通过默认端口`18080`访问此端点（除非已更改）。

## 高级配置指南

如前所述，Mesos 上的 Cassandra 通过环境变量接受运行时配置。我们可以使用以下环境变量来引导框架的配置。在初始运行后，配置会从 ZooKeeper 中存储的框架状态中读取：

```
# name of the cassandra cluster, this will be part of the framework name in Mesos
CASSANDRA_CLUSTER_NAME=dev-cluster

# Mesos ZooKeeper URL to locate leading master
MESOS_ZK=zk://localhost:2181/mesos

# ZooKeeper URL to be used to store framework state
CASSANDRA_ZK=zk://localhost:2181/cassandra-mesos

# The number of nodes in the cluster (default 3)
CASSANDRA_NODE_COUNT=3

# The number of seed nodes in the cluster (default 2)
# set this to 1, if you only want to spawn one node
CASSANDRA_SEED_COUNT=2

# The number of CPU Cores for each Cassandra Node (default 2.0)
CASSANDRA_RESOURCE_CPU_CORES=2.0

# The number of Megabytes of RAM for each Cassandra Node (default 2048)
CASSANDRA_RESOURCE_MEM_MB=2048

# The number of Megabytes of Disk for each Cassandra Node (default 2048)
CASSANDRA_RESOURCE_DISK_MB=2048

# The number of seconds between each health check of the Cassandra node (default 60)
CASSANDRA_HEALTH_CHECK_INTERVAL_SECONDS=60

# The default bootstrap grace time - the minimum interval between two node starts
# You may set this to a lower value in pure local development environments.
CASSANDRA_BOOTSTRAP_GRACE_TIME_SECONDS=120

# The number of seconds that should be used as the mesos framework timeout (default 604800 seconds / 7 days)
CASSANDRA_FAILOVER_TIMEOUT_SECONDS=604800

# The mesos role to used to reserve resources (default *). If this is set, the framework accepts offers that have resources for that role or the default role *
CASSANDRA_FRAMEWORK_MESOS_ROLE=*

# A pre-defined data directory specifying where Cassandra should write its data. 
# Ensure that this directory can be created by the user the framework is running as (default. [mesos sandbox]).
# NOTE:
# This field is slated to be removed and the framework will be able to allocate the data volume itself.
CASSANDRA_DATA_DIRECTORY=.
```

这里有一些参考资料：

+   [`github.com/mesosphere/cassandra-mesos`](https://github.com/mesosphere/cassandra-mesos)

+   [`mesosphere.github.io/cassandra-mesos/`](http://mesosphere.github.io/cassandra-mesos/)

# Mesos 上的 Elasticsearch-Logstash-Kibana（ELK）栈

本节将介绍**Elasticsearch-Logstash-Kibana**（**ELK**）栈，并解释如何在 Mesos 上设置它，同时讨论在设置过程中常见的问题。

## Elasticsearch、Logstash 和 Kibana 简介

ELK 栈，**Elasticsearch**、**Logstash**和**Kibana**的组合，是一个端到端的**日志分析**解决方案。Elasticsearch 提供搜索功能，Logstash 是日志管理软件，而 Kibana 作为可视化层。该栈由名为**Elastic**的公司提供商业支持。

### Elasticsearch

Elasticsearch 是一个基于 Lucene 的开源分布式搜索引擎，旨在实现高可扩展性和快速的搜索查询响应时间。它通过提供一个强大的 REST API 来简化 Lucene 的使用，Lucene 是一个高性能的搜索引擎库。以下是 Elasticsearch 中的一些重要概念：

+   **文档**：这是存储在索引中的一个 JSON 对象

+   **索引**：这是一个文档集合

+   **类型**：这是一个表示文档类别的索引逻辑分区

+   **字段**：这是文档中的一个键值对

+   **映射**：用于将每个字段与其数据类型进行映射

+   **Shard**：这是存储索引数据的物理位置（数据存储在一个主分片上，并复制到一组副本分片上）

### Logstash

这是一个收集和处理由各种系统生成的日志事件的工具。它包括一组丰富的输入和输出连接器，用于获取日志并使其可供分析。一些重要功能包括：

+   将日志转换为通用格式以便于使用的能力

+   处理多种日志格式的能力，包括自定义格式

+   丰富的输入和输出连接器集

### Kibana

这是一个基于 Elasticsearch 的数据可视化工具，具有多种图表和仪表板功能。它依赖于存储在 Elasticsearch 索引中的数据，完全使用 HTML 和 JavaScript 开发。其一些最重要的功能包括：

+   用于仪表板构建的图形用户界面

+   丰富的图表集（地图、饼图、直方图等）

+   将图表嵌入用户应用程序的能力

### ELK 栈数据管道

查看以下图示（来源：*Learning ELK Stack*，Packt 出版社）：

![ELK 栈数据管道](img/B05186_09_02.jpg)

在标准 ELK 栈管道中，各种应用程序服务器的日志通过 Logstash 传输到中央索引模块。该索引模块随后将输出传输到 Elasticsearch 集群，在那里可以直接查询或通过 Kibana 在仪表板中进行可视化。

## 在 Mesos 上设置 Elasticsearch-Logstash-Kibana

本节解释如何在 Mesos 上设置 Elasticsearch、Logstash 和 Kibana。我们将首先介绍如何在 Mesos 上设置 Elasticsearch，然后是 Logstash 和 Kibana。

### Mesos 上的 Elasticsearch

我们将使用 Marathon 来部署 Elasticsearch，这可以通过两种方式完成：通过 Docker 镜像（强烈推荐），以及通过`elasticsearch-mesos jar`。这两种方式将在以下部分进行解释。

我们可以使用以下 Marathon 文件将 Elasticsearch 部署到 Mesos 上。它使用 Docker 镜像：

```
{
  "id": "elasticsearch-mesos-scheduler",
  "container": {
    "docker": {
      "image": "mesos/elasticsearch-scheduler",
      "network": "HOST"
    }
},
"args": ["--zookeeperMesosUrl", "zk://zookeeper-node:2181/mesos"],
  "cpus": 0.2,
  "mem": 512.0,
  "env": {
    "JAVA_OPTS": "-Xms128m -Xmx256m"
  },
  "instances": 1
}
```

确保将`zookeeper-node`更改为集群中 ZooKeeper 节点的地址。我们可以将其保存到`elasticsearch.json`文件，并通过以下命令在 Marathon 上部署：

```
$ curl -k -XPOST -d @elasticsearch.json -H "Content-Type: application/json" http://marathon-machine:8080/v2/apps

```

如前所述，我们也可以使用 JAR 文件通过以下 Marathon 文件将 Elasticsearch 部署到 Mesos 上：

```
{
  "id": "elasticsearch",
  "cpus": 0.2,
  "mem": 512,
  "instances": 1,
  "cmd": "java -jar scheduler-0.7.0.jar --frameworkUseDocker false --zookeeperMesosUrl zk://10.0.0.254:2181 --frameworkName elasticsearch --elasticsearchClusterName mesos-elasticsearch --elasticsearchCpu 1 --elasticsearchRam 1024 --elasticsearchDisk 1024 --elasticsearchNodes 3 --elasticsearchSettingsLocation /home/ubuntu/elasticsearch.yml",
  "uris": ["https://github.com/mesos/elasticsearch/releases/download/0.7.0/scheduler-0.7.0.jar"],
  "env": {
    "JAVA_OPTS": "-Xms256m -Xmx512m"
  },
  "ports": [31100],
  "requirePorts": true,
  "healthChecks": [
    {
     "gracePeriodSeconds": 120,
      "intervalSeconds": 10,
      "maxConsecutiveFailures": 6,
      "path": "/",
      "portIndex": 0,
      "protocol": "HTTP",
      "timeoutSeconds": 5
    }
  ]
}
```

在这两种情况下，都需要`JAVA_OPTS`环境变量，如果没有设置，可能会导致 Java 堆内存空间的问题。我们可以将其保存为`elasticsearch.json`文件，并通过以下命令提交给 Marathon：

```
$ curl -k -XPOST -d @elasticsearch.json -H "Content-Type: application/json" http://MARATHON_IP_ADDRESS:8080/v2/apps

```

Docker 镜像和 JAR 文件都需要以下命令行参数，类似于`--zookeeperMesosUrl`参数：

```
    --dataDir
         The host data directory used by Docker volumes in the executors. [DOCKER MODE ONLY]
         Default: /var/lib/mesos/slave/elasticsearch

    --elasticsearchClusterName
         Name of the Elasticsearch cluster
         Default: mesos-ha

    --elasticsearchCpu
         The amount of CPU resource to allocate to the Elasticsearch instance.
         Default: 1.0

    --elasticsearchDisk
         The amount of Disk resource to allocate to the Elasticsearch instance
         (MB).
         Default: 1024.0

    --elasticsearchExecutorCpu
         The amount of CPU resource to allocate to the Elasticsearch executor.
         Default: 0.1

    --elasticsearchExecutorRam
         The amount of ram resource to allocate to the Elasticsearch executor
         (MB).
         Default: 32.0

    --elasticsearchNodes
         Number of Elasticsearch instances.
         Default: 3

    --elasticsearchPorts
         User specified Elasticsearch HTTP and transport ports. [NOT RECOMMENDED]
         Default: <empty string>

    --elasticsearchRam
         The amount of ram resource to allocate to the Elasticsearch instance
         (MB).
         Default: 256.0

    --elasticsearchSettingsLocation
         Path or URL to Elasticsearch yml settings file. [In docker mode file must be in /tmp/config] E.g. '/tmp/config/elasticsearch.yml' or 'https://gist.githubusercontent.com/mmaloney/5e1da5daa58b70a3a671/raw/elasticsearch.yml'
         Default: <empty string>

    --executorForcePullImage
         Option to force pull the executor image. [DOCKER MODE ONLY]
         Default: false

    --executorImage
         The docker executor image to use. E.g. 'elasticsearch:latest' [DOCKER
         MODE ONLY]
         Default: elasticsearch:latest

    --executorName
         The name given to the executor task.
         Default: elasticsearch-executor

    --frameworkFailoverTimeout
         The time before Mesos kills a scheduler and tasks if it has not recovered
         (ms).
         Default: 2592000.0

    --frameworkName
         The name given to the framework.
         Default: elasticsearch

    --frameworkPrincipal
         The principal to use when registering the framework (username).
         Default: <empty string>

    --frameworkRole
         Used to group frameworks for allocation decisions, depending on the
         allocation policy being used.
         Default: *

    --frameworkSecretPath
         The path to the file which contains the secret for the principal
         (password). Password in file must not have a newline.
         Default: <empty string>

    --frameworkUseDocker
         The framework will use docker if true, or jar files if false. If false, the user must ensure that the scheduler jar is available to all slaves.
         Default: true

    --javaHome
         When starting in jar mode, if java is not on the path, you can specify
         the path here. [JAR MODE ONLY]
         Default: <empty string>

    --useIpAddress
         If true, the framework will resolve the local ip address. If false, it
         uses the hostname.
         Default: false

    --webUiPort
         TCP port for web ui interface.
         Default: 31100

    --zookeeperMesosTimeout
         The timeout for connecting to zookeeper for Mesos (ms).
         Default: 20000

    * --zookeeperMesosUrl
         Zookeeper urls for Mesos in the format zk://IP:PORT,IP:PORT,...)
         Default: zk://mesos.master:2181
```

### Mesos 上的 Logstash

本节介绍如何在 Mesos 上运行 Logstash。Logstash 部署到集群后，任何在 Mesos 上运行的程序都可以记录事件，事件随后通过 Logstash 被传递并发送到中央日志位置。

我们可以将 Logstash 作为 Marathon 应用程序运行，并通过以下 Marathon 文件在 Mesos 上部署：

```
{
  "id": "/logstash",
  "cpus": 1,
  "mem": 1024.0,
  "instances": 1,
  "container": {
    "type": "DOCKER",
    "docker": {
      "image": "mesos/logstash-scheduler:0.0.6",
      "network": "HOST"
    }
  },
  "env": {
    "ZK_URL": "zk://123.0.0.12:5181/logstash",
    "ZK_TIMEOUT": "20000",
    "FRAMEWORK_NAME": "logstash",
    "FAILOVER_TIMEOUT": "60",
    "MESOS_ROLE": "logstash",
    "MESOS_USER": "root",
    "LOGSTASH_HEAP_SIZE": "64",
    "LOGSTASH_ELASTICSEARCH_URL": "http://elasticsearch.service.consul:1234",
    "EXECUTOR_CPUS": "0.5",
    "EXECUTOR_HEAP_SIZE": "128",
    "ENABLE_FAILOVER": "false",
    "ENABLE_COLLECTD": "true",
    "ENABLE_SYSLOG": "true",
    "ENABLE_FILE": "true",
    "ENABLE_DOCKER": "true",
    "EXECUTOR_FILE_PATH": "/var/log/*,/home/jhf/example.log"
  }
}
```

在这里，我们使用了用于部署的 Docker 镜像，其配置可以根据您的集群规格进行更改。将前面的文件保存为 `logstash.json` 并使用以下命令提交给 Marathon：

```
$ curl -k -XPOST -d @logstash.json -H "Content-Type: application/json" http://MARATHON_IP_ADDRESS:8080/v2/apps

```

#### Mesos 上的 Logstash 配置

Logstash 和 Elasticsearch 经 Mesos 版本 0.25.0 及更高版本测试。我们需要将 Logstash 添加到每个 Mesos 主节点的角色列表中。这可以通过以下命令完成：

```
$ sudo echo logstash > /etc/mesos-master/roles

```

如果 Logstash 的目的是监控 `syslog`（一种消息日志标准），则需要在集群中每个 Mesos 节点的资源列表中添加 TCP 和 UDP 端口 `514`。可以通过在 `/etc/mesos-slave/resources` 文件中添加以下条目来实现：

```
ports(logstash):[514-514]
```

要监控 `collectd`，我们需要通过在 `/etc/mesos-slave/resources` 文件中添加以下行来将 TCP 和 UDP 端口 `25826` 添加到 Logstash 角色的资源中：

```
ports(logstash):[25826-25826]
```

### Kibana 在 Mesos 上

如果我们在 Mesos 上运行 Kibana，那么每个 Kibana 实例将作为 Docker 镜像在 Mesos 集群中运行。对于每个 Elasticsearch 实例，可以部署一个或多个 Kibana 实例来为用户提供服务。

我们可以从以下代码库克隆 Mesos 项目上的 Kibana：

```
$ git clone https://github.com/mesos/kibana

```

使用以下命令构建项目：

```
$ cd kibana
$ gradlew jar

```

这将生成 Kibana JAR 文件（`kibana.jar`）。

一旦 `kibana.jar` 文件生成，我们可以使用以下命令进行部署：

```
$ java -jar /path/to/kibana.jar -zk zk://zookeeper:2181/mesos -v 4.3.1 -es http://es-host:9200

```

在这里，`-zk` 代表 ZooKeeper URI，`-es` 指向我们在前一节中部署的 Elasticsearch 端点。请根据需要设置它们。

`kibana.jar` 文件还支持以下命令行选项：

| 短关键词 | 关键词 | 定义 |
| --- | --- | --- |
| `-zk` | `-zookeeper` | 这是 Mesos ZooKeeper 的 URL（必需） |
| `-di` | `-dockerimage` | 这是要使用的 Docker 镜像名称（默认值为 `kibana`） |
| `-v` | `-version` | 这是要使用的 Kibana Docker 镜像的版本（默认值为 `latest`） |
| `-mem` | `-requiredMem` | 这是分配给单个 Kibana 实例的内存量（单位：MB，默认值为 `128`） |
| `-cpu` | `-requiredCpu` | 这是分配给单个 Kibana 实例的 CPU 数量（默认值为 `0.1`） |
| `-disk` | `-requiredDisk` | 这是分配给单个 Kibana 实例的磁盘空间量（单位：MB，默认值为 `25`） |
| `-es` | `-elasticsearch` | 这些是启动时用于启动 Kibana 的 Elasticsearch URL |

这里有一些参考资料：

+   [`mesos-elasticsearch.readthedocs.org/en/latest/#elasticsearch-mesos-framework`](http://mesos-elasticsearch.readthedocs.org/en/latest/#elasticsearch-mesos-framework)

+   [`github.com/mesos/logstash`](https://github.com/mesos/logstash)

+   [`github.com/mesos/kibana`](https://github.com/mesos/kibana)

# Kafka on Mesos

本节将介绍 Kafka，并解释如何在 Mesos 上进行设置，同时讨论在设置过程中常见的问题。

## Kafka 简介

Kafka 是一个分布式发布-订阅消息系统，旨在提供速度、可扩展性、可靠性和耐久性。Kafka 中使用的一些关键术语如下所示：

+   **主题**：这些是 Kafka 维护消息流的类别。

+   **生产者**：这些是上游进程，它们将消息发送到特定的 Kafka 主题。

+   **消费者**：这些是下游进程，它们监听主题中传入的消息，并根据要求处理它们。

+   **代理**：Kafka 集群中的每个节点称为代理。

请查看以下 Kafka 的高层次示意图（来源：[`kafka.apache.org/documentation.html#introduction`](http://kafka.apache.org/documentation.html#introduction)）：

![Kafka 介绍](img/B05186_09_03.jpg)

Kafka 集群为每个主题维护一个分区日志，类似于以下内容（来源：[`kafka.apache.org/documentation.html#intro_topics`](http://kafka.apache.org/documentation.html#intro_topics)）：

![Kafka 介绍](img/B05186_09_04.jpg)

## Kafka 的使用案例

这里描述了一些 Kafka 的重要用途：

+   **网站活动跟踪**：网站活动事件，例如页面浏览和用户搜索，可以由 Web 应用程序发送到 Kafka 主题。下游处理系统可以订阅这些主题并消费消息，用于批量分析、监控、实时仪表盘等用例。

+   **日志聚合**：Kafka 作为传统日志聚合系统的替代方案。可以从各种服务收集物理日志文件并推送到不同的 Kafka 主题，在那里不同的消费者可以读取和处理它们。Kafka 抽象了文件细节，从而实现了更快的处理并支持多种数据源。

+   **流处理**：框架，例如 Spark Streaming，可以从 Kafka 主题中消费数据，按照要求处理，然后将处理后的输出发布到另一个 Kafka 主题，在那里该输出可以被其他应用程序消费。

## Kafka 设置

在 Mesos 上安装 Kafka 之前，确保机器上已安装以下应用程序：

+   Java 版本 7 或更高版本（[`openjdk.java.net/install/`](http://openjdk.java.net/install/)）

+   Gradle（[`gradle.org/installation`](http://gradle.org/installation)）

我们可以从以下仓库克隆并构建 Kafka on Mesos 项目：

```
$ git clone https://github.com/mesos/kafka
$ cd kafka
$ ./gradlew jar

```

我们还需要 Kafka 执行器，可以通过以下命令下载：

```
$ wget https://archive.apache.org/dist/kafka/0.8.2.2/kafka_2.10-0.8.2.2.tgz

```

我们还需要设置以下环境变量，以指向`libmesos.so`文件：

```
$ export MESOS_NATIVE_JAVA_LIBRARY=/usr/local/lib/libmesos.so

```

一旦这些设置好，我们可以使用`kafka-mesos.sh`脚本在 Mesos 上启动并配置 Kafka。在此之前，我们需要创建`kafka-mesos.properties`文件，其内容如下：

```
storage=file:kafka-mesos.json
master=zk://master:2181/mesos
zk=master:2181
api=http://master:7000
```

如果我们不希望每次都将参数传递给调度器，可以使用该文件来配置调度器（`kafka-mesos.sh`）。调度器支持以下命令行参数：

| 选项 | 描述 |
| --- | --- |
| `--api` | 这是 API URL，例如`http://master:7000`。 |
| `--bind-address` | 这是调度器绑定地址（例如 master，`0.0.0.0`，`192.168.50.*`，以及`if:eth1`）。默认值是`all`。 |
| `--debug <Boolean>` | 这是调试模式。默认值是`false`。 |
| `--framework-name` | 这是框架名称。默认值是`kafka`。 |
| `--framework-role` | 这是框架角色。默认值是`*`。 |
| `--framework-timeout` | 这是框架超时时间（30s、1m 或 1h）。默认值是`30d`。 |
| `--jre` | 这是 JRE 压缩文件（`jre-7-openjdk.zip`）。默认值是`none`。 |
| `--log` | 这是要使用的日志文件。默认值是`stdout`。 |
| `--master` | 这些是主连接设置。一些示例如下：`- master:5050` `- master:5050,master2:5050` `- zk://master:2181/mesos` `- zk://username:password@master:2181` `- zk://master:2181,master2:2181/mesos` |
| `--principal` | 这是用于注册框架的主体（用户名）。默认值是`none`。 |
| `--secret` | 这是用于注册框架的密码（密码）。默认值是`none`。 |
| `--storage` | 这是集群状态的存储位置。一些示例如下：`- file:kafka-mesos.json` `- zk:/kafka-mesos` 默认值是`file:kafka-mesos.json`。 |
| `--user` | 这是运行任务的 Mesos 用户。默认值是`none`。 |
| `--zk` | 这是 Kafka 的`zookeeper.connect`。一些示例如下：`- master:2181` `- master:2181,master2:2181` |

现在，我们可以使用调度器通过以下命令运行 Kafka 调度器：

```
#Start the kafka scheduler
$ ./kafka-mesos.sh scheduler

```

接下来，我们需要做的事情是以默认设置启动一个 Kafka 代理。这可以通过以下命令完成：

```
$ ./kafka-mesos.sh broker add 0

broker added:   id: 0
 active: false
 state: stopped
 resources: cpus:1.00, mem:2048, heap:1024, port:auto
 failover: delay:1m, max-delay:10m
 stickiness: period:10m

```

此时，我们的 Kafka 集群将有一个代理尚未启动。我们可以通过以下命令验证这一点：

```
$ ./kafka-mesos.sh broker list

broker:
 id: 0
 active: false
 state: stopped
 resources: cpus:1.00, mem:2048, heap:1024, port:auto
 failover: delay:1m, max-delay:10m
 stickiness: period:10m

```

我们现在可以使用以下命令启动这个代理：

```
$ ./kafka-mesos.sh broker start 0

broker started:
 id: 0
 active: true
 state: running
 resources: cpus:1.00, mem:2048, heap:1024, port:auto
 failover: delay:1m, max-delay:10m
 stickiness: period:10m, hostname:slave0
 task:
 id: broker-0-d2d94520-2f3e-4779-b276-771b4843043c
 running: true
 endpoint: 192.168.25.62:31000
 attributes: rack=r1

```

如果显示上面的输出，则我们的代理已经准备好生成和消费消息。我们现在可以使用`kafkacat`来测试这个设置。

`kafkacat`可以通过以下命令安装到系统中：

```
$ sudo apt-get install kafkacat

$ echo "test" |kafkacat -P -b "192.168.25.62:31000" -t testTopic -p 0

```

既然我们已经将测试推送到代理上，我们可以通过以下命令将其读取回来：

```
$ kafkacat -C -b "192.168.25.62:31000" -t testTopic -p 0 -e
test

```

现在，让我们来看一下如何一次性向集群添加更多代理。运行以下命令：

```
$ ./kafka-mesos.sh broker add 0..2 --heap 1024 --mem 2048

```

前面的命令将会向集群添加三个`kafka`代理，输出如下：

```
brokers added:

 id: 0
 active: false
 state: stopped
 resources: cpus:1.00, mem:2048, heap:1024, port:auto
 failover: delay:1m, max-delay:10m
 stickiness: period:10m

 id: 1
 active: false
 state: stopped
 resources: cpus:1.00, mem:2048, heap:1024, port:auto
 failover: delay:1m, max-delay:10m
 stickiness: period:10m

 id: 2
 active: false
 state: stopped
 resources: cpus:1.00, mem:2048, heap:1024, port:auto
 failover: delay:1m, max-delay:10m
 stickiness: period:10m

```

我们可以通过以下命令一次性启动所有三个代理：

```
$   ./kafka-mesos.sh broker start 0..2

brokers started:

 id: 0
 active: true
 state: running
 resources: cpus:1.00, mem:2048, heap:1024, port:auto
 failover: delay:1m, max-delay:10m
 stickiness: period:10m, hostname:slave0
 task:
 id: broker-0-d2d94520-2f3e-4779-b276-771b4843043c
 running: true
 endpoint: 192.168.25.62:31000
 attributes: rack=r1

 id: 1
 active: true
 state: running 

 id: 2
 active: true
 state: running 

```

如果我们需要更改 Kafka 日志存储数据的位置，我们需要首先停止特定代理，然后使用以下命令更新位置：

```
$ ./kafka-mesos.sh broker stop 0

broker stopped:
 id: 0
 active: false
 state: stopped
 resources: cpus:1.00, mem:2048, heap:1024, port:auto
 failover: delay:1m, max-delay:10m
 stickiness: period:10m, hostname:slave0, expires:2015-07-10 15:51:43+03

$ ./kafka-mesos.sh broker update 0 --options log.dirs=/mnt/kafka/broker0

broker updated:
 id: 0
 active: false
 state: stopped
 resources: cpus:1.00, mem:2048, heap:1024, port:auto
 options: log.dirs=/mnt/kafka/broker0
 failover: delay:1m, max-delay:10m
 stickiness: period:10m, hostname:slave0, expires:2015-07-10 15:51:43+03

```

完成后，我们可以使用以下命令重新启动代理：

```
$ ./kafka-mesos.sh broker start 0

broker started:
 id: 0
 active: true
 state: running
 resources: cpus:1.00, mem:2048, heap:1024, port:auto
 failover: delay:1m, max-delay:10m
 stickiness: period:10m, hostname:slave0
 task:
 id: broker-0-d2d94520-2f3e-4779-b276-771b4843043c
 running: true
 endpoint: 192.168.25.62:31000
 attributes: rack=r1

```

### Kafka 日志管理

我们可以通过以下命令获取集群中任何代理的日志的最后 100 行（`stdout` - 默认或`stderr`）：

```
$ ./kafka-mesos.sh broker log 0

```

如果我们需要从`stderr`文件读取，则可以使用以下命令：

```
$ ./kafka-mesos.sh broker log 0 --name stderr

```

我们可以通过将文件名传递给`--name`选项来读取`kafka-*/log/`目录中的任何文件。例如，如果我们需要读取`server.log`，可以使用以下命令：

```
$ ./kafka-mesos.sh broker log 0 --name server.log

```

此外，如果我们需要从日志中读取更多行数，可以使用`--lines`选项，方法如下：

```
$ ./kafka-mesos.sh broker log 0 --name server.log --lines 200

```

## 高级配置指南

以下是在集群中*添加*代理时可用的配置选项：

```
$ ./kafka-mesos.sh help broker add
Add broker
Usage: broker add <broker-expr> [options]
Option        Description
--bind-address        broker bind address (broker0, 192.168.50.*, if:eth1). Default - auto
--constraints         constraints (hostname=like:master,rack=like:1.*). See below.
--cpus <Double>       cpu amount (0.5, 1, 2)
--failover-delay      failover delay (10s, 5m, 3h)
--failover-max-delay  max failover delay. See failoverDelay.
--failover-max-tries  max failover tries. Default - none
--heap <Long>         heap amount in Mb
--jvm-options         jvm options string (-Xms128m -XX:PermSize=48m)
--log4j-options       log4j options or file. Examples
 log4j.logger.kafka=DEBUG\, kafkaAppender
 file:log4j.properties
--mem <Long>          mem amount in Mb
--options             options or file. Examples:
 log.dirs=/tmp/kafka/$id,num.io.threads=16
 file:server.properties
--port                port or range (31092, 31090..31100). Default - auto
--stickiness-period   stickiness period to preserve same node for broker (5m, 10m, 1h)
--volume              pre-reserved persistent volume id

Generic        Options
Option         Description
------  -----------
--api      Api url. Example: http://master:7000broker-expr examples:

 0      - broker 0
 0,1    - brokers 0,1
 0..2   - brokers 0,1,2
 0,1..2 - brokers 0,1,2
 *      - any broker

attribute filtering:
 *[rack=r1]            - any broker having rack=r1
 *[hostname=slave*]    - any broker on host with name starting with 'slave'
 0..4[rack=r1,dc=dc1]  - any broker having rack=r1 and dc=dc1

constraint examples:
 like:master     - value equals 'master'
 unlike:master   - value not equals 'master'
 like:slave.*    - value starts with 'slave'
 unique          - all values are unique
 cluster         - all values are the same
 cluster:master  - value equals 'master'
 groupBy         - all values are the same
 groupBy:3       - all values are within 3 different groups

```

现在我们来看一下启动代理时可用的选项：

```
$ ./kafka-mesos.sh help broker start
Start broker
Usage: broker start <broker-expr> [options]
Option     Description
------     -----------
--timeout  timeout (30s, 1m, 1h). 0s - no timeout

Generic  Options
Option   Description
------   -----------
--api    Api url. Example: http://master:7000

broker    - expr examples:
 0       - broker 0
 0,1     - brokers 0,1
 0..2    - brokers 0,1,2
 0,1..2  - brokers 0,1,2
 *       - any broker

attribute filtering:
 *[rack=r1]           - any broker having rack=r1
 *[hostname=slave*]   - any broker on host with name starting with 'slave'
 0..4[rack=r1,dc=dc1] - any broker having rack=r1 and dc=dc1

```

以下是在集群中*更新*代理时可用的配置选项：

```
$ ./kafka-mesos.sh help broker update

Update broker
Usage: broker update <broker-expr> [options]

Option                Description
------                -----------
--bind-address        broker bind address (broker0, 192.168.50.*, if:eth1). Default - auto
--constraints         constraints (hostname=like:master,rack=like:1.*). See below.
--cpus <Double>       cpu amount (0.5, 1, 2)
--failover-delay      failover delay (10s, 5m, 3h)
--failover-max-delay  max failover delay. See failoverDelay.
--failover-max-tries  max failover tries. Default - none
--heap <Long>         heap amount in Mb
--jvm-options         jvm options string (-Xms128m -XX:PermSize=48m)
--log4j-options       log4j options or file. Examples:
 log4j.logger.kafka=DEBUG\, kafkaAppender
 file:log4j.properties
--mem <Long>          mem amount in Mb
--options             options or file. Examples:
 log.dirs=/tmp/kafka/$id,num.io.threads=16
 file:server.properties
--port                port or range (31092, 31090..31100). Default - auto
--stickiness-period   stickiness period to preserve same node for broker (5m, 10m, 1h)
--volume              pre-reserved persistent volume id

Generic Options
Option  Description
------  -----------
--api   Api url. Example: http://master:7000

broker-expr examples:
 0       - broker 0
 0,1     - brokers 0,1
 0..2    - brokers 0,1,2
 0,1..2  - brokers 0,1,2
 *       - any broker

attribute filtering:
 *[rack=r1]           - any broker having rack=r1
 *[hostname=slave*]   - any broker on host with name starting with 'slave'
 0..4[rack=r1,dc=dc1] - any broker having rack=r1 and dc=dc1

constraint examples:
 like:master     - value equals 'master'
 unlike:master   - value not equals 'master'
 like:slave.*    - value starts with 'slave'
 unique          - all values are unique
 cluster         - all values are the same
 cluster:master  - value equals 'master'
 groupBy         - all values are the same
 groupBy:3       - all values are within 3 different groups

Note: use "" arg to unset an option

```

以下是在集群中停止代理时可用的配置选项：

```
$ ./kafka-mesos.sh help broker stop

Stop broker
Usage: broker stop <broker-expr> [options]

Option     Description
------     -----------
--force    forcibly stop
--timeout  timeout (30s, 1m, 1h). 0s - no timeout

Generic  Options
Option   Description
------   -----------
--api    Api url. Example: http://master:7000

broker-expr examples:
 0      - broker 0
 0,1    - brokers 0,1
 0..2   - brokers 0,1,2
 0,1..2 - brokers 0,1,2
 *      - any broker
attribute filtering:
 *[rack=r1]           - any broker having rack=r1
 *[hostname=slave*]   - any broker on host with name starting with 'slave'
 0..4[rack=r1,dc=dc1] - any broker having rack=r1 and dc=dc1

```

以下是在集群中向代理添加主题时可用的配置选项：

```
$ ./kafka-mesos.sh help topic add
Add topic
Usage: topic add <topic-expr> [options]

Option                  Description
------                  -----------
--broker                <broker-expr>. Default - *. See below.
--options               topic options. Example: flush.ms=60000,retention.ms=6000000
--partitions <Integer>  partitions count. Default - 1
--replicas <Integer>    replicas count. Default - 1

topic-expr examples:
 t0        - topic t0
 t0,t1     - topics t0, t1
 *         - any topic
 t*        - topics starting with 't'

broker-expr examples:
 0      - broker 0
 0,1    - brokers 0,1
 0..2   - brokers 0,1,2
 0,1..2 - brokers 0,1,2
 *      - any broker

```

参考资料：[`github.com/mesos/kafka`](https://github.com/mesos/kafka)。

# 总结

本章介绍了如 Cassandra、ELK 堆栈和 Kafka 等一些重要的大数据存储框架，并涵盖了如何在分布式基础设施上使用 Mesos 进行这些框架的设置、配置和管理等主题。

我希望本书已经为您提供了管理当今现代数据中心需求复杂性的所有资源。通过遵循使用您选择的 DevOps 工具部署 Mesos 集群的详细分步指南，您现在应该能够顺利处理组织的系统管理需求。
