# 第八章：我们想聊天

在上一章中，我们刚刚深入研究了 Nagios 插件的规划和编码。我们研究了理解插件是什么、期望它做什么以及如何将其与监控系统集成所需的部分；这是因为创建脚本或程序不仅仅是编写代码本身：这只是一个漫长而复杂工作流程的最后一步。

现在，我们将尝试一些稍微不同的事情，创建一个小客户端，将信息发送到 Slack 频道。这将使我们接触一些新话题，比如 JSON，并且了解如何与基于云的服务进行互动。我们不会编写一个功能完备的客户端（可以读取和写入），而只是实现发送功能，因为 Bash 不是构建完整交互式客户端的最佳工具。这里的目标是编写一个工具，我们可以用它向频道发送通知，比如通知频道成员某个任务的结果、定时任务的执行情况等。

# Slack 消息服务

**Slack** 是 **Searchable Log of All Conversation and Knowledge** 的缩写，它是一个广泛用于小型和大型团队共享信息、文档和想法的协作工具。Slack 一目了然，提供以下功能：

+   **聊天室：** 无论是公共还是私人，聊天室允许团队成员讨论任何话题，而不会干扰到其他人。一个频道是持久存在的，可以有一个话题，任何被邀请的人都可以参与其中。

+   **直接消息：** 用户可以向其他人或群组发送直接消息，以便进行私人对话。

+   **集成搜索：** Slack 中的所有内容都可以搜索，包括聊天中共享的消息、上传的文件以及我们接触过的人；这可能是该平台最有趣的功能之一。

+   **通话：** 可以在频道内或直接消息中进行直接或群组通话；并且无需外部应用程序，因此无需离开平台。

+   **团队：** 任何人都可以使用团队所有者提供的 URL 或邀请加入团队；这可能是该工具最具社区感的功能。

+   **与外部服务的集成：** Slack 可以连接多种外部服务，以增强其功能，从 Google Drive 到 Dropbox，从 GitHub 到 Zendesk，仅举几个例子。

+   **客户端：** 该平台拥有大量客户端，无论是本地客户端还是通过网络的客户端，适用于多个平台，例如 Windows、macOS、Linux、Android，仅举几例，因此我们不需要另一个客户端。

所以我们希望通过向某个频道发送消息并将其显示得漂亮来与 Slack 互动。第一步将是创建一个新的团队，网址如下：

[`slack.com/`](https://slack.com/)

![](img/00020.jpeg)

创建新团队

在第一步中，我们将创建一个新团队。我们需要完成所有常规操作，输入电子邮件、确认代码并选择团队 URL：

![](img/00021.jpeg)

一个团队 URL 将聚集有共同兴趣的人，如果你完成了，邀请一些朋友；你的团队就准备好加入行列了！

![](img/00022.jpeg)

我们已经准备好分享我们的消息了。创建团队空间完成后，我们将有两个默认频道可用：

```
#general #random

```

我们可以使用默认频道，也可以创建一个新频道；在我们的例子中，我们将创建一个全新的公共频道，名为 `#test`。

在这里，我们将通过 WebHook 发送脚本创建的所有消息。现在，我们引入了一个新术语——WebHook，这对我们的脚本至关重要，因为它是我们与 Slack 交互的一种方式。所以，最好停下来深入理解 Slack 中 WebHook 的概念。

# Slack WebHooks

什么是 WebHook？我们可以将其定义为一种使网页对用户输入做出响应的方法，基于简单的 HTTP POST 方法来支持用户定义的 HTTP 回调。听起来还是有点模糊，是吧？换句话说，Slack 有一些端点，敏感的 URL；当你通过 HTTP 向这些端点发送内容时，你实际上是在与 Slack 通信。使这些 WebHook 有趣的是它们是无状态的，因为它们不依赖于与服务保持持续连接；你只需在需要发布或检索一些信息时向 Slack 发送请求。Slack 支持两种不同类型的 WebHook：

+   **Incoming WebHook:** 这是我们在希望某些消息出现在测试频道时发送消息的 URL

+   **Outgoing WebHook:** 这是 Slack 用于通知我们某些频道事件的 URL

我们将使用 Incoming WebHook 来进行我们的通知脚本。那么，我们需要哪些东西呢？我们需要以下内容：

+   Slack Incoming WebHook 已连接到我们的某个频道。

+   一个包含我们要发布的消息的 JSON。

+   一个应用程序，它将连接到 URL 并发布 JSON。这将是我们的脚本。

1.  我们的第一个任务是创建一个新的 Incoming WebHook，并将其连接到我们的测试频道。作为团队管理员，我们需要登录到 [`my.slack.com/services/new/incoming-webhook/`](https://my.slack.com/services/new/incoming-webhook/)，然后从下拉菜单中选择我们的测试频道，如下所示：

![](img/00023.jpeg)

1.  从下拉菜单中选择 #test 频道。

1.  现在，点击“添加 Incoming WebHooks 集成”按钮，我们将进入 Incoming WebHooks 页面，在这里我们可以找到用于发送消息到测试频道的 URL。

![](img/00024.jpeg)

1.  在 Incoming WebHooks 页面中，您可以找到您新创建的 WebHook。

在我们的例子中，WebHook 的 URL 为：[`hooks.slack.com/services/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`](https://hooks.slack.com/services/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)。

如前所述，我们有两种方式可以通过这个 WebHook 发送消息：

+   作为 POST 请求的负载参数中的 JSON 字符串

+   作为 POST 请求体中的 JSON 字符串

所以，JSON 对我们的消息系统至关重要，但到底什么是 JSON 呢？

# 什么是 JSON？

**JSON** 或 **JavaScript 对象表示法** 是一种开放的标准格式（ECMA-404），广泛用于应用程序之间交换数据。它在 2007 年作为 JavaScript 编程语言的一个子集创建，并迅速被许多语言采纳，成为一种中立的数据传递方式，不受发送和接收应用程序语言的限制。我们可以看到 JSON 文件以两种不同的结构建模。

一个由名称：值对组成的对象，使用括号打开和关闭，每个名称与相应的值之间用冒号分隔，每对之间用逗号分隔，如以下示例：

```
{
"name" : "Janet",
"state" : "California",
"cake" : "Toffee sticky pudding"
}

```

数组中的值按顺序排列，数组以方括号打开和关闭，值之间用逗号分隔：`[ "1", "2", "3"]`。

JSON 中的一个值可以是数字、对象、数组、字符串、true、false 或 null。

所以，JSON 将是我们使用 WebHook 将消息传递到频道的格式，它将以由名称：值对组成的对象结构呈现。Slack 中最简单的消息将包含一个简单的 `"text"` 关键字作为 JSON 的名称部分，消息本身作为其值：

```
{
    "text": "This is the first line of a message This is 
             the second line."
}

```

现在，我们已经将第一条消息格式化为整齐的 JSON，我们必须将该内容传递到我们的`#test`频道。

# 你喜欢使用 cURL 吗？

发布 JSON 内容的最简单方法之一是使用外部工具，比如 cURL，其任务是通过 URL 传输数据。我们有两种数据传输方式：

+   直接作为 HTTP POST 请求正文中的 JSON，带有特定的 content-type 头，这是首选方法。

+   作为 URL 转义的 JSON，作为 POST 正文的一部分，通过 payload 参数传递

在第一种情况下，我们将使用 cURL 配合以下参数：

```
-X POST

```

它指定了与 HTTP 服务器通信的方法。默认方法是 `GET`，但在这里，我们必须使用 `POST` 来传输一些信息：

```
-H 'Content-type: application/json' 

```

此选项允许我们将额外的头信息发送给 HTTP 服务器。在我们的案例中，我们正在发送一个 **多用途互联网邮件扩展** (**MIME**) 类型，通知 Slack 服务器它应该期望在正文中接收一个 JSON（rfc4627）应用程序类型的对象：

```
--data '{"text":"This is our first message,.n which continues on another line."}' 

```

`--data` 选项允许我们将 JSON 对象作为 `POST` 请求的正文发送，以便它可以传递给 HTTP 服务器进行处理：

[`hooks.slack.com/services/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`](https://hooks.slack.com/services/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)

这是最后一部分：`cURL` 将调用的 WebHook 地址，以进行 `POST` 请求。现在，我们已经具备了所有必要的信息，所以只是创建我们的命令行问题。如果你还没有安装 `cURL` 工具，请先安装它，然后在命令行中输入以下命令：

```
cURL -X POST -H 'Content-type: application/json' --data 
'{
"text":"This is a line of text.nAnd this is another one.
"}' https://hooks.slack.com/services/T4P7TPSP9/B4ND2E2E4/
lIzhH84lg21ZJ0zdaeQHZ7ls

```

这是我们发送到频道的第一条消息：

![](img/00025.jpeg)

一个简单的 cURL 请求给我们带来了第一条消息。

发送消息的第二种方式是将其 URL 编码放入 `POST` 请求体的有效载荷参数中。您的 URL 将变成一个充满了百分号字符的杂乱字符串，但它是更传统的方式，因此你可能会更有信心使用它。

要在有效载荷参数中发送 JSON 文件，我们需要以下内容：

```
-X POST 

```

这指定了用于与 HTTP 服务器通信的方法。默认方法是 `GET`，但在这里，我们需要 `POST` 一些信息：

```
--data-URLencode 'payload={"text":"This is our second message."}'

```

这个 URL 编码了我们的 JSON，以便它可以被发布。数据部分的结构稍有不同，因为为了符合 CGI 规范，它必须以一个关键字开始：

[`hooks.slack.com/services/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`](https://hooks.slack.com/services/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)

最后，我们有了 URL，它与第一种方法相同。让我们组合我们的命令行并试试看：

```
cURL -X POST  --data-URL
encode 'payload={"text":"This is our second message."}' 
https://hooks.slack.com/services/T4P7TPSP9/B4ND2E2E4/lI
zhH84lg21ZJ0zdaeQHZ7ls

```

![](img/00026.jpeg)

创建一个新团队。

就是这样，我们的第二条消息已经显示在频道中。它仍然是基本形式，没有任何额外的效果，但它完成了工作，帮助我们理解如何与 Slack 服务器互动。无论如何，基本功能已经很好，但为什么不尝试用一些特殊效果来美化我们的对话呢？

# 格式化我们的消息

我们可以为消息添加一些样式，从文本属性到链接和按钮，让它们不仅仅是一堆简单的文本。实际上，手动修改所有有效载荷来检查哪些属性组合最适合您的消息可能会太麻烦，但 Slack 提供了一个在线的 *消息构建器*，让你可以在不发布消息的情况下自定义和预览消息。只需打开浏览器，访问 [`api.slack.com/docs/messages/builder,`](https://api.slack.com/docs/messages/builder) 并开始享受其中的乐趣：

![](img/00027.jpeg)![](img/00028.jpeg)

有效载荷编辑器允许你在发送消息之前进行尝试。

在上方的框中，我们可以根据需要构造有效载荷，并在下方的框中预览它，所以让我们看看可以添加到消息中的一些有趣的内容：

+   **粗体：** 好吧，这是经典之作。你只需将文本字符串用两个星号包裹，即可将其加粗，试试这个有效载荷：

```
{
    "text": "This is a *bold string* and this a *bold* word"
}

```

![](img/00029.jpeg)

这是我们在消息构建器中看到的粗体文本效果。

+   **斜体：** 这强调了单词或句子的重点，你只需要将字符串用两个下划线包围即可实现这一效果：

```
{
    "text": "This is a _string in italics_."
}

```

+   **代码：** 如果你正在编写属于命令行或某种代码的文本，可以将其包裹在反引号之间，使其突出显示：

```
{
    "text": "We can use the code attribute to write some code    
             like:n`x=y+1`"
}

```

![](img/00030.jpeg)

这是您的代码在频道中的显示效果。

+   **代码块：** 那么如何将 `多行` 代码块包裹起来呢？让我们看看：

```
{
    "text": "We can use the block code attribute to write some 
     multi line code like:n```y=1nx=y+1nx=2```"
}

```

![](img/00031.jpeg)

`多行` 代码块与单行代码字符串相比会略有不同。

+   **URL 链接:** 你可以通过将 URL 包围在`<>`中，在消息中插入可点击的链接。你可以用两种不同的方式插入链接：

    +   只需将链接本身用`<>`包围：`<http://www.packtpub.com>`

    +   在前面的语法中添加`"|linked"`，使得链接字符串指向的 URL 为`<http://www.zarrelli.org|this>`"

因此，我们的载荷使用这两种语法的形式可能是这样的：

```
{
    "text": "This is a web link <http://www.packtpub.com> and 
          <http://www.zarrelli.org|this too>"
}

```

![](img/00032.jpeg)

我们有两种不同的方式链接 URL：

+   **电子邮件地址链接:** 类似于 URL 链接，只需将电子邮件链接和`"|linked"`用`<>`包围：

```
{
    "text": "Just email the <mail-
to:giorgio.zarrelli@gmail.com|author>."
}

```

![](img/00033.jpeg)

点击突出显示的单词，您的电子邮件客户端将启动，并自动填写收件人地址。

+   **日期:** 我们可以使用 Unix 时间戳和一些选择器格式化消息中的日期，并可以修改其显示方式。我们可以选择性地添加 URL 链接，但必须始终提供回退文本，以防时间令牌转换失败。此时的关键词是`<!date>`，但其语法稍复杂：

```
{
    "text": "<!date^unix_timestamp^ Some optional text {date_selector}|Fallback text>"
}

```

我们有很多不同的选择器可以用来修改频道中日期和时间的显示方式：

+   **`{date}`:** 你的日期将以经典的*3 月 26 日*形式显示，试试以下载荷：

```
{
    "text": "<!date¹⁴⁹⁰⁵³¹⁶⁹⁵^ {date}|Fallback>"
}

```

所以，我们的`cURL`命令将是：

```
cURL -X POST -H 'Content-type: application/json' --data 
'{"
text": 
"<!date¹⁴⁹⁰⁵³¹⁶⁹⁵^ {date}|Fallback>"
}' https://hooks.slack.com/services/T4P7TPSP9/B4ND2E2E4/
lIzhH84lg21ZJ
0zdaeQHZ7ls

```

![](img/00034.jpeg)

下面是我们消息在频道中的显示方式：

+   **`{date_short}`**: 正如名称所示，这是一个更紧凑的格式，显示为`"3 月 26 日"`

+   **`{date_long}`**: 这将显示一个扩展日期，格式为`"星期天，3 月 26 日"`

+   **`{date_pretty}`**: 这将显示为`{date}`，但当合适时，会使用`"昨天"`、`"今天"`或`"明天"`。

+   **`{date_short_pretty}`**: 这将以`{date_short}`格式显示日期，但当合适时，会使用`"昨天"`、`"今天"`或`"明天"`。

+   **`{date_long_pretty}`**: 这将显示为`{date_long}`，但当合适时，会使用`"昨天"`、`"今天"`或`"明天"`。

+   **`{time}`**: 这将以 12 小时格式显示时间，在我们的例子中是`1:34 PM`；但如果客户端设置为 24 小时格式，它将显示为`13:34`

+   **`{time_secs}`**: 这将以 12 小时格式显示时间，精确到秒，如`1:34:55 PM`；如果客户端设置为 24 小时格式，则会显示为`13:34:55`

我们还可以将 URL 添加到日期中，这样当你点击日期/时间时，会跳转到相应的网站，格式如下：

```
{
    "text": 
"<!date¹⁴⁹⁰⁵³¹⁶⁹⁵^{date}^http://www.packtpub.com|Fallback>"
}

```

显然，你可以混合格式化器，以便得到更有意义的消息，如下所示：

```
{
    "text": "<!date¹⁴⁹⁰⁵³¹⁶⁹⁵^Let's meet on {date} at 
{time}|Meeting info>"
}

```

完整的`cURL`命令如下所示：

```
cURL -X POST -H 'Content-type: application/json' --data '{"text": 
"<!date¹⁴⁹⁰⁵³¹⁶⁹⁵^Let's meet on {date} at {time}|Meeting info>"}' https://hooks.slack.com/services/T4P7TPSP9/B4ND2E2E4/lIzhH84lg21ZJ
0zdaeQHZ7ls

```

除了日期，我们还可以在消息中使用一些特殊命令，让观众注意到我们：

+   **`<!here>`:** 这将通知所有在频道中活跃的团队成员：

```
{
  "text": "<!here><!date¹⁴⁹⁰⁵³¹⁶⁹⁵^{date}^http://www.packtpub.com|Fallback>"
}

```

+   **`<!channel>`:** 这将通知所有在频道中的团队成员，无论他们的状态如何。频道名称旁将出现一个通知图标。

+   **`<!group>`:** 这是`<!channel>`的同义词，二者都可以在频道或小组内使用。

+   **`<!everyone>`:** 这会通知我们所有的团队成员。通常可以在团队频道中使用，通常称为#general。

![](img/00035.jpeg)

使用其中一个通知标签将导致通知图标出现在频道名称附近。

如果我们想引起观众的注意，可以使用在社交网络中流行的经典工具：所谓的表情符号。Slack 允许我们显示任何我们喜欢的表情符号，因此让我们访问[`unicodey.com/emoji-data/table.htm,`](https://unicodey.com/emoji-data/table.htm)，选择你最喜欢的小图案。

一旦我们选择了我们的表情符号，就可以像这样构建一个有效负载：

```
{
    "text": "Guys, read carefully the examples:bangbang:, take 
your time :hourglass:, be :b:rave and love the :shell:"
}

```

然后执行`cURL`：

```
cURL -X POST -H 'Content-type: application/json' --data '{ "text": "Guys, read carefully the examples:bangbang:, take your :hourglass:, be :b:rave and love the :shell:"}' https://hooks.slack.com/services/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

```

以下截图中的结果非常棒：它们以一种花哨的方式吸引注意力，因此我们的团队成员将不会被我们的消息所困扰！

![](img/00036.jpeg)

一条花哨的消息可以很好地传达声明的紧迫性。最后，我们可以通过使用`<@user| 可选的处理符号>`来直接指定某个用户，我们还可以使用以下方式覆盖我们想要发送消息的频道：

```
"channel": "#name_of_channel"

```

所以，我们可以向频道`#general`中的用户`Giorgio`发送请求，邀请他加入测试频道：

```
{ 
   "text": "Hey <@giorgio|Giorgio> did you join the 
    <#test|Test>   channel?", "channel": "#general"
}'

```

所以，完整的`cURL`命令行如下：

```
cURL -X POST -H 'Content-type: application/json' --data '{ "text": "Hey <@giorgio|Giorgio> did you join the <#test|Test> channel?", "channel": "#general"}' https://hooks.slack.com/services/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

```

使用此消息，用户 Giorgio 将在`#general`频道收到通知；内容将引起他的注意。

你不需要将消息的全部文本转换为 HTML，但有三个字符必须被转换为 HTML 实体：

&必须替换为&amp

<必须替换为&lt

> 必须替换为&gt

到目前为止，我们做的事情相对简单——这就是你可以在没有任何障碍的情况下使用普通的 JSON 来完成的，但如果我们想进一步美化我们的消息，我们必须使用消息附件，它将使我们能够发送图片、附加按钮等。因此，我们的下一步将是使用消息附件，看看如何不仅美化我们的消息，还能使它们更有用和更有效。

# 消息附件

消息附件可以让我们传递更多内容给用户，并且可以用更多的花哨效果显示出来；但我们必须注意 Slack 规定的一个限制：每条消息最多只能包含 20 个附件。这是有道理的，否则我们的消息会变得非常混乱，分散普通用户的注意力。

到目前为止，我们看到的是一个简单的 JSON：一个一级对象，大致是这样的示例：

```
{
    "text": "This is the first line of a messagen
    This is the second line."
}

```

在消息附件中，尽管我们将看到更多细节、更多内容修饰符，以及像我们刚刚看到的那种不够复杂的平面结构，我们需要一个结构化的容器，仍然是 JSON；但这次，它将是一个数组，包含多个属性，像这样：

```
{
    "attachments": [
        {
            "fallback": "Text to be displayed in case of client 
                         not supporting formatted text",
            "color": "#ff1493",
            "pretext": "This goes above the attachment",
            "author_name": "Giorgio Zarrelli",
            "author_link": "http://www.zarrelli.org",
            "author_icon": "https://www.zarrelli.org/blog/wp-content/uploads/2017/03/IMG_20161113_150052.jpg",
            "title": "Title example",
            "title_link": "http://www.zarrelli.org",
            "text": "This text is optional and it is shown in the  
                    attachment",
            "fields": [
                {
                    "title": "Priority",
                    "value": "Medium",
                    "short": false
                }
            ],
            "image_URL": 
"http://www.zarrelli.org/path/to/image.jpg",
            "thumb_URL": "https://www.zarrelli.org/blog/wp-content/uploads/2017/03/IMG_20161113_150052-1-e1490610507795.jpg",
            "footer": "Slack API",
            "footer_icon": "https://www.zarrelli.org/blog/wp-content/uploads/2017/03/IMG_20161113_150052.jpg",
            "ts": 1490531695
        }
    ]
}

```

这个片段看起来怎么样？我们可以在下面的截图中看到附件是如何显示的：

![](img/00037.jpeg)

一条消息中包含很多内容，并且有一丝粉色的点缀！

这是一个不错的输出，远比之前的基本示例更好；但是事情变得有些复杂，因为 JSON 的结构变得更加复杂，且我们有更多的字段可用。所以，让我们看一下主要指令，这样我们就可以理解它们的含义和限制。

我们正在查看的是 20 个可能附件中的一个，它以一个回退开始。正如名称所示，这是没有任何标记的纯文本，如果客户端不支持格式化文本，它将被显示：

+   **`color`**：你可以为消息指定颜色，为左侧栏着色。这对于让消息在讨论流中脱颖而出，或是快速查看其严重性非常有用。颜色有三种预定义属性：

    +   `good`：这将把侧边栏变为绿色。

    +   `warning`：这将把侧边栏变为黄色。

    +   `danger`：这将把侧边栏变为红色。

除了这些预定义的设置外，你可以使用任何在十六进制代码中定义的颜色，将左侧的栏变成一个引人注目的高亮标记：

+   **`Pretext`**：这是可选文本，显示在附件上方。

+   **`author_name`**：这是消息作者的名字。

+   **`author_link`**：任何有效的 URL 都会将作者的名字内容转化为一个链接。只有在`author_name`可用时才有效。

+   **`author_icon`**：任何指向 16x16 像素图像的有效 URL，将显示在`author_name`的左侧：只有在作者的名字可用时才有效。

如果有的话，作者的信息将显示在每条消息的最开始处。

+   **`title`**：正如名称所示，这是消息的标题，它会以更大的字号和粗体文本显示在消息的顶部，但在作者信息之前。

+   **`title_link`**：这是一个完整的 URL，将使标题变为可点击的链接。

+   **`Text`**：这是消息的实际内容，内容可以使用我们在前面页面中看到的基本格式进行格式化。如果内容超过 500 个字符或 5 行换行，内容将折叠，并且“显示更多”链接将允许用户展开内容。文本字段中的任何 URL 都不会展开。

+   **`Fields`**：这是主数组中的一个数组，里面的元素将在附件中以表格形式显示。你可以在数组中放多个哈希值，只需用逗号分隔它们，如下例所示：

```
"fields": [
         {
                   "title": "Who's in charge",
                   "value": "Giorgio",
                   "short": true
                },
                {
                   "title": "Priority",
                   "value": "Medium",
                   "short": true
                }
          ]

```

+   **`title`**：这是简单文本，显示在值上方，以粗体显示。它不能包含任何标记。

+   **`value`**：这可以包含格式化为我们在前面页面中看到的标记的多行文本。

+   **`short`**：这个可选项将使值变得足够简短，可以与其他值并排显示，如下所示：

![](img/00038.jpeg)

值可以并排显示。

+   **`image_URL`：** 这是我们选择的任何有效 URL，指向 PNG、JPEG、GIF 或 BMP 格式的有效图像。显示的尺寸为 400 x 500 像素，任何宽度或高度超过该尺寸的图像会自动缩放，保持原始纵横比。该图像将显示在消息附件中。

+   **`thumb_URL`：** 这是指向 PNG、JPEG、GIF 或 BMP 格式图像文件的任何 URL。它将在附件消息的右侧显示，并会按 75 x 75 像素的比例缩放保持纵横比。文件大小限制为小于 500 KB。

+   **`footer`：** 这是简短的文本片段，最多 300 个字符，用于向读者提供一些额外的信息。

+   **`footer_icon`：** 提供指向图像的有效 URL，图像将显示在页脚文本旁边。该图像将以 16 x 16 像素的固定大小显示，仅当你提供了页脚时才会显示。

+   **`ts`：** 每条消息都有自己的时间戳，当发布时，然而我们可以通过 `ts` 字段和以纪元时间格式表达的时间信息，向消息附件中提及的事件或发生的事情附加特定的时间戳。

就我们关心的内容而言，重点在于消息格式化。有些花哨的功能如按钮，但这需要一个能够从频道读取并响应用户操作的完整应用程序。目前，我们将坚持通过脚本与频道进行更简单的交互，只是发送格式良好的信息；但没有人阻止你从这个例子出发，构建更复杂且更符合需求的功能。

# 我们的简易 Slack 聊天脚本

是时候开始规划我们的 Slack 脚本了，第一步是问自己，我们希望它做什么。让我们回顾一下需求，脚本必须执行以下操作：

+   接受要显示的文本消息：必填

+   接受消息标题：必填

+   接受 title_link：可选，仅在标题可用时使用

+   接受回退消息：必填

+   接受 author_name：可选

+   接受 author_link：可选，仅在 author_link 可用时使用

+   接受 author_icon：可选，仅在 author_link 可用时使用

+   接受颜色：必填

+   接受前置文本：可选

+   接受 fields：必填，必须包括标题、值和短标记

+   接受 image_URL：可选

+   接受 thumb_URL：可选

+   接受 footer：可选

+   接受 footer_icon：可选，仅在页脚可用时使用

+   接受 ts：可选

现在，我们有了一个矩阵来开始构建我们的模块并解析命令行，准备开始编码了。我们知道该怎么做，但由于任务的复杂性，我们将一步步进行，逐步添加模块；由于我们的核心信息是 JSON 格式，我们将从编码其结构开始。但在第一步之前，我们应该做些什么呢？我们需要考虑使用哪些工具。作为开始，我们至少需要 cURL，所以请检查你的系统是否已安装它；如果没有，请安装它。我们脚本的第一行将包含 sha-bang，并尝试在`PATH`中定位该工具：

```
#!/bin/bash
# License: GPL
# 
# Author: Giorgio Zarrelli <zarrelli@linux.it>
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License version 2 as
# published by the Free Software Foundation.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program. If not, see <http://www.gnu.org/licenses/>.
#
# Retrieve the full path to the system utilities
cURL=$(which cURL)

```

这些第一行代码类似于之前的 Nagios 插件，且这个脚本同样以许可证声明开始。声明许可证可能没什么用，但如果我们计划将脚本提供给公众使用，就有责任告知潜在用户他们能做些什么。此书的作者鼓励在 GNU GPL 许可证下发布软件，因为它使得使用该软件创建新程序并重用代码更加容易。但最终选择使用哪种许可证，取决于程序的创建者。要了解可用的各种 GNU 许可证，我们只需访问[`www.gnu.org/licenses/licenses.html`](https://www.gnu.org/licenses/licenses.html)，查看其中一个许可证，这一定能满足我们的需求。注意，对于这个脚本，我们将使用小写变量，以便熟悉不同编码者采用的不同符号约定。

所以，我们指向了`cURL`工具，但我们怎么能确认它已安装并且可以访问呢？嗯，我们需要记住，赋值给变量的命令替换结果是`which`的输出，只有当传递给`which`的参数在用户`$PATH`环境变量所指示的目录中可访问时，它才会输出某些内容。做个测试，让我们以 cURL 作为参数调用`which`：

```
zarrelli:~$ which cURL
/usr/bin/cURL

```

现在，让我们用一些简短的说明来调用`which`：

```
zarrelli:~$ which cr234a
zarrelli:~$ 

```

输出为空，所以我们的`cURL`变量没有值。再做一次检查，让我们以 root 用户测试`ifconfig`：

```
root:# which ifconfig
/sbin/ifconfig

```

然后，让我们再作为非特权用户检查一次：

```
zarrelli:~$ which ifconfig
zarrelli:~$

```

因为`ifconfig`的路径仅在 root 用户的`$PATH`变量中，所以对于非特权用户，`which`将返回空值。基于这个情况，我们可以用几行代码实现一个检查：

```
if [ -z "$cURL" ]
   then
         echo "Cannot reach the utility, is it in the $PATH or 
even installed?"
         exit 1
      else
         echo "The utility is reachable"
fi

```

所以，如果`$cURL`变量不为空，表示该工具可访问；如果为空，则会收到警告，并带着错误退出脚本：

```
zarrelli:~$ ./my_slack.sh 
The utility is reachable

```

很好，似乎有效。现在让我们把`cURL=$(which cURL)`改成`cURL=$(which cur1l2)`。

让我们再次运行脚本：

```
zarrelli:~$ /bin/bash -x my_slack.sh 
++ which cur1l2
+ cURL=
+ '[' -z '' ']'
+ echo 'Cannot reach the utility, is it in the $PATH or even in 
stalled?'
Cannot reach the utility, is it in the $PATH or even installed?
+ exit 1

```

正确，脚本退出是因为`which`无法在用户的`$PATH`变量中找到那个无意义的字符串。所以，这个检查可能会派上用场，但按现在的写法并不那么有用，因此让我们将其做成一个函数：

```
# Check if which comes back with a path to a utility
check_which()
{
   for i in "$@"
   do
         if [ -z "$i" ];
               then
                     echo "Cannot reach the utility $i, is it in 
the $PATH or even installed?"
                      exit 1
               else
                      :
           fi
    done
}
check_which "$cURL"

```

我们只需检查输出的路径是否为工具的路径；如果什么也不输出，则变量为空，我们会退出并输出一条信息以及退出代码。如果变量有内容，我们就不做任何处理，因为我们假设这是工具的路径。这个检查类似于我们为 Nagios 插件所做的检查：

```
# Check if we have all the system tools we need
path_exists()
{
for i in "$@"
do
       if [ -e "$i" ];
               then 
                          (( VERB )) && echo "$i is a valid path"
                          disk_exists "$DISK"
               else
                          (( VERB )) && echo "$i is not reachable, 
is this the correct path?"
                        echo "SMART UNKNOWN: Please check the 
plugin"
                        exit "$STATE_UNKNOWN"
        fi
done
}

```

另一个检查测试的是变量内容是否指向一个文件。哪个更好？取决于你要检查的内容。如果我们需要验证变量指向一个真实的文件，那么`[ -e "$i" ]`是我们要找的。否则，当我们想做更通用的检查时，`[ -z "$i" ]`就能完成任务。

我们在脚本中接下来需要做什么？让我们回顾一下几页前我们做的一个`cURL`：

```
cURL -X POST -H 'Content-type: application/json' --data '{"text": "<!date¹⁴⁹⁰⁵³¹⁶⁹⁵^ {date}|Fallback>"}' https://hooks.slack.com/services/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 

```

一旦我们处理完`cURL`命令，就必须管理请求头：

```
-X POST
-H 'Content-type: application/json'
--data

```

这些是静态的，不会改变，因此我们可以直接使用它们而不需要放入变量中。

我们不能忘记我们的 WebHook URL：

```
# WebHook URLwebhook="https://hooks.slack.com/services/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"

```

现在，是时候构建我们的第一个静态负载了；这部分有点棘手。由于负载是一个长多行的 JSON，将其写成一长行会很麻烦，因此我们将把这个负担交给一个函数，它将为我们生成这些内容；而且它也会被格式化得很好：

```
generate_payload()
{
   cat <<MARKER
{
    "attachments": [
       {
            "fallback": "Text to be displayed in case of client 
                          not supporting formatted text",
            "color": "#ff1493",
            "pretext": "This goes above the attachment",
            "author_name": "Giorgio Zarrelli",
            "author_link": "http://www.zarrelli.org",
            "author_icon": "https://www.zarrelli.org/blog/wp-content/uploads/2017/03/IMG_20161113_150052.jpg",
            "title": "Title example",
            "title_link": "http://www.zarrelli.org",
            "text": "This text is optional and it is shown in the        
                        attachment",
            "fields": [
                {
                   "title": "Priority",
                   "value": "Medium",
                   "short": false
                }
          ],
           "image_URL": 
"http://www.zarrelli.org/path/to/image.jpg",
           "thumb_URL": "https://www.zarrelli.org/blog/wp-
content/uploads/2017/03/IMG_20161113_150052-1-e1490610507795.jpg",
           "footer": "Slack API",
           "footer_icon": "https://www.zarrelli.org/blog/wp-content/uploads/2017/03/IMG_20161113_150052.jpg",
            "ts": 1490531695
        }
    ]
}
MARKER
}

```

我们使用了*here*文档，正如我们之前看到的，这是处理 Bash 脚本中多行内容的最佳方法之一。函数将为我们创建内容，因此让我们通过在脚本底部添加`generate_payload`来验证这一点；然后运行它：

```
zarrelli:~$ ./my_slack.sh 
{
    "attachments": [
        {
            "fallback": "Text to be displayed in case of client 
                         not supporting formatted text",
            "color": "#ff1493",
            "pretext": "This goes above the attachment",
            "author_name": "Giorgio Zarrelli",
            "author_link": "http://www.zarrelli.org",
            "author_icon": "https://www.zarrelli.org/blog/wp
content/uploads/2017/03/IMG_20161113_150052.jpg",
            "title": "Title example",
            "title_link": "http://www.zarrelli.org",
            "text": "This text is optional and it is shown in the 
                       attachment",
            "fields": [
                {
                    "title": "Priority",
                    "value": "Medium",
                    "short": false
                }
            ],
             "image_URL": 
"http://www.zarrelli.org/path/to/image.jpg",
            "thumb_URL": "https://www.zarrelli.org/blog/wp
content/uploads/2017/03/IMG_20161113_150052-1-e1490610507795.jpg",
            "footer": "Slack API",
            "footer_icon": "https://www.zarrelli.org/blog/wp
content/uploads/2017/03/IMG_20161113_150052.jpg",
            "ts": 1490531695
        }
    ]
}

```

太好了，它工作了！我们的内容已经生成，我们可以继续构建命令行。让我们删除脚本底部对函数的调用，并添加以下行：

```
"$cURL" -f -X POST -H 'Content-type: application/json' --data "$(generate_payload)" "$webhook" 

```

`$(generate_payload)` 是一个命令替换，它会将函数生成的输出赋值给`--data`；但不要忘记将其括在双引号中，否则你的输出将逐行处理，而不是作为一个整体。现在是时候保存并执行脚本，检查我们的`#test`频道：

```
zarrelli:~$ ./my_slack.sh 
ok

```

![](img/00039.jpeg)

我们检查了脚本，以确保它按预期工作。

很好，脚本运行正常，我们可以在`#test`频道看到结果，以及命令行中一个小小的`OK`。嗯，还不错，但我们不能依赖第三方输出来判断是否出了问题，因此我们需要修改命令行，以获取一些有用的响应：

```
"$cURL" -f -X POST -H 'Content-type: application/json' --data "$(generate_payload)" "$webhook" && echo " - Success exit code: $?" || echo "There was an error, exit code: $?"

```

我们向`cURL`添加了一个`-f`标志，这样在出错时它会悄无声息地退出，让我们可以在输出中写入有意义的信息。它不是百分百可靠的，正如我们将看到的，有时错误信息会泄漏出来，但它仍然是可用的。然后我们添加了这个：

```
&& echo " - Success exit code: $?" || echo "There was an error, 
exit code: $?"

```

我们以前已经见过这种测试。我们在检查命令是否成功执行，并将退出代码`"$?"`输出到`stdout`。让我们看一下：

```
zarrelli:~$ ./my_slack.sh 
ok - Success exit code: 0

```

太好了！cURL 刚刚在命令行上打印了 `ok`，而且由于执行顺利，我们打印了一个带有 `exit code` 的 `Success` 消息。现在，让我们删除最后一行中的 `$webhook` 中的 `k`，并再次执行脚本：

```
zarrelli:~$ ./my_slack.sh 
cURL: (3) <URL> malformed

```

出现了一个错误，退出代码：`3`。

它应该悄悄失败，但无论如何，我们成功地写出了有意义的错误信息；这正是我们想要的。

我们的第一步已经完成：我们可以向 WebHook 发送一个静态消息，并在 `#test` 频道显示该消息。这很有趣，但不够灵活。我们真正想要的是根据输入修改消息。为了实现这个目标，我们需要将附件中的所有部分转换为变量，以便能够在命令行中传递值。让我们开始创建一些变量：

```
# Message attachment variables
fallback=${fallback:="This is a text shown on older clients"}
text=${text:="This line of text is optional"}

```

现在，我们只需要修改有效负载：

```
{
            "fallback": "$fallback",
            "color": "#ff1493",
            "pretext": "This goes above the attachment",
            "author_name": "Giorgio Zarrelli",
            "author_link": "http://www.zarrelli.org",
            "author_icon": "https://www.zarrelli.org/
            blog/wp-content/uploads/2017/03/IMG_20161113_150052.jpg",
            "title": "Title example",
            "title_link": "http://www.zarrelli.org",
            "text": "$text",
            "fields": [
                {
                    "title": "Priority",
                    "value": "Medium",
                    "short": false
                }
            ],
                "image_URL": 
               "http://www.zarrelli.org/path/to/image.jpg",
                "thumb_URL": "https://www.zarrelli.org/
                 blog/wpcontent/uploads/2017/03/IMG_20
                161113_150052-1-e1490610507795.jpg",
            "footer": "Slack API",
            "footer_icon": "https://www.zarrelli.org/blog/wpcontent/
             uploads/2017/03/IMG_20161113_150052.jpg",
            "ts": 1490531695
        }

```

现在，让我们运行脚本，看看我们的修改是否生效：

```
zarrelli:~$ ./my_slack.sh 
ok - Success exit code: 0

```

看起来它有效；我们可以通过检查 `#test` 频道来确认结果，正如我们在下面的截图中所见：

![](img/00040.jpeg)

我们的变量正在被脚本考虑。

既然我们的变量似乎在正常工作，接下来让我们创建一大堆新的变量：

```
# Message attachment variables
fallback=${fallback:="This is a text shown on older clients"}
color=${color:="good"}
pretext=${pretext:="Announcement:"} author_name=${author_name:="Giorgio Zarrelli"}
author_link=${author_link:="http://www.zarrelli.org"}
author_icon=${author_icon:="https://www.zarrelli.org/blog/wp
content/uploads/2017/03/IMG_20161113_150052.jpg"}
title=${title:="New message"}
title_link=${title_link:="Announcement:"}
text=${text:="This line of text is optional"}
fields_title=${fields_title:="Priority"}
fields_value=${fields_value:="Medium"}
fields_short=${fields_short:="true"}
im-
age_URL=${image_URL:="http://www.zarrelli.org/path/to/image.jpg"}
thumb_URL=${thumb_URL:="https://www.zarrelli.org/blog/wp-content/uploads/2017/03/IMG_20161113_150052-1-e1490610507795.jpg"}
footer=${footer:="Mastering Bash"}
footer_icon=${footer_icon:="https://www.zarrelli.org/blog/wp-content/uploads/2017/03/IMG_20161113_150052.jpg"}
ts=${ts:="1490531695"}

```

显然，有效负载必须相应地进行修改：

```
 {
            "fallback": "$fallback",
            "color": "$color",
            "pretext": "$pretext",
            "author_name": "$author_name",
            "author_link": "$author_link",
            "author_icon": "$author_icon",
            "title": "$title",
            "title_link": "$title_link",
            "text": "$text",
            "fields": [
                {
                    "title": "$fields_title",
                    "value": "$fields_value",
                    "short": "fields_short"
                }
            ],
            "image_URL": "$image_URL",
            "thumb_URL": "$thumb_URL",
            "footer": "$footer",
            "footer_icon": "$footer_icon",
            "ts": "$ts"
    }

```

再次，让我们测试一下我们的代码：

```
zarrelli:~$ ./my_slack.sh 
ok - Success exit code: 0

```

这里的截图显示了我们新格式化的消息：

![](img/00041.jpeg)

看起来我们新的有效负载工作得很好。

现在我们已经有了所有的变量，接下来是创建一个菜单，帮助我们管理用户输入。让我们开始编写一个帮助函数；我们在上一章已经看过如何做了，但这次我们有很多选项要处理，因此我们将每个变量都关联一个选项：

```
-f --fallback
-c --color 
-p --pretext
-an --author_name
-al --author_link
-ai --author_icon
-t --title
-tl --title_link
-tx --text
-ft --fields_title
-fv --fields_value
-fs --fields_short
-iu --image_URL
-tu --thumb_URL
-fr --footer
-fi --footer_icon
-ts –-timestamp

```

我们使用的短选项和长选项是由我们决定的，但我们必须记住一个黄金法则：这些选项必须对潜在的脚本用户有意义，而不是对我们自己，因此我们需要站在用户的角度，尝试以他们的方式思考。一旦我们确定了最佳选项，就可以开始创建实际的命令行解析器：

```
# Parse parameters on the command line
while (( $# > 0 ))
   do
               case "$1" in
               -h | --help)
                         print_help
                             exit 1
                             ;;
               -f | --fallback)
                           shift
                           fallback="$1"
                           ;;
                    -c | --color)
                             shift
                          color="$1"
                         ;;
                    -p | -pretext)
                           shift
                         pretext="$1"
                             ;;
                    -an | --author_name)
                            shift
                         author_name="$1"
                           ;;
               -al | --author_link)
                           shift
                         author_link="$1"
                           ;;
         -ai | --author_icon)
                           shift
                         author_icon="$1"
                           ;;
               -t | --title)
                           shift

                           ;;
               -tl | --title_link)
                               shift
                             title_link="$1"
                               ;;
               -tx | --text)
                           shift
                         author_icon="$1"
                           ;;
               -ft | --fields_title)
                           shift
                        fields_
                          ;;
               -fv | --fields_value)
                           shift
                        fields_value="$1"
                          ;;
               -fs | --fields_title)
                           shift
                         fields_short="$1"
                           ;;
               -iu | --image_URL)
                           shift
                         image_URL="$1"
                            ;;
               -tu | --thumb_URL)
                           shift
                         thumb_URL="$1"
                           ;;
               -fr | --footer)
                           shift
                         footer="$1"
                           ;;
               -fi | --footer_icon)
                           shift
                         image_URL="$1"
                           ;;
               -ts | --timestamp)
                           shift
                        ts="$1"
                          ;;
     *) echo "Unknown argument: $1"
                          print_help
                          exit 1
                          ;;
         esac
         shift
       done

```

现在是 `print_help` 函数：

```
# Print help and usage
print_help()
{
   cat << HERE

   Slack sender v1.0
         ---------------
Please enter one or more of the following options: 
            -h | --help
            -f | --fallback
              -c | --color
              -p | -pretext
             -an | --author_name
             -al | --author_link
             -ai | --author_icon
              -t | --title
             -tl | --title_link
             -tx | --text
             -ft | --fields_title
             -fv | --fields_value
             -fs | --fields_title
             -iu | --image_URL
             -tu | --thumb_URL
             -fr | --footer
             -fi | --footer_icon
             -ts | --timestamp
HERE
}

```

现在，让我们注释掉脚本的最后一行：也就是调用 cURL 的那一行。然后，我们通过 `-h` 参数来调用脚本：

```
zarrelli:~$ ./my_slack.sh -h

Slack sender v1.0
     ------------------- Please enter one or more of the following options:
 -h | --help
 -f | --fallback
 -c | --color
 -p | -pretext
 -an | --author_name
 -al | --author_link
 -ai | --author_icon
 -t | --title
 -tl | --title_link
 -tx | --text
 -ft | --fields_title
 -fv | --fields_value
 -fs | --fields_title
 -iu | --image_URL
 -tu | --thumb_URL
 -fr | --footer
 -fi | --footer_icon
 -ts | --timestamp

```

有一个小问题：即使是 Nagios 插件也有这个问题，因此是时候解决它了。让我们在没有任何参数的情况下调用脚本：

```
zarrelli:~$ ./my_slack.sh -h
zarrelli:~$

```

什么都没有，我们没有任何反馈，因此甚至不知道是否可以使用 `-h` 参数来获取更多信息。我们该如何解决这个障碍呢？好吧，我们有不同的选择；例如，我们可以修改 `while` 条件，或者采用其他策略：

```
if (( $# == 0 ))
   then
         echo "No options provided, you can use -h for help but 
this time I will do it for you..."
         print_help
fi

```

我们必须将这些行放在菜单创建之前，它们会检查输入：如果命令行没有给出任何参数，它将写入错误信息并调用 `print_help` 函数。现在是时候测试我们的脚本，看看会发生什么：

```
./my_slack.sh -c warning -an Me -t "My cli test" -tx "This is a cli text"
ok - Success exit code: 0

```

看起来它有效，`#test` 频道的截图确认了我们的猜测：

![](img/00042.jpeg)

我们的脚本现在接受命令行参数。

# 总结

我们现在能够向`#test`频道发送格式化的消息，但这就是我们可以用这个脚本做的全部吗？不，正如你随着时间推移、积累经验会发现的，编程也是在设定我们的努力范围：我们必须定义目标，进行相应的计划，完成目标，并评估结果。在专业环境中，做得过头违反了项目管理的基本规则，即所谓的*铁三角*，它定义了项目质量作为范围、时间和成本的交集，这三个约束条件驱动我们创建程序。如果在一个程序上花费过长时间或超出目标，成本将会膨胀，整体质量——不是代码质量，而是我们项目的质量——将会受到影响。

这个脚本是一个关于如何规划和执行的示例，展示了如何检查我们需要的信息来编写一个可运行的脚本，以及如何记录我们的步骤。有很多方法可以改进这个框架，例如，允许用户在命令行上传递一个日期，而不是以纪元时间的形式，而是以`date`命令允许的格式传递，然后通过一个小函数或甚至直接在代码中进行转换。我们还可以，例如，检查选项参数，以便用户在命令行中指定选项时，强制传递一个参数。这是一个操作领域，这些可以是如何玩得开心并进一步开发脚本的建议。我们现在将继续深入探讨如何利用子壳执行多个并行进程，以及，像往常一样，如何玩得开心！
