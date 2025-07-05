# 第三章：代理与缓存

Nginx 作为 Web 加速器和前端服务器设计，拥有强大的工具，可以将复杂的任务委托给上游服务器，同时专注于繁重的工作。反向代理就是其中一个工具，它使 Nginx 成为任何高性能 Web 服务的重要组成部分。

通过抽象化 HTTP 的复杂性并以可扩展和高效的方式处理，Nginx 使 Web 应用能够专注于解决它们被设计来解决的问题，而不会陷入底层细节。

在本章中，您将学习：

+   如何将 Nginx 设置为反向代理

+   如何让代理对上游服务器和最终用户透明

+   如何处理上游错误

+   如何使用 Nginx 缓存

你将了解到如何使用 Nginx 反向代理的所有功能，并将其转变为一个强大的加速和扩展 Web 服务的工具。

# Nginx 作为反向代理

HTTP 是一个复杂的协议，处理不同模态的数据，并具有许多优化，如果实施得当，可以显著提高 Web 服务的性能。

同时，Web 应用开发人员可以减少处理底层问题和优化的时间。将 Web 应用服务器与前端服务器解耦的这一概念，将重点从管理前端的传入流量转移到 Web 应用服务器上的功能、应用逻辑和特性。这正是 Nginx 作为解耦点发挥作用的地方。

解耦的一个例子是 SSL 终止：Nginx 接收并处理传入的 SSL 连接，将请求通过普通的 HTTP 转发到应用服务器，并将接收到的响应重新封装为 SSL。应用服务器不再需要处理证书存储、SSL 会话、加密和未加密传输等问题。

其他解耦的例子如下：

+   高效处理静态文件并将动态部分委托给上游

+   限制速率、请求和连接

+   压缩来自上游的响应

+   缓存来自上游的响应

+   加速上传和下载

通过将这些功能转移到由 Nginx 提供支持的前端，您实际上是在投资于您网站的可靠性。

# 设置 Nginx 作为反向代理

Nginx 可以轻松配置为反向代理：

```
location /example {
    proxy_pass http://upstream_server_name;
}
```

在前面的代码中，`upstream_server_name` 是上游服务器的主机名。当收到某个位置的请求时，它会被传递到具有指定主机名的上游服务器。

如果上游服务器没有主机名，可以使用 IP 地址代替：

```
location /example {
    proxy_pass http://192.168.0.1;
}
```

如果上游服务器监听非标准端口，可以将端口添加到目标 URL 中：

```
location /example {
    proxy_pass http://192.168.0.1:8080;
}
```

前面示例中的目标 URL 没有路径。这使得 Nginx 按原样转发请求，而不重新写入原始请求中的路径。

如果目标 URL 中指定了路径，它将替换掉原始请求中与位置匹配部分对应的路径。例如，考虑以下配置：

```
location /download {
    proxy_pass http://192.168.0.1/media;
}
```

如果收到对 `/download/BigFile.zip` 的请求，目标 URL 中的路径是 `/media`，它对应原始请求 URI 中的 `/download` 部分。这个部分会在传递到上游服务器之前被替换成 `/media`，因此传递的请求路径看起来像 `/media/BigFile.zip`。

如果 `proxy_pass` 指令被用在正则表达式位置内，匹配的部分将无法计算。在这种情况下，必须使用没有路径的目标 URI：

```
location ~* (script1|script2|script3)\.php$ {
    proxy_pass http://192.168.0.1;
}
```

同样的原则适用于请求路径通过 rewrite 指令改变并被 `proxy_pass` 指令使用的情况。

变量也可以是目标 URL 的一部分：

```
location ~* ^/(index|content|sitemap)\.html$ {
    proxy_pass http://192.168.0.1/html/$1;
}
```

实际上，任何部分，甚至整个目标 URL 都可以通过变量来指定：

```
location /example {
    proxy_pass $destination;
}
```

这为指定上游服务器的目标 URL 提供了足够的灵活性。在第五章，*管理入站和出站流量*中，我们将了解如何指定多个服务器作为上游并在它们之间分配连接。

## 正确设置后端

正确配置后端的方法是避免将所有内容都传递给它。Nginx 提供了强大的配置指令，帮助确保只有特定的请求被委托给后端。

请考虑以下配置：

```
location ~* \.php$ {
    proxy_pass http://backend;
    [...]
}
```

这会将每个以 `.php` 结尾的 URI 请求传递给 PHP 解释器。这不仅由于正则表达式的广泛使用而效率低下，而且在大多数 PHP 设置中还是一个严重的安全问题，因为它可能允许攻击者执行任意代码。

Nginx 对此问题提供了一个优雅的解决方案，即 `try_files` 指令。`try_files` 指令接受一个文件列表和一个作为最后参数的路径位置。Nginx 按顺序尝试指定的文件，如果都不存在，则进行内部重定向到指定的位置。请看下面的示例：

```
location / {
    try_files $uri $uri/ @proxy;
}

location @proxy {
    proxy_pass http://backend;
}
```

上述配置首先查找与请求 URI 对应的文件，接着查找与请求 URI 对应的目录，希望返回该目录的索引；如果这些文件或目录都不存在，最后会进行内部重定向到指定的名为 `@proxy` 的位置。

这个配置确保了每当请求 URI 指向文件系统中的对象时，它将由 Nginx 自行处理，使用高效的文件操作；只有当文件系统中没有匹配的请求 URI 时，才会委托给后端。

## 增加透明度

一旦请求被转发到上游服务器，原始请求的某些属性会丢失。例如，转发请求中的虚拟主机将被目标 URL 的主机/端口组合替代。转发请求是从 Nginx 代理的 IP 地址接收的，而上游服务器基于客户端的 IP 地址的功能可能无法正常工作。

转发请求需要进行调整，以便上游服务器能够获取原始请求缺失的信息。这可以通过 `proxy_set_header` 指令轻松实现：

```
proxy_set_header <header> <value>;
```

`proxy_set_header` 指令接受两个参数，第一个是你希望在代理请求中设置的头部名称，第二个是该头部的值。同样，这两个参数都可以包含变量。

下面是如何从原始请求中传递虚拟主机名的方法：

```
location @proxy {
    proxy_pass http://192.168.0.1;
    proxy_set_header Host $host;
}
```

变量 `$host` 具有智能功能。它不仅仅传递原始请求中的虚拟主机名，还会在原始请求的主机头为空或缺失时，使用处理请求的服务器的名称。如果你坚持要使用原始请求中的虚拟主机名，可以使用 `$http_host` 变量，而不是 `$host`。

现在你知道如何操作代理请求，我们可以让上游服务器了解原始客户端的 IP 地址。这可以通过设置 `X-Real-IP` 和/或 `X-Forwarded-For` 头部来实现：

```
location @proxy {
    proxy_pass http://192.168.0.1;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
```

这样，上游服务器就能通过 `X-Real-IP` 或 `X-Forwarded-For` 头部了解到原始客户端的 IP 地址。大多数应用服务器支持该头部，并采取适当的措施以正确反映原始 IP 地址。

## 处理重定向

下一个挑战是重写重定向。当上游服务器发出临时或永久重定向（HTTP 状态码 `301` 或 `302`）时，位置或刷新头部中的绝对 URI 需要被重写，以便它包含正确的主机名（即原始请求所到达的服务器的主机名）。

这可以通过使用 `proxy_redirect` 指令来实现：

```
location @proxy {
    proxy_pass http://localhost:8080;
    proxy_redirect http://localhost:8080/app http://www.example.com;
}
```

假设有一个运行在 `http://localhost:8080/app` 的 Web 应用程序，而原始服务器的地址是 `http://www.example.com`。假设 Web 应用程序发出了一个临时重定向（HTTP 302）到 `http://localhost:8080/app/login`。使用前述配置，Nginx 将会将位置头部中的 URI 重写为 `http://www.example.com/login`。

如果重定向 URI 没有被重写，客户端将被重定向到 `http://localhost:8080/app/login`，这个地址只在本地域名下有效，因此 Web 应用程序无法正常工作。使用 `proxy_redirect` 指令后，重定向 URI 将被 Nginx 正确重写，Web 应用程序将能够正确地进行重定向。

`proxy_redirect` 指令的第二个参数中的主机名可以省略：

```
location @proxy {
    proxy_pass http://localhost:8080;
    proxy_redirect http://localhost:8080/app /;
}
```

使用变量，前面的代码可以进一步简化为以下配置：

```
location @proxy {
    proxy_pass http://localhost:8080;
    proxy_redirect http://$proxy_host/app /;
}
```

同样的透明度选项也可以应用于 cookies。在前面的示例中，假设 cookies 被设置为 `localhost:8080` 域名，因为应用服务器在 `http://localhost:8080` 上响应。由于 cookie 域名与请求域名不匹配，浏览器将不会返回这些 cookies。

## 处理 cookies

为了确保 cookies 正常工作，cookie 中的域名需要通过 Nginx 代理进行重写。为此，您可以使用如下所示的 `proxy_cookie_domain` 指令：

```
location @proxy {
    proxy_pass http://localhost:8080;
    proxy_cookie_domain localhost:8080 www.example.com;
}
```

在前面的示例中，Nginx 将上游响应中的 cookie 域名 `localhost:8080` 替换为 `www.example.com`。上游服务器设置的 cookies 将指向 `www.example.com` 域名，浏览器将在后续请求中返回这些 cookies。

如果由于应用服务器位于不同路径，需要重写 cookie 路径，可以使用 `proxy_cookie_path` 指令，如以下代码所示：

```
location @proxy {
    proxy_pass http://localhost:8080;
    proxy_cookie_path /my_webapp/ /;
}
```

在这个示例中，每当 Nginx 检测到一个具有在 `proxy_cookie_path` 指令第一个参数中指定的前缀 (`/my_webapp/`) 的 cookie 时，它会将这个前缀替换为 `proxy_cookie_path` 指令第二个参数中的值 (`/`)。

将所有内容整合在一起，对于 `www.example.com` 域名和运行在 `localhost:8080` 的 Web 应用，我们可以得到以下配置：

```
location @proxy {
    proxy_pass http://localhost:8080;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_redirect http://$proxy_host/app /;
    proxy_cookie_domain $proxy_host www.example.com;
    proxy_cookie_path /my_webapp/ /;
}
```

上述配置确保了 Web 应用服务器的透明性，使其无需知道自己运行在哪个虚拟主机上。

## 使用 SSL

如果上游服务器支持 SSL，只需将目标 URL 的协议更改为 `https`，即可安全连接到上游服务器：

```
location @proxy {
    proxy_pass https://192.168.0.1;
}
```

如果需要验证上游服务器的真实性，可以通过 `proxy_ssl_verify` 指令启用此功能：

```
location @proxy {
    proxy_pass https://192.168.0.1;
    proxy_ssl_verify on;
}
```

上游服务器的证书将与知名认证机构的证书进行验证。在类 Unix 操作系统中，它们通常存储在 `/etc/ssl/certs` 中。

如果上游使用的证书无法通过知名认证机构验证，或是自签名证书，可以使用 `proxy_ssl_trusted_certificate` 指令指定并声明其为受信任的证书。此指令指定上游服务器证书的路径，或 PEM 格式认证上游服务器所需的证书链。参考以下示例：

```
location @proxy {
    proxy_pass https://192.168.0.1;
    proxy_ssl_verify on;
    proxy_ssl_trusted_certificate /etc/nginx/upstream.pem;
}
```

如果 Nginx 需要对上游服务器进行身份验证，可以使用 `proxy_ssl_certificate` 和 `proxy_ssl_certificate_key` 指令来指定客户端证书和密钥。`proxy_ssl_certificate` 指令指定 PEM 格式的客户端证书路径，而 `proxy_ssl_certificate_key` 指令指定 PEM 格式的客户端证书私钥路径。参考以下示例：

```
location @proxy {
    proxy_pass https://192.168.0.1;
    proxy_ssl_certificate /etc/nginx/client.pem;
    proxy_ssl_certificate_key /etc/nginx/client.key;
}
```

在建立与上游服务器的安全连接时，将使用指定的证书，并通过指定的私钥验证其真实性。

## 处理错误

如果 Nginx 在与上游服务器通信时遇到问题，或上游服务器返回错误，则可以选择采取某些操作。

上游服务器连接错误可以通过`error_page`指令进行处理：

```
location ~* (script1|script2|script3)\.php$ {
    proxy_pass http://192.168.0.1;
    error_page 500 502 503 504 /50x.html;
}
```

这将使 Nginx 在发生上游连接错误时，从文件`50x.html`返回文档。

这不会改变响应中的 HTTP 状态码。如果要将 HTTP 状态码更改为成功状态，可以使用以下语法：

```
location ~* (script1|script2|script3)\.php$ {
    proxy_pass http://192.168.0.1;
    error_page 500 502 503 504 =200 /50x.html;
}
```

在上游服务器出现故障时，可以使用`error_page`指令，指向一个命名位置，采取更复杂的操作：

```
location ~* (script1|script2|script3)\.php$ {
    proxy_pass http://upstreamA;
    error_page 500 502 503 504 @retry;
}

location @retry {
    proxy_pass http://upstreamB;
    error_page 500 502 503 504 =200 /50x.html;
}
```

在上述配置中，Nginx 首先尝试通过将请求转发到`upstreamA`服务器来处理该请求。如果发生错误，Nginx 将切换到名为`@retry`的位置，尝试与`upstreamB`服务器进行连接。请求 URI 在切换时保持不变，这样`upstreamB`服务器将接收到相同的请求。如果这仍然没有解决问题，Nginx 将返回一个静态文件`50x.html`，假装没有发生错误。

如果上游已回复但返回错误，可以使用`proxy_intercept_errors`指令进行拦截，而不是将其传递给客户端：

```
location ~* (script1|script2|script3)\.php$ {
    proxy_pass http://upstreamA;
    proxy_intercept_errors on;
    error_page 500 502 503 504 403 404 @retry;
}

location @retry {
    proxy_pass http://upstreamB;
    error_page 500 502 503 504 =200 /50x.html;
}
```

在上述配置中，即使`upstreamA`服务器回复但返回错误的 HTTP 状态码，如`403`或`404`，`upstreamB`服务器也会被调用。这给了`upstreamB`修复`upstreamA`的软错误的机会，如果需要的话。

然而，这种配置模式不应过度扩展。在第五章，*管理进出流量*中，我们将了解如何以更优雅的方式处理此类情况，而不需要复杂的配置结构。

## 选择外部 IP 地址

有时，当你的代理服务器有多个网络接口时，必须选择使用哪个 IP 地址作为上游连接的外部地址。默认情况下，系统会选择与默认路由中目标主机所在网络相邻的接口地址。

要选择特定的 IP 地址用于外部连接，可以使用`proxy_bind`指令：

```
location @proxy {
    proxy_pass https://192.168.0.1;
    proxy_bind 192.168.0.2;
}
```

这将使 Nginx 在建立连接之前，将外部套接字绑定到 IP 地址`192.168.0.2`。然后，上游服务器将看到来自 IP 地址`192.168.0.2`的连接。

## 加速下载

Nginx 在处理大型上传和下载等重负载操作时非常高效。这些操作可以通过内置功能和第三方模块委托给 Nginx 处理。

为了加速下载，上游服务器必须能够发出指向需要返回的资源位置的 `X-Accel-Redirect` 头，而不是返回来自上游的响应。考虑以下配置：

```
location ~* (script1|script2|script3)\.php$ {
    proxy_pass https://192.168.0.1;
}

location /internal-media/ {
    internal;
    alias /var/www/media/;
}
```

使用前述配置，一旦 Nginx 在上游响应中检测到 `X-Accel-Redirect` 头，它将执行指向该头中指定位置的内部重定向。假设上游服务器指示 Nginx 执行内部重定向到 `/internal-media/BigFile.zip`。这个路径将与 `/internal-media` 位置匹配。该位置指定了文档根目录为 `/var/www/media`。因此，如果文件 `/var/www/media/BigFile.zip` 存在，它将通过高效的文件操作返回给客户端。

对于许多 Web 应用服务器来说，这一功能大大提高了速度——既因为它们可能无法高效处理大文件下载，也因为代理会降低大文件下载的效率。

# 缓存

一旦将 Nginx 设置为反向代理，合理的做法是将其转变为缓存代理。幸运的是，使用 Nginx 可以非常容易地实现这一点。

## 配置缓存

在启用某个位置的缓存之前，您需要先配置缓存。缓存是一个包含缓存项文件的文件系统目录，并且有一个存储缓存项信息的共享内存段。

可以使用 `proxy_cache_path` 指令声明一个缓存：

```
proxy_cache_path <path> keys_zone=<name>:<size> [other parameters...];
```

上述命令声明了一个位于路径 `<path>` 的缓存，并有一个名为 `<name>` 的共享内存段，其大小为 `<size>`。

此指令必须在配置的`http`部分中指定。每个指令实例声明一个新的缓存，并且必须为共享内存段指定一个唯一的名称。考虑以下示例：

```
http {
    proxy_cache_path /var/www/cache keys_zone=my_cache:8m;
    [...]
}
```

上述配置声明了一个位于 `/var/www/cache` 的缓存，并有一个名为 `my_cache` 的共享内存段，其大小为 8MB。每个缓存项在内存中大约占用 128 字节，因此上述配置为大约 64,000 个项分配了空间。

以下表格列出了 `proxy_cache_path` 的其他参数及其含义：

| 参数 | 描述 |
| --- | --- |
| `levels` | 指定缓存目录的层级结构 |
| `inactive` | 指定缓存项在未被使用时将从缓存中移除的时间，无论其是否新鲜 |
| `max_size` | 指定所有缓存项的最大大小（总大小） |
| `loader_files` | 指定 **缓存加载器** 进程在每次迭代中加载的文件数 |
| `loader_sleep` | 指定缓存加载器进程在每次迭代之间休眠的时间间隔 |
| `loader_threshold` | 指定缓存加载器进程每次迭代的时间限制 |

一旦 Nginx 启动，它会处理所有配置的缓存并为每个缓存分配共享内存段。

之后，一个名为缓存加载器的特殊进程负责将缓存项加载到内存中。缓存加载器以迭代的方式加载项目。`loader_files`、`loader_sleep`和`loader_threshold`参数定义了缓存加载器进程的行为。

在运行时，一个名为**缓存管理器**的特殊进程监控所有缓存项所占用的总磁盘空间，并在总空间超过`max_size`参数指定的大小时，驱逐请求较少的缓存项。

## 启用缓存

要为某个位置启用缓存，您需要使用`proxy_cache`指令指定缓存：

```
location @proxy {
    proxy_pass http://192.168.0.1:8080;
    proxy_cache my_cache;
}
```

`proxy_cache`指令的参数是指向通过`proxy_cache_path`指令配置的缓存的共享内存段的名称。相同的缓存可以在多个位置使用。如果能够确定上游响应的过期时间间隔，则该响应将被缓存。Nginx 的过期时间间隔的主要来源是上游响应。以下表格解释了哪些上游响应头会影响缓存以及如何影响：

| 上游响应头 | 它如何影响缓存 |
| --- | --- |
| `X-Accel-Expires` | 此项指定缓存项的过期时间间隔（以秒为单位）。如果值以`@`开头，则后面的数字是该项到期的 UNIX 时间戳。此头部的优先级更高。 |
| `Expires` | 此项指定缓存项的过期时间戳。 |
| `Cache-Control` | 启用或禁用缓存 |
| `Set-Cookie` | 这会禁用缓存 |
| `Vary` | 特殊值`*`禁用缓存。 |

还可以通过`proxy_cache_valid`指令明确指定各种响应代码的过期时间间隔：

```
location @proxy {
    proxy_pass http://192.168.0.1:8080;
    proxy_cache my_cache;
    proxy_cache_valid 200 301 302 1h;
}
```

这将把状态码`200`、`301`、`302`的响应过期时间间隔设置为`1h`（1 小时）。请注意，`proxy_cache_valid`指令的默认状态码列表是`200`、`301`和`302`，因此上述配置可以简化为如下：

```
location @proxy {
    proxy_pass http://192.168.0.1:8080;
    proxy_cache my_cache;
    proxy_cache_valid 10m;
}
```

要为负面响应（如`404`）启用缓存，可以在`proxy_cache_valid`指令中扩展状态码列表：

```
location @proxy {
    proxy_pass http://192.168.0.1:8080;
    proxy_cache my_cache;
    proxy_cache_valid 200 301 302 1h;
    proxy_cache_valid 404 1m;
}
```

上述配置将缓存`404`响应`1m`（1 分钟）。负面响应的过期时间间隔故意设置为比正面响应要低得多。这样的一种乐观方法确保了更高的可用性，因为负面响应预计会改善，认为它们是暂时性的，并假设其预期寿命较短。

## 选择缓存键

选择正确的缓存键对缓存的最佳操作至关重要。缓存键必须选择得当，以最大化缓存的预期效率，前提是每个缓存项对于所有后续请求都有有效的内容，这些请求评估为相同的键。这需要一些解释。

首先，让我们考虑效率。当 Nginx 向上游服务器请求以重新验证缓存项时，显然会加重上游服务器的负担。每次缓存命中后，Nginx 会减少对上游服务器的压力，相比于没有缓存时将请求转发到上游服务器的情况。因此，缓存的效率可以表示为*效率 = (命中次数 + 未命中次数) / 未命中次数*。

因此，当无法缓存时，每个请求都会导致缓存未命中，效率为 1。但当我们每次缓存未命中后，接下来有 99 次缓存命中时，效率计算为*(99 + 1) / 1 = 100*，也就是提高了 100 倍！

其次，如果一个文档已经缓存，但并不适用于所有评估为相同键的请求，客户端可能会看到不适用于其请求的内容。

例如，上游服务器会分析`Accept-Language`头，并返回以最适合语言显示的文档版本。如果缓存键不包含语言，第一个请求文档的用户将获得他们语言的版本，并会触发该语言的缓存。所有后续请求该文档的用户都会看到缓存中的版本，因此他们可能会看到错误的语言版本。

如果缓存键包含文档的语言，缓存将为每种请求的语言存储同一文档的多个独立项，所有用户都会看到正确语言版本的文档。

默认的缓存键是`$scheme$proxy_host$request_uri`。

由于以下原因，这可能并不是最优的：

+   位于`$proxy_host`的 Web 应用服务器可以负责多个域名

+   网站的 HTTP 和 HTTPS 版本可以是相同的（`$scheme`变量是多余的，从而在缓存中会重复项）

+   内容可能会根据查询参数有所不同

因此，考虑到之前描述的所有情况，并且鉴于网站的 HTTP 和 HTTPS 版本是相同的，且内容会根据查询参数有所不同，我们可以将缓存键设置为一个更优的值`$host$request_uri$is_args$args`。要更改默认的缓存项键，可以使用`proxy_cache_key`指令：

```
location @proxy {
    proxy_pass http://192.168.0.1:8080;
    proxy_cache my_cache;
    proxy_cache_key "$host$uri$is_args$args";
}
```

此指令以脚本作为参数，该脚本在运行时被评估为缓存键的值。

## 提高缓存效率和可用性

可以提高缓存的效率和可用性。你可以防止项目在未达到一定请求次数之前被缓存。这可以通过使用`proxy_cache_min_uses`指令来实现：

```
location @proxy {
    proxy_pass http://192.168.0.1:8080;
    proxy_cache my_cache;
    proxy_cache_min_uses 5;
}
```

在前面的示例中，响应会在项被请求不低于五次时缓存。这可以防止缓存中被不常用的项填满，从而减少用于缓存的磁盘空间。

一旦项目过期，可以在不被驱逐的情况下重新验证它。要启用重新验证，可以使用`proxy_cache_revalidate`指令：

```
location @proxy {
    proxy_pass http://192.168.0.1:8080;
    proxy_cache my_cache;
    proxy_cache_revalidate on;
}
```

在上述示例中，一旦缓存项过期，Nginx 将通过向上游服务器发出条件请求来重新验证该缓存项。此请求将包括 `If-Modified-Since` 和/或 `If-None-Match` 头部，作为参考缓存版本。如果上游服务器响应 `304 Not Modified`，则缓存项保持在缓存中，并且过期时间戳会被重置。

可以禁止多个同时请求同时填充缓存。根据上游服务器的反应时间，这可能加速缓存的填充，同时减少上游服务器的负载。要启用此行为，可以使用 `proxy_cache_lock` 指令：

```
location @proxy {
    proxy_pass http://backend;
    proxy_cache my_cache;
    proxy_cache_lock on;
}
```

一旦启用此行为，仅允许一个请求填充与之相关的缓存项。其他与此缓存项相关的请求将等待，直到缓存项被填充或锁定超时过期。锁定超时时间可以通过 `proxy_cache_lock_directive` 指令指定。

如果需要更高的缓存可用性，可以配置 Nginx，在请求引用缓存项时，回复陈旧的数据。当 Nginx 作为分发网络中的边缘服务器时，这非常有用。即使主站点遇到连接问题，用户和搜索引擎爬虫也能看到你的网站是可用的。要启用陈旧数据的回复，可以使用 `proxy_cache_use_stale` 指令：

```
location @proxy {
    proxy_pass http://backend;
    proxy_cache my_cache;
    proxy_cache_use_stale error timeout http_500 http_502 http_503 http_504;
}
```

上述配置启用了在连接错误、上游错误（`502`、`503` 或 `504`）以及连接超时的情况下，使用陈旧数据进行回复。下表列出了 `proxy_cache_use_stale` 指令参数的所有可能值：

| 值 | 含义 |
| --- | --- |
| `error` | 发生了连接错误，或在发送请求或接收回复过程中发生了错误 |
| `timeout` | 连接在设置、发送请求或接收回复过程中超时 |
| `invalid_header` | 上游服务器返回了空的或无效的回复 |
| `updating` | 在缓存项更新时启用陈旧的回复 |
| `http_500` | 上游服务器返回了 HTTP 状态码 `500`（内部服务器错误） |
| `http_502` | 上游服务器返回了 HTTP 状态码 `502`（错误网关） |
| `http_503` | 上游服务器返回了 HTTP 状态码 `503`（服务不可用） |
| `http_504` | 上游服务器返回了 HTTP 状态码 `504`（网关超时） |
| `http_403` | 上游服务器返回了 HTTP 状态码 `403`（禁止） |
| `http_404` | 上游服务器返回了 HTTP 状态码 `404`（未找到） |
| `off` | 禁用使用陈旧的回复 |

## 处理异常和边界情况

当缓存不需要或不高效时，可以绕过或禁用缓存。这种情况可能出现在以下实例中：

+   资源是动态的，取决于外部因素而变化

+   资源是用户特定的，并且根据 cookies 的不同而有所变化

+   缓存的价值不大

+   资源不是静态的，例如视频流

当强制绕过时，Nginx 会将请求转发到后台服务器，而不会查找缓存中的项。可以使用 `proxy_cache_bypass` 指令来配置绕过：

```
location @proxy {
    proxy_pass http://backend;
    proxy_cache my_cache;
    proxy_cache_bypass $do_not_cache $arg_nocache;
}
```

这个指令可以接受一个或多个参数。当其中任何一个评估为真（非空值且不为 0）时，Nginx 不会查找缓存中的项，而是直接将请求转发到上游服务器。该项仍然可以存储在缓存中。

要防止将项存储在缓存中，可以使用 `proxy_no_cache` 指令：

```
location @proxy {
    proxy_pass http://backend;
    proxy_cache my_cache;
    proxy_no_cache $do_not_cache $arg_nocache;
}
```

这个指令的作用与 `proxy_cache_bypass` 指令完全相同，但它防止将项目存储在缓存中。当仅指定 `proxy_no_cache` 指令时，项目仍然可以从缓存中返回。结合使用 `proxy_cache_bypass` 和 `proxy_no_cache` 可以完全禁用缓存。

现在，让我们考虑一个实际的例子，即当需要为所有用户特定页面禁用缓存时。假设你有一个由 WordPress 驱动的网站，想要为所有页面启用缓存，但为所有定制的或用户特定的页面禁用缓存。要实现这一点，你可以使用类似下面的配置：

```
location ~* wp\-.*\.php|wp\-admin {
    proxy_pass http://backend;

    proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
}

location / {
    if ($http_cookie ~* "comment_author_|wordpress_|wp-postpass_" ) {
        set $do_not_cache 1;
    }

    proxy_pass http://backend;

    proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;

    proxy_cache my_cache;
    proxy_cache_bypass $do_not_cache;
    proxy_no_cache $do_not_cache;
}
```

在前面的配置中，我们首先将所有与 WordPress 管理区域相关的请求委托给上游服务器。然后，我们使用 `if` 指令检查 WordPress 登录 cookies，如果存在，则将 `$do_not_cache` 变量设置为 `1`。接着，我们为所有其他位置启用缓存，但使用 `proxy_cache_bypass` 和 `proxy_no_cache` 指令禁用当 `$do_not_cache` 变量设置为 `1` 时的缓存。这将禁用所有带有 WordPress 登录 cookies 的请求的缓存。

前面的配置可以扩展以从参数或 HTTP 头中提取无缓存标志，从而进一步调整缓存。

# 总结

在这一章中，你学习了如何使用代理和缓存——这是 Nginx 最重要的特性之一。这些特性实际上定义了 Nginx 作为 Web 加速器的角色，掌握这些特性对于充分发挥 Nginx 的功能至关重要。

在下一章，我们将探讨如何在 Nginx 中重写引擎的工作原理以及访问控制的基础知识。
