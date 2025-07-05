# 第八章：数据库服务故障排除

数据库服务在当今企业基础设施中起着至关重要的作用，然而本章并非面向程序员、数据库管理员或开发人员。相反，接下来的内容将采用另一种视角，专门为主要处理 CentOS 7 的系统管理员和故障排除人员提供帮助。因此，让我们从探讨数据库常见问题开始，提供一个 MariaDB、MySQL 和 PostgreSQL 故障排除指南。

在本章中，我们将：

+   学习如何在 CentOS 7 上快速启动并运行 MariaDB

+   学习如何使用 MariaDB 重置和恢复 root 用户的丢失密码

+   学习如何使用`mysqltuner`调优 MariaDB 和 MySQL 服务器

+   了解如何在 MariaDB 和 MySQL 服务器之间运行指标

+   学习如何卸载 MariaDB 并恢复 MySQL 服务器

+   学习如何在 CentOS 7 上安装和配置 PostgreSQL

# 启动并运行 MariaDB

在 CentOS 的最新版本中，你会发现 MariaDB 已经取代了 MySQL。虽然可以说这两种数据库系统之间有相似之处，但同样重要的是要认识到它们是截然不同的系统。在这个前提下，通常会在安装阶段遇到问题，可能会出现以下令人不适的消息：

```
Redirecting to /bin/systemctl start  mysqld.service. 
Failed to issue method call: Unit mysqld.service failed to load: No such file or directory.

```

在这种情况下，你的第一步是验证当前已安装的内容。这可以通过运行以下一个或多个命令来实现：

```
# which mysql
# ls -la /bin/my*
# ps -ef | grep mysql

```

然而，由于输出可能会让未经训练的眼睛感到困惑，这个问题会带来一定的混淆，且它是尝试运行 MySQL 安装和启动命令时发生的。是的，这确实是一个容易犯的错误，而解决此问题的最有效方法是探索正确的 MariaDB 安装方式。

要安装 MariaDB，你应该使用以下命令：

```
# yum install mariadb mariadb-server

```

要启动 MariaDB 服务，只需执行以下命令：

```
# systemctl start mariadb.service
# systemctl enable mariadb.service

```

其输出结果将如下所示：

```
ln -s '/usr/lib/systemd/system/mariadb.service' '/etc/systemd/system/multi-user.target.wants/mariadb.service'

```

为完成安装，现在应运行以下命令：

```
# mysql_secure_installation

```

请记住，此时你尚未设置密码。因此，当系统提示时，只需按 *回车* 键表示当前没有密码，并按照提示完成安全安装过程。作为补充，你也可能会看到以下错误消息：

```
/usr/bin/mysql_secure_installation: line 379: find_mysql_client: command not found

```

此消息可以安全忽略，因为它表示 Fedora 19 相关的一个已知错误（有关 Red Hat Bugzilla - Bug 1020055 的更多信息，请参见本章末尾的参考资料）。

完整的 `mysql_secure_installation` 过程如下所示：

```
NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB. SERVERS IN PRODUCTION USE! PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user. If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none): type enter
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB root user without the proper authorisation.

Set root password? [Y/n] <Y>
New password: your-password
Re-enter new password: your-password
Password updated successfully!
Reloading privilege tables..
... Success!

By default, a MariaDB installation has an anonymous user, allowing anyone to log into MariaDB without having to have a user account created for them. This is intended only for testing, and to make the installation go a bit smoother. You should remove them before moving into a production environment.

Remove anonymous users? [Y/n] <Y>
... Success!

Normally, root should only be allowed to connect from 'localhost'. This ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] <Y>
... Success!

By default, MariaDB comes with a database named 'test' that anyone can access. This is also intended only for testing, and should be removed before moving into a production environment.

Remove test database and access to it? [Y/n] <Y>
- Dropping test database...
... Success!
- Removing privileges on test database...
... Success!

Reloading the privilege tables will ensure that all changes made so far will take effect immediately.

Reload privilege tables now? [Y/n] <Y>
... Success!

Cleaning up...

All done! If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!

```

从此时起，一切应重新变得熟悉，你可以随时通过以下命令检查 MariaDB 的状态：

```
# systemctl status mariadb.service

```

# 使用 MariaDB 重置和恢复 root 密码

重置 root 密码的操作对系统故障排除人员而言至关重要。它确实会发生——没错，比你想象的要多得多——但按照接下来的步骤操作，可以避免危机的发生。

首先，您需要像这样停止 MariaDB 服务：

```
# systemctl stop mariadb.service

```

下一步是以以下方式激活“安全模式”：

```
# mysqld_safe --skip-grant-tables --skip-networking

```

现在，运行以下序列以访问 MySQL 控制台并连接到数据库：

```
# mysql -u root
# use mysql;

```

此时，我们需要为 root 用户创建一个新密码，刷新新的权限，并像这样退出 MySQL 控制台：

```
# update user set password=PASSWORD("NEW_PASSWORD") where User='root';
# flush privileges;
# exit

```

完成这些步骤后，您可以选择重启服务器，或者以以下方式简单地停止并重新启动 MariaDB 服务：

```
# systemctl stop mariadb.service
# systemctl start mariadb.service

```

最后，您可以通过输入以下命令，使用新的 root 用户凭证访问数据库：

```
# mysql -u root -p

```

# 调优 MariaDB 和 MySQL

MYSQL tuner 是一个有用的工具，它可以连接到正在运行的数据库实例，并根据当前的工作负载提供一系列配置建议。当然，这类工具并不总能为您的系统提供完美的答案，但您应在典型负载下让数据库系统运行一两天后，再应用修改过的配置。

要安装 `mysqltuner` 包（这是 EPEL 仓库的一部分），请键入以下内容：

```
# yum install mysqltuner

```

若要从命令行运行 `mysqltuner` 包，您可以随时使用以下语法：

```
# mysqltuner

```

现在，根据您的系统配置和硬件，`mysqltuner` 的输出看起来会类似于以下内容：

```
>>  MySQLTuner 1.2.0 - Major Hayden <major@mhtx.net>
 >>  Bug reports, feature requests, and downloads at http://mysqltuner.com/
 >>  Run with '--help' for additional options and output filtering
Please enter your MySQL administrative login: <username>
Please enter your MySQL administrative password: <password>

-------- General Statistics --------------------------------------
[--] Skipped version check for MySQLTuner script
[OK] Currently running supported MySQL version 5.5.41-MariaDB
[OK] Operating on 64-bit architecture

-------- Storage Engine Statistics -------------------------------
[--] Status: +Archive -BDB +Federated +InnoDB -ISAM -NDBCluster
[--] Data in PERFORMANCE_SCHEMA tables: 0B (Tables: 17)
[!!] InnoDB is enabled but isn't being used
[OK] Total fragmented tables: 0

-------- Security Recommendations  -------------------------------
[OK] All database users have passwords assigned

-------- Performance Metrics -------------------------------------
[--] Up for: 1h 35m 23s (33 q [0.006 qps], 15 conn, TX: 16K, RX: 1K)
[--] Reads / Writes: 75% / 25%
[--] Total buffers: 288.0M global + 2.8M per thread (151 max threads)
[OK] Maximum possible memory usage: 708.0M (38% of installed RAM)
[OK] Slow queries: 0% (0/33)
[OK] Highest usage of available connections: 0% (1/151)
[OK] Key buffer size / total MyISAM indexes: 128.0M/99.0K
[!!] Key buffer hit rate: 73.3% (15 cached / 4 reads)
[!!] Query cache is disabled
[OK] Temporary tables created on disk: 0% (0 on disk / 2 total)
[!!] Thread cache is disabled
[OK] Table cache hit rate: 2700% (27 open / 1 opened)
[OK] Open file limit used: 2% (23/1K)
[OK] Table locks acquired immediately: 100% (59 immediate / 59 locks)
[!!] Connections aborted: 6%

-------- Recommendations -----------------------------------------
General recommendations:
 Add skip-innodb to MySQL configuration to disable InnoDB
 MySQL started within last 24 hours - recommendations may be inaccurate
 Enable the slow query log to troubleshoot bad queries
 Set thread_cache_size to 4 as a starting value
 Your applications are not closing MySQL connections properly
Variables to adjust:
 query_cache_size (>= 8M)
 thread_cache_size (start at 4)

```

`mysqltuner` 包旨在帮助您对 MySQL 安装的性能和稳定性进行持续调整。它的输出内容较为冗长，展示了当前的配置变量，并附带一系列当前状态数据，总结了一些常见的性能建议。

建议在对数据库进行任何更改之前，让数据库在典型条件下运行至少 24 小时。然而，如果您希望利用这些建议中的一项或多项，您可以将它们添加到现有的 MySQL 配置文件中，或者与之集成：

```
# nano /etc/my.cnf

```

完成后，您需要重启相关的数据库服务，以便利用所做的更改。

您可以通过输入以下命令来执行此操作：

```
# systemctl restart mariadb.service

```

然后使用以下命令确认数据库的状态：

```
# systemctl status mariadb.service

```

其输出可能看起来像这样：

```
mariadb.service - MariaDB database server
 Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled)
 Active: active (running) since Mon 2015-05-04 07:10:58 EDT; 9s ago
 Process: 4756 ExecStartPost=/usr/libexec/mariadb-wait-ready $MAINPID (code=exited, status=0/SUCCESS)
 Process: 4726 ExecStartPre=/usr/libexec/mariadb-prepare-db-dir %n (code=exited, status=0/SUCCESS)
 Main PID: 4755 (mysqld_safe)
 CGroup: /system.slice/mariadb.service
 ├─4755 /bin/sh /usr/bin/mysqld_safe --basedir=/usr
 └─4911 /usr/libexec/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib64/mysql/plugin --log-error=/...

May 04 07:10:56 centos7 systemd[1]: Starting MariaDB database server...
May 04 07:10:56 centos7 mysqld_safe[4755]: 150504 07:10:56 mysqld_safe Logging to '/var/log/mariadb/mariadb.log'.
May 04 07:10:56 centos7 mysqld_safe[4755]: 150504 07:10:56 mysqld_safe Starting mysqld daemon with databases from /var/lib/mysql
May 04 07:10:58 centos7 systemd[1]: Started MariaDB database server.

```

记住，由于系统需求和数据库需求是不断变化的，您可以根据需要随时运行 `mysqltuner`，查看是否有新的建议。

# 从 MariaDB 和 MySQL 获取指标

指标不仅可以帮助您对数据库服务器进行性能分析，还能为异常行为提供证据。对于故障排除人员来说，这类数据非常重要，您可以通过运行以下命令来获取这些数据：

```
# mysqladmin -u root -p status

```

输出将提供以下信息：

+   `运行时间`: 该值表示数据库服务器已经运行的秒数。

+   `线程`: 该值表示当前连接的客户端数量。

+   `问题`: 该值表示自数据库服务器启动以来服务的查询数量。

+   `慢查询`: 该值表示超过 `long_query_time` 的查询数量。

+   `打开`: 该值表示已提供给客户端的表数量。

+   `刷新表`: 该值表示数据库服务器服务的刷新请求数量，包括 `flush`、`refresh` 和 `reload` 命令。

+   `打开的表`: 该值表示当前打开的表数量。

+   `每秒查询平均`: 该值表示数据库服务器每秒接收到的查询数量。

然而，由于前面的命令缺少详细信息，你可以通过使用以下语法获得更多细节：

```
# mysqladmin -u root -p extended-status

```

可以通过使用以下命令获得替代或实时方法：

```
# mysqladmin -i 10 -u root -p processlist

```

请注意使用了`-i`选项。这表示命令将在 10 秒后刷新。基于此，你可以建立数据库服务器事件的实时监控，从而识别、捕捉并终止可能导致系统整体变慢的查询。

# 返回 MySQL

在某些环境下，你可能希望 CentOS 7 使用 MySQL，而不是安装 MariaDB。为此，你需要确保 MariaDB 没有被安装，可以通过运行以下命令来实现：

```
# yum remove mysql-server mysql-libs mysql-devel mysql*

```

你现在应该检查是否已经卸载，使用以下命令确认：

```
# rpm -qa | grep mysql

```

要开始安装 MySQL，你应该从[`dev.mysql.com/downloads/repo/yum/`](http://dev.mysql.com/downloads/repo/yum/)下载 YUM 仓库配置文件。

现在，你不需要账户来下载这个文件，但对于那些不想浏览 Oracle 网站的人，在写这本书时，你可以跳过上述过程，使用以下语法：

```
# rpm -Uvh http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm

```

然后，你可以像这样运行适当的安装命令：

```
# yum install mysql-community-server

```

通过运行以下命令，确保 MySQL 守护进程在启动时自动启动：

```
# systemctl enable mysqld

```

现在，你可以启动服务器：

```
# systemctl start mysqld

```

然后，按常规方式运行 MySQL 安全安装程序：

```
# mysql_secure_installation

```

最后，重新运行以下命令将确认该过程是否成功：

```
# ps -ef | grep mysql

```

# 安装和配置 PostgreSQL 9

PostgreSQL 快速、强大、跨平台，并且具有优秀的血统。然而，为了在出现不规则或意外事件时进行故障排查，最好的方法是首先记住该数据库服务是如何安装的。

要开始这个过程，我们必须以以下方式添加相关仓库：

```
# rpm -iUvh http://yum.postgresql.org/9.3/redhat/rhel-7-x86_64/pgdg-centos93-9.3-1.noarch.rpm

```

你应始终通过访问仓库本身确认你正在下载正确的版本，但完成此步骤后，你现在可以像这样安装 PostgreSQL：

```
# yum install postgresql93-server

```

在这个阶段，你可能需要对 PostgreSQL 做一些配置更改。首先，像这样用你喜欢的文本编辑器打开以下文件：

```
# nano /var/lib/pgsql/9.3/data/postgresql.conf

```

前面的配置较为冗长，在大多数情况下，你只需取消注释相关行或替换相应的值以适应你的环境。例如，要让 PostgreSQL 数据库服务器监听特定的 IP 地址，你需要取消注释以下这一行：

```
#listen_addresses = 'localhost'
```

然后，你可以像这样更改为你选择的 IP 地址：

```
listen_addresses = 'XXX.XXX.XXX.XXX'
```

然而，取决于你使用的 Linux 版本，在你可能需要或不需要查找 `tcpip_socket` 参数的情况下，你应该改用以下语法：

```
listen_addresses = '*'
```

更进一步，你还应花时间对该文件中的其他设置进行相关修改。这些设置可以包括使用的端口、支持的最大连接数、身份验证超时、资源使用情况以及 PostgreSQL 提供的更多功能，在此之后再打开以下文件，以调整对数据库服务器的网络访问：

```
# nano /var/lib/pgsql/9.3/data/pg_hba.conf

```

例如，如果你想为整个 IP 范围（例如，192.168.1.0/24）提供本地网络访问，可以使用以下语法：

```
host all all 192.168.1.0 255.255.255.0 trust
```

另外，为了心安理得，你可以使用以下语法，通过稍少的输入实现类似的效果：

```
host all all 192.168.1.0/24 md5
```

所以，在完成前面的步骤以启用 PostgreSQL 启动时，你必须输入：

```
# systemctl enable postgresql-9.3

```

要初始化数据库服务器，你必须输入：

```
# /usr/pgsql-9.3/bin/postgresql93-setup initdb
# systemctl start postgresql-9.3

```

最后，如果你想访问数据库服务器，你必须切换到预定义的 PostgreSQL 用户帐户，像这样：

```
# su - postgres

```

然后，你可以按照以下方式连接到 PostgreSQL 数据库：

```
# psql

```

然后，使用以下表达式，你将能够查看你可以使用的所有 `psql` 命令，以便开始添加用户和模板，以及创建数据库：

```
# \?

```

例如，要随时退出 `psql` 控制台，可以使用以下语法：

```
# \q

```

最后，返回标准 bash 控制台后，你可以随时通过自定义以下命令来确认 PostgreSQL 的连接性：

```
# psql -h <database_hostname> -U <username> -d <database_name>

```

另外，你也可以使用 `ps` 命令，像这样：

```
# ps -ef | grep posgres

```

# 总结

从安装与优化，到恢复 root 用户的丢失密码，本章我们重点讨论了与 MariaDB 和 PostgreSQL 故障排除过程相关的一些问题。正如开头所提到的，本文并不打算处理具体的编程或开发问题，但由于这些数据库系统在企业中普遍存在，因此你将需要处理它们；在这方面，我们讨论了一系列的主题，为故障排除几乎任何数据库服务提供了一个有用的起点。

在下一章，我们将继续本书的主题，讨论一种独特的故障排除 Web 服务的方法。

# 参考文献

+   MySQL 项目主页：[`dev.mysql.com/`](http://dev.mysql.com/)

+   MySQL YUM 下载：[`dev.mysql.com/downloads/repo/yum/`](http://dev.mysql.com/downloads/repo/yum/)

+   MariaDB 项目首页：[`mariadb.org/en/`](https://mariadb.org/en/)

+   MariaDB 常见问题解答：[`mariadb.com/kb/en/mariadb/faq/`](https://mariadb.com/kb/en/mariadb/faq/)

+   使用 Yum 安装 MariaDB：[`mariadb.com/kb/en/mariadb/yum/`](https://mariadb.com/kb/en/mariadb/yum/)

+   Red Hat Bugzilla – Bug 1020055，[`bugzilla.redhat.com/show_bug.cgi?id=1020055`](https://bugzilla.redhat.com/show_bug.cgi?id=1020055)

+   MySQLTuner 首页：[`mysqltuner.com`](http://mysqltuner.com)

+   MySQLTuner GitHub 首页：[`github.com/major/MySQLTuner-perl`](https://github.com/major/MySQLTuner-perl)

+   MariaDB-5.3.4 性能基准测试：[`blog.mariadb.org/benchmarking-mariadb-5-3-4/`](https://blog.mariadb.org/benchmarking-mariadb-5-3-4/)

+   排查问题 – 启动 MySQL 服务器：[`dev.mysql.com/doc/refman/5.1/en/starting-server.html`](http://dev.mysql.com/doc/refman/5.1/en/starting-server.html)

+   MariaDB 与 MySQL – 兼容性：[`mariadb.com/kb/en/mariadb/mariadb-vs-mysql-compatibility/`](https://mariadb.com/kb/en/mariadb/mariadb-vs-mysql-compatibility/)

+   PostgreSQL 9.3.6 文档：[`www.postgresql.org/docs/9.3/static/index.html`](http://www.postgresql.org/docs/9.3/static/index.html)

+   PostgreSQL 客户端认证：[`www.postgresql.org/docs/9.3/static/client-authentication.html`](http://www.postgresql.org/docs/9.3/static/client-authentication.html)

+   PostgreSQL 数据库角色：[`www.postgresql.org/docs/9.3/static/user-manag.html`](http://www.postgresql.org/docs/9.3/static/user-manag.html)

+   PostgreSQL 仓库：[`yum.postgresql.org/repopackages.php`](http://yum.postgresql.org/repopackages.php)
