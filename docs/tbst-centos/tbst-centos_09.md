# 第九章：Web 服务故障排除

故障排除并不总是关于灾难恢复或修复损坏的系统。事实上，大多数故障排除者往往花时间发现不断改进系统的方法，或者协助其他同事充分利用现有技术。有些人会称之为 Dev/Ops，但无论你如何看待，基本原则始终不变。你是故障排除者，你是支持网络中至关重要的一部分；因此，考虑到这一点，我们将暂时放下“拯救世界”的任务，以更积极的方式来探讨 Web 服务的主题。

在本章中，我们将回顾调查 Web 服务的主题，目的是改进并扩展你作为故障排除者的知识。

在本章中，我们将：

+   学习如何使用 cURL 审核服务器

+   发现检查你的 Akamai 头信息的方法

+   学习如何在 Apache 上实现 Varnish

+   发现如何使用 cURL 验证你的 Varnish 安装

+   学习如何使用 cURL 访问 FTP 目录

+   学习如何通过安装`mod_status`来监控 Apache

# 使用 cURL 审核服务器

当 Web 服务器开始出现问题时，可能有很多原因。然而，作为故障排除者，在遇到这个问题时，请记住，你并不是在看应用程序本身（这是程序员的领域，他们不会感谢你加入其中），而是看服务器的状态。

本质上，你可以说这是一个审查服务器及其提供网页或 Web 应用程序能力的过程。那么，让我们从检查确认 cURL 是否已安装开始。

为此，你应该使用以下语法：

```
# yum install curl

```

完成此步骤后，你现在可以运行你的第一个 cURL 命令了：

```
$ curl http://www.example.com

```

更具体地说，你可以通过以下方式选择特定的位置：

```
$ curl http://www.example.com/path/to/homepage.html

```

或者，你可以像这样传递一个字符串：

```
$ curl http://www.example.com/path/to/homepage.html?query=string

```

前面每个命令都会显示目标 URL 的全部 HTTP 内容；是的，这可能会让屏幕看起来有点乱。因此，你可以调用 tidy 选项，如下所示：

```
$ curl http://www.example.com | tidy -i

```

然而，如果你希望捕获数据并将输出保存到你选择的文件中，可以通过使用命令行重定向方法来实现，如下所示：

```
$ curl http://www.example.com > /path/to/folder/example.html

```

或者，你可以以以下方式使用`-o`选项：

```
$ curl -o /path/to/folder/example.txt http://www.example.com

```

请注意，通过调用`-o`选项的方法时，目标文件必须首先指定。然而，由于前面的示例表明我们将输出保存到文本文件中，你可以很高兴地将其更改为几乎任何类型的文件，如下所示：

```
$ curl -o /path/to/folder/example.html http://www.example.com

```

现在，假设网络连接良好，我们选择了 cURL，因为我们现在处理的是一个可能存在显示网页困难的 Web 服务器的特定问题。

正如我们之前所见，默认情况下，cURL 仅仅会输出所请求网页的内容。然而，通过使用一些额外的选项（或参数），你可以扩展它的功能，并请求更多的细节。

例如，如果我们使用`-w`选项（write-out），你可以通过以下语法获取任何网页的状态码：

```
$ curl -w "%{http_code}\n %{content_type}\n" http://www.example.com/path/to/page.html

```

在这里，`\n`用于将结果输出到新的一行（你还可以用`\t`输出制表符，或用`\r`输出回车符）。你现在应该知道该网页的 HTTP 状态码和 HTTP 内容类型。

例如，你可以尝试这样：

```
$ curl -w "%{http_code}\n %{content_type}\n" https://www.packtpub.com/virtualization-and-cloud/troubleshooting-centos

```

前述命令的结果稍显冗长，无法完整打印，但在输出的末尾，你应该能看到以下内容（其中目标数据按要求分行显示）：

```
 </body>
</html>
200
 text/html; charset=utf-8

```

此外，你甚至可以像这样包含远程 IP 地址：

```
$ curl -w "%{remote_ip}\n %{http_code}\n %{content_type}\n" http://www.example.com/path/to/page.html

```

该命令的输出应该在末尾显示类似以下的内容：

```
 </body>
</html>
23.205.169.129
 200
 text/html;charset=UTF-8

```

你可以使用以下命令获取网页的大小（以字节为单位）：

```
$ curl -w "%{size_download}\n" http://www.example.com/path/to/page.html

```

该命令的结果将在输出的末尾显示`63175`字节：

```
 </body>
</html>
63175

```

另一方面，如果你正在处理一个使用`301`和`302`重定向方法的 Web 服务器，我们可以像这样使用`-L`选项：

```
$ curl -Lw "%{remote_ip}\n %{http_code}\n %{content_type}\n" http://www.example.com/

```

最后，如果你想确保你对服务器网页的调查提供了 cURL 可能遇到的所有头信息的完整列表，你应该通过以下方式调用`-v`选项来增加输出详细程度：

```
$ curl -v http://www.example.com

```

例如，再次你可以像这样测试 Red Hat：

```
$ curl -v http://www.redhat.com

```

该命令的结果将提供以下输出：

```
* About to connect() to www.redhat.com port 80 (#0)
*   Trying 104.66.92.228...
* Connected to www.redhat.com (104.66.92.228) port 80 (#0)
> GET / HTTP/1.1
> User-Agent: curl/7.29.0
> Host: www.redhat.com
> Accept: */*
>
< HTTP/1.1 301 Moved Permanently
< Content-Type: text/html; charset=iso-8859-1
< Location: http://www.redhat.com/en
< Server: Apache
< Content-Length: 296
< Expires: Mon, 04 May 2015 14:53:10 GMT
< Cache-Control: max-age=0, no-cache
< Pragma: no-cache
< Date: Mon, 04 May 2015 14:53:10 GMT
< Connection: keep-alive
< Set-Cookie: AWSELB=014101F31CE28463C273156EDFEB4013EF4DC7B4B58B2D0587192FCB8DB58F8B0E7B8A652EC4DCB07BB3CC9D65387BA7D24617BF645CEBCF6476050FABBDF5D9227C0A5A30;PATH=/;MAX-AGE=30
<
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>301 Moved Permanently</title>
</head><body>
<h1>Moved Permanently</h1>
<p>The document has moved <a href="http://www.redhat.com/en">here</a>.</p>
<hr>
<address>Apache Server at www.redhat.com Port 80</address>
</body></html>
* Connection #0 to host www.redhat.com left intact

```

对于那些希望仅将输出限制为响应头的用户，应该像这样使用`-I`选项，而不是使用前面的命令：

```
$ curl -I http://www.example.com

```

例如，如果你像这样重新尝试 Red Hat：

```
$ curl -I http://www.redhat.com

```

该命令的结果将提供以下输出：

```
HTTP/1.1 301 Moved Permanently
Content-Type: text/html; charset=iso-8859-1
Location: http://www.redhat.com/en
Server: Apache
Content-Length: 0
Expires: Mon, 04 May 2015 14:55:54 GMT
Cache-Control: max-age=0, no-cache
Pragma: no-cache
Date: Mon, 04 May 2015 14:55:54 GMT
Connection: keep-alive
Set-Cookie: AWSELB=014101F31CE28463C273156EDFEB4013EF4DC7B4B53E1B0B83C0D272B9D220605DDE604A12C4DCB07BB3CC9D65387BA7D24617BF642030376EDB73D2D8C226E62350AE4B75;PATH=/;MAX-AGE=30

```

在这一阶段，关于 cURL 的内容还有很多可以讨论。事实上，你可以写一本关于这个主题的书；然而，在我们偏离主要话题之前，你会很高兴知道你可以通过阅读手册来了解更多关于 cURL 的内容：

```
$ man curl

```

# 使用 cURL 调试 Akamai 头信息

CDN（内容分发网络）正变得越来越普遍，其中最流行的是 Akamai。然而，虽然 CDN 能够带来好处，但在你排查 Web 服务、应用程序，甚至一个简单的主页时，它们也可能成为一个绊脚石。换句话说，使用任何类型的 CDN 时，你通常在与缓存的对象进行交互，而你需要验证流量行为。因此，考虑到这一点，我们现在将讨论如何通过`cURL`来解决这一问题：

首先，我们必须发出一个正确格式的 Pragma 头，为此，你可以使用以下语法：

```
$ curl -IXGET http://www.example.com/path/to/home.html

```

然而，如果你希望包含调试信息，可以使用：

```
$ curl -IXGET -H "Pragma: akamai-x-cache-on, akamai-x-cache-remote-on, akamai-x-check-cacheable, akamai-x-get-cache-key, akamai-x-get-extracted-values, akamai-x-get-nonces, akamai-x-get-ssl-client-session-id, akamai-x-get-true-cache-key, akamai-x-serial-no" http://www.example.com/path/to/home.html

```

例如，由于 Red Hat 是 Akamai 的已知用户，如果你尝试：

```
$ curl -IXGET -H "Pragma: akamai-x-cache-on, akamai-x-cache-remote-on, akamai-x-check-cacheable, akamai-x-get-cache-key, akamai-x-get-extracted-values, akamai-x-get-nonces, akamai-x-get-ssl-client-session-id, akamai-x-get-true-cache-key, akamai-x-serial-no" http://www.redhat.com/en

```

然后，除非自本书出版以来发生了重大变化（而网页总是会随着时间改变），否则输出结果将类似于这样：

```
HTTP/1.1 200 OK
Content-Language: en
Content-Type: text/html; charset=utf-8
ETag: "1430747458-1"
Last-Modified: Mon, 04 May 2015 13:50:58 GMT
Link: <http://www.redhat.com/en>; rel="canonical"
Server: Apache
X-Drupal-Cache: HIT
X-Powered-By: PHP/5.3.3
X-RedHat-Debug: 1
X-Check-Cacheable: NO
Expires: Mon, 04 May 2015 13:51:17 GMT
Cache-Control: max-age=0, no-cache
Pragma: no-cache
Date: Mon, 04 May 2015 13:51:17 GMT
Transfer-Encoding:  chunked
X-Cache: TCP_MISS from a2-20-133-122.deploy.akamaitechnologies.com (AkamaiGHost/7.2.0-15182023) (-)
X-Cache-Key: /L/1890/356403/3d/www.rollover.redhat.com.akadns.net/en cid=__
X-True-Cache-Key: /L/www.rollover.redhat.com.akadns.net/en cid=__
X-Akamai-Session-Info: name=CRS_VERSION; value=2.2.6
X-Akamai-Session-Info: name=DC_FORWARD_IP; value=54.187.212.127; full_location_id=X-DC-Origin-IP
X-Akamai-Session-Info: name=HEADER_NAMES; value=User-Agent%3aHost%3aAccept%3aPragma; full_location_id=
X-Akamai-Session-Info: name=INITORIGINIP; value=54.187.212.127
X-Akamai-Session-Info: name=NL_2580_BLACKLIST_NAME; value=Black List
X-Akamai-Session-Info: name=NL_6042_ORACLE_NAME; value=Oracle bot-block (per Keith Watkins by Sri Sankaran)
X-Akamai-Session-Info: name=NSCPCODE; value=298900
X-Akamai-Session-Info: name=OVERRIDE_HTTPS_IE_CACHE_BUST; value=all
X-Akamai-Session-Info: name=PARENT_SETTING; value=TD
X-Akamai-Session-Info: name=SITESHIELDMAP; value=s187.akamaiedge.net
X-Akamai-Session-Info: name=SQLI_SELECT_STATEMENT_COUNT; value=0
X-Akamai-Session-Info: name=SRTOPATH; value=/s/global.css
X-Akamai-Session-Info: name=SS4PMAP; value=www.redhat.com
X-Akamai-Session-Info: name=WAF_CREATE_ASSERTION_EXPIRE_TIME; value=1430747537
X-Akamai-Session-Info: name=WAF_CRS_ALLOWED_HTTP_VERSIONS; value=HTTP/0.9 HTTP/1.0 HTTP/1.1
X-Akamai-Session-Info: name=WAF_CRS_ALLOWED_METHODS; value=GET HEAD POST OPTIONS
X-Akamai-Session-Info: name=WAF_CRS_ARG_LENGTH; value=64000
X-Akamai-Session-Info: name=WAF_CRS_ARG_NAME_LENGTH; value=256
X-Akamai-Session-Info: name=WAF_CRS_CMD_INJECTION_ANOMALY_RULE_TUPLE; value=
X-Akamai-Session-Info: name=WAF_CRS_CMD_INJECTION_ANOMALY_SCORE; value=0
X-Akamai-Session-Info: name=WAF_CRS_CMD_INJECTION_ANOMALY_SCR; value=
X-Akamai-Session-Info: name=WAF_CRS_CRITICAL_ANOMALY_SCORE; value=5
X-Akamai-Session-Info: name=WAF_CRS_DEFAULT_ACTION; value=alert
X-Akamai-Session-Info: name=WAF_CRS_ERROR_ANOMALY_SCORE; value=4
X-Akamai-Session-Info: name=WAF_CRS_INBOUND_ANOMALY_RULE_SCR; value=
X-Akamai-Session-Info: name=WAF_CRS_INBOUND_ANOMALY_RULE_TUPLE; value=
X-Akamai-Session-Info: name=WAF_CRS_INBOUND_ANOMALY_SCORE; value=0
X-Akamai-Session-Info: name=WAF_CRS_INBOUND_MSG; value=
X-Akamai-Session-Info: name=WAF_CRS_INFO_ANOMALY_SCORE; value=1
X-Akamai-Session-Info: name=WAF_CRS_INVALID_HTTP_RULE_TUPLE; value=
X-Akamai-Session-Info: name=WAF_CRS_INVALID_HTTP_SCORE; value=0
X-Akamai-Session-Info: name=WAF_CRS_INVALID_HTTP_SCR; value=
X-Akamai-Session-Info: name=WAF_CRS_MAX_NUM_ARGS; value=255
X-Akamai-Session-Info: name=WAF_CRS_NOTICE_ANOMALY_SCORE; value=2
X-Akamai-Session-Info: name=WAF_CRS_OUTBOUND_ANOMALY_RULE_SCR; value=
X-Akamai-Session-Info: name=WAF_CRS_OUTBOUND_ANOMALY_RULE_TUPLE; value=
X-Akamai-Session-Info: name=WAF_CRS_OUTBOUND_ANOMALY_SCORE; value=0
X-Akamai-Session-Info: name=WAF_CRS_PHP_INJECTION_RULE_SCR; value=
X-Akamai-Session-Info: name=WAF_CRS_PHP_INJECTION_RULE_TUPLE; value=
X-Akamai-Session-Info: name=WAF_CRS_PHP_INJECTION_SCORE; value=0
X-Akamai-Session-Info: name=WAF_CRS_RESTRICTED_EXTENSIONS; value=asa asax ascx backup bak bat cdx cer cfg cmd com config conf cs csproj csr dat db dbf dll dos htr htw ida idc idq inc ini key licx lnk log mdb old pass pdb pol printer pwd resources resx sql sys vb vbs vbproj vsdisco webinfo xsd xsx
X-Akamai-Session-Info: name=WAF_CRS_RESTRICTED_HEADERS; value=Proxy-Connection Lock-Token Content-Range Translate Via If
X-Akamai-Session-Info: name=WAF_CRS_RFI_ANOMALY_RULE_SCR; value=
X-Akamai-Session-Info: name=WAF_CRS_RFI_ANOMALY_RULE_TUPLE; value=
X-Akamai-Session-Info: name=WAF_CRS_RFI_ANOMALY_SCORE; value=0
X-Akamai-Session-Info: name=WAF_CRS_RISK_GROUPS; value=
X-Akamai-Session-Info: name=WAF_CRS_RISK_SCRS; value=
X-Akamai-Session-Info: name=WAF_CRS_RISK_TUPLES; value=
X-Akamai-Session-Info: name=WAF_CRS_SQL_INJECTION_RULE_TUPLE; value=
X-Akamai-Session-Info: name=WAF_CRS_SQL_INJECTION_SCORE; value=
X-Akamai-Session-Info: name=WAF_CRS_SQL_INJECTION_SCR; value=
X-Akamai-Session-Info: name=WAF_CRS_TOTAL_ANOMALY_SCORE; value=0
X-Akamai-Session-Info: name=WAF_CRS_TOTAL_ARG_LENGTH; value=64000
X-Akamai-Session-Info: name=WAF_CRS_TROJAN_RULE_SCR; value=
X-Akamai-Session-Info: name=WAF_CRS_TROJAN_RULE_TUPLE; value=
X-Akamai-Session-Info: name=WAF_CRS_TROJAN_SCORE; value=0
X-Akamai-Session-Info: name=WAF_CRS_WARNING_ANOMALY_SCORE; value=3
X-Akamai-Session-Info: name=WAF_CRS_XSS_RULE_SCR; value=
X-Akamai-Session-Info: name=WAF_CRS_XSS_RULE_TUPLE; value=
X-Akamai-Session-Info: name=WAF_CRS_XSS_SCORE; value=0
X-Akamai-Session-Info: name=WAF_DATA_HEADER_SIGN_VAL; value=HnWuQRcXIUfMGG3LF/9PllRcUxUkocv8aFiFmuExQZE=
X-Akamai-Session-Info: name=WAF_DATA_HEADER_VAL; value=/en 1430747537
X-Akamai-Session-Info: name=WAF_HA_STATUS; value=checking
X-Akamai-Session-Info: name=WAF_MYSQLI_COUNT; value=0
X-Serial: 1890
Connection: keep-alive
Connection: Transfer-Encoding
Set-Cookie: WL_DCID=origin-www-c; expires=Mon, 04-May-2015 21:51:17 GMT; path=/
Set-Cookie: AWSELB=7DE7FB19045D425DE69229FBB7F229663FD24433135E354B67BC8404E265E1F485365E31B3F24C3F30EB76C3348446159423E486323BDC9105B0C92244E19C46091861E2C5;PATH=/;MAX-AGE=30
Set-Cookie: AWSELB=014101F31CE28463C273156EDFEB4013EF4DC7B4B52244855012161AA58C6EBAB965CAFA77C4DCB07BB3CC9D65387BA7D24617BF645CEBCF6476050FABBDF5D9227C0A5A30;PATH=/;MAX-AGE=30
X-Cache-Remote: TCP_MISS from a195-10-11-245.deploy.akamaitechnologies.com (AkamaiGHost/7.2.0-15182023) (-)

```

我同意这个例子相当长，但 Akamai 头部信息确实非常广泛。当然，这个过程不仅限于网页本身，也可以用来定位你选择的任何对象。

例如，如果你想定位特定的图像，可以使用：

```
$ curl -IXGET -H "Pragma: akamai-x-cache-on, akamai-x-cache-remote-on, akamai-x-check-cacheable, akamai-x-get-cache-key, akamai-x-get-extracted-values, akamai-x-get-nonces, akamai-x-get-ssl-client-session-id, akamai-x-get-true-cache-key, akamai-x-serial-no" http://www.example.com/path/to/image.jpg

```

或者，你可以像这样定位一个 CSS 文件：

```
$ curl -IXGET -H "Pragma: akamai-x-cache-on, akamai-x-cache-remote-on, akamai-x-check-cacheable, akamai-x-get-cache-key, akamai-x-get-extracted-values, akamai-x-get-nonces, akamai-x-get-ssl-client-session-id, akamai-x-get-true-cache-key, akamai-x-serial-no" http://www.example.com/path/to/style.css

```

在这一点上，我假设你已经了解了大多数 HTTP 响应代码（如果不了解，可以在本章末尾找到参考链接），但在我们结束这一主题之前，让我们简要看一下你可能遇到的一些不太显眼的头部：

+   `X-Check-Cacheable`：该值将告诉我们目标对象是否可以被 Akamai 缓存。

+   `X-Cache-Key`：忽略前两个值，从结果字符串的第三个值开始，你将看到 CP 代码和相关的 TTL，尽管当通过 Edge-Control 头部在应用程序级别设置时，TTL 可能会有所不同。

+   `X-Cache`：该值将告诉我们 Akamai Edge 服务器返回的输出内容。然而，该值还会指示以下几种情况：

    +   `TCP_HIT`：该值表示对象在缓存中是新鲜的，并且该对象是从磁盘缓存中获取的。

    +   `TCP_MISS`：该值表示对象不在缓存中；服务器从源获取了该对象。

    +   `TCP_REFRESH_HIT`：该值表示对象在缓存中是过期的，并且我们成功地在 If-Modified-Since 请求下刷新了源对象。

    +   `TCP_REFRESH_MISS`：该值表示对象在缓存中是过期的，并且刷新操作从源获取了新对象，作为对我们的 If-Modified-Since 请求的响应。

    +   `TCP_REFRESH_FAIL_HIT`：该值表示对象在缓存中是过期的，并且我们刷新失败（无法访问源对象），因此我们返回了过期的对象。

    +   `TCP_IMS_HIT`：该值表示来自客户端的 If-Modified-Since 请求和对象在缓存中是新鲜的，并且已被提供。

    +   `TCP_NEGATIVE_HIT`：该值表示该对象之前返回了一个“未找到”消息（或任何其他不可缓存的负面响应），并且缓存的响应对这个新请求是有效的。

    +   `TCP_MEM_HIT`：该值表示对象位于磁盘并且已经在内存缓存中。服务器直接从内存中提供，而没有访问磁盘。

    +   `TCP_DENIED`：该值表示由于某种原因，你被拒绝了访问客户端的权限。

    +   `TCP_COOKIE_DENY`：该值表示你在进行 Cookie 认证时被拒绝访问（如果在配置中使用了集中式或分散式授权功能）。

所以，正如你所看到的，使用 cURL 调试 Akamai 头部信息非常简单。是的，也有浏览器插件可以完成相同的工作，但知道如何用 cURL 操作要有趣得多。

# 添加 Varnish 到 Apache

Varnish 是一个高性能的 HTTP 加速器，不仅有助于减少服务器的总体负载，还有助于提高网站响应时间。因此，它变得非常受欢迎；因此，我们将研究在 Apache Web 服务器与 Varnish 同时设置的过程。

在开始之前，我们假设已安装了 Apache。此外，您需要知道完成接下来的步骤需要访问 EPEL 仓库。请参考第四章，《故障排除包管理和系统升级》中有关如何在 CentOS 7 上下载和安装 EPEL 仓库的说明。

因此，当您准备好时，让我们开始安装 Varnish：

```
# yum install varnish

```

安装 Varnish 成功后，我们需要在启动时启用服务。可以通过输入以下命令来实现：

```
# systemctl enable varnish

```

然后，我们需要激活服务，如下所示：

```
# systemctl start varnish

```

因此，在完成基本安装后，您现在可以通过输入以下命令来检查 Varnish 的状态：

```
# systemctl status varnish

```

然后，通过输入以下命令来检查您正在运行的版本：

```
# varnishd -V

```

在这个阶段，我们需要完成此服务的基本配置，并使其能够与 Apache 配合工作。为此，我们将从打开主 Apache 配置文件开始，使用您喜爱的文本编辑器，如下所示：

```
# nano /etc/httpd/conf/httpd.conf

```

现在，向下滚动以找到以下行：

```
Listen 80
```

用以下行替换它：

```
Listen 127.0.0.1:8080
```

如果 Web 服务器正在运行一个或多个虚拟主机，则需要进行以下调整以反映 Apache 监听的新端口：

```
<VirtualHost *:8080>
```

现在保存文件并运行以下命令来检查语法：

```
# httpd -t

```

输出应为：

```
Syntax OK

```

现在，完成了这些步骤后，我们将对原始 Varnish 安装进行第一个配置更改：

```
# nano /etc/varnish/varnish.params

```

向下滚动并查找以下行：

```
VARNISH_LISTEN_PORT=6081
```

用以下内容替换它：

```
VARNISH_LISTEN_PORT=80
```

现在向下滚动并找到以下行：

```
VARNISH_STORAGE=
```

这就是 Varnish 变得有趣的地方，作为故障排除者，您可以确定最适合优化 Web 性能的方法。目前，您会注意到 Varnish 配置为使用服务器的硬盘来缓存文件，在这种情况下，您有两个选项。

在预期需要大量缓存、RAM 有限或打算构建专用 Varnish 存储以缓存文件的情况下，通过对默认设置进行简单调整来指定缓存大小。

例如，如果您希望创建 20 GB 的磁盘缓存，可以使用以下行：

```
VARNISH_STORAGE="file,/var/lib/varnish/varnish_storage.bin,20G"
```

但是，如果您希望体验使用 RAM 缓存方法的终极 Varnish 体验，则可以通过自定义以下行来实现，以反映系统的需求：

```
VARNISH_STORAGE="malloc,1G"
```

现在让我们进一步探讨这个问题。

例如，如果您希望 RAM 缓存高达 4 GB 的数据，可以使用以下方法：

```
VARNISH_STORAGE="malloc,4G"
```

或者，如果您想改善性能不太强大的基于 RAM 的环境，可以将此值更改为 512 MB，如下所示：

```
VARNISH_STORAGE="malloc,512m"
```

现在，你可以保存并关闭这个文件，然后打开以下文件：

```
# nano /etc/varnish/default.vcl

```

这个文件是 Varnish 的整体配置文件。我现在不会详细讲解具体内容，因为有很多基于 Varnish 的书籍已经详细覆盖了这个话题。然而，为了故障排除，你需要进行一些基本的修改，以确保系统正常运行。为此，只需确保以下部分符合相关系统的要求：

```
vcl 4.0;
# Default backend definition. Set this to point to your content server.
backend default {
    .host = "127.0.0.1";
    .port = "8080";
}
```

完成此操作后，你现在应该按如下方式重启 Apache：

```
# systemctl restart httpd.service

```

然后按如下方式重启 Varnish：

```
# systemctl restart varnish.service

```

干得好！Varnish 的安装已经完成；你可以像往常一样继续访问基于 Apache 的网站，但现在有了额外的优势，能够体验到更快的速度和性能。

# 使用 cURL 测试 Varnish

如果 Web 服务器依赖于 Varnish，那么确保网页被缓存并及时提供尤其重要。

要确认这一点，你可以使用以下语法开始：

```
# curl -I http://www.example.com/index.html

```

使用 `-I` 选项仅显示头部信息后，如果 Varnish 已安装，你应该看到类似这样的内容：

```
HTTP/1.1 200 OK
Date: Fri, 06 Mar 2015 00:59:24 GMT
Server: Apache/2.4.6 (CentOS) PHP/5.5.22
X-Powered-By: PHP/5.5.22
Content-Type: text/html; charset=UTF-8
X-Varnish: 5 3
Age: 16
Via: 1.1 varnish-v4
Content-Length: 97422
Connection: keep-alive

```

在前面的示例中，最重要的行是以下几行：

```
X-Varnish: 5 3
Age: 16

```

现在，让我们快速浏览一下这些值的解释：

+   `X-Varnish: XXXXXXXX XXXXXXXX`：`X-Varnish` 头不仅包含当前请求的 ID，还维护了填充缓存的请求 ID。如果只有一个数字，你应该知道缓存是由当前请求填充的，可以认为是所谓的缓存未命中。

+   `Age: XXXX`：这个值表示内容在缓存中存储的时间。如果显示为零（0），则意味着该页面根本没有被缓存。

当然，显示的确切值可能不同，但通过这个例子，你现在不仅能够确认并验证服务器上 Varnish 的功能，还可以时刻关注给定的 `Age` 值（你将知道页面在缓存中存在的时间（秒））。

# 使用 cURL 访问 FTP 目录

通过实践，每个人都可以使用 FTP 客户端，但当你需要脚本化某些事件时，就需要调用 cURL 来完成所有的繁重工作。

因此，从最基础的层面开始，访问带有现有用户名和密码的 FTP 目录的最简单方法如下：

```
$ curl ftp://exampleftpsite.com -u <username>

```

当系统提示时，只需在提示符下输入你的密码：

```
$ curl ftp://exampleftpsite.com  -u <username>
Enter host password for user '<username>':

```

现在，如果你想在 FTP 目录中搜索特定的文件列表，可以像这样使用 `-s` 静默选项和 `grep` 组合：

```
$ curl ftp://exampleftpsite.com -u <username> -s | grep <keyword>

```

你可以使用以下命令完成搜索并上传文件：

```
$ curl -T filename.zip ftp://exampleftpsite.com -u <username>

```

或者你可以使用以下语法直接下载：

```
$ curl ftp://exampleftpsite.com -u <username> -o filename.zip

```

同样，当提示时，你需要输入正确的密码，找到相关文件后，可以使用以下语法来发现你当前的位置：

```
$ pwd

```

最后，为了结束这一部分，你可以通过以下方式删除一个文件：

```
$ curl ftp://exampleftpsite.com -X 'DELE filename.zip' -u <username>

```

记住，在删除文件时需要格外小心，因为系统不会给出任何提示。使用 cURL 删除文件的操作是自动进行的。

# 启用 Apache 中的 mod_status

`mod_status` 是一个 Apache 模块，帮助监控 Web 服务器负载和当前的 `httpd` 连接。它配有一个 HTML 界面，并且可以通过任何浏览器访问。

要使用 `mod_status`，我们需要对 `VirtualHosts` 文件进行一些基本的配置更改，因此让我们从头开始，使用以下命令创建一个简单的虚拟主机：

```
# nano  /etc/httpd/conf.d/vhost.conf

```

添加以下几行：

```
<VirtualHost *:80>
   DocumentRoot /var/www/html
   ServerName servername.domain.com
</VirtualHost>
```

记住：如果你使用的是 Varnish，请确保它使用正确的端口：

```
<VirtualHost *:8080>
   DocumentRoot /var/www/html
   ServerName servername.domain.com
</VirtualHost>
```

完成此操作后，你可以在适当的 `<VirtualHost></VirtualHost>` 指令之间添加以下几行，以启用 `mod_status`：

```
<Location /server-status>
  SetHandler server-status
  Order allow,deny
  Allow from all
</Location>
```

最终的结果应该是这样的：

```
<VirtualHost *:8080>
   DocumentRoot /var/www/html
   ServerName servername.domain.com

  <Location /server-status>
    SetHandler server-status
    Order allow,deny
    Allow from all
  </Location>
</VirtualHost>
```

你可以看到那行写着 `Allow from all`。对于前面的示例来说，这没问题，但出于安全考虑，你应该将连接访问限制为特定的 IP 地址。

因此，更好的选择是使用以下语法：

```
<Location /server-status>
  SetHandler server-status
  Order deny, allow
  Deny from all
  Allow from localhost
</Location>
```

当然，你应该根据系统的需求定制前面的示例代码；完成后，保存并关闭文件，然后重新启动 Apache：

```
# systemctl restart httpd

```

现在，打开浏览器，使用以下格式访问选定的虚拟主机：

`http://XXX.XXX.XXX.XXX/server-status`

这将使你能够完全访问静态的 `server-status` 页面。然而，如果你在 URL 后面加上 `refresh` 选项，如下所示，页面应该每 5 秒刷新一次：

`http://XXX.XXX.XXX.XXX/server-status/?refresh=5`

生成的页面将显示一系列信息，包括服务器运行时间、服务器负载、连接总数、CPU 使用率、进程 ID、请求总数以及当前正在处理的请求。所有这些都在你尝试诊断与特定网站相关的问题时非常有用。

# 总结

本章的目的是以一种非常不同的方式来看待 Web 服务故障排除。从使用 cURL 审核服务器到使用 Varnish 做性能优化，我们不仅考虑了系统管理员的需求，还探索了 CDN 和 Dev/Ops 的世界，目的是展示故障排除人员可以有多么灵活，以及你的技能将变得多么重要。

在下一章，我们将讨论一些故障排除 DNS 基于服务时使用的技巧。

# 参考文献

+   Curl 的 Wikipedia 主页：[`en.wikipedia.org/wiki/CURL`](http://en.wikipedia.org/wiki/CURL)

+   Curl 主页：[`curl.haxx.se`](http://curl.haxx.se)

+   HTTP 状态码的 Wikipedia 主页：[`en.wikipedia.org/wiki/List_of_HTTP_status_codes`](http://en.wikipedia.org/wiki/List_of_HTTP_status_codes)

+   HTTPie，curl 的替代工具：[`github.com/jakubroztocil/httpie`](https://github.com/jakubroztocil/httpie)

+   Varnish 官网：[`www.varnish-cache.org`](https://www.varnish-cache.org)

+   Varnish 管理员文档：[`www.varnish-cache.org/docs/trunk/index.html`](https://www.varnish-cache.org/docs/trunk/index.html)
