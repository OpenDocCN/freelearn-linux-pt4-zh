# 使用 OpenStack 部署 KVM 实例

在本章中，我们将涵盖以下主题：

+   为 OpenStack 部署准备主机

+   安装和配置 OpenStack Keystone 身份认证服务

+   安装和配置 OpenStack Glance 镜像服务

+   安装和配置 OpenStack Nova 计算服务

+   安装和配置 OpenStack Neutron 网络服务

+   使用 OpenStack 构建和检查 KVM 实例

+   停止 OpenStack 中的 KVM 实例

+   终止 OpenStack 中的 KVM 实例

# 介绍

OpenStack 是一个云操作系统，它简化了虚拟机或容器的部署和管理，以可扩展且高可用的方式进行操作。它在计算资源池（物理或虚拟服务器）上运行，并提供智能调度机制，选择合适的主机来构建或迁移虚拟机。

OpenStack 允许更轻松地管理虚拟镜像，并提供集中式的创建和管理软件定义网络的方式。它与各种外部和内部项目集成，以提供用户和服务认证。OpenStack 的模块化设计允许根据需要添加和移除服务，在最小化的生产环境部署中，可能仅由两个项目（镜像服务和计算服务）组成。

以下图表显示了不断增长的 OpenStack 项目列表及其之间的相互作用：

![](img/00012.jpeg)

OpenStack 组件及其相互作用

在本章中，我们将使用 OpenStack Newton 版本中的 Keystone、Glance、Nova 和 Neutron 项目，在 Ubuntu Xenial 16.04 服务器上创建一个简单的 OpenStack 部署，使用两台计算节点。

获取有关 OpenStack 项目的更多信息，请访问 [`www.openstack.org/software/`](https://www.openstack.org/software/)。

# 为 OpenStack 部署准备主机

在本指南中，我们将安装 OpenStack 所依赖的基础设施组件，如数据库服务器、消息队列和缓存服务。我们将在本章中使用的项目依赖这些服务进行通信和持久化存储。

# 准备工作

本指南中，我们将需要以下组件：

+   一台具有强大虚拟化能力的 Ubuntu 服务器

+   需要访问互联网以安装软件包

# 如何操作…

为了简化部署并专注于 OpenStack 的配置方面，我们将使用一台物理服务器来托管所有服务。在生产环境中，通常会将每个服务分配到各自的服务器集群上，以实现可扩展性和高可用性。通过本章中概述的步骤，你应该能够在多台主机上部署所有服务，只需根据需要替换配置文件中的 IP 地址和主机名。

1.  更新主机并安装 Newton 版本的软件包仓库：

```
root@controller:~# apt install software-properties-common
root@controller:~# add-apt-repository cloud-archive:newton
root@controller:~# apt update && apt dist-upgrade
root@controller:~# reboot
root@controller:~# apt install python-openstackclient

```

1.  安装 MariaDB 数据库服务器：

```
root@controller:~# apt install mariadb-server python-pymysql
root@controller:~# cat /etc/mysql/mariadb.conf.d/99-openstack.cnf
[mysqld]
bind-address = 10.208.130.36
default-storage-engine = innodb
innodb_file_per_table
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
root@controller:~#

```

根据你的主机，替换数据库服务器绑定的网络接口的 IP 地址。

1.  重启服务并确保安装安全：

```
root@controller:~# service mysql restart
root@controller:~# mysql_secure_installation

```

为了简便起见，我们将在本章中使用 `lxcpassword` 作为所有服务的密码。

1.  安装 RabbitMQ 消息服务，创建一个新用户、密码并设置权限：

```
root@controller:~# apt install rabbitmq-server
root@controller:~# rabbitmqctl add_user openstack lxcpassword
Creating user "openstack" ...
root@controller:~# rabbitmqctl set_permissions openstack ".*" ".*" ".*"
Setting permissions for user "openstack" in vhost "/" ...
root@controller:~#

```

1.  安装并配置 `memcached` 服务：

```
root@controller:~# apt install memcached python-memcache
root@controller:~# sed -i 's/127.0.0.1/10.208.130.36/g'   
/etc/memcached.conf
root@controller:~# cat /etc/memcached.conf | grep -vi "#" | sed 
'/^$/d'
-d
logfile /var/log/memcached.log
-m 64
-p 11211
-u memcache
-l 10.208.130.36
root@controller:~# service memcached restart
root@controller:~#

```

# 它是如何工作的……

OpenStack 使用 SQL 数据库，如 Mysql/MariaDB/Percona，来存储关于其服务的信息。在接下来的教程中，我们将为 Keystone、Glance、Nova 和 Neutron 项目创建数据库。我们在第 1 到第 3 步安装和配置 MariaDB。

我们在第 4 步安装并配置的消息队列提供了一种集中方式，让各个服务通过生成和消费消息来相互通信。OpenStack 支持几种不同的消息总线实现，如 RabbitMQ、Qpid 和 ZeroMQ。

身份服务 Keystone 使用 `memcached` 守护进程缓存认证令牌。我们将在第 5 步安装并配置它。

# 安装和配置 OpenStack Keystone 身份服务

Keystone 项目提供的身份服务是一个集中点，用于管理认证和授权，其他 OpenStack 组件，如 Nova 计算和镜像服务 Glance，都会使用它。Keystone 还维护着一个服务目录，用户可以通过查询定位服务和它们提供的端点。

在这个教程中，我们将安装并配置 Keystone，创建两个项目（服务的所有权单元），并将用户和角色分配给这些项目。

# 准备工作

对于这个教程，我们将需要以下内容：

+   一台具有卓越虚拟化能力的 Ubuntu 服务器

+   访问互联网以安装软件包

+   如 *为 OpenStack 部署准备主机* 中所述，安装并配置数据库服务器、消息队列和 `memcached`。

# 如何操作……

为了安装、配置、创建新项目、用户角色和凭据，请按照这里提供的步骤顺序执行：

1.  创建 keystone 数据库并授予 keystone 用户权限：

```
root@controller:~# mysql -u root -plxcpassword
MariaDB [(none)]> CREATE DATABASE keystone;
Query OK, 1 row affected (0.01 sec)
MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO
'keystone'@'localhost' IDENTIFIED BY 'lxcpassword';
Query OK, 0 rows affected (0.00 sec)
MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 
'keystone'@'%' IDENTIFIED BY 'lxcpassword';
Query OK, 0 rows affected (0.01 sec)
MariaDB [(none)]> exit
Bye
root@controller:~#

```

1.  从我们之前配置的仓库中安装身份服务 Keystone：

```
root@controller:~# apt install keystone

```

1.  创建以下最小化的 Keystone 配置：

```
root@controller:~# cat /etc/keystone/keystone.conf
[DEFAULT]
log_dir = /var/log/keystone

[assignment]
[auth]
[cache]
[catalog]
[cors]
[cors.subdomain]
[credential]
[database]
connection = mysql+pymysql://keystone:lxcpassword@controller/keystone

[domain_config]
[endpoint_filter]
[endpoint_policy]
[eventlet_server]
[federation]
[fernet_tokens]
[identity]
[identity_mapping]
[kvs]
[ldap]
[matchmaker_redis]
[memcache]
[oauth1]
[os_inherit]
[oslo_messaging_amqp]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[paste_deploy]
[policy]
[profiler]
[resource]
[revoke]
[role]
[saml]
[security_compliance]
[shadow_users]
[signing]
[token]
provider = fernet
[tokenless_auth]
[trust]
[extra_headers]
Distribution = Ubuntu
root@controller:~#

```

1.  填充 Keystone 数据库：

```
root@controller:~# su -s /bin/sh -c "keystone-manage db_sync" keystone
...
root@controller:~#

```

1.  初始化 Fernet 密钥存储库：

```
root@controller:~# keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
root@controller:~# keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
root@controller:~#

```

1.  启动 Keystone 服务：

```
root@controller:~# keystone-manage bootstrap --bootstrap-password lxcpassword --bootstrap-admin-url http://controller:35357/v3/ --bootstrap-internal-url http://controller:35357/v3/ --bootstrap-public-url http://controller:5000/v3/ --bootstrap-region-id RegionOne
root@controller:~#

```

1.  在 Apache 中添加以下配置，并重启服务：

```
root@controller:~# cat /etc/apache2/apache2.conf
...
ServerName controller 
...
root@controller:~# service apache2 restart

```

1.  删除与 Keystone 一起打包的默认 SQLite 数据库：

```
root@controller:~# rm -f /var/lib/keystone/keystone.db

```

1.  通过定义以下环境变量来创建一个管理员帐户：

```
root@controller:~# export OS_USERNAME=admin
root@controller:~# export OS_PASSWORD=lxcpassword
root@controller:~# export OS_PROJECT_NAME=admin
root@controller:~# export OS_USER_DOMAIN_NAME=default
root@controller:~# export OS_PROJECT_DOMAIN_NAME=default
root@controller:~# export OS_AUTH_URL=http://controller:35357/v3
root@controller:~# export OS_IDENTITY_API_VERSION=3
root@controller:~#

```

1.  在 Keystone 中为服务创建一个项目并列出它：

```
root@controller:~# openstack project create --domain default --description "KVM Project" service
+-------------+-----------------------------------+
| Field        | Value                            |
+-------------+-----------------------------------+
| description  | KVM Project                      |
| domain_id    | default                          |
| enabled      | True                             |
| id           | 9a1a863fe41b42b2955b313f2cca0ef0 |
| is_domain    | False                            |
| name         | service                          |
| parent_id    | default                          |
+-------------+-----------------------------------+
root@controller:~# openstack project list
+----------------------------------+---------+
| ID                               | Name    |
+----------------------------------+---------+
| 06f4e2d7e384474781803395b24b3af2 | admin   |
| 9a1a863fe41b42b2955b313f2cca0ef0 | service |
+----------------------------------+---------+
root@controller:~#

```

1.  创建一个无特权的项目和一个用户，这个用户将被常规客户端使用，而不是 OpenStack 服务：

```
root@controller:~# openstack project create --domain default --description "KVM User Project" kvm
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | KVM User Project                 |
| domain_id   | default                          |
| enabled     | True                             |
| id          | eb9cdc2c2b4e4f098f2d104752970d52 |
| is_domain   | False                            |
| name        | kvm                              |
| parent_id   | default                          |
+-------------+----------------------------------+
root@controller:~#
root@controller:~# openstack user create --domain default --password-prompt kvm
User Password:
Repeat User Password:
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 1e83e0c8ca194f2e9d8161eb61d21030 |
| name                | kvm                              |
| password_expires_at | None                             |
+---------------------+----------------------------------+
root@controller:~#

```

1.  创建一个用户角色并将其与 KVM 项目和用户关联：

```
root@controller:~# openstack role create user
+-----------+----------------------------------+
| Field     | Value                            |
+-----------+----------------------------------+
| domain_id | None                             |
| id        | 331c0b61e9784112874627264f03a058 |
| name      | user                             |
+-----------+----------------------------------+
root@controller:~# openstack role add --project kvm --user kvm user
root@controller:~#

```

1.  配置 **Web 服务网关接口** (**WSGI**) 中间件管道以供 Keystone 使用：

```
root@controller:~# cat /etc/keystone/keystone-paste.ini
# Keystone PasteDeploy configuration file.
[filter:debug]
use = egg:oslo.middleware#debug
[filter:request_id]
use = egg:oslo.middleware#request_id
[filter:build_auth_context]
use = egg:keystone#build_auth_context
[filter:token_auth]
use = egg:keystone#token_auth
[filter:admin_token_auth]
use = egg:keystone#admin_token_auth
[filter:json_body]
use = egg:keystone#json_body
[filter:cors]
use = egg:oslo.middleware#cors
oslo_config_project = keystone
[filter:http_proxy_to_wsgi]
use = egg:oslo.middleware#http_proxy_to_wsgi
[filter:ec2_extension]
use = egg:keystone#ec2_extension
[filter:ec2_extension_v3]
use = egg:keystone#ec2_extension_v3
[filter:s3_extension]
use = egg:keystone#s3_extension
[filter:url_normalize]
use = egg:keystone#url_normalize
[filter:sizelimit]
use = egg:oslo.middleware#sizelimit
[filter:osprofiler]
use = egg:osprofiler#osprofiler
[app:public_service]
use = egg:keystone#public_service
[app:service_v3]
use = egg:keystone#service_v3
[app:admin_service]
use = egg:keystone#admin_service
[pipeline:public_api]
pipeline = cors sizelimit http_proxy_to_wsgi osprofiler url_normalize request_id build_auth_context token_auth json_body ec2_extension public_service
[pipeline:admin_api]
pipeline = cors sizelimit http_proxy_to_wsgi osprofiler url_normalize request_id build_auth_context token_auth json_body ec2_extension s3_extension admin_service
[pipeline:api_v3]
pipeline = cors sizelimit http_proxy_to_wsgi osprofiler url_normalize request_id build_auth_context token_auth json_body ec2_extension_v3 s3_extension service_v3
[app:public_version_service]
use = egg:keystone#public_version_service
[app:admin_version_service]
use = egg:keystone#admin_version_service
[pipeline:public_version_api]
pipeline = cors sizelimit osprofiler url_normalize public_version_service
[pipeline:admin_version_api]
pipeline = cors sizelimit osprofiler url_normalize admin_version_service
[composite:main]
use = egg:Paste#urlmap
/v2.0 = public_api
/v3 = api_v3
/ = public_version_api
[composite:admin]
use = egg:Paste#urlmap
/v2.0 = admin_api
/v3 = api_v3
/ = admin_version_api
root@controller:~#

```

1.  为管理员和 KVM 用户请求令牌：

```
root@controller:~# openstack --os-auth-url http://controller:35357/v3 --os-project-domain-name default --os-user-domain-name default --os-project-name admin --os-username admin token issue
+------------+----------------------------------+
| Field      | Value                            |
+------------+----------------------------------+
| expires    | 2017-04-26 18:29:03+00:00        |
| id         | gAAAAABZMIdwefsdfB8e4rFk5IALgM4U |
| project_id | 123c1e6f33584dd1876c0a34249a6e11 |
| user_id    | cc14c5dbbd654c438e52d38efaf4f1a6 |
+------------+----------------------------------+
root@controller:~# openstack --os-auth-url http://controller:5000/v3 --os-project-domain-name default --os-user-domain-name default --os-project-name kvm --os-username kvm token issue
+------------+----------------------------------+
| Field      | Value                            |
+------------+----------------------------------+
| expires    | 2017-04-26 18:29:52+00:00        |
| id         | gAAAAABZANkQmInUifl6Up_PzdH_9OHd |
| project_id | 10a92eccbad9439d9e56c4edda6b211f |
| user_id    | a186b226ed1e4717b25bb978f2bc9958 |
+------------+----------------------------------+
root@controller:~#

```

1.  创建包含管理员和用户凭证的文件：

```
root@controller:~# cat rc.admin
export OS_PROJECT_DOMAIN_NAME=default
export OS_USER_DOMAIN_NAME=default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=lxcpassword
export OS_AUTH_URL=http://controller:35357/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
root@controller:~#
root@controller:~# cat rc.kvm
export OS_PROJECT_DOMAIN_NAME=default
export OS_USER_DOMAIN_NAME=default
export OS_PROJECT_NAME=kvm
export OS_USERNAME=kvm
export OS_PASSWORD=lxcpassword
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
root@controller:~#

```

1.  源代码管理员凭证文件：

```
root @ controller: ~ #. rc.admin 
root @ controller: ~ #

```

1.  为管理员用户请求认证令牌：

```
root@controller:~# openstack token issue

+------------+----------------------------------+
| Field      | Value                            |
+------------+----------------------------------+
| expires    | 2017-04-26 18:30:41+00:00        |
| id         | gAAAAABZANlBdsu-DTmz6ME2Z8JFKjJM |
| project_id | 123c1e6f33584dd1876c0a34249a6e11 |
| user_id    | cc14c5dbbd654c438e52d38efaf4f1a6 |
+------------+----------------------------------+
root@controller:~#

```

# 它是如何工作的…

我们首先在步骤 1 中在 MariaDB 中创建 Keystone 数据库，并授予必要的用户权限。在步骤 2 中，我们安装 Keystone 包。

在步骤 3 中，我们为服务创建配置文件。正如从输出中看到的，大多数选项已被省略，假定使用默认值。

在步骤 4 中，我们运行一个脚本，通过创建数据库架构来填充 Keystone 数据库。

Keystone 使用令牌来认证和授权用户及服务。可用的令牌格式包括 UUID、PKI 和 Fernet 令牌。对于本次部署，我们将使用 Fernet 令牌。Fernet 令牌不需要在后端存储中持久化。在步骤 5 中，我们初始化 Fernet 密钥库。

有关可用身份令牌的更多信息，请参考 [`docs.openstack.org/admin-guide/identity-tokens.html`](http://docs.openstack.org/admin-guide/identity-tokens.html)。

在步骤 6 中，我们引导 Keystone，并在步骤 7 中更新 Apache 配置，在步骤 8 中进行一些清理。

在步骤 9 中，我们导出一个包含 Keystone 用户、密码和端点的环境变量列表。

在步骤 10 中，我们在 Keystone 中创建第一个项目，其他服务将使用该项目。项目代表所有权单元，其中所有资源都属于该项目。在步骤 11 和 12 中，我们创建一个无特权的项目及相关用户。

在步骤 13 中，我们为 Keystone 配置 WSGI 中间件管道。

在步骤 14 中，我们为管理员和 KVM 用户请求并获取令牌，在步骤 15 中，我们创建两个环境变量文件，以便在需要在用户之间切换时使用。

在步骤 16 和 17 中，我们源管理员凭证和项目端点，并获得授权令牌。

# 安装和配置 OpenStack Glance 镜像服务

Glance 镜像服务提供一个 API，供我们用于发现、注册和获取虚拟机镜像。稍后当我们使用 Nova 计算来构建新的 KVM 实例时，Nova 服务会向 Glance 发送请求以获取所需的镜像类型。

在本教程中，我们将安装 Glance 并注册一个新的 Ubuntu 镜像。

# 准备工作

对于本教程，我们将需要以下内容：

+   配备优秀虚拟化能力的 Ubuntu 服务器

+   具有安装包的网络访问权限

+   安装和配置好的数据库服务器、消息队列和 `memcached`，如在 *为 OpenStack 部署准备主机* 教程中所述。

+   我们在 *安装和配置 OpenStack Keystone 身份服务* 教程中部署的 Keystone 服务

# 如何操作…

要安装、配置并在 Glance 注册图像，请按照以下步骤操作：

1.  创建 Glance 数据库和用户：

```
root@controller:~# mysql -u root -plxcpassword
MariaDB [(none)]> CREATE DATABASE glance;
Query OK, 1 row affected (0.00 sec)
MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'localhost' IDENTIFIED BY 'lxcpassword';
Query OK, 0 rows affected (0.00 sec)
MariaDB [(none)]> GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' IDENTIFIED BY 'lxcpassword';
Query OK, 0 rows affected (0.00 sec)
MariaDB [(none)]> exit
Bye
root@controller:~#

```

1.  创建 Glance 用户并将其添加到管理员角色：

```
root@controller:~# openstack user create --domain default --password-prompt glance
User Password:
Repeat User Password:
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | e566c6e2012148daa374cd68077b38df |
| name                | glance                           |
| password_expires_at | None                             |
+---------------------+----------------------------------+
root@controller:~# openstack role add --project service --user glance admin
root@controller:~#

```

1.  创建 Glance 服务定义：

```
root@controller:~# openstack service create --name glance --description "OpenStack Image" image
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Image                  |
| enabled     | True                             |
| id          | d4d42a586551461c8b445b927f2144e1 |
| name        | glance                           |
| type        | image                            |
+-------------+----------------------------------+
root@controller:~#

```

1.  在 Keystone 中创建 Glance API 端点：

```
root@controller:~# openstack endpoint create --region RegionOne image public http://controller:9292
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 618af0c845194f508752f230364d6e0e |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | d4d42a586551461c8b445b927f2144e1 |
| service_name | glance                           |
| service_type | image                            |
| url          | http://controller:9292           |
+--------------+----------------------------------+
root@controller:~# openstack endpoint create --region RegionOne image internal http://controller:9292
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 991a1b03f7194139b98bafe19acf3518 |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | d4d42a586551461c8b445b927f2144e1 |
| service_name | glance                           |
| service_type | image                            |
| url          | http://controller:9292           |
+--------------+----------------------------------+
root@controller:~# openstack endpoint create --region RegionOne image admin http://controller:9292
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 991a1b03f7194139b98bafe19acf3322 |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | d4d42a586551461c8b445b927f2144e1 |
| service_name | glance                           |
| service_type | image                            |
| url          | http://controller:9292           |
+--------------+----------------------------------+
root@controller:~# 

```

1.  安装 Glance 服务：

```
root@controller:~# apt install glance

```

1.  配置服务：

```
root@controller:~# cat /etc/glance/glance-api.conf
[DEFAULT]
[cors]
[cors.subdomain]

[database]
connection = mysql+pymysql://glance:lxcpassword@controller/glance

[glance_store]
stores = file,http
default_store = file
filesystem_store_datadir = /var/lib/glance/images/

[image_format]
disk_formats = ami,ari,aki,vhd,vhdx,vmdk,raw,qcow2,vdi,iso,root-tar

[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = glance
password = lxcpassword

[matchmaker_redis]
[oslo_concurrency]
[oslo_messaging_amqp]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_middleware]
[oslo_policy]
[paste_deploy]
flavor = keystone

[profiler]
[store_type_location_strategy]
[task]
[taskflow_executor]
root@controller:~#
root@controller:~# cat /etc/glance/glance-registry.conf
[DEFAULT]

[database]
connection = mysql+pymysql://glance:lxcpassword@controller/glance

[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = glance
password = lxcpassword

[matchmaker_redis]
[oslo_messaging_amqp]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_policy]
[paste_deploy]
flavor = keystone

[profiler]
root@controller:~#

```

1.  填充 Glance 数据库：

```
root@controller:~# su -s /bin/sh -c "glance-manage db_sync" glance
...
root@controller:~#

```

1.  启动 Glance 服务守护进程：

```
root@controller:~# service glance-registry restart
root@controller:~# service glance-api restart
root@controller:~#

```

1.  下载 Ubuntu 发行版的 QCOW2 图像：

```
root@controller:~# wget https://uec-images.ubuntu.com/releases/16.04/release-20170330/ubuntu-16.04-server-cloudimg-amd64-disk1.img
Saving to: ‘ubuntu-16.04-server-cloudimg-amd64-disk1.img’
ubuntu-16.04-server-cloudimg-amd64-disk1.img 100%[===================================================>] 309.75M 31.1MB/s in 13s

2017-04-26 17:40:21 (24.5 MB/s) - ‘ubuntu-16.04-server-cloudimg-amd64-disk1.img’ saved [324796416/324796416]
root@controller:~#

```

1.  将图像添加到 Glance 服务：

```
root@controller:~# openstack image create "ubuntu_16.04" --file ubuntu-16.04-server-cloudimg-amd64-disk1.img --disk-format qcow2 --container-format bare --public
+------------------+------------------------------------------------------+
| Field            | Value                                                |
+------------------+------------------------------------------------------+
| checksum         | 87b0b7a4b03dd0bb2177d5cc02c80720                     |
| container_format | bare                                                 |
| created_at       | 2017-04-26T17:41:44Z                                 |
| disk_format      | qcow2                                                |
| file             | /v2/images/abce08d2-2f9f-4545-a414-32019d41c0cd/file |
| id               | abce08d2-2f9f-4545-a414-32019d41c0cd                 |
| min_disk         | 0                                                    |
| min_ram          | 0                                                    |
| name             | ubuntu_16.04                                         |
| owner            | 123c1e6f33584dd1876c0a34249a6e11                     |
| protected        | False                                                |
| schema           | /v2/schemas/image                                    |
| size             | 324796416                                            |
| status           | active                                               |
| tags             |                                                      |
| updated_at       | 2017-04-26T17:41:45Z                                 |
| virtual_size     | None                                                 |
| visibility       | public                                               |
+------------------+------------------------------------------------------+
root@controller:~#

```

1.  列出可用图像及其在文件系统上的位置：

```
root@controller:~# openstack image list
+--------------------------------------+--------------+--------+
| ID                                   | Name         | Status |
+--------------------------------------+--------------+--------+
| abce08d2-2f9f-4545-a414-32019d41c0cd | ubuntu_16.04 | active |
+--------------------------------------+--------------+--------+
root@controller:~# ls -lah /var/lib/glance/images/
drwxr-xr-x 2 glance glance 4.0K Apr 26 17:51 .
drwxr-xr-x 4 glance glance 4.0K Apr 26 17:32 ..
-rw-r----- 1 glance glance 310M Apr 26 17:41 abce08d2-2f9f-4545-a414-32019d41c0cd
root@controller:~# qemu-img info /var/lib/glance/images/abce08d2-2f9f-4545-a414-32019d41c0cd
image: /var/lib/glance/images/abce08d2-2f9f-4545-a414-32019d41c0cd
file format: qcow2
virtual size: 2.2G (2361393152 bytes)
disk size: 310M
cluster_size: 65536
Format specific information:
 compat: 0.10
 refcount bits: 16
root@controller:~# 

```

# 它是如何工作的...

我们首先在第 1 步中在 MariaDB 中创建 Glance 数据库。

在第 2 步和第 3 步中，我们为 Glance 项目创建用户、角色和服务。在第 4 步中，我们在 Keystone 中定义 Glance API 服务端点。Nova 服务和 OpenStack 工具可以使用这些端点查询 Glance 可用的图像。

在第 5 步中，我们安装 Glance 包，并在第 6 步中创建一个最小配置文件。

然后我们在第 7 步中创建数据库模式，通过执行 `glance-manage` Python 脚本并在第 8 步中重启 Glance 服务。

在第 9 步中，我们下载一个 QCOW2 Ubuntu 图像，并在第 10 步中将其添加到 Glance 注册表。

最后，在步骤 11 中，我们列出新添加的图像并在主机文件系统上查看它。

# 安装和配置 OpenStack Nova 计算服务

OpenStack 计算服务，代号 Nova，管理计算资源池及其上运行的虚拟机。Nova 是一组服务，用于创建和管理虚拟机的生命周期。我们将使用 Nova 创建、检查、停止、删除和迁移 KVM 实例。

获取有关各种 Nova 服务的更多信息，请参考：[`docs.openstack.org/developer/nova/`](http://docs.openstack.org/developer/nova/)。

在本教程中，我们将安装和配置以下 Nova 组件：

+   `nova-api`：这是通过 RESTful API 接受并响应用户请求的服务。我们将在创建、运行、停止和迁移 KVM 实例时使用它。

+   `nova-scheduler`：这是根据过滤器（如可用内存、磁盘和 CPU 资源）决定在哪里配置实例的服务。

+   `nova-compute`：这是在计算主机上运行的服务，负责管理 KVM 实例的生命周期，从配置到删除。

+   `nova-conductor`：这是位于我们之前创建的 Nova 数据库和 `nova-compute` 服务之间的服务。

+   `nova-consoleauth`：这是授权用户使用各种控制台连接虚拟机的服务。

+   `nova-novncproxy`：这是提供访问运行 VNC 实例的服务。

# 准备工作

对于本教程，我们需要：

+   一台具有强大虚拟化能力的 Ubuntu 服务器

+   安装软件包时需要访问互联网

+   按照*为 OpenStack 部署准备主机*中的说明，安装并配置数据库服务器、消息队列和`memcached`。

+   我们在*安装和配置 OpenStack Keystone 身份服务*中部署的 Keystone 服务。

+   我们在*安装和配置 OpenStack Glance 镜像服务*中部署的 Glance 服务。

# 如何做...

要安装和配置之前提到的 Nova 服务，请执行以下步骤：

1.  在 MariaDB 中创建 Nova 数据库和用户：

```
root@controller:~# mysql -u root -plxcpassword
MariaDB [(none)]> CREATE DATABASE nova_api;
Query OK, 1 row affected (0.00 sec)
MariaDB [(none)]> CREATE DATABASE nova;
Query OK, 1 row affected (0.00 sec)
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'localhost' IDENTIFIED BY 'lxcpassword';
Query OK, 0 rows affected (0.03 sec)
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' IDENTIFIED BY 'lxcpassword';
Query OK, 0 rows affected (0.00 sec)
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'localhost' IDENTIFIED BY 'lxcpassword';
Query OK, 0 rows affected (0.00 sec)
MariaDB [(none)]> GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY 'lxcpassword';
Query OK, 0 rows affected (0.00 sec)
MariaDB [(none)]> exit
Bye
root@controller:~#

```

1.  创建 Nova 用户，并将其添加到身份服务中的管理员角色：

```
root@controller:~# openstack user create --domain default --password-prompt nova
User Password:
Repeat User Password:
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 038aa8840aca449dbd3e653c5d2c5a08 |
| name                | nova                             |
| password_expires_at | None                             |
+---------------------+----------------------------------+
root@controller:~# openstack role add --project service --user nova admin
root@controller:~#

```

1.  创建 Nova 服务和端点：

```
root@controller:~# openstack service create --name nova --description "OpenStack Compute" compute
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Compute                |
| enabled     | True                             |
| id          | 04132edd7f654f56ba0cc23ac182c9aa |
| name        | nova                             |
| type        | compute                          |
+-------------+----------------------------------+
root@controller:~# openstack endpoint create --region RegionOne compute public http://controller:8774/v2.1/%(tenant_id)s
+--------------+-------------------------------------------+
| Field        | Value                                     |
+--------------+-------------------------------------------+
| enabled      | True                                      |
| id           | 5fc54236c324412db135dff88807e820          |
| interface    | public                                    |
| region       | RegionOne                                 |
| region_id    | RegionOne                                 |
| service_id   | 04132edd7f654f56ba0cc23ac182c9aa          |
| service_name | nova                                      |
| service_type | compute                                   |
| url          | http://controller:8774/v2.1/%(tenant_id)s |
+--------------+-------------------------------------------+
root@controller:~# openstack endpoint create --region RegionOne compute internal http://controller:8774/v2.1/%(tenant_id)s
+--------------+-------------------------------------------+
| Field        | Value                                     |
+--------------+-------------------------------------------+
| enabled      | True                                      |
| id           | a0f623ed345e4bdb8fced929b7fe6b3f          |
| interface    | internal                                  |
| region       | RegionOne                                 |
| region_id    | RegionOne                                 |
| service_id   | 04132edd7f654f56ba0cc23ac182c9aa          |
| service_name | nova                                      |
| service_type | compute                                   |
| url          | http://controller:8774/v2.1/%(tenant_id)s |
+--------------+-------------------------------------------+
root@controller:~# openstack endpoint create --region RegionOne compute admin http://controller:8774/v2.1/%(tenant_id)s
+--------------+-------------------------------------------+
| Field        | Value                                     |
+--------------+-------------------------------------------+
| enabled      | True                                      |
| id           | 3964db0d281545acbaa6c18abc44a216          |
| interface    | admin                                     |
| region       | RegionOne                                 |
| region_id    | RegionOne                                 |
| service_id   | 04132edd7f654f56ba0cc23ac182c9aa          |
| service_name | nova                                      |
| service_type | compute                                   |
| url          | http://controller:8774/v2.1/%(tenant_id)s |
+--------------+-------------------------------------------+
root@controller:~#

```

1.  安装 Nova 软件包，它将提供 API、指挥器、控制台和调度程序服务：

```
root@controller:~# apt install nova-api nova-conductor nova-consoleauth nova-novncproxy nova-scheduler

```

1.  创建 Nova 配置文件：

```
root@controller:~# cat /etc/nova/nova.conf
[DEFAULT]
dhcpbridge_flagfile=/etc/nova/nova.conf
dhcpbridge=/usr/bin/nova-dhcpbridge
log-dir=/var/log/nova
state_path=/var/lib/nova
force_dhcp_release=True
verbose=True
ec2_private_dns_show_ip=True
enabled_apis=osapi_compute,metadata
transport_url = rabbit://openstack:lxcpassword@controller
auth_strategy = keystone
my_ip = 10.208.132.45
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver

[database]
connection = mysql+pymysql://nova:lxcpassword@controller/nova

[api_database]
connection = mysql+pymysql://nova:lxcpassword@controller/nova_api

[oslo_concurrency]
lock_path = /var/lib/nova/tmp

[libvirt]
use_virtio_for_bridges=True

[wsgi]
api_paste_config=/etc/nova/api-paste.ini

[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = lxcpassword

[vnc]
vncserver_listen = $my_ip
vncserver_proxyclient_address = $my_ip

[glance]
api_servers = http://controller:9292
root@controller:~#

```

1.  创建数据库表：

```
root@controller:~# su -s /bin/sh -c "nova-manage api_db sync" nova
...
root@controller:~# su -s /bin/sh -c "nova-manage db sync" nova
...
root@controller:~#

```

1.  启动 Nova 服务：

```
root@controller:~# service nova-api restart
root@controller:~# service nova-consoleauth restart
root@controller:~# service nova-scheduler restart
root@controller:~# service nova-conductor restart
root@controller:~# service nova-novncproxy restart
root@controller:~#

```

1.  安装`nova-compute`服务，它将配置 KVM 实例：

```
root@controller:~# apt install nova-compute

```

1.  按如下方式更新 Nova 配置文件：

```
root@controller:~# cat /etc/nova/nova.conf
[DEFAULT]
dhcpbridge_flagfile=/etc/nova/nova.conf
dhcpbridge=/usr/bin/nova-dhcpbridge
log-dir=/var/log/nova
state_path=/var/lib/nova
force_dhcp_release=True
verbose=True
ec2_private_dns_show_ip=True
enabled_apis=osapi_compute,metadata
transport_url = rabbit://openstack:lxcpassword@controller
auth_strategy = keystone
my_ip = 10.208.132.45
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver
compute_driver = libvirt.LibvirtDriver

[database]
connection = mysql+pymysql://nova:lxcpassword@controller/nova

[api_database]
connection = mysql+pymysql://nova:lxcpassword@controller/nova_api

[oslo_concurrency]
lock_path = /var/lib/nova/tmp

[libvirt]
use_virtio_for_bridges=True

[wsgi]
api_paste_config=/etc/nova/api-paste.ini

[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = lxcpassword

[vnc]
enabled = True
vncserver_listen = $my_ip
vncserver_proxyclient_address = $my_ip
novncproxy_base_url = http://controller:6080/vnc_auto.html

[glance]
api_servers = http://controller:9292

root@controller:~#

```

1.  指定要使用的虚拟化驱动程序：

```
root@controller:~# cat /etc/nova/nova-compute.conf
[DEFAULT]
compute_driver=libvirt.LibvirtDriver
[libvirt]
virt_type=kvm
root@controller:~# 

```

1.  重启`nova-compute`服务并列出可用服务：

```
root@controller:~# service nova-compute restart
root@controller:~# openstack compute service list
+----+------------------+------------+----------+---------+-------+----------------------+
| ID | Binary           | Host       | Zone     | Status  | State | Updated At           |
+----+------------------+------------+----------+---------+-------+----------------------+
| 8  | nova-consoleauth | controller | internal | enabled | up    | 2017-04-26T17:58     |
| 9  | nova-scheduler   | controller | internal | enabled | up    | 2017-04-26T17:58     |
| 10 | nova-conductor   | controller | internal | enabled | up    | 2017-04-26T17:58     |
| 15 | nova-compute     | controller | nova     | enabled | up    | None                 |
+----+------------------+------------+----------+---------+-------+----------------------+
root@controller:~# pgrep -lf nova | uniq -f1
14110 nova-consoleaut
14176 nova-conductor
14239 nova-novncproxy
20877 nova-api
20994 nova-scheduler
21065 nova-compute
root@controller:~#

```

# 它是如何工作的...

在步骤 1 和 2 中，我们在 MariaDB 中创建了 Nova 数据库和用户。在步骤 3 中，我们创建了可以用来发送 API 调用的服务和端点。

在步骤 4 和 5 中，我们安装了 Nova 服务的软件包并创建了一个简单的配置文件。

在步骤 6 中，我们创建了数据库表模式，并在步骤 7 中启动了 Nova 服务。

对于这个示例部署，我们使用单个节点来运行所有感兴趣的 OpenStack 服务。不过，您也可以使用第二个节点专门运行`nova-compute`服务来配置 KVM 虚拟机。在步骤 8 中，我们安装`nova-compute`服务，更新配置文件，并在步骤 9 和 10 中检查`nova-compute`服务的外部配置。

我们通过确保所有 Nova 服务都已配置并在步骤 11 中运行来完成此任务。

# 安装和配置 OpenStack Neutron 网络服务

OpenStack Neutron 项目提供网络服务来管理虚拟实例之间的网络连接。它负责设置虚拟接口、配置软件桥接、创建路由和管理 IP 地址。

有关各种 Neutron 服务的更多信息，请参阅[`docs.openstack.org/security-guide/networking/architecture.html`](https://docs.openstack.org/security-guide/networking/architecture.html)。

在这个配方中，我们将安装和配置以下 Neutron 组件：

+   `neutron-server`：这是提供 API 的服务，用于动态请求和配置虚拟网络。

+   `neutron-plugin-ml2`：这是一个框架，使得可以使用各种网络技术，例如我们在前面章节中看到的 Linux Bridge、Open vSwitch、GRE 和 VXLAN。

+   `neutron-linuxbridge-agent`：这是提供 Linux Bridge 插件代理的服务。

+   `neutron-l3-agent`：这是执行软件定义网络之间转发和 NAT 功能的守护进程，通过创建虚拟路由器实现。

+   `neutron-dhcp-agent`：此服务控制 DHCP 守护进程，负责为计算节点上运行的实例分配 IP 地址

+   `neutron-metadata-agent`：此服务将实例元数据传递给 Neutron

在早期的食谱中，我们手动配置并使用了 Linux 桥接和 Open vSwitch，之后将网络管理委托给了 libvirt。OpenStack Neutron 与 libvirt 集成，并通过暴露 API 调用进一步自动化此过程，其他服务如 Nova 可利用这些 API。

# 准备工作

对于此食谱，我们需要：

+   一台具有强大虚拟化能力的 Ubuntu 服务器

+   获取互联网访问以进行软件包安装

+   如*为 OpenStack 部署准备主机*食谱中所述，安装并配置了数据库服务器、消息队列和`memcached`：

+   我们在*安装和配置 OpenStack Keystone 身份服务*食谱中部署的 Keystone 服务

+   我们在*安装和配置 OpenStack Nova 计算服务*食谱中配置的 Nova 服务

# 如何操作...

要安装、配置并创建一个由 Neutron 管理的网络，请执行以下步骤：

1.  创建 Neutron 数据库：

```
root@controller:~# mysql -u root -plxcpassword
MariaDB [(none)]> CREATE DATABASE neutron;
Query OK, 1 row affected (0.00 sec)
MariaDB [(none)]> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'localhost' IDENTIFIED BY 'lxcpassword';
Query OK, 0 rows affected (0.00 sec)
MariaDB [(none)]> GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY 'lxcpassword';
Query OK, 0 rows affected (0.00 sec)
MariaDB [(none)]> exit
Bye
root@controller:~#

```

1.  创建 Neutron 用户并将其添加到 Keystone 中的管理员角色：

```
root@controller:~# openstack user create --domain default --password-prompt neutron
User Password:
Repeat User Password:
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 02934ad74c94461482b95fff32d36894 |
| name                | neutron                          |
| password_expires_at | None                             |
+---------------------+----------------------------------+
root@controller:~# openstack role add --project service --user neutron admin
root@controller:~#

```

1.  创建 Neutron 服务和端点：

```
root@controller:~# openstack service create --name neutron --description "OpenStack Networking" network
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description | OpenStack Networking             |
| enabled     | True                             |
| id          | 24b32d32d4b54e3ab2d785a1817b8e7e |
| name        | neutron                          |
| type        | network                          |
+-------------+----------------------------------+
root@controller:~# openstack endpoint create --region RegionOne network public http://controller:9696
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 544821d511e04847869fc601f2ebf0f7 |
| interface    | public                           |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 24b32d32d4b54e3ab2d785a1817b8e7e |
| service_name | neutron                          |
| service_type | network                          |
| url          | http://controller:9696           |
+--------------+----------------------------------+
root@controller:~# openstack endpoint create --region RegionOne network internal http://controller:9696
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 05e276ec603f424f85be8705ce7fe86a |
| interface    | internal                         |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 24b32d32d4b54e3ab2d785a1817b8e7e |
| service_name | neutron                          |
| service_type | network                          |
| url          | http://controller:9696           |
+--------------+----------------------------------+
root@controller:~# openstack endpoint create --region RegionOne network admin http://controller:9696
+--------------+----------------------------------+
| Field        | Value                            |
+--------------+----------------------------------+
| enabled      | True                             |
| id           | 836b4309186146fb9143544490cd0bc1 |
| interface    | admin                            |
| region       | RegionOne                        |
| region_id    | RegionOne                        |
| service_id   | 24b32d32d4b54e3ab2d785a1817b8e7e |
| service_name | neutron                          |
| service_type | network                          |
| url          | http://controller:9696           |
+--------------+----------------------------------+
root@controller:~#

```

1.  安装 Neutron 包：

```
root@controller:~# apt install neutron-server neutron-plugin-ml2 neutron-linuxbridge-agent neutron-l3-agent neutron-dhcp-agent neutron-metadata-agent
...
root@controller:~#

```

1.  创建 Neutron 配置文件：

```
root@controller:~# cat /etc/neutron/neutron.conf
[DEFAULT]
core_plugin = ml2
service_plugins = router
allow_overlapping_ips = True
transport_url = rabbit://openstack:lxcpassword@controller
auth_strategy = keystone
notify_nova_on_port_status_changes = True
notify_nova_on_port_data_changes = True

[agent]
root_helper = sudo /usr/bin/neutron-rootwrap /etc/neutron/rootwrap.conf

[cors]
[cors.subdomain]
[database]
connection = mysql+pymysql://neutron:lxcpassword@controller/neutron

[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = neutron
password = lxcpassword

[matchmaker_redis]
[nova]
auth_url = http://controller:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = nova
password = lxcpassword

[oslo_concurrency]
[oslo_messaging_amqp]
[oslo_messaging_notifications]
[oslo_messaging_rabbit]
[oslo_messaging_zmq]
[oslo_policy]
[qos]
[quotas]
[ssl]
root@controller:~#

```

1.  定义我们将与 Neutron 一起使用的网络类型和扩展：

```
root@controller:~# cat /etc/neutron/plugins/ml2/ml2_conf.ini
[DEFAULT]
[ml2]
type_drivers = flat,vlan,vxlan
tenant_network_types = vxlan
mechanism_drivers = linuxbridge,l2population
extension_drivers = port_security

[ml2_type_flat]
flat_networks = provider

[ml2_type_geneve]
[ml2_type_gre]
[ml2_type_vlan]
[ml2_type_vxlan]
vni_ranges = 1:1000

[securitygroup]
enable_ipset = True
root@controller:~#

```

1.  定义将添加到软件桥接器中的接口，以及桥接器将绑定的 IP 地址，根据需要替换 IP 地址和接口名称（此示例中为 `eth1`）：

```
root@controller:~# cat /etc/neutron/plugins/ml2/linuxbridge_agent.ini
[DEFAULT]
[agent]
[linux_bridge]
physical_interface_mappings = provider:eth1

[securitygroup]
enable_security_group = True
firewall_driver = neutron.agent.linux.iptables_firewall.IptablesFirewallDriver

[vxlan]
enable_vxlan = True
local_ip = 10.208.132.45
l2_population = True
root@controller:~#

```

1.  按如下方式配置 Layer 3 代理：

```
root@controller:~# cat /etc/neutron/l3_agent.ini
[DEFAULT]
interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver

[AGENT]
root@controller:~#

```

1.  配置 DHCP 代理：

```
root@controller:~# cat /etc/neutron/dhcp_agent.ini
[DEFAULT]
interface_driver = neutron.agent.linux.interface.BridgeInterfaceDriver
dhcp_driver = neutron.agent.linux.dhcp.Dnsmasq
enable_isolated_metadata = True

[AGENT]
root@controller:~#

```

1.  为元数据代理创建配置：

```
root@controller:~# cat /etc/neutron/metadata_agent.ini [DEFAULT]
nova_metadata_ip = controller
metadata_proxy_shared_secret = lxcpassword

[AGENT]
[cache]
root@controller:~#

```

1.  更新 Nova 服务的配置文件，以包含 Neutron。以下是一个完全简化的工作示例：

```
root@controller:~# cat /etc/nova/nova.conf
[DEFAULT]
dhcpbridge_flagfile=/etc/nova/nova.conf
dhcpbridge=/usr/bin/nova-dhcpbridge
log-dir=/var/log/nova
state_path=/var/lib/nova
force_dhcp_release=True
verbose=True
ec2_private_dns_show_ip=True
enabled_apis=osapi_compute,metadata
transport_url = rabbit://openstack:lxcpassword@controller
auth_strategy = keystone
my_ip = 10.208.132.45
use_neutron = True
firewall_driver = nova.virt.firewall.NoopFirewallDriver
compute_driver = libvirt.LibvirtDriver
scheduler_default_filters = RetryFilter, AvailabilityZoneFilter, RamFilter, ComputeFilter, ComputeCapabilitiesFilter, ImagePropertiesFilter, ServerGroupAntiAffinityFilter, ServerGroupAffinityFilter

[database]
connection = mysql+pymysql://nova:lxcpassword@controller/nova

[api_database]
connection = mysql+pymysql://nova:lxcpassword@controller/nova_api

[oslo_concurrency]
lock_path = /var/lib/nova/tmp

[libvirt]
use_virtio_for_bridges=True

[wsgi]
api_paste_config=/etc/nova/api-paste.ini

[keystone_authtoken]
auth_uri = http://controller:5000
auth_url = http://controller:35357
memcached_servers = controller:11211
auth_type = password
project_domain_name = default
user_domain_name = default
project_name = service
username = nova
password = lxcpassword

[vnc]
enabled = True
vncserver_listen = $my_ip
vncserver_proxyclient_address = $my_ip
novncproxy_base_url = http://controller:6080/vnc_auto.html

[glance]
api_servers = http://controller:9292

[libvirt]
virt_type = kvm

[neutron]
url = http://controller:9696
auth_url = http://controller:35357
auth_type = password
project_domain_name = default
user_domain_name = default
region_name = RegionOne
project_name = service
username = neutron
password = lxcpassword
service_metadata_proxy = True
metadata_proxy_shared_secret = lxcpassword
root@controller:~#

```

1.  填充 Neutron 数据库：

```
root@controller:~# su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
INFO [alembic.runtime.migration] Context impl MySQLImpl.
INFO [alembic.runtime.migration] Will assume non-transactional DDL.
 Running upgrade for neutron ...
INFO [alembic.runtime.migration] Context impl MySQLImpl.
INFO [alembic.runtime.migration] Will assume non-transactional DDL.
INFO [alembic.runtime.migration] Running upgrade -> kilo, kilo_initial
...
root@controller:~#

```

1.  重启所有 Neutron 服务和 Nova：

```
root@controller:~# service neutron-server restart
root@controller:~# service neutron-linuxbridge-agent restart
root@controller:~# service neutron-dhcp-agent restart
root@controller:~# service neutron-metadata-agent restart
root@controller:~# service neutron-l3-agent restart
root@controller:~# service nova-api restart
root@controller:~# service nova-compute restart
root@controller:~#

```

1.  验证 Neutron 服务是否已注册：

```
root@controller:~# openstack network agent list
+--------------------------------------+--------------------+------+-------------------+-------+-------+---------------------------+
| ID | Agent Type | Host | Availability Zone | Alive | State | Binary |
+--------------------------------------+--------------------+------+-------------------+-------+-------+---------------------------+
| 9242d71d-de25-4b3e-8aa8-62691ef72001 | Linux bridge agent | kvm2 | None | True | UP | neutron-linuxbridge-agent |
| 92b601de-06df-4b10-88c7-8f27bc48f6ab | L3 agent | kvm2 | nova | True | UP | neutron-l3-agent |
| d249f986-9b26-4c5d-8ea5-311daf3b395d | DHCP agent | kvm2 | nova | True | UP | neutron-dhcp-agent |
| f3cac79b-a7c3-4672-b846-9268f2d58706 | Metadata agent | kvm2 | None | True | UP | neutron-metadata-agent |
+--------------------------------------+--------------------+------+-------------------+-------+-------+---------------------------+
root@controller:~#

```

1.  创建一个新网络：

```
root@controller:~# openstack network create nat
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | UP                                   |
| availability_zone_hints   |                                      |
| availability_zones        |                                      |
| created_at                | 2017-04-26T18:17:24Z                 |
| description               |                                      |
| headers                   |                                      |
| id                        | b7ccb514-21fc-4ced-b74f-026e7e358bba |
| ipv4_address_scope        | None                                 |
| ipv6_address_scope        | None                                 |
| mtu                       | 1450                                 |
| name                      | nat                                  |
| port_security_enabled     | True                                 |
| project_id                | 123c1e6f33584dd1876c0a34249a6e11     |
| project_id                | 123c1e6f33584dd1876c0a34249a6e11     |
| provider:network_type     | vxlan                                |
| provider:physical_network | None                                 |
| provider:segmentation_id  | 37                                   |
| revision_number           | 3                                    |
| router:external           | Internal                             |
| shared                    | False                                |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| tags                      | []                                   |
| updated_at                | 2017-04-26T18:17:24Z                 |
+---------------------------+--------------------------------------+
root@controller:~#

```

1.  定义 DNS 服务器、默认网关和将分配给客户机的子网范围：

```
root@controller:~# openstack subnet create --network nat --dns-nameserver 8.8.8.8 --gateway 192.168.0.1 --subnet-range 192.168.0.0/24 nat
+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| allocation_pools  | 192.168.0.2-192.168.0.254            |
| cidr              | 192.168.0.0/24                       |
| created_at        | 2017-04-26T18:17:41Z                 |
| description       |                                      |
| dns_nameservers   | 8.8.8.8                              |
| enable_dhcp       | True                                 |
| gateway_ip        | 192.168.0.1                          |
| headers           |                                      |
| host_routes       |                                      |
| id                | 296250a7-f241-4f84-adbb-64a45c943094 |
| ip_version        | 4                                    |
| ipv6_address_mode | None                                 |
| ipv6_ra_mode      | None                                 |
| name              | nat                                  |
| network_id        | b7ccb514-21fc-4ced-b74f-026e7e358bba |
| project_id        | 123c1e6f33584dd1876c0a34249a6e11     |
| project_id        | 123c1e6f33584dd1876c0a34249a6e11     |
| revision_number   | 2                                    |
| service_types     | []                                   |
| subnetpool_id     | None                                 |
| updated_at        | 2017-04-26T18:17:41Z                 |
+-------------------+--------------------------------------+
root@controller:~#

```

1.  更新 Neutron 中的子网信息：

```
root@controller:~# neutron net-update nat --router:external
Updated network: nat
root@controller:~#

```

1.  创建一个新的软件路由器：

```
root@controller:~# openstack router create router

+-------------------------+--------------------------------------+
| Field                   | Value                                |
+-------------------------+--------------------------------------+
| admin_state_up          | UP                                   |
| availability_zone_hints |                                      |
| availability_zones      |                                      |
| created_at              | 2017-04-26T18:18:05Z                 |
| description             |                                      |
| external_gateway_info   | null                                 |
| flavor_id               | None                                 |
| headers                 |                                      |
| id                      | f9cd8c96-a53c-4585-ad21-0e409f3b4d70 |
| name                    | router                               |
| project_id              | 10a92eccbad9439d9e56c4edda6b211f     |
| project_id              | 10a92eccbad9439d9e56c4edda6b211f     |
| revision_number         | 3                                    |
| routes                  |                                      |
| status                  | ACTIVE                               |
| updated_at              | 2017-04-26T18:18:05Z                 |
+-------------------------+--------------------------------------+
root@controller:~#

```

1.  作为管理员用户，将我们之前创建的子网添加为路由器的接口：

```
root@controller:~# . rc.admin
root@controller:~# neutron router-interface-add router nat
Added interface 2e1e2fd3-1819-489b-a21f-7005862f9de7 to router router.
root@controller:~#

```

1.  列出 Neutron 创建的网络命名空间：

```
root@controller:~# ip netns
qrouter-f9cd8c96-a53c-4585-ad21-0e409f3b4d70
qdhcp-b7ccb514-21fc-4ced-b74f-026e7e358bba
root@controller:~#

```

1.  列出软件路由器上的端口：

```
root@controller:~# neutron router-port-list router
+--------------------------------------+------+-------------------+------------------------------------------------------------------------------------+
| id | name | mac_address | fixed_ips |
+--------------------------------------+------+-------------------+------------------------------------------------------------------------------------+
| 2e1e2fd3-1819-489b-a21f-7005862f9de7 | | fa:16:3e:0e:db:14 | {"subnet_id": "296250a7-f241-4f84-adbb-64a45c943094", "ip_address": "192.168.0.1"} |
+--------------------------------------+------+-------------------+------------------------------------------------------------------------------------+
root@controller:~#

```

1.  列出 Neutron 网络，并确保我们之前创建的网络已存在：

```
root@controller:~# openstack network list
+--------------------------------------+------+--------------------------------------+
| ID                                   | Name | Subnets                              |
+--------------------------------------+------+--------------------------------------+
| b7ccb514-21fc-4ced-b74f-026e7e358bba | nat  | 296250a7-f241-4f84-adbb-64a45c943094 |
+--------------------------------------+------+--------------------------------------+
root@controller:~#

```

# 它是如何工作的...

我们通过在第 1 步中为 Neutron 创建一个新的数据库开始本教程。接着，我们创建 Neutron 服务的用户并将其添加到服务的管理员角色。在第 2 步和第 3 步中，我们定义将供 Nova 使用的服务端点。在第 4 步和第 5 步中，我们安装 Neutron 软件包并创建一个基础配置文件。在第 6 步中，我们选择 VXLAN 类型的网络用于本例部署。在第 7 步至第 10 步中，我们配置桥接代理、3 层代理、DHCP 代理和元数据代理。

在第 11 步中，我们更新 Nova 配置文件，添加关于 Neutron 服务的部分。在第 12 步中，我们创建数据库模式，并在第 13 步中重启所有 Neutron 服务，包括 `nova-api` 和 `nova-compute`。

在第 14 步，我们验证 Neutron 服务是否已注册，并继续在第 15 步创建一个新网络。

在第 18 步，我们定义一个新的软件路由器。在第 19 步中，我们将之前创建的子网添加到路由器中，然后在第 21 步中验证新的路由配置。

第 22 步确保我们之前定义的网络是活动的。

# 使用 OpenStack 构建和检查 KVM 实例

在本教程中，我们将使用之前部署的 OpenStack 基础设施构建我们的第一个 KVM 实例。构建新 KVM 实例包括以下步骤：

1.  我们向 `nova-api` 服务发送 API 请求。

1.  `nova-api` 服务向 nova-scheduler 服务请求目标计算主机。

1.  `nova-scheduler` 根据配置的过滤器（如可用内存、磁盘和 CPU 使用情况）选择一个可用的计算主机。

1.  一旦 `nova-scheduler` 选择了一个合适的主机，选定主机上的 `nova-compute` 服务会向 Glance 仓库请求镜像（如果镜像尚未缓存）。镜像传输到新服务器后，`nova-compute` 会构建新的 KVM 实例。

# 准备就绪

对于本教程，我们需要以下内容：

+   数据库服务器、消息队列和 `memcached` 已安装并配置，具体内容请参见 *为 OpenStack 部署准备主机* 章节。

+   Glance 服务和可用镜像。有关如何部署 Glance 并添加新镜像的信息，请参见 *安装和配置 OpenStack Glance 镜像服务* 章节。

+   我们在 *安装和配置 OpenStack Keystone 身份服务* 章节中部署的 Keystone 服务。

+   在 *安装和配置 OpenStack Nova 计算服务* 章节中配置的 Nova 服务。

+   在 *安装和配置 OpenStack Neutron 网络服务* 章节中部署的 Neutron 服务。

# 如何操作...

使用 OpenStack **命令行接口** (**CLI**) 构建一个新的 KVM 实例，请执行以下步骤：

1.  确保我们有一个可用的 Glance 镜像：

```
root@controller:~# openstack image list
+--------------------------------------+--------------+--------+
| ID                                   | Name         | Status |
+--------------------------------------+--------------+--------+
| abce08d2-2f9f-4545-a414-32019d41c0cd | ubuntu_16.04 | active |
+--------------------------------------+--------------+--------+
root@controller:~#

```

1.  创建一个新的实例类型：

```
root@controller:~# openstack flavor create --id 0 --vcpus 1 --ram 1024 --disk 5000 kvm.medium
+----------------------------+------------+
| Field                      | Value      |
+----------------------------+------------+
| OS-FLV-DISABLED:disabled   | False      |
| OS-FLV-EXT-DATA:ephemeral  | 0          |
| disk                       | 5000       |
| id                         | 0          |
| name                       | kvm.medium |
| os-flavor-access:is_public | True       |
| properties                 |            |
| ram                        | 1024       |
| rxtx_factor                | 1.0        |
| swap                       |            |
| vcpus                      | 1          |
+----------------------------+------------+
root@controller:~#
root@controller:~# openstack flavor list
+----+------------+------+------+-----------+-------+-----------+
| ID | Name       | RAM  | Disk | Ephemeral | VCPUs | Is Public |
+----+------------+------+------+-----------+-------+-----------+
| 0 | kvm.medium  | 1024 | 5000 | 0         | 1     | True      |
+----+------------+------+------+-----------+-------+-----------+
root@controller:~#

```

1.  创建一个新的 SSH 密钥对：

```
root@controller:~# openstack keypair create --public-key ~/.ssh/kvm_rsa.pub kvmkey
+-------------+-------------------------------------------------+
| Field       | Value                                           |
+-------------+-------------------------------------------------+
| fingerprint | e9:7e:e6:05:8b:a4:31:c3:5e:41:65:0e:29:23:eb:2a |
| name        | kvmkey                                          |
| user_id     | cc14c5dbbd654c438e52d38efaf4f1a6                |
+-------------+-------------------------------------------------+
root@controller:~# openstack keypair list
+--------+-------------------------------------------------+
| Name   | Fingerprint                                     |
+--------+-------------------------------------------------+
| kvmkey | e9:7e:e6:05:8b:a4:31:c3:5e:41:65:0e:29:23:eb:2a |
+--------+-------------------------------------------------+
root@controller:~#

```

1.  定义允许 SSH 和 ICMP 访问的安全组规则：

```
root@controller:~# openstack security group rule create --proto icmp default

+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| created_at        | 2017-04-26T18:17:13Z                 |
| description       |                                      |
| direction         | ingress                              |
| ethertype         | IPv4                                 |
| headers           |                                      |
| id                | ca28501a-1b3b-448f-8c1b-0fa6f9fa9263 |
| port_range_max    | None                                 |
| port_range_min    | None                                 |
| project_id        | 123c1e6f33584dd1876c0a34249a6e11     |
| project_id        | 123c1e6f33584dd1876c0a34249a6e11     |
| protocol          | icmp                                 |
| remote_group_id   | None                                 |
| remote_ip_prefix  | 0.0.0.0/0                            |
| revision_number   | 1                                    |
| security_group_id | 050b8174-d961-4706-ab63-1cdd2a25fbdd |
| updated_at        | 2017-04-26T18:17:13Z                 |
+-------------------+--------------------------------------+
root@controller:~#
root@controller:~# openstack security group rule create --proto tcp --dst-port 22 default

+-------------------+--------------------------------------+
| Field             | Value                                |
+-------------------+--------------------------------------+
| created_at        | 2017-04-26T18:17:18Z                 |
| description       |                                      |
| direction         | ingress                              |
| ethertype         | IPv4                                 |
| headers           |                                      |
| id                | 334130c3-42b2-4f1b-aba6-c46e91ad203e |
| port_range_max    | 22                                   |
| port_range_min    | 22                                   |
| project_id        | 123c1e6f33584dd1876c0a34249a6e11     |
| project_id        | 123c1e6f33584dd1876c0a34249a6e11     |
| protocol          | tcp                                  |
| remote_group_id   | None                                 |
| remote_ip_prefix  | 0.0.0.0/0                            |
| revision_number   | 1                                    |
| security_group_id | 050b8174-d961-4706-ab63-1cdd2a25fbdd |
| updated_at        | 2017-04-26T18:17:18Z                 |
+-------------------+--------------------------------------+
root@controller:~#

```

1.  列出之前定义的可用网络：

```
root@controller:~# openstack network list
+--------------------------------------+------+--------------------------------------+
| ID                                   | Name | Subnets                              |
+--------------------------------------+------+--------------------------------------+
| b7ccb514-21fc-4ced-b74f-026e7e358bba | nat  | 296250a7-f241-4f84-adbb-64a45c943094 |
+--------------------------------------+------+--------------------------------------+
root@controller:~#

```

1.  创建一个新的 KVM 实例并列出其状态：

```
root@controller:~# openstack server create --flavor kvm.medium --image ubuntu_16.04 --nic net-id=b7ccb514-21fc-4ced-b74f-026e7e358bba --security-group default --key-name kvmkey ubuntu_instance
+--------------------------------------+-------------------------------------------+
| Field                                | Value                                     |
+--------------------------------------+-------------------------------------------+
| OS-DCF:diskConfig                    | MANUAL                                    |
| OS-EXT-AZ:availability_zone          |                                           |
| OS-EXT-SRV-ATTR:host                 | None                                      |
| OS-EXT-SRV-ATTR:hypervisor_hostname  | None                                      |
| OS-EXT-SRV-ATTR:instance_name        |                                           |
| OS-EXT-STS:power_state               | NOSTATE                                   |
| OS-EXT-STS:task_state                | scheduling                                |
| OS-EXT-STS:vm_state                  | building                                  |
| OS-SRV-USG:launched_at               | None                                      |
| OS-SRV-USG:terminated_at             | None                                      |
| accessIPv4                           |                                           |
| accessIPv6                           |                                           |
| addresses                            |                                           |
| adminPass                            | Z23yEuDLBjLe                              |
| config_drive                         |                                           |
| created                              | 2017-04-26T19:11:23Z                      |
| flavor                               | kvm.medium (0)                            |
| hostId                               |                                           |
| id                                   | 0f4745b1-9d4b-4e8a-82f7-9eaa1f9bb08f      |
| image                                | ubuntu_16.04 (abce08d2-a414-32019d41c0cd) |
| key_name                             | kvmkey                                    |
| name                                 | ubuntu_instance                           |
| os-extended-volumes:volumes_attached | []                                        |
| progress                             | 0                                         |
| project_id                           | 123c1e6f33584dd1876c0a34249a6e11          |
| properties                           |                                           |
| security_groups                      | [{u'name': u'default'}]                   |
| status                               | BUILD                                     |
| updated                              | 2017-04-26T19:11:23Z                      |
| user_id                              | cc14c5dbbd654c438e52d38efaf4f1a6          |
+--------------------------------------+-------------------------------------------+
root@controller:~# openstack server list
+---------------------+-----------------+---------+------------------+--------------+
| ID                  | Name            | Status  | Networks         | Image Name   |
+---------------------+-----------------+---------+------------------+--------------+
| 0f4745b1-...-9bb08f | ubuntu_instance | BUILD   | nat=192.168.0.11 | ubuntu_16.04 |
+---------------------+-----------------+---------+------------------+--------------+
root@controller:~#

```

1.  确保容器已成功启动：

```
root@controller:~# pgrep -lfa qemu
23388 /usr/bin/qemu-system-x86_64 -name instance-00000005 -S -machine pc-i440fx-xenial,accel=kvm,usb=off -cpu Haswell-noTSX,+abm,+pdpe1gb,+rdrand,+f16c,+osxsave,+dca,+pdcm,+xtpr,+tm2,+est,+smx,+vmx,+ds_cpl,+monitor,+dtes64,+pbe,+tm,+ht,+ss,+acpi,+ds,+vme -m 1024 -realtime mlock=off -smp 1,sockets=1,cores=1,threads=1 -uuid 0f4745b1-9d4b-4e8a-82f7-9eaa1f9bb08f -smbios type=1,manufacturer=OpenStack Foundation,product=OpenStack Nova,version=14.0.4,serial=6d6366d9-4569-6233-dad6-4927587cc79f,uuid=0f4745b1-9d4b-4e8a-82f7-9eaa1f9bb08f,family=Virtual Machine -no-user-config -nodefaults -chardev socket,id=charmonitor,path=/var/lib/libvirt/qemu/domain-instance-00000005/monitor.sock,server,nowait -mon chardev=charmonitor,id=monitor,mode=control -rtc base=utc,driftfix=slew -global kvm-pit.lost_tick_policy=discard -no-hpet -no-shutdown -boot strict=on -device piix3-usb-uhci,id=usb,bus=pci.0,addr=0x1.0x2 -drive file=/var/lib/nova/instances/0f4745b1-9d4b-4e8a-82f7-9eaa1f9bb08f/disk,format=qcow2,if=none,id=drive-virtio-disk0,cache=none -device virtio-blk-pci,scsi=off,bus=pci.0,addr=0x4,drive=drive-virtio-disk0,id=virtio-disk0,bootindex=1 -netdev tap,fd=26,id=hostnet0,vhost=on,vhostfd=28 -device virtio-net-pci,netdev=hostnet0,id=net0,mac=fa:16:3e:3c:c0:0f,bus=pci.0,addr=0x3 -chardev file,id=charserial0,path=/var/lib/nova/instances/0f4745b1-9d4b-4e8a-82f7-9eaa1f9bb08f/console.log -device isa-serial,chardev=charserial0,id=serial0 -chardev pty,id=charserial1 -device isa-serial,chardev=charserial1,id=serial1 -device usb-tablet,id=input0 -vnc 0.0.0.0:0 -k en-us -device cirrus-vga,id=video0,bus=pci.0,addr=0x2 -device virtio-balloon-pci,id=balloon0,bus=pci.0,addr=0x5 -msg timestamp=on
root@controller:~# openstack server list
+---------------------------+-----------------+--------+------------------+--------------+
| ID                        | Name            | Status | Networks         | Image Name   |
+---------------------------+-----------------+--------+------------------+--------------+
| 0f4745b1-...-9eaa1f9bb08f | ubuntu_instance | ACTIVE | nat=192.168.0.11 | ubuntu_16.04 |
+---------------------------+-----------------+--------+------------------+--------------+
root@controller:~#

```

1.  检查 KVM 实例：

```
root@controller:~# openstack server show ubuntu_instance
+--------------------------------------+-------------------------------------------+
| Field                                | Value                                     |
+--------------------------------------+-------------------------------------------+
| OS-DCF:diskConfig                    | MANUAL                                    |
| OS-EXT-AZ:availability_zone          | nova                                      |
| OS-EXT-SRV-ATTR:host                 | controller                                |
| OS-EXT-SRV-ATTR:hypervisor_hostname  | controller                                |
| OS-EXT-SRV-ATTR:instance_name        | instance-00000001                         |
| OS-EXT-STS:power_state               | Running                                   |
| OS-EXT-STS:task_state                | None                                      |
| OS-EXT-STS:vm_state                  | active                                    |
| OS-SRV-USG:launched_at               | 2017-04-26T19:11:37.000000                |
| OS-SRV-USG:terminated_at             | None                                      |
| accessIPv4                           |                                           |
| accessIPv6                           |                                           |
| addresses                            | nat=192.168.0.11                          |
| config_drive                         |                                           |
| created                              | 2017-04-26T19:11:23Z                      |
| flavor                               | kvm.medium (0)                            |
| hostId                               | c8d0t2jgdlkasdjg0iu4kjdg3o43045t          |
| id                                   | 0f4745b1-9d4b-4e8a-82f7-9eaa1f9bb08f      |
| image                                | ubuntu_16.04 (abce08d2-a414-32019d41c0cd) |
| key_name                             | kvmkey                                    |
| name                                 | ubuntu_instance                           |
| os-extended-volumes:volumes_attached | []                                        |
| progress                             | 0                                         |
| project_id                           | 123c1e6f33584dd1876c0a34249a6e11          |
| properties                           |                                           |
| security_groups                      | [{u'name': u'default'}]                   |
| status                               | ACTIVE                                    |
| updated                              | 2017-04-26T19:11:23Z                      |
| user_id                              | cc14c5dbbd654c438e52d38efaf4f1a6          |
+--------------------------------------+-------------------------------------------+
root@controller:~#

```

# 它是如何工作的……

我们通过确保有可用的 Glance 镜像可以选择来开始本食谱。在第 1 步中，我们列出了 Glance 中所有可用的镜像。在第 2 步中，我们创建了一个新的实例规格；我们为新的实例类型指定了分配的 CPU、内存和磁盘资源。在第 3 步中，虽然不是强制性的，但我们创建了一个新的 SSH 密钥对，稍后可以用来 SSH 登录到新实例。在第 4 步中，我们创建了两个新的安全组规则，允许 SSH 和 ICMP 流量。如果我们希望能够 ping 和 SSH 到新实例，这将非常方便。在创建实例之前，我们需要列出 Neutron 中的可用网络，这些网络将成为客户机的一部分；我们在第 5 步中完成了这一步。

在所有先前的先决条件都已就绪后，我们在第 6 步中创建了一个新的 KVM 实例，指定了实例规格、Glance 镜像、网络、安全组和 SSH 密钥。然后，我们继续列出该实例的状态。注意，任务状态显示为 scheduling，这意味着 `nova-scheduler` 正在选择一个主机来部署该实例，状态是 BUILD。由于我们在这个示例部署中只使用一个主机，实例将在同一计算服务器上部署。从构建命令的输出中，我们还可以看到分配给新实例的 IP 地址。

在第 7 步中，我们可以看到新的实例已成功创建，其状态现在显示为 ACTIVE，且已启动新的 QEMU 进程。

最后，在第 8 步中，我们检查正在运行的实例；请注意，电源状态字段现在显示为 Running，状态字段显示为 active。

# 使用 OpenStack 停止 KVM 实例

在这个简短的食谱中，我们将使用熟悉的 `openstack` 命令语法停止上一个食谱中我们创建的正在运行的 KVM 实例。

# 准备工作

对于这个食谱，我们需要以下内容：

+   一个已安装并配置的数据库服务器、消息队列和 `memcached`，如 *为 OpenStack 部署准备主机* 食谱中所述。

+   一个可用镜像的 Glance 服务。有关如何部署 Glance 并添加新镜像的信息，请参阅 *安装和配置 OpenStack Glance 镜像服务* 食谱。

+   我们在 *安装和配置 OpenStack Keystone 身份服务* 食谱中部署的 Keystone 服务。

+   我们在 *安装和配置 OpenStack Nova 计算服务* 食谱中配置的 Nova 服务。

+   我们在 *安装和配置 OpenStack Neutron 网络服务* 食谱中部署的 Neutron 服务。

+   一个由 OpenStack 部署的正在运行的 KVM 实例。

# 如何操作……

要使用 OpenStack 停止正在运行的 KVM 客户机，请执行以下简单步骤：

1.  列出已部署的 OpenStack 实例：

```
root@controller:~# openstack server list
+---------------------------+-----------------+--------+------------------+--------------+
| ID                        | Name            | Status | Networks         | Image Name   |
+---------------------------+-----------------+--------+------------------+--------------+
| 0f4745b1-...-9eaa1f9bb08f | ubuntu_instance | ACTIVE | nat=192.168.0.11 | ubuntu_16.04 |
+---------------------------+-----------------+--------+------------------+--------------+
root@controller:~#

```

1.  停止实例：

```
root@controller:~# openstack server stop ubuntu_instance
root@controller:~#

```

1.  使用 libvirt 列出 KVM 客户机：

```
root@controller:~# virsh list --all
 Id Name              State
----------------------------------------------------
 -  instance-00000001 shut off

root@controller:~#

```

1.  确保实例的 QEMU 进程已终止：

```
root@controller:~# pgrep -lfa qemu
root@controller:~#

```

1.  检查 KVM 来宾的状态：

```
root@controller:~# openstack server list
+---------------------------+-----------------+--------+------------------+--------------+
| ID                        | Name            | Status | Networks         | Image Name   |
+---------------------------+-----------------+--------+------------------+--------------+
| 0f4745b1-...-9eaa1f9bb08f | ubuntu_instance | SHUTOFF| nat=192.168.0.11 | ubuntu_16.04 |
+---------------------------+-----------------+--------+------------------+--------------+
root@controller:~#

```

# 它是如何工作的...

我们从第 1 步开始，列出由 OpenStack 配置的可用 KVM 实例。在第 2 步中，我们通过指定实例的名称来停止实例。请注意，我们也可以使用实例 ID 来停止它。由于 OpenStack 使用 libvirt 管理 KVM 实例的生命周期，在第 3 步中，我们看到该实例确实已经被销毁。在第 4 步中，我们确保该实例的 QEMU 进程也已经终止。在最后一步，我们可以看到实例的状态现在是 SHUTOFF，而不是 ACTIVE。处于此状态的实例可以通过执行以下命令重新启动：

```
root@controller:~# openstack server start ubuntu_instance
root@controller:~# openstack server list
+---------------------------+-----------------+--------+------------------+--------------+
| ID                        | Name            | Status | Networks         | Image Name   |
+---------------------------+-----------------+--------+------------------+--------------+
| 0f4745b1-...-9eaa1f9bb08f | ubuntu_instance | ACTIVE | nat=192.168.0.11 | ubuntu_16.04 |
+---------------------------+-----------------+--------+------------------+--------------+
root@controller:~#

```

# 使用 OpenStack 终止 KVM 实例

在本食谱中，我们将终止一个由 OpenStack 配置的 KVM 实例。终止实例会通过 libvirt 将其取消定义，释放分配的 CPU、内存和磁盘资源回到可用资源池，并将其 IP 地址在 Neutron 数据库中标记为可用。

# 准备就绪

对于这个食谱，我们需要以下内容：

+   已安装并配置数据库服务器、消息队列和`memcached`，如*为 OpenStack 部署准备主机*食谱中所述。

+   可用镜像的 Glance 服务。有关如何部署 Glance 并添加新镜像的更多信息，请参考*安装和配置 OpenStack Glance 镜像服务*食谱。

+   我们在*安装和配置 OpenStack Keystone 身份服务*食谱中部署的 Keystone 服务。

+   我们在*安装和配置 OpenStack Nova 计算服务*食谱中配置的 Nova 服务。

+   在*安装和配置 OpenStack Neutron 网络服务*食谱中部署的 Neutron 服务。

+   一个正在运行的 KVM 实例，由 OpenStack 配置。

# 如何操作...

要终止一个正在运行的实例，请执行以下步骤：

1.  获取要终止的实例的名称或 ID：

```
root@controller:~# openstack server start ubuntu_instance
root@controller:~# openstack server list
+---------------------------+-----------------+--------+------------------+--------------+
| ID                        | Name            | Status | Networks         | Image Name   |
+---------------------------+-----------------+--------+------------------+--------------+
| 0f4745b1-...-9eaa1f9bb08f | ubuntu_instance | ACTIVE | nat=192.168.0.11 | ubuntu_16.04 |
+---------------------------+-----------------+--------+------------------+--------------+
root@controller:~#

```

1.  通过提供名称删除实例：

```
root@controller:~# openstack server delete ubuntu_instance
root@controller:~#

```

1.  确保实例已被取消定义：

```
root@controller:~# openstack server list

root@controller:~# virsh list --all
 Id   Name   State
----------------------------------------------------

root@controller:~#

```

1.  检查`nova-api`、`neutron-server`和`nova-compute`的日志：

```
root@controller:~# cat /var/log/nova/nova-api.log | grep -i delete
2017-05-04 15:30:07.733 20915 INFO nova.osapi_compute.wsgi.server [req-54dbe80f-9942-43d8-949a-d80daa2440a9 cc14c5dbbd654c438e52d38efaf4f1a6 123c1e6f33584dd1876c0a34249a6e11 - default default] 10.184.226.74 "DELETE /v2.1/123c1e6f33584dd1876c0a34249a6e11/servers/0f4745b1-9d4b-4e8a-82f7-9eaa1f9bb08f HTTP/1.1" status: 204 len: 339 time: 0.1859989
root@controller:~#
root@controller:~# cat /var/log/neutron/neutron-server.log | grep -i delete
2017-05-04 15:30:08.402 17910 INFO neutron.wsgi [req-5c9674d6-c596-4b17-b975-54625ac7adb2 cc14c5dbbd654c438e52d38efaf4f1a6 123c1e6f33584dd1876c0a34249a6e11 - - -] 10.184.226.74 - - [04/May/2017 15:30:08] "DELETE /v2.0/ports/fdaf6ea1-b76a-4895-a028-db15831132fa.json HTTP/1.1" 204 173 0.320351
root@controller:~#
root@controller:~# cat /var/log/nova/nova-compute.log
...
2017-05-04 15:30:07.747 21065 INFO nova.compute.manager [req-54dbe80f-9942-43d8-949a-d80daa2440a9 cc14c5dbbd654c438e52d38efaf4f1a6 123c1e6f33584dd1876c0a34249a6e11 - - -] [instance: 0f4745b1-9d4b-4e8a-82f7-9eaa1f9bb08f] Terminating instance
2017-05-04 15:30:07.952 21065 INFO nova.virt.libvirt.driver [-] [instance: 0f4745b1-9d4b-4e8a-82f7-9eaa1f9bb08f] Instance destroyed successfully.
2017-05-04 15:30:07.953 21065 INFO os_vif [req-54dbe80f-9942-43d8-949a-d80daa2440a9 cc14c5dbbd654c438e52d38efaf4f1a6 123c1e6f33584dd1876c0a34249a6e11 - - -] Successfully unplugged vif VIFBridge(active=True,address=fa:16:3e:3c:c0:0f,bridge_name='brqb7ccb514-21',has_traffic_filtering=True,id=fdaf6ea1-b76a-4895-a028-db15831132fa,network=Network(b7ccb514-21fc-4ced-b74f-026e7e358bba),plugin='linux_bridge',port_profile=<?>,preserve_on_delete=False,vif_name='tapfdaf6ea1-b7')
2017-05-04 15:30:07.970 21065 INFO nova.virt.libvirt.driver [req-54dbe80f-9942-43d8-949a-d80daa2440a9 cc14c5dbbd654c438e52d38efaf4f1a6 123c1e6f33584dd1876c0a34249a6e11 - - -] [instance: 0f4745b1-9d4b-4e8a-82f7-9eaa1f9bb08f] Deleting instance files /var/lib/nova/instances/0f4745b1-9d4b-4e8a-82f7-9eaa1f9bb08f_del
2017-05-04 15:30:07.974 21065 INFO nova.virt.libvirt.driver [req-54dbe80f-9942-43d8-949a-d80daa2440a9 cc14c5dbbd654c438e52d38efaf4f1a6 123c1e6f33584dd1876c0a34249a6e11 - - -] [instance: 0f4745b1-9d4b-4e8a-82f7-9eaa1f9bb08f] Deletion of /var/lib/nova/instances/0f4745b1-9d4b-4e8a-82f7-9eaa1f9bb08f_del complete
...
root@controller:~#

```

# 它是如何工作的...

在第 1 步中，我们通过列出 Nova 所知道的所有实例，并记录下我们想要删除的实例的名称。

在第 2 步中，我们通过指定实例的名称来删除该实例。请注意，我们也可以使用实例的 ID 来删除。在第 3 步中，我们确认该实例已被 libvirt 取消定义，并且在 OpenStack 中不再可用。

在第 4 步中，我们可以看到发送到`nova-api`、`neutron-server`和`nova-compute`服务的 API 调用及其采取的操作。
