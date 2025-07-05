# 第九章：部署 Apache HTTPD 服务器

**超文本传输协议**（**HTTP**）服务器通常被称为 Web 服务器。顾名思义，这是一个为客户端（通常是互联网中的 Web 浏览器）提供内容的网络服务。这通常意味着提供网页，但也可以提供其他文档，如图像、声音、视频，甚至是 RHEL 的 ISO 文件。

在 RHEL 7 上打包的 Web 服务器是 Apache `httpd` 服务。这是互联网上最常见的 Web 服务器，由 Apache 软件基金会开发。RHEL 已将 `httpd` 更新到 2.4 版本，取代了之前版本 RHEL 中使用的 2.2 版本。

本章将涵盖以下主题：

+   配置 `httpd` 服务

+   控制 `httpd` 服务

+   添加服务器模块

+   使用虚拟主机

# 配置 `httpd` 服务

Apache `httpd` Web 服务器可以将页面提供给互联网上的客户端或我们内部局域网中的客户端，因此不要觉得部署 Web 服务器时一定需要加强安全性。当然，如果网站面向互联网，可能需要额外的安全性和隔离服务。然而，我们正在实验室环境中工作，将更多关注 Web 服务器的配置。

## 安装 Apache 2.4

默认情况下可能没有安装所需的包，因此我们至少需要添加 `httpd` 包。此外，你可能还想添加文档。考虑仅在开发服务器上添加文档；我不建议在生产服务器上添加文档。我们将按以下方式将两个包添加到服务器：

```
$ sudo yum install httpd httpd-manual

```

即使在这个阶段，添加这么少的工作量，我们也可以通过以下命令启动服务并使用浏览器访问 Web 服务器：

```
$ sudo systemctl start httpd
$ sudo systemct enable httpd

```

如果我们有图形环境，可以从本地系统使用 Firefox 浏览器访问本地主机。我们将看到一个类似于以下截图的欢迎页面：

![Installing Apache 2.4](img/image00299.jpeg)

虽然我希望认为我的工作已经完成；但不知为何，我感觉你可能还需要一点额外的指导。

## 配置

在 RHEL 7 上，`httpd` 服务的配置基础是 `/etc/httpd` 目录。通过 `tree` 命令，我们可以有效地展示配置层次或服务。这里需要的唯一配置是 `httpd.conf`，但是 Red Hat 采用了更加模块化的方法，现在包括许多子配置文件到主文件中。以下是 `tree` 输出的截图，显示了默认安装后所有文件的位置：

![The configuration](img/image00300.jpeg)

### 提示

如果前述的 `tree` 没有安装，可以通过 `sudo yum install tree` 安装。

我们已经从`/etc/httpd`目录运行了`tree`命令。我们可以看到`/etc/httpd/conf/httpd.conf`文件，这是主配置文件。它包括来自`/etc/httpd/conf.d`和`/etc/httpd/conf.modules.d`的其他文件。此外，还有三个符号链接的目录：

+   `logs`

+   `modules`

+   `run`

我们在浏览网站时看到的页面内容来自于`/etc/httpd/conf.d/welcome.conf`中的配置。当没有实际网站内容时，会生成默认的欢迎页面。

随着`userdir.conf`和`autoindex.conf`文件的引入，独立模块配置与 RHEL 6 中的`httpd`配置有很大不同，后者将这些文件都包含在主`httpd.conf`文件中。

我们已经提到过，Web 服务器的配置根目录是`/etc/httpd`目录。相关配置文件位于`/etc/httpd/conf/httpd.conf`中。该文件中的一些关键指令如下：

+   `ServerRoot`：`/etc/httpd`

+   `DocumentRoot`：`/var/www/html`

+   `DirectoryIndex`：`index.html`

`ServerRoot`指令，正如我们所看到的，它是我们可以找到 Web 服务器配置、日志和模块的位置。`DocumentRoot`指令表示可以找到 Web 内容的位置，而`DirectoryIndex` HTML 页面是默认的查找页面。通过使用`echo`，我们可以简单地创建我们自己的内容，如下所示。我们以 root 用户身份运行以下命令，创建一个非常基础的欢迎页面：

```
# echo '<h1>Welcome to our site</h1>' > /var/www/html/index.html

```

使用单引号允许将标签作为字面值传递。

我们可以直接查看页面；由于我们没有更改任何配置，因此无需重新加载服务。此时页面应类似于以下截图所示：

![配置](img/image00301.jpeg)

我们可以通过主机名或 IP 地址在本地或远程访问此站点。我们可以使用`http://192.168.0.69`（或分配给主机接口的任何 IP 地址）访问相同的页面。要远程访问页面，防火墙需要在其`firewalld`规则中包括`http`和`https`：

在以下代码中，我们打开了`80`端口的 HTTP 端口和`443`端口的 HTTPS 端口。这些是用于访问 Web 服务器的协议和默认端口：

```
$ sudo firewall-cmd --add-service=http --permanent
$ sudo firewall-cmd --add-service=https --permanent
$ sudo firewall-cmd --reload

```

作为 Web 服务器的管理员，我们不一定是网站内容的开发者，但我们可以证明 Web 服务器在配置后正常工作。

## 配置 `DocumentRoot` 目录

`DocumentRoot`目录应该对`httpd`服务可读。服务使用的帐户列在`httpd.conf`中。与相应指令一起使用的默认帐户如下所示：

```
User apache
Group apache

```

在理想情况下，`/var/www/html`目录的权限应为八进制表示的`2750`或符号表示的`rwx r_s _`。为目录设置组特殊位确保该目录中新创建的所有内容将由目录的组所有者拥有。通过这种方式，我们无需授予其他用户任何权限；只要目录归`apache`组所有，文件就会对该组可访问。

首先，我们将为这个目录设置组所有权。由于我们已经在该目录之后创建了`index`页面，因此我们将使用`-R`选项，如以下命令所示：

```
$ sudo chgrp -R apache /var/www/html 

```

现在，我们将为该目录设置组特殊位；这确保了在该结构中创建的所有新文件和目录都将归`apache`组所有：

```
$ sudo chmod g+s /var/www/html/

```

最后，我们将移除授予其他用户的权限，从而帮助保护内容，具体如下：

```
$ sudo chmod -R o= /var/www/html/

```

虽然这并非绝对必要，但在一开始配置这个选项可以为以后可能需要更高安全性的情况节省工作。在`httpd.conf`中，我们还有一个目录块，用来配置`DocumentRoot`的访问和选项。以下截图显示了与`DocumentRoot`目录相关的目录块：

![配置 DocumentRoot 目录](img/image00302.jpeg)

`Directory`块类似于基于 XML 的数据。它使用一个开始标签并在其中设置目标目录。该块以`</Directory>`标签结束：

```
Options Indexes FollowSymLinks

```

`Indexes`选项允许创建索引页面。如果客户端访问的 URL 中没有包含页面名称，并且没有`index.html`，该选项会列出目录的内容。这对下载目录可能非常有用，避免了你需要创建一个页面来链接所有可用的下载内容；然而，在`DocumentRoot`中，我们可能不希望启用这个设置，因为它可能会带来安全风险。

`FollowSymLinks`选项可能不言自明，它允许你跟踪符号链接的路径。符号链接是指向文件系统中其他文件和目录的指针。

`AllowOverride`指令指定可以通过用户控制的`.htaccess`文件使用的设置。对于虚拟主机来说，管理员可能无法访问 Web 服务器的配置文件，因为他们只是租用服务器空间。他们可以通过将`.htaccess`文件上传到与`Directory`块相关的目录根目录来实现配置。作为主服务器管理员，你可以控制可以读取的设置；在这里，我们不允许即使`.htaccess`文件存在也读取任何设置：

```
AllowOverride None

```

这里的最终设置是 Apache 2.4 中的新功能，它取代了 Apache 2.2 及更早版本中的`Allow from / Deny from`指令：

```
Require all granted

```

默认设置相当于 Apache 2.2 中的`Allow from all`设置。

如果需要，你可以使用类似于以下命令的配置来调整主机访问：

```
Require 192.168.0.0/24
Require 127.0.0.1

```

前面的访问控制只允许 localhost 和 `192.168.0` 网络访问相关的 `Directory` 块。

我们将通过编辑 `/etc/httpd/conf/httpd.conf` 文件来更改配置设置。一旦我们找到 `/var/www/html` 的正确 `Directory` 块，就可以移除 `Index` 选项并将其保留如下：

```
Options FollowSymLinks

```

这些更改应该被保存，但我们需要查看在尝试重启服务器之前，如何检查我们的设置。

# 控制 Apache 网络服务

当我们准备好测试所做的更改时，可以使用以下命令进行预飞行检查：

```
# apachectl configtest

```

你可能会收到一个错误信息，报告无法解析主机名。这是一个警告，目前没有问题。警告信息如下截图所示：

![Controlling the Apache web service](img/image00303.jpeg)

末尾的 `Syntax Ok` 消息是我们希望看到的；有了这个信息，我们就知道可以重启 web 服务器了。向服务发出 `reload` 命令将强制进行优雅重启；在启动重启之前，等待活跃连接完成：

```
$ sudo systemctl reload httpd

```

更改的影响是有限的，因为我们已经允许了本地网络和本地主机的访问。移除 `indexes` 选项没有效果，前提是 `index.html` 页面存在。如果我们移除 `index.html` 页面，将会收到访问被禁止的消息，因为系统将无法生成该页面。这一点很重要，因为它阻止了黑客能够获取我们网站服务器的目录列表。

## 设置服务器名称

我们还需要整理一下关于服务器名称的警告。这个问题是通过在 `httpd.conf` 文件中使用 `ServerName` 指令来控制的。在文件的顶部添加 `ServerName` 指令将解决该问题：

```
ServerName web.theurbanpenguin.com

```

要完全重启服务，我们将使用以下命令：

```
$ sudo systemctl restart httpd

```

## 设置自定义错误页面

如果我们尝试访问服务器上不存在的页面，浏览器将显示一个标准的 `页面未找到` 信息。我们可以通过添加自己的自定义页面来让这个过程更加可控和用户友好。

在 `httpd.conf` 的全局部分，我们可以添加以下指令来处理 `404 页面未找到` 错误。这个全局部分可以影响所有的目录块，但如果需要，我们也可以仅对单个目录块添加该指令。为 404 错误添加代码只会影响该特定错误，但我们也可以根据需要添加其他代码：

```
ErrorDocument 404 /404.html

```

我们可以使用以下命令重新加载服务器：

```
$ sudo systemctl reload httpd

```

现在，当访问错误的页面时，观看者将会看到我们创建的自定义错误页面：`404.html`。此页面应创建在`DocumentRoot`中，因为我们使用了`/404.html`语法。如果我们有许多自定义页面，我们很可能会在`DocumentRoot`中创建一个`error`目录，然后将页面引用为`/error/404.html`。

# 加载模块

Red Hat 已经摆脱了在标准的`httpd.conf`文件中加载模块的方式。在之前的 Red Hat 版本（版本 6）中，配置文件可能会被许多`LoadModule`指令所填满。

这些模块现在通过配置文件在`/etc/httpd/conf.modules.d/`中加载。这样，主配置文件变得不那么混乱，可以根据需要轻松添加额外的配置文件。

要查看从命令提示符中当前加载的模块，请使用以下命令：

```
$ sudo httpd -M

```

我们可以看到我们加载了许多模块。我们可以将输出导入`wc`命令来计算行数。使用 RHEL 7.1 演示系统，输出为`82`：

```
$ sudo httpd -M | wc -l

```

通过原始输出，我们应该能够看到`userdir_module`已加载。如果我们不需要在 Web 服务器上支持用户主目录，我们就不需要这个模块。要加载此模块，将引用此 Apache 模块的`LoadModule`指令设置在`/etc/httpd/conf.modules.d/00-base.conf`文件中。为确保在将来重新启动 Web 服务器时不加载它，请注释掉如下行：

```
LoadModule userdir_module modules/mod_userdir.so

```

现在注释掉该行后，您需要重新启动 Web 服务器，但在检查加载的模块时，应能够验证`userdir_module`未加载。

# 虚拟服务器

Apache 有能力从同一服务器实例支持多个站点。这提供了很大的灵活性，同时也便于管理。这种灵活性称为虚拟主机。在 Apache 中有三种基本的虚拟主机运行方式：

| **基于名称** | 这为每个站点使用不同的名称和一个公共 IP 地址，可能是最流行的虚拟主机形式之一 |
| --- | --- |
| **基于 IP** | 这使用不同的 IP 地址为每个站点服务 |
| **基于端口** | 这使用各自的端口号为每个站点服务 |

我们将查看实施每种方法所需的所有三种方法和`httpd.conf`中的配置。

## 基于名称

基于名称的虚拟主机是通过引入 HTTP 协议版本 1.1（也称为 HTTP/1.1）变得可能的。当浏览器只能支持 HTTP/1.0 时，协议尝试加载网页，经历以下步骤：

1.  将`theurbanpinguin.org`主机名解析为一个 IP 地址。

1.  通过 TCP 协议和端口`80`连接到解析后的 IP 地址。

1.  请求`/index.html`页面。

    因此，在任何给定的 IP 地址上，只能托管一个网站（由其域名定义）。如果将另一个域名—例如 `theboldeagle.net`—指向同一个 IP 地址，那么用户访问 URL 时将看到与访问另一个 URL 完全相同的内容，因为 Web 服务器无法区分这两个请求。

    当 HTTP/1.1 协议和支持 HTTP/1.1 的浏览器将域名和文档路径发送给 Web 服务器时，下一步（第 3 步）如下所示：

1.  从 `theurbanpinguin.org` 服务器请求 `/index.html` 页面。

现在可以在同一个 IP 地址上托管两个或更多不同的网站，因为 Web 服务器可以区分不同的域名，因为域名现在是 HTTP 请求的一部分。

所有现代 Web 浏览器都支持 HTTP/1.1。基于名称的虚拟主机有两个方面，我们现在来看看。

### 域名解析

网站的所有名称需要通过 DNS 或本地主机文件映射到同一个 IP 地址。如果系统中使用了其中任何一个名称，返回的 IP 地址将始终相同。请求被重定向到文件系统中的正确位置，由 `httpd` 服务和传入的 `http` 头处理。

### Apache 配置

基于名称的虚拟主机的关键在于 `httpd.conf` 文件中的配置块指令：

```
<VirtualHost virtual host IP>
.
.
.
</VirtualHost>

```

几乎所有 Apache 条目都可以在这里使用，包括我们之前看到的 `ErrorDocument` 指令。我们在下面的示例中包含了最典型的条目：

```
<VirtualHost *.80>
 ServerName	www.packtpub.com
 ServerAdmin andrew@example.com
 DocumentRoot "/var/www/packt/html"
 ErrorLog "/var/www/packt/logs/packt_error"
 TransferLog "/var/www/packt/logs/packt_access"
</VirtualHost>

```

使用前面的示例，如果我们在访问服务器的 IP 地址时使用 [`www.packtpub.com/`](https://www.packtpub.com/) 的 URL，我们将被定向到 `/var/www/packt/html` 中的网页。

为了让 Apache 找到正确的虚拟主机条目，我们必须告诉它这些虚拟主机对应的 IP 地址。不过，仅仅使用之前的配置块定义虚拟主机是不够的，因为 Apache 默认不允许基于名称的虚拟主机。为了让基于名称的虚拟主机工作，我们需要使用 `NameVirtualHost` 指令，并将其包含在 `httpd.conf` 文件中。该指令必须位于主服务器部分：

```
NameVirtualHost *:80

```

## 基于 IP

使用的 IP 地址必须绑定到主 RHEL 服务器上。在 Apache 的 `httpd.conf` 中，我们再次使用 `VirtualHost` 配置块，但不需要设置 `NameVirtualHost` 指令。我们将不再使用 `*:80` 作为 `VirtualHost` 配置块的 IP 和端口选择符，而是使用 Apache 服务器监听的真实 IP 地址，即分配给 Apache 服务器所在机器某个接口的 IP 地址。以下命令显示了可以添加到主 `httpd.conf` 文件中的可能配置：

```
<VirtualHost 192.168.0.221:80>
  ServerName www.example.com
  ServerAdmin andrew@example.com
  DocumentRoot "/var/www/example/html"
</VirtualHost>
```

默认情况下，Apache 会监听机器上所有接口的传入连接，因此只需要在`<VirtualHost>`开头指令中指定一个接口的 IP 地址就足够了。然而，为了确保 Apache 监听到正确的接口，应该在`httpd.conf`文件的主配置部分中包含以下命令：

```
Listen 192.168.0.221:80

```

使用前面的示例，当我们通过 IP 地址`192.168.0.221`访问主机时，会被重定向到`/var/www/example/html`中的网页。

## 基于端口

端口基础的虚拟主机仅涉及一个方面：使用主配置文件`httpd.conf`进行的 Apache 配置。以下是一个示例：

```
<VirtualHost 192.168.0.220:7070>
  ServerName www.example.com
  ServerAdmin andrew@example.com
  DocumentRoot "/var/www/example/html"
</VirtualHost>
```

使用前面的示例，当我们通过 IP 地址为`192.168.0.220`的 Apache 主机访问端口`7070`时，会被重定向到`/var/www/example/html`中的网页。

在所有`VirtualHost`块中，除了我们迄今为止展示的代码之外，还可以预期会有一个`Directory`块。通过这种方式，可以为每个虚拟主机正确设置选项和访问控制列表。

# 自动化虚拟主机

如果我们为虚拟主机创建一个模板文件，就可以通过脚本轻松地添加新的虚拟主机。首先，我们需要一个类似以下命令的模板文件：

```
<VirtualHost *:80>
  ServerAdmin webmaster@dummy-host.example.com
  ServerName dummy-host.example.com
  DocumentRoot /var/www/dummy-host.example.com
  ErrorLog /var/log/httpd/dummy-host.example.com-error_log
  CustomLog /var/log/httpd/dummy-host.example.com-access_log
  UseCanonicalName Off
  ServerSignature On
  <Directory "/var/www/vhosts/dummy-host.example.com">
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
  </Directory>
</VirtualHost>
```

如果该文件保存为`/etc/httpd/conf.d/template`，由于文件名未以`.conf`结尾，它将不会被用作配置文件。我们可以使用它作为模板，并通过类似以下命令的脚本来使用它：

```
#!/bin/bash
CONFDIR=/etc/httpd/conf.d
WEBDIR=/var/www/
mkdir -p $WEBDIR/$1
sed s/"dummy-host.example.com"/$1/g \
$CONFDIR/template > $CONFDIR/$1.conf
echo "This is a website in construction for $1" \
> $WEBDIR/$1/index.html
systemctl reload httpd

```

如果脚本名为`/root/vhost.sh`，我们可以通过以下方式运行它：

```
# /root/vhost.sh www.example.com

```

上面的脚本将创建一个新的配置，并将`dummy-host.example.com`替换为`www.example.com`。

# 总结

本章介绍了在 RHEL 7.1 上运行的 Apache HTTPD 服务。我们查看了`ServerRoot`为`/etc/httpd`，`DocumentRoot`为`/var/www/html`。在掌握基础知识后，你学会了如何配置一个带有自定义错误页面和虚拟主机的服务器。

在下一章，我们将详细介绍 SELinux，并试图让你明白，使用 SELinux 时，不会对服务交付产生负面影响。实际上，“负面”这个词完全不准确。SELinux 将为你提供一个安全而稳健的平台，让你可以无惧安全威胁地部署面向公众的服务，并为现有的、较弱的**DAC**（**Discretionary Access Controls**）添加**强制访问控制**（**MAC**）。
