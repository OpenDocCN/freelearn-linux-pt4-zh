# 序言

首先，你将了解 Linux shell 及其选择 bash shell 的原因。然后，你将学习如何编写一个简单的 bash 脚本，以及如何使用 Linux 编辑器编辑你的 bash 脚本。

接下来，你将学习如何定义变量以及变量的可见性。之后，你将学习如何将命令执行输出存储到变量中，这称为命令替换。此外，你将学习如何使用 bash 选项和 Visual Studio Code 调试代码。你将学习如何通过接受用户输入使你的 bash 脚本具有交互性。然后，你将学习如何读取用户传递给脚本的选项及其值。接下来，你将学习如何编写条件语句，如`if`语句，以及如何使用`case`语句。之后，你将学习如何使用 vim 和 Visual Studio Code 创建代码片段。对于重复的任务，你将学习如何编写 for 循环，如何遍历简单的值，以及如何遍历目录内容。此外，你将学习如何编写嵌套循环。与此同时，你还将学习如何编写 while 循环和 until 循环。接着，我们将进入函数部分，函数是可重用的代码块。你将学习如何编写函数以及如何使用它们。之后，你将接触到 Linux 中最强大的工具之一——流编辑器（Stream Editor）。由于我们仍在讨论文本处理，我们将介绍 AWK，这是 Linux 中最优秀的文本处理工具之一。

之后，你将通过编写更好的正则表达式来提升你的文本处理技能。最后，你将学习 Python，作为 bash 脚本的替代方案。

# 本书适合谁阅读

本书面向系统管理员和开发人员，帮助他们编写更好的 shell 脚本以自动化工作。有编程经验者优先。如果你没有任何 shell 脚本背景也没关系，本书将从头开始讨论所有内容。

# 本书内容

第一章，*使用 Bash 脚本的是什么和为什么*，将介绍 Linux shell，如何编写你的第一个 shell 脚本，如何准备编辑器，如何调试你的 shell 脚本，以及一些基础的 bash 编程，如声明变量、变量作用域和命令替换。

第二章，*创建交互式脚本*，介绍了如何使用`read`命令读取用户输入，如何向脚本传递选项，如何控制输入文本的可见性，以及如何限制输入字符的数量。

第三章，*附加条件*，将介绍`if`语句、`case`语句以及其他测试命令，如`else`和`elif`。

第四章，*创建代码片段*，介绍如何使用编辑器（如 vim 和 Visual Studio Code）创建和使用代码片段。

第五章，*替代语法*，将讨论如何使用`[`进行高级测试，以及如何执行算术运算。

[第六章，*使用循环迭代*，将教你如何使用`for`循环、`while`循环和`until`循环来遍历简单值和复杂值。

第七章，*通过函数创建构建模块*，将介绍函数，并解释如何创建函数、列出内建函数、将参数传递给函数，以及编写递归函数。

第八章，*介绍流编辑器*，将介绍 sed 工具的基础知识，如何操作文件，如添加、替换、删除和转换文本。

第九章，*自动化 Apache 虚拟主机*，包含一个 sed 的实用示例，并解释如何使用 sed 自动创建虚拟主机。

第十章，*AWK 基础*，将讨论 AWK 及如何使用它来过滤文件内容。同时，我们也将讨论一些 AWK 编程基础。

第十一章，*正则表达式*，介绍正则表达式、它们的引擎以及如何与 sed 和 AWK 结合使用，以增强你的脚本。

第十二章，*使用 AWK 总结日志*，将展示如何使用 AWK 处理`httpd.conf` Apache 日志文件，并提取有用的、格式化良好的数据。

第十三章，*使用 AWK 改进 lastlog*，将向你展示如何使用 AWK 通过过滤和处理 lastlog 输出，利用 lastlog 命令生成漂亮的报告。

第十四章，*使用 Python 作为 Bash 脚本替代*，将讨论 Python 编程语言基础，并解释如何编写一些 Python 脚本作为 Bash 脚本的替代。

# 为了最大限度地从本书中获益

我假设你有一些编程背景。即使没有编程背景，本书也会从基础开始。

你应该了解一些 Linux 基础知识，如基本命令`ls`、`cd`和`which`。

# 下载示例代码文件

你可以从你的账户在 [www.packtpub.com](http://www.packtpub.com) 下载本书的示例代码文件。如果你是在其他地方购买本书，你可以访问 [www.packtpub.com/support](http://www.packtpub.com/support) 并注册，直接将文件通过邮件发送给你。

你可以通过以下步骤下载代码文件：

1.  登录或注册 [www.packtpub.com](http://www.packtpub.com/support)。

1.  选择“支持”标签。

1.  点击“代码下载和勘误”。

1.  在搜索框中输入书名，并按照屏幕上的指示操作。

下载文件后，请确保使用以下最新版本的解压工具解压或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

本书的代码包也托管在 GitHub 上，地址为 [`github.com/PacktPublishing/Mastering-Linux-Shell-Scripting-Second-Edition`](https://github.com/PacktPublishing/Mastering-Linux-Shell-Scripting-Second-Edition)。如果代码有更新，它将会在现有的 GitHub 仓库中更新。

我们的丰富书籍和视频目录中也提供了其他代码包，您可以访问 **[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**。快去看看吧！

# 下载彩色图像

我们还提供了一个 PDF 文件，里面有本书中使用的截图/图表的彩色图像。您可以从 [`www.packtpub.com/sites/default/files/downloads/MasteringLinuxShellScriptingSecondEdition_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/MasteringLinuxShellScriptingSecondEdition_ColorImages.pdf) 下载。

# 使用的约定

本书中使用了多种文本约定。

`CodeInText`：表示文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 账号。示例：“编辑您的脚本，使其像下面的完整代码块一样运行，示例：`$HOME/bin/hello2.sh`”

一段代码如下所示：

```
if [ $file_compression = "L" ] ; then 
tar_opt=$tar_l 
elif [ $file_compression = "M" ]; then 
tar_opt=$tar_m 
else 
tar_opt=$tar_h 
fi 
```

任何命令行输入或输出如下所示：

```
$ type ls
ls is aliased to 'ls --color=auto'  
```

**粗体**：表示一个新术语、一个重要的词汇或您在屏幕上看到的词汇。例如，菜单或对话框中的词汇以这种方式出现在文本中。示例：“另一个非常有用的功能可以在 'Preferences | Plugins' 标签页中找到”

警告或重要提示通常以这种形式出现。

小贴士和技巧通常以这种形式出现。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：请发送电子邮件至`feedback@packtpub.com`，并在邮件主题中提及书名。如果您有关于本书的任何问题，请通过`questions@packtpub.com`联系我们。

**勘误**：尽管我们已尽最大努力确保内容的准确性，但错误还是可能发生。如果您在本书中发现错误，我们将非常感激您向我们报告。请访问 [www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击 “Errata Submission Form” 链接并输入详细信息。

**盗版**：如果您在互联网上发现我们作品的任何非法复制版本，我们将非常感激您提供该位置地址或网站名称。请通过`copyright@packtpub.com`联系我们，并附上该材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域拥有专业知识，并且有兴趣撰写或为书籍贡献内容，请访问 [authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。在阅读并使用本书后，为什么不在您购买书籍的网站上留下评论呢？潜在的读者可以看到并利用您的公正意见做出购买决策，我们在 Packt 也能了解您对我们产品的看法，而我们的作者也可以看到您对他们书籍的反馈。谢谢！

如需了解更多关于 Packt 的信息，请访问 [packtpub.com](https://www.packtpub.com/)。
