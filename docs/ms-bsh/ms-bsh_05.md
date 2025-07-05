# 菜单、数组和函数

编写脚本通常意味着需要处理用户交互。你需要知道用户对脚本的期望，并让用户知道他们可以选择的选项。因此，我们提供一些选择给用户，他们提供答案，我们根据一些预设值来评估它们，并决定下一步要做什么。这就意味着要有一种方法，能够将一些数据展示给用户，收集他们的答案，循环选择项，并相应地做出反应。有多种方法可以实现这一点，我们将看到如何使用一些标准结构来完成这个任务。在本章结束时，我们将能够高效地提供、收集、存储和处理数据。

# case 语句

当你有更多的选择时，可以通过一系列的 if else 语句来处理它们：

```
if [condition];
then
command
else
command
fi

```

`if`子句可以根据需要进行嵌套，但从长远来看，选择过多会使代码混乱，降低可读性。编程的基本原则之一就是保持代码可读性，使其*优雅*，因为优雅在这里不仅仅意味着美丽，还意味着在时间上保持一致性。始终保持有意义的缩进，使子句突出。尽量用尽可能少的代码，始终采用相同的符号表示法，使代码紧凑高效。因此，拥有一连串的*if/then/else/fi*，并且有大量缩进，可能并不是你的脚本的最佳选择，但有一种替代方法已经被广泛采用，那就是使用`case`语句，形式如下：

```
case expression in condition_1) command_1 command_n ;; condition_2) command_1 command_n ;; condition_n | z) command_1 command_n ;; esac

```

这个表达式实际上是一个条件，必须匹配`condition x)`中给定的模式。一旦匹配为真，对应的命令块就会被执行：

```
condition_x)
command_1
command_n
;;

```

每一块命令被称为**子句**，并以`;;`双分号结束。所有的案例语句都包含在`case`和`esac`中，每个条件可以表示为`condition_1)`或`condition_1 | condition_2 | condition_n)`。

每个条件都可以作为触发子句内部命令执行的替代匹配项。

让我们看两个例子，展示如何使用*if/then/else/fi*和*case/esac*结构处理相同的选项：

```
#!/bin/bash
echo "Please, give me some input"
read input
if [[ $input =~ ^[[:digit:]]+$ ]]; 
then
echo "These are digits"
exit 0
elif [[ $input =~ ^[[:alpha:]]+$ ]]; 
then
echo "These are chars"
exit 0
else
echo "Dunno…"
exit 1
fi

```

现在，让我们看一些测试：

```
zarrelli:~$ ./if-statement.sh Please, give me some input 123 These are digits zarrelli:~$ ./if-statement.sh Please, give me some input abc These are chars zarrelli:~$ ./if-statement.sh Please, give me some input 12a Dunno... zarrelli:~$ ./if-statement.sh Please, give me some input !der Dunno...

```

这不是复杂的代码。我们请求一些输入，然后检查几个条件，看输入的文本是否完全由数字或字符组成；如果不是，则使用`dunno`作为默认答案。我们使用了稍微复杂一点的*if/then/else/fi*版本。我们采用了`elif`来检查另一个条件是否符合要求。我们本可以继续使用一系列`elif`来检查用户是否输入了字母数字或其他类型的字符，但从这个小示例中可以看出，代码开始变得有些难以阅读；它不那么清晰了。现在，让我们试试使用`case`语句，稍微做点不同的尝试：

```
#!/bin/bash
echo "Please, give me some input"
read input
case ${input//[[:alpha:]]} in
"") 
echo "There were alphabetic chars only" 
exit 0
;;
*[[:alnum:]]*) 
echo "There were digits in the string"
exit 0
;;
*) 
echo "There were non alphanumeric chars" 
exit 1
;;
esac

```

我们在这里做的是创建一个条件，以便从输入变量值中去除所有字母字符。剩下的部分会与`""`进行比较。这意味着什么？它简单地表示，如果在去除字符串中的所有字母字符后，剩下的是一个空字符串。如果条件不满足，我们将检查第二个条件：如果剩下的原始字符串中还有一些字母数字字符，意味着原始字符串中包含了数字字符。如果第二个条件也满足，则意味着在去除字母和数字字符后，剩下的字符串中包含其他字符。

你之前见过或使用过`case`语句吗？是的，可能比你想象的更常见。现在让我们做点有趣的事情；稍后我们会看到为什么它如此有趣。前往 Linux 发行版的`/et/init.d/`目录，仍然是**SystemV**兼容的，并查看你在那里找到的任何脚本。这些是处理系统服务启动/关闭的脚本，例如 cron 或 dbus，以及可以提供的其他服务，如 ssh、Apache 等。查看这些脚本时，有一件事立刻引起了注意，它们具有以下结构：

```
#!/bin/bash
case "$1" in
start)
:
exit 0
;;
stop)
:
exit 0
;;
status)
:
exit 0
;;
restart)
:
exit 0
;;
condrestart)
if $condition
then
exit 0
fi
exit 1
;;
*)
echo $"Usage: $0 {start|stop|restart|condrestart|status}"
exit 1
;;
esac
exit 0

```

这可以作为我们交互式脚本的基础，因为它提供了一个基本框架来处理用户交互。如你所见，每个条件后面都跟着一个`:`，作为基础脚本，没有任何操作会被执行；对于每个条件，我们都会优雅地退出并返回成功代码，除了条件重启和默认选项。条件重启实际上是可选的，但它允许你基于你设置的条件重新启动服务，所以是否保留或删除这个部分由你决定。正如我们在本书前面看到的，`case`结构有点类似于*if/then/else/fi*结构，后者用来匹配给定的不同字符串选项。该结构被`case`和`esac`标记包围；注意到`esac`是`case`的倒写。考虑以下条件：

```
string1 | string2 | stringn) do_something do_somethingn ;;

```

它以一个或多个需要匹配的字符串开始；每个可能的匹配项由`|`分隔，并以`;;`结束。如果有多个匹配项为真，只有第一个会被考虑。最后一个选项通常是一个星号，通常可以视为一个“通配符”，因为`*`会匹配用户输入的任何字符串。所以，如果前面的匹配项没有触发任何条件，这最后一个选项仍然会被匹配，通常用于编写帮助信息或一些命令行使用说明，因为如果用户没有输入正确的选项，它将始终显示。

要匹配的模式字符串可以选择性地以`(`开头，例如，`(string1 | string2 | stringn)`。记住，在`esac`之前的最后一个`;;`可以省略，不会导致任何问题。

每个开始子句的字符串是一个可选的匹配项，用于`case condition`中的匹配。通常，这是一个必须检查是否与每个开始子句的选项字符串匹配的文本字符串。如果条件是一个变量，它将通过参数扩展、变量扩展（波浪符）、命令替换、进程替换和去除引号来展开，但不会执行路径名扩展、大括号扩展或单词分割。鉴于此，你不需要对变量进行引号处理以确保安全处理。

从 Bash 4 开始，出现了一些子句终止符，我们已经在上一章中见过它们：

+   `;&`使得执行继续进行，并与下一个条件关联的命令被执行

+   `;;&`使得 Shell 检查该选项，并在条件匹配时执行关联的命令

如果未找到匹配项，则退出状态为 0，否则退出状态为最后执行的命令的状态。

让我们看看它们是如何表述的，并继续修改基础脚本：

```
#!/bin/bash
case "$1" in
start)
echo "We are starting..."
exit 0
;;
stop)
echo "We are stopping..."
exit 0
;;
status)
echo "We are checking the status..."
exit 0
;;
restart)
echo "We are restarting..."
exit 0
;;
*)
echo $"Usage: $0 {start|stop|restart|status}"
exit 1
;;
esac
exit 0

```

现在，让我们试试：

```
zarrelli:~$ ./terminators.sh Usage: ./terminators.sh {start|stop|restart|status}

```

如果没有任何参数，给定选项上无法找到匹配项，因此会触发全匹配星号，并执行`echo`打印使用信息到`stdout`：

```
zarrelli:~$ ./terminators.sh start We are starting... zarrelli:~$ ./terminators.sh stop We are stopping... zarrelli:~$ ./terminators.sh restart We are restarting... zarrelli:~$ ./terminators.sh status We are checking the status...

```

所有其他选项都很简单；我们删除了`condrestart`选项只是为了使脚本更简洁、更易读。

现在，让我们在最后一个子句上使用`;&`终止符：

```
restart) echo "We are checking the status..." exit 0 ;&

```

现在，使用状态作为参数执行脚本：

```
zarrelli:$ ./terminators.sh status We are checking the status...

```

嗯，尴尬，什么都没变。为什么？仔细看看子句：`;&`终止符前面是`exit 0`，所以在达到终止符之前，脚本的执行就停止了。好吧，让我们删除`exit 0`并再次使用状态参数调用脚本：

```
zarrelli:~$ ./terminators-last.sh status We are checking the status... We are restarting...

```

有趣，不是吗？我们从一个块级跳转到另一个块，因此我们在执行重启块命令后，立即执行了状态块命令。这正是我们期望`;&`运算符的行为，因为执行需要继续到下一个块，一旦满足第一个条件。但是现在，让我们做点别的，修改重启子句：

```
restart) echo "We are restarting..." ;;&

```

让我们执行脚本：

```
zarrelli:~$ ./terminators-last.sh status We are checking the status... We are restarting... Usage: ./terminators-last.sh {start|stop|restart|status}

```

发生了什么？简单来说，一旦匹配了`status`字符串，第一个`echo`命令被执行，并且`;&`运算符使得与重启子句关联的命令被调用，而没有进行其他字符串检查。在重启子句中，`;;&`运算符导致对下一个子句进行字符串检查，但由于这是与通配符的字符串匹配，它无论如何都匹配，因此执行了`usage`字符串的`echo`命令。但如果我们将状态子句和重启子句之间的运算符颠倒，会发生什么呢？

```
zarrelli:~$ ./terminators-last.sh status We are checking the status... Usage: ./terminators-last.sh {start|stop|restart|status}

```

我们进入了`status`条件，执行了代码，然后继续执行`restart`代码。这里命令并没有执行，因为`;;&`只有在字符串匹配时才会触发命令执行，但我们的`status`参数并不匹配`restart`选项。在下一行，我们使用了`;&`，这会将我们级联到下一个条件，且无论是否匹配，那个条件的代码都会被执行。如果你想强制执行`restart`条件下的命令，可以修改匹配选项：

```
restart | status) echo "We are restarting..." ;&

```

现在，让我们试试看：

```
zarrelli:~$ ./terminators-last.sh status We are checking the status... We are restarting... Usage: ./terminators-last.sh {start|stop|restart|status}

```

在这种情况下，我们提供了两个可能的重启条件匹配项，`restart`或`status`。第一个失败了，但第二个匹配上了，于是命令执行了，然后下一个条件命令也被执行了。

我们在一个启动脚本中看到过`case`结构，该结构与用户的交互较少，但现在，让我们开始使用这个结构，做出一些更有趣的事情：

```
#!/bin/bash
clear
echo -n "May I create an archive out of the current directory files? [yes or no]: "
read input
case $input in
[yY] | [yY][eE][sS] )
echo -e
echo "Yes, of course...I am proceeding"
echo -e "Archiving the following files\n"
now=$(date +%Y.%m.%d.%H.%M.%S)
filename=${PWD##*/}
tar cvzf ${now}.${filename}.tgz *
echo -e
echo "Archive $now.${filename}.tgz created!"
;;
[nN] | [nN][oO] )
echo -e
echo "No, so have a lovely day".;
echo -e
exit 1
;;
*)
echo -e
echo "Please just answer yes or no, y, n, in lower or capital."
echo -e
;;
esac

```

这个简单的脚本可以用来归档一个目录的内容。它会询问用户内容，并将用户的回答与小写和大写的`y`、`yes`、`n`和`no`进行比对。这里没有什么难的，我们只是在将之前章节中学到的内容组合起来。我们首先清除屏幕上的所有旧内容，然后通过`echo -n`询问用户输入`yes`或`no`。这样我们就不会输出新的一行，用户的回答将出现在双冒号后面的同一行。接下来的步骤是与字符列表进行比对。

`[yY] | [yY][Ee][Ss] )`将匹配小写和大写的`y`，还会匹配`yes`、`YES`以及这些字符大小写混合的所有情况。如果匹配成功，我们会告知用户我们将继续归档文件。注意我们使用了`echo -e`，它允许解析反斜杠转义序列，因此我们可以使用`\n`来插入新的一行并跳到终端的下一行。接下来的指令是命令替换，我们将`date`命令的输出赋值给变量`now`。我们正在创建将来拼接成唯一归档文件名的部分。在这个例子中，我们得到了一个由`year.month.day.hour.minute.second`组成的日期。我们将使用这个字符串作为归档文件的前缀，这样每秒都会生成一个唯一的文件名。但这也是一个限制，因为如果我们在同一时刻创建两个归档，后者将覆盖前者，两个文件的名字是一样的。

时刻牢记你的目标和限制，并坚持它们。在创建某些变量或条件时，你必须从你工作的范围出发，而不是过度思考你正在做的事情。这里的一个例子是归档名称的前缀。给我们一个在秒级时间范围内唯一的名称，允许我们每秒都有一个新的文件名，但如果相同的脚本在相同的时间被两次调用，例如从两个不同的终端调用，就有可能发生归档被覆盖的风险。为了避免这种情况，我们可以创建一个函数来伪造随机字符串作为前缀，从而避免这个问题，或者至少大大减少名称冲突的可能性。本书稍后会介绍如何创建一个随机字符串，但现在有必要吗？我们创建一个示例来展示如何使用`case`语句处理用户输入并创建归档，因此这个脚本不太可能在同一时间被执行两次。捕捉这种情况可能很好，但由于它不在这个项目的范围内，我们不会这样做，因为所花费的时间并不足以通过结果来证明其价值，也不太可能防止这种事件的发生。相反，在编写脚本时，要花时间明确自己脚本的功能，可能的陷阱，可能发生的错误，以及可以在脚本中编写的解决方案。

专业编程不仅仅是编码，它还包括规划，尝试理解可能发生的事情、你想要什么，以及如何达成目标。首先，问问自己你想要实现什么目标，如何实现，自己是否有足够的知识、资源、时间等来实现它。然后，做出相应的规划和开发。这不仅适用于你，也适用于你的客户，因为大多数时候最难的部分是理解客户真正想要的是什么，不管他是否意识到这一点，编码需要多少时间和资源，客户是否愿意给你足够的时间和资源。最后，问问自己，给定这些前提条件，你是否能够完成任务。假设你被要求用汇编语言编写一个简单的计算器，经过一些学习、练习和多次尝试，你可能可以完成。但如果你只有三天时间，从零开始，你能完成吗？所以，定义目标、目标的限制和资源，计划执行过程，考虑到你的代码可能遇到的陷阱，然后，考虑一个合理的应急预案：你的电脑可能会坏，或者你可能感冒了，任何事情都有可能发生，因此需要留出一定的应急时间，因为客户有一个交付日期，你必须按时交付代码，尽管可能有感冒、流感、电脑故障等情况。最后，保持规律。假设你每天有四小时的编码时间，你知道在这个时间内，你可以编写 50 行代码；但如果付出额外的努力，你最多可以写 65 行代码。不要把 65 行作为目标，要坚持一个平均值。你应该对这个进度有信心，因为你会每天进行编码，而且不能每天都冲刺。根据你知道自己能持续一段时间的努力来制定规律，以避免自己和客户面临不愉快的局面。

所以，在这段插曲之后，让我们继续检查 `yes` 条件中的最后几个有趣的命令：

```
filename=${PWD##*/} tar cvzf ${now}.${filename}.tgz *

```

第一行帮助我们使用参数扩展找到当前目录的名称：我们获取 `$PWD` 环境变量的内容，并删除与模式匹配的最长部分，在我们的例子中是路径直到最后一个斜杠，并将结果赋值给名为 filename 的变量。第二条指令从本地目录中所有文件创建一个归档文件，文件名为 `*`，并将之前准备好的不同部分合并成归档名。注意 `${}`，它允许我们在拼接时保留变量的值。那么，现在是执行脚本的时候了：

```
May I create an archive out of the current directory files? [yes or no]: yEs Yes, of course...I am proceeding Archiving the following files... base.sh case-statement.sh if-statement.sh terminators-last.sh terminators.sh user-case.sh Archive 2017.02.26.12.24.55.Scripts.tgz created! And check if the archive has been really created: zarrelli:~$ tar -tf 2017.02.26.12.24.55.Scripts.tgz base.sh case-statement.sh if-statement.sh terminators-last.sh terminators.sh user-case.sh

```

我们能够列出归档中的文件，简单的 `ls` 命令将再次确认结果：

```
zarrelli:~$ ls -A1 2017.02.26.12.24.55.Scripts.tgz base.sh case-statement.sh if-statement.sh terminators-last.sh terminators.sh user-case.sh

```

所有文件都在正确的位置，我们也可以看到新创建的归档文件。但是我们能确保一切都正常吗？让我们创建一个 `test` 目录并将所有文件复制进去：

```
zarrelli:~$ mkdir test zarrelli:~$ cp * test cp: -r not specified; omitting directory 'test' zarrelli:~$ chmod -R 0550 test

```

我们将文件复制到 `test` 目录，并设置目录权限，以使任何人都无法写入。现在，让我们进入该目录并运行脚本：

```
May I create an archive out of the current directory files? [yes or no]: yes Yes, of course...I am proceeding Archiving the following files... base.sh case-statement.sh if-statement.sh terminators-last.sh terminators.sh user-case.sh tar (child): 2017.02.27.08.40.03.test.tgz: Cannot open: Permission denied tar (child): Error is not recoverable: exiting now tar: Child returned status 2 tar: Error is not recoverable: exiting now Archive 2017.02.27.08.40.03.test.tgz created!

```

很有趣，我们看到了一些错误消息，但脚本仍然说我们有一个归档文件，来检查一下：

```
zarrelli:~$ ls -A1 base.sh case-statement.sh if-statement.sh terminators-last.sh terminators.sh user-case.sh

```

不，我们没有新的归档文件，这正是我们预期的，因为我们的用户无法在测试目录中写入任何内容。因此，这种情况是有可能发生的；有时候，我们的脚本会遇到问题，比如它无法从目录或某些文件中读取或写入，这就是我们现在必须规划的事情：一个应急方法来应对这个潜在问题。我们可以做的是检查 `tar` 命令的退出代码：如果它与 `0` 不同，表示归档创建失败，否则一切正常。所以，让我们重写 `yes` 条件，并在 `tar` 命令后添加一个测试：

```
tar cvzf $now.${filename}.tgz * if [ $? -ne 0 ] then echo "Sorry there was an issue creating the archive..." exit 1 else echo -e echo "Archive ${now}.${filename}.tgz created!" exit 0 fi ;;

```

小心不要在 `tar` 和 `test` 之间写任何命令，因为 `$?` 捕获的是最后执行命令的退出代码。现在，让我们检查一下结果：

```
May I create an archive out of the current directory files? [yes or no]: yes Yes, of course...I am proceeding Archiving the following files... tar (child): 2017.02.27.09.20.17.test.tgz: Cannot open: Permission denied tar (child): Error is not recoverable: exiting now base.sh case-statement.sh if-statement.sh terminators-last.sh terminators.sh user-case.sh tar: 2017.02.27.09.20.17.test.tgz: Cannot write: Broken pipe tar: Child returned status 2 tar: Error is not recoverable: exiting now Sorry there was an issue creating the archive…

```

不错！现在，我们的脚本告诉我们发生了问题，并且它停止显示归档成功创建的消息，尽管 `tar` 命令失败了。无论如何，输出有点混乱。我们已经从错误消息中得知存在错误，所以让我们通过修改 `tar cvzf $now.${filename}.tgz * 2>/dev/null` 来清理输出。

我们刚刚将标准错误重定向到 `/dev/null`，所以错误信息不会显示到 `stdout`。

大多数时候，最好是屏蔽系统或应用程序的错误，给客户提供由你设计的、更有意义的错误消息。请记住，并不是所有用户都是系统管理员或程序员，也不熟悉操作系统、应用程序错误消息或代码。

看一下输出：

```
May I create an archive out of the current directory files? [yes or no]: yes Yes, of course...I am proceeding Archiving the following files... base.sh case-statement.sh if-statement.sh terminators-last.sh terminators.sh user-case.sh Sorry there was an issue creating the archive.

```

这样其实更简洁，甚至可以进一步优化，去掉文件列表。你知道怎么做吗？提示：`stdout`…

但是，再看看以下语句：

```
[yY] | [yY][eE][sS] ) echo -e echo "Yes, of course...I am proceeding" echo -e "Archiving the following files...\n" now=$(date +%Y.%m.%d.%H.%M.%S) filename=${PWD##*/} if tar cvzf $now.${filename}.tgz * 2>/dev/null then echo -e echo "Archive ${now}.${filename}.tgz created!" exit 0 else echo "Sorry there was an issue creating the archive..." exit 1 fi ;;

```

结果是一样的，但我们使用 `if` 语句的方式更符合习惯，因为它的目的是测试条件是否为真，或者在本例中，命令是否成功或失败。不过，你有很多方法可以实现同样的结果；看看这里：

```
zarrelli:~$ rm base.sh && echo "File deleted" || echo "File not deleted" rm: cannot remove 'base.sh': Permission denied File not deleted

```

在测试目录中，`remove` 命令因权限不足而失败，但我们可以向上进入一个目录并创建一个测试文件：

```
zarrelli:~$ touch test1 zarrelli:~$ rm test1 && echo "File deleted" || echo "File not deleted" File deleted

```

我使用了逻辑与/或运算符来利用我通常称之为短路的特性。请阅读以下语法的前述示例：

*如果 [command1] 为真，则我们也会评估命令 2 [但如果第一个条件不为真，则执行命令 3]*

使用逻辑与（AND），`rm test1` 和 `echo "file deleted"` 两个命令都必须为真，整体表达式才会成立（在 OR（||）的左侧）。如果第一个命令未能返回真，第二个命令将不会被考虑（短路）。

如果第一个部分`left_command && right_command`的结果为假，OR 操作会触发最后一个命令的执行。但如果在`||`之前的第一个部分为真，则第二部分不会被触发。因为对于整体表达式`left_command || right_command`来说，只要其中一个为真即可，若第一个为真，则第二个命令甚至不会被评估（短路）。

这种错误处理方式在出现问题时不会导致脚本退出，这在大多数情况下是可取的行为，但有时我们也可以使用一个技巧，在出错时让我们退出：

```
#!/bin/bash -e

```

如果任何命令在子 shell 或大括号中退出时返回非零代码，这将导致脚本退出。如果失败的命令是紧跟在`while`或`until`命令后的命令列表的一部分，或者是*fi/elif*测试的一部分，或是在`&&`或`||`后执行的命令的一部分，则不适用此规则。

我们稍后会看到更多关于如何使用 case 构造的例子，目前我们要看一些有趣的内容，它将影响你收集、存储和处理数据的方式。所以，准备好学习数组吧。

# 数组

将数组看作一种可以容纳多个对象的结构，类似于一个具有一个或多个值的变量。想象一下你有几个朋友，你想把他们的名字写下来：

```
friend_1=Anthony friend_2=Mike friend_3=Noel friend_4=Tarek friend_5=Dionysios

```

一旦你实例化了变量，你就可以进行解引用，解引用是指检索一个值的操作。这是可以的，但它在某种程度上将你限制在一些局限性中，例如你必须调用准确的变量名才能访问它的值，不能轻松地在它们之间循环，不能快速判断值的数量，等等。针对这些操作，有一种合适的结构，它为我们提供了方便，使我们可以将这些值作为一个整体进行处理——这就是数组：

```
friends=(Anthony Mike Noel Tarek Dionysios)

```

数组中的元素是有索引的，位置是在赋值时确定的，所以`Anthony`将位于第一个位置，`Dionysios`位于第五个位置。但一旦声明并实例化，我们可以在特定位置向数组添加元素：

```
friends[6]=Claudia

```

你怎么检查目前为止所说的是否正确？一个好方法是访问不同的元素，并打印出不同位置的值：

```
zarrelli:~$ friends=(Anthony Mike Noel Tarek Dionysios) ; echo ${friends[0]} ; echo ${friends[4]} ; echo ${friends[5]} ; friends[5]=Claudia ; echo ${friends[5]} Anthony Dionysios Claudia

```

从之前的例子中，我们可以看到一些有趣的事情：

+   数组的第一个位置的索引是 0。

+   通过`${array_name[index]}`的形式访问一个值。

+   如果没有赋值，一个位置不会持有任何值。

+   我们可以使用索引给任何位置赋值。

现在我们来给列表添加另一个人：

```
friends[-2]=Ilaria

```

现在，如果能有一种方法一次性打印出整个数组的内容就好了，因为元素数量在增长，回显所有索引值需要一些时间。所以，我们可以使用`array_name[@]`或者`array_name[*]`来访问数组的全部内容：

```
zarrelli:~$ echo ${friends[@]} Anthony Mike Noel Tarek Ilaria Claudia

```

这里有趣的是`Ilaria`在数组中的位置。我们将这个名字插入到位置-2，因此使用负索引提供了 Bash 4.2 中引入的新功能，允许我们从数组的末尾开始定位位置。所以，-2 意味着从数组末尾开始两个插槽。但现在，让我们回到数组声明。我们刚刚看到了一种创建数组的方式：

```
array_name=(element_1 element_2 element_n)

```

还有其他方式可以创建数组：

```
array_name[index]

```

在这种情况下，索引必须是正整数，因为我们没有任何插槽可以向后计数：

```
zarrelli:~$ test[2]="Here I am!" ; for i in {0..5} ; do echo $i ${test[$i]} ; done 0 1 2 Here I am! 3 4 5 declare -a array_name

```

这里不需要索引，即使给出，也会被忽略。这种声明数组的方式在你还不知道将存储哪些值时非常有用：

```
#!/bin/bash
declare -a friends
clear
echo -n "Can you please tell me the name of some of your friends: "
read -a friends
echo "So, your friends are: ${friends[@]}"

```

即使你使用另一种形式实例化数组，在实例化之前放置`declare -a array_name`也能加速后续对数组本身的操作。

我们刚刚声明了一个名为`friends`的数组，并使用了内置的 read 命令，但这次我们给了`-a`选项，强制 read 从用户那里获取任何单词，并按顺序分配给命名数组的索引。请注意，`-a`会在第一次赋值前强制取消数组的设置。现在，让我们试试这个小脚本：

```
Can you please tell me the name of some of your friends: Ilaria Max Ron So, your friends are: Ilaria Max Ron

```

从 Bash 4 开始，有一种新的数组类型，称为**关联数组**。它们与我们之前看到的索引数组稍有不同：可以把它想象成一组两个关联的数组：

```
#!/bin/bash
declare -A friends
clear
echo -n "Can you please tell me the name of one of your friends: "
read name
echo -n "And now his email address: "
read address
friends[$name]=${address}
echo -e "So, your friend name is: ${!friends[@]}\nHis email address is: ${friends[@]}"

```

我们刚刚声明了一个名为`friends`的关联数组，并请求用户提供两个值，一个是名称，另一个是电子邮件，但我们将它们存储在两个不同的变量中，而不是直接插入数组。插入数组是接下来的操作。使用名称值作为索引，地址值作为关联内容：

```
Can you please tell me the name of one of your friends: Giorgio Zarrelli And now his email address: giorgio@whatever.net So, your friend name is: Giorgio Zarrelli His email address is: giorgio@whatever.net

```

为了演示目的，我们没有检查输入，但请看这个：

```
Can you please tell me the name of one of your friends: And now his email address: ./declare-array-associative.sh: line 9: friends[$name]: bad array subscript So, your friend name is: His email address is: 

```

关联数组的索引不能完全为空，因此我们可以修改之前的脚本，添加对名称值的检查：

```
read name if [[ -z "$name" ]] then echo "The name value cannot be blank" exit 1 fi

```

我们刚刚检查了`name`变量是否未设置或为空，这为我们省去了很多麻烦。

命令行上的标准参数分隔符是空格，但你可以使用 IFS 环境变量改变脚本读取单个参数的方式：

```
#!/bin/bash
IFS=","
declare friends
clear
echo -n "Can you please tell me the name of some of your friends: "
read -a friends
echo "So, your friends are: "
for i in ${!friends[*]}
do 
echo "$i - ${friends[$i]}"
done

```

现在，让我们执行它并提供参数`Anthony Mike`：

```
Can you please tell me the name of some of your friends: Anthony Mike So, your friends are: 0 - Anthony Mike

```

两个名称位于相同的索引位置，因此它们不会被视为两个不同的朋友。那么，让我们现在用逗号来分隔这些名称：

```
Can you please tell me the name of some of your friends: Noel,Tarek So, your friends are: 0 - Noel 1 – Tarek

```

在这里，`Noel`位于索引 0，而`Tarek`位于索引 1，因此它们实际上是不同的名称，存储在数组的不同位置。但是，如果用户没有及时回答呢？嗯，另一个环境变量可以帮助我们：

```
#!/bin/bash
IFS=","
TMOUT=3
declare friends
clear
echo -n "Can you please tell me the name of some of your friends: "
read -a friends
if [ ${#friends[@]} -eq 0 ]
then
echo "You did not provide me with any names"
exit 1
else
echo "So, your friends are: "
for i in ${!friends[*]}
do 
echo "$i - ${friends[$i]}"
done
fi
exit 0

```

我们刚刚将三秒的值赋给了`TMOUT`环境变量，该变量定义了 shell 和`read`内建命令的标准超时周期。用于交互式 shell 时，如果在超时到期之前终端没有输入，shell 本身会退出。与`read`内建命令一起使用时，它定义了超时周期，如果没有输入，命令将在超时后终止。在我们的案例中，当超时发生时，我们会检查存入数组中的元素数量：如果为 0，我们会打印警告信息并以 1 退出：

```
Can you please tell me the name of some of your friends: You did not provide me with any names

```

如果我们在数组中找到某个元素，我们将循环并打印所有值和相关索引。

```
Can you please tell me the name of some of your friends: Anthony,Mike,Tarek So, your friends are: 0 - Anthony 1 - Mike 2 - Tarek

```

Bash 4 引入了一个新的内建命令`mapfile`，用于从标准输入（如果提供了`-u`选项，则是文件描述符）读取行，并将其加载到索引数组中。这个可以用来做什么呢？来看一下这个示例——我们首先创建一个名为`file.txt`的文件，里面列出了我们的朋友：

```
zarrelli:~$ cat friends.txt Anthony Dionysios Ilaria Mike Noel Tarek

```

现在，让我们创建一个脚本，利用`mapfile`内建命令：

```
#!/bin/bash
declare -a friends
echo -e
echo -e "Reading friends list from friends.txt file..."
mapfile friends < friends.txt
echo -e "File content loaded!"
echo -e "So, your friends are: \n${friends[@]}"

```

最后，让我们运行脚本：

```
zarrelli:~$ ./mapfile-array.sh Reading friends list from friends.txt file... File content loaded! So, your friends are: Anthony Dionysios Ilaria Mike Noel Tarek

```

很容易看出为什么`mapfile`很有用：我们加载了文件中的所有行，而不需要使用任何循环或处理每一行。事实上，如果使用内建的`read -a`，它只会将文件的第一行加载到数组中，之后我们还得用某种循环处理文件的其余部分。使用`mapfile`时，你只需要加载所有内容，就这么简单。

那么，让我们回顾一下存储值到数组中的不同方法：

```
array_name[i]=value

```

这非常直接。通过索引选择数组中的位置并赋值。它可以是任何算术表达式中的整数。如果是负数，那么数组中可以访问从最后一个值开始的`i`个位置：

```
zarrelli:~$ my_array[$((3*2))]=my_value ; echo ${my_array[6]} my_value

```

我们还可以省略索引，在这种情况下，值将被分配到索引 0 位置：

```
my_array=my_other_value ; for i in {0..6} ; do echo $i ${my_array[$i]} ; done 0 my_other_value 1 2 3 4 5 6 my_value

```

对于关联数组来说也是如此：

```
zarrelli:~$ my_associative=my_value ; for i in {0..5} ; do echo $i ${my_associative[$i]} ; done 0 my_value 1 2 3 4 5

```

在这种情况下，`0`实际上是作为字符串使用的，正如它在关联数组中应该是的。

存储数据到数组中的另一种方法是之前看到的复合赋值，但它只适用于索引数组：

```
zarrelli:~$ friends=(Anthony Mike Noel Tarek Dionysios) ; echo ${friends[0]} ; echo ${friends[4]} ; echo ${friends[5]} ; friends[5]=Claudia ; echo ${friends[5]} Anthony Dionysios

```

使用这种方法时，我们需要小心，因为数组在赋值之前会被取消设置，因此所有之前的值都会丢失：

```
zarrelli:~/$ friends=(Anthony Mike Noel Tarek Dionysios) ; echo -n "Old array values: ${friends[@]}" ; friends=(Ilaria) ; echo -e ; echo -n "New array values: ${friends[@]}" ; echo -e Old array values: Anthony Mike Noel Tarek Dionysios New array values: Ilaria

```

我们可以使用`+=`操作符保留数组中的旧内容：

```
zarrelli:~$ friends=(Anthony Mike Noel Tarek Dionysios) ; echo -n "Old array values:${friends[@]}" ; friends+=(Ilaria) ; echo -e ; echo -n "New array values: ${friends[@]}" ; echo -e Old array values:Anthony Mike Noel Tarek Dionysios New array values: echo Anthony Mike Noel Tarek Dionysios Ilaria

```

然后，我们使用键进行复合赋值：

```
zarrelli:~$ my_array=([2]=first_value [4]=second_value) ; for i in {0..5} ; do echo $i ${my_array[$i]} ; done 0 1 2 first_value 3 4 second_value 5

```

关联数组也是如此：

```
#!/bin/bash
declare -A friends
friends=([Mike]="is a friend" [Anthony]="is another friend")
for i in Mike Anthony
do 
echo "$i - ${friends[$i]}"
done
And now let's try it:
zarrelli:~$ ./associative.sh 
Mike - is a friend
Anthony - is another friend

```

请注意，关联数组并不意味着键的顺序；正如从前面的示例中看到的，它们是无序的。

最后，我们看到`mapfile`方法：

```
zarrelli:~$ mapfile < friends.txt ; echo ${MAPFILE[@]} Anthony Dionisios Ilaria Mike Noel Tarek

```

我们使用了稍微更紧凑的`mapfile`命令形式，因为我们没有指定数组的名称来读取文件内容。在这种情况下，当没有提供数组时，`mapfile`会将数据存储到默认的`MAPFILE`数组中。

现在我们已经看到了不同的存储值的方法，是时候以各种方式检索它们了：

```
${my_array[i]} 

```

`i` 可以是算术表达式中的任何整数。如果它是负数，那么数组中距离最后一个值 `i` 个位置的值将可用：

```
zarrelli:~$ my_array=("first value" "second value" "third value" "fourth value" "fifth value") ; echo "${my_array[-3]}" third value

```

我们可以注意到这里有几个有趣的点。

`-2` 索引指向数组中的最后一个位置，该位置被 `"fifth value"` 填充，并减去两个槽，因此我们从后往前数，直到到达 5-2 的槽。数组索引中的第三个位置（索引从 0 开始）包含字符串 `"third value"`。

第二，我们使用了带有空格的字符串，得益于双引号将它们保留了下来。为了确保输出正确，我们在回显时也引用了检索到的值。

类似地，我们可以通过使用名为`$my_associative[string]`的形式，将元素的值检索到关联数组中。

其中字符串是存储在数组中的与我们想要检索的值相关的键之一：

```
my_associative=([George]=first_value [Anthony]=second_value) ; echo ${my_associative[Anthony]} second_value

```

我们还可以一次性检索所有存储的值，使用以下方式：

```
${my_array[@]} ${my_array[*]} ${my_associative[@]} ${my_associative[*]}

```

正如我们从以下示例中看到的：

```
zarrelli:~$ echo ${my_array[@]} first value second value third value fourth value fifth value zarrelli:~$ echo ${my_array[*]} first value second value third value fourth value fifth value zarrelli:~$ echo ${my_associative[@]} my_value second_value first_value zarrelli:~$ echo ${my_associative[*]} my_value second_value first_value

```

但如果你不想要所有的值，我们可以通过以下语法以 *切片* 的方式获取它们：

```
${my_array[@]:S:O} ${my_array[*]:S:O}

```

其中 `S` 为我们开始的索引值，`O` 为读取值的偏移量：

```
zarrelli:~$ my_array=("first value" "second value" "third value" "fourth value" "fifth value") ; echo "${my_array[@]:3:2}" fourth value fifth value zarrelli:~$ my_array=("first value" "second value" "third value" "fourth value" "fifth value") ; echo "${my_array[*]:3:2}" fourth value fifth value

```

在这两个示例中，我们从索引位置 3 开始读取，实际上读取了接下来的两个值。如果我们省略其中一个值，剩余的值将作为从位置 0 开始的偏移量来考虑：

```
zarrelli:~$ my_array=("first value" "second value" "third value" "fourth value" "fifth value") ; echo "${my_array[*]:2}" third value fourth value fifth value

```

我们还可以使用本书中前面提到的子字符串删除操作符：

```
zarrelli:~$ my_array=("first value" "second value" "third value" "fourth value" "fifth value") ; echo "${my_array[@]%%fou*}" first value second value third value fifth value

```

或者：

```
zarrelli:~$ my_array=("first value" "second value" "third value" "fourth value" "fifth value") ; echo "${my_array[@]#s?cond}" first value value third value fourth value fifth value

```

或者：

```
zarrelli:~$ my_array=("first value" "second value" "third value" "fourth value" "fifth value") ; echo "${my_array[@]/third/forth-1}" first value second value forth-1 value fourth value fifth value

```

以此类推。

请注意，`${array_name[@]}`和`${array_name[*]}`在参数扩展时遵循与`$@`和`$*`相同的规则，第一个表示将所有参数视为一个单一字符串，后者则将参数视为单独的单词并加以引用。

现在我们知道如何从数组中存储和检索数据，接下来要看看如何删除它们。我们可以使用以下命令：

+   `unset array_name`

+   `unset array_name[@]`

+   `unset array_name[*]`

请看以下示例：

```
zarrelli:~$ my_array=(one two three four five) ; echo "The content of the array is: ${my_array[@]}" ; unset my_array ; echo "Now the content of the array is: ${my_array[@]}" The content of the array is: one two three four five Now the content of the array is: zarrelli:~$ my_array=(one two three four five) ; echo "The content of the array is: ${my_array[@]}" ; unset my_array[@] ; echo "Now the content of the array is: ${my_array[@]}" The content of the array is: one two three four five Now the content of the array is: zarrelli:~$ my_array=(one two three four five) ; echo "The content of the array is: ${my_array[@]}" ; unset my_array[*] ; echo "Now the content of the array is: ${my_array[@]}" The content of the array is: one two three four five Now the content of the array is: 

```

我们可以在定义的索引位置取消设置单个值：

```
zarrelli:~$ my_array=(one two three four five) ; echo "The content of the array is: ${my_array[@]}" ; unset my_array[2] ; echo "Now the content of the array is: ${my_array[@]}" The content of the array is: one two three four five Now the content of the array is: one two four five

```

对于关联数组也是如此：

```
#!/bin/bash
declare -A friends
friends=([Mike]="is a friend" [Anthony]="is another friend")
unset friends[Mike]
for i in Mike Anthony
do 
echo "$i - ${friends[$i]}"
done

```

如您从以下输出中所看到的：

```
zarrelli:~$ ./associative-remove.sh Mike - Anthony - is another friend

```

你也可以将空值分配给数组和单个值，无论是对于索引数组还是关联数组：

```
zarrelli:~$ my_array=(one two three four five) ; echo "The content of the array is: ${my_array[@]}" ; my_array[2]="" ; echo "Now the content of the array is: ${my_array[@]}" The content of the array is: one two three four five Now the content of the array is: one two four five

```

或者：

```
zarrelli:~$ my_array=(one two three four five) ; echo "The content of the array is: ${my_array[@]}" ; my_array=() ; echo "Now the content of the array is: ${my_array[@]}" The content of the array is: one two three four five Now the content of the array is: 

```

对于关联数组，只需在前面的脚本中修改这一行：

```
friends=([Mike]="is a friend" [Anthony]="is another friend") friends[Mike]="" for i in Mike Anthony

```

现在，运行它：

```
zarrelli:~$ ./associative-remove.sh Mike - Anthony - is another friend

```

否则，再次修改这些行：

```
friends=([Mike]="is a friend" [Anthony]="is another friend") friends=() for i in Mike Anthony

```

现在，运行脚本：

```
zarrelli:~$ ./associative-remove.sh Mike - Anthony - 

```

有一些最终说明，涉及一些我们可以用来处理数组的有趣符号：

```
${#array_name[index]}

```

以下代码解释了在索引位置指向的数组值的长度：

```
my_array=(one two three four five) ; echo "The length of ${my_array[4]} is of ${#my_array[4]} characters" The length of five is of 4 characters

```

或者：

```
#!/bin/bash
declare -A friends
friends=([Mike]="is a friend" [Anthony]="is another friend")
echo "The lenght of \"${friends[Anthony]}"\ is ${#friends[Anthony]}"
And executing it gives us:
zarrelli:~$ ./associative-count.sh 
The lenght of "is another friend" is 17

```

数组的另一个有趣扩展是：

#{#array_name[*]} 或 #{#array_name[@]}

这可以扩展为数组中的元素数量：

```
zarrelli:~$ my_array=(one two three four five) ; echo "We have ${#my_array[*]} elements in the array" We have 5 elements in the array

```

或者：

```
#!/bin/bash
declare -A friends
friends=([Mike]="is a friend" [Anthony]="is another friend")
echo "We have ${#friends[@]} elements in the array"

```

这给我们带来了以下结果：

```
zarrelli:~$ ./associative-elements.sh We have 2 elements in the array

```

现在我们拥有了所有元素，可以看看如何遍历数组的内容。我们可以从已经看到的简单例子开始：

```
#!/bin/bash
declare -a my_array
my_array=("one" "two" "three" "four" "five")
for (( i=0 ; i<${#my_array[*]} ; i++ ));
do 
echo "${my_array[i]}" 
done

```

现在，让我们执行它：

```
zarrelli:~$ ./loop1.sh one two three four five

```

这是一种简单的方法，但有一些限制：索引从 0 开始，并且预期进展是顺序的。但我们可以做一些事情来克服这些限制：

```
#!/bin/bash
declare -a my_array
my_array=("one" "two" "three" "four" "five")
for i in ${my_array[*]} ; 
do 
echo "$i" 
done

```

我们修改了`for`语句，现在`i`将实例化为从`$(my_array[*]}`扩展中获得的每个元素：

```
zarrelli:~$ ./loop2.sh one two three four five

```

到目前为止，一切顺利。我们可以访问值，但索引呢？请记住，`${!array_name[@]}` 和 `${!array_name[*]}` 展开为数组的索引列表。只需注意，在引号中使用`@`会将每个键展开为一个单独的单词。因此，了解这一点后，我们可以同时检索值和索引：

```
#!/bin/bash
declare -A friends
friends=([Mike]="is a friend" [Anthony]="is another friend")
for i in ${!friends[*]}
do 
echo "$i - ${friends[$i]}"
done

```

这将给我们以下结果：

```
zarrelli:~$ ./loop3.sh Mike - is a friend Anthony - is another friend

```

最后，稍微复杂一点的内容：

```
#!/bin/bash
declare -A friends
friends=([Mike]="is a friend" [Anthony]="is another friend")
indexes=(${!friends[*]})
for ((i=0 ; i<${#friends[*]} ; i++));
do 
echo "${indexes[i]} - ${friends[${indexes[i]}]}"
done

```

我们将朋友数组的所有索引存储在另一个名为`indexes`的数组中，然后使用它来从前一个数组中获取内容：

```
zarrelli:~$ ./loop4.sh Mike - is a friend Anthony - is another friend

```

我们很快会看到更多关于迭代的内容，但接下来我们要关注的是如何通过利用 Bash 提供的另一个构造——函数——来使我们的代码简洁、整洁并且可重用。

# 函数

在本书的这一部分，我们已经足够了解如何编写自己的代码，处理变量，与用户和环境交互，做很多事情，因此我们已经准备好制造混乱了。我们知道如何写一堆代码行，但我们仍然不知道如何保持代码整洁、简洁，更重要的是，如何使代码可重用。正如我们从到目前为止的示例中轻松猜到的那样，脚本或命令行是单向处理的代码流；构成我们命令的字符是从左到右、从上到下读取的。因此，当你传递一个构造或赋值时，它就完成了，如果你想按照之前的方式处理某些内容，你必须再次编写执行该过程的代码。所以，如果你编写的不仅仅是一个小脚本，你很可能会陷入大量重复代码、凌乱布局和低效的困境；但 Bash 和其他任何编程语言一样，为我们提供了一种方法来克服这些问题。我们说的就是函数。什么是函数？一个例子比千言万语更能说明函数的作用。让我们创建一个小的代码片段：

```
#!/bin/bash
if (("$1" < "$2"))
then
echo "Great! The integer $1 is less than $2"
else
echo "The integer $1 is not less than $2..."
fi

```

它接受两个位置参数作为输入，以检查第一个参数是否小于第二个参数，假设输入的是整数：

```
zarrelli:~$ ./minor-no-function.sh 1 2 Great! The integer 1 is less than 2 Now, let's move part of the code into a function: #!/bin/bash minor() { if (("$1" < "$2")) then echo "Great! The integer $1 is less than $2" else echo "The integer $1 is not less than $2..." fi } minor "$1" "$2"

```

现在是时候尝试我们全新的函数了：

```
zarrelli:~$ ./test.sh 1 2 Great! The integer 1 is less than 2 What did we do? First, we see that a function declaration has the following structure: function_name() { instruction_1 … instruction_n }

```

但它也可以有如下结构：

```
function _name() { instruction_1 … instruction_n }

```

它甚至可以使用`function`关键字来声明，如下所示：

```
function function_name { instruction_1 … instruction_n }

```

我们也可以有一个单行定义：

```
zarrelli:~$ print_me() { echo "This is your input:"; echo "$1"; } ; print_me 1 This is your input: 1

```

注意最后一个命令后的`;`。我们在第四章*, **引用和转义**中也看到了匿名函数的使用。

无论你想使用什么样的声明，函数只需通过调用它的名称并接受位置参数来触发，如下所示：

```
function_name arg1 argn

```

正如我们在前一章所看到的，函数可以返回一个值，因为请记住，函数内部处理的值只有在函数被触发后才可用：

```
#!/bin/bash
minor()
{
if (("$1" < "$2"))
then
echo "Great! The integer $1 is less than $2"
echo "Assigning \$1 to the variable \"var\"" 
var="$1"
echo "The value of var inside the function is: $var"
else
echo "The integer $1 is not less than $2..."
fi
}
echo "The value of var outside the function before it is triggered is: $var"
minor "$1" "$2"
echo "The value of var outside the function after it is triggered is: $var"

```

在这个示例中，我们将第一个位置变量的值赋给了名为`var`的变量，然后在函数内外打印这个值，直到它被触发之前以及触发之后：

```
zarrelli:~$ ./minor-function.sh 1 2 The value of var outside the function before it is triggered is: Great! The integer 1 is less than 2 Assigning S1 to the variable "var" The value of var inside the function is: 1 The value of var outside the function after it is triggered is: 1

```

我们注意到了一些有趣的事情。

如果我们尝试在函数触发之前打印 `var` 的值，我们什么也得不到。这是因为尽管函数的代码在命令之前被读取，但 `echo "The value of var outside the function before it is triggered is: $var"` 这条语句在函数触发之前执行，因而没有机会操作变量并给 `var` 赋值。

其次，`echo "The value of var outside the function before it is triggered is: $var"` 这条语句虽然在函数之前执行，但它实际上是第一个在终端上打印的消息。

在函数内部赋值或创建的内容在函数外部是可用的，因为函数在与脚本相同的 Shell 上下文中运行，因此它们共享相同的环境和变量。但如果我希望创建只在函数内部可用的变量呢？我们可以通过在变量前添加 `local` 内建命令来修改赋值指令：

```
local var= "$1"

```

我们再次运行脚本：

```
zarrelli:~$ ./minor-function.sh 1 2 The value of var outside the function before it is triggered is: Great! The integer 1 is less than 2 Assigning S1 to the variable "var" The value of var inside the function is: 1 The value of var outside the function after it is triggered is: 

```

我们可以将变量从脚本的主体中隐藏，但我们也可以让函数返回一些值：

```
#!/bin/bash
OK=10
NOT_OK=50
minor()
{
if (("$1" < "$2"))
then
echo "Returning the value of OK"
return "$OK"
else
echo "Returning the value of NOT_OK"
return "$NOT_OK"
fi
}
print_return()
{
if (("$3" == "$OK")) ; then
echo "Great! The integer $1 is less than $2"
exit 0
elif (("$3" == "$NOT_OK")) ; then
echo "The integer $1 is not less than $2..."
exit 1
else
echo "Something gone wild..."
echo "The first integer has the value of $1 and the second of $2..."
exit 1
fi 
}
minor "$1" "$2"
print_return "$1" "$2" "$?"

```

在这个示例中，我们玩了一下创建一个新函数来打印实际的消息，并且我们看到使用函数的一个初步好处：代码中的整数比较现在变得更简洁，行数更少，可读性更强。另一方面，打印返回值时，输入的是前两个位置参数，第三个参数是 `minor` 函数的返回代码（`$?`）。引入一个专注于打印消息的函数的另一个好处是，它帮助我们实现了分层：`print_return` 函数负责呈现层，`minor` 函数负责处理层。因此，每次我们想修改显示信息的方式时，就不需要修改核心函数，从而避免了引入任何错误。另一方面，如果我们想修改核心函数，我们可以随意更改，而不需要修改呈现层，只要核心输出保持不变。

如果你有一些认为可以在多个脚本中使用的函数，将它们写在一个文件里，然后在脚本中引用该文件并从中使用这些函数是个不错的主意。这样，你就拥有了一个自己的函数库，在需要时可以重复使用，避免每次都要编写相同的代码。

但是我们能否将引用其他变量的变量传递给函数呢？让我们试试这个：

```
zarrelli:$ cat inference.sh #!/bin/bash FIRST_VALUE=SECOND_VALUE SECOND_VALUE=20 print_value() { echo "The value of \$1 is: $1" } print_value "${FIRST_VALUE}" exit 0

```

所以，`FIRST_VALUE` 引用了 `SECOND_VALUE`，而 `SECOND_VALUE` 的值是 `20`，因此我们期待在尝试打印 `$FIRST_VALUE` 时，看到 `20`：

```
zarrelli:~$ ./inference.sh The value of $1 is: SECOND_VALUE

```

这并不是我们预期的结果，对吧？之所以会这样，是因为 Bash 将变量名 `SECOND_VALUE` 视为一串字符。它只是一个按字面值取用的字符串，而不是指向值为`(20)`的指针。不过我们依然可以解决这个问题；只需在之前的脚本中添加 `print_value "${!FIRST_VALUE}"`，并放在 `exit 0` 之前，然后再运行它：

```
zarrelli:~$ ./inference.sh The value of $1 is: SECOND_VALUE The value of $1 is: 20

```

我们使用了所谓的间接引用来实际引用一个值的值。这种语法 `$``{!variable_name}` 是在 Bash 2 中引入的，它使得间接引用不再那么难以书写，但有时你可能会遇到旧版本：

```
zarrelli:~$ a=b ; b=c ; echo $a ; eval a=\$$a ; echo $a b c

```

我们看到的 `$$a` 实际上是值的值，然后我们通过 `eval` 对其进行转义，强制评估并将其赋值给 `a`。

那么，在将变量传递给函数后，如何进行解引用呢？以下是一些可以尝试的代码行：

```
#!/bin/bash
a10=20
print_value()
{
echo -e
echo -e "The name of the variable passed as \$1 to the function is: $1\n"
b20=\$"$1"
echo -e "b20 holds the reference to the content of the variable passed on the command line: $b20\n"
c30=${b20//[[:punct:]][[:alpha:]]}
echo -e "But playing with parameter substitution we got an untyped value out of it: $c30\n"
eval d40=\$$1
e50=$(($d40+$c30))
echo "And we used it as in integer to add to the original value we received"
echo -e "as input so the integer extracted from the name of the variable added to the variable value is: $e50\n" 
eval $1=$e50
echo -e "Thanks to eval we assign the new value to the original input\n" 
echo -e "The value of \$1 now is: $e50\n"
}
echo -e
echo "The value of a10 before triggering the function is: $a10"
print_value a10
echo -e "The value of a10 after triggering the function is: $a10\n"
exit 0

```

所以我们从一个叫做 `a10` 的变量开始，它的值是 `10`。然后，我们在触发函数之前打印了它的值，接着调用函数并传递了该变量的名称。在 `print_value` 函数的第一步中，打印了传递给该函数的第一个位置参数的值。现在，你拥有了足够的知识来理解这段代码及其操作。我们稍微玩了一下间接引用、解引用和参数替换，所以脚本的简单输出应该能让一切变得清晰：

```
zarrelli:~$ ./dereference.sh The value of a10 before triggering the function is: 20 The name of the variable passed as $1 to the function is: a10 b20 holds the reference to the content of the variable passed on the command line: $a10 But playing with parameter substitution we got an untyped value out of it: 10 And we used it as in integer to add to the original value we received as input so the integer extracted from the name of the variable added to the variable value is: 30 Thanks to eval we assign the new value to the original input The value of $1 now is: 30 The value of a10 after triggering the function is: 30

```

现在我们明白了：`a10` 的值从原来的 `20` 变成了新的 `30`，而且现在我们知道了原因和过程。在离开函数章节之前，再提几点：我们已经讨论过匿名函数：

```
zarrelli:~$ x=10 ; y=5 ; { z=$(($x*$y)) ; echo "Value of z inside the function: $z" ; } ; echo "Value of z outside the function: $z" Value of z inside the function: 50 Value of z outside the function: 50

```

只要记住在结束大括号之前的最后一个分号，并查看返回的值：

```
zarrelli:~$ cat minor-function-return-message.sh #!/bin/bash OK=10 NOT_OK=50 minor() { if (("$1" < "$2")) then echo "Returning the value of OK" return "$OK" else echo "Returning the value of NOT_OK" return "$NOT_OK" fi } message=$(minor "$1" "$2") echo "$message"

```

一旦调用，脚本会输出一个有意义的错误信息，克服了内置 `return` 的限制，它只能返回整数：

```
zarrelli@moveaway:~/Documents/My books/Mastering bash/Chapter 5/Scripts$ ./minor-function-return-message.sh 1 2

```

之前的代码返回了 `OK` 的值，并且作为代码块，函数 `stdin` 和 `stdout` 可以很容易地重定向：

```
zarrelli:~$ cat redirect.sh #!/bin/bash file=friends.txt parse() { while read lineofile do echo $lineofile done }<$file parse

```

这将给我们以下输出：

```
zarrelli:~$ ./redirect.sh Anthony Dionisios Ilaria Mike Noel Tarek

```

现在就这些，接下来是时候为我们的脚本加入一些趣味了。

# 总结

你已经学会了如何与用户交互、读取他们的输入并将其存储在合适的结构中，循环遍历值，并利用函数让我们的代码更加整洁和可重用。现在是时候探索一些我们已经使用过的结构了：我们在谈论的是迭代。
