# 像打字机和文件资源管理器一样操作

在本章中，我们将介绍以下内容：

+   基本的字符串和文件搜索

+   使用通配符和正则表达式

+   脚本中的数学和计算

+   仅使用 Bash 对字符串进行剥离/修改/排序/删除/搜索

+   使用 SED 和 AWK 删除/替换子字符串

+   使用 echo 和 printf 格式化数据/输出

+   为不同语言准备脚本的国际化

+   根据文件内容计算统计信息并减少重复项

+   使用文件属性与条件逻辑

+   读取定界数据并修改输出格式

# 介绍

希望前面的 Bash 入门课程章节已经提供了有关 Bash 工具和功能的提示。另一方面，本章介绍了几种*附加*技术，使得在搜索项目和文本或自动化文件资源管理器/文件系统操作时，Bash 的功能更加广泛。

单独来看，Bash 只是一个强大的脚本语言，但 Bash 的灵活性在于能够“粘合”其他技术（工具或语言），使得输出更加有用。换句话说，Bash 是一个基础平台，就像一些汽车爱好者在进行改装之前选择一个特定的平台。经过改装的车能做所有事情吗？当然不能，但它可以在特定情况下变得更强大或更有用，至少为移动提供了四个轮子。

常见的脚本不仅包含一系列自动化命令，通常还包括**逻辑**来修改字符串，例如以下内容：

+   删除尾部字符

+   替换单词的部分（子字符串）

+   在文件中搜索字符串

+   查找文件

+   测试文件类型（目录、文件、空文件等）

+   执行简单计算

+   限制搜索或数据的范围（过滤）

+   修改变量的内容（字符串变量中的字符串）

这种修改、限制甚至替换输入/输出数据的逻辑，在需要执行广泛搜索特定字符串或处理大量数据时，可以非常强大。终端可能会阻塞；充满输出或巨大的数据文件可能会让人感到探索起来非常艰巨！

然而，还有一个非常重要的概念仍然需要讨论，那就是**递归**功能。递归功能可以应用于脚本函数、逻辑，甚至命令操作。例如，你可以使用**grep**来**递归**地遍历整个目录，直到没有更多文件，或者可以递归地执行一个函数，直到满足某个条件（例如，在一个字符串中一次打印一个字符）：

```
# e.g. File system
# / (start here)
# /home (oh we found home)
# /home/user (neat there is a directory inside it called user)
# /home/user/.. (even better, user has files - lets look in them too)
# /etc/ # We are done with /home and its "children" so lets look in /etc
# ... # Until we are done
```

小心递归（特别是函数），因为它有时可能会很慢，取决于结构的复杂性（例如，文件系统或文件大小）。如果存在逻辑错误，你可能会让函数永远递归执行下去！

本章的内容将集中在限制数据、利用数据、修改数据、国际化数据、替换数据，甚至是搜索数据。

# 基本的字符串和文件搜索

想象一下，在一个大花园里寻找四叶草。这将是非常困难的（对于计算机来说，这依然非常困难）。幸运的是，文字不是图像，计算机上的文本可以根据格式轻松搜索。之所以使用**格式**这个术语，是因为如果你的工具无法理解某种类型的文本（**编码**），那么你可能会遇到无法识别**模式**，甚至无法检测到文本的情况！

通常，当你查看控制台、文本文件、源代码（C、C++、Bash、HTML）、电子表格、XML 等类型时，你看到的是**ASCII**或**UTF**格式。ASCII 是`*NIX`世界中在控制台上常用的格式。还有 UTF**编码方案**，它是对 ASCII 的改进，支持计算机最初没有的各种扩展字符。它有多种格式，如 UTF-8、UTF-16 和 UTF-32。

当你听到编码和解码这两个词时，它类似于加密和解密。其目的不是隐藏某些内容，而是将某些数据转化为适合特定用途的形式。例如，用于传输、语言使用和压缩。

ASCII 和 UTF 并不是你目标数据可能采用的唯一格式。在各种类型的文件中，你可能会遇到不同类型的数据编码。这是一个与数据相关的不同问题，需要额外的考虑。

在本示例中，我们将开始搜索字符串的过程，并介绍几种在大量数据堆中搜索你自己目标的方法。让我们深入探讨。

# 准备工作

除了打开一个终端（如果需要，还可以打开你最喜欢的文本编辑器），我们只需要几个核心命令，如`grep`、`ls`、`mkdir`、`touch`、`traceroute`、`strings`、`wget`、`xargs`和`find`。

假设你的用户已经拥有正确的使用权限（当然也需要授权），我们将需要生成数据来开始搜索：

```
$ ~/
$ wget --recursive --no-parent https://www.packtpub.com www.packtpub.com # Takes awhile
$ traceroute packtpub.com > traceroute.txt
$ mkdir -p www.packtpub.com/filedir www.packtpub.com/emptydir
$ touch www.packtpub.com/filedir/empty.txt
$ touch www.packtpub.com/findme.xml; echo "<xml>" www.packtpub.com/findme.xml
```

# 如何操作...

通过递归**爬取**Packt Publishing 网站获得的数据，我们可以看到，在**www.packtpub.com**上，整个网站是可以访问的。哇！我们还创建了一些测试数据目录和文件。

1.  接下来，打开一个终端并创建以下脚本：

```
#!/bin/bash

# Let's find all the files with the string "Packt"
DIRECTORY="www.packtpub.com/"
SEARCH_TERM="Packt"

# Can we use grep?
grep "${SEARCH_TERM}" ~/* > result1.txt 2&> /dev/null

# Recursive check
grep -r "${SEARCH_TERM}" "${DIRECTORY}" > result2.txt

# What if we want to check for multiple terms?
grep -r -e "${SEARCH_TERM}" -e "Publishing" "${DIRECTORY}" > result3.txt

# What about find?
find "${DIRECTORY}" -type f -print | xargs grep "${SEARCH_TERM}" > result4.txt

# What about find and looking for the string inside of a specific type of content?
find "${DIRECTORY}" -type f -name "*.xml" ! -name "*.css" -print | xargs grep "${SEARCH_TERM}" > result5.txt

# Can this also be achieved with wildcards and subshell?
grep "${SEARCH_TERM}" $(ls -R "${DIRECTORY}"*.{html,txt}) > result6.txt
RES=$?

if [ ${RES} -eq 0 ]; then
  echo "We found results!"
else
  echo "It broke - it shouldn't happen (Packt is everywhere)!"
fi

# Or for bonus points - a personal favorite
history | grep "ls" # This is really handy to find commands you ran yesterday!

# Aaaannnd the lesson is:
echo "We can do a lot with grep!"
exit 0
```

注意脚本中使用的`~/* ?`。这指的是我们的主目录，并引入了`*`通配符，允许我们指定从此位置开始的任何内容。本章后面会详细介绍通配符和正则表达式的概念。

1.  如果你保持在主目录（`~/`）并运行该脚本，输出应该类似于以下内容：

```
$ bash search.sh; ls -lah result*.txt
We found results!
We can do a lot with grep!
-rw-rw-r-- 1 rbrash rbrash 0 Nov 14 14:33 result1.txt
-rw-rw-r-- 1 rbrash rbrash 1.2M Nov 14 14:33 result2.txt
-rw-rw-r-- 1 rbrash rbrash 1.2M Nov 14 14:33 result3.txt
-rw-rw-r-- 1 rbrash rbrash 1.2M Nov 14 14:33 result4.txt
-rw-rw-r-- 1 rbrash rbrash 33 Nov 14 14:33 result5.txt
-rw-rw-r-- 1 rbrash rbrash 14K Nov 14 14:33 result6.txt
```

# 它是如何工作的...

本节有点*难懂*，因为我们要引入一个更广泛的话题——如何在字符串中使用正则表达式和通配符。我们已经介绍了它们，但也向你展示了，即使不使用它们，你也可以专门搜索诸如`${SEARCH_TERM}`或*Packt*之类的术语——这只是需要更多的工作和更多的语句。你能想象为每个术语（如`Packt1`、`Packt2`、`Packt3`等）写一个特定的`grep`语句吗？那可不有趣。

使用 Packt Publishing 网站作为*基准*数据集，我们通过`grep`命令在目录中搜索，目标仅限于我们的当前位置——用户的主目录。**Grep**是一个强大的工具，可以用来解析命令和文件的输出，使用模式、正则表达式和用户提供的参数。在这个例子中，我们并未预期会找到匹配*Packt*的字符串，因为**www.packtpub.com**和**www.Packtpub.com**并不相同。因此，`result1.txt`是一个空文件。

`grep`和许多其他工具都可以区分大小写。要使用不区分大小写的`grep`，请使用`-i`标志。

在第二次使用`grep`时，我们使用了递归标志（`-r`）并找到了许多匹配项。默认情况下，`grep`会返回匹配项的路径（包括文件名）和匹配所在的行。如果你想找到行号，也可以使用标志（`-n`）。

在第三个示例中，我们演示了如何使用多个用户提供的参数来运行`grep`：

```
$ grep -e "Packt" -e "Publishing" -r ~/www.packtpub.com/
```

在这个示例中，我们正在使用*暴力破解*机制进行搜索，这意味着我们将完全依靠我们的力量找到所有内容。当对大量数据进行搜索时，甚至是在`PacktPublishing`网站上执行像搜索这样看似简单的操作时，更先进和更有针对性的算法能更高效、快速地帮你找到你想要的内容，而不是像我们现在这样做！

在第四个和第五个执行示例中，我们使用了`find`命令。我们还将其与管道和**`xargs`**命令结合使用。单独使用`find`是一个非常强大的 CLI 工具，可以用来执行搜索功能（因此，如果使用不当或恶意使用，也可能造成损害）：

```
$ find "${DIRECTORY}" -type f -print | xargs grep "${SEARCH_TERM}" > result4.txt
```

在前面的`find`命令中，我们使用了`-type f`**，**这意味着我们仅在`${DIRECTORY}`中查找文件。然后，我们将结果通过管道传输到`xargs`命令，并与 grep 一起使用。等等！什么是 xargs！？`Xargs`是一个常用于与管道配合使用的命令，用于将换行符（回车）数据传递给另一个命令。例如，如果我们运行`ls -l`（带长格式标志），结果将像这样返回（我们添加了不可见的换行符或`\n`来说明）：

```
$ ls -l
drwxr-xr-x 7 rbrash rbrash 4096 Nov 13 21:48 Desktop\n
drwxr-xr-x 2 rbrash rbrash 4096 Feb 11 2017 Documents\n
drwxr-xr-x 7 rbrash rbrash 32768 Nov 14 10:54 Downloads\n
-rw-r--r-- 1 rbrash rbrash 8980 Feb 11 2017 examples.desktop\n
...
```

如果我们将结果直接通过管道传递给另一个期望输入的命令，就会出错！

```
$ someProgram Desktop\n Documents\n Downloads\n ...
```

相反，`someProgram`要求输入值用空格分隔，而*不是*换行符：

```
$ someProgram Desktop Documents Downloads ...
```

这就是你使用`xargs`的原因：去除或转换换行符，避免出现问题。

回到第二个 `find` 命令的例子，你可以看到我们使用了 `-name` 和 `! -name` 参数。`-name` 很简单；我们在寻找一个具有特定用户提供名称的文件。在第二个 `! -name` 实例中，`!` 表示没有或 *不* 包含这个名称。这就是所谓的 **反向逻辑**。

我们还在与 `grep` 的第一次示例中使用了 `*` 通配符，但这次在不同的上下文中使用它（稍后会进一步讨论）。这次，我们用 `*` 来匹配文件扩展名前的任何内容（`*.xml` 或 `*.css`）。它甚至可以这样使用：

```
$ ls -l /etc/*/*.txt
-rw-r--r-- 1 root root 17394 Nov 10 2015 /etc/X11/rgb.txt
```

在以下 `grep` 命令中，我们使用内联子 Shell 执行 `ls` 命令，配合通配符。然后，我们通过将 `${RES}` 设置为 `$?` 来获取结果。`$?` 是一个特殊变量，用来获取返回码。通过 `${RES}` 中的值，我们现在可以在找到结果时提供一些条件逻辑，并适当地 `echo`：

```
We found results!
```

就在我们退出 Shell 之前，我们想给大家一个额外的提示：你可以使用 `history` 命令和 `grep` 来搜索你过去执行过的命令。

# 使用通配符和正则表达式

如我们在上一节中所看到的，出现了递归函数的新概念和通配符的引入。本节将基于这些基本的原语，通过使用正则表达式和通配符扩展更高级的搜索方法。

本节还将通过一系列内置的 Bash 特性以及一些单行命令（巧妙的小技巧）来增强我们的搜索。简而言之：

+   一个通配符可以是：`*`，`{*.ooh,*.ahh}`，`/home/*/path/*.txt`，`[0-10]`，`[!a]`，`?`，`[a,p] m`

+   一个正则表达式可以是：`$`，`^`，`*`，`[]`，`[!]`，`|`（使用时要小心转义此符号）

**通配符匹配** 基本上指的是一个计算机术语，可以用通俗的语言简单描述为 **扩展模式匹配**。通配符是用来描述模式的 **符号**，而 **正则表达式** 是 **regular expression** 的简称，表示用来描述要匹配数据序列的模式。

Bash 中的通配符匹配非常强大，但可能不是执行更复杂或精细模式匹配的最佳场所。在这些情况下，Python 或其他语言/工具可能更合适。

正如我们可以想象的，通配符和模式匹配非常有用，但并不是所有工具或应用程序都能使用它们。不过，通常它们可以在命令行中与诸如 `grep` 等工具一起使用。例如：

```
$ ls -l | grep '[[:lower:]][[:digit:]]' # Notice no result
$ touch z0.test
$ touch a1.test
$ touch A2.test
$ ls -l | grep '[[:lower:]][[:digit:]]'
-rw-rw-r-- 1 rbrash rbrash 0 Nov 15 11:31 z0.test
-rw-rw-r-- 1 rbrash rbrash 0 Nov 15 11:31 a1.test
```

使用 `ls` 命令并将其通过管道传递给带有正则表达式的 `grep`，我们可以看到在使用 `touch` 创建了三个文件并重新运行命令后，正则表达式使我们能够正确地过滤出以小写字母开头并紧接着一个数字的文件。

如果我们想进一步增强 `grep`（或其他命令），我们可以使用以下任意一种方法：

+   `[:alpha:]`：字母字符（不区分大小写）

+   `[:lower:]`：小写可打印字符

+   `[:upper:]`：大写可打印字符

+   `[:digit:]`：十进制数字 0 到 9

+   `[:alnum:]`：字母数字字符（所有数字和字母字符）

+   `[:space:]`：空白字符，表示空格、制表符和换行符

+   `[:graph:]`：可打印字符，不包括空格

+   `[:print:]`：可打印字符，包括空格

+   `[:punct:]`：标点符号（例如，句号）

+   `[:cntrl:]`：控制字符（如使用*Ctrl* + *C*时生成的不可打印字符）

+   `[:xdigit:]`：十六进制字符

# 准备开始

除了打开终端（并且如果需要的话，还可以打开你喜欢的文本编辑器），我们只需要几个核心命令：`grep`、`tr`、`cut`、**和`touch`。我们假设在前一步中我们爬取的`www.packtpub.com`目录仍然可用：

```
$ cd ~/
$ touch {a..c}.test
$ touch {A..C}[0..2].test2
$ touch Z9.test3 Z9\,test2 Z9..test2
$ touch ~/Desktop/Test.pdf
```

# 如何操作……

让我们开始吧：

1.  打开终端，并选择一个你喜欢的编辑器来创建一个新的脚本。

1.  在你的脚本中，添加以下内容：

```
#!/bin/bash
STR1='123 is a number, ABC is alphabetic & aBC123 is alphanumeric.'

echo "-------------------------------------------------"
# Want to find all of the files beginning with an uppercase character and end with .pdf?
ls * | grep [[:upper:]]*.pdf

echo "-------------------------------------------------"
# Just all of the directories in your current directory?
ls -l [[:upper:]]*

echo "-------------------------------------------------"
# How about all of the files we created with an expansion using the { } brackets?
ls [:lower:].test .

echo "-------------------------------------------------"
# Files with a specific extension OR two?
echo ${STR1} > test.txt
ls *.{test,txt} 

echo "-------------------------------------------------"
# How about looking for specific punctuation and output on the same line
echo "${STR1}" | grep -o [[:punct:]] | xargs echo

echo "-------------------------------------------------"
# How about using groups and single character wildcards (only 5 results)
ls | grep -E "([[:upper:]])([[:digit:]])?.test?" | tail -n 5

exit 0
```

1.  现在，执行脚本，你的控制台应该会被输出结果淹没。最重要的是，让我们看看最后五个结果。注意到结果中有`Z9(,)`和`Z9.test(3)`吗？这就是正则表达式的强大作用！好了，我们已经知道可以使用变量创建并搜索一堆文件夹或文件了，但是我能否使用正则表达式来查找像变量参数这样的东西呢？当然可以！请看下一步。

1.  在控制台中，尝试以下命令：

```
$ grep -oP 'name="\K.*?(?=")' www.packtpub.com/index.html
```

1.  再次在控制台中，尝试以下命令：

```
$ grep -P 'name=' www.packtpub.com/index.html
```

1.  在查找可能跨越多行的 IF 实例时，使用`tr`等命令来删除换行符，我们能做得更好吗？

```
$ tr '\n' ' ' < www.packtpub.com/index.html | grep -o '<title>.*</title>' 
```

1.  现在，让我们使用`cut`来清理屏幕上的一些杂乱信息，作为收尾。通常，控制台的宽度是`80`个字符，因此让我们添加行号并修剪`grep`的输出：

```
$ grep -nP 'name=' www.packtpub.com/index.html | cut -c -80
```

整本书都在讲解如何使用正则表达式解析数据，但关键点是，正则表达式并不总是最适合的选择，特别是在性能和像 HTML 这样的标记语言中。例如，在解析 HTML 时，最好使用一个能够理解该语言及其特定语法的解析器。

# 它是如何工作的……

正如你可能已经猜到的，没有正则表达式和通配符的情况下，通过大量数据进行查询，对于没有经验的人来说可能是一场噩梦。更可怕的情况可能发生在你的表达式没有使用正确的术语或有效（且准确）表达式的情况下。然而，通配符在命令行中非常有用，尤其是在你需要**拼接**字符串、快速查找数据和寻找文件时。有时候，如果我仅仅是想找到特定发生位置的文件名和大致位置/行号，搜索结果的可用性可能并不重要。例如，这个 CSS 类在哪个文件里？

好吧，你已经通过脚本并执行了多个命令，了解了如何在表面上使用正则表达式和通配符。让我们回过头来，走一遍这个过程。

在第 1 步中，我们打开了控制台，创建了一个简单的脚本并执行。然后，输出结果显示在控制台上：

```
$ bash test.sh 
-------------------------------------------------
Linux-Journal-2017-08.pdf
Linux-Journal-2017-09.pdf
Linux-Journal-2017-10.pdf
Test.pdf
-------------------------------------------------
-rw-rw-r-- 1 rbrash rbrash 0 Nov 15 22:13 A0.test2
-rw-rw-r-- 1 rbrash rbrash 0 Nov 15 22:13 A1.test2
-rw-rw-r-- 1 rbrash rbrash 0 Nov 15 22:13 A2.test2
-rw-rw-r-- 1 rbrash rbrash 0 Nov 15 22:13 B0.test2
-rw-rw-r-- 1 rbrash rbrash 0 Nov 15 22:13 B1.test2
-rw-rw-r-- 1 rbrash rbrash 0 Nov 15 22:13 B2.test2
-rw-rw-r-- 1 rbrash rbrash 0 Nov 15 22:13 C0.test2
-rw-rw-r-- 1 rbrash rbrash 0 Nov 15 22:13 C1.test2
-rw-rw-r-- 1 rbrash rbrash 0 Nov 15 22:13 C2.test2
-rw-rw-r-- 1 rbrash rbrash 0 Nov 15 22:13 Z9,test2
-rw-rw-r-- 1 rbrash rbrash 0 Nov 15 22:13 Z9..test2
-rw-rw-r-- 1 rbrash rbrash 0 Nov 15 22:13 Z9.test3

Desktop:
total 20428
drwxrwxr-x 2 rbrash rbrash 4096 Nov 15 12:55 book
# Lots of files here too

Documents:
total 0

Downloads:
total 552776
-rw------- 1 root root 1024 Feb 11 2017 ~
... # I have a lot of files for this book

Music:
total 0

Pictures:
total 2056
drwxrwxr-x 2 rbrash rbrash 4096 Sep 6 21:56 backgrounds

Public:
total 0

Templates:
total 0

Videos:
total 4
drwxrwxr-x 13 rbrash rbrash 4096 Aug 11 10:42 movies
-------------------------------------------------
a.test b.test c.test
-------------------------------------------------
a.test b.test c.test test.txt
-------------------------------------------------
, & .
-------------------------------------------------
C0.test2
C1.test2
C2.test2
Z9,test2
Z9.test3
```

这可能会更可怕！对吧？在第一行中，我们开始查找以大写字母开头的 PDF 文件。命令`ls * | grep [[:upper:]]*.pdf`使用了带有`*`通配符（表示所有文件）的`ls`命令，然后将输出传递给`grep`，并使用一个简单的正则表达式。这个正则表达式是`[[:upper:]]`，后面跟着另一个**`*`**通配符和`.pdf`字符串的组合。这将返回我们的搜索结果，最少会包含`Test.pdf`（我的结果也返回了一个流行的 Linux 期刊的 PDF 文件）。

然后，我们几乎进行相同的搜索，使用`ls -l [[:upper:]]*`，但使用带正则表达式的`ls`命令会返回大量数据（如果所有文件夹都有内容的话）。它从脚本所在的当前目录开始，然后深入到一个目录并打印内容。一个很好的特性是使用`-l`标志，它会产生*详细*的结果并打印目录的大小（以字节为单位）。

接下来，我们使用`ls`命令查找所有以小写字母开头并以`.test`扩展名结尾的文件。你可能没注意到，当你设置这个命令时，你也看到了通配符和扩展的实际应用：`touch {a..c}.test`。`touch`命令创建了三个文件：`a.test`、`b.test`和`c.test`。带有这个简单正则表达式的`ls`命令返回了这三个文件的名称。

再次，我们使用带有（`*`）通配符和扩展括号的`ls`命令来匹配文件扩展名：`ls *.{test,txt}`。它会查找任意名称（`*`）的文件，这些文件会与一个句点（`.`）连接，后面跟着`test`或`txt`扩展名。

接下来，在第 7 步中，我们结合了通过管道、`grep`、`xargs`和正则表达式学到的一些内容，命令是：`echo "${STR1}" | grep -o [[:punct:]] | xargs echo`。由于`grep`的输出是以`\n`（换行符）分隔的形式（每个找到的实例一个新行），这会打破我们希望将所有值按这种形式回显到控制台的意图，因此我们需要`xargs`来修复输出，以便`echo`可以正确使用这些参数。例如，`echo "item1\n item2\n item3\n"`是无法工作的，但使用`xargs`后，它看起来像这样：`echo "item1"  "item2" "item3"`。

在最后的命令中，我们终于达到了一个*更疯狂*的正则表达式，实际上它其实相当简单：`ls | grep -E "([[:upper:]])([[:digit:]])?.test?." | tail -n 5`。它引入了一些概念，包括组（括号）、（`?`）通配符，以及如何组合多个表达式组件，并使用`tail`命令。

使用`grep`、`-E`（表达式标志）和两个组（括号内的表达式），我们可以将它们与`?`正则操作符组合使用。这个操作符是用于表示单个字符的通配符：

```
C0.test2
C1.test2
C2.test2
Z9,test2
Z9.test3
```

我们可以看到，最后五个结果被返回，且第一个字母是大写，后跟一个数字，一个字符（可能是`.`或`,`），然后是“test”这个词和一个数字。我们创建了一个名为`Z9..test2`的测试文件。注意它没有包含在列表项中吗？这是因为我们没有使用这样的表达式：

```
$ ls | grep -E "([[:upper:]])([[:digit:]])?.?.test?"
```

在第 4 步中，我们使用`grep`和`-oP`标志运行了一个特定的正则表达式，`grep -oP 'name="\K.*?(?=")' www.packtpub.com/index.html`，对我们最近抓取的`www.packtpub.com`的档案进行了处理。`-o`标志表示仅输出匹配的值，`-P`用于使用 Perl 正则表达式。

注意那些被双引号包围的值吗？它是在寻找与模式`name="anythingGoesHere"`匹配的*任何*内容。单独来看，这并不特别有用，但它说明了快速提取值的一个方法（例如，如果`name`非常具体，你可以把`name=`改成其他值，得到完全相同的结果！）。

在相同的上下文中，在第 5 步中，我们也可以找到所有`name=`的出现：`grep -P 'name=' www.packtpub.com/index.html`。这种类型的命令对于理解信息的上下文或仅仅了解其存在非常有用；这回到我们寻找 CSS、C/C++以及其他数据/源文件中的值的概念。

进入第 6 步，我们需要寻找标题的 HTML 标签。通常情况下，你应该使用专门的 HTML 解析器，但如果我们想快速用 grep 结合正则表达式来处理——是完全可以的！`tr '\n' ' ' < www.packtpub.com/index.html | grep -o '<title>.*</title>'` 命令使用了转换函数(`tr`)，将`\n`或换行符特殊字符转换为空格。这在数据标记可能跨越多行时非常有用。

在最后的步骤中，我们将进行一些细化操作，以执行广泛的搜索。我们只需使用`grep`来提供行号和文件名。使用`cut`，我们可以修剪控制台输出中的剩余字符（这非常有用）：

```
$ grep -nHP 'name=' www.packtpub.com/index.html | cut -c -80
```

正则表达式也可以通过许多在线正则模拟器来测试！一个常见且免费的在线工具是：[`regexr.com/`](https://regexr.com/)。

别忘了，某些正则表达式功能也允许你在组内嵌套命令！我们没有演示这一功能，但它在某些使用场景中是有效的，并且结果是可以接受的！

# 脚本中的数学和计算

在经历了繁琐的通配符和正则表达式介绍后，我们将进入在控制台上执行一些基本数学运算的部分。如果你还没尝试过，运行类似以下命令会发生什么呢？它看起来是这样的：

```
$ 1*5
1*5: command not found
```

命令未找到？当然，我们知道计算机能够进行*数学运算*，但显然 Bash 无法以这种方式解释数学运算。我们必须确保 Bash 能够通过以下方法正确**解释**这些操作：

+   `expr` 命令（过时）

+   **`bc`** 命令

+   POSIX Bash shell 扩展

+   另一个语言/程序来做这些*脏*活

让我们再试一次，但使用 POSIX Bash shell 扩展：

```
$ echo $((1*5))
5
```

我们得到了期望的答案 `5`，但它错在哪里呢？它出错的地方在于使用除法和浮点数，因为 Bash 主要处理整数：

```
$ echo $((1/5))
0
```

没错，`1` 除以 `5` 是 `0`，但缺少了一个余数！这就是为什么我们可能依赖其他方法来进行简单的数学运算。

使用方程和 `math` 在脚本中的众多用途之一是确定文件系统分区的大小。你能想象如果磁盘空间过满会发生什么吗？或者如果某个目录达到预定大小，我们可能想要自动归档它？当然，这只是理论上的，但如果我们让文件系统悄悄地变满，确实可能会发生故障！

以下这个方案是关于确定一个 tarball（及其内容）大小、目标分区剩余空间以及操作是否可以继续或取消的。

# 准备工作

这个方案将考虑一些有趣的因素：

+   Bash 并非无所不能

+   还有其他工具（例如，`bc`）

+   我们可以用其他语言，比如 C，自己创建一个

+   创建一个 tarball

有时，在一些小型嵌入式系统上，Python 可能不可用，但 Bash（或其近亲）和 C 是可用的。这时，能够在没有额外程序（可能不可用）的情况下进行数学运算就显得非常方便！

我们将使用以下命令来确保所有必要的工具已经安装，以便进行此次实验：

```
$ sudo apt-get install -y bc tar
```

现在，我们需要创建一个名为 `archive.tar.gz` 的 tarball：

```
$ dd if=/dev/zero of=empty.bin bs=1k count=10000
$ tar -zcvf archive.tar.gz empty.bin
$ rm empty.bin
```

我们意识到，创建/编译一个不是用 Bash 编写的简单程序的目的是超出了本书的范围，但这确实是一个有用的技能。为了做到这一点，我们需要安装 GCC，它是 GNU 编译器集合的简称。听起来可能很复杂，但我们向你保证，我们已经做了所有的艰苦工作：

```
$ sudo apt-get install -y gcc
```

上面的命令安装了编译器，现在我们需要 C 源代码（以便编译一个*简单*的 C 程序）。打开控制台并通过以下命令获取代码：

```
$ wget https://raw.githubusercontent.com/PacktPublishing/Bash-Cookbook/master/chapter%2002/mhelper.c
```

这段代码也可以在 Github 上找到，链接：[`github.com/PacktPublishing/Bash-Cookbook`](https://github.com/PacktPublishing/Bash-Cookbook)。

要编译这段代码，我们将使用 `gcc` 和 `-lm`（指的是 `libmath`），如下所示：

```
$ gcc -Wall -02 -o mhelper main.c -lmath
```

如果编译器成功完成（应该是的），你将得到一个名为 `mhelper`（或数学助手）的工具二进制文件。我们也可以通过使用 `sudo` 和 `cp` 将它复制到 `/bin` 来将其添加到本地命令列表中：

```
$ sudo cp mhelper /bin; sudo chmod a+x /bin/mhelper;
```

现在，`mhelper` 可以用来进行基本的运算，如除法、乘法、加法和减法：

```
$ mhelper "var1" "-" "var2"
```

`mhelper` 代码并不是为了特别强大且能处理特定边缘情况而设计的，而是为了演示另一个工具的使用。Python 和 numpy 会是很好的替代品！

# 如何操作...

使用 `mhelper` 二进制文件、`bc` 和其他表达式，我们可以开始这道菜谱：

1.  首先，打开终端和编辑器，创建一个新的脚本文件 `mathexp.sh`，并在其中添加以下内容：

```
#!/bin/bash
# Retrieve file system information and remove header
TARBALL="archive.tar.gz"
CURRENT_PART_ALL=$(df --output=size,avail,used /home -B 1M | tail -n1)

# Iterate through and fill array
IFS=' ' read -r -a array <<< $CURRENT_PART_ALL

# Retrieve the size of the contents of the tarball
COMPRESSED_SZ=$(tar tzvf "${TARBALL}" | sed 's/ \+/ /g' | cut -f3 -d' ' | sed '2,$s/^/+ /' | paste -sd' ' | bc)

echo "First inspection - is there enough space?"
if [ ${array[1]} -lt ${COMPRESSED_SZ} ]; then
    echo "There is not enough space to decompress the binary"
    exit 1
else
  echo "Seems we have enough space on first inspection - continuing"
  VAR=$((${array[0]} - ${array[2]}))
  echo "Space left: ${VAR}"
fi

echo "Safety check - do we have at least double the space?"
CHECK=$((${array[1]}*2))
echo "Double - good question? $CHECK"

# Notice the use of the bc command?
RES=$(echo "$(mhelper "${array[1]}" "/" "2")>0" | bc -l)
if [[ "${RES}" == "1" ]]; then
  echo "Sppppppaaaaccee (imagine zombies)"
fi

# We know that this will break! (Bash is driven by integers)
# if [ $(mhelper "${array[2]}" "/" "2") -gt 0 ]; then
  #~ echo "Sppppppaaaaccee (imagine zombies) - syntax error"
# fi
# What if we tried this with Bash and a concept again referring to floats
# e.g., 0.5
# It would break
# if [ $((${array[2]} * 0.5 )) -gt 0 ]; then
  # echo "Sppppppaaaaccee (imagine zombies) - syntax error"
# fi

# Then untar
tar xzvf ${TARBALL}
RES=$?
if [ ${RES} -ne 0 ]; then
  echo "Error decompressing the tarball!"
  exit 1
fi

echo "Decompressing tarball complete!"
exit 0
```

1.  现在，运行脚本应该会产生类似于以下的输出：

```
First inspection - is there enough space?
Seems we have enough space on first inspection - continuing
Space left: 264559
Safety check - do we have at least double the space?
Double - good question ? 378458
Sppppppaaaaccee (imagine zombies)
empty.bin
Decompressing tarball complete!
```

很棒！所以我们可以使用 Bash 大小计算，`bc` 命令和我们的二进制。如果你想计算一个圆的半径（这肯定会得到一个浮动值）或者一个百分比，例如，你将需要注意 Bash 中这个限制。

最后值得注意的是，`expr` 命令仍然存在，但它已经被弃用。建议在新的脚本中使用 `$(( 你的方程式 ))` 作为首选方法。

使用相同的前提条件，利用 `mhelper` 二进制文件或 `$((..))` 脚本，你还可以计算需要变量输出的情况的百分比（例如，不是整数的情况）。例如，在计算基于百分比的屏幕尺寸时，虽然你会期望得到一个整数，你也可以在计算后进行四舍五入。

# 它是如何工作的……

首先，正如这个教程所暗示的那样——我们注意到 Bash shell 不喜欢带有小数点或甚至非整数的十进制数字。等等，数学！？不幸的是，我们无法隐藏所有细节，但在编程中，有几个概念是你应该了解的：

+   有符号和无符号数字

+   浮点数、双精度数和整数

第一个概念相当简单——当前的计算机是二进制的，这意味着它们使用零（0）和一（1）进行计算。这意味着它们的运算是基于 2^ 的幂次进行的。在不深入基础计算机科学课程的情况下，如果你看到一个数据类型是 int（整数），并且它是一个 32 位数字，那么如果它从 `0` 开始，最大值是十进制下的 `4,294,967,295`（2³²）。这做出了一个关键假设，那就是所有数字（包括 0）都是正数。这个正负特性叫做 **符号**！如果数据类型提到有符号或无符号——现在你知道它是什么意思了！

然而，是否带符号会有一个后果，那就是最大正数或负数的值会减少，因为一个比特用于表示符号。一个有符号的 32 位 int（也可以称为 `int32`）现在的范围是 `(-)2,147,483,647` 到 `(+)2,147,483,647`。

作为作者的说明，我意识到一些计算机科学的定义并不完全符合计算机科学的标准，意思是我稍微调整了它们的含义，以确保在 *大多数* 一般情况下关键点能清晰传达。

另一方面，Bash 只使用整数，可能你已经看到，当你将一个值像 `1/5` 进行除法时，结果是 `0`。没错，它无法整除，但结果是 `0.20`（作为一个小数）。我们也不能对带有小数点的数字进行乘法运算！因此，我们必须使用其他程序，比如 `bc` 或 `mhelper`。

如果你对计算机有兴趣，你也知道有浮点数、双精度数和其他数据类型来表示数字。`Mhelper`和`bc`可以帮助你处理这些类型的数字，当整数的概念无法使用时（例如，除法得到的数字不是整数时）。

1.  回到步骤 1：

    +   我们创建了一个脚本，它将检查`/home`目录，使用`df`命令确定有多少可用空间。通过使用`tail`，另一个可以用来减少输出的命令，我们跳过了输出的第一行，并将所有输出通过管道传入`$CURRENT_PART_ALL`变量（或所有当前分区信息）。

    +   然后，`$CURRENT_PART_ALL`变量的内容通过`read`命令读入一个数组。注意错误重定向的使用：`<<<`。这被称为**her****e-string**，简单来说就是展开变量并将其传递到`` `stdin.` ``。

    +   现在，`/home`分区的存储信息已经在一个数组中，我们有一个`tarball`（或压缩并包含内部内容的文件），我们需要知道`tarball`内部内容的大小。为此，我们使用一个冗长的命令，通过多个管道命令来获取包含元素的大小，并将其传递给`bc`命令。

    +   在确定我们归档文件中包含的元素大小后，我们将计算出的大小与剩余可用空间进行验证。这个值位于`array element[1]`中。如果可用空间小于等于提取的文件，则退出。否则，执行后打印剩余空间。

    +   为了好玩，我们结合了分叉子壳程序以获取`mhelper`的除法结果，这些结果通过`bc`管道传递。这样，我们就能确定是否有足够的空间，并以`boolean`值为真（1）或假（0）表示。

    +   由于我们假设有足够的空间，我们解压（解压并提取内容）`$TARBALL`。如果`tar`命令返回的值不等于`0`，则退出并报错。否则，退出并表示成功。

1.  执行脚本后，tarball（`empty.bin`）的内容应该出现在当前工作目录中。

在脚本中，我们在注释中加入了两个不同的评估，它们会返回浮动值或语法错误。我们加入它们是为了提醒你并帮助强化主要的教学内容。

我们错过了什么吗？当然有！我们从未检查过 tarball 本身的大小，也没有确保它的大小包括在执行检查以确定剩余可用空间时的使用空间中。进行和强制大小限制时，一定要小心！

# 仅使用 Bash 进行字符串的剥离/修改/排序/删除/查找

到目前为止，我们已经看到 Linux 中可用命令的强大之处，其中一些命令是最强大的：`sed` 和 `grep`。然而，尽管我们可以轻松地将这些命令一起使用，*仅使用 sed* 或者使用另一个非常有用的命令 `awk`，我们也可以利用 Bash 本身来节省时间并减少外部依赖，从而实现可移植的解决方案！

那么，我们该怎么做呢？让我们从几个使用 Bash 语法的例子开始：

```
#!/bin/bash
# Index zero of VARIABLE is the char 'M' & is 14 bytes long
VARIABLE="My test string"
# ${VARIABLE:startingPosition:optionalLength}
echo ${VARIABLE:3:4}
```

在前面的例子中，我们可以看到一种特殊的调用子字符串功能的方式，使用 `${...}`，其中 `VARIABLE` 是脚本中的字符串变量（甚至是全局变量），接下来的变量是 `:`。在 `:` 后面是 `startingPosition` 参数（记住字符串就是字符数组，每个字符可以通过索引访问），并且还有一个可选的分号和长度参数（`optionalLength`）。

如果我们运行这个脚本，输出会是：

```
$ bash script.sh
test
```

你可能会问，这怎么可能呢？嗯，这可以通过 Bash 中等效的 `substr`（C 语言和许多其他编程语言中的函数）来实现，具体通过使用 **`${...}`** 语法。这告诉 bash 查找名为 `VARIABLE` 的变量，并获取两个参数：从字节/字符 `3`（技术上是 `4`，因为在 Bash 中数组是从元素 `0` 开始的）开始，长度为 `4`（只打印四个字符）。`echo` 的结果是 `test`。

我们可以做更多吗？比如去除最后一个字符？删除单词？搜索？当然可以，所有这些内容都会在本教程中讲解！

# 准备开始

通过创建一些模拟常见日常问题的数据集来准备练习：

```
$ rm -rf testdata; mkdir -p testdata
$ echo "Bob, Jane, Naz, Sue, Max, Tom$" > testdata/garbage.csv 
$ echo "Zero, Alpha, Beta, Gama, Delta, Foxtrot#" >> testdata/garbage.csv 
$ echo "1000,Bob,Green,Dec,1,1967" > testdata/employees.csv
$ echo "2000,Ron,Brash,Jan,20,1987" >> testdata/employees.csv
$ echo "3000,James,Fairview,Jul,15,1992" >> testdata/employees.csv
```

使用这两个 CSV 文件，我们将：

+   去掉 `garbage.csv` 前两行的多余空格

+   删除 `garbage.csv` 中每行的最后一个字符

+   将 `garbage.csv` 中前两行的每个字符转换为大写

+   将 `employees.csv` 中的 `Bob` 替换为 `Robert`

+   在 `employees.csv` 每行的开头插入一个 `#`

+   删除 `employees.csv` 中每行的精确出生日期列/字段

# 如何做到...

让我们开始我们的活动：

1.  打开一个新的终端，并用你喜欢的编辑器打开一个新文件。将以下内容添加到新脚本中，并将其保存为 `builtin-str.sh`：

```
#!/bin/bash

# Let's play with variable arrays first using Bash's equivalent of substr

STR="1234567890asdfghjkl"

echo "first character ${STR:0:1}"
echo "first three characters ${STR:0:3}"

echo "third character onwards ${STR: 3}"
echo "forth to sixth character ${STR: 3: 3}"

echo "last character ${STR: -1}"

# Next, can we compare the alphabeticalness of strings?

STR2="abc"
STR3="bcd"
STR4="Bcd"

if [[ $STR2 < $STR3 ]]; then
 echo "STR2 is less than STR3"
else
 echo "STR3 is greater than STR2"
fi

# Does case have an effect? Yes, b is less than B
if [[ $STR3 < $STR4 ]]; then
 echo "STR3 is less than STR4"
else
 echo "STR4 is greater than STR3"
fi
```

1.  执行脚本 `bash builtin-str.sh` 并注意我们如何从字符串中剥离最后一个字符，甚至比较字符串。

1.  再次，打开一个名为 `builtin-strng.sh` 的新文件，并将以下内容添加到其中：

```
#!/bin/bash
GB_CSV="testdata/garbage.csv"
EM_CSV="testdata/employees.csv"
# Let's strip the garbage out of the last lines in the CSV called garbage.csv
# Notice the forloop; there is a caveat

set IFS=,
set oldIFS = $IFS
readarray -t ARR < ${GB_CSV}

# How many rows do we have?
ARRY_ELEM=${#ARR[@]}
echo "We have ${ARRY_ELEM} rows in ${GB_CSV}"
```

让我们去除垃圾—去掉空格：

```

INC=0
for i in "${ARR[@]}"for i in "${ARR[@]}"
do
:
res="${i//[^ ]}"
TMP_CNT="${#res}"
while [ ${TMP_CNT} -gt 0 ]; do
i=${i/, /,}
TMP_CNT=$[$TMP_CNT-1]
done
ARR[$INC]=$i
INC=$[$INC+1]
done
```

让我们删除每行的最后一个字符：

```
INC=0
for i in "${ARR[@]}"
do
: 
ARR[$INC]=${i::-1}
INC=$[$INC+1]
done
```

现在，让我们将所有字符都转换为大写！

```
INC=0for i in "${ARR[@]}"
do
:
ARR[$INC]=${i^^}
printf "%s" "${ARR[$INC]}"
INC=$[$INC+1]
echo
done

# In employees.csv update the first field to be prepended with a # character
set IFS=,
set oldIFS = $IFS
readarray -t ARR < ${EM_CSV}

# How many rows do we have?
ARRY_ELEM=${#ARR[@]}

echo;echo "We have ${ARRY_ELEM} rows in ${EM_CSV}"
# Let's add a # at the start of each line
INC=0
for i in "${ARR[@]}"
do
:
ARR[$INC]="#${i}"
printf "%s" "${ARR[$INC]}"
INC=$[$INC+1]
echo
done

# Bob had a name change, he wants to go by the name Robert - replace it!
echo
echo "Let's make Bob, Robert!"
INC=0
for i in "${ARR[@]}"
do
:
# We need to iterate through Bobs first
ARR[$INC]=${i/Bob/Robert}
printf "%s" "${ARR[$INC]}"
INC=$[$INC+1]
echo
done
```

我们将删除 `birth` 列中的日期。要删除的字段是 5（但实际上是 -4）：

```
echo;
echo "Lets remove the column: birthday (1-31)"
INC=0
COLUM_TO_REM=4
for i in "${ARR[@]}"
do
 :
# Prepare to also parse the ARR element into another ARR for
# string manipulation
 TMP_CNT=0
 STR=""
 IFS=',' read -ra ELEM_ARR <<< "$i"
 for field in "${ELEM_ARR[@]}"
 do
   # Notice the multiple argument in an if statement
   # AND that we catch the start of it once
   if [ $TMP_CNT -ne 0 ] && [ $TMP_CNT -ne $COLUM_TO_REM ]; then
   STR="${STR},${field}"
   elif [ $TMP_CNT -eq 0 ]
   then
   STR="${STR}${field}"
   fi 
   TMP_CNT=$[$TMP_CNT+1]
 done
 ARR[$INC]=$STR
 echo "${ARR[$INC]}"
 INC=$[$INC+1]
done
```

1.  执行脚本 `bash builtin-strng.sh` 并查看输出。

你有没有注意到所有重定向输入或输出的机会？想象一下这些可能性！此外，之前的许多脚本可以改用另一个工具 AWK 来执行。

# 它是如何工作的...

这份食谱有些迭代性，但它应该*反复强调*（请原谅这个双关语）来展示 Bash 内置了相当多的功能来操作字符串或任何结构化数据。然而，有一个基本假设，那就是许多操作系统使用 C 语言编写程序。

+   字符串是字符的数组。

+   像`,`这样的字符和其他任何字符一样。

+   因此，我们可以评估或测试某个字符的存在，用于分隔行中的字段，甚至用它来构建数组。

现在，回顾一下这份食谱中的步骤：

1.  运行脚本后，我们在控制台上得到了以下输出：

```
$ builtin-str.sh
first character 1
first three characters 123
third character onwards 4567890asdfghjkl
forth to sixth character 456
last character l
STR2 is less than STR3
STR4 is greater than STR3
```

我们从字符串`STR="1234567890asdfghjkl"`开始，当脚本在第一步运行时：

+   在第一步中，我们从位置零（0）开始打印出一个字符。记住，这是一个数组，位置`0`是起始元素。

+   接下来，我们提取了前三个字符，得到：`123`。

+   然而，如果我们想要获取位置 3 之后的所有字符怎么办？我们可以使用`${STR: 3}`而不是`${STR: 0-3}`。

+   然后，基于之前的内容，如果我们想要获取位置 4 的字符（数组中的第四个元素，但由于从零（0）开始计数，因此这是位置三（3）），我们使用`${STR: 3: 3}`。

+   最后，要仅获取最后一个字符，我们可以使用`${STR:-1}`。

为了完成食谱中的第一个脚本，我们还需要三个字符串。如果我们希望将它们彼此比较，可以使用条件逻辑来实现。记住，bcd 小于 BCD。

使用简单的 Bash 构造来比较字符串非常有用，当你想写一个脚本来快速比较文件名以指定执行顺序时。例如，先运行`001-test.sh`脚本，再运行`002-test.sh`。

1.  在这部分食谱中，我们从一个冗长的脚本开始，以易于解释的方式进行复制。我们涵盖了在不使用 AWK 和 SED 的情况下，你可以在 Bash shell 中使用的一些技巧：

```
$ ./builtin-strng.sh 
We have 2 rows in testdata/garbage.csv
BOB,JANE,NAZ,SUE,MAX,TOM
ZERO,ALPHA,BETA,GAMA,DELTA,FOXTROT

We have 3 rows in testdata/employees.csv
#1000,Bob,Green,Dec,1,1967
#2000,Ron,Brash,Jan,20,1987
#3000,James,Fairview,Jul,15,1992

Let's make Bob, Robert!
#1000,Robert,Green,Dec,1,1967
#2000,Ron,Brash,Jan,20,1987
#3000,James,Fairview,Jul,15,1992

Lets remove the column: birthday (1-31)
#1000,Robert,Green,Dec,1967
#2000,Ron,Brash,Jan,1987
#3000,James,Fairview,Jul,1992
```

这是脚本的详细解析，但在此之前需要简要介绍**数组**、**readarray**、**IFS**和**oldIFS**。这个练习的重点不是深入讲解数组（这将在后面讨论），而是要知道你可以自动使用它们来创建动态列表，例如文件或文件中的行。它们通过`${ARR[@]}`符号引用，且每个元素可以通过其索引值在方括号`[...]`中引用。

`readarray`命令使用`IFS`和`oldIFS`变量将输入解析成数组。它根据一个共同的分隔符（IFS）分隔数据，如果这些值被修改，oldIFS 可以保持原值：

+   在第一步中，我们使用 read 读取`garbage.csv`（`${GB_CSV}`），然后使用`${#ARR[@]}`来获取数组中元素的数量。我们不使用这个值，但值得注意的是文件的结构以及它是否被正确读取。然后，对于数组中的每个成员，我们通过计算空格的数量来移除空格，再通过额外的 while 循环，执行`${i/, /,}`直到完成。修正后的值然后重新插入到数组中。

+   在下一步中，我们使用`${i::-1}`和 for 循环来删除每行的最后一个字符。然后，将结果重新插入到数组中。

+   使用`for`循环和`ARR[$INC]=${i^^}`，数组中的所有字符都被转为大写，我们使用`printf`打印数组（稍后在其他示例中会详细介绍）。

+   接下来是`employees.csv`，我们再次使用`readarray`将其读入数组。然后，我们在每一行的开头添加一个井号（`#`），并将其重新插入到`ARR[$INC]="#${i}"`数组中。

+   然后，我们搜索子字符串`Bob`并用`ARR[$INC]=${i/Bob/Robert}`替换它。要使用内建的查找和替换功能，我们使用以下语法：**`${variable/valueToFind/valueToReplaceWith}`**。请注意，这也是前面步骤中移除空格所用的原理。

+   最后一步稍微复杂一点，*有点冗长*，意味着它可以通过使用其他工具如 AWK 来缩短和执行，但为了让示例易于阅读——它写得有点像 C 程序。这里，我们要删除实际的生日值（0-31），或者说是第 5 列（如果考虑到数组从 0 开始，索引是 4）。首先，我们使用 for 循环遍历数组，然后使用 read 将输入值也作为数组！接着，对于数组`${ELEM_ARR[@]}`中的每个字段，我们检查它是否不是第一个值，也不是我们希望删除的列。我们通过连接构建正确的字符串，然后将其重新插入数组，再使用`echo`打印每个值。

**数组**是一种数据结构，重要的是要考虑数据可以以多种方式进行操作。就像我们如何按行拆分文件以创建元素数组一样，我们也可以将这些元素拆分成自己的数组！

# 使用 SED 和 AWK 来删除/替换子字符串

同样，当我们需要删除某个恼人的字符或删除字符串中的某些部分时，我们总是可以依赖这两个强大的命令：`sed`和`awk`。虽然我们看到 Bash 确实有类似的内建功能，但完整的工具能提供相同的功能甚至更复杂的功能。那么，我们什么时候应该使用这些工具呢？

+   当我们不太关心通过使用 Bash 内建功能可能获得的速度时

+   当需要更复杂的功能时（例如需要多维数组或编辑流等编程结构）

+   当我们专注于可移植性时（Bash 可能被嵌入或是一个有限版本，并且可能需要独立工具）

已经有完整的书籍写了关于 SED 和 AWK，你可以随时在[`www.gnu.org/software/sed/`](https://www.gnu.org/software/sed/)和[`www.gnu.org/software/gawk/`](https://www.gnu.org/software/gawk/)上找到更多信息。

**流编辑器**（**SED**）是一个方便的文本处理工具，非常适合一行命令，并提供一个简单的编程语言和正则表达式匹配。或者，AWK 也是强大的，可以说比 SED 更多。它提供了一个更完整的编程语言，具有各种数据结构和其他构造。但是，当处理可能包含字段或结构化数据的文件（例如 CSV）时，它更适合。然而，当使用管道（例如`grep X | sed ... > file.txt`）时，SED 在处理文本替换时可能更好。

# 准备工作

让我们通过创建一些模仿常见日常问题的数据集来为这个练习做好准备：

```
$ rmdir testdata; mkdir -p testdata
$ echo "Bob, Jane, Naz, Sue, Max, Tom$" > testdata/garbage.csv 
$ echo "Bob, Jane, Naz, Sue, Max, Tom#" >> testdata/garbage.csv 
$ echo "1000,Bob,Green,Dec,1,1967" > testdata/employees.csv
$ echo" 2000,Ron,Brash,Jan,20,1987" >> testdata/employees.csv
$ echo "3000,James,Fairview,Jul,15,1992" >> testdata/employees.csv
```

使用这两个 CSV 文件，我们将要：

+   在`garbage.csv`的前两行中移除额外的空格。

+   从`garbage.csv`中的每行删除最后一个字符。

+   在`garbage.csv`的前两行中将每个字符的大小写更改为大写。

+   在`employees.csv`中用`Robert`替换`Bob`。

+   在`employees.csv`的每行开头插入一个`#`。

+   在每行的`employees.csv`中移除出生日期列/字段的确切日期。

# 如何做...

就像只使用 Bash 练习一样，我们将执行类似的操作如下：

创建一个名为`some-strs.sh`的脚本，并使用以下内容打开一个新的终端：

```
#!/bin/bash
STR="1234567890asdfghjkl"
echo -n "First character "; sed 's/.//2g' <<< $STR # where N = 2 (N +1)
echo -n "First three characters "; sed 's/.//4g' <<< $STR

echo -n "Third character onwards "; sed -r 's/.{3}//' <<< $STR
echo -n "Forth to sixth character "; sed -r 's/.{3}//;s/.//4g' <<< $STR

echo -n "Last character by itself "; sed 's/.*\(.$\)/\1/' <<< $STR
echo -n "Remove last character only "; sed 's/.$//' <<< $STR
```

执行脚本并查看结果。

创建另一个名为`more-strsng.sh`的脚本，然后执行它：

```
#!/bin/sh
GB_CSV="testdata/garbage.csv"
EM_CSV="testdata/employees.csv"
# Let's strip the garbage out of the last lines in the CSV called garbage.csv
# Notice the forloop; there is a caveat

set IFS=,
set oldIFS = $IFS
readarray -t ARR < ${GB_CSV}

# How many rows do we have?
ARRY_ELEM=${#ARR[@]}
echo "We have ${ARRY_ELEM} rows in ${GB_CSV}"

# Let's strip the garbage - remove spaces
INC=0
for i in "${ARR[@]}"
do
   : 
  ARR[$INC]=$(echo $i | sed 's/ //g')
  echo "${ARR[$INC]}"
  INC=$[$INC+1]
done

# Remove the last character and make ALL upper case
INC=0
for i in "${ARR[@]}"
do
   : 
  ARR[$INC]=$(echo $i | sed 's/.$//' | sed -e 's/.*/\U&/' )
  echo "${ARR[$INC]}"
  INC=$[$INC+1]
done

```

我们希望在每行开头添加`#`，并且我们还将在每个文件基础上使用`sed`工具。我们只需删除 Bob 并通过对文件进行操作而将其更改为 Robert：

```
set IFS=,
set oldIFS = $IFS
readarray -t ARR < ${EM_CSV}

INC=0
for i in "${ARR[@]}"
do
 : 
 ARR[$INC]=$(sed -e 's/^/#/' <<< $i )
 echo "${ARR[$INC]}"
 INC=$[$INC+1]
done

sed -i 's/Bob/Robert/' ${EM_CSV}
sed -i 's/^/#/' ${EM_CSV} # In place, instead of on the data in the array
cat ${EM_CSV}
# Now lets remove the birthdate field from the files
# Starts to get more complex, but is done without a loop or using cut
awk 'BEGIN { FS=","; OFS="," } {$5="";gsub(",+",",",$0)}1' OFS=, ${EM_CSV}
```

检查结果——是否更容易获得仅使用 Bash 内置构造的配方的结果？在许多情况下可能是的，如果它们可用的话。

# 工作原理...

在运行本教程中的两个脚本后，我们可以看到一些项目出现（特别是如果我们比较内置的 Bash 功能用于搜索、替换和子字符串）。

1.  在执行`some-strs.sh`后，我们可以在控制台中看到以下输出：

```
$ bash ./some-strs.sh 
First character 1
First three characters 123
Third character onwards 4567890asdfghjkl
Forth to sixth character 456
Last character by itself l
Remove last character only 1234567890asdfghjk
```

到目前为止，我们已经多次看到 `echo` 命令的使用，但 `-n` 标志意味着我们不应自动创建新的一行（或回车）。`<<<` 输入重定向用于将值作为字符串输入，这之前也已经使用过，因此这不会是新信息。基于这一点，在第一次使用 `sed` 时，我们这样写：`sed 's/.//2g' <<< $STR`。与通过正则表达式结合 `sed` 的各种方式相比，这个脚本中的 `sed` 使用非常简单。首先，你有命令（`sed`），然后是参数（`'s/.//2g'`），接着是输入（`<<< $STR`）。你也可以像这样组合参数：`'s/.//2g;s/','/'.'/g'`。要获取第一个字符，我们在替换模式（`s/`）中使用 `sed`，并通过（`/2g`）来获取两个字符，其中 `g` 代表全局模式。

之所以是 `2g` 而不是 `1g`，是因为会自动返回一个空字节，因此，如果你需要 *n* 个字符，那么必须指定 *n+1* 个字符。要返回前三个字符，我们只需要将 `sed` 参数中的 `2g` 改为 `4g`。

在脚本的下一个代码块中，我们使用如下的 `sed`：`sed -r 's/.{3}//'` 和 `sed -r '$s/.{3}//;s/.//4g'`。你可以看到，在第一次执行 `sed` 时，`-r` 用于指定正则表达式，因此我们使用正则表达式返回位置为 4 的字符串（再次提到，那些麻烦的数组和字符串），以及后续的内容。在第二次执行时，我们从第三个字符开始，但限制输出只包含 3 个字符。

在脚本的第三个代码块中，我们想要获取字符串的最后一个字符，使用 `sed 's/.*\(.$\)/\1/'`，然后使用 `sed 's/.$//'` 获取去掉最后一个字符的整个字符串。第一次，我们使用分组和通配符创建正则表达式，只返回一个字符（即字符串中的最后一个字符）；而在第二次，我们使用 `.$` 模式创建一个表达式，返回去掉最后一个字符的所有内容。

需要注意的是，搜索和替换也可以通过指定空值来执行删除操作。你还可以使用 `-i` 标志进行就地编辑，并通过其他标志/参数执行删除。

1.  进入下一个脚本，执行后，控制台应该类似如下所示：

```
$ bash more-strsng.sh 
We have 2 rows in testdata/garbage.csv
Bob,Jane,Naz,Sue,Max,Tom$
Zero,Alpha,Beta,Gama,Delta,Foxtrot#
BOB,JANE,NAZ,SUE,MAX,TOM
ZERO,ALPHA,BETA,GAMA,DELTA,FOXTROT
#1000,Robert,Green,Dec,1,1967
#2000,Ron,Brash,Jan,20,1987
#3000,James,Fairview,Jul,15,1992
#1000,Robert,Green,Dec,1,1967
#2000,Ron,Brash,Jan,20,1987
#3000,James,Fairview,Jul,15,1992
#1000,Robert,Green,Dec,1967
#2000,Ron,Brash,Jan,1987
#3000,James,Fairview,Jul,1992
```

同样，在第一个代码块中，我们将 CSV 读取到数组中，并且对每个元素进行替换操作，去除空格：`sed 's/ //g'`。

在第二个代码块中，我们再次遍历数组，但我们去掉最后一个字符，使用 `sed 's/.$//'`，然后将输出通过管道转换为大写，使用 `sed -e 's/.*/\U&/'`。在管道的第一部分，我们使用 `.$` 搜索最后一个字符并将其移除（`//`）。接着，我们使用表达式选择所有内容，并通过 `\U&` 将其转换为大写（注意，这是 GNU sed 允许的特殊情况）。如果需要小写，则可以使用 `\L&`。

在第三个代码块中，我们再次使用了`for each`循环和子 Shell，但我们没有将输入回显到`sed`。Sed 也可以通过`<<<`输入方向来接受这样的输入。使用`sed -e 's/^/#/'`，我们从字符串的开头（由`^`指定）开始，添加一个`#`。

接下来，对于最后三个示例，我们将在实际文件本身上执行操作，而*不是*内存中加载的数组，方法是使用带有`-i`标志的`sed`。这是一个重要的区别，因为它会对作为输入使用的文件产生直接影响；这很可能是你在脚本中所期望的！要将`Bob`替换为`Robert`，这和删除空格一样，只是我们指定了替换内容。然而，我们是在对*整个*输入的 CSV 文件进行替换！我们还可以为文件中的每一行添加哈希符号。

在最后一个示例中，我们简要使用 AWK 来展示这个工具的强大。在这个示例中，我们指定了分隔符（FS 和 OFS），然后使用 AWK 语言中的`gsub sub`命令指定第五列来删除该列或字段。Begin 指定了 AWK 在解析输入时使用的规则，如果有多个规则，则按照接收到的顺序执行。

或者，我们可以使用`awk 'BEGIN { FS=","} { print $1}' testdata/employees.csv`打印第一列或字段，甚至通过指定`NR==1`来打印第一次出现的内容，像这样：`awk 'BEGIN { FS=","} NR==1{ print $1}'`。指定返回的记录数量在使用`grep`命令时非常有用，尤其是在返回大量匹配结果时。

再次，AWK 和 SED 能做的事情非常多。结合正则表达式（regex），各种用法的解释和示例足以填满一本书，专门讲解每个命令。你可以查看网上文档中的工具，以便了解一些平台差异。

# 使用 echo 和 printf 格式化数据/输出

有时候，找到你要找的字符串或数据是任务中最简单的部分，但格式化输出数据却很棘手。例如，以下是一些需要调整的微妙情况：

+   输出时不加换行符（\n）

+   输出原始十六进制（hex）数据

+   打印原始十六进制值和可打印的 ASCII 字符

+   字符串连接

+   转义特定字符

+   对齐文本

+   打印水平分隔线

除了技巧，我们还可以将值打印到屏幕上，这些值也可以是浮动值（除了数学公式的部分）。等等，什么是十六进制数？是的，存在另一种数据类型，或者至少是一种表示方式。为了理解什么是十六进制数，我们首先需要记住计算机使用 **二进制**，它由 1 和 0（即 1 和 0）组成。然而，二进制对我们人类并不友好（通常我们在查看数字时使用十进制格式），所以有时需要其他表示方式，其中一种就是 **十六进制**。如你所料，它是基数 16，因此看起来像 0x0 到 0xF（0x0，0x1，...，0x9，0xA，0xB，...，0xF）。以下是一个例子：

```
$ echo -en '\xF0\x9F\x92\x80\n'

$ printf '\xF0\x9F\x92\x80\n'

```

在前面的例子中，`printf` 和 `echo` 都可以用来打印原始十六进制和 Unicode 字符。通过一个 Unicode 引用，我找到了 **骷髅** 字符（`F0 9F 92 80`）的 UTF-8 编码，然后使用 `\xFF` 格式化它。注意 FF 的位置，它在每个 **字节** 中。

知道了“原始十六进制”值后，你能做什么呢？嗯，你可以发送 shell 可以解释为不同内容的字符，或者你可以打印出漂亮的东西！查看更多详情请访问 [unicode-table.com](https://unicode-table.com)。

等等，另一个术语叫做 **字节**？是的，还有一个叫做 **比特**。**比特** 是指 0 或 1，而 **字节是 8 个比特**（一个字节由八个比特组成！明白了吗？）。

顺便提一下，根据平台或度量的不同——请注意，1 千字节或 KB 可能意味着 1,024 字节（B），或者在许多市场数据表中，1 KB = 1,000 B。此外，当你看到 Kb 时——它并不意味着千字节。它意味着 **千比特！**

再次提醒，了解计算基础知识，如数据类型和基本数据形式之间的转换，是你技能库中非常有用的工具。这可能会出现在一两次面试中！

然而，我们有点超前了——什么是 `echo` 和 `printf`？这两个命令你可能已经在本书中看到过，它们允许你将变量的内容等输出到控制台或文件中。**Echo** 更“直接”，而 **printf** 则可以使用 C 风格的参数提供相同的功能，甚至更多。事实上，`printf` 相对于 `echo` 的一个主要优势是它可以格式化字符、填充字符，甚至对齐字符。

好的，开始工作吧。

# 准备工作

对于这个练习，不需要额外的工具或脚本——只需你、你的终端和 Bash。

# 如何操作……

让我们开始如下活动：

1.  打开一个新的脚本文件，命名为 `echo-mayhem.sh`，使用你喜欢的编辑器，并打开一个新的终端。在终端中输入以下内容并执行该脚本：

```
#!/bin/bash

# What about echo?
echo -n "Currently we have seen the command \"echo\" used before"
echo " in the previous script"
echo
echo -n "Can we also have \t tabs? \r\n\r\n?"
echo " NO, not yet!"
echo
echo -en "Can we also have \t tabs? \r\n\r\n?"
echo " YES, we can now! enable interpretation of backslash escapes"
echo "We can also have:"
echo -en '\xF0\x9F\x92\x80\n' # We can also use \0NNN for octal instead of \xFF for hexidecimal
echo "Check the man pages for more info ;)"
```

1.  在查看 `echo-mayhem.sh` 的结果后，创建另一个名为 `printf-mayhem` 的脚本，并输入以下内容：

```
#!/bin/bash
export LC_NUMERIC="en_US.UTF-8"
printf "This is the same as echo -e with a new line (\\\n)\n"

DECIMAL=10.0
FLOAT=3.333333
FLOAT2=6.6666 # On purpose two missing values

printf "%s %.2f\n\n" "This is two decimal places: " ${DECIMAL}

printf "shall we align: \n\n %.3f %-.6f\n" ${FLOAT} ${FLOAT2}
printf " %10f %-6f\n" ${FLOAT} ${FLOAT2}
printf " %-10f %-6f\n" ${FLOAT} ${FLOAT2}

# Can we also print other things?
printf '%.0s-' {1..20}; printf "\n"

# How about we print the hex value and a char for each value in a string?
STR="No place like home!"
CNT=$(wc -c <<< $STR})
TMP_CNT=0

printf "Char Hex\n"

while [ ${TMP_CNT} -lt $[${CNT} -1] ]; do
  printf "%-5s 0x%-2X\n" "${STR:$TMP_CNT:1}" "'${STR:$TMP_CNT:1}" 
  TMP_CNT=$[$TMP_CNT+1]
done
```

1.  执行 `printf-mayhem.sh` 的内容，并查看其微妙的差异。

# 它是如何工作的……

虽然这是一个关于数据类型的非常重要的话题（特别是处理数学或计算时），我们将这个问题的解决方案分成了两部分：

1.  在第一步中，echo 相当简单。我们之前提到过有特殊字符和转义符。`\t`表示制表符，`\r\n`表示 Windows 中的换行符（尽管在 Linux 中，`\n\n`就足够了），我们还可以打印出一个漂亮的 UTF 字符：

```
$ bash echo-mayhem.sh 
Currently we have seen the command "echo" used before in the previous script

Can we also have \t tabs? \r\n\r\n? NO, not yet!

Can we also have       tabs? 

? YES, we can now! enable interpretation of backslash escapes
We can also have:

Check the man pages for more info ;)
```

1.  然而，第二步的结果有点不同，我们可以从下面的代码中看到这一点。让我们深入探讨一下，因为这看起来不仅仅是对齐不当的问题：

```
$ bash printf-mayhem.sh 
This is the same as echo -e with a new line (\n)
This is two decimal places: 10.00

shall we align: 

 3.333 6.666600
   3.333333 6.666600
 3.333333 6.666600
--------------------
Char Hex
N 0x4E
o 0x6F
  0x20
p 0x70
l 0x6C
a 0x61
c 0x63
e 0x65
  0x20
l 0x6C
i 0x69
k 0x6B
e 0x65
  0x20
h 0x68
o 0x6F
m 0x6D
e 0x65
! 0x21
  0x0 
```

1.  如我们所见，在前面的步骤执行后，有几个有趣的地方。首先我们注意到的是，`printf`是强化版的 echo，它提供了相同的功能，甚至更多的功能，比如对齐、用`%s`打印字符串，以及小数点（例如`%.2f`）。深入了解后，我们可以看到，使用`%`后跟一个值，我们可以限制小数点的位数。注意，通常在`%`符号后面紧跟的字符——这就是你如何格式化后续参数。使用像`%10f`这样的值时，我们将为该值分配 10 个空格，或者说，分配 10 个字符的宽度。如果我们使用`%-10`，则表示我们将值左对齐。除了几乎水平的规则使用扩展之外，我们还通过“逐步”处理字符串“No place like home!”来进行演示。使用`while`循环，我们打印出每个 ASCII 字符（使用`%-c`）及其对应的十六进制值（`%-2X`）。

注意，即使是空格也有一个十六进制（hex）值，它是`0x20`。如果你运行了脚本并得到了 `"printf-mayhem.sh: line 26: printf: !: invalid number"`，那是因为你在`"'${STR:$TMP_CNT:1}"`中漏掉了单引号`'`。这意味着如何将返回的值解释为字符串/字符或数字值。

# 准备好你的脚本以适应不同语言的国际化

太好了，你有了这个很棒的脚本，但它是用标准英语写的，你希望面向那些讲其他语言的人。在像加拿大这样的国家，他们（我们）有两种官方语言：英语和法语。有时候，双语组件通过立法和本地化语言法律得以执行。

为了绕过这个问题，假设你是一个编写了脚本的个人，这个脚本首先用英语打印特定的字符串。他/她希望将所有字符串放入变量中，以便可以使用系统语言变量动态地进行替换。这里是基础知识：

+   创建一个利用**gettext**并设置适当变量的 Shell 脚本

+   创建一个包含必要语言定义的**po**文件

+   为你的脚本安装输出语言本地化文件

+   使用不同于原始语言的语言运行脚本（通过设置 LANG 变量）

不过，在开始之前，有两个术语需要讨论：国际化（i18n）和本地化（L10n）。国际化是一个使翻译和本地化/适应特定脚本或程序成为可能的过程，而本地化则是指将程序/应用程序适配特定文化的过程。

从一开始就翻译脚本是节省时间并提高多语言工作的成功率的一种有效方法。然而，请注意，如果开发人员只精通一种语言，或者翻译技能不立即可用，这可能是一个耗时的过程。

例如，在英语中，有几种方言。在美国，一个过程的产物或剩余物可以称为**artifact**，但在加拿大英语中，它可能被称为**artefact**。这可能不会引起注意（或被忽略），但程序可以通过特定的本地化自动适应。

# 准备就绪

让我们准备好进行练习，确保已经安装了以下应用程序和支持库：

```
$ sudo apt-get install -y gettext
```

接下来，验证你的语言环境变量（LANG）：

```
$ locale
LANG=en_CA:en
LANGUAGE=en_CA:en
LC_CTYPE="en_CA:en"
LC_NUMERIC="en_CA:en"
LC_TIME="en_CA:en"
LC_COLLATE="en_CA:en"
LC_MONETARY="en_CA:en"
LC_MESSAGES="en_CA:en"
LC_PAPER="en_CA:en"
LC_NAME="en_CA:en"
LC_ADDRESS="en_CA:en"
LC_TELEPHONE="en_CA:en"
LC_MEASUREMENT="en_CA:en"
LC_IDENTIFICATION="en_CA:en"
LC_ALL=
```

我们假设你的环境可能已经将某种形式的英语设为默认语言（`en_CA:en`是加拿大英语）——记得记录返回的值以便后续使用！

如果出现问题，你可能需要恢复你的语言和区域设置。网上有许多帖子提供帮助，几个提示是：`$ export LC_ALL="en_US.UTF-8"`；`sudo locale-gen`；以及`sudo dpkg-reconfigure locales`。

# 如何操作……

我们的活动开始如下：

1.  打开一个新的终端，并创建一个名为`hellobonjour.sh`的脚本，内容如下：

```
#!/bin/bash
. gettext.sh
function i_have() {
  local COUNT=$1
  ###i18n: Please leave $COUNT as is
  echo -e "\n\t" $(eval_ngettext "I have \$COUNT electronic device" "I have \$COUNT electronic devices" $COUNT)

}

echo $(gettext "Hello")
echo

echo $(gettext "What is your name?")
echo

###i18n: Please leave $USER as is
echo -e "\t" $(eval_gettext "My name is \$USER" )
echo

echo $(gettext "Do you have electronics?")

i_have 0
i_have 1
i_have 2

```

1.  运行`xgettext`以生成适当的字符串。我们不会使用这些结果，但这就是你如何生成一个极简的 PO 文件：

```
$ xgettext --add-comments='##i18n' -o hellobonjour_fr.po hellobonjour.sh --omit-header
```

1.  将已经编译好的字符串列表复制到名为`hellobonjour_fr.po`的语言 PO 文件中：

```
# Hellobonjour.sh
# Copyright (C) 2017 Ron Brash
# This file is distributed under the same license as the PACKAGE package.
# Ron Brash <ron.brash@gmail.com>, 2017
# Please ignore my terrible google translations; 
# As always, some is better than none!
#, fuzzy
msgid ""
msgstr ""
"Project-Id-Version: 1.0\n"
"Report-Msgid-Bugs-To: i18n@example.com\n"
"POT-Creation-Date: 2017-12-08 12:19-0500\n"
"PO-Revision-Date: YEAR-MO-DA HO:MI+ZONE\n"
"Last-Translator: FULL NAME <EMAIL@ADDRESS>\n"
"Language-Team: French Translator <fr@example.org>\n"
"Language: fr\n"
"MIME-Version: 1.0\n"
"Content-Type: text/plain; charset=iso-8859-1\n"
"Content-Transfer-Encoding: 8bit\n"

#. ##i18n: Please leave $COUNT as is
#: hellobonjour.sh:6
#, sh-format
msgid "I have $COUNT electronic device"
msgid_plural "I have $COUNT electronic devices"
msgstr[0] "J'ai $COUNT appareil electronique"
msgstr[1] "J'ai $COUNT appareils electroniques"

#: hellobonjour.sh:10
msgid "Hello"
msgstr "Bonjour"

#: hellobonjour.sh:13
msgid "What is your name?"
msgstr "Comment t'appelles tu?"

#. ##i18n: Please leave $USER as is
#: hellobonjour.sh:17
#, sh-format
msgid "My name is $USER"
msgstr "Mon nom est $USER"

#: hellobonjour.sh:20
msgid "Do you have electronics?"
msgstr "Avez-vous des appareils electroniques?"
```

1.  接下来，使用`msgfmt`将 PO 文件编译为具有`.mo`扩展名的二进制语言文件，并将其放入我们指定的语言文件夹中：

```
$ rm -rf locale/fr/LC_MESSAGES
$ mkdir -p locale/fr/LC_MESSAGES
$ sudo msgfmt -o locale/fr/LC_MESSAGES/hellobonjour.mo hellobonjour_fr.po
```

1.  一旦你将语言文件就位，创建以下名为`translator.sh`的脚本：

```
#!/bin/bash
./hellobonjour.sh

export TEXTDOMAIN="hellobonjour"
export TEXTDOMAINDIR=`pwd`/locale

export LANGUAGE=fr
./hellobonjour.sh
```

1.  执行`translator.sh`后，检查两次执行`translator.sh`的结果：

```
$ bash translator.sh
```

# 它是如何工作的……

不言而喻，翻译可能是一个棘手的事情，尤其是在处理编码时，以及生成在语言层面上有意义的结果时。此外，即使脚本中的值发生轻微变化，也可能会破坏 PO 文件，导致生成的脚本没有完全翻译（有时甚至根本没有翻译）。

在稍后修改脚本时，请小心不要破坏*keys*。

1.  第一步相当简单——你只需创建一个脚本。如果你运行脚本，你会看到纯粹的英文输出，但至少复数和单数的输出是正确的。请注意`. gettext.sh`；这一行准备了`gettext`以进行国际化/本地化。在脚本中，我们还使用了`gettext`、`eval_gettext`和`eval_ngettext`。这些是允许翻译发生的函数。对于简单的翻译，使用`gettext`，对于包含变量的翻译，使用`eval_gettext`，对于包含复数对象的翻译，使用`eval_ngettext`。正如你可能注意到的，`eval_ngettext`稍微复杂一些：`$(eval_ngettext "I have \$COUNT electronic device" "I have \$COUNT electronic devices" $COUNT)`。`eval_ngettext`的第一个参数是单数翻译，第二个是复数翻译，而计数是用来确定是使用单数还是复数值的变量。变量在原始脚本中以转义符`\$COUNT`来表示，翻译字符串中的变量将出现在翻译文件中，形式为`$COUNT`，而没有转义符：

```
./hellobonjour.sh 
Hello

What is your name?

   My name is rbrash

Do you have electronics?

   I have 0 electronic devices

   I have 1 electronic device

   I have 2 electronic devices
```

1.  在第二步中，我们使用`xgettext`创建了一个名为 PO 文件的语言文件。PO 是“便携对象”（Portable Object）的缩写。请注意，我们省略了头部信息，因为它会产生额外的输出。这在你想要写笔记、版本信息，甚至指定使用的编码时特别有用。

1.  我们并没有从头开始编写翻译，而是使用了我们值得信赖的朋友 Google 翻译来生成一些基础翻译，然后将其复制到`xgettext`的输出中。Xgettext 几乎生成了相同的文件！请注意`msgid`、`msgstr`、`msgplural`和`msgstr[...]`。`Msgid`和`msgid_plural`用于匹配原始值，就像它们是一个键一样。例如，当脚本运行时，`gettext`看到`"I have $COUNT electronic device"`，然后知道输出一个特定的翻译，匹配那个相同的`msgid`：

```
msgid "I have $COUNT electronic device"
msgid_plural "I have $COUNT electronic devices"
msgstr[0] ".."
```

1.  `hellobonjour_fr.po`包含了我们的所有翻译，现在我们可以使用一个叫做`msgfmt`的命令，它用于生成 MO 文件或机器对象。如果你用像`vi`这样的编辑器打开这个文件，你会发现它包含了一堆符号，代表二进制数据和字符串。这个文件不应该被编辑，而应该编辑输入的 PO 文件本身。

1.  接下来，我们创建一个名为`translator.sh`的文件。它运行`hellobonjour.sh`并包含几行代码，这些代码设置了三个重要的变量：`TEXTDOMAIN`、`TEXTDOMAINDIR`和`LANGUAGE`。`TEXTDOMAIN`通常是用来描述二进制文件或脚本的变量（可以将其视为命名空间），而`TEXTDOMAINDIR`是`gettext`查找翻译的目录。请注意，它位于本地相对目录中，而不是`/usr/share/locale`（尽管它也可以是）。最后，我们将`LANGUAGE`设置为法语（fr）。

1.  当我们执行`translator.sh`时，`hellobonjour.sh`会被运行两次，第一次输出英文，第二次输出法语：

```
$ bash translator.sh 

Hello

What is your name?

   My name is rbrash

Do you have electronics?

   I have 0 electronic devices

   I have 1 electronic device

   I have 2 electronic devices
Bonjour

Comment t'appelles tu?

   Mon nom est rbrash

Avez-vous des appareils electroniques?

   J'ai 0 appareils electroniques

   J'ai 1 appareil electronique

   J'ai 2 appareils electroniques
```

不要使用旧的格式 $"my string" 来进行翻译。它存在安全风险！

# 基于文件内容计算统计数据并减少重复项

初看之下，基于文件内容计算统计数据可能不是使用 Bash 脚本完成的最有趣的任务，但在某些情况下，它却能派上用场。让我们假设程序从多个命令获取用户输入。我们可以计算输入的长度，以确定它是否过短或过长。或者，我们也可以确定字符串的大小，以便为用其他编程语言（如 C/C++）编写的程序确定缓冲区大小：

```
$ wc -c <<< "1234567890"
11 # Note there are 10 chars + a new line or carriage return \n
$ echo -n "1234567890" | wc -c
10
```

我们可以使用像`wc`这样的命令来计算单词出现的次数、行数总数以及许多其他功能，结合脚本提供的功能来使用。

更好的是，如果我们使用一个名为**strings**的命令将所有可打印的 ASCII 字符串输出到一个文件呢？`strings`程序会输出*每一个*字符串的出现次数——即使它们是重复的。通过使用其他程序，如`sort`和`uniq`（或者这两者的组合），我们也可以对文件内容进行排序，并减少重复项，如果我们想计算文件中*唯一*行的数量：

```
$ strings /bin/ls > unalteredoutput.txt
$ ls -lah unalteredoutput.txt 
-rw-rw-r-- 1 rbrash rbrash 22K Nov 24 11:17 unalteredoutput.txt
$ strings /bin/ls | sort -u > sortedoutput.txt
$ ls -lah sortedoutput.txt 
-rw-rw-r-- 1 rbrash rbrash 19K Nov 24 11:17 usortedoutput.txt
```

现在我们已经知道了执行一些基本统计的几个基本前提，接下来继续进行实际步骤。

# 准备工作

让我们通过创建一个数据集来准备练习：

```
$ mkdir -p testdata
$ cat /etc/hosts > testdata/duplicates.txt; cat /etc/hosts >> testdata/duplicates.txt
```

# 如何做...

我们已经在之前的步骤中看到了这些概念，甚至在其中一个食谱中也使用了`wc`，所以我们直接开始：

1.  打开终端并运行以下命令：

```
$ wc -l testdata/duplicates.txt
$ wc -c testdata/duplicates.txt
```

1.  正如你可能已经注意到的，输出中包含了文件名。我们能用 AWK 移除它吗？当然可以，但我们也可以使用名为`cut`的命令来移除它。`-d`选项代表分隔符，我们想选择一个字段（由`-f1`指定）：

```
$ wc -c testdata/duplicates.txt | cut -d ' ' -f1
$ wc -c testdata/duplicates.txt | awk '{ print $1 }'
```

1.  假设我们有一个充满字符串的庞大文件。我们能减少返回的结果吗？当然可以，但首先我们使用`sort`命令对`testdata/duplicates.txt`中的元素进行排序，然后使用`sort`命令生成仅包含唯一元素的列表：

```
$ sort testdata/duplicates.txt
$ sort -u testdata/duplicates.txt
$ sort -u testdata/duplicates.txt | wc -l
```

# 工作原理...

总体而言，除了计数出现次数和排序的好处外，本脚本没有引入真正的抽象概念。排序可能是一个耗时的过程，尤其是在减少不需要的或多余数据时，或者当顺序很重要时，但当进行大规模操作时，它也可以带来回报，并且预处理通常会更快地得到结果。

接下来开始实际步骤：

1.  运行这两个`wc`命令会生成文件`testdata/duplicates.txt`的字符数和行数统计。同时，这也暴露了另一个问题：数据可能会被文件名前的空格填充：

```
$ wc -l testdata/duplicates.txt
18 testdata/duplicates.txt
$ wc -c testdata/duplicates.txt
438 testdata/duplicates.txt
```

1.  在步骤 2 中，我们使用`awk`和`cut`命令删除第二个字段。`cut`命令是一个有用的命令，用于修剪字符串，可以按分隔符分割，或使用硬编码的值（例如删除 X 个字符）。使用`cut`时，`-d`表示分隔符，这里是空格（`' '`），`-f1`表示字段`1`：

```
$ wc -c testdata/duplicates.txt | cut -d ' ' -f1
438
$ wc -c testdata/duplicates.txt | awk '{ print $1 }'
438
```

1.  在最后一步，我们运行`sort`命令三次。第一次我们仅仅对`testdata/duplicates.txt`中的元素进行排序，然后使用`-u`选项对其进行排序并保留唯一元素，最后使用`final`命令统计唯一元素的数量。当然，返回的值是`9`，因为我们在原始重复文件中有 18 行：

```
$sort testdata/duplicates.txt

127.0.0.1 localhost
127.0.0.1 localhost
127.0.1.1 moon
127.0.1.1 moon
::1 ip6-localhost ip6-loopback
::1 ip6-localhost ip6-loopback
fe00::0 ip6-localnet
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
ff02::2 ip6-allrouters
# The following lines are desirable for IPv6 capable hosts
# The following lines are desirable for IPv6 capable hosts

$ sort -u testdata/duplicates.txt

127.0.0.1 localhost
127.0.1.1 moon
::1 ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
# The following lines are desirable for IPv6 capable hosts
$ sort -u testdata/duplicates.txt | wc -l
9
```

# 使用文件属性和条件逻辑

在本书的早些时候，我们提到了各种针对字符串、数字和变量的测试。通过 Bash 内置的类似概念，我们也可以使用各种属性来对文件和目录进行测试。这是在条件逻辑介绍的基础上扩展，来执行文件测试。例如：是否存在某个文件？它是目录吗？等等。

但是，稍等一下，我们难道不可以直接使用执行结果并检查返回码吗？当然可以！这是你可以使用的另一种方法，尤其是当你使用支持所有 Bash 功能的 Bash 版本时。这只是“换个方式做事”的另一种方法。

让我们首先从一些常见的标志开始，如果满足以下条件则返回真：

+   `-e`：文件存在

+   `-f`：这是一个常规文件，而不是目录或设备文件

+   `-s`：该文件不为空或大小为零

+   `-d`：这是一个目录

+   `-r`：该文件具有读取权限

+   `-w`：该文件具有写入权限

+   `-x`：该文件具有执行权限

+   `-O`：这是文件的所有者，当前用户

+   `-G`：如果用户与您属于同一组，则执行该操作

+   `f1`（`-nt`，`-ot`，`-ef`）`f2`：指示`f1`是否比`f2`更新，或者比`f2`更旧，或者是否与`f2`是硬链接到同一文件

文件测试操作的更多信息可以在 GNU Bash 手册中找到：[`www.gnu.org/software/bash/manual/html_node/Bash-Conditional-Expressions.html`](https://www.gnu.org/software/bash/manual/html_node/Bash-Conditional-Expressions.html)。

# 准备开始

让我们通过创建一些文本文件和目录并添加内容来为练习做好准备：

```
$ cd ~/
$ mkdir -p fileops
$ touch fileops/empty.txt
$ echo "abcd1234!!" > fileops/string.txt
$ echo "yieldswordinthestone" > fileops/swordinthestone.txt
$ touch fileops/read.txt fileops/write.txt fileops/exec.txt fileops/all.txt
$ chmod 111 fileops/exec.txt; chmod 222 fileops/write.txt; chmod 444 fileops/read.txt; fileops/all.txt;chmod 777 fileops/all.txt
$ sudo useradd bob
$ echo "s the name" > fileops/bobs.txt
$ sudo chown bob.bob fileops/bobs.txt
```

这个食谱讲解的是执行一些简单的文件测试，并结合之前条件逻辑中学到的一些知识，但有个小变化——使用来自 CLI 的用户输入和文件权限。

注意命令`chmod`，`useradd`和`chmod`。Chmod 是你用来更改文件权限（例如执行权限等）的命令。

# 如何操作……

让我们按如下方式开始我们的活动：

1.  打开一个新的终端，启动你选择的编辑器并创建一个新的脚本。以下是脚本中的代码片段：

```
#!/bin/bash
FILE_TO_TEST=""

function permissions() {

  echo -e "\nWhat are our permissions on this $2?\n"
  if [ -r $1 ]; then 
    echo -e "[R] Read" 
  fi
  if [ -w $1 ]; then 
    echo -e     "[W] Write" 
  fi
  if [ -x $1 ]; then 
    echo -e "[X] Exec" 
  fi
}

function file_attributes() {

  if [ ! -s $1 ]; then
    echo "\"$1\" is empty" 
  else 
    FSIZE=$(stat --printf="%s" $1 2> /dev/null)
    RES=$?
    if [ $RES -eq 1 ]; then
      return
    else
      echo "\"$1\" file size is: ${FSIZE}\""
    fi
  fi

  if [ ! -O $1 ]; then
    echo -e "${USER} is not the owner of \"$1\"\n"
  fi
  if [ ! -G $1 ]; then
    echo -e "${USER} is not among the owning group(s) for \"$1\"\n"
  fi

  permissions $1 "file"

}
```

1.  执行脚本并尝试访问各种文件，包括不存在的目录和文件。你注意到了什么？

1.  现在使用以下命令删除该文件夹：

```
$ sudo rm -rf fileops
```

# 它是如何工作的……

首先，在深入研究脚本本身甚至文件的属性/属性之前，我们需要了解 Linux 及其兄弟操作系统的一些情况：

+   文件和目录可以有所有者。这意味着它们可以有一个所有者（用户）和与其所有权相关联的组。为此，我们可以使用`chown`和`chgrp`命令。

+   文件和目录可以应用不同的权限。这意味着它们可能是可执行的、可读的、可写的和/或所有的。为此，我们可以使用`chmod`命令和适当的权限设置。

+   文件和目录也可以是空的。

太棒了！此外，还有两个需要介绍的概念：

+   `read`命令，用于等待用户输入并将其读入变量。它在脚本中也非常有用，用于“暂停”功能。

+   递归函数。请注意，脚本中的特定函数在退出或用户按下*ctl + C*之前会不断调用。这就是递归，除非停止或应用限制。

到目前为止，我们还了解了函数、参数、输入/输出、返回码、子 shell 和条件逻辑。您可能没有注意到`!`字符，这用于否定语句。例如，如果我们使用`-e`测试运算符测试`fileops/bobs.txt`的存在，它将返回 true。相反，我们可以测试其相反情况，即`fileops/bobs.txt`不存在。

与反转或否定语句相同的逻辑也可以通过 if/else 功能实现，但有时候这可能会提高脚本的“可读性”和“流畅性”。最终，是否使用反转取决于脚本编写者。

1.  太棒了！我们已经创建了我们的脚本，并准备执行它。

1.  执行脚本后，我们看到：

```
$ ./files-extended.sh 
Welcome to the file attributes tester

To exit, press CTRL + C

What is the complete path of the file you want to inspect?
 # 
```

如果我们回顾一下这个配方的设置，我们知道我们在`fileops/`目录内创建了几个文件，并且其中一些文件具有不同的权限，其中一个文件是由名为`Bob`的用户拥有的。

让我们尝试一些执行（按顺序）：

+   `fileops/bobs.txt`

+   `fileops/write.txt`

+   `fileops/exec.txt`

+   `fileops/all.txt`

+   `thisDoesNotExist.txt`：

```
# fileops/bobs.txt

"fileops/bobs.txt" file size is: 11"
rbrash is not the owner of "fileops/bobs.txt"

rbrash is not among the owning group(s) for "fileops/bobs.txt"

What are our permissions on this file?

[R] Read

What is the complete path of the file you want to inspect?
 # fileops/write.txt

"fileops/write.txt" is empty

What are our permissions on this file?

[W] Write

What is the complete path of the file you want to inspect?
 # fileops/exec.txt

"fileops/exec.txt" is empty

What are our permissions on this file?

{X] Exec

What is the complete path of the file you want to inspect?
 # fileops/all.txt

"fileops/all.txt" is empty

What are our permissions on this file?

[R] Read
[W] Write
{X] Exec

What is the complete path of the file you want to inspect?
 # fileops

Directory "fileops" has children:

all.txt
bobs.txt
empty.txt
exec.txt
read.txt
string.txt
swordinthestone.txt
write.txt

What are our permissions on this directory?

[R] Read
[W] Write
{X] Exec

What is the complete path of the file you want to inspect?
 # thisDoesNotExist.txt

Error: "thisDoesNotExist.txt" does not exist!
$
```

因为`thisDoesNotExist.txt`不存在，脚本会突然退出并将您放回到控制台提示符。我们测试了各种标志、否定、所有权，甚至我们非常有用的实用程序`xargs`。

# 读取分隔数据和更改输出格式

每天，我们会打开许多不同格式的文件。但是，当处理大量数据时，使用标准格式始终是一个好习惯。其中之一称为**逗号分隔值**，或 CSV，它使用逗号（,）来分隔**元素**或**分隔**每一行。当您拥有大量数据或记录，并且这些数据将以脚本方式使用时，这特别有用。例如，在每个学期，系统管理员 Bob 需要创建一系列新用户并设置他们的信息。Bob 还从负责出勤的人那里得到一个标准化的 CSV（就像下面的片段中那样）：

```
Rbrash,Ron,Brash,01/31/88,+11234567890,rbrash@acme.com,FakePassword9000
...
```

如果管理员 Bob 只希望将此信息读入数组并创建用户，那么对他来说，解析 CSV 并在一个单一的脚本化操作中创建每个记录相对较为简单。这使 Bob 可以将时间和精力集中在其他重要问题上，比如解决最终用户的 WiFi 问题。

虽然这只是一个琐碎的例子，但这些文件可能以不同形式存在，使用**分隔符**（例如逗号`,`或美元符号`$`）、不同的数据和不同的结构。但是，每个文件的基础都是每一行都是一个需要被读入某种结构（不管是什么）的记录，在 SQL、Bash 数组等中：

```
Line1Itself: Header (optional and might not be present)
Line2ItselfIsOneREc:RecordDataWithDelimiters:endline (windows \r\n, in Linux \n)
....
```

在上述伪 CSV 的示例中，有一个可能是可选的标题（不出现），然后有几行（每行是一条记录）。Bob 有许多方法可以解析 CSV，但他可能会使用应用策略的专业函数，例如：

```
$ Loop through each item until done
for each line in CSV:
    # Do something with the data such as create a user
    # Loop through Next item if it exists
```

要读取数据，Bob 或您自己可能会使用：

+   对于循环和数组

+   一种迭代器的形式

+   手动逐行读取（效率低下）

一旦读入任何输入数据，下一步就是对数据本身进行处理。是要进行转换？是要立即使用？经过消毒处理？存储？还是转换为另一种格式？就像 Bob 一样，使用脚本读入的数据可以执行许多操作。

关于输出数据，我们也可以将其转换为 XML、JSON，甚至将其作为 SQL 插入到数据库中。不幸的是，此过程需要了解至少两个方面：输入数据的格式和输出数据的格式。

知道常见的数据格式及其通常应用验证时，可以在构建自动化脚本和识别未来任何变化时提供极大帮助。强制执行数据验证还有几个好处，可以在脚本突然无预警中断时挽救局面！

本文旨在引导你逐步阅读一个简单的 CSV 并将数据输出到一些任意的格式中。

# 准备工作

让我们通过创建一些模仿常见日常问题的数据集来为这项练习做好准备：

```
$ cd ~/
$ echo 
$ echo -e "XML_HDR='<?xml version="1.0" encoding="UTF-8"?>'\\nSRT_CONTR='<words type="greeting">'\\nEND_CONTR='</words>'" > xml-parent.tpl
$ echo -e "ELM='\"<word lang=\"\$1\">\"\$2\"</word>\"'" > word.tpl
$ echo -e "\"EN\",\"Hello\"\n\"FR\",\"Bonjour\"" > words.csv
```

在 Bash 中，单引号（'）用于字面字符串。在这种情况下，我们希望字符串的每一部分都存在，不需要转义斜杠和双引号。

要运行此脚本，您需要安装以下应用程序，以便在脚本中使用：

```
$ sudo apt-get install npm sed awk
$ sudo npm install -g xml2json-command
$ sudo ln -s /usr/bin/nodejs /usr/bin/node
```

# 如何操作...

让我们按以下方式开始活动：

1.  打开终端并创建名为 `data-csv-to-xml.sh` 的脚本，内容如下。保存后，使用 `$ bash data-csv-to-xml.sh` 执行该脚本：

```
#!/bin/bash

# Import template variables
source xml-parent.tpl
source word.tpl

OUTPUT_FILE="words.xml"
INPUT_FILE="words.csv"
DELIMITER=','

# Setup header
echo ${XML_HDR} > ${OUTPUT_FILE}
echo ${SRT_CONTR} >> ${OUTPUT_FILE}

# Enter content
echo ${ELM} | \
sed '{:q;N;s/\n/\\n/g;t q}'| \
awk \
'{ print "awk \x27 BEGIN{FS=\"'${DELIMITER}'\"}{print "$0"}\x27 '${INPUT_FILE}'"}' | \
 sh >> ${OUTPUT_FILE}

# Append trailer
echo ${END_CONTR} >> ${OUTPUT_FILE}

cat ${OUTPUT_FILE}
```

1.  审查输出时，请注意，“美观”的 XML 不是必需的，实际上，我们甚至不需要将 XML 分布在多行上。如果 web 应用程序只需要纯数据，那么额外的换行符和制表符就是不必要的传输数据。

1.  创建另一个名为 `data-xml-to-json.sh` 的脚本，内容如下。保存后，使用 `$ data-xml-to-json.sh` 执行该脚本：

```
!#/bin/bash
INPUT_FILE"words.xml"
OUTPUT_FILE="words.json"

# Easy one line!
xml2json < ${INPUT_FILE} ${OUTPUT_FILE}
```

1.  审查输出，看看它是多么简单！在两个脚本中，是否有可以改进的地方？

# 它是如何工作的...

我们已经讨论了几个重要的方面，比如 SED 和 AWK 命令的强大功能，甚至 CSV 文件，但我们还没有讨论能够**转换**数据格式和结构的重要性。CSV 是一种基础且非常常见的数据格式，但不幸的是，它并不是某些应用程序的最佳选择，因此我们可能会使用 XML 或 JSON。这里有两个脚本（或者说一个脚本和一个工具），可以将我们的原始数据转换成各种格式：

1.  执行 `data-csv-to-xml.sh`时，我们注意到几个要点：我们使用了两个源模板文件，可以根据需要进行修改，然后是一个大型的管道命令，利用了 sed 和 AWK。在输入时，我们将每个 CSV 值提取出来，并使用 `word.tpl` 中的格式模板构建一个 `<word lang="x">Y</word>` 的 XML 元素，其中 `$0` 是第一列，`$1` 是第二列。该脚本将生成 `words.csv` 并输出以下内容：

```
$ bash data-csv-to-xml.sh
<?xml version="1.0" encoding="UTF-8"?>
<words type="greeting">
<word lang="EN">"Hello"</word>
<word lang="FR">"Bonjour"</word>
</words>
```

1.  在第二个脚本中，我们仅仅将 `words.xml` 作为输入传递给命令 `xml2json`。输出将是 JSON 格式。很酷吧？

```
!#/bin/bash
{
  "words": {
    "type": "greeting",
    "word": [
      {
        "lang": "EN",
        "$t": "\"Hello\""
      },
      {
        "lang": "FR",
        "$t": "\"Bonjour\""
      }
    ]
  }
}
```

关于三种数据格式（CSV、XML 和 JSON）之间的差异及其原因，留给读者自行发现。另一个需要探索的练习是执行数据验证，以确保数据的完整性和约束。例如，XML 可以使用 XSD 模式来强制数据限制。
